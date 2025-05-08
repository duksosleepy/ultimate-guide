# Part 8

*Date: 2025-05-04*

## Setup RHEL host

### Install Docker & Docker Compose

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker $USER
newgrp docker

sudo systemctl enable --now docker
```

### Setup Domain and Cloudflare Service (Caddy + Cloudflare DDNS + Wildcard SSL)

```bash
docker network create proxy_network
docker network create agent_network

mkdir -p ~/docker-services/caddy
cd ~/docker-services/caddy
nano docker-compose.yml

networks:
  proxy:
    name: proxy_network
    external: true
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
      - ./config/Caddyfile:/etc/caddy/Caddyfile
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
                        duksosleepy.dev vault
                }
        }
        email songkhoi123@gmail.com
        acme_dns cloudflare {
                api_token env.CLOUDFLARE_API_TOKEN
        }
}

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
docker compose down
docker compose up --build -d
docker compose exec caddy caddy fmt --overwrite /etc/caddy/Caddyfile
```

### Setup Portainer

```bash
mkdir -p ~/docker-services/portainer
cd ~/docker-services/portainer
nano docker-compose.yml

networks:
  proxy:
    name: proxy_network
    external: true
services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:latest
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    ports:
      - "8000:8000"
    expose:
      - 9443
    networks:
      - proxy
volumes:
  portainer_data:
```

Portainer available in: https://portainer.duksosleepy.dev
