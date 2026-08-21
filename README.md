# 🏠 Homelab Documentation

Enterprise-grade infrastructure built and managed from scratch.

## Architecture

```
Internet → OPNsense Firewall (172.16.10.1)
    ├── VLAN10 (LAN) — 172.16.10.0/24
    │   ├── .135 — Pi-hole (DNS)
    │   ├── .136 — Homelab Server (Proxmox)
    │   ├── .146 — Grafana + Prometheus
    │   ├── .106 — Open WebUI (ai.clarkelab.dev)
    │   ├── .118 — Vaultwarden (vault.clarkelab.dev)
    │   └── .150 — Nginx Proxy Manager
    ├── VLAN02 (SERVERS) — 172.16.20.0/24
    └── VLAN03 (IOT) — 172.16.30.0/24
```

## Hardware

| Device | Role | Location |
|--------|------|----------|
| Lenovo M710q (i7, 16GB, 2x10G SFP+) | OPNsense Firewall | 172.16.10.1 |
| Proxmox Homelab (.233) | Main server, Suricata IDS | LXC/VM host |
| Proxmox K-Cluster (.234) | Kubernetes cluster | LXC/VM host |
| Proxmox GamingServer (.235) | GPU workloads, mining | LXC/VM host |
| Cisco 3850 (clab-core01) | Core switch, VLANs | 172.16.10.2 |
| Synology DS920+ "Cheesecake" | Storage, backups | 172.16.10.46 |

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

- **Open WebUI** — AI interface (ai.clarkelab.dev)
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
