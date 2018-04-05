# Install KVM

## KVM install
```
apt-get update
apt-get install -y qemu-kvm libvirt-bin virtinst bridge-utils cpu-checker

kvm-ok || echo "KVM install issue"

```


## Bridged networking
```
# install bridge network with apt
sudo apt-get install lxd bridge-utils

sudo cat  >>/etc/network/interfaces << EOF
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto br0
iface br0 inet dhcp
    bridge_ports eth0

iface eth0 inet manual
EOF

systemctl restart networking

brctl show

```

## Create a VM
### Ubuntu 16.04
```
virt-install --os-variant list

export VM_NAME=ubuntu1604-4
virt-install \
  --virt-type=kvm \
  --name="${VM_NAME}" \
  --ram=2048 \
  --vcpus=2 \
  --os-variant=ubuntu16.04 \
  --virt-type=kvm \
  --hvm \
  --cdrom=/var/lib/libvirt/boot/ubuntu-16.04.3-server-amd64.iso \
  --network=bridge=br0,model=virtio \
  --graphics vnc \
  --disk path=/var/lib/libvirt/images/"${VM_NAME}".qcow2,size=20,bus=virtio,format=qcow2

#get vnc port
virsh dumpxml "${VM_NAME}" | grep vnc
#tunnle and use vnc client to connect to server and perform manual install
ssh silverstone.local.lan -L 5900:127.0.0.1:5900
# get mac address
virsh dumpxml "${VM_NAME}" | grep "mac address"

virsh autostart "${VM_NAME}"

virsh snapshot-create-as --name "fresh_install" --domain "${VM_NAME}"
```

## Ubuntu customisations
```
#as root
useradd platt -G 'sudo,admin' -m -s /bin/bash
sed -i 's|%sudo.*|%sudo   ALL=(ALL:ALL) NOPASSWD:ALL|g' /etc/sudoers
rm /etc/alternatives/editor
ln -s /usr/bin/vi /etc/alternatives/editor

apt-get update && apt-get upgrade -y && reboot
```
