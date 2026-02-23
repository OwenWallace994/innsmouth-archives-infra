# Prometheus + Node Exporter (Example Docker Compose Configuration)

This document provides a clean, professional explanation of a minimal Prometheus + Node Exporter monitoring stack suitable for a portfolio or GitHub repository.

---

## Example `docker-compose.yml`

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.path=/prometheus"
      - "--web.listen-address=0.0.0.0:9090"
    networks:
      - monitoring
    ports:
      - "9090:9090"   # expose Prometheus UI

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    pid: "host"
    networks:
      - monitoring
    command:
      - "--path.rootfs=/host"

volumes:
  prometheus-data:

networks:
  monitoring:
```

---

## Explanation

This Docker Compose example defines a minimal monitoring stack using **Prometheus** and **Node Exporter**.

### Prometheus
Prometheus is configured with:

- A bind-mounted configuration file (`prometheus.yml`)
- A persistent data volume (`prometheus-data`) for the TSDB
- Explicit command arguments defining:
  - The configuration file location  
  - The TSDB storage path  
  - The listen address (`0.0.0.0:9090`)

Port **9090** is exposed to allow access to the Prometheus UI from a web browser.

### Node Exporter
Node Exporter is responsible for collecting host-level system metrics. It uses:

- `pid: "host"` to access the host's process namespace  
- `--path.rootfs=/host` to provide full host metrics rather than container-only metrics  
- The shared `monitoring` network for communication with Prometheus  

### Network & Volumes
All services run within an isolated `monitoring` network, and persistent volumes are explicitly declared to ensure portability and reproducibility.
