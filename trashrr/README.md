# Media Stack with VPN Routing via Gluetun

This setup routes multiple media services through a VPN using [Gluetun](https://github.com/qdm12/gluetun). It leverages Docker Compose and shares a common external network `fetchrr_net` across all services.

---

## 🔧 Requirements

- Docker & Docker Compose
- Existing external Docker network: `fetchrr_net`
- Valid ProtonVPN WireGuard configuration
- Environment variables (`.env` file or environment) for:
  - `PUID`, `PGID`
  - `TZ`
  - `wgPrivateKey`
  - `country`

---

## 📦 Services

### 🔐 Gluetun (VPN Gateway)

- VPN Provider: ProtonVPN (WireGuard)
- Enables port forwarding for qBittorrent
- Exposes ports for:
  - 8090: qBittorrent Web UI
  - 8989: Sonarr
  - 9696: Prowlarr
  - 7878: Radarr
  - 9117: Jackett
  - 8888, 8388 (HTTP/Shadowsocks)

### 🧲 qBittorrent

- Runs entirely through Gluetun using `network_mode: service:gluetun`
- Web UI available at `http://localhost:8090`
- Stores downloads and configs in mounted volumes

### 🔎 Jackett

- Indexer proxy
- Config stored in `/volume1/docker/jackett/config`
- Routed through Gluetun

### 🔍 Prowlarr

- Manages indexers for Sonarr and Radarr
- Shares `/volume1/Media` with other services
- Routed through Gluetun

### 🎬 Radarr

- Movie automation tool
- Routed through Gluetun
- Config and data directories mapped

### 📺 Sonarr

- TV series automation tool
- Routed through Gluetun
- Shares media volume with Radarr and Prowlarr

### 📝 Bazarr

- Subtitle management tool
- Not routed through Gluetun (direct connection on port `6767`)
- Mounts media path for subtitle management

---

## 🌐 Network

All services connect to an external Docker network named `fetchrr_net`.  
Ensure it exists before running:

```bash
docker network create fetchrr_net
```

---

## 🌐 Additional Services

### ☁️ Cloudflared

- Securely tunnels local services to the internet using Cloudflare
- Uses $TUNNEL_TOKEN
- Joins the same external network: fetchrr_net

### 🎥 Plex

- Media server for local/remote streaming
- Uses host networking (bypasses Gluetun)
- Optional mod: plex-absolute-hama for better metadata parsing
- VAAPI hardware acceleration enabled via /dev/dri

### 📺 Jellyfin

- Open-source alternative to Plex, supports DLNA and hardware transcoding
- Hardware acceleration via VAAPI

### 💬 Requestrr

- Media request bot for Discord
- Handles requests via chat
