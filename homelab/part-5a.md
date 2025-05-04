# Part 5a

*Date: 2025-05-04*

## Setup Domain and Cloudflare Service (Caddy + Cloudflare DDNS + Wildcard SSL)

### Setup Caddy Proxy

```bash
mkdir -p ~/docker-services/caddy
cd ~/docker-services/caddy
nano docker-compose.yml

version: '3.8'
services:
  caddy:
    container_name: caddy
    image: serfriz/caddy-cloudflare-ddns:latest
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./config/Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy_data:/data
      - ./caddy_config:/config
    environment:
      - CLOUDFLARE_API_TOKEN=3XTm7P7ppfiiLwYlMfJXNYWjeHscmB84IDMYQzrJ
      - TZ=Asia/Ho_Chi_Minh
```

```bash
mkdir -p data config
```
