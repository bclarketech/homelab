# ☸️ Kubernetes — K3s Cluster

Production-style K3s cluster running on one Proxmox-hosted control plane and five bare-metal worker nodes.

## Design

| Layer | Choice | Why |
|-------|--------|-----|
| Distribution | **K3s** | Single binary, low overhead, easy to shell into and debug |
| Control plane | LXC container on Proxmox (K-Cluster host) | Keeps the API server patched and snapshot-able with the rest of the lab |
| Workers | Bare metal, Ubuntu 24.04 LTS | Full hardware access for workloads, no nested-virtualization tax |
| Ingress | Traefik (K3s default) | Bundled, no extra install |
| Storage | local-path provisioner (default) | Simple node-local volumes; Longhorn is the next step |
| Management | **Rancher** + cert-manager | Web GUI, cluster health, and future GitOps |
| Runtime | containerd 2.3.4-k3s1 | Bundled with K3s |

> All addresses below are documentation placeholders on the lab's `10.0.10.0/24` service VLAN. Real addressing, tokens, and kubeconfigs are never committed.

## Topology

```
                 ┌──────────────────────────────┐
                 │  k3s-cp  (10.0.10.70)        │
                 │  Control plane — LXC on PVE  │
                 │  4 vCPU / 4 GB / 20 GB       │
                 │  k3s v1.36.5+k3s1            │
                 └──────────────┬───────────────┘
                                │  :6443
      ┌─────────────┬───────────┼───────────┬─────────────┐
      │             │           │           │             │
 ┌────▼────┐  ┌─────▼────┐ ┌────▼─────┐ ┌───▼──────┐ ┌────▼─────┐
 │ node-03 │  │ node-04  │ │ node-05  │ │ node-06  │ │ node-07  │
 │ .71     │  │ .72      │ │ .73      │ │ .74      │ │ .75      │
 │ 8c/14GB │  │ 8c/14GB  │ │ 8c/14GB  │ │ 8c/22GB  │ │ 8c/14GB  │
 └─────────┘  └──────────┘ └──────────┘ └──────────┘ └──────────┘
```

## Node Inventory

| Node | Role | vCPU / RAM | OS | K3s |
|------|------|-----------|----|-----|
| `k3s-cp` | control-plane | 4 / 4 GB | Ubuntu 24.04 (LXC) | v1.36.5+k3s1 |
| `k8s-node-03` | worker | 8 / 14 GB | Ubuntu 24.04.5 | v1.36.5+k3s1 |
| `k8s-node-04` | worker | 8 / 14 GB | Ubuntu 24.04.5 | v1.36.5+k3s1 |
| `k8s-node-05` | worker | 8 / 14 GB | Ubuntu 24.04.5 | v1.36.5+k3s1 |
| `k8s-node-06` | worker | 8 / 22 GB | Ubuntu 24.04.5 | v1.36.5+k3s1 |
| `k8s-node-07` | worker | 8 / 14 GB | Ubuntu 24.04.5 | v1.36.5+k3s1 |

## Build Steps

### 1. Control plane (LXC)

The container needs to be **privileged with nesting enabled** — the kubelet expects `/dev/kmsg`, which an unprivileged container does not expose:

```bash
pct set <vmid> --features nesting=1,keyctl=1 --unprivileged 0
pct set <vmid> -lxc.mount.entry /dev/kmsg dev/kmsg none bind,optional,create=file
pct set <vmid> -lxc.apparmor.profile unconfined
pct set <vmid> -lxc.cap.drop ""
pct set <vmid> -lxc.mount.auto "proc:rw sys:rw"
pct set <vmid> -lxc.cgroup2.devices.allow a
```

Then install the server:

```bash
curl -sfL https://get.k3s.io | \
  INSTALL_K3S_EXEC="server --write-kubeconfig-mode 644 --node-name k3s-cp" sh -
```

`/dev/kmsg` is the critical piece — without it the kubelet crash-loops and the API server never becomes reachable.

### 2. Workers

On each bare-metal node:

```bash
curl -sfL https://get.k3s.io | \
  K3S_URL=https://10.0.10.70:6443 \
  K3S_TOKEN="<K3S_NODE_TOKEN>" sh -
```

The join token lives on the control plane at `/var/lib/rancher/k3s/server/node-token` and must never be committed.

### 3. Verify

```bash
sudo k3s kubectl get nodes -o wide
sudo k3s kubectl get pods -A
```

Expected: one `control-plane` node plus five `Ready` workers running `v1.36.5+k3s1`.

## Operations

### Add a worker
Install with the same `K3S_URL` + `K3S_TOKEN` pair. No control-plane changes required.

### Drain a node for maintenance
```bash
sudo k3s kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
# ... maintenance ...
sudo k3s kubectl uncordon <node>
```

### Rotate the join token
```bash
sudo systemctl stop k3s
sudo rm /var/lib/rancher/k3s/server/token
sudo systemctl start k3s   # regenerates
sudo cat /var/lib/rancher/k3s/server/node-token
```

### Rancher management UI

Rancher runs in the `cattle-system` namespace behind Traefik, with cert-manager issuing its certs:

```bash
sudo k3s kubectl get pods -n cattle-system
sudo k3s kubectl get svc -n cattle-system rancher
```

Reachable at `https://rancher.<control-plane-ip>.sslip.io` — Traefik's LoadBalancer publishes 80/443 on the control plane and every worker, so any node IP serves the UI. The bootstrap admin password is set on first login and stored in the password manager; it is never documented in this repo.

### Upgrade
K3s upgrades in place — bump the install channel or re-run the installer with the target version:

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.36.5+k3s1 sh -
```

Workers first, control plane last.

## Pitfalls Learned

| Symptom | Cause | Fix |
|---------|-------|-----|
| Kubelet crash-loop, API unreachable inside LXC | `/dev/kmsg` not present | Bind-mount `/dev/kmsg` and set `lxc.mount.auto: proc:rw sys:rw` |
| `kubectl` inside container hangs on `127.0.0.1:6443` | API server never started (kubelet dependency) | Same `/dev/kmsg` fix, then restart `k3s` |
| Worker joins but immediately fails, port `6444` in use | Leftover **k3s server** from an earlier install on that node | `sudo /usr/local/bin/k3s-uninstall.sh`, remove `/etc/rancher/k3s/config.yaml`, reinstall as agent |
| `etcd`/`server` flags ignored on a worker | Stale `/etc/rancher/k3s/config.yaml` with server-only options | Delete or rename the file before installing the agent |

## Next Steps

- Longhorn for replicated block storage
- TLS for lab apps via cert-manager + the existing Nginx Proxy Manager
- GitOps with Fleet (bundled with Rancher), Argo CD, or Flux
- Monitoring the cluster with the existing Prometheus/Grafana stack
