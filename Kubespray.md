# Using kubespray

## switch off swap
Switch off swap for kubernetes - if you do this after you run kubespray you will need to clear your fact cache
```
sudo sed -i '/.*swap.*/d' /etc/fstab && sudo swapoff -a
```

## Clone repo
```
git clone git@github.com:kubernetes-incubator/kubespray.git
```

## Run
```
ansible-playbook --become -i inventory/mycluster/hosts.ini cluster.yml
```
To troubleshoot run with debug and verbose
```
ANSIBLE_DEBUG=y ansible-playbook --become -i inventory/mycluster/hosts.ini cluster.yml  -vvvv
```

## Working with kubespray cluster
```
kubectl --kubeconfig=admin.conf get pods --all-namespaces
```
