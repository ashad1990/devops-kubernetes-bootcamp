# 🚀 DevOps Kubernetes Bootcamp — Hands-On Container Orchestration

[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.29+-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io)
[![Docker](https://img.shields.io/badge/Docker-24.0+-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docker.com)
[![Helm](https://img.shields.io/badge/Helm-3.x-0F1689?style=for-the-badge&logo=helm&logoColor=white)](https://helm.sh)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen?style=for-the-badge)](CONTRIBUTING.md)

> **"Kubernetes has become the Linux of distributed systems."** — Kelsey Hightower

---

## 🌍 Why Kubernetes? Why Now?

Kubernetes (K8s) is no longer optional for modern DevOps engineers — it is the **industry standard** for deploying, scaling, and managing containerized workloads. As of 2024:

- �� **96% of organizations** are either using or evaluating Kubernetes (CNCF Annual Survey)
- ☁️ Every major cloud provider — AWS (EKS), Azure (AKS), Google Cloud (GKE) — runs managed Kubernetes at its core
- 📈 The global container orchestration market is projected to exceed **$8 billion by 2026**
- 🔧 Kubernetes is the beating heart of modern CI/CD pipelines, microservices architectures, and GitOps workflows

Whether you are deploying a startup MVP or managing a Fortune 500 platform serving millions of users, Kubernetes provides the primitives to build **resilient, scalable, self-healing infrastructure**. This bootcamp gives you the hands-on skills to wield those primitives with confidence.

---

## 🎯 Learning Objectives

By the end of this bootcamp, you will be able to:

1. 📦 **Explain Kubernetes architecture** — describe the role of every control plane and worker node component and how they interact
2. 🛠️ **Deploy and manage workloads** — create Pods, ReplicaSets, Deployments, StatefulSets, DaemonSets, Jobs, and CronJobs from scratch
3. 🔗 **Expose applications** — configure ClusterIP, NodePort, LoadBalancer Services, and Ingress controllers with TLS termination
4. 🗂️ **Manage configuration and secrets** — use ConfigMaps and Secrets to decouple configuration from container images
5. 💾 **Handle persistent storage** — provision PersistentVolumes and PersistentVolumeClaims with multiple storage backends and StorageClasses
6. 🏷️ **Organise resources** — apply Namespaces, ResourceQuotas, LimitRanges, Labels, Selectors, and Annotations effectively
7. 🔒 **Secure clusters** — implement RBAC with ServiceAccounts, Roles, ClusterRoles, and Bindings; understand Pod Security Standards
8. 📡 **Configure cluster networking** — understand CNI, DNS resolution, NetworkPolicies, and Service mesh concepts
9. 🎁 **Package and release with Helm** — create, install, upgrade, and rollback Helm charts; use Helmfile for multi-chart management
10. 📊 **Observe running systems** — integrate Prometheus, Grafana, and the EFK/Loki stack for metrics, logs, and traces
11. 🔄 **Implement GitOps workflows** — apply GitOps principles with Argo CD or Flux for declarative continuous delivery
12. 🧰 **Troubleshoot real-world problems** — diagnose CrashLoopBackOff, Pending Pods, OOMKilled, and networking failures systematically

---

## ✅ Prerequisites

Before starting this bootcamp, ensure you are comfortable with:

### 🐳 Docker & Containers
- Building images with a `Dockerfile`
- Running and inspecting containers (`docker run`, `docker logs`, `docker exec`)
- Understanding layers, registries, and image tagging

### 💻 Command-Line Interface
- Navigating the filesystem, piping commands, using `grep` and `awk`
- Basic shell scripting (variables, loops, conditionals)
- SSH and file transfer basics

### 📄 YAML Fundamentals
- Scalars, sequences (lists), and mappings (dicts)
- Indentation rules and multi-line strings
- Understanding anchors and aliases (helpful but not required)

### ☁️ Cloud / Networking Basics (helpful)
- IP addressing, ports, DNS resolution
- Familiarity with a cloud provider console (AWS, GCP, or Azure)
- Basic understanding of load balancers and firewalls

> 💡 **New to Docker?** Complete the [Docker Getting Started guide](https://docs.docker.com/get-started/) before proceeding. Estimated time: 2–3 hours.

---

## 📁 Repository Structure

| File / Directory | Description | Est. Read Time |
|---|---|---|
| 📄 [`README.md`](README.md) | Course overview, objectives, navigation guide (this file) | 10 min |
| 📘 [`CONCEPTS.md`](CONCEPTS.md) | Deep-dive into all 17 Kubernetes concept areas with examples | 3–4 hrs |
| ⌨️ [`COMMANDS.md`](COMMANDS.md) | Complete `kubectl` cheat sheet — every command you need | 30 min |
| ⚡ [`QUICK-START.md`](QUICK-START.md) | Deploy your first app in 10 minutes | 10 min |
| 🔧 [`SETUP.md`](SETUP.md) | Environment setup for local (minikube/kind) and cloud clusters | 45 min |
| 🛠️ [`TOOLS.md`](TOOLS.md) | Essential ecosystem tools — Helm, k9s, Lens, Kustomize, Argo CD | 30 min |
| 📚 [`RESOURCES.md`](RESOURCES.md) | Curated reading list, courses, certifications (CKA/CKAD/CKS) | 20 min |
| 🔍 [`TROUBLESHOOTING.md`](TROUBLESHOOTING.md) | Debugging runbooks for the most common Kubernetes failures | 45 min |
| 🧪 [`LABS/`](LABS/) | 20+ step-by-step hands-on lab exercises | 8–12 hrs |
| 🏗️ [`PROJECTS/`](PROJECTS/) | 5 capstone projects (beginner → advanced) | 10–20 hrs |
| ✅ [`SOLUTIONS/`](SOLUTIONS/) | Reference solutions for labs and projects | — |
| 🙈 [`.gitignore`](.gitignore) | Standard ignores for kubeconfig, secrets, temp files | — |
| ⚖️ [`LICENSE`](LICENSE) | MIT License | — |
| 🤝 [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to contribute labs, fixes, and improvements | 10 min |

---

## 📚 Course Curriculum

### 🟢 Module 1 — Foundations (Days 1–2) ⏱ ~6 hrs
| Topic | Resource | Time |
|---|---|---|
| Kubernetes Architecture & Components | [CONCEPTS.md §1–2](CONCEPTS.md#1-container-orchestration-fundamentals) | 45 min |
| Cluster Setup (minikube / kind / cloud) | [SETUP.md](SETUP.md) | 45 min |
| Your First Pod & Deployment | [QUICK-START.md](QUICK-START.md) | 15 min |
| kubectl Essentials | [COMMANDS.md](COMMANDS.md) | 30 min |
| Lab 01: Explore a Running Cluster | [LABS/lab-01-explore/](LABS/lab-01-explore/) | 45 min |
| Lab 02: Deploy Nginx from Scratch | [LABS/lab-02-deploy-nginx/](LABS/lab-02-deploy-nginx/) | 45 min |

### 🔵 Module 2 — Core Workloads (Days 3–4) ⏱ ~6 hrs
| Topic | Resource | Time |
|---|---|---|
| Pods — single, multi-container, init | [CONCEPTS.md §3](CONCEPTS.md#3-pods) | 40 min |
| ReplicaSets & Deployments | [CONCEPTS.md §4–5](CONCEPTS.md#4-replicasets) | 45 min |
| Rolling Updates & Rollbacks | [CONCEPTS.md §5](CONCEPTS.md#5-deployments) | 30 min |
| StatefulSets & DaemonSets | [CONCEPTS.md §13](CONCEPTS.md#13-statefulsets-and-daemonsets) | 45 min |
| Jobs & CronJobs | [CONCEPTS.md §14](CONCEPTS.md#14-jobs-and-cronjobs) | 30 min |
| Lab 03: Rolling Update Strategy | [LABS/lab-03-rolling-update/](LABS/lab-03-rolling-update/) | 60 min |

### 🟡 Module 3 — Networking & Exposure (Days 5–6) ⏱ ~6 hrs
| Topic | Resource | Time |
|---|---|---|
| Services (ClusterIP, NodePort, LB) | [CONCEPTS.md §6](CONCEPTS.md#6-services) | 45 min |
| Ingress & Ingress Controllers | [CONCEPTS.md §12](CONCEPTS.md#12-ingress-and-networking) | 45 min |
| DNS & Service Discovery | [CONCEPTS.md §12](CONCEPTS.md#12-ingress-and-networking) | 30 min |
| NetworkPolicies | [CONCEPTS.md §12](CONCEPTS.md#12-ingress-and-networking) | 30 min |
| Lab 04: Expose a Multi-Service App | [LABS/lab-04-services/](LABS/lab-04-services/) | 60 min |
| Lab 05: Ingress with TLS | [LABS/lab-05-ingress-tls/](LABS/lab-05-ingress-tls/) | 60 min |

### 🟠 Module 4 — Configuration & Storage (Days 7–8) ⏱ ~6 hrs
| Topic | Resource | Time |
|---|---|---|
| ConfigMaps | [CONCEPTS.md §7](CONCEPTS.md#7-configmaps) | 30 min |
| Secrets | [CONCEPTS.md §8](CONCEPTS.md#8-secrets) | 30 min |
| Volumes & Persistent Storage | [CONCEPTS.md §9](CONCEPTS.md#9-volumes-and-persistent-storage) | 60 min |
| StorageClasses & Dynamic Provisioning | [CONCEPTS.md §9](CONCEPTS.md#9-volumes-and-persistent-storage) | 30 min |
| Lab 06: Stateful PostgreSQL Deployment | [LABS/lab-06-stateful-postgres/](LABS/lab-06-stateful-postgres/) | 90 min |

### 🔴 Module 5 — Security & RBAC (Days 9–10) ⏱ ~6 hrs
| Topic | Resource | Time |
|---|---|---|
| Namespaces & Resource Quotas | [CONCEPTS.md §10](CONCEPTS.md#10-namespaces-and-resource-quotas) | 30 min |
| Labels, Selectors & Annotations | [CONCEPTS.md §11](CONCEPTS.md#11-labels-selectors-and-annotations) | 30 min |
| RBAC — Roles, Bindings, ServiceAccounts | [CONCEPTS.md §15](CONCEPTS.md#15-rbac-and-security) | 60 min |
| Pod Security Standards | [CONCEPTS.md §15](CONCEPTS.md#15-rbac-and-security) | 30 min |
| Lab 07: Least-Privilege ServiceAccount | [LABS/lab-07-rbac/](LABS/lab-07-rbac/) | 60 min |

### 🟣 Module 6 — Advanced Operations (Days 11–14) ⏱ ~10 hrs
| Topic | Resource | Time |
|---|---|---|
| Helm Charts & Templating | [CONCEPTS.md §16](CONCEPTS.md#16-helm-and-package-management) | 90 min |
| Monitoring with Prometheus & Grafana | [CONCEPTS.md §17](CONCEPTS.md#17-monitoring-and-logging-patterns) | 60 min |
| Logging with EFK / Loki | [CONCEPTS.md §17](CONCEPTS.md#17-monitoring-and-logging-patterns) | 60 min |
| Capstone Project 1: Microservices App | [PROJECTS/project-01/](PROJECTS/project-01/) | 4 hrs |
| Capstone Project 2: GitOps with Argo CD | [PROJECTS/project-02/](PROJECTS/project-02/) | 4 hrs |

---

## 🗺️ Navigation Guide

### 📖 Self-Paced Learning Path

```
START HERE
    │
    ▼
[SETUP.md] ──── Set up your local cluster (minikube or kind)
    │
    ▼
[QUICK-START.md] ── Deploy your first app, see Kubernetes in action
    │
    ▼
[CONCEPTS.md] ──── Work through each concept section top-to-bottom
    │              (read the explanation, study the examples, try them)
    ▼
[LABS/] ────────── Complete labs in order (lab-01 through lab-20)
    │
    ▼
[PROJECTS/] ────── Pick a capstone project to consolidate your skills
    │
    ▼
[RESOURCES.md] ─── Study for CKA/CKAD certification
```

### 👩‍🏫 Instructor-Led Schedule (2-Week Bootcamp)

| Day | Morning (3 hrs) | Afternoon (3 hrs) |
|-----|-----------------|-------------------|
| 1 | Architecture overview + cluster setup | kubectl hands-on + Lab 01 |
| 2 | Pods deep-dive | ReplicaSets + Deployments + Lab 02 |
| 3 | Services + networking fundamentals | Ingress + Lab 03 |
| 4 | ConfigMaps + Secrets | Volumes + Lab 04 |
| 5 | StatefulSets + DaemonSets | Jobs + CronJobs + Lab 05 |
| 6 | Namespaces + RBAC | Pod Security + Lab 06 |
| 7 | Helm basics | Helm advanced + Lab 07 |
| 8 | Monitoring setup | Logging setup + Lab 08 |
| 9 | Troubleshooting workshop | TROUBLESHOOTING.md runbooks |
| 10 | Capstone project kickoff | Project work |

### ⚡ Quick Reference

| I want to… | Go to… |
|---|---|
| Set up my environment | [SETUP.md](SETUP.md) |
| Understand a K8s concept | [CONCEPTS.md](CONCEPTS.md) |
| Find a kubectl command | [COMMANDS.md](COMMANDS.md) |
| Do a hands-on exercise | [LABS/](LABS/) |
| Build a full project | [PROJECTS/](PROJECTS/) |
| Fix a broken cluster | [TROUBLESHOOTING.md](TROUBLESHOOTING.md) |
| Install a tool | [TOOLS.md](TOOLS.md) |
| Prepare for CKA/CKAD | [RESOURCES.md](RESOURCES.md) |

---

## ⏱️ Time Commitment Overview

| Pace | Total Hours | Recommended Schedule |
|------|-------------|----------------------|
| 🐢 Self-paced (casual) | 50–60 hrs | 2 hrs/day → ~4–5 weeks |
| 🚶 Self-paced (focused) | 50–60 hrs | 4 hrs/day → ~2 weeks |
| 🏃 Bootcamp (instructor-led) | 60 hrs | Full-time, 2 weeks |
| ⚡ Refresher (experienced) | 10–15 hrs | Focus on CONCEPTS.md + LABS |

---

## 🛠️ Tools You'll Use

| Tool | Purpose | Install |
|------|---------|---------|
| `kubectl` | Kubernetes CLI | [Install guide](https://kubernetes.io/docs/tasks/tools/) |
| `minikube` | Local single-node cluster | [Install guide](https://minikube.sigs.k8s.io/docs/start/) |
| `kind` | Local multi-node cluster via Docker | [Install guide](https://kind.sigs.k8s.io/docs/user/quick-start/) |
| `helm` | Package manager for Kubernetes | [Install guide](https://helm.sh/docs/intro/install/) |
| `k9s` | Terminal-based cluster UI | [Install guide](https://k9scli.io/topics/install/) |
| `Lens` | GUI cluster management | [Install guide](https://k8slens.dev/) |
| `kubectx/kubens` | Fast context/namespace switching | [Install guide](https://github.com/ahmetb/kubectx) |

---

## 🤝 Contributing

We welcome contributions! Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting a pull request. Areas where contributions are especially valuable:

- 🧪 New lab exercises
- 🐛 Bug fixes and corrections
- 🌐 Translations
- 📸 Diagrams and visual aids
- ☁️ Cloud-provider-specific setup guides

---

## 📜 License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgements

- The [Kubernetes project](https://kubernetes.io) and its incredible open-source community
- [CNCF](https://cncf.io) for stewarding the cloud-native ecosystem
- [Kelsey Hightower](https://github.com/kelseyhightower) for making Kubernetes accessible to everyone
- All contributors who have submitted labs, corrections, and improvements

---

<div align="center">

**Happy learning! 🎉 May your Pods always be Running and your Nodes never NotReady.**

[⬆ Back to Top](#-devops-kubernetes-bootcamp--hands-on-container-orchestration)

</div>
