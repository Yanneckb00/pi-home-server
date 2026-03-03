## Yanneck Home Server

This repository contains the home server stack running on a Raspberry Pi / Ubuntu ARM64.  
It provides a reverse‑proxied, HTTPS‑secured set of self‑hosted services managed via Docker Compose.

- **Container runtime**: Docker + Docker Compose
- **Reverse proxy**: Traefik (HTTPS, routing, Let's Encrypt)
- **Dynamic DNS**: DuckDNS
- **Core services**: AdGuard Home, Portainer, Jellyfin, Navidrome, Feishin, Filebrowser, Docmost, Homarr, Home Assistant, Beets

## Project layout

- **Project root**: `home-server/`
- **Main stack definition**: `home-server/docker-compose.yaml`
- **Traefik configuration & certificates**: `home-server/traefik/`
- **Persistent data**: bound into the containers via volumes as configured in `docker-compose.yaml`
- **Extended documentation (English)**: `home-server/.docs/`

## Prerequisites (host)

Run these commands on the Raspberry Pi / Ubuntu host.

### System update and basic security

```bash
sudo apt update && sudo apt upgrade -y

# firewall (ufw)
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

### Install Docker & Docker Compose

```bash
# install docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker "$USER"    # log out and back in or reboot

# install docker compose (Debian/Ubuntu package)
sudo apt install docker-compose -y
```

### Optional: VPN (WireGuard)

```bash
sudo apt install wireguard -y
```

Follow the interactive setup instructions and configure port forwarding on your router  
for UDP port `51820` to your Raspberry Pi, if you want remote VPN access.

### Optional: Advanced security hardening

```bash
# fail2ban against SSH attacks
sudo apt install fail2ban -y

# automatic security updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## Environment and networking

The stack assumes:

- A Docker network called `proxy` (external) used by Traefik and all HTTP services.
- Public DNS records on DuckDNS pointing to your home IP (e.g. `*.your-domain.duckdns.org`).
- Port forwarding for **80/tcp** and **443/tcp** from your router to the Raspberry Pi.

Create or adapt your `.env` files under `home-server/` (never commit real secrets) to define:

- `DUCKDNS_DOMAIN`, `DUCKDNS_SUBDOMAINS`, `DUCKDNS_TOKEN`
- Service‑specific ports (e.g. `PORTAINER_PORT`, `NAVIDROME_PORT`, `JELLYFIN_PORT_TCP`, `HOMARR_PORT`, …)
- Timezone (`TZ`), UID/GID (`PUID`/`PGID`), and other app secrets (e.g. `DOCMOST_SECRET`, database URLs).

Traefik uses these values to route requests like:

- `https://traefik.${DUCKDNS_DOMAIN}` → Traefik dashboard
- `https://portainer.${DUCKDNS_DOMAIN}` → Portainer
- `https://adguard.${DUCKDNS_DOMAIN}` → AdGuard Home
- `https://navidrome.${DUCKDNS_DOMAIN}` → Navidrome
- `https://jellyfin.${DUCKDNS_DOMAIN}` → Jellyfin
- `https://feishin.${DUCKDNS_DOMAIN}` → Feishin
- `https://files.${DUCKDNS_DOMAIN}` → Filebrowser
- `https://docmost.${DUCKDNS_DOMAIN}` → Docmost
- `https://home.${DUCKDNS_DOMAIN}` → Homarr
- `https://smart.${DUCKDNS_DOMAIN}` → Home Assistant
- `https://beets.${DUCKDNS_DOMAIN}` → Beets

## Bringing the stack up

1. Clone this repository to the Raspberry Pi.
2. Ensure Docker and Docker Compose are installed (see above).
3. Create the external Docker network if it does not exist:

   ```bash
   docker network create proxy
   ```

4. Navigate into the project:

   ```bash
   cd home-server
   ```

5. Configure your environment variables (`.env` files as described in the docs).
6. Start all services:

   ```bash
   docker compose up -d
   ```

### Updating containers

From inside `home-server/`:

```bash
docker compose pull          # pull newer images
docker compose up -d         # recreate containers with new images
```

## Included services (from docker-compose)

Grouped as in `docker-compose.yaml`:

- **Administration**
  - **Traefik** (`traefik`): reverse proxy, HTTP→HTTPS redirect, Let's Encrypt.
  - **Portainer** (`portainer`): Docker management UI.
  - **DuckDNS** (`duckdns`): dynamic DNS updater.
  - **AdGuard Home** (`adguard`): DNS‑based ad and tracking blocker.

- **Media**
  - **Navidrome** (`navidrome`): self‑hosted music streaming backend.
  - **Jellyfin** (`jellyfin`): media server for video, music and photos.
  - **Feishin** (`feishin`): web UI/music player for Subsonic/Navidrome.

- **Productivity**
  - **Filebrowser** (`filebrowser`): web file manager.
  - **Docmost** (`docmost`, `docmost-db`, `docmost-redis`): documentation and knowledge base (Postgres + Redis backend).

- **Management / Smart Home**
  - **Homarr** (`homarr`): dashboard / homepage for your services.
  - **Home Assistant** (`homeassistant`): home automation hub.
  - **Beets** (`beets`): music library management.

For per‑service details (mount paths, ports, environment variables) see `home-server/docker-compose.yaml`.

## Helpful host tools

### TMUX (terminal multiplexer)

```bash
sudo apt install tmux -y
```

Basic usage:

- **New session**: `tmux new -s <name>`
- **Detach from session**: `Ctrl+B`, then `D`
- **Re‑attach**: `tmux attach -t <name>`

### Cockpit (web administration)

```bash
sudo apt update -y
sudo apt install cockpit -y
sudo ufw allow 9090/tcp
sudo systemctl enable --now cockpit.socket

# alternatively
sudo systemctl start cockpit
sudo systemctl enable cockpit
```

Then open `https://<raspberry-ip>:9090` in your browser.

## Useful host commands

- **Reboot**: `sudo reboot`
- **Check Docker containers**: `docker ps`
- **View logs for a service**: `docker compose logs -f <service-name>`

## Documentation and credits

- Inspiration for service selection: `https://techhut.tv/must-have-home-server-services-2025/`
- TechHut homelab repo: `https://github.com/TechHutTV/homelab`