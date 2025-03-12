# Part 2

*Date: 2025-03-12*

## KVM settings

## Setting up the basics

### Backup Data

```bash
du -sh /home/*

mkdir -p /backup

tar -czf /backup/home_backup.tar.gz -C /home .
```

### Shrink /home partition

```bash
umount /home
xfs_repair -n /dev/rhel/home

lvremove /dev/rhel/home

lvcreate -L 200G -n home rhel

mkfs.xfs /dev/rhel/home

mount /dev/rhel/home /home

tar -xzf /backup/home_backup.tar.gz -C /home

chown -R root:root /home
```
### Create Logical Volume for KVM

```bash
lvcreate -l 90%FREE -n kvmstorage rhel

lvdisplay /dev/rhel/kvmstorage

mkfs.xfs -d su=64k,sw=4 -m reflink=1 /dev/rhel/kvmstorage

mkdir -p /var/lib/libvirt/images

mount -o noatime,nodiratime,discard /dev/rhel/kvmstorage /var/lib/libvirt/images

echo "/dev/rhel/kvmstorage  /var/lib/libvirt/images  xfs  noatime,nodiratime,discard  0 0" >> /etc/fstab
```

### KVM Installation and Configuration

```bash
sudo dnf install -y @virtualization
sudo dnf install -y virt-install virt-manager libvirt libvirt-devel qemu-kvm qemu-img libvirt-client libvirt-python

sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd

virt-host-validate
```

### Installing the web console to manage virtual machines

```bash
dnf install cockpit

systemctl enable --now cockpit.socket

firewall-cmd --add-service=cockpit --permanent

firewall-cmd --reload

dnf install cockpit-machines
```


### Configure the second NVMe drive (/dev/nvme1n1) using XFS, LVM-thin, and LVM-cache to store virtual machine images

```bash
1. Create a Physical Volume on the second NVMe drive

pvcreate /dev/nvme1n1

2. Create a new Volume Group

vgcreate vg_images /dev/nvme1n1

3. Create a Logical Volume for thin pool metadata (approximately 0.5% of total capacity)

lvcreate -L 8G -n thin_meta vg_images

4. Create a Logical Volume for thin pool data (approximately 80% of total capacity)

lvcreate -L 1.5T -n thin_data vg_images

5. Create a Logical Volume for cache data (approximately 15% of total capacity)

lvcreate -L 250G -n cache_data vg_images

6. Create a Logical Volume for cache metadata (approximately 1% of cache data size)

lvcreate -L 2G -n cache_meta vg_images

7. Convert to a thin pool

lvconvert --type thin-pool --poolmetadata vg_images/thin_meta vg_images/thin_data

8. Convert to a cache pool

lvconvert --type cache-pool --poolmetadata vg_images/cache_meta vg_images/cache_data

9. Attach cache pool to thin pool

lvconvert --cache --cachepool vg_images/cache_data vg_images/thin_data

10. Create a thin logical volume from the thin pool

lvcreate -V 3T -T vg_images/thin_data -n vm_images

11. Format with XFS filesystem

mkfs.xfs /dev/vg_images/vm_images

12. Create a mount point directory and mount the volume

mkdir -p /var/lib/libvirt/images-thin
mount -o noatime,nodiratime /dev/vg_images/vm_images /var/lib/libvirt/images-thin

13. Add to /etc/fstab for automatic mounting at boot

echo "/dev/vg_images/vm_images  /var/lib/libvirt/images-thin  xfs  noatime,nodiratime  0 0" >> /etc/fstab

14. Create an additional storage pool in libvirt

virsh pool-define-as --name thin_pool --type dir --target /var/lib/libvirt/images-thin
virsh pool-build thin_pool
virsh pool-start thin_pool
virsh pool-autostart thin_pool
```
### Configure storage pools in libvirt

```bash
virsh pool-list --all

virsh pool-define-as --name default --type dir --target /var/lib/libvirt/images
virsh pool-build default
virsh pool-start default
virsh pool-autostart default

virsh pool-define-as --name thin_pool --type dir --target /var/lib/libvirt/images-thin
virsh pool-build thin_pool
virsh pool-start thin_pool
virsh pool-autostart thin_pool
```

### KVM/Qemu Virtualization Tuning

```bash
cat > /etc/sysctl.d/99-kvm-tuning.conf << EOF
vm.swappiness = 5
vm.dirty_ratio = 20
vm.dirty_background_ratio = 5

kernel.sched_child_runs_first = 1
kernel.sched_autogroup_enabled = 1

net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
net.core.netdev_max_backlog = 30000
net.ipv4.tcp_fastopen = 3

vm.dirty_expire_centisecs = 500
vm.vfs_cache_pressure = 50

fs.file-max = 2097152

vm.nr_hugepages = 48
vm.hugetlb_shm_group = 36
vm.nr_overcommit_hugepages = 4
EOF


sysctl -p /etc/sysctl.d/99-kvm-tuning.conf

echo 'none' > /sys/block/nvme0n1/queue/scheduler
echo 'none' > /sys/block/nvme1n1/queue/scheduler

echo '2048' > /sys/block/nvme0n1/queue/read_ahead_kb
echo '2048' > /sys/block/nvme1n1/queue/read_ahead_kb

echo '128' > /sys/block/nvme0n1/queue/nr_requests
echo '128' > /sys/block/nvme1n1/queue/nr_requests

cat > /etc/udev/rules.d/60-scheduler.rules << EOF
# Set scheduler for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"
# Set read-ahead for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/read_ahead_kb}="2048"
# Set nr_requests for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/nr_requests}="128"
EOF

mkdir -p /dev/hugepages
echo "hugetlbfs  /dev/hugepages  hugetlbfs  mode=1770,gid=36  0 0" >> /etc/fstab
mount -a

cat >> /etc/libvirt/qemu.conf << EOF
hugetlbfs_mount = "/dev/hugepages"
EOF

systemctl restart libvirtd
```

### Configuration for AMD CPU and tuned
```bash
mkdir -p /etc/tuned/amd-ryzen-vm
cat > /etc/tuned/amd-ryzen-vm/tuned.conf << EOF
[main]
summary=Optimize for AMD Ryzen KVM host
include=virtual-host

[cpu]
force_latency=1
governor=performance
energy_perf_bias=performance
min_perf_pct=100

[vm]
transparent_hugepages=never

[sysctl]
kernel.numa_balancing=0
EOF

dnf install -y tuned

systemctl enable --now tuned
tuned-adm profile amd-ryzen-vm

echo "on" > /sys/devices/system/cpu/smt/control

cat > /etc/modprobe.d/kvm-amd.conf << EOF
options kvm_amd nested=1
options kvm ignore_msrs=1
options kvm report_ignored_msrs=0
EOF

modprobe -r kvm_amd
modprobe kvm_amd
```
