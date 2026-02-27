# Homelab Status

> Last updated: 2026-02-26

A running overview of what's live, what's in progress, and what's planned.

---

## ✅ Live & Operational

### 🖥️ Proxmox VE Host
Hypervisor running all LXC containers.

### 📦 LXC: jellyfin (`10.0.0.134`)

| Service | Description |
|---|---|
| **Jellyfin** | Self-hosted media server with network share storage |
| **qBittorrent** | Torrent client for media downloads |
| **NordVPN** | VPN tunnel for download traffic |
| **FlareSolverr** | Cloudflare bypass proxy for indexers |

### 📦 LXC: mediastack (`10.0.0.136`)

| Service | Description |
|---|---|
| **Nginx Proxy Manager** | Reverse proxy handling subdomain-based HTTPS access |
| **Sonarr** | Automated TV show management |
| **Radarr** | Automated movie management |
| **Prowlarr** | Indexer manager for Sonarr & Radarr |
| **Jellyseerr** | Media request and discovery frontend |
| **Uptime Kuma** | Self-hosted uptime monitoring |
| **Homepage** | Homelab dashboard |
| **Watchtower** | Automated Docker container updates |
| **FlareSolverr** | Cloudflare bypass proxy for indexers |

### 📦 LXC: adguard (`10.0.0.137`)

| Service | Description |
|---|---|
| **Tailscale** | VPN mesh network for secure remote access |

---

## 🔧 In Progress

### 📦 LXC: adguard (`10.0.0.137`)

| Service | Status | Blocker |
|---|---|---|
| **AdGuard Home** | Awaiting setup | Router on order — will configure DNS ad-blocking once hardware arrives |

---

## 📋 Planned

- Configure router as DNS gateway pointing to AdGuard Home (`10.0.0.137`)
- Set up Tailscale for remote access
- Expand monitoring dashboards (Grafana / Prometheus)
- Automate container provisioning with Ansible
- Document full network topology

---

## 🗒️ Notes

- All services run as LXC containers on a Proxmox-based homelab
- Download traffic on the `jellyfin` LXC is routed through NordVPN
- HTTPS access managed through Nginx Proxy Manager with a real domain
- Uptime Kuma and Homepage are accessible via NPM subdomains
