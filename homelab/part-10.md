# Part 10

*Date: 2025-05-04*

## Setup Database Server

### Configure Network

```bash
sudo nano /etc/netplan/....yaml

network:
  ethernets:
    ens3:
      addresses:
        - 192.168.1.7/24
      dhcp4: false
      dhcp6: false
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 192.168.1.1
          - 1.1.1.1
        search: []
      optional: true
  version: 2

sudo netplan apply
```
