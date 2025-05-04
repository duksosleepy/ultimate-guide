# Part 5a

*Date: 2025-05-04*

## Setup Domain and Cloudflare Service (Caddy + Cloudflare DDNS + Wildcard SSL)

### Setup Caddy Proxy

```bash
mkdir -p ~/docker-services/caddy
cd ~/docker-services/caddy
nano docker-compose.yml

networks:
  proxy:
    name: proxy_network
  agent-network:
    name: agent_network
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
      - CLOUDFLARE_API_TOKEN=clf_token
      - TZ=Asia/Ho_Chi_Minh
    networks:
      - proxy

```

```bash
mkdir -p data config
nano config/Caddyfile
{
    dynamic_dns {
        provider cloudflare {env.CLOUDFLARE_API_TOKEN}
        domains {
            duksosleepy.dev portainer
            duksosleepy.dev k8s
        }
    }
    # Global options
    email songkhoi123@gmail.com
    acme_dns cloudflare {
        api_token env.CLOUDFLARE_API_TOKEN
    }
}

# Portainer configuration
portainer.duksosleepy.dev {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }

    reverse_proxy portainer:9443 {
        transport http {
            tls
            tls_insecure_skip_verify
        }
    }

    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        X-Content-Type-Options "nosniff"
        X-Frame-Options "SAMEORIGIN"
        Referrer-Policy "strict-origin-when-cross-origin"
        -Server
    }

    # Logging
    log {
        output file /data/logs/portainer-access.log {
            roll_size 10MB
            roll_keep 5
        }
        format json
    }
}

# Kubernetes main entry
k8s.duksosleepy.dev {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }

    reverse_proxy 192.168.1.27:80 {
        header_up Host {host}
    }

    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        X-Content-Type-Options "nosniff"
        Referrer-Policy "strict-origin-when-cross-origin"
        -Server
    }

    # Logging
    log {
        output file /data/logs/k8s-access.log {
            roll_size 10MB
            roll_keep 5
        }
        format json
    }
}
```

```bash
mkdir -p ~/docker-services/portainer
cd ~/docker-services/portainer
nano docker-compose.yml

networks:
  proxy:
    name: proxy_network
    external: true
  agent-network:
    name: agent_network
    external: true
services:
  agent:
    image: portainer/agent:latest
    container_name: portainer-agent
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent-network

  portainer:
    container_name: portainer
    image: portainer/portainer-ce:latest
    restart: unless-stopped
    command: -H tcp://agent:9001 --tlsskipverify
    volumes:
      - portainer_data:/data
    expose:
      - 9443
    networks:
      - proxy
      - agent-network
volumes:
  portainer_data:
```
