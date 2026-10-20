# 🏠 Homelab Documentation

Enterprise-grade infrastructure built and managed from scratch.

## Architecture

```
Internet → OPNsense Firewall (10.0.0.1)
    ├── VLAN10 (LAN) — 10.0.10.0/24
    │   ├── .10 — Pi-hole (DNS — primary + backup)
    │   ├── .20 — Homelab Server (Proxmox — main host)
    │   ├── .30 — Grafana + Prometheus (monitoring)
    │   ├── .40 — Open WebUI (ai.clarkelab.dev)
    │   ├── .50 — Vaultwarden (vault.clarkelab.dev)
    │   ├── .60 — Nginx Proxy Manager (reverse proxy)
    │   ├── .70 — K3s control plane (k3s-cp, LXC on K-Cluster)
    │   └── .71–.75 — K3s worker nodes (bare metal)
    ├── VLAN02 (SERVERS) — 10.0.20.0/24
    └── VLAN03 (IOT) — 10.0.30.0/24
```

> IPs in this repo are documentation placeholders. Real addressing and credentials stay in the lab.

## Hardware

| Device | Role |
|--------|------|
| Lenovo M710q (i7, 16GB, 2x10G SFP+) | OPNsense Firewall |
| Proxmox Homelab | Main server, Suricata IDS |
| Proxmox K-Cluster | Control plane host, GPU node |
| Proxmox GamingServer | GPU workloads, mining |
| 5× bare-metal nodes (8 cores, 14–22 GB) | K3s workers |
| Cisco 3850 (clab-core01) | Core switch, VLANs |
| Synology DS920+ "Cheesecake" | Storage, backups |

## Security Stack

- **Suricata IDS/IPS** — 52k+ rules, Prometheus metrics
- **OPNsense** — Stateful firewall, VLANs, DHCP
- **Pi-hole** — DNS filtering (primary + backup)
- **Vaultwarden** — Password management

## Monitoring

- **Prometheus** — Metrics collection
- **Grafana** — Dashboards (Node Exporter Full)
- **Uptime Kuma** — Service health checks

## Kubernetes

**K3s v1.36.5** — one Proxmox-hosted control plane plus five bare-metal workers, all on the service VLAN.

- Control plane runs in a privileged LXC with `/dev/kmsg` bound in, so the kubelet behaves exactly as it would on metal
- Workers are Ubuntu 24.04 LTS agents joined with a shared node token
- Traefik ingress and `local-path` storage come bundled; **Rancher** (with cert-manager) provides the GUI, and Longhorn is the next addition
- Automated node provisioning via PXE (Ubuntu 24.04 autoinstall) + Ansible inventory

See **[kubernetes/](./kubernetes/)** for topology, build steps, operations, and the pitfalls hit along the way.

## Services

- **Open WebUI** — AI interface
- **Nginx Proxy Manager** — Reverse proxy, SSL termination
- **Home Assistant** — Smart home automation
- **Pi-hole** — DNS ad-blocking

## Docs Structure

```
homelab/
├── network/          # Network topology, VLANs, firewall rules
├── security/         # Suricata config, IDS rules, hardening
├── monitoring/       # Prometheus, Grafana dashboards
├── services/         # Docker compose, service configs
├── kubernetes/       # K3s cluster: topology, build, operations
├── scripts/          # Automation scripts
└── diagrams/         # Network diagrams
```

---

*B-Clarke Technology LLC*
