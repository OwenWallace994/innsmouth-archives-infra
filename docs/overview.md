# Innsmouth Archives – Overview

Innsmouth Archives is a self-hosted infrastructure stack running on a Linux server. It is designed to provide:

- Media streaming (Jellyfin)
- Photo backup and browsing (Immich)
- Metrics and monitoring (Prometheus + Grafana)
- Reverse proxying and TLS termination (Caddy)

The goals of this setup are:

- Keep services isolated and reproducible using Docker
- Use a single public domain with subdomains for clean routing
- Maintain observability into system health and performance
- Make it possible to rebuild or migrate the stack if needed

## High-Level Components

- **Linux host** – physical or virtual machine running Docker and Caddy.
- **Docker** – containers for Jellyfin, Immich, Prometheus, Grafana, and any supporting services.
- **Caddy** – entrypoint for all HTTPS traffic, routing based on subdomain.
- **Prometheus** – collects metrics from exporters and services.
- **Grafana** – visualizes Prometheus data through dashboards.

## Public Endpoints

- `stream.innsmoutharchives.com` – Jellyfin
- `photos.innsmoutharchives.com` – Immich
- `metrics.innsmoutharchives.com` – Grafana

This document is a high-level overview. Service-specific configuration and setup details are documented in:

- `docs/caddy-setup.md`
- `docs/docker-setup.md`
- `docs/jellyfin-setup.md`
- `docs/immich-setup.md`
- `docs/monitoring-setup.md`
- `docs/troubleshooting.md`
