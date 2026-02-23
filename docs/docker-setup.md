# Docker Setup

This document describes how Docker is used to deploy, isolate, and manage the services that make up the Innsmouth Archives infrastructure. All major application components—Jellyfin, Immich, Prometheus, and Grafana—run inside Docker containers on a Linux host. Docker Compose is used to orchestrate the entire stack, define networks, manage persistent storage, and simplify upgrades.

---

## 1. Purpose of Docker in This Environment

Docker provides:

- Isolated service environments
- Predictable and reproducible deployments
- Clean separation of application code, configuration, and data
- Simplified service upgrades and rollback strategies
- Network segmentation between services
- Persistent storage using volumes

Docker enables each service to run independently, communicate only as needed, and avoid dependency conflicts.

---

## 2. Directory Structure

The recommended directory layout for Docker-related files:

```
docker/
  docker-compose.yml
  env.example
  .env    (not committed to Git)
caddy/
  Caddyfile.example
docs/
media/
  screenshots/
  diagrams/
```

- `docker-compose.yml` contains the service definitions.
- `env.example` is a sanitized template for environment variables.
- `.env` stores real deployment secrets and must **not** be committed.

---

## 3. Environment Variables

Environment variables are used to configure sensitive information such as:

- Database usernames and passwords
- Redis/Postgres configuration
- API keys or service tokens
- Internal service URLs

Steps:

1. Copy the template:
   ```
   cp docker/env.example docker/.env
   ```
2. Fill in all required values.
3. Ensure `.env` is *not committed* to Git.

---

## 4. Docker Compose

Docker Compose orchestrates all major services in the stack. A typical `docker-compose.yml` defines:

- Jellyfin  
- Immich (with Postgres, Redis, and ML services)
- Prometheus  
- Grafana  
- Networks  
- Volumes

Bringing the stack online:

```
cd docker
docker compose up -d
```

This starts all services in the background.

---

## 5. Verifying Service Access

After the containers are running, confirm that each public-facing service is accessible through its Caddy reverse-proxied subdomain:

- Jellyfin: https://stream.innsmoutharchives.com
- Immich: https://photos.innsmoutharchives.com
- Grafana: https://metrics.innsmoutharchives.com

If a service fails to load, check logs:

```
docker compose logs <service_name>
```

---

## 6. Troubleshooting Docker

### Check container status:
```
docker ps -a
```

### View logs:
```
docker compose logs <service>
```

### Restart a single service:
```
docker compose restart <service>
```

### Rebuild a service (if updating images):
```
docker compose pull
docker compose up -d
```

### Common issues:

- Incorrect port mapping  
- Missing environment variables  
- Incorrect volume permissions  
- Containers failing due to missing database credentials  

---

## 7. Data Persistence

Docker volumes ensure that data remains intact across updates and rebuilds.

You should have volumes for:

- Jellyfin config/cache  
- Immich photos  
- Immich Postgres  
- Immich Redis  
- Prometheus TSDB  
- Grafana config and dashboards  

Ensure these volumes are backed up or included in a snapshot strategy.

---

## 8. Notes and Recommendations

- Keep `.env` out of Git  
- Regularly update images using `docker compose pull`  
- Review release notes for Immich updates (frequent changes)  
- Ensure host firewall rules allow only necessary inbound traffic  
- Consider using watchtower or a similar tool for automated updates  

