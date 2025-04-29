# Part 5

*Date: 2025-03-13*

## Create Oracle Node

## Ubuntu Observability Stack

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

docker rm -f portainer
docker volume rm portainer_data

docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest

```
### Install Pihole and Wireguard Server
```bash
curl -sSL https://install.pi-hole.net | bash

sudo apt install wireguard wireguard-tools

mkdir /etc/wireguard
cd /etc/wireguard

umask 077
wg genkey | tee server_private_key | wg pubkey > server_public_key
wg genkey | tee client_private_key | wg pubkey > client_public_key
wg genpsk > client.psk

nano wg0.conf

[Interface]
Address = 10.100.0.1/24, fd08:4711::1/64
SaveConfig = true
ListenPort = 51910
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o ens18 -j MASQUERADE


echo "PrivateKey = $(cat server_private_key)" >> wg0.conf

echo "[Peer]" >> wg0.conf
echo "PublicKey = $(cat client_public_key)" >> wg0.conf
echo "PresharedKey = $(cat client.psk)" >> wg0.conf
echo "AllowedIPs = 10.100.0.2/32, fd08:4711::2/128" >> wg0.conf



nano /etc/sysctl.conf
net.ipv4.ip_forward=1

systemctl enable wg-quick@wg0
chown -R root:root /etc/wireguard/
chmod -R og-rwx /etc/wireguard/*

sudo ufw allow 51910/udp comment 'WireGuard VPN'
sudo ufw allow 51910/tcp comment 'WireGuard VPN'
```

### Extra: for client

```bash
echo "[Interface]" > client.conf
echo "Address = 10.100.0.2/32, fd08:4711::2/128" >> client.conf
echo "DNS = 10.100.0.1" >> client.conf
echo "PrivateKey = $(cat client_private_key)" >> client.conf

nano client_name.conf

[Peer]
AllowedIPs = 10.100.0.0/24, fd08::/64
Endpoint = Your static IP:51910
PersistentKeepalive = 25


echo "PublicKey = $(cat server_public_key)" >> client.conf
echo "PresharedKey = $(cat client.psk)" >> client.conf


qrencode -t ansiutf8 < ~/wg-clients/client.conf
scp holu@203.0.113.1:wg-clients/client.conf .

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
