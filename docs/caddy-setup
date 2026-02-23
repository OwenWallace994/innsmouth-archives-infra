# Caddy Setup (Bare Metal)

## 3.1 The Purpose of Caddy

Caddy provides:

- Automatic HTTPS certificates (Let’s Encrypt)
- Subdomain routing
- Reverse proxying to Docker containers
- HTTP → HTTPS redirection
- Centralized access to all services

**Services proxied through Caddy:**

- Jellyfin
- Immich
- Grafana

## 3.2 Installation

Update repositories:

```bash
sudo apt update
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
```

Add Caddy repo + keys:

```bash
curl -1s https://dl.cloudsmith.io/public/caddy/stable/gpg.key | sudo apt-key add -
curl -1s https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt   | sudo tee /etc/apt/sources.list.d/caddy-stable.list
```

Install:

```bash
sudo apt update
sudo apt install caddy
```

Check service:

```bash
sudo systemctl status caddy
```

**Caddyfile location:**  
`/etc/caddy/Caddyfile`

## 3.3 Basic Configuration

Example Caddyfile:

```caddy
stream.innsmoutharchives.com {
    reverse_proxy localhost:8096
}

photos.innsmoutharchives.com {
    reverse_proxy localhost:2283
}

metrics.innsmoutharchives.com {
    reverse_proxy localhost:3000
}
```

**Port mapping explanation:**

- Jellyfin → 8096
- Immich → 2283
- Grafana → 3000

Caddy automatically manages HTTPS + certificate renewal.

## 3.4 DNS Requirements

Each subdomain must point to your server’s **public IP**.

Required records:

```
A stream.innsmoutharchives.com      <server_public_ip>
A photos.innsmoutharchives.com      <server_public_ip>
A metrics.innsmoutharchives.com     <server_public_ip>
```

## 3.5 Reloading and Validation

Reload Caddy:

```bash
sudo systemctl reload caddy
```

Validate config:

```bash
caddy validate --config /etc/caddy/Caddyfile
```

Live logs:

```bash
sudo journalctl -u caddy -f
```

## 3.6 Relevant File Locations

- **Primary config:** `/etc/caddy/Caddyfile`
- **Certificates:** `/var/lib/caddy/`
- **Systemd unit:** `/usr/lib/systemd/system/caddy.service`
- **Logs:** `sudo journalctl -u caddy -f`

# Docker Setup

## 4.1 Purpose

Docker provides:

- Service isolation
- Controlled deployments
- Persistent data via volumes
- Simplified updates

## 4.2 Directory Structure

```
docker/
    docker-compose.yml
    env.example

caddy/
    Caddyfile.example

docs/
media/
```

## 4.3 Environment Variables

Copy env template:

```bash
cp docker/env.example docker/.env
```

Populate `.env` with:

- PostgreSQL credentials
- Redis configuration
- Service secrets
- API keys

**Do NOT commit `.env` to Git.**

## 4.4 Docker Compose

Run:

```bash
cd docker
docker compose up -d
```

Verify service access:

- Jellyfin → https://stream.innsmoutharchives.com
- Immich → https://photos.innsmoutharchives.com
- Grafana → https://metrics.innsmoutharchives.com

# Monitoring Setup

## 5.1 Prometheus

Prometheus scrapes:

- Node exporter
- Docker metrics
- Application exporters

Data is stored using persistent volumes.

## 5.2 Grafana

Grafana dashboards display:

- CPU
- RAM
- Disk usage
- Container performance
- Service uptime

Access at:

`https://metrics.innsmoutharchives.com`

# Jellyfin Setup

Purpose: Local media streaming.

Runs in Docker with volumes for:

- Config
- Cache
- Media library

Access at:

`https://stream.innsmoutharchives.com`

# Immich Setup

Self-hosted photo system.

Components:

- Immich API
- Immich Web
- Machine Learning Service
- Postgres
- Redis

Persistent volumes for:

- Photos
- Database
- Configuration

Access at:

`https://photos.innsmoutharchives.com`

# Troubleshooting

## 8.1 OS Corruption Recovery (Snowstorm)

Recovery steps:

- Reinstall OS
- Reinstall Docker and Caddy
- Rebuild containers
- Recreate DNS routing
- Reconfigure metrics
- Restore volumes

## 8.2 Docker Debugging

```bash
docker compose logs <service>
docker ps -a
docker network ls
```

Common issues:

- Missing environment variables
- Incorrect ports
- Volume permission issues

## 8.3 Caddy Failures

Logs:

```bash
sudo journalctl -u caddy -f
```

Common issues:

- Incorrect DNS
- Wrong upstream ports
- Port conflicts (80/443 already in use)

## 8.4 Permissions

Fix ownership:

```bash
sudo chown -R $USER:$USER /path/to/volume
```
