# Immich in Docker – Setup Guide

This document explains how to deploy **Immich**, a high‑performance self‑hosted photo/video backup solution, using **Docker Compose**.

---

## 1. Prerequisites

Ensure you have:
- Docker
- Docker Compose v2
- A machine with hardware acceleration recommended (but optional)

---

## 2. Directory Structure

```bash
mkdir -p ~/docker/immich
cd ~/docker/immich
```

---

## 3. docker-compose.yml

Create this file inside `~/docker/immich`:

```yaml
services:
  immich-server:
    image: ghcr.io/immich-app/immich-server:latest
    container_name: immich-server
    depends_on:
      - immich-redis
      - immich-database
    environment:
      - DB_HOST=immich-database
      - DB_PORT=5432
      - DB_USERNAME=postgres
      - DB_PASSWORD=postgres
      - DB_DATABASE_NAME=immich
      - REDIS_HOST=immich-redis
    volumes:
      - ./library:/usr/src/app/upload
    ports:
      - "2283:2283"
    restart: unless-stopped

  immich-microservices:
    image: ghcr.io/immich-app/immich-server:latest
    container_name: immich-microservices
    depends_on:
      - immich-redis
      - immich-database
    environment:
      - DB_HOST=immich-database
      - DB_PORT=5432
      - DB_USERNAME=postgres
      - DB_PASSWORD=postgres
      - DB_DATABASE_NAME=immich
      - REDIS_HOST=immich-redis
    volumes:
      - ./library:/usr/src/app/upload
    restart: unless-stopped

  immich-web:
    image: ghcr.io/immich-app/immich-web:latest
    container_name: immich-web
    depends_on:
      - immich-server
    environment:
      - IMMICH_SERVER_URL=http://immich-server:2283
    ports:
      - "3001:3001"
    restart: unless-stopped

  immich-machine-learning:
    image: ghcr.io/immich-app/immich-machine-learning:latest
    container_name: immich-machine-learning
    restart: unless-stopped

  immich-redis:
    image: redis:6
    container_name: immich-redis
    restart: unless-stopped

  immich-database:
    image: postgres:14
    container_name: immich-database
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=immich
    volumes:
      - ./db:/var/lib/postgresql/data
    restart: unless-stopped
```

---

## 4. Start Immich

Run:

```bash
docker compose up -d
```

Visit:

```
http://localhost:3001
```

---

## 5. Backups

Back up:
- `./library`
- `./db`

---

## 6. Update Immich

```bash
docker compose pull
docker compose up -d
```

## 7. Change Username and Password

- This can be done either within Immich or within 'docker-compose.yml'
