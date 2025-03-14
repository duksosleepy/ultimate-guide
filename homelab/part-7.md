# Part 5

*Date: 2025-03-13*

## Create Kubernet Cluster

## Install Talosctl & kubectl
```bash
curl -sL https://talos.dev/install | sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Create cluster
```bash
export CONTROL_PLANE_IP=192.168.1.7

talosctl gen config talos-cluster \
https://$CONTROL_PLANE_IP:6443 \
--output-dir ./talos-cluster \
--nodes 192.168.1.7,192.168.1.8,192.168.1.9
```
