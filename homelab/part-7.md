# Part 5

*Date: 2025-03-13*

## Create Kubernet Cluster

## Install Talosctl & kubectl
```bash
curl -sL https://talos.dev/install | sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Create cluster & apply control plane
```bash
export CONTROL_PLANE_IP=192.168.1.27

talosctl gen config talos-cluster \
https://$CONTROL_PLANE_IP:6443 \
--output-dir ./talos-cluster \
--nodes 192.168.1.27,192.168.1.25,192.168.1.26

talosctl get disks --insecure --nodes $CONTROL_PLANE_IP

talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file talos-cluster/controlplane.yaml

export TALOSCONFIG="talos-cluster/talosconfig"

talosctl config endpoint $CONTROL_PLANE_IP

talosctl config node $CONTROL_PLANE_IP

talosctl bootstrap
```
```yaml
  network:
      hostname: talos-control
      interfaces:
        - interface: eth0
          addresses:
            - 192.168.1.27/24
          routes:
            - network: 0.0.0.0/0
              gateway: 192.168.1.1
          dhcp: true

      nameservers:
        - 192.168.1.1
        - 8.8.4.4
        - 8.8.8.8
        - 1.1.1.1

      extraHostEntries:
          - ip: 192.168.1.27
            aliases:
              - talos-control
          - ip: 192.168.1.25
            aliases:
              - talos-worker1
          - ip: 192.168.1.26
            aliases:
              - talos-worker2
  time:
      disabled: false
      servers:
          - time.cloudflare.com
```

### Apply worker nodes

```bash
talosctl apply-config --insecure --nodes 192.168.1.25 --file talos-cluster/worker.yaml
talosctl apply-config --insecure --nodes 192.168.1.26 --file talos-cluster/worker.yaml

talosctl kubeconfig .

kubectl get nodes --kubeconfig=./kubeconfig
```
```yaml
  network:
      hostname: talos-worker1
      interfaces:
        - interface: eth0
          addresses:
            - 192.168.1.25/24
          routes:
            - network: 0.0.0.0/0
              gateway: 192.168.1.1
          dhcp: true

      nameservers:
        - 192.168.1.1
        - 8.8.4.4
        - 8.8.8.8
        - 1.1.1.1

      extraHostEntries:
          - ip: 192.168.1.27
            aliases:
              - talos-control
          - ip: 192.168.1.25
            aliases:
              - talos-worker1
          - ip: 192.168.1.26
            aliases:
              - talos-worker2
  time:
      disabled: false
      servers:
          - time.cloudflare.com
```

```bash
talosctl  shutdown -n ...

talosctl dashboard
talosctl mount
talosctl processes
talosctl dmesg  | head -n 20
```
### How to re apply edited-config

```bash
export TALOSCONFIG="talos-cluster/talosconfig"

talosctl edit machineconfig --nodes 192.168.1.25 --mode=reboot
```
