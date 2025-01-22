## Kubernetes Installation Steps using Kubeadm

### History of Commands Used:

#### **1. Set Hostname**
```bash
hostname
hostnamectl set-hostname master-node
logout
```

#### **2. Update and Install Dependencies**
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

#### **3. Add Kubernetes Repository**
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

#### **4. Install Kubernetes Components**
```bash
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
kubeadm version
```

#### **5. Update and Install Container Runtime**
```bash
sudo apt update
sudo apt upgrade
sudo apt install containerd
```

#### **6. Configure Containerd**
```bash
sudo mkdir /etc/containerd
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

#### **7. Verify Configuration**
```bash
ps -p 1
cat /etc/containerd/config.toml | grep -i SystemdCgroup -B 50
```

#### **8. Initialize Kubernetes Master Node**
```bash
sudo kubeadm init --apiserver-advertise-address=10.128.15.197 --pod-network-cidr=10.244.0.0/16 --upload-certs --v=5
```

#### **9. Enable IP Forwarding**
```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo nano /etc/sysctl.conf
# Add: net.ipv4.ip_forward=1
sudo sysctl -p
cat /proc/sys/net/ipv4/ip_forward
```

#### **10. Retry Initialization (if required)**
```bash
sudo kubeadm init --apiserver-advertise-address=10.128.15.197 --pod-network-cidr=10.244.0.0/16 --upload-certs --v=5
sudo cat /etc/kubernetes/admin.conf
```

#### **11. Configure kubectl for the Master Node**
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### **12. Install Pod Network (Calico)**
```bash
kubectl apply -f https://docs.projectcalico.org/v3.25/manifests/calico.yaml
```

#### **13. Install Ingress Controller**
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

#### **14. Verify the Installation**
```bash
kubectl get nodes
kubectl get pods -n kube-system
```

### Notes:
- Ensure the container runtime (containerd) is correctly configured with `SystemdCgroup = true`.
- IP forwarding must be enabled before initializing the Kubernetes cluster.
- Always verify the status of the cluster using `kubectl get nodes` and `kubectl get pods -n kube-system`.

### Troubleshooting:
- If any port-related errors occur, ensure the required ports (6443, 10250, etc.) are not blocked by firewalls.
- For IP forwarding issues, check `/proc/sys/net/ipv4/ip_forward` and update `/etc/sysctl.conf` accordingly.
- 


------------------------------------------------------------------------------------------------------------------------------------

Notes: care of IP add-into Kubeadm --init need to replace by local host IP add 
#!/bin/bash

# Step 1: Set Hostname
echo "Setting the hostname to master-node"
hostnamectl set-hostname master-node
logout

# Step 2: Update and Install Dependencies
echo "Updating package list and installing dependencies"
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Step 3: Add Kubernetes Repository
echo "Adding Kubernetes APT repository"
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update

# Step 4: Install Kubernetes Components
echo "Installing Kubernetes components"
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
kubeadm version

# Step 5: Update and Install Container Runtime
echo "Updating and installing container runtime (containerd)"
sudo apt update
sudo apt upgrade -y
sudo apt install -y containerd

# Step 6: Configure Containerd
echo "Configuring containerd"
sudo mkdir -p /etc/containerd
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd

# Step 7: Verify Configuration
echo "Verifying containerd configuration"
ps -p 1
cat /etc/containerd/config.toml | grep -i SystemdCgroup -B 50

# Step 8: Initialize Kubernetes Master Node
echo "Initializing Kubernetes master node"
sudo kubeadm init --apiserver-advertise-address=10.128.15.197 --pod-network-cidr=10.244.0.0/16 --upload-certs --v=5

# Step 9: Enable IP Forwarding
echo "Enabling IP forwarding"
sudo sysctl -w net.ipv4.ip_forward=1
sudo nano /etc/sysctl.conf
# Add: net.ipv4.ip_forward=1
sudo sysctl -p
cat /proc/sys/net/ipv4/ip_forward

# Step 10: Retry Initialization (if required)
echo "Retrying Kubernetes initialization"
sudo kubeadm init --apiserver-advertise-address=10.44.2.145 --pod-network-cidr=10.244.0.0/16 --upload-certs --v=5
sudo cat /etc/kubernetes/admin.conf

# Step 11: Configure kubectl for the Master Node
echo "Configuring kubectl for master node"
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Step 12: Install Pod Network (Calico)
echo "Installing Calico pod network"
kubectl apply -f https://docs.projectcalico.org/v3.25/manifests/calico.yaml

# Step 13: Install Ingress Controller
echo "Installing Nginx Ingress Controller"
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Step 14: Verify the Installation
echo "Verifying Kubernetes installation"
kubectl get nodes
kubectl get pods -n kube-system

# Notes:
# Ensure the container runtime (containerd) is correctly configured with SystemdCgroup = true.
# IP forwarding must be enabled before initializing the Kubernetes cluster.
# Always verify the status of the cluster using kubectl get nodes and kubectl get pods -n kube-system.
# Troubleshooting:
# If any port-related errors occur, ensure the required ports (6443, 10250, etc.) are not blocked by firewalls.
# For IP forwarding issues, check /proc/sys/net/ipv4/ip_forward and update /etc/sysctl.conf accordingly.

echo "Kubernetes setup complete!"


