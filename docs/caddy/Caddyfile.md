# Reverse Proxy Configuration Using Caddy

This document provides a clean example of how to configure **Caddy** as a reverse proxy for multiple self-hosted services.  
The example demonstrates routing for:

- A media server  
- A root-domain redirect  
- A photo service  
- A metrics dashboard with hardened security headers  

All domains and ports in this example are placeholders and can be adjusted to fit any deployment.

---

## Example Caddyfile

```caddyfile
# Media Server (e.g., Jellyfin)
media.example.com {
    reverse_proxy 127.0.0.1:8096
    encode gzip
}

# Redirect root domain to the media server
example.com {
    redir https://media.example.com{uri}
}

# Photo Backup or Gallery Service (e.g., Immich)
photos.example.com {
    reverse_proxy 127.0.0.1:2283
}

# Metrics / Monitoring Dashboard (e.g., Grafana)
metrics.example.com {
    encode gzip
    reverse_proxy 127.0.0.1:3000

    # Security-hardening HTTP headers
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "DENY"
        X-XSS-Protection "1; mode=block"
    }
}
```

---

## Notes

- `reverse_proxy` forwards each subdomain to a service running on a local port.  
- `encode gzip` enables response compression for improved performance.  
- The metrics dashboard includes additional HTTP security headers suitable for public exposure.  
- This structure keeps services isolated behind subdomains while maintaining a single, clean entry point through Caddy.

---
