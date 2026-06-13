# Media Server Stack (Jellyfin, *Arrs, qBittorrent, Nginx, DuckDNS)

This project sets up a comprehensive self-hosted **media server stack** using Docker Compose. It includes a media server ([Jellyfin](https://jellyfin.org/)), automation tools ([Radarr](https://radarr.video/), [Sonarr](https://sonarr.tv/), [Prowlarr](https://prowlarr.com/), [Jellyseerr](https://github.com/Fallenbagel/jellyseerr)), a torrent client ([qBittorrent](https://www.qbittorrent.org/)) securely routed through a VPN ([Gluetun](https://github.com/qdm12/gluetun)), a Cloudflare bypass tool ([FlareSolverr](https://github.com/FlareSolverr/FlareSolverr)), fronted by **Nginx** for reverse proxy, and dynamically updated DNS provided by **DuckDNS**.

---

## 📦 Services Overview

### 1. **Nginx**
- Acts as a reverse proxy for the stack.
- Handles HTTP (80) and HTTPS (443) traffic.
- Configurable via `nginx/arr.default.conf`.

### 2. **DuckDNS**
- Provides a free dynamic DNS service.
- Automatically updates your IP address for your DuckDNS subdomain.
- Credentials are managed centrally in the `.env` file.

### 3. **Jellyfin**
- An open-source media server for movies, TV shows, and music.
- Accessible via web browser or apps (mobile, smart TV, etc.).
- Mounts your local media libraries into the container at `/data`.

### 4. **Jellyseerr**
- Request management and media discovery tool for Jellyfin.
- Allows users to request missing movies and TV shows, integrating directly with Radarr and Sonarr.

### 5. **Radarr & Sonarr**
- **Radarr**: Manages movie collections, automates downloading, and organizes files.
- **Sonarr**: Manages TV show collections, automates downloading, and organizes files.
- Both integrate with Prowlarr for indexers and qBittorrent for downloads.

### 6. **Prowlarr**
- Indexer manager that integrates with Radarr, Sonarr, and other *Arr apps.
- Centralizes the management of torrent and Usenet indexers.

### 7. **Gluetun & qBittorrent**
- **Gluetun**: A lightweight VPN client container that routes traffic securely. Configured here for **Surfshark** using **OpenVPN**.
- **qBittorrent**: Torrent client that shares Gluetun's network (`network_mode: "service:gluetun"`), ensuring all torrent traffic goes through the VPN. If the VPN drops, qBittorrent loses internet access, preventing IP leaks.

### 8. **FlareSolverr**
- A proxy server to bypass Cloudflare and DDoS-IT protection pages, often required by Prowlarr and various indexers.

---

## 📂 Project Structure

```text
.
├── docker-compose.yml       # Main Docker Compose configuration
├── .env                     # Environment variables (credentials, versions, domains)
├── nginx/
│   └── arr.default.conf     # Nginx reverse proxy configuration
├── duckdns/                 # DuckDNS container config/data
├── jellyfin/
│   ├── config/              # Jellyfin configuration and database
│   └── cache/               # Jellyfin transcoding/image cache
├── jellyseerr/
│   └── config/              # Jellyseerr configuration
├── prowlarr/
│   └── config/              # Prowlarr configuration
├── radarr/
│   └── config/              # Radarr configuration
├── sonarr/
│   └── config/              # Sonarr configuration
├── gluetun/
│   └── config/              # Gluetun VPN configuration
├── qbittorrent/
│   └── config/              # qBittorrent configuration
└── /mnt/c/Media/            # (External) Your media library directory
```

---

## ⚙️ Configuration

### 1. Environment Variables (`.env`)
All credentials, subdomains, and application versions are stored in the `.env` file. **Update the following before starting:**

```sh
cp sample.env .env
```

```env
# VPN credentials (e.g., Surfshark)
VPN_USER=your_vpn_username
VPN_PASS=your_vpn_password

# qBittorrent default credentials
QBIT_USER=admin
QBIT_PASS=your_secure_password

# DuckDNS credentials
DUCKDNS_DOMAIN=your_subdomain
DUCKDNS_TOKEN=your_duckdns_token

# Jellyfin published URL (used for external access links)
JELLYFIN_PUBLISHED_URL=https://your_subdomain.duckdns.org
```

### 2. Media Directories
- Ensure your media is available at `/mnt/c/Media` (or update the host paths in `docker-compose.yml`).
- This directory is mounted as `/data` inside Jellyfin, Radarr, Sonarr, and qBittorrent.
- **Important**: Ensure Radarr, Sonarr, and qBittorrent use consistent paths (e.g., `/data/Movies` and `/data/TV`) to enable hardlinking/atomic moves.

### 3. Nginx
- Edit `nginx/arr.default.conf` to set up reverse proxy rules for your services and configure SSL (e.g., using Let's Encrypt).

### 4. Gluetun (VPN)
- The stack is pre-configured for **Surfshark** using **OpenVPN**. If you use a different VPN provider, update the `VPNSP` and related environment variables in the `gluetun` service in `docker-compose.yml`.

---

## 🚀 Usage

### Start the stack
```bash
docker compose up -d
```

### Stop the stack
```bash
docker compose down
```

### Update the stack
```bash
docker compose pull
docker compose up -d
```

### Check logs
```bash
# View logs for a specific service
docker compose logs -f jellyfin
docker compose logs -f gluetun
docker compose logs -f qbittorrent
```

---

## 🌍 Access & Ports

Once the stack is running, you can access the web interfaces via their respective local ports:

| Service       | Local URL                           | Notes                                      |
|---------------|-------------------------------------|--------------------------------------------|
| **Nginx**     | `http://localhost` / `https://...`  | Reverse proxy (Ports 80, 443)              |
| **Jellyfin**  | `http://localhost:8096`             | Media Server                               |
| **Jellyseerr**| `http://localhost:5055`             | Request Management                         |
| **Radarr**    | `http://localhost:7878`             | Movie Management                           |
| **Sonarr**    | `http://localhost:8989`             | TV Show Management                         |
| **Prowlarr**  | `http://localhost:9696`             | Indexer Management                         |
| **qBittorrent**| `http://localhost:8080`            | Torrent Client (Routed via Gluetun VPN)    |
| **FlareSolverr**| `http://localhost:8191`           | Cloudflare Bypass Proxy                    |

*Note: For remote access, configure Nginx to reverse proxy these internal ports to your DuckDNS domain.*

---

## 🔒 Security & Networking Notes

- **VPN Routing**: qBittorrent is strictly routed through the Gluetun VPN container. The kill-switch is inherent to the `network_mode: "service:gluetun"` setup.
- **Reverse Proxy**: Do **not** expose the *Arr apps or qBittorrent directly to the internet. Use Nginx to reverse proxy them and enforce HTTPS/Authentication.
- **Firewall**: Gluetun is configured with `FIREWALL_LAN_INPUT_PORTS=8080` to allow local network access to the qBittorrent WebUI while keeping the rest of the VPN container secure.
- **Secrets**: Keep your `.env` file private and never commit it to public repositories.

---

## 🛠️ Requirements

- **Docker & Docker Compose** (v2+) installed.
- **Media Library**: A directory with your media files (default: `/mnt/c/Media`).
- **Accounts**:
  - A **DuckDNS** account with a registered subdomain.
  - A **VPN** account that supports OpenVPN/WireGuard (configured for Surfshark by default).
  - (Optional) Indexer accounts to add into Prowlarr.

---

## 📖 References

- [Jellyfin Documentation](https://jellyfin.org/docs/)
- [LinuxServer.io Images](https://docs.linuxserver.io/) (Radarr, Sonarr, Prowlarr, qBittorrent, DuckDNS)
- [Gluetun Wiki](https://github.com/qdm12/gluetun-wiki)
- [Jellyseerr Documentation](https://github.com/Fallenbagel/jellyseerr/wiki)
- [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr)
- [Nginx Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)