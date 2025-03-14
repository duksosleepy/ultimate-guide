# Part 3

*Date: 2025-03-12*

## KVM settings

## Setting up the network

### Network Device List
```bash
nmcli con show

nmcli device show

virsh net-list --all
```

### Bridge configuration for KVM

```bash
nmcli con add type bridge con-name br0 ifname br0 autoconnect yes

nmcli connection modify br0 ipv4.method auto

nmcli connection modify public-network ipv4.dns "8.8.8.8"

nmcli connection add type bridge-slave con-name enp2s0-bridge ifname enp2s0 master br0

cat > bridge.sh << EOF
nmcli con down enp2s0
nmcli con up br0
EOF
sh ./bridge.sh

virsh net-destroy default
virsh net-autostart default --disable

cat > bridge.xml << EOF
<network>
  <name>br0</name>
  <forward mode="bridge"/>
  <bridge name="br0" />
</network>
EOF
virsh net-define ./bridge.xml
virsh net-start br0
virsh net-autostart br0

firewall-cmd --permanent --zone=trusted --add-interface=br0

firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i br0 -o br0 -j ACCEPT

firewall-cmd --reload

```
