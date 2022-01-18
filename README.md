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

## Install the Hardway
