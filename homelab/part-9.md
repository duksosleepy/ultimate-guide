# Part 9

*Date: 2025-05-04*

## Setup Auth Server (Open Suse Leap Micro)

### Install Portainer Agent

```bash
sudo transactional-update
sudo transactional-update pkg install docker docker-compose docker-compose-switch && reboot
systemctl disable --now transactional-update.timer && reboot
systemctl enable --now docker.service
sudo usermod -aG docker $USER
newgrp docker

docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts
```
