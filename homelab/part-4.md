# Part 4

*Date: 2025-03-12*

## KVM settings

## Let's create VM

### OpenSuse Leap Micro
```bash
mkdir -p /var/lib/libvirt/images-thin/iso
mkdir -p /var/lib/libvirt/images-thin/vms

cd /var/lib/libvirt/images-thin/iso
wget https://download.opensuse.org/distribution/leap/15.6/iso/openSUSE-Leap-Micro.x86_64-Default-SelfInstall.iso


virt-install \
  --name auth-server \
  --memory 8192 \
  --vcpus 4,sockets=1,cores=2,threads=2 \
  --cpu host-passthrough,cache.mode=passthrough \
  --features kvm_hidden=on \
  --boot uefi \
  --disk path=/var/lib/libvirt/images-thin/opensuse-leap-micro.qcow2,format=qcow2,size=100,bus=virtio,cache=none,io=native,discard=unmap \
  --controller type=scsi,model=virtio-scsi \
  --network bridge=br0,model=virtio \
  --graphics vnc,listen=0.0.0.0 \
  --noautoconsole \
  --osinfo detect=on,name=opensuse15.5 \
  --cdrom /var/lib/libvirt/images-thin/iso/openSUSE-Leap-Micro.x86_64-Default-SelfInstall.iso
```

### Talos Control Node

```bash
wget https://factory.talos.dev/image/376567988ad370138ad8b2698212367b8edcb69b5fd68c80be1f2ec7d603b4ba/v1.8.3/nocloud-amd64.iso talos-amd64.iso

virt-install \
  --name talos-control \
  --memory 10240 \
  --vcpus 4,sockets=1,cores=2,threads=2 \
  --cpu host-passthrough \
  --features kvm_hidden=on \
  --boot uefi \
  --disk path=/var/lib/libvirt/images-thin/talos-control.qcow2,format=qcow2,size=120,bus=virtio,discard=unmap \
  --controller type=scsi,model=virtio-scsi \
  --network bridge=br0,model=virtio \
  --graphics vnc,listen=0.0.0.0 \
  --os-variant generic \
  --noautoconsole \
  --cdrom /var/lib/libvirt/images-thin/iso/nocloud-amd64.iso
```
### Talos Worker Node

```bash
virt-install \
  --name talos-worker1 \
  --memory 16384 \
  --vcpus 6,sockets=1,cores=3,threads=2 \
  --cpu host-passthrough \
  --features kvm_hidden=on \
  --boot uefi \
  --disk path=/var/lib/libvirt/images-thin/talos-worker1.qcow2,format=qcow2,size=300,bus=virtio,discard=unmap \
  --controller type=scsi,model=virtio-scsi \
  --network bridge=br0,model=virtio \
  --graphics vnc,listen=0.0.0.0 \
  --os-variant generic \
  --noautoconsole \
  --cdrom /var/lib/libvirt/images-thin/iso/nocloud-amd64.iso



virt-install \
  --name talos-worker2 \
  --memory 16384 \
  --vcpus 6,sockets=1,cores=3,threads=2 \
  --cpu host-passthrough \
  --features kvm_hidden=on \
  --boot uefi \
  --disk path=/var/lib/libvirt/images-thin/talos-worker2.qcow2,format=qcow2,size=300,bus=virtio,discard=unmap \
  --controller type=scsi,model=virtio-scsi \
  --network bridge=br0,model=virtio \
  --graphics vnc,listen=0.0.0.0 \
  --os-variant generic \
  --noautoconsole \
  --cdrom /var/lib/libvirt/images-thin/iso/nocloud-amd64.iso
```
### Database server

```bash
wget https://releases.ubuntu.com/noble/ubuntu-24.04.2-live-server-amd64.iso

virt-install \
  --name database-server \
  --memory 8192 \
  --vcpus 4,sockets=1,cores=2,threads=2 \
  --cpu host-passthrough \
  --features kvm_hidden=on \
  --boot uefi \
  --disk path=/var/lib/libvirt/images-thin/database-server.qcow2,format=qcow2,size=500,bus=virtio,discard=unmap \
  --controller type=scsi,model=virtio-scsi \
  --network bridge=br0,model=virtio \
  --graphics vnc,listen=0.0.0.0 \
  --os-variant ubuntu24.04 \
  --noautoconsole \
  --cdrom /var/lib/libvirt/images-thin/iso/ubuntu-24.04.2-live-server-amd64.iso
```
