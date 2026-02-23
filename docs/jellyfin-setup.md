# Jellyfin in Docker – Setup Guide

This document describes how to run **Jellyfin** using **Docker** and **Docker Compose** for a clean, reproducible media server setup.

---

## 1. Overview

[Jellyfin](https://jellyfin.org/) is a free, open-source media server. Running it in Docker gives you:

- Easy installation and upgrades
- Isolation from the host system
- Simple backup/restore (just back up your config & media)

This guide uses **Docker Compose** and the popular `linuxserver/jellyfin` image.

---

## 2. Prerequisites

Before you start, you should have:

- A machine running **Linux**, **Windows**, or **macOS**
- **Docker** installed  
- **Docker Compose** installed (v2 recommended, often bundled with recent Docker Desktop / Docker Engine)
- Basic command-line familiarity

### 2.1. Verify Docker installation

```bash
docker --version
docker compose version  # or: docker-compose --version
```

If these commands return version numbers and no errors, you’re good to go.

---

## 3. Directory Layout

Choose a directory on your host where Jellyfin’s configuration and media will live. For example:

```bash
mkdir -p ~/docker/jellyfin/config
mkdir -p ~/docker/jellyfin/cache
mkdir -p ~/docker/jellyfin/media/movies
mkdir -p ~/docker/jellyfin/media/tv
mkdir -p ~/docker/jellyfin/media/music
```

- `config` – Jellyfin configuration, metadata, plugins, etc.
- `cache` – Transcoding cache and temporary data
- `media` – Your actual media library (organize however you like)

> **Tip:** Put these on a disk with enough space for your library. Backing up `config` is especially important.

---

## 4. Docker Compose Configuration

Create a file called `docker-compose.yml` in `~/docker/jellyfin` (or your chosen folder).

```yaml
version: "3.9"

services:
  jellyfin:
    image: linuxserver/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped

    # Map host ports to container ports
    ports:
      - "8096:8096"   # HTTP
      # - "8920:8920" # HTTPS (optional, usually handled by a reverse proxy)

    # Environment configuration (user, group, timezone)
    environment:
      - PUID=1000        # Set this to your user ID
      - PGID=1000        # Set this to your group ID
      - TZ=Etc/UTC       # Use your timezone, e.g., America/Chicago

    # Mount volumes into the container
    volumes:
      - ./config:/config
      - ./cache:/cache
      - ./media/movies:/media/movies
      - ./media/tv:/media/tv
      - ./media/music:/media/music
```

### 4.1. Finding your PUID and PGID (Linux)

On Linux, run:

```bash
id
```

Example output:

```text
uid=1000(myuser) gid=1000(myuser) groups=1000(myuser),27(sudo)
```

In this case, set:

- `PUID=1000`
- `PGID=1000`

On Windows/macOS with Docker Desktop, you can usually leave `PUID` and `PGID` as `1000`, or remove them entirely if you don’t need fine-grained permission control.

### 4.2. Adjusting Timezone

Replace `Etc/UTC` with your own timezone string, e.g.:

- `America/Chicago`
- `Europe/Berlin`
- `Asia/Tokyo`

You can find valid timezone strings in the [IANA time zone database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

---

## 5. Starting Jellyfin

From the directory containing your `docker-compose.yml`, run:

```bash
docker compose up -d
```

or, if your system uses the older `docker-compose` binary:

```bash
docker-compose up -d
```

- `-d` means “detached mode”, so the containers run in the background.
- Docker will pull the `linuxserver/jellyfin` image (if you don’t already have it) and start the container.

Check that it’s running:

```bash
docker ps
```

You should see a container named `jellyfin` in the list.

---

## 6. Accessing the Web UI

Once the container is running, open a browser and navigate to:

```text
http://<HOST_IP>:8096
```

Examples:

- On the same machine: `http://localhost:8096`
- On another device in your network: `http://192.168.1.50:8096` (replace with your server’s IP)

You’ll be greeted by the Jellyfin setup wizard, where you will:

1. Create an admin user
2. Set your preferred language
3. Point Jellyfin to your media folders (e.g., `/media/movies`, `/media/tv`, `/media/music`)
4. Configure metadata providers and additional settings

After finishing the wizard, Jellyfin will start scanning your media library.

---

## 7. Managing the Container

### 7.1. Stopping Jellyfin

From the same directory as your `docker-compose.yml`:

```bash
docker compose down
```

This stops and removes the container but **keeps your data** in `config`, `cache`, and `media`.

### 7.2. Viewing Logs

To see Jellyfin logs:

```bash
docker compose logs -f
```

The `-f` flag “follows” the logs in real time (press `Ctrl + C` to exit).

---

## 8. Updating Jellyfin

To update to the latest image:

```bash
docker compose pull
docker compose up -d
```

This will:

1. Pull the newest `linuxserver/jellyfin` image
2. Recreate the container using your existing configuration and media

> **Note:** Always back up your `config` directory before major upgrades.

---

## 9. Optional: Docker Network & Reverse Proxy

For production setups, you may want to put Jellyfin behind a reverse proxy (e.g., **Caddy**, **Nginx**, or **Traefik**) and enable HTTPS.

Example of putting Jellyfin on a custom Docker network:

```yaml
networks:
  proxy:
    external: true

services:
  jellyfin:
    image: linuxserver/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    networks:
      - proxy
    ports:
      - "8096:8096"
    volumes:
      - ./config:/config
      - ./cache:/cache
      - ./media/movies:/media/movies
      - ./media/tv:/media/tv
      - ./media/music:/media/music
```

Then, in your reverse proxy container, join the same `proxy` network and proxy requests to `jellyfin:8096`.

> This keeps your application networking encapsulated and lets you expose only your reverse proxy to the public internet.

---

## 10. Backup & Restore

To back up your Jellyfin setup, make a copy of:

- `./config`
- (Optionally) `./cache` (can be rebuilt, but backing it up may speed things up)
- Your `./media` directories (or wherever your media is stored)

Restoring is as simple as:

1. Copying those directories to a new host
2. Recreating the same `docker-compose.yml`
3. Running `docker compose up -d`

Jellyfin will start with your existing users, settings, and metadata.

---

## 11. Troubleshooting

**Container won’t start**  
- Run `docker compose logs jellyfin` and look for obvious errors (permissions, missing volumes, port conflicts).  
- Ensure the `config` and `media` directories are readable by the container user (matching `PUID`/`PGID` if used).

**Can’t access from other devices**  
- Confirm your firewall allows traffic on port `8096`.  
- Verify your server’s IP and try `http://SERVER_IP:8096` from a browser on the same network.

**Playback or transcoding issues**  
- Check the **Playback** and **Transcoding** settings in the Jellyfin admin dashboard.  
- If you have a capable GPU and want hardware acceleration, enable and configure it there (and ensure the relevant device is passed into the container).

---

## 12. Summary

You now have a reproducible, Docker-based Jellyfin setup with:

- Persistent configuration and media volumes
- Simple commands to start, stop, and update
- A structure that can be extended with reverse proxies, hardware acceleration, and more

Commit this file (`jellyfin-setup.md`) to your repository along with your `docker-compose.yml` to document your media server setup.
