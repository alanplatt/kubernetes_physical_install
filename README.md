# kubernetes_physical_install
Install kubernetes on some virtual/physical servers


## lxc/lxd Spike
```
# install ubuntu on physical hosts
# create usbdrive installer and install

# install lxd using snap on physcial hosts
sudo snap install lxd
sudo usermod -a -G lxd platt
newgrp lxd

/snap/bin/lxd init
sudo reboot

# install lxd with apt
sudo apt-get install lxd



# Bridged networking
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

cat >>  /etc/sysctl.conf<< EOF
fs.inotify.max_queued_events = 1048576
fs.inotify.max_user_instances = 1048576
fs.inotify.max_user_watches = 1048576
EOF

lxd init

#snip
#Do you want to configure the LXD bridge (yes/no)? yes
#Here we tell LXD to use our already-preconfigured bridge. This opens a new workflow as follows:
# - Would you like to setup a network bridge now? NO
# - Do you want to use an existing bridge? YES
# br0 as above

lxc profile show default #this should show your config above

#launch some servers
lxc launch ubuntu:16.04 ubuntu1604-4
lxc launch ubuntu:16.04 ubuntu1604-5
lxc launch ubuntu:16.04 ubuntu1604-6

lxc exec ubuntu1604-4 -- sed -i 's/manual/dhcp/g' /etc/network/interfaces.d/50-cloud-init.cfg
lxc exec ubuntu1604-5 -- sed -i 's/manual/dhcp/g' /etc/network/interfaces.d/50-cloud-init.cfg
lxc exec ubuntu1604-6 -- sed -i 's/manual/dhcp/g' /etc/network/interfaces.d/50-cloud-init.cfg

lxc restart ubuntu1604-4
lxc restart ubuntu1604-5
lxc restart ubuntu1604-6

lxc config get ubuntu1604-4 volatile.eth0.hwaddr
lxc config get ubuntu1604-5 volatile.eth0.hwaddr
lxc config get ubuntu1604-6 volatile.eth0.hwaddr


lxc exec ubuntu1604-4 -- useradd platt -G 'sudo,admin' -m -s /bin/bash
lxc exec ubuntu1604-5 -- useradd platt -G 'sudo,admin' -m -s /bin/bash
lxc exec ubuntu1604-6 -- useradd platt -G 'sudo,admin' -m -s /bin/bash

lxc exec ubuntu1604-4 -- sed 's|%sudo.*|%sudo   ALL=(ALL:ALL) NOPASSWD:ALL|g' /etc/sudoers
lxc exec ubuntu1604-5 -- sed 's|%sudo.*|%sudo   ALL=(ALL:ALL) NOPASSWD:ALL|g' /etc/sudoers
lxc exec ubuntu1604-6 -- sed 's|%sudo.*|%sudo   ALL=(ALL:ALL) NOPASSWD:ALL|g' /etc/sudoers

lxc exec ubuntu1604-4 -- rm /etc/alternatives/editor
lxc exec ubuntu1604-5 -- rm /etc/alternatives/editor
lxc exec ubuntu1604-6 -- rm /etc/alternatives/editor

lxc exec ubuntu1604-4 -- ln -s /usr/bin/vi /etc/alternatives/editor
lxc exec ubuntu1604-5 -- ln -s /usr/bin/vi /etc/alternatives/editor
lxc exec ubuntu1604-6 -- ln -s /usr/bin/vi /etc/alternatives/editor

lxc exec ubuntu1604-4 -- mkdir -p /home/platt/.ssh
lxc exec ubuntu1604-5 -- mkdir -p /home/platt/.ssh
lxc exec ubuntu1604-6 -- mkdir -p /home/platt/.ssh

lxc exec ubuntu1604-4 -- chmod 700 /home/platt/.ssh
lxc exec ubuntu1604-5 -- chmod 700 /home/platt/.ssh
lxc exec ubuntu1604-6 -- chmod 700 /home/platt/.ssh

lxc file push --mode=0600 /home/platt/.ssh/authorized_keys ubuntu1604-4/home/platt/.ssh/authorized_keys
lxc file push --mode=0600 /home/platt/.ssh/authorized_keys ubuntu1604-5/home/platt/.ssh/authorized_keys
lxc file push --mode=0600 /home/platt/.ssh/authorized_keys ubuntu1604-6/home/platt/.ssh/authorized_keys

lxc exec ubuntu1604-4 -- chown -R platt: /home/platt/.ssh
lxc exec ubuntu1604-5 -- chown -R platt: /home/platt/.ssh
lxc exec ubuntu1604-6 -- chown -R platt: /home/platt/.ssh
```

## Install docker and kubeadm (all nodes)
```
apt-get update
apt-get install -y docker.io apt-transport-https

cat << EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

sed -i '/.*swap.*/d' /etc/fstab
apt-get update
apt-get install -y kubelet kubeadm kubectl

#add --cgroup-driver=system to kubelet start up parameters in /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

#required to build containers
lxc config set workhorse security.nesting true
```

## Install etcd
[kubernetes docs](https://kubernetes.io/docs/setup/independent/high-availability/#installing-prerequisites-on-masters)
```
```
