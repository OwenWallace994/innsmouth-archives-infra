# About This Project

**innsmoutharchives.com** is a long-term passion project where I combine my creative interests — maritime folklore, diving stories, and horror writing — with my technical growth as a self-hosting enthusiast.

The website and overall branding lean heavily into atmospheric, nautical-horror themes by design. However, this repository is strictly focused on the **technical infrastructure** that powers the project, including:

- Dockerized service deployment  
- Reverse proxy configuration using Caddy  
- Monitoring with Prometheus and Grafana  
- Secure subdomain routing  
- Documentation of a reproducible, well-structured homelab environment  

While the creative content behind Innsmouth Archives involves storytelling, interviews with divers/sailors, and oceanic horror themes, the infrastructure showcased here is fully professional and demonstrates my ability to design and maintain real Linux-based systems.

This repo is part of my portfolio and reflects my practical skills in DevOps tooling, containerization, and self-hosted service architecture — independent of the site's creative tone.

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

Ubuntu Server
   └─ Docker
      ├─ Immich
      ├─ Jellyfin
      ├─ Grafana
      └─ Prometheus
   └─ Caddy
      └─ Routes to subdomains

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
  - Internal configuration and dashboards are documented under `docs/monitoring-setup.md`.

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

[Click here for an example Caddyfile.](docs/caddy/Caddyfile.md)

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

## 5. Repository Structure 
```text
innsmouth-archives-infra/
├─ README.md
├─ docs/
│  ├─ overview.md
│  ├─ caddy-setup.md
│  ├─ docker-setup.md
│  ├─ jellyfin-setup.md
│  ├─ immich-setup.md
│  └─ monitoring-setup.md
├─ Docker/
│  └─ docker-compose.md
├─ caddy/
│  └─Caddyfile.md
└─ media/
   ├─ screenshots/
   └─ diagrams/
