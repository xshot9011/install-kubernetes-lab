# Install Kubernetes Cluster

1. [System requirements](#system-requirements)
    1. [Required software ](#required-software)
    2. [Init VM](#init-vm)
2. [Deployment with Kubeadm](#deployment-with-kubeadm)
    1. [Letting iptable see bridge traffic (ALL NODES)](#letting-iptable-see-bridge-traffic-all-nodes)
    2. [Install & Config Container Runtime (ALL NODES)](#install-&-config-container-runtime-all-nodes)
    3. [Install kubectl, kubeadm, kubelet (ALL NODES)](#install-kubectl-kubeadm-kubelet-all-nodes)
    4. [Initialize controlplane node (MASTER NODES)](#initialize-controlplane-node-master-nodes)
    5. [Join the worker node (WORKER NODES)](#join-the-worker-node-worker-nodes)
3. [Install the Hardway](#install-the-hardway)

## System requirements

### Required software 

- Vagrant version 2.2.19
- VirtualBox version 6.1.30
- Vagrantfile
    - 1 Master node
    - 1 Worker node

`I have low spec hardware`

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
vagrant destroy
```

### IP Address

- Master node
    - `--pod-network-cidr 10.244.0.0/16`
    - `--apiserver-advertise-address=192.168.56.2` (bind to <ip> interface)

## Deployment with Kubeadm

Related: [doc](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### Letting iptable see bridge traffic (ALL NODES)

```bash
# Load br_netfilter module
sudo modprobe br_netfilter
lsmod | grep br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
# Make Linux correctly see bridged traffic
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

```

### Install & Config Container Runtime (ALL NODES)

```bash
# Remove all previous docker
sudo apt-get remove docker docker-engine docker.io containerd runc
# Update source list
sudo apt-get update
# Install package
sudo apt-get install ca-certificates curl gnupg lsb-release --yes
# Add Docker's official pub key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# Add package to source list
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# Reload
sudo apt-get update
# Get the selected Docker version
apt-cache madison docker-ce
# Install contianer runtime
sudo apt-get install docker-ce=5:18.09.1~3-0~ubuntu-bionic docker-ce-cli=5:18.09.1~3-0~ubuntu-bionic containerd.io --yes
# Config Docker daemon; using systemd to manage container's cgroup
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl status docker
```

### Install kubectl, kubeadm, kubelet (ALL NODES)

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Initialize controlplane node (MASTER NODES)

```bash
sudo su
kubeadm init --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address=192.168.56.2
```

For more option

- `--control-plane-endpoint=<load_balancer_ip_or_domain>`
- `--cri-socket=<for_non_docker>`

After running command above; token to join the node

```txt
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.2:6443 --token xatsaj.c439ycnvyheenpso \
        --discovery-token-ca-cert-hash sha256:88f33234395ef8cdacf4076c68956d26a471e2fa10806627453d7258fdce227f
```

We will follow the previous given output

```bash
# Logout from root user
logout
# Create .config
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
## The Pod network solutions is not installed; Node status is NotReady
# Deploy pod network
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

### Join the worker node (WORKER NODES)

```bash
sudo su
kubeadm join 192.168.56.2:6443 --token <token> \
    --discovery-token-ca-cert-hash <ca_hashed_token>
```

## Install the Hardway

[Solution](https://www.youtube.com/playlist?list=PL2We04F3Y_41jYdadX55fdJplDvgNGENo) here; Don't be cheater