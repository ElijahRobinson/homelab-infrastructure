# Homelab Status

> Last updated: 2026-02-26

A running overview of what's live, what's in progress, and what's planned.

---

## ✅ Live & Operational

| Service | Description |
|---|---|
| **Proxmox VE** | Hypervisor running all VMs and LXC containers |
| **Jellyfin** | Self-hosted media server with network share storage |
| **Nginx Proxy Manager** | Reverse proxy handling subdomain-based HTTPS access to internal services |
| **Uptime Kuma** | Self-hosted monitoring and uptime tracking for all services |

---

## 🔧 In Progress

| Service | Status | Blocker |
|---|---|---|
| **AdGuard Home** | Awaiting setup | Router on order — will configure DNS ad-blocking once hardware arrives |

---

## 📋 Planned

- Expand monitoring dashboards (Grafana / Prometheus)
- Automate VM provisioning with Ansible
- Document full network topology

---

## 🗒️ Notes

- All services run on a Proxmox-based homelab
- HTTPS access managed through Nginx Proxy Manager with a real domain
- Uptime Kuma monitors all internal services
