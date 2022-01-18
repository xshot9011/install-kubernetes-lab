# Install Kubernetes Cluster

1. [System requirements](#system-requirements)
    1. [Required software ](#required-software)
    2. [Init VM](#init-vm)
2. [Deployment with Kubeadm](#deployment-with-kubeadm)
3. [Install the Hardway](#install-the-hardway)

## System requirements

### Required software 

- Vagrant version 2.2.19
- VirtualBox version 6.1.30
- Vagrantfile
    - 1 Master node
    - 2 Worker node

### Init VM

You can edit the number of worker and master here `Vagrantfile`

```bash
NUM_MASTER_NODE = 1
NUM_WORKER_NODE = 2
```

Run:

```bash
# To start VM
vagrant up [vm_name]
# To shutdown VM
vagrant halt [vm_name]
# To remove VM
vagrant destroy [vm_name]
```

## Deployment with Kubeadm

Related: [doc](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### Set up 

```bash
sudo modprobe br_netfilter
lsmod | grep br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

## Install the Hardway
