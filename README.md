# Media Server with Jellyfin, Nginx, and DuckDNS

This project sets up a self-hosted **media server** using [Jellyfin](https://jellyfin.org/), fronted by **Nginx** for reverse proxy, and dynamically updated DNS provided by **DuckDNS**.  
It is designed to run via **Docker Compose**, making it portable and easy to maintain.

---

## 📦 Services Overview

### 1. **Nginx**
- Acts as a reverse proxy for Jellyfin.
- Handles HTTP (80) and HTTPS (443) traffic.
- Configurable via `nginx/default.conf`.

### 2. **DuckDNS**
- Provides a free dynamic DNS service.
- Automatically updates your IP address for your DuckDNS subdomain.
- Configuration is stored in `duckdns/duckdns.env`.

### 3. **Jellyfin**
- An open-source media server for movies, TV shows, and music.
- Accessible via web browser or apps (mobile, smart TV, etc.).
- Mounts your local media libraries into the container.

---

## 📂 Project Structure

```

.
├── docker-compose.yml       # Main Docker Compose configuration
├── nginx/
│   └── default.conf         # Nginx reverse proxy configuration
└── duckdns/
    └── duckdns.env          # DuckDNS API token and subdomain

````

---

## ⚙️ Configuration

### DuckDNS
1. Create a free account on [DuckDNS](https://www.duckdns.org/).
2. Get your token and subdomain.
3. Add them to `duckdns/duckdns.env`:

```env
SUBDOMAINS=<your-subdomain>
TOKEN=<your-token>
````

### Jellyfin

* Place your media in directories (e.g., `/mnt/c/Media/Movies` and `/mnt/c/Media/Series`).
* These paths are mounted **read-only** inside the container.

### Nginx

* Edit `nginx/default.conf` to match your domain and SSL configuration.

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

### Check logs

```bash
docker logs jellyfin
docker logs nginx
docker logs duckdns
```

---

## 🌍 Access

* Jellyfin Web Interface:
  [http://localhost:8096](http://localhost:8096) (local)
  [https://<your-subdomain>.duckdns.org](https://<your-subdomain>.duckdns.org) (remote via Nginx + DuckDNS)

---

## 🔒 Security Notes

* Make sure to configure HTTPS (e.g., with Let’s Encrypt and Certbot).
* Do **not** expose Jellyfin directly to the internet without Nginx.
* Use strong DuckDNS tokens and keep `.env` files private.

---

## 🛠️ Requirements

* Docker & Docker Compose installed.
* Media library available locally.
* A DuckDNS account with a registered subdomain.

---

## 📖 References

* [Jellyfin Documentation](https://jellyfin.org/docs/)
* [DuckDNS Setup](https://www.duckdns.org/install.jsp)
* [Nginx Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
