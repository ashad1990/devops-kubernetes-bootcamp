# Environment Setup Guide

This guide covers everything you need to set up a complete Kubernetes development environment on macOS, Linux, or Windows.

---

## Table of Contents

1. [System Requirements](#system-requirements)
2. [Install kubectl](#install-kubectl)
3. [Install minikube](#install-minikube)
4. [Install kind](#install-kind)
5. [Docker Desktop](#docker-desktop)
6. [Install Helm 3](#install-helm-3)
7. [Optional Tools](#optional-tools)
8. [Configure kubectl](#configure-kubectl)
9. [Cloud Kubernetes Setup](#cloud-kubernetes-setup)
10. [Verify Complete Setup](#verify-complete-setup)
11. [Troubleshooting Installation](#troubleshooting-installation)

---

## System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **RAM**   | 4 GB    | 8 GB+       |
| **CPU**   | 2 cores | 4+ cores    |
| **Disk**  | 20 GB free | 40 GB+ free |
| **OS**    | See below | Latest stable |

### Supported Operating Systems

- **macOS**: 12 Monterey or later (Intel and Apple Silicon)
- **Linux**: Ubuntu 20.04+, Debian 11+, RHEL/CentOS 8+, Fedora 36+
- **Windows**: Windows 10 version 2004+ or Windows 11 (WSL2 recommended)

### Prerequisites

- Docker Desktop (or Docker Engine on Linux) installed and running
- Internet connection for image pulls
- Administrative/sudo privileges for installation

---

## Install kubectl

`kubectl` is the command-line tool for interacting with Kubernetes clusters.

**Latest stable version:** v1.29+

### macOS

**Using Homebrew (recommended):**

```bash
brew install kubectl
```

**Direct download (Intel):**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

**Direct download (Apple Silicon / ARM):**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

### Linux

**Using apt (Debian/Ubuntu):**

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubectl
```

**Using yum (RHEL/CentOS/Fedora):**

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
EOF

sudo yum install -y kubectl
```

**Direct download (Linux AMD64):**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

### Windows

**Using Chocolatey:**

```powershell
choco install kubernetes-cli
```

**Using winget:**

```powershell
winget install -e --id Kubernetes.kubectl
```

**Direct download:**

```powershell
curl.exe -LO "https://dl.k8s.io/release/v1.29.0/bin/windows/amd64/kubectl.exe"
# Move kubectl.exe to a directory in your PATH
```

### Verify kubectl Installation

```bash
kubectl version --client
kubectl version --client --output=yaml
```

Expected output:

```
Client Version: v1.29.x
Kustomize Version: v5.x.x
```

---

## Install minikube

minikube runs a local single-node Kubernetes cluster inside a VM or container.

**Latest stable version:** v1.32+

### macOS

**Using Homebrew (recommended):**

```bash
brew install minikube
```

**Direct download (Intel):**

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

**Direct download (Apple Silicon):**

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-arm64
sudo install minikube-darwin-arm64 /usr/local/bin/minikube
```

### Linux

**Direct download:**

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**Using apt (Debian/Ubuntu):**

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```

**Using rpm (RHEL/CentOS):**

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
sudo rpm -Uvh minikube-latest.x86_64.rpm
```

### Windows

**Using Chocolatey:**

```powershell
choco install minikube
```

**Using winget:**

```powershell
winget install -e --id Kubernetes.minikube
```

**Direct download:**

Download the installer from: `https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe`

### Start and Verify minikube

```bash
# Start with Docker driver (recommended)
minikube start --driver=docker

# Start with more resources
minikube start --driver=docker --cpus=4 --memory=8192

# Check cluster status
minikube status

# Verify cluster is accessible
kubectl cluster-info

# Open the Kubernetes dashboard
minikube dashboard

# Stop minikube
minikube stop

# Delete the cluster
minikube delete
```

---

## Install kind

kind (Kubernetes IN Docker) runs local Kubernetes clusters using Docker containers as nodes. It supports multi-node clusters.

**Latest stable version:** v0.22+

### macOS

```bash
brew install kind
```

### Linux

```bash
# AMD64
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# ARM64
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### Windows

```powershell
choco install kind
# or
winget install Kubernetes.kind
```

### Create a kind Cluster

**Single-node cluster:**

```bash
kind create cluster --name bootcamp
```

**Multi-node cluster with config:**

Create `kind-config.yaml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: bootcamp
nodes:
  - role: control-plane
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
  - role: worker
  - role: worker
```

```bash
kind create cluster --config kind-config.yaml
kind get clusters
kubectl cluster-info --context kind-bootcamp
```

---

## Docker Desktop

Docker Desktop includes a built-in Kubernetes option that is easy to enable.

### Enable Kubernetes in Docker Desktop

1. Open Docker Desktop
2. Click the gear icon (Settings)
3. Select **Kubernetes** from the left menu
4. Check **Enable Kubernetes**
5. Click **Apply & Restart**
6. Wait for the Kubernetes indicator (bottom-left) to turn green

### Switch kubectl Context

```bash
kubectl config use-context docker-desktop
kubectl cluster-info
```

### Resource Allocation (recommended)

In Docker Desktop → Resources:
- **CPUs**: 4+
- **Memory**: 6 GB+
- **Disk image size**: 60 GB+

---

## Install Helm 3

Helm is the Kubernetes package manager. Version 3 does not require Tiller.

**Latest stable version:** v3.14+

### macOS

```bash
brew install helm
```

### Linux

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Using apt:**

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | \
  sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Windows

```powershell
choco install kubernetes-helm
# or
winget install Helm.Helm
```

### Verify and Basic Helm Usage

```bash
helm version
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
helm install my-release bitnami/nginx
helm list
helm uninstall my-release
```

---

## Optional Tools

### k9s — Terminal UI for Kubernetes

```bash
# macOS
brew install k9s

# Linux
curl -sS https://webinstall.dev/k9s | bash

# Windows
choco install k9s
```

Launch: `k9s`  
Key shortcuts: `:pod` (list pods), `:svc` (services), `d` (describe), `l` (logs), `e` (edit), `ctrl+d` (delete)

### Lens — Desktop Kubernetes IDE

Download from: https://k8slens.dev/  
Available for macOS, Linux, and Windows. Provides cluster visualization, log streaming, terminal access, and Helm management.

### kubectx and kubens — Fast Context/Namespace Switching

```bash
# macOS
brew install kubectx

# Linux
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

Usage:

```bash
kubectx                    # list contexts
kubectx my-cluster         # switch context
kubens                     # list namespaces
kubens kube-system         # switch namespace
```

### stern — Multi-Pod Log Tailing

```bash
# macOS
brew install stern

# Linux
curl -LO https://github.com/stern/stern/releases/latest/download/stern_linux_amd64.tar.gz
tar -xzf stern_linux_amd64.tar.gz
sudo mv stern /usr/local/bin/
```

Usage:

```bash
stern my-app               # tail logs from all pods matching "my-app"
stern . -n default         # tail all pods in default namespace
stern my-app --since 10m   # last 10 minutes of logs
```

---

## Configure kubectl

### kubeconfig Location and Format

The default kubeconfig file is at `~/.kube/config`. You can override with the `KUBECONFIG` environment variable.

**kubeconfig structure:**

```yaml
apiVersion: v1
kind: Config
clusters:
  - name: my-cluster
    cluster:
      server: https://127.0.0.1:6443
      certificate-authority-data: <base64-encoded-ca>
contexts:
  - name: my-context
    context:
      cluster: my-cluster
      user: my-user
      namespace: default
current-context: my-context
users:
  - name: my-user
    user:
      client-certificate-data: <base64-encoded-cert>
      client-key-data: <base64-encoded-key>
```

**Common context commands:**

```bash
kubectl config view                          # show full config
kubectl config get-contexts                  # list all contexts
kubectl config current-context              # show current context
kubectl config use-context my-context       # switch context
kubectl config set-context --current --namespace=my-namespace  # set default namespace
```

**Merge multiple kubeconfigs:**

```bash
export KUBECONFIG=~/.kube/config:~/.kube/config-cluster2
kubectl config view --flatten > ~/.kube/config-merged
```

### Shell Autocompletion

**Bash:**

```bash
# Add to ~/.bashrc
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc
```

**Zsh:**

```zsh
# Add to ~/.zshrc
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
echo 'alias k=kubectl' >> ~/.zshrc
echo 'compdef __start_kubectl k' >> ~/.zshrc
source ~/.zshrc
```

**Fish:**

```fish
kubectl completion fish | source
```

### Useful Aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kdel='kubectl delete'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
alias kns='kubectl config set-context --current --namespace'
alias kctx='kubectl config use-context'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kgn='kubectl get nodes'
alias kga='kubectl get all'
```

---

## Cloud Kubernetes Setup

### Google Kubernetes Engine (GKE)

```bash
# Install gcloud CLI
curl https://sdk.cloud.google.com | bash
gcloud init

# Create a GKE cluster
gcloud container clusters create bootcamp-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type e2-standard-2

# Get credentials
gcloud container clusters get-credentials bootcamp-cluster --zone us-central1-a

# Delete cluster
gcloud container clusters delete bootcamp-cluster --zone us-central1-a
```

### Amazon Elastic Kubernetes Service (EKS)

```bash
# Install eksctl
brew install eksctl   # macOS
# or
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create EKS cluster
eksctl create cluster \
  --name bootcamp-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3

# Delete cluster
eksctl delete cluster --name bootcamp-cluster --region us-east-1
```

### Azure Kubernetes Service (AKS)

```bash
# Install Azure CLI
brew install azure-cli   # macOS
# or
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash   # Linux

# Login
az login

# Create resource group
az group create --name bootcamp-rg --location eastus

# Create AKS cluster
az aks create \
  --resource-group bootcamp-rg \
  --name bootcamp-cluster \
  --node-count 3 \
  --enable-addons monitoring \
  --generate-ssh-keys

# Get credentials
az aks get-credentials --resource-group bootcamp-rg --name bootcamp-cluster

# Delete cluster
az aks delete --resource-group bootcamp-rg --name bootcamp-cluster
```

---

## Verify Complete Setup

Run through this checklist to confirm everything is working:

```bash
# 1. kubectl installed and working
kubectl version --client
# Expected: Client Version: v1.29.x

# 2. minikube installed
minikube version
# Expected: minikube version: v1.32.x

# 3. minikube cluster running
minikube start --driver=docker
minikube status
# Expected: host: Running, kubelet: Running, apiserver: Running

# 4. kubectl connected to cluster
kubectl cluster-info
kubectl get nodes
# Expected: node in Ready state

# 5. Helm installed
helm version
# Expected: version.BuildInfo{Version:"v3.14.x"...}

# 6. Deploy a test workload
kubectl create deployment hello-world --image=nginx:latest
kubectl get pods
kubectl expose deployment hello-world --port=80 --type=NodePort
minikube service hello-world --url

# 7. Clean up
kubectl delete deployment hello-world
kubectl delete service hello-world

# 8. kind installed (optional)
kind version
# Expected: kind v0.22.x

# 9. Check optional tools
k9s version
helm repo list
```

**All green? You're ready for the bootcamp!**

---

## Troubleshooting Installation

### kubectl: command not found

```bash
# Check if kubectl is in PATH
which kubectl
echo $PATH

# Add /usr/local/bin to PATH if missing
export PATH=$PATH:/usr/local/bin
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
```

### minikube start fails: "Exiting due to PROVIDER_DOCKER_NOT_RUNNING"

```bash
# Ensure Docker is running
docker info
# If not running, start Docker Desktop or Docker daemon
sudo systemctl start docker   # Linux
```

### minikube start fails: "Insufficient memory"

```bash
# Start with less memory
minikube start --driver=docker --memory=2048

# Or increase Docker Desktop memory allocation in Settings → Resources
```

### kubectl cannot connect: "connection refused"

```bash
# Check cluster is running
minikube status
# If stopped, restart
minikube start

# Check kubeconfig context
kubectl config current-context
kubectl config get-contexts
```

### Helm: "Error: INSTALLATION FAILED: cannot re-use a name"

```bash
# List existing releases
helm list --all-namespaces
# Uninstall the conflicting release
helm uninstall <release-name>
```

### Windows WSL2: minikube/Docker not accessible

```bash
# Ensure WSL2 integration is enabled in Docker Desktop:
# Settings → Resources → WSL Integration → enable your distro

# Verify Docker works inside WSL2
docker run hello-world
```

### kind cluster: "ERROR: failed to create cluster: node(s) already exist"

```bash
# Delete existing cluster first
kind delete cluster --name bootcamp
kind create cluster --name bootcamp
```

### Certificate errors / TLS issues

```bash
# Reset minikube certificates
minikube stop
minikube delete
minikube start --driver=docker

# Or regenerate certs
minikube ssh -- sudo kubeadm certs renew all
```

### Disk space issues

```bash
# Remove unused Docker resources
docker system prune -a

# Remove stopped minikube clusters
minikube delete --all --purge
```

---

*For additional help, visit the [Kubernetes documentation](https://kubernetes.io/docs/) or open an issue in this repository.*
