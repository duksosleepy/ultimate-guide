# Part 5

*Date: 2025-03-13*

## Create Oracle Node

## Ubuntu observability stack

### Update & Upgrade

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl wget git apt-transport-https ca-certificates gnupg lsb-release
```

### Install Docker & Docker Compose

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker $USER

sudo apt update
sudo apt install -y docker-compose-plugin
```
### Uptime-kuma

```bash
mkdir -p ~/docker-services/uptime-kuma
cd ~/docker-services/uptime-kuma

cat > docker-compose.yml << 'EOL'
version: '3.8'
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    volumes:
      - ./data:/app/data
    ports:
      - "3001:3001"
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
EOL

docker compose up -d

docker ps | grep uptime-kuma

sudo ufw allow 3001/tcp
```
### NetData

```bash
mkdir -p ~/docker-services/netdata
cd ~/docker-services/netdata

cat > docker-compose.yml << 'EOL'
version: '3.8'
services:
  netdata:
    image: netdata/netdata:latest
    container_name: netdata
    hostname: ubuntu-server
    ports:
      - "19999:19999"
    restart: unless-stopped
    cap_add:
      - SYS_PTRACE
    security_opt:
      - apparmor:unconfined
    volumes:
      - ./netdataconfig:/etc/netdata
      - ./netdatalib:/var/lib/netdata
      - ./netdatacache:/var/cache/netdata
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - PGID=999
      - DOCKER_USR=netdata
EOL

docker compose up -d

docker ps | grep netdata

sudo ufw allow 19999/tcp
```
