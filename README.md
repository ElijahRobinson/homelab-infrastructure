# Homelab Infrastructure

Full homelab setup documentation including reverse proxy, DNS ad-blocking, monitoring, and media stack running on Proxmox VE.

## Architecture

| LXC | IP | Services |
|-----|----|----------|
| jellyfin | 10.0.0.134 | Jellyfin, qBittorrent, NordVPN, FlareSolverr |
| adguard | 10.0.0.137 | AdGuard Home, Tailscale |
| mediastack | 10.0.0.136 | NPM, Sonarr, Radarr, Prowlarr, Jellyseerr, Uptime Kuma, Homepage, Watchtower, FlareSolverr |

**Proxmox host:** 10.0.0.234 (Tailscale IP: 100.92.204.101)  
**Domain:** idiotproductions.net (Cloudflare)

---

## Services

### Reverse Proxy — Nginx Proxy Manager
- Wildcard SSL cert via Cloudflare DNS challenge (`*.idiotproductions.net`)
- All services accessible over HTTPS
- Running on `10.0.0.136:81`

### DNS & Ad Blocking — AdGuard Home
- Running on `10.0.0.137`
- DNS rewrite: `*.idiotproductions.net` → `10.0.0.136`
- Upstream DNS: Cloudflare (1.1.1.1)

### Remote Access — Tailscale
- Installed on Proxmox host (pve01)
- Subnet routing: `10.0.0.0/24` advertised
- Split DNS: `idiotproductions.net` queries routed via BIND9 on Proxmox (port 53)
- Tailscale DNS nameserver: `100.92.204.101`

### Monitoring — Uptime Kuma
- Accessible at `https://uptime.idiotproductions.net`
- Discord webhook notifications
- Monitors: Jellyfin, Jellyseerr, Sonarr, Radarr, Prowlarr, NPM, AdGuard, Uptime Kuma

### Dashboard — Homepage
- Accessible at `https://homepage.idiotproductions.net`
- Displays all services with icons

### Auto-updates — Watchtower
- Runs daily at 4am UTC
- Auto-removes old images
- Monitors all containers on host

---

## Proxmox LXC Configuration

All LXCs use Ubuntu 22.04. For Tailscale and Docker to work in unprivileged LXCs, add to `/etc/pve/lxc/<ID>.conf`:

```
features: keyctl=1,nesting=1
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

---

## Cloudflare Setup

- DNS record: `*.idiotproductions.net` A → `10.0.0.136` (DNS only, grey cloud)
- API token: Zone:DNS:Edit permission on `idiotproductions.net` (used by NPM for DNS challenge)

---

## Remote Access Flow

```
Phone (Tailscale) → DNS query for *.idiotproductions.net
  → Tailscale split DNS → BIND9 on pve01 (100.92.204.101)
  → Resolves to 10.0.0.136
  → Tailscale subnet route → NPM on mediastack LXC
  → Proxied to correct service
```

---

## Notes

- NordVPN runs in the Jellyfin LXC for qBittorrent only. Its kill switch blocks Docker container networking, which is why all infrastructure services were moved to a separate LXC.
- AdGuard LXC IP was changed from 10.0.0.135 to 10.0.0.137 due to IP conflict with a Nintendo Switch on the network.
- BIND9 is installed on Proxmox host to serve DNS over Tailscale interface.
- GL.iNet Flint 2 router pending setup for network-wide DNS via AdGuard.
