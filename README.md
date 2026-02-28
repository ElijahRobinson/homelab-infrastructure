# Homelab Infrastructure

Full homelab setup running on Proxmox VE — reverse proxy, DNS ad-blocking, remote access, monitoring, and a complete media automation stack.

## Hardware

| Component | Details |
|-----------|---------|
| Host | HP ProDesk 300 |
| Hypervisor | Proxmox VE |
| Domain | idiotproductions.net (Cloudflare) |
| Proxmox IP | 10.0.0.234 |
| Tailscale IP | 100.92.204.101 |

---

## Architecture

Three LXC containers running Ubuntu 22.04, each with a dedicated role:

| LXC | IP | Services |
|-----|----|---------|
| jellyfin | 10.0.0.134 | Jellyfin, qBittorrent, NordVPN, FlareSolverr |
| mediastack | 10.0.0.136 | Nginx Proxy Manager, Sonarr, Radarr, Prowlarr, Jellyseerr, Uptime Kuma, Homepage, Watchtower |
| adguard | 10.0.0.137 | AdGuard Home, Tailscale |

The jellyfin and mediastack LXCs are intentionally split — NordVPN's kill switch inside the jellyfin LXC would otherwise block Docker container networking for the media management services.

```
Internet → Cloudflare DNS → Nginx Proxy Manager (10.0.0.136)
                                      ↓
                          Routes to internal services
                          
Phone (Tailscale) → Split DNS → BIND9 on Proxmox host
                                      ↓
                          Resolves *.idiotproductions.net → 10.0.0.136
                                      ↓
                          Tailscale subnet route → NPM → service
```

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

### Media Stack
- **Jellyfin** — media server, streams to local network and via Tailscale
- **Jellyseerr** — request management UI, integrates with Sonarr/Radarr
- **Sonarr** — TV show automation
- **Radarr** — movie automation
- **Prowlarr** — indexer manager, feeds Sonarr and Radarr
- **qBittorrent** — torrent client, runs behind NordVPN with kill switch enabled

### Monitoring — Uptime Kuma
- Accessible at `https://uptime.idiotproductions.net`
- Discord webhook notifications
- Monitors: Jellyfin, Jellyseerr, Sonarr, Radarr, Prowlarr, NPM, AdGuard

### Dashboard — Homepage
- Accessible at `https://homepage.idiotproductions.net`
- Aggregates all services with status icons

### Auto-updates — Watchtower
- Runs daily at 4AM UTC
- Auto-removes old images
- Monitors all containers on host

---

## Proxmox LXC Configuration

All LXCs run Ubuntu 22.04 unprivileged. For Tailscale and Docker to work, add to `/etc/pve/lxc/<ID>.conf`:

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

## Troubleshooting

### qBittorrent not downloading — NordVPN kill switch

**Symptom:** Sonarr/Radarr queue shows items as "Queued", torrents appear in qBittorrent but sit at 0% with 0 peers and 0 seeds. DHT shows 0 nodes.

**Cause:** NordVPN is installed as a system service inside the jellyfin LXC with the kill switch enabled. If NordVPN disconnects for any reason (reboot, crash, token expiry), all network traffic from qBittorrent is blocked.

**Fix:**
```bash
# Check VPN status
nordvpn status

# Reconnect if disconnected
nordvpn connect

# Whitelist LAN so local services (Sonarr/Radarr) can always reach qBittorrent
nordvpn whitelist add subnet 10.0.0.0/24

# Enable auto-connect so VPN reconnects on reboot
nordvpn set autoconnect on
```

**Why the LAN whitelist matters:** Sonarr and Radarr run on a different LXC (10.0.0.136) and communicate with qBittorrent over the local network. Without the subnet whitelist, the kill switch can block this cross-LXC traffic even when the VPN is connected.

---

### NordVPN overwrites /etc/resolv.conf

**Symptom:** DNS failures inside the jellyfin LXC after NordVPN connects.

**Cause:** NordVPN replaces `/etc/resolv.conf` with its own DNS servers, which can break local resolution.

**Fix:** Cron job to restore DNS every minute:

```bash
# /etc/nordvpn/dns-fix.sh
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 1.1.1.1" >> /etc/resolv.conf
```

```bash
# Add to crontab
* * * * * /etc/nordvpn/dns-fix.sh
```

---

### AdGuard LXC IP conflict

**Symptom:** Network conflicts on 10.0.0.135.

**Cause:** A Nintendo Switch on the network claimed 10.0.0.135 via DHCP.

**Fix:** Reassigned AdGuard LXC to 10.0.0.137. Updated DNS rewrite rules and all service configs to reflect the new IP.

---

## Notes

- NordVPN runs in the jellyfin LXC for qBittorrent only. Its kill switch breaks Docker networking, which is why all infrastructure services live in a separate mediastack LXC.
- BIND9 is installed on the Proxmox host to serve DNS queries coming in over the Tailscale interface.
- GL.iNet Flint 2 router pending setup for network-wide DNS via AdGuard.
