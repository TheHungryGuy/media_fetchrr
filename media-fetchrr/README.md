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

## Connecting to Other Containers

Other containers can access the Gluetun container using `network_mode`.

<details>
<summary><code>.../other/docker-compose.yml</code></summary>

```yaml
services:
  myservice:
    image: myimage
    network_mode: container:gluetun
```

</details>

If both are in the same `docker-compose` file, the service name can be used.

<details>
<summary><code>.../gluetun/docker-compose.yml</code></summary>

```yaml
services:
  gluetun: ...
  myservice:
    image: myimage
    network_mode: service:gluetun
```

</details>

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
