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
    │   └── .60 — Nginx Proxy Manager (reverse proxy)
    ├── VLAN02 (SERVERS) — 10.0.20.0/24
    └── VLAN03 (IOT) — 10.0.30.0/24
```

## Hardware

| Device | Role |
|--------|------|
| Lenovo M710q (i7, 16GB, 2x10G SFP+) | OPNsense Firewall |
| Proxmox Homelab | Main server, Suricata IDS |
| Proxmox K-Cluster | Kubernetes cluster |
| Proxmox GamingServer | GPU workloads, mining |
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
├── kubernetes/       # K-Cluster setup and workloads
├── scripts/          # Automation scripts
└── diagrams/         # Network diagrams
```

---

*B-Clarke Technology LLC*
