# Kubernetes Tools Reference

A comprehensive reference for every tool used in this bootcamp — from core CLI utilities to monitoring stacks.

---

## Table of Contents

1. [Core Tools](#core-tools)
   - kubectl
   - minikube
   - kind
   - Docker
   - Helm 3
2. [GUI Tools](#gui-tools)
   - k9s
   - Lens
   - Octant
   - Kubernetes Dashboard
3. [Productivity Tools](#productivity-tools)
   - kubectx / kubens
   - stern
   - kustomize
   - kubeval
   - kube-ps1
4. [Developer Tools](#developer-tools)
   - Skaffold
   - Tilt
5. [Monitoring Tools](#monitoring-tools)
   - Prometheus
   - Grafana

---

## Core Tools

### kubectl

**Description:** The official Kubernetes command-line tool. `kubectl` lets you interact with any Kubernetes cluster — local or remote — to deploy applications, inspect resources, view logs, and manage cluster state.

**Version:** 1.29+ (always match or be within one minor version of your cluster)

**Install:**

```bash
# macOS (Homebrew)
brew install kubectl

# Linux (curl)
curl -LO "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# Windows (Chocolatey)
choco install kubernetes-cli

# Windows (Winget)
winget install Kubernetes.kubectl
```

**Key Usage Examples:**

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes -o wide

# Working with pods
kubectl get pods -A                        # all namespaces
kubectl get pods -n kube-system
kubectl describe pod <name>
kubectl logs <pod> -c <container> -f       # follow logs
kubectl exec -it <pod> -- /bin/bash        # interactive shell

# Apply/delete manifests
kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml
kubectl apply -f ./directory/             # all YAMLs in dir

# Rollouts
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>

# Resource inspection
kubectl get all -n <namespace>
kubectl top pods
kubectl top nodes

# Context management
kubectl config get-contexts
kubectl config use-context <name>
kubectl config current-context
```

**Why Use It:** It is the only official, universally-supported interface to Kubernetes. Every other tool wraps or extends kubectl.

---

### minikube

**Description:** Runs a single-node (or multi-node) Kubernetes cluster locally inside a VM or container. The fastest way to get a Kubernetes environment on a developer laptop.

**Version:** 1.32+

**Install:**

```bash
# macOS
brew install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Windows (Chocolatey)
choco install minikube

# Windows (Winget)
winget install Kubernetes.minikube
```

**Key Usage Examples:**

```bash
minikube start                            # default start
minikube start --driver=docker            # use docker driver
minikube start --nodes=3                  # multi-node cluster
minikube start --kubernetes-version=v1.28.0
minikube status
minikube stop
minikube delete

# Addons
minikube addons list
minikube addons enable ingress
minikube addons enable metrics-server
minikube addons enable dashboard

# Tunnel and service access
minikube tunnel                           # expose LoadBalancer services
minikube service <svc-name> --url
minikube dashboard                        # open dashboard in browser

# SSH into node
minikube ssh
```

**Why Use It:** Zero infrastructure cost, works offline, ships with common addons, and mirrors real cluster behaviour closely enough for learning all Kubernetes concepts.

---

### kind (Kubernetes IN Docker)

**Description:** Runs Kubernetes clusters in Docker containers. Designed for testing Kubernetes itself but perfect for CI pipelines and local multi-node setups.

**Version:** 0.22+

**Install:**

```bash
# macOS
brew install kind

# Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x kind && sudo mv kind /usr/local/bin/

# Windows (Chocolatey)
choco install kind

# Go install (any platform)
go install sigs.k8s.io/kind@latest
```

**Key Usage Examples:**

```bash
kind create cluster                       # default cluster
kind create cluster --name my-cluster
kind create cluster --config kind-config.yaml   # multi-node

# kind-config.yaml example:
# kind: Cluster
# apiVersion: kind.x-k8s.io/v1alpha4
# nodes:
# - role: control-plane
# - role: worker
# - role: worker

kind get clusters
kind delete cluster --name my-cluster

# Load local image into kind
kind load docker-image my-app:local --name my-cluster

kubectl cluster-info --context kind-my-cluster
```

**Why Use It:** Lighter than minikube for CI, supports multiple nodes on a single machine, and is the standard tool for testing Kubernetes controllers and operators.

---

### Docker

**Description:** The container runtime and image build tool. While Kubernetes supports multiple container runtimes (containerd, CRI-O), Docker remains the dominant tool for building and pushing images.

**Version:** 24.x+

**Install:**

```bash
# macOS / Windows — Docker Desktop
# https://www.docker.com/products/docker-desktop/

# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Add user to docker group (Linux)
sudo usermod -aG docker $USER
```

**Key Usage Examples:**

```bash
# Build and tag
docker build -t my-app:v1.0 .
docker build -t registry.example.com/my-app:latest .

# Push/pull
docker push registry.example.com/my-app:latest
docker pull nginx:1.25

# Run container
docker run -d -p 8080:80 --name web nginx:1.25
docker exec -it web /bin/sh

# Inspect
docker images
docker ps -a
docker logs web -f
docker inspect web

# Compose (multi-container)
docker compose up -d
docker compose down
```

**Why Use It:** Every Kubernetes workload starts as a container image. Docker is the standard tool for building, tagging, and pushing those images.

---

### Helm 3

**Description:** The Kubernetes package manager. Helm packages Kubernetes manifests into reusable, versioned "charts" with templating support and lifecycle management (install, upgrade, rollback).

**Version:** 3.14+

**Install:**

```bash
# macOS
brew install helm

# Linux (script)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows (Chocolatey)
choco install kubernetes-helm

# Windows (Scoop)
scoop install helm
```

**Key Usage Examples:**

```bash
# Repo management
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search
helm search repo nginx
helm search hub wordpress

# Install / upgrade
helm install my-nginx bitnami/nginx
helm install my-release bitnami/nginx --values custom-values.yaml
helm upgrade my-nginx bitnami/nginx --set replicaCount=3
helm upgrade --install my-nginx bitnami/nginx  # idempotent

# Inspect
helm list -A
helm status my-nginx
helm get values my-nginx
helm get manifest my-nginx

# Rollback
helm rollback my-nginx 1
helm history my-nginx

# Chart development
helm create my-chart
helm lint my-chart/
helm template my-release my-chart/
helm package my-chart/
```

**Why Use It:** Deploying complex applications (Prometheus, cert-manager, ArgoCD) would require managing hundreds of YAML files manually. Helm reduces this to a single install command with customisable values.

---

## GUI Tools

### k9s

**Description:** A terminal-based UI for Kubernetes that provides a curses-style dashboard in your terminal. Lets you navigate resources, view logs, exec into pods, and manage workloads without typing full kubectl commands.

**Version:** 0.31+

**Install:**

```bash
# macOS
brew install k9s

# Linux (binary)
curl -Lo k9s.tar.gz https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_amd64.tar.gz
tar xf k9s.tar.gz && sudo mv k9s /usr/local/bin/

# Windows (Chocolatey)
choco install k9s

# Homebrew (Linux)
brew install k9s
```

**Key Usage:**

```bash
k9s                          # open with current context
k9s -n kube-system           # open in specific namespace
k9s --context staging        # open with specific context

# Inside k9s:
# :pods         → navigate to pods view
# :deployments  → navigate to deployments
# l             → logs
# d             → describe
# e             → edit YAML
# ctrl-d        → delete resource
# /             → filter/search
# ?             → help
```

**Why Use It:** Dramatically speeds up day-to-day cluster operations. Essential for anyone spending significant time managing clusters.

---

### Lens

**Description:** A full desktop IDE for Kubernetes. Provides a graphical interface to manage multiple clusters, view resource graphs, and integrate extensions.

**Install:** Download from https://k8slens.dev/ (macOS, Windows, Linux installers available).

**Why Use It:** Best graphical experience for multi-cluster management. Great for teams and those who prefer GUI over CLI.

---

### Octant

**Description:** A web-based Kubernetes dashboard by VMware (now open source). Runs locally and provides a rich view of cluster resources without requiring any in-cluster components.

**Install:**

```bash
# macOS
brew install octant

# Linux / Windows: download from
# https://github.com/vmware-archive/octant/releases
```

**Why Use It:** No in-cluster installation required. Good for visualising resource relationships and plugin ecosystem.

---

### Kubernetes Dashboard

**Description:** The official web UI for Kubernetes, deployed as a pod in the cluster.

**Install / Access:**

```bash
# With minikube
minikube dashboard

# Manual deploy
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Access via proxy
kubectl proxy
# Open: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

**Why Use It:** Official, widely recognised, integrates with RBAC. Good for organisations that need a sanctioned in-cluster dashboard.

---

## Productivity Tools

### kubectx / kubens

**Description:** `kubectx` switches between Kubernetes contexts (clusters). `kubens` switches between namespaces. Both dramatically reduce typing.

**Install:**

```bash
# macOS
brew install kubectx

# Linux
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

**Usage:**

```bash
kubectx                      # list contexts
kubectx production           # switch to production context
kubectx -                    # switch back to previous context

kubens                       # list namespaces
kubens kube-system           # switch to kube-system namespace
kubens -                     # switch back
```

**Why Use It:** Switching contexts and namespaces with kubectl is verbose. These two commands make it a single word.

---

### stern

**Description:** Multi-pod log tailing. `stern` streams logs from multiple pods simultaneously with colour-coded output, regex filtering, and timestamps.

**Install:**

```bash
# macOS
brew install stern

# Linux
curl -Lo stern https://github.com/stern/stern/releases/latest/download/stern_linux_amd64
chmod +x stern && sudo mv stern /usr/local/bin/
```

**Usage:**

```bash
stern my-app                 # logs from all pods matching "my-app"
stern . -n production        # all pods in production namespace
stern my-app --since 5m      # last 5 minutes
stern my-app --container web # specific container only
stern "web|api" --regex      # regex pod selector
```

**Why Use It:** `kubectl logs` only tails one pod at a time. When debugging a deployment with multiple replicas, stern gives you a unified view.

---

### kustomize

**Description:** A built-in Kubernetes configuration management tool. Uses layered overlays to customise base YAML files without templating (no placeholders).

**Install:** Built into kubectl (`kubectl apply -k`), or standalone:

```bash
# macOS
brew install kustomize

# Linux
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
sudo mv kustomize /usr/local/bin/
```

**Usage:**

```bash
kustomize build ./overlays/production | kubectl apply -f -
kubectl apply -k ./overlays/staging

# kustomization.yaml structure:
# resources:
#   - ../../base
# patches:
#   - patch.yaml
# images:
#   - name: my-app
#     newTag: v2.0
```

**Why Use It:** Kustomize is built into kubectl and is the standard way to manage environment-specific configuration without duplicating YAML.

---

### kubeval

**Description:** Validates Kubernetes YAML manifests against the official Kubernetes JSON Schema. Catches errors before `kubectl apply`.

**Install:**

```bash
# macOS
brew install kubeval

# Linux
curl -Lo kubeval.tar.gz https://github.com/instrumenta/kubeval/releases/latest/download/kubeval-linux-amd64.tar.gz
tar xf kubeval.tar.gz && sudo mv kubeval /usr/local/bin/
```

**Usage:**

```bash
kubeval my-deployment.yaml
kubeval --kubernetes-version 1.29.0 manifests/*.yaml
find . -name "*.yaml" | xargs kubeval
```

**Why Use It:** Catches schema errors and invalid field names immediately, before applying to any cluster.

---

### kube-ps1

**Description:** A shell prompt segment that shows the current Kubernetes context and namespace. Prevents mistakes caused by being in the wrong cluster.

**Install:**

```bash
# macOS
brew install kube-ps1

# Add to ~/.bashrc or ~/.zshrc:
source "$(brew --prefix)/opt/kube-ps1/share/kube-ps1.sh"
PS1='$(kube_ps1)'$PS1

# Manual install
git clone https://github.com/jonmosco/kube-ps1.git
source kube-ps1/kube-ps1.sh
```

**Example prompt output:** `(⎈ production|kube-system) user@host:~$`

**Why Use It:** One of the most common Kubernetes mistakes is running a destructive command in the wrong cluster. kube-ps1 makes your current context always visible.

---

## Developer Tools

### Skaffold

**Description:** Automates the build-push-deploy cycle during Kubernetes development. Watches source files, rebuilds the container image on change, and re-deploys automatically.

**Version:** 2.x

**Install:**

```bash
# macOS
brew install skaffold

# Linux
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64
chmod +x skaffold && sudo mv skaffold /usr/local/bin/
```

**Usage:**

```bash
skaffold dev          # watch mode: rebuild & redeploy on file change
skaffold run          # one-shot build and deploy
skaffold build        # build only
skaffold delete       # clean up deployed resources
```

**Why Use It:** Eliminates the manual build → tag → push → apply loop during active development.

---

### Tilt

**Description:** A developer experience platform for Kubernetes. Like Skaffold but with a live web UI showing build/deploy status, logs, and resource health in real time.

**Install:**

```bash
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/tilt-dev/tilt/master/scripts/install.sh | bash

# Windows (Scoop)
scoop bucket add tilt https://github.com/tilt-dev/scoop-bucket
scoop install tilt
```

**Usage:**

```bash
tilt up           # start dev loop, open browser UI
tilt down         # tear down resources
tilt ci           # run one-shot in CI mode
```

**Why Use It:** Best-in-class developer experience for Kubernetes-native development, especially on microservices projects with multiple containers.

---

## Monitoring Tools

### Prometheus

**Description:** Open-source monitoring and alerting toolkit. Scrapes metrics from Kubernetes components and applications via HTTP endpoints, stores them in a time-series database, and evaluates alerting rules.

**Deploy with Helm:**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace

kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090 -n monitoring
# Open: http://localhost:9090
```

**Key Concepts:**
- **Scrape config** — defines which endpoints to scrape for metrics
- **PromQL** — query language for time-series data
- **Alertmanager** — handles alert routing (email, Slack, PagerDuty)
- **ServiceMonitor** — CRD for auto-discovering scrape targets

**Why Use It:** The de-facto standard Kubernetes monitoring tool. Tight integration with the ecosystem via `kube-state-metrics` and `node-exporter`.

---

### Grafana

**Description:** Visualisation platform for metrics, logs, and traces. Connects to Prometheus (and dozens of other data sources) to render dashboards, graphs, and alerts.

**Access after Helm deploy:**

```bash
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring
# Open: http://localhost:3000
# Default credentials: admin / prom-operator

# Get auto-generated password
kubectl get secret prometheus-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
```

**Key Features:**
- Pre-built Kubernetes dashboards (Node Exporter Full, K8s Cluster Overview)
- Alerting with notification channels
- Dashboard-as-code (JSON / Grafonnet)

**Why Use It:** Prometheus stores metrics but has limited visualisation. Grafana provides the production-grade dashboard experience every operations team needs.

---

## Quick Reference Card

| Tool | Purpose | One-liner |
|------|---------|-----------|
| kubectl | Cluster management CLI | `kubectl get pods -A` |
| minikube | Local cluster | `minikube start` |
| kind | Local cluster (Docker) | `kind create cluster` |
| Helm | Package manager | `helm install app chart/` |
| k9s | Terminal UI | `k9s` |
| kubectx | Switch contexts | `kubectx prod` |
| kubens | Switch namespaces | `kubens monitoring` |
| stern | Multi-pod logs | `stern my-app` |
| kustomize | Config overlays | `kubectl apply -k .` |
| Skaffold | Dev loop | `skaffold dev` |
| Prometheus | Metrics | `helm install prom ...` |
| Grafana | Dashboards | `kubectl port-forward ...` |
