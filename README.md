### Disable swap:
```bash
sudo swapoff -a
```

### 1. Install Container Runtime (containerd)
```bash
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl

# Install containerd
sudo apt install -y containerd

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Enable and restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### 2. Add Kubernetes Repository and Install Tools
```bash
# Add Kubernetes GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes apt repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install Kubernetes tools
sudo apt update
sudo apt install -y kubelet kubeadm kubectl

# Prevent automatic updates
sudo apt-mark hold kubelet kubeadm kubectl
```

### 3. Kernel Modules and Sysctl Settings (Required for Networking)
```bash
# Load kernel modules
sudo modprobe br_netfilter

# Persist module load
echo 'br_netfilter' | sudo tee /etc/modules-load.d/k8s.conf

# Apply sysctl settings
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply settings without reboot
sudo sysctl --system
```

### 4. Initialize Kubernetes Control Plane (Master Node Only)
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
### After successful init:
```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 5. Install a CNI Plugin (e.g., Calico)
```bash
kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calico.yaml
```

### Join Worker Nodes
```bash
sudo kubeadm join <MASTER_IP>:<PORT> --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```
### Prompted by the master node after init

### Cluster Verification

```bash
kubectl get nodes
kubectl get pods -A
```