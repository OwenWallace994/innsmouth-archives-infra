# Innsmouth Archives – Self-Hosted Infrastructure

Innsmouth Archives is a self-hosted infrastructure stack running on a Linux server, built to provide:

- Media streaming (Jellyfin)
- Photo backup and management (Immich)
- Metrics and monitoring (Prometheus + Grafana)
- Reverse proxy and TLS termination (Caddy)
- Clean separation of services via Docker

All services are exposed through subdomains on a custom domain:

- `stream.innsmoutharchives.com` – Jellyfin
- `photos.innsmoutharchives.com` – Immich
- `metrics.innsmoutharchives.com` – Grafana (with Prometheus as the data source)

This repository documents the architecture, configuration, and operational practices for the stack.

---

## 1. High-Level Architecture

**Core components:**

- **Linux host**  
  Base OS running Docker and Caddy.

- **Docker**  
  Each major service (Jellyfin, Immich, Prometheus, Grafana, supporting databases) runs in its own container with defined volumes and networks.

- **Caddy**  
  Acts as the public entry point:
  - Terminates HTTPS using Let’s Encrypt
  - Routes subdomains to the appropriate Docker services
  - Handles basic security headers and logging

- **Metrics stack**  
  - **Prometheus**: scrapes metrics from the host and services
  - **Grafana**: visualizes metrics and dashboards at `metrics.innsmoutharchives.com`

A high-level diagram and more detail will be available in:

- `docs/overview.md`
- `media/diagrams/architecture.png` (planned)

---

## 2. Services

### 2.1 Jellyfin (Media Streaming)

- Purpose: Self-hosted media server for movies, TV, etc.
- Access: `https://stream.innsmoutharchives.com`
- Deployment:
  - Runs in Docker
  - Uses mounted volumes for:
    - Configuration
    - Media libraries
- Notes:
  - Behind Caddy reverse proxy
  - Accessible only via HTTPS

### 2.2 Immich (Photo Management)

- Purpose: Self-hosted photo backup and gallery
- Access: `https://photos.innsmoutharchives.com`
- Deployment:
  - Dockerized service with required dependencies (database, Redis, etc.)
  - Persistent volumes for:
    - Application data
    - Photo storage
- Notes:
  - Also proxied via Caddy
  - Intended as a long-term replacement for third-party cloud photo services

### 2.3 Prometheus + Grafana (Monitoring)

- Purpose: Infrastructure and service monitoring
- Access: `https://metrics.innsmoutharchives.com`
- Deployment:
  - Prometheus:
    - Scrapes metrics from exporters and services
    - Stores time-series data
  - Grafana:
    - Connects to Prometheus as a data source
    - Provides dashboards for:
      - Host CPU, memory, disk
      - Container status
      - Service availability (planned)
- Notes:
  - Internal configuration and dashboards are documented under `docs/monitoring-setup.md` (planned).

---

## 3. Domain & Routing

The stack uses a single domain with dedicated subdomains for each service:

- `stream.innsmoutharchives.com` → Jellyfin container
- `photos.innsmoutharchives.com` → Immich container
- `metrics.innsmoutharchives.com` → Grafana/Prometheus

Caddy is responsible for:

- Handling HTTPS certificates via Let’s Encrypt
- Reverse proxying requests to Docker services
- Managing virtual hosts per subdomain

An example (redacted) Caddy configuration is stored in:

- `caddy/Caddyfile.example`

---

## 4. Tech Stack

- **Operating System:** Linux (server environment)
- **Reverse Proxy:** Caddy
- **Containerization:** Docker / Docker Compose
- **Media:** Jellyfin
- **Photos:** Immich
- **Metrics:** Prometheus
- **Dashboards:** Grafana
- **Version Control & Documentation:** Git + Markdown

---

## 5. Repository Structure (Planned)

```text
innsmouth-archives-infra/
├─ README.md
├─ docs/
│  ├─ overview.md
│  ├─ caddy-setup.md
│  ├─ docker-setup.md
│  ├─ jellyfin-setup.md
│  ├─ immich-setup.md
│  ├─ monitoring-setup.md
│  ├─ troubleshooting.md
│  └─ roadmap.md
├─ docker/
│  ├─ docker-compose.yml
│  └─ env.example
├─ caddy/
│  ├─ Caddyfile.example
│  └─ notes.md
└─ media/
   ├─ screenshots/
   └─ diagrams/
