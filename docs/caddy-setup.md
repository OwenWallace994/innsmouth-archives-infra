# Caddy Setup

This document describes how Caddy is installed, configured, and used as the reverse proxy and TLS entrypoint for the Innsmouth Archives infrastructure. Caddy is running directly on the host (bare metal), not inside Docker. It manages HTTPS certificates, handles virtual hosts for each subdomain, and forwards traffic to the appropriate Docker services.

---

## 1. Purpose of Caddy in This Environment

Caddy provides:

- Automatic HTTPS using Let’s Encrypt
- Reverse proxy routing based on subdomains
- A single public-facing entrypoint for all services
- Simple, declarative configuration via a Caddyfile
- Service isolation (each backend remains inside Docker)

The services exposed through Caddy include:

- Jellyfin (`stream.innsmoutharchives.com`)
- Immich (`photos.innsmoutharchives.com`)
- Grafana / Prometheus (`metrics.innsmoutharchives.com`)

---

## 2. Installation (Bare Metal)

Caddy was installed directly on the host using the official repository packages.

Typical installation steps:

```bash
sudo apt update
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https

curl -1s https://dl.cloudsmith.io/public/caddy/stable/gpg.key | sudo apt-key add -
curl -1s https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt | sudo tee /etc/apt/sources.list.d/caddy-stable.list

sudo apt update
sudo apt install caddy
