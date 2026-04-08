# Kubernetes POC Setup (v1.29)

- Single-node / Multi-node ready
- Local VM / Cloud VM
- Minimal setup using kubeadm + containerd (modern approach)
- containerd (default) | Docker (optional)

---

## Table of Contents
- [System Requirements](#system-requirements)
- [Pre-flight Checklist](#pre-flight-checklist)
- [Step 1 — System Preparation](#step-1--system-preparation)
- [Step 2 — Container Runtime](#step-2--container-runtime)
  - [Option A: containerd (Recommended)](#option-a-containerd-recommended)
  - [Option B: Docker (Legacy)](#option-b-docker-legacy)
- [Step 3 — Install Kubernetes v1.29](#step-3--install-kubernetes-v129)
- [Step 4 — Initialize Master Node](#step-4--initialize-master-node)
- [Step 5 — Configure kubectl](#step-5--configure-kubectl)
- [Step 6 — Install Network Plugin (Flannel)](#step-6--install-network-plugin-flannel)
- [Step 7 — Join Worker Nodes](#step-7--join-worker-nodes)
- [Step 8 — MetalLB Setup (Extensive)](#step-8--metallb-setup-extensive)
- [Verify & Test](#verify--test)
- [Troubleshooting](#troubleshooting)

---

## System Requirements

### Minimum (Single-Node POC)

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU       | 2 cores | 4 cores     |
| RAM       | 2 GB    | 4 GB        |
| Disk      | 20 GB   | 50 GB       |
| OS        | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04 LTS |
| Network   | Static IP or DHCP reserved | Static IP |

### Multi-Node Setup (1 Master + 1+ Workers)

| Node   | Role          | Min CPU | Min RAM |
|--------|---------------|---------|---------|
| node-1 | Control Plane | 2 cores | 2 GB    |
| node-2 | Worker        | 2 cores | 2 GB    |

> **Note:** All nodes must be able to reach each other over the network. Ports 6443 (API server), 2379-2380 (etcd), 10250 (kubelet), 10256 (kube-proxy), and 30000-32767 (NodePort range) must be open between nodes.

---

## Pre-flight Checklist

Before starting on **all nodes**:

- [ ] Ubuntu 20.04 / 22.04 LTS (or Debian 11/12)
- [ ] Unique hostname per node (`hostnamectl set-hostname <name>`)
- [ ] Static IP or DHCP reservation per node
- [ ] SSH access with sudo privileges
- [ ] Internet access (for pulling images and packages)
- [ ] Swap disabled
- [ ] Firewall rules open for Kubernetes ports

```bash
# Set unique hostnames
sudo hostnamectl set-hostname master-node    # on master
sudo hostnamectl set-hostname worker-node-1  # on worker

# Add /etc/hosts entries (on all nodes)
sudo tee -a /etc/hosts <<EOF
192.168.1.10  master-node
192.168.1.11  worker-node-1
EOF
```

---

## Step 1 — System Preparation

> Run on **all nodes** (master + workers)

```bash
set -e

echo "🔧 Updating system..."
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y curl apt-transport-https ca-certificates gnupg lsb-release

# --------------------------------
# Disable Swap (required by kubelet)
# --------------------------------
echo "❌ Disabling swap..."
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
# Verify swap is off
free -h

# --------------------------------
# Kernel Modules
# --------------------------------
echo "⚙️ Enabling kernel modules..."
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Verify modules loaded
lsmod | grep -E "overlay|br_netfilter"

# --------------------------------
# Sysctl / Networking
# --------------------------------
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

---

## Step 2 — Container Runtime

Choose **one** runtime per node. `containerd` is the modern default; `docker` is the legacy path.

---

### Option A: containerd (Recommended)

> Run on **all nodes**

```bash
echo "📦 Installing containerd..."
sudo apt install -y containerd

# Generate default config
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Enable SystemdCgroup (required for kubeadm)
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Verify the change
grep "SystemdCgroup" /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd

# Verify containerd is running
sudo systemctl status containerd
```

---

### Option B: Docker (Legacy)

> Run on **all nodes**  
> Docker uses `cri-dockerd` as a shim since Kubernetes v1.24+ removed native Docker support.

```bash
echo "🐳 Installing Docker..."
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

# Install cri-dockerd shim (required for k8s v1.24+)
VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest | grep tag_name | cut -d '"' -f 4 | sed 's/v//')
wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd_${VER}.3-0.ubuntu-jammy_amd64.deb
sudo dpkg -i cri-dockerd_${VER}.3-0.ubuntu-jammy_amd64.deb

sudo systemctl daemon-reload
sudo systemctl enable cri-docker.socket cri-docker
sudo systemctl start cri-docker.socket cri-docker

# Verify
sudo systemctl status cri-docker
```

> **Note:** When using Docker, pass `--cri-socket unix:///var/run/cri-dockerd.sock` to all `kubeadm` commands.

---

## Step 3 — Install Kubernetes v1.29

> Run on **all nodes**

```bash
echo "📥 Installing Kubernetes v1.29..."
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update -y
sudo apt install -y kubelet kubeadm kubectl

# Pin versions to prevent accidental upgrades
sudo apt-mark hold kubelet kubeadm kubectl

# Verify versions
kubeadm version
kubectl version --client
kubelet --version
```

---

## Step 4 — Initialize Master Node

> Run on **master node only**

### With containerd (default)

```bash
echo "🚀 Initializing cluster..."
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --kubernetes-version=1.29.0

# Save the kubeadm join command output — you will need it for workers!
```

### With Docker (cri-dockerd)

```bash
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --kubernetes-version=1.29.0 \
  --cri-socket unix:///var/run/cri-dockerd.sock
```

> After init completes, you will see a `kubeadm join` command at the bottom of the output. **Copy and save it.**

---

## Step 5 — Configure kubectl

> Run on **master node** (for the current user)

```bash
echo "🔑 Configuring kubectl..."
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verify master is Ready (may take ~60s)
kubectl get nodes
kubectl get pods -n kube-system
```

---

## Step 6 — Install Network Plugin (Flannel)

> Run on **master node only**

```bash
echo "🌐 Installing Flannel CNI..."
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Wait for Flannel pods to be Running
kubectl get pods -n kube-flannel --watch
```

### Single-Node POC: Remove Master Taint

```bash
# Allow workloads to be scheduled on the master (POC only — not for production)
kubectl taint nodes --all node-role.kubernetes.io/control-plane- || true
```

---

## Step 7 — Join Worker Nodes

> Run on **each worker node**

Use the `kubeadm join` command saved from Step 4.

### With containerd

```bash
sudo kubeadm join <MASTER_IP>:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

### With Docker (cri-dockerd)

```bash
sudo kubeadm join <MASTER_IP>:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --cri-socket unix:///var/run/cri-dockerd.sock
```

### If Token Expired (default TTL: 24h)

```bash
# On master — generate a new join command
kubeadm token create --print-join-command
```

### Verify Workers Joined

```bash
# On master
kubectl get nodes -o wide
```

---

## Step 8 — MetalLB Setup (Extensive)

MetalLB provides LoadBalancer-type Services for bare-metal / VM clusters (where cloud load balancers are unavailable).

### Overview

| Mode         | Description                                              |
|--------------|----------------------------------------------------------|
| L2 Mode      | ARP-based, works on any flat Layer-2 network (simplest) |
| BGP Mode     | Requires a BGP router, true HA load balancing            |

> This guide uses **L2 Mode** — suitable for POC, home labs, and single-subnet VMs.

### 8.1 — Strict ARP (Required for L2 Mode)

```bash
# Check current strictARP setting
kubectl get configmap kube-proxy -n kube-system -o yaml | grep strictARP

# Enable strictARP
kubectl get configmap kube-proxy -n kube-system -o yaml | \
  sed -e 's/strictARP: false/strictARP: true/' | \
  kubectl apply -f - -n kube-system
```

### 8.2 — Install MetalLB

```bash
echo "🌐 Installing MetalLB v0.14.5..."
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml

# Wait for MetalLB pods to be ready
echo "⏳ Waiting for MetalLB pods..."
kubectl wait --namespace metallb-system \
  --for=condition=ready pod \
  --selector=app=metallb \
  --timeout=120s

# Verify
kubectl get pods -n metallb-system
kubectl get daemonset -n metallb-system
kubectl get deployment -n metallb-system
```

### 8.3 — Configure IP Address Pool

> Replace `192.168.1.240-192.168.1.250` with an **unused IP range on your LAN subnet**.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.240-192.168.1.250   # <-- change to your subnet range
EOF
```

### 8.4 — Configure L2 Advertisement

```bash
cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
EOF
```

### 8.5 — Verify MetalLB Configuration

```bash
kubectl get ipaddresspool -n metallb-system
kubectl get l2advertisement -n metallb-system
kubectl describe ipaddresspool default-pool -n metallb-system
```

### 8.6 — MetalLB: Multiple Pools (Advanced)

You can define multiple pools for different service tiers:

```yaml
# metallb-pools.yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: public-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.240-192.168.1.245
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: internal-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.246-192.168.1.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - public-pool
  - internal-pool
```

```bash
kubectl apply -f metallb-pools.yaml
```

To pin a Service to a specific pool:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    metallb.universe.tf/address-pool: internal-pool
spec:
  type: LoadBalancer
  ...
```

### 8.7 — BGP Mode (Reference)

> Skip if using L2 Mode for POC.

```yaml
apiVersion: metallb.io/v1beta2
kind: BGPPeer
metadata:
  name: router
  namespace: metallb-system
spec:
  myASN: 64500
  peerASN: 64501
  peerAddress: 192.168.1.1  # your router IP
---
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  name: default
  namespace: metallb-system
```

---

## Verify & Test

```bash
# --------------------------------
# Cluster Health
# --------------------------------
kubectl get nodes -o wide
kubectl get pods -A

# --------------------------------
# Deploy Test App (nginx)
# --------------------------------
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --type=LoadBalancer --port=80

# Wait for External IP (MetalLB assigns it)
kubectl get svc nginx --watch

# Test — once EXTERNAL-IP is assigned
curl http://<EXTERNAL-IP>

# --------------------------------
# Full Status Check
# --------------------------------
kubectl get svc
kubectl get endpoints
kubectl describe svc nginx
```

---

## Troubleshooting

### Node stuck in NotReady

```bash
kubectl describe node <node-name>
journalctl -u kubelet -f
# Usually: CNI not installed, containerd not running, or swap still on
```

### kubeadm init fails

```bash
sudo kubeadm reset -f
sudo rm -rf /etc/cni/net.d
sudo rm -rf $HOME/.kube
# Fix the issue then re-run kubeadm init
```

### MetalLB not assigning IP

```bash
kubectl logs -n metallb-system -l app=metallb,component=controller
kubectl logs -n metallb-system -l app=metallb,component=speaker
# Common: strictARP not enabled, IP pool exhausted, wrong subnet range
```

### containerd not found as CRI

```bash
sudo systemctl status containerd
grep "SystemdCgroup" /etc/containerd/config.toml
# Should be: SystemdCgroup = true
sudo systemctl restart containerd kubelet
```

### Token expired for worker join

```bash
# On master
kubeadm token create --print-join-command
```

---

> **POC complete.** For production, consider: HA control plane (3 masters), etcd backup, proper RBAC, TLS certs rotation, and a persistent storage solution (Longhorn, Rook-Ceph).
