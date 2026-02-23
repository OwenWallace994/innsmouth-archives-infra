# Monitoring Stack in Docker – Grafana + Prometheus

This guide provides a full Docker Compose setup for running **Grafana**, **Prometheus**, and optional **Node Exporter** for system metrics.

---

## 1. Directory Setup

```bash
mkdir -p ~/docker/monitoring
cd ~/docker/monitoring
mkdir prometheus
```

---

## 2. Prometheus Configuration

Create the file:

`prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['nodeexporter:9100']
```

---

## 3. docker-compose.yml

Create this file inside `~/docker/monitoring`:

```yaml
version: "3.9"

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana
    restart: unless-stopped

  nodeexporter:
    image: prom/node-exporter:latest
    container_name: nodeexporter
    ports:
      - "9100:9100"
    restart: unless-stopped

volumes:
  grafana-storage:
```

---

## 4. Start the Stack

```bash
docker compose up -d
```

---

## 5. Access Interfaces

- **Grafana:** http://localhost:3000  
  Default login: `admin` / `admin`

- **Prometheus:** http://localhost:9090  
- **Node Exporter Metrics:** http://localhost:9100/metrics

---

## 6. Add Prometheus to Grafana

Grafana → *Connections* → *Add new data source* → **Prometheus**

Enter:

```
http://prometheus:9090
```

Save & test.

---

## 7. Update Containers

```bash
docker compose pull
docker compose up -d
```

---

## 8. Backup

Back up:

- `./prometheus/prometheus.yml`
- `grafana-storage` Docker volume (automatically managed)
