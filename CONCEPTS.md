# 📘 Kubernetes Concepts — Complete Reference Guide

> A comprehensive reference covering all core Kubernetes concepts with practical, copy-paste-ready YAML examples.

---

## Table of Contents

1. [Container Orchestration Fundamentals](#1-container-orchestration-fundamentals)
2. [Kubernetes Architecture](#2-kubernetes-architecture)
3. [Pods](#3-pods)
4. [ReplicaSets](#4-replicasets)
5. [Deployments](#5-deployments)
6. [Services](#6-services)
7. [ConfigMaps](#7-configmaps)
8. [Secrets](#8-secrets)
9. [Volumes and Persistent Storage](#9-volumes-and-persistent-storage)
10. [Namespaces and Resource Quotas](#10-namespaces-and-resource-quotas)
11. [Labels, Selectors, and Annotations](#11-labels-selectors-and-annotations)
12. [Ingress and Networking](#12-ingress-and-networking)
13. [StatefulSets and DaemonSets](#13-statefulsets-and-daemonsets)
14. [Jobs and CronJobs](#14-jobs-and-cronjobs)
15. [RBAC and Security](#15-rbac-and-security)
16. [Helm and Package Management](#16-helm-and-package-management)
17. [Monitoring and Logging Patterns](#17-monitoring-and-logging-patterns)

---

## 1. Container Orchestration Fundamentals

**What it is:** Container orchestration is the automated management of containerized applications across a cluster of machines — handling deployment, scaling, networking, and self-healing without manual intervention.

**Why it matters in DevOps:** Manual container management breaks down beyond a handful of services. Orchestration platforms like Kubernetes let teams focus on writing applications rather than managing infrastructure, enabling reliable continuous delivery at scale.

### Example 1: Understanding the Problem — Manual vs. Orchestrated

```yaml
# Without orchestration: you manually decide where each container runs.
# If a node dies, containers are gone. Scale requires SSH to each machine.

# With Kubernetes: declare your desired state and K8s makes it reality.
# This Deployment declares "I want 3 replicas of my app always running":
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
    version: "1.0"
spec:
  replicas: 3               # K8s ensures 3 Pods always exist
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: nginx:1.25
          resources:
            requests:
              cpu: "100m"   # 0.1 CPU core guaranteed
              memory: "128Mi"
            limits:
              cpu: "500m"   # Cannot exceed 0.5 CPU core
              memory: "256Mi"
```

### Example 2: Self-Healing in Action

```bash
# Kubernetes continuously reconciles desired state with actual state.
# Delete a Pod manually — the ReplicaSet controller recreates it automatically:
kubectl delete pod my-app-abc12  # Pod is deleted

# Within seconds, a new Pod appears:
kubectl get pods -l app=my-app
# NAME             READY   STATUS    RESTARTS
# my-app-abc12     Terminating
# my-app-xyz99     Running   <-- new Pod, auto-created
```

### Example 3: Declarative vs. Imperative

```bash
# IMPERATIVE — tell Kubernetes what to do step by step:
kubectl run nginx --image=nginx:1.25 --port=80
kubectl scale deployment nginx --replicas=3
kubectl expose deployment nginx --type=ClusterIP --port=80

# DECLARATIVE — describe the desired end state in a file:
kubectl apply -f deployment.yaml   # K8s figures out how to get there
kubectl apply -f service.yaml      # Idempotent: safe to re-run

# Best practice: always prefer declarative (apply -f) in production.
# Store manifests in Git for version control and audit trails.
```

### Common Use Cases
- Auto-restarting crashed containers (self-healing)
- Bin-packing containers onto nodes to maximise resource utilisation
- Zero-downtime rolling deployments
- Horizontal autoscaling based on CPU/memory/custom metrics
- Multi-cloud and hybrid-cloud workload placement

### Best Practices
- Always set resource `requests` and `limits` on every container
- Use declarative manifests stored in version control (GitOps)
- Design applications to be stateless where possible
- Use health checks (`livenessProbe`, `readinessProbe`) on every container
- Separate concerns: one process per container

---

## 2. Kubernetes Architecture

**What it is:** Kubernetes runs as a distributed system with two types of nodes: the **control plane** (the cluster's brain) and **worker nodes** (where workloads run). The control plane components manage cluster state; worker node components run and network the containers.

**Why it matters in DevOps:** Understanding architecture helps you diagnose failures, plan capacity, secure the cluster, and make informed decisions about control plane sizing and high availability.

### Control Plane Components

| Component | Role |
|---|---|
| **kube-apiserver** | Front door — all kubectl commands hit this HTTPS REST API |
| **etcd** | Distributed key-value store — the single source of truth for all cluster state |
| **kube-scheduler** | Assigns unscheduled Pods to nodes based on resources and constraints |
| **kube-controller-manager** | Runs control loops: ReplicaSet, Deployment, Node, Job controllers, etc. |
| **cloud-controller-manager** | Integrates with cloud provider APIs (load balancers, storage, DNS) |

### Worker Node Components

| Component | Role |
|---|---|
| **kubelet** | Agent on every node — ensures containers in Pods are running and healthy |
| **kube-proxy** | Maintains iptables/IPVS rules to implement Service networking |
| **Container Runtime** | Runs containers — containerd (default), CRI-O, or Docker Engine |

### Example 1: Inspect Control Plane Components

```bash
# View control plane Pods in a kubeadm cluster:
kubectl get pods -n kube-system

# NAME                                    READY   STATUS
# etcd-master                             1/1     Running
# kube-apiserver-master                   1/1     Running
# kube-controller-manager-master          1/1     Running
# kube-scheduler-master                   1/1     Running
# coredns-xxx                             1/1     Running
# kube-proxy-xxx                          1/1     Running

# Check node status:
kubectl get nodes -o wide
```

### Example 2: Inspect Component Health

```bash
# Check API server and controller health:
kubectl get componentstatuses   # deprecated but still works in many versions

# Get detailed node information (kubelet version, OS, container runtime):
kubectl describe node <node-name>

# View kubelet logs on a node:
journalctl -u kubelet -f
```

### Example 3: etcd Backup (Critical for DR)

```bash
# Always back up etcd before cluster upgrades or major changes.
# Run on the control plane node where etcd is installed:
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify the snapshot:
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db --write-out=table
```

### Example 4: Node Maintenance — Cordon and Drain

```yaml
# Before rebooting a node for maintenance, gracefully evict its workloads:
# Step 1: Cordon (mark unschedulable — no new Pods land here)
# kubectl cordon <node-name>

# Step 2: Drain (evict existing Pods with grace period)
# kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data --grace-period=60

# Step 3: Perform maintenance, then uncordon:
# kubectl uncordon <node-name>
```

### Common Use Cases
- High-availability control plane (3 or 5 etcd nodes for production)
- Node auto-provisioning with Cluster Autoscaler
- Separating node pools by workload type (GPU nodes, spot instances)
- Multi-zone deployments for fault tolerance

### Best Practices
- Run etcd on SSDs with sufficient IOPS; back it up regularly
- Use an odd number of etcd members (3, 5) to maintain quorum
- Isolate control plane nodes — don't schedule workloads on them
- Enable audit logging on the API server
- Rotate certificates before they expire (use `kubeadm certs check-expiration`)

---

## 3. Pods

**What it is:** A Pod is the smallest deployable unit in Kubernetes — a wrapper around one or more containers that share the same network namespace, IP address, and mounted volumes.

**Why it matters in DevOps:** Every higher-level workload (Deployment, StatefulSet, Job) ultimately manages Pods. Understanding Pod spec is prerequisite knowledge for everything else.

### Example 1: Single-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: dev
spec:
  containers:
    - name: nginx
      image: nginx:1.25-alpine       # Use specific tags, never :latest in production
      ports:
        - containerPort: 80
          name: http
      resources:
        requests:
          cpu: "100m"
          memory: "64Mi"
        limits:
          cpu: "200m"
          memory: "128Mi"
      # Readiness probe: Pod only receives traffic when this passes
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
      # Liveness probe: Pod is restarted if this fails
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 15
        periodSeconds: 20
        failureThreshold: 3
```

### Example 2: Multi-Container Pod (Sidecar Pattern)

```yaml
# The sidecar pattern: a helper container runs alongside the main container
# sharing the same Pod network and volumes.
apiVersion: v1
kind: Pod
metadata:
  name: app-with-log-sidecar
spec:
  volumes:
    - name: shared-logs           # Shared volume both containers can access
      emptyDir: {}

  containers:
    # Primary application container
    - name: app
      image: my-app:1.0
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app

    # Sidecar: ships logs to a central logging system
    - name: log-shipper
      image: fluent/fluent-bit:2.1
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app
          readOnly: true            # Sidecar only reads; app writes
      resources:
        requests:
          cpu: "50m"
          memory: "32Mi"
        limits:
          cpu: "100m"
          memory: "64Mi"
```

### Example 3: Init Containers

```yaml
# Init containers run to completion BEFORE the main container starts.
# Use them to: wait for dependencies, seed databases, fetch configs, etc.
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
    # Init 1: Wait for the database to be ready
    - name: wait-for-db
      image: busybox:1.36
      command:
        - sh
        - -c
        - |
          until nc -z postgres-service 5432; do
            echo "Waiting for postgres..."; sleep 2
          done
          echo "Postgres is ready!"

    # Init 2: Run database migrations
    - name: run-migrations
      image: my-app:1.0
      command: ["python", "manage.py", "migrate"]
      env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url

  containers:
    - name: app
      image: my-app:1.0
      ports:
        - containerPort: 8000
```

### Example 4: Pod with Security Context

```yaml
# Security context restricts what a container can do on the node.
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true              # Pod must not run as root
    runAsUser: 1000                 # Run as UID 1000
    runAsGroup: 3000
    fsGroup: 2000                   # Volume files owned by GID 2000
    seccompProfile:
      type: RuntimeDefault          # Use the container runtime's default seccomp profile

  containers:
    - name: app
      image: my-app:1.0
      securityContext:
        allowPrivilegeEscalation: false   # Cannot gain more privileges
        readOnlyRootFilesystem: true      # Filesystem is read-only
        capabilities:
          drop:
            - ALL                   # Drop all Linux capabilities
          add:
            - NET_BIND_SERVICE      # Only add what's strictly needed
```

### Example 5: Pod Lifecycle and Restart Policies

```yaml
# restartPolicy controls what happens when containers exit:
#   Always (default) — always restart (good for long-running services)
#   OnFailure        — restart only on non-zero exit code (good for jobs)
#   Never            — never restart (good for one-shot tasks)

apiVersion: v1
kind: Pod
metadata:
  name: one-shot-task
spec:
  restartPolicy: OnFailure
  containers:
    - name: task
      image: busybox:1.36
      command: ["sh", "-c", "echo 'Task complete'; exit 0"]
```

### Common Use Cases
- Running a web server with a sidecar proxy (Envoy, Nginx)
- Database migration init containers
- Log shipping sidecars (Fluent Bit, Filebeat)
- Injecting configuration or secrets before app startup
- Ambassador pattern for external service proxying

### Best Practices
- Never deploy bare Pods in production — use Deployments instead
- Always set `readinessProbe` to prevent traffic to unready Pods
- Always set `livenessProbe` to auto-restart stuck Pods
- Set both `requests` and `limits` on every container
- Use `readOnlyRootFilesystem: true` and `runAsNonRoot: true`

---

## 4. ReplicaSets

**What it is:** A ReplicaSet ensures a specified number of identical Pod replicas are always running. If a Pod crashes or is deleted, the ReplicaSet controller creates a replacement.

**Why it matters in DevOps:** ReplicaSets provide the fundamental availability guarantee for stateless services. In practice you rarely create them directly — Deployments manage them — but understanding them clarifies how scaling and self-healing work.

### Example 1: Basic ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx
spec:
  replicas: 3                       # Maintain exactly 3 Pods at all times
  selector:
    matchLabels:
      app: nginx                    # ReplicaSet manages Pods with this label
  template:
    metadata:
      labels:
        app: nginx                  # MUST match spec.selector.matchLabels
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

### Example 2: Scaling a ReplicaSet

```bash
# Scale imperatively:
kubectl scale replicaset nginx-rs --replicas=5

# Scale declaratively (update replicas in YAML, then apply):
# Change replicas: 3  →  replicas: 5  in the manifest, then:
kubectl apply -f nginx-rs.yaml

# Watch Pods scale up in real time:
kubectl get pods -l app=nginx -w
```

### Example 3: Inspecting ReplicaSet Status

```bash
# Describe shows desired, current, ready replica counts and events:
kubectl describe replicaset nginx-rs

# Output includes:
# Replicas:    3 current / 3 desired
# Pods Status: 3 Running / 0 Waiting / 0 Succeeded / 0 Failed

# See which Pods a ReplicaSet owns:
kubectl get pods -l app=nginx --show-labels
```

### Common Use Cases
- Maintaining availability for stateless web services
- Basis for Deployment rolling update mechanism
- Pod-level horizontal scaling

### Best Practices
- Use Deployments instead of bare ReplicaSets for production workloads
- Ensure Pod template labels exactly match the selector
- Use `matchExpressions` for more flexible Pod selection when needed

---

## 5. Deployments

**What it is:** A Deployment manages a ReplicaSet and provides declarative updates — rolling deployments, rollbacks, and pause/resume. It is the standard way to run stateless applications in Kubernetes.

**Why it matters in DevOps:** Deployments are the backbone of continuous delivery. They enable zero-downtime releases by incrementally replacing old Pods with new ones, and allow instant rollbacks when something goes wrong.

### Example 1: Full Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
  labels:
    app: web-app
    team: platform
spec:
  replicas: 4
  selector:
    matchLabels:
      app: web-app

  # Rolling update strategy: controls how the transition happens
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Allow 1 extra Pod above desired count during update
      maxUnavailable: 0    # Never go below desired count (zero-downtime guarantee)

  # History limit: how many old ReplicaSets to keep for rollback
  revisionHistoryLimit: 5

  template:
    metadata:
      labels:
        app: web-app
        version: "2.0"
    spec:
      # Give Pods 30s to finish in-flight requests before being killed
      terminationGracePeriodSeconds: 30

      containers:
        - name: web-app
          image: my-registry/web-app:2.0
          ports:
            - containerPort: 8080
          env:
            - name: ENV
              value: production
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 3
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

### Example 2: Rolling Update and Rollback

```bash
# Trigger a rolling update by changing the container image:
kubectl set image deployment/web-app web-app=my-registry/web-app:2.1

# Or edit the manifest and apply:
kubectl apply -f web-app-deployment.yaml

# Watch the rollout progress:
kubectl rollout status deployment/web-app

# View rollout history (shows revision numbers):
kubectl rollout history deployment/web-app

# Inspect a specific revision:
kubectl rollout history deployment/web-app --revision=3

# ROLLBACK to the previous revision:
kubectl rollout undo deployment/web-app

# ROLLBACK to a specific revision:
kubectl rollout undo deployment/web-app --to-revision=2
```

### Example 3: Pause, Update, Resume (Canary-style)

```bash
# Pause the rollout so you can make multiple changes atomically:
kubectl rollout pause deployment/web-app

# Make changes (e.g., update image AND env vars):
kubectl set image deployment/web-app web-app=my-registry/web-app:2.2
kubectl set env deployment/web-app LOG_LEVEL=debug

# Resume the rollout — Kubernetes applies all changes together:
kubectl rollout resume deployment/web-app
```

### Example 4: Recreate Strategy (with downtime)

```yaml
# Use Recreate when your app cannot run two versions simultaneously
# (e.g., database schema migrations that break old versions).
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-migrator
spec:
  replicas: 2
  selector:
    matchLabels:
      app: db-migrator
  strategy:
    type: Recreate          # Kill ALL old Pods, then create new ones
  template:
    metadata:
      labels:
        app: db-migrator
    spec:
      containers:
        - name: app
          image: my-app:3.0
```

### Example 5: Horizontal Pod Autoscaler (HPA)

```yaml
# Automatically scale the Deployment based on CPU utilisation:
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70      # Scale up when avg CPU > 70%
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### Common Use Cases
- Stateless web services and APIs
- Zero-downtime application deployments
- A/B testing and canary releases
- Scheduled scale-ups for anticipated traffic spikes

### Best Practices
- Set `maxUnavailable: 0` and `maxSurge: 1` for zero-downtime
- Use `revisionHistoryLimit` to limit etcd bloat (5–10 is reasonable)
- Always set `readinessProbe` — only ready Pods receive traffic
- Pin image tags: never use `:latest` in Deployments
- Use `PodDisruptionBudget` to protect against voluntary disruptions

---

## 6. Services

**What it is:** A Service is a stable network endpoint that exposes a set of Pods. Pods are ephemeral and their IPs change; a Service provides a constant DNS name and IP (ClusterIP) that routes traffic to healthy Pods via label selectors.

**Why it matters in DevOps:** Services are the glue of microservices networking. Without them, every deployment would break client connections because Pod IPs change on every restart.

### Example 1: ClusterIP Service (Internal Only)

```yaml
# ClusterIP is the default — accessible only within the cluster.
# Other Pods connect via the DNS name: my-app-svc.default.svc.cluster.local
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: my-app           # Routes traffic to Pods with this label
  ports:
    - name: http
      protocol: TCP
      port: 80            # Service port (what clients connect to)
      targetPort: 8080    # Container port (what the app listens on)
```

### Example 2: NodePort Service (External via Node)

```yaml
# NodePort opens a port (30000-32767) on every cluster node.
# Access from outside: http://<any-node-ip>:30080
apiVersion: v1
kind: Service
metadata:
  name: my-app-nodeport
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80            # ClusterIP port
      targetPort: 8080    # Container port
      nodePort: 30080     # Node port (omit to auto-assign in range 30000-32767)
```

### Example 3: LoadBalancer Service (Cloud)

```yaml
# LoadBalancer provisions a cloud load balancer (AWS ELB, GCP LB, Azure LB).
# Gets an external IP automatically. Best for production exposure on cloud.
apiVersion: v1
kind: Service
metadata:
  name: my-app-lb
  annotations:
    # AWS-specific: create an NLB instead of a Classic LB
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 443
      targetPort: 8443
  # Pin the external IP (optional):
  # loadBalancerIP: "203.0.113.10"
```

### Example 4: ExternalName Service (External DNS Alias)

```yaml
# ExternalName maps a Service name to an external DNS name.
# Useful for pointing in-cluster apps at external managed databases
# without hardcoding hostnames inside Pods.
apiVersion: v1
kind: Service
metadata:
  name: prod-database
  namespace: backend
spec:
  type: ExternalName
  externalName: mydb.us-east-1.rds.amazonaws.com
  # Pods use: prod-database.backend.svc.cluster.local
  # which resolves to: mydb.us-east-1.rds.amazonaws.com
```

### Example 5: Headless Service (StatefulSet DNS)

```yaml
# Headless service (clusterIP: None) does NOT get a stable IP.
# Instead, DNS returns individual Pod IPs directly.
# Required by StatefulSets for stable per-Pod DNS names.
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None           # Makes it headless
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
# Each Pod gets a DNS entry: mysql-0.mysql-headless.default.svc.cluster.local
```

### Common Use Cases
- ClusterIP: internal microservice communication
- NodePort: dev/test environments, on-premises without a cloud LB
- LoadBalancer: production cloud-hosted services
- ExternalName: migrating from external services to in-cluster ones
- Headless: StatefulSets, direct Pod addressing

### Best Practices
- Name your Service ports (e.g., `name: http`) for clarity and Istio compatibility
- Use `sessionAffinity: ClientIP` when your app requires sticky sessions
- Prefer Ingress over many LoadBalancer Services to reduce cloud costs
- Add health checks at the LB level when using cloud LoadBalancer Services

---

## 7. ConfigMaps

**What it is:** A ConfigMap stores non-sensitive configuration data as key-value pairs, decoupling configuration from container images. ConfigMaps can be injected into Pods as environment variables, command-line arguments, or files in a volume.

**Why it matters in DevOps:** The Twelve-Factor App methodology requires strict separation of config from code. ConfigMaps implement this principle in Kubernetes, enabling the same image to run in dev, staging, and production with different configurations.

### Example 1: Create and Use ConfigMap as Env Vars

```yaml
# Define the ConfigMap:
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  APP_ENV: production
  LOG_LEVEL: info
  MAX_CONNECTIONS: "100"
  API_BASE_URL: "https://api.example.com"
---
# Consume the ConfigMap in a Pod:
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app
      image: my-app:1.0
      envFrom:
        - configMapRef:
            name: app-config    # Injects ALL keys as env vars
```

### Example 2: Selective Env Var Injection

```yaml
# Inject specific keys with custom environment variable names:
apiVersion: v1
kind: Pod
metadata:
  name: app-selective-env
spec:
  containers:
    - name: app
      image: my-app:1.0
      env:
        - name: ENVIRONMENT        # Custom env var name in container
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV         # Key from ConfigMap
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
```

### Example 3: ConfigMap as Volume (Config Files)

```yaml
# ConfigMap can hold multi-line config files (nginx.conf, app.properties, etc.)
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
      listen 80;
      server_name example.com;
      location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
      }
    }
  default.conf: |
    # Additional nginx config block
    gzip on;
    gzip_types text/plain application/json;
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  volumes:
    - name: nginx-config-vol
      configMap:
        name: nginx-config
  containers:
    - name: nginx
      image: nginx:1.25
      volumeMounts:
        - name: nginx-config-vol
          mountPath: /etc/nginx/conf.d   # Files appear here at runtime
          readOnly: true
```

### Example 4: ConfigMap for Command Arguments

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: job-args
data:
  batch-size: "500"
  dry-run: "false"
---
apiVersion: v1
kind: Pod
metadata:
  name: batch-job
spec:
  containers:
    - name: processor
      image: my-batch:1.0
      command: ["python", "process.py"]
      args:
        - "--batch-size=$(BATCH_SIZE)"
        - "--dry-run=$(DRY_RUN)"
      env:
        - name: BATCH_SIZE
          valueFrom:
            configMapKeyRef:
              name: job-args
              key: batch-size
        - name: DRY_RUN
          valueFrom:
            configMapKeyRef:
              name: job-args
              key: dry-run
```

### Common Use Cases
- Application configuration per environment (dev/staging/prod)
- Nginx, Prometheus, or application config files
- Feature flags and runtime parameters
- Command-line argument templating

### Best Practices
- Never store sensitive data in ConfigMaps — use Secrets instead
- Use `immutable: true` for ConfigMaps that should not change (protects against accidental edits and reduces API server load)
- Rolling-update Pods after ConfigMap changes: volume mounts auto-update but env vars do not
- Keep ConfigMaps small; for large files consider mounting from a PVC

---

## 8. Secrets

**What it is:** Secrets store sensitive data — passwords, tokens, TLS certificates, SSH keys — in base64-encoded form. Kubernetes stores them in etcd separately from ConfigMaps and provides access controls to limit which Pods and users can read them.

**Why it matters in DevOps:** Hard-coding credentials in images or ConfigMaps is a major security risk. Secrets enable credential management with RBAC controls, integration with external vault systems, and rotation without image rebuilds.

### Example 1: Opaque Secret (Username/Password)

```bash
# Create a secret imperatively (values are base64-encoded automatically):
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password='S3cur3P@ssw0rd!'

# Or from files (avoids credentials in shell history):
kubectl create secret generic db-credentials \
  --from-file=username=./username.txt \
  --from-file=password=./password.txt
```

```yaml
# Declarative (values must be manually base64-encoded):
# echo -n 'admin' | base64  →  YWRtaW4=
# echo -n 'S3cur3P@ssw0rd!' | base64  →  UzNjdXIzUEBzc3cwcmQh
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=
  password: UzNjdXIzUEBzc3cwcmQh
```

### Example 2: Consuming Secrets as Env Vars

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-secret
spec:
  containers:
    - name: app
      image: my-app:1.0
      env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
```

### Example 3: TLS Secret for HTTPS

```bash
# Generate a self-signed cert (or use cert-manager in production):
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=example.com/O=example"

# Create a TLS Secret from the cert and key files:
kubectl create secret tls example-tls \
  --cert=tls.crt \
  --key=tls.key
```

```yaml
# Reference the TLS Secret in an Ingress:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  tls:
    - hosts:
        - example.com
      secretName: example-tls      # The TLS Secret name
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-app-svc
                port:
                  number: 80
```

### Example 4: docker-registry Secret (Private Registry)

```bash
# Create a Secret to pull images from a private registry:
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=myuser@example.com
```

```yaml
# Reference the pull secret in a Pod spec:
apiVersion: v1
kind: Pod
metadata:
  name: private-app
spec:
  imagePullSecrets:
    - name: regcred           # K8s uses this to authenticate to the registry
  containers:
    - name: app
      image: registry.example.com/myapp:1.0
```

### Common Use Cases
- Database credentials and API keys
- TLS certificates for Ingress HTTPS termination
- Private container registry authentication
- SSH private keys for Git clones
- OAuth client secrets

### Best Practices
- Enable etcd encryption at rest to protect Secret data on disk
- Use external secret managers (HashiCorp Vault, AWS Secrets Manager) with the External Secrets Operator
- Restrict Secret access with RBAC: only the Pods that need a Secret should be able to read it
- Mount Secrets as volumes (files) rather than env vars where possible — env vars can be logged accidentally
- Set `immutable: true` on Secrets that should not change

---

## 9. Volumes and Persistent Storage

**What it is:** By default, a container's filesystem is ephemeral — data disappears when the container restarts. Volumes provide persistent (or shared) storage. Kubernetes supports many volume types from in-memory `emptyDir` to cloud block storage via PersistentVolumeClaims.

**Why it matters in DevOps:** Stateful applications (databases, message queues, file stores) require persistent storage that survives Pod restarts, reschedules, and node failures. Getting storage right is critical for data integrity and application reliability.

### Example 1: emptyDir (Shared In-Pod Scratch Space)

```yaml
# emptyDir is created when a Pod starts and deleted when it stops.
# Perfect for sharing data between containers in the same Pod.
apiVersion: v1
kind: Pod
metadata:
  name: shared-storage-pod
spec:
  volumes:
    - name: scratch          # Lives only as long as the Pod lives
      emptyDir:
        medium: ""           # "" = node disk; "Memory" = tmpfs (RAM)
        sizeLimit: 1Gi       # Optional: limit volume size

  containers:
    - name: writer
      image: busybox:1.36
      command: ["sh", "-c", "echo 'hello' > /data/message; sleep 3600"]
      volumeMounts:
        - name: scratch
          mountPath: /data

    - name: reader
      image: busybox:1.36
      command: ["sh", "-c", "cat /data/message; sleep 3600"]
      volumeMounts:
        - name: scratch
          mountPath: /data    # Both containers see the same directory
```

### Example 2: PersistentVolume and PersistentVolumeClaim

```yaml
# Step 1: Admin creates a PersistentVolume (cluster-level storage resource)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce           # Only one node can mount read-write at a time
  persistentVolumeReclaimPolicy: Retain   # Keep data after PVC is deleted
  storageClassName: standard
  hostPath:                   # For local dev only; use cloud volumes in production
    path: /data/postgres
---
# Step 2: Developer/workload requests storage via a PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 10Gi           # Request 10Gi; K8s binds to a matching PV
---
# Step 3: Reference the PVC in a Pod
apiVersion: v1
kind: Pod
metadata:
  name: postgres
spec:
  volumes:
    - name: postgres-data
      persistentVolumeClaim:
        claimName: postgres-pvc    # Bind to the PVC
  containers:
    - name: postgres
      image: postgres:15
      env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
      volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
```

### Example 3: StorageClass (Dynamic Provisioning)

```yaml
# StorageClass enables on-demand volume creation — no pre-provisioning needed.
# The cloud provider (AWS, GCP, Azure) creates the volume automatically.
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: ebs.csi.aws.com       # AWS EBS CSI driver
parameters:
  type: gp3                         # gp3 is faster and cheaper than gp2
  iops: "3000"
  throughput: "125"
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer  # Create volume in same AZ as Pod
reclaimPolicy: Delete              # Delete volume when PVC is deleted
allowVolumeExpansion: true         # Allow PVC to be expanded after creation
---
# A PVC that uses the StorageClass triggers dynamic provisioning:
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-data
spec:
  storageClassName: fast-ssd
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi                # Cloud creates a 50Gi EBS gp3 volume
```

### Example 4: hostPath Volume (Node Local — Dev Only)

```yaml
# hostPath mounts a directory from the node's filesystem into the Pod.
# WARNING: Creates tight coupling to a specific node. Dev/monitoring only.
apiVersion: v1
kind: Pod
metadata:
  name: node-log-reader
spec:
  volumes:
    - name: node-logs
      hostPath:
        path: /var/log/pods        # Path on the node
        type: DirectoryOrCreate    # Create if it doesn't exist
  containers:
    - name: log-reader
      image: busybox:1.36
      command: ["tail", "-f", "/host-logs/latest"]
      volumeMounts:
        - name: node-logs
          mountPath: /host-logs
          readOnly: true
```

### Common Use Cases
- Databases (PostgreSQL, MySQL, MongoDB) — ReadWriteOnce PVCs
- Shared file storage (NFS, EFS) — ReadWriteMany PVCs
- Caching and scratch space — emptyDir
- Log collection — hostPath with DaemonSets
- Configuration files — ConfigMap volumes

### Best Practices
- Use dynamic provisioning via StorageClasses — avoid manual PV creation
- Set `WaitForFirstConsumer` binding mode to co-locate volumes with Pods
- Use `ReadWriteOnce` for databases; `ReadWriteMany` only for shared file systems
- Set appropriate reclaim policies: `Retain` for databases, `Delete` for ephemeral data
- Test PVC expansion before relying on it in production

---

## 10. Namespaces and Resource Quotas

**What it is:** Namespaces partition a cluster into virtual sub-clusters, providing isolation between teams, environments, or applications. ResourceQuotas and LimitRanges enforce resource consumption limits per namespace.

**Why it matters in DevOps:** Multi-tenant clusters need guardrails. Namespaces prevent one team from starving another of CPU/memory and provide security boundaries for RBAC policies.

### Example 1: Create and Use Namespaces

```yaml
# Create namespaces for different environments or teams:
apiVersion: v1
kind: Namespace
metadata:
  name: team-alpha
  labels:
    team: alpha
    environment: production
---
apiVersion: v1
kind: Namespace
metadata:
  name: team-beta
  labels:
    team: beta
    environment: staging
```

```bash
# Work within a specific namespace:
kubectl get pods -n team-alpha

# Set default namespace for your session:
kubectl config set-context --current --namespace=team-alpha

# View resources across all namespaces:
kubectl get pods --all-namespaces
kubectl get pods -A   # shorthand
```

### Example 2: ResourceQuota

```yaml
# Limit total resource consumption in a namespace:
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-alpha-quota
  namespace: team-alpha
spec:
  hard:
    # Compute resources:
    requests.cpu: "10"          # Total CPU requests across all Pods
    requests.memory: 20Gi       # Total memory requests
    limits.cpu: "20"
    limits.memory: 40Gi
    # Object count limits:
    pods: "50"                  # Max 50 Pods in this namespace
    services: "20"
    persistentvolumeclaims: "10"
    secrets: "30"
    configmaps: "30"
```

### Example 3: LimitRange (Default Container Limits)

```yaml
# LimitRange sets default and maximum resource constraints per container/Pod.
# Prevents Pods from running without resource limits (which causes quota failures).
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team-alpha
spec:
  limits:
    - type: Container
      default:                 # Applied when container has no limits set
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:          # Applied when container has no requests set
        cpu: "100m"
        memory: "128Mi"
      max:                     # Hard upper bound per container
        cpu: "2"
        memory: "2Gi"
      min:                     # Hard lower bound
        cpu: "50m"
        memory: "32Mi"
    - type: PersistentVolumeClaim
      max:
        storage: 50Gi          # PVCs cannot exceed 50Gi in this namespace
```

### Common Use Cases
- Environment isolation (dev/staging/prod in the same cluster)
- Team isolation in a shared cluster
- Chargeback and cost attribution
- Security boundary enforcement with RBAC

### Best Practices
- Use distinct namespaces for each team and environment
- Always pair namespaces with ResourceQuotas and LimitRanges
- Use `kubens` for quick namespace switching during operations
- Apply NetworkPolicies at the namespace level for network isolation

---

## 11. Labels, Selectors, and Annotations

**What it is:** Labels are key-value pairs attached to objects for identification and grouping. Selectors query objects by labels. Annotations store non-identifying metadata (build info, tooling hints, monitoring config).

**Why it matters in DevOps:** Labels power Kubernetes' entire selection mechanism — Services route to Pods by label, Deployments manage Pods by label, monitoring systems discover targets by label. Consistent labelling is essential for operational clarity.

### Example 1: Standard Label Conventions

```yaml
# Follow the Kubernetes recommended label schema:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments-service
  labels:
    # Recommended standard labels:
    app.kubernetes.io/name: payments-service
    app.kubernetes.io/instance: payments-prod
    app.kubernetes.io/version: "3.1.2"
    app.kubernetes.io/component: backend
    app.kubernetes.io/part-of: ecommerce-platform
    app.kubernetes.io/managed-by: helm
    # Custom labels:
    team: payments
    environment: production
    tier: api
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: payments-service
      environment: production
  template:
    metadata:
      labels:
        app.kubernetes.io/name: payments-service
        environment: production
    spec:
      containers:
        - name: payments
          image: payments:3.1.2
```

### Example 2: Label Selectors in kubectl

```bash
# Filter resources by equality:
kubectl get pods -l app=nginx
kubectl get pods -l environment=production,tier=frontend

# Filter using set-based selectors:
kubectl get pods -l 'environment in (production, staging)'
kubectl get pods -l 'tier notin (test)'
kubectl get pods -l '!debug'            # Does NOT have the 'debug' label

# Add or remove labels from live objects:
kubectl label pod my-pod version=2.0
kubectl label pod my-pod version-         # Remove the 'version' label
```

### Example 3: matchExpressions for Complex Selection

```yaml
# matchExpressions allows AND/OR/NOT logic in selectors:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-front
spec:
  selector:
    matchLabels:
      app: web
    matchExpressions:
      - key: environment
        operator: In
        values: [production, staging]
      - key: debug
        operator: DoesNotExist    # Only select Pods without a 'debug' label
  template:
    metadata:
      labels:
        app: web
        environment: production
    spec:
      containers:
        - name: web
          image: web:1.0
```

### Example 4: Annotations

```yaml
# Annotations store metadata that tools and operators read.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
  annotations:
    # Build and CI/CD metadata:
    build.ci/pipeline-url: "https://ci.example.com/jobs/123"
    build.ci/commit-sha: "abc1234def5678"
    build.ci/build-date: "2024-01-15T10:30:00Z"
    # Monitoring hints for Prometheus:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
    prometheus.io/path: "/metrics"
    # Contact information:
    "contact/team": "platform@example.com"
    "contact/slack": "#platform-alerts"
    # Kubernetes tooling hints:
    kubectl.kubernetes.io/last-applied-configuration: ""  # Set by kubectl apply
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-service
  template:
    metadata:
      labels:
        app: api-service
    spec:
      containers:
        - name: api
          image: api:2.0
```

### Common Use Cases
- Service-to-Pod routing via label selectors
- Prometheus service discovery via annotations
- Environment and team attribution for cost tracking
- Node affinity/anti-affinity scheduling rules

### Best Practices
- Adopt and enforce a consistent label schema across your organisation
- Use `app.kubernetes.io/*` labels for interoperability with tools like Helm and Argo CD
- Use annotations for metadata that tools consume — not for filtering
- Label nodes with hardware type, region, and AZ for affinity rules

---

## 12. Ingress and Networking

**What it is:** Ingress is an API object that manages external HTTP/HTTPS access to services, providing host/path-based routing, TLS termination, and virtual hosting — all through a single load balancer. NetworkPolicies control traffic between Pods.

**Why it matters in DevOps:** Ingress consolidates external access through one point, enabling TLS, authentication, and routing without a LoadBalancer Service per microservice (reducing cloud costs dramatically).

### Example 1: Basic Ingress with Path Routing

```yaml
# Prerequisites: An Ingress Controller must be installed (nginx-ingress, Traefik, etc.)
# Install nginx-ingress: helm install ingress-nginx ingress-nginx/ingress-nginx

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  ingressClassName: nginx
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /users
            pathType: Prefix
            backend:
              service:
                name: users-svc
                port:
                  number: 80
          - path: /orders
            pathType: Prefix
            backend:
              service:
                name: orders-svc
                port:
                  number: 80
    - host: admin.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: admin-svc
                port:
                  number: 80
```

### Example 2: Ingress with TLS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    # cert-manager automatically provisions and renews TLS certificates
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - example.com
        - www.example.com
      secretName: example-tls-cert    # cert-manager populates this Secret
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-svc
                port:
                  number: 80
```

### Example 3: NetworkPolicy — Default Deny All

```yaml
# Best practice: start with deny-all, then explicitly allow required traffic.
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}              # Applies to ALL Pods in the namespace
  policyTypes:
    - Ingress
    - Egress
---
# Allow specific traffic: frontend → backend on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend              # This policy applies to backend Pods
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend     # Only allow traffic from frontend Pods
      ports:
        - protocol: TCP
          port: 8080
```

### Example 4: DNS and Service Discovery

```bash
# Kubernetes DNS format:
# <service-name>.<namespace>.svc.cluster.local

# Within same namespace — short name works:
curl http://payments-svc/api/payments

# Cross-namespace — include namespace:
curl http://payments-svc.payments-ns.svc.cluster.local/api/payments

# Verify DNS resolution inside a Pod:
kubectl run dns-test --image=busybox:1.36 --rm -it --restart=Never -- \
  nslookup payments-svc.payments-ns.svc.cluster.local
```

### Common Use Cases
- HTTPS termination for web applications
- Path-based routing across microservices
- Rate limiting and authentication at the edge (via Ingress annotations)
- Namespace network isolation with NetworkPolicies
- Zero-trust networking between services

### Best Practices
- Use cert-manager with Let's Encrypt for automated TLS certificate management
- Implement NetworkPolicies as default-deny with explicit allow rules
- Use Ingress class annotations to select the correct Ingress controller
- Monitor Ingress controller metrics (request rate, latency, error rate)

---

## 13. StatefulSets and DaemonSets

**What it is:** **StatefulSets** manage stateful applications (databases, queues) where each Pod needs a stable identity, predictable DNS name, and persistent storage. **DaemonSets** ensure one Pod runs on every node (or a subset), ideal for infrastructure agents.

**Why it matters in DevOps:** Not all workloads are stateless. StatefulSets safely run distributed databases and message brokers. DaemonSets are the standard way to deploy log shippers, monitoring agents, and CNI plugins cluster-wide.

### Example 1: StatefulSet for PostgreSQL

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-headless    # Must match headless Service name
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
          volumeMounts:
            - name: data                  # References volumeClaimTemplate below
              mountPath: /var/lib/postgresql/data
  # volumeClaimTemplates creates a unique PVC for EACH Pod replica:
  # postgres-data-postgres-0, postgres-data-postgres-1, etc.
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: [ReadWriteOnce]
        storageClassName: fast-ssd
        resources:
          requests:
            storage: 50Gi
---
# Headless service for stable DNS names:
# postgres-0.postgres-headless.default.svc.cluster.local
# postgres-1.postgres-headless.default.svc.cluster.local
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
    - port: 5432
```

### Example 2: DaemonSet for Log Collection

```yaml
# Run Fluent Bit on every node to collect and forward container logs:
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
spec:
  selector:
    matchLabels:
      app: fluent-bit
  template:
    metadata:
      labels:
        app: fluent-bit
    spec:
      # DaemonSets often need to run on control plane nodes too:
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          operator: Exists
          effect: NoSchedule
      containers:
        - name: fluent-bit
          image: fluent/fluent-bit:2.1
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "200m"
              memory: "256Mi"
          volumeMounts:
            - name: varlog
              mountPath: /var/log
              readOnly: true
            - name: containers
              mountPath: /var/lib/docker/containers
              readOnly: true
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: containers
          hostPath:
            path: /var/lib/docker/containers
```

### Example 3: StatefulSet Ordered Operations

```bash
# StatefulSets scale and roll out in order: 0, 1, 2, ...
# and scale down in reverse: 2, 1, 0 (critical for distributed databases)

# Scale up (pods are created sequentially):
kubectl scale statefulset postgres --replicas=5
# Creates: postgres-3, then postgres-4

# Scale down:
kubectl scale statefulset postgres --replicas=2
# Deletes: postgres-4, then postgres-3, then postgres-2

# Rolling update (ordered, one at a time):
kubectl rollout status statefulset/postgres
```

### Common Use Cases
- StatefulSets: PostgreSQL, MySQL, Cassandra, Kafka, Elasticsearch, ZooKeeper
- DaemonSets: Log shippers (Fluent Bit, Filebeat), monitoring agents (Prometheus Node Exporter), CNI plugins (Calico, Cilium), security scanners

### Best Practices
- StatefulSets require a headless Service for Pod DNS stability
- Use `podManagementPolicy: Parallel` only when ordering doesn't matter
- DaemonSets must tolerate `NoSchedule` taints to run on tainted nodes
- Use `updateStrategy: type: RollingUpdate` on DaemonSets for zero-downtime updates

---

## 14. Jobs and CronJobs

**What it is:** A **Job** runs one or more Pods to completion — when all Pods succeed, the Job is done. A **CronJob** creates Jobs on a schedule (like Unix cron). Unlike Deployments, Jobs are designed for finite, batch workloads.

**Why it matters in DevOps:** Database migrations, report generation, data cleanup, backup tasks, and CI pipeline steps all fit the Job model. CronJobs replace traditional cron daemons with cluster-native scheduling, retries, and history.

### Example 1: Simple Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  completions: 1              # Must complete successfully once
  parallelism: 1              # Run one Pod at a time
  backoffLimit: 3             # Retry up to 3 times on failure
  activeDeadlineSeconds: 300  # Fail the Job after 5 minutes regardless
  template:
    spec:
      restartPolicy: OnFailure
      containers:
        - name: migration
          image: my-app:2.0
          command: ["python", "manage.py", "migrate", "--no-input"]
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: url
```

### Example 2: Parallel Job (Worker Pool)

```yaml
# Process a queue with multiple parallel workers.
# Each Pod picks a work item; Job completes when 10 items are processed.
apiVersion: batch/v1
kind: Job
metadata:
  name: image-processor
spec:
  completions: 10             # Total work items
  parallelism: 3              # Process 3 at a time
  completionMode: Indexed     # Each Pod gets a unique index (0-9)
  template:
    spec:
      restartPolicy: OnFailure
      containers:
        - name: processor
          image: image-processor:1.0
          env:
            - name: JOB_INDEX
              valueFrom:
                fieldRef:
                  fieldPath: metadata.annotations['batch.kubernetes.io/job-completion-index']
```

### Example 3: CronJob for Scheduled Tasks

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
spec:
  schedule: "0 2 * * *"          # Every day at 2:00 AM UTC (standard cron syntax)
  timeZone: "UTC"                 # Explicit timezone (K8s 1.27+)
  concurrencyPolicy: Forbid       # Don't start new Job if previous is still running
  successfulJobsHistoryLimit: 3   # Keep last 3 successful Jobs
  failedJobsHistoryLimit: 1       # Keep last 1 failed Job (for debugging)
  startingDeadlineSeconds: 300    # Skip if not started within 5 min of schedule
  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: postgres:15
              command:
                - sh
                - -c
                - |
                  pg_dump -Fc $DATABASE_URL > /backup/$(date +%Y%m%d-%H%M%S).dump
              env:
                - name: DATABASE_URL
                  valueFrom:
                    secretKeyRef:
                      name: db-secret
                      key: url
              volumeMounts:
                - name: backup-storage
                  mountPath: /backup
          volumes:
            - name: backup-storage
              persistentVolumeClaim:
                claimName: backup-pvc
```

### Example 4: Monitor and Clean Up Jobs

```bash
# List all Jobs and their status:
kubectl get jobs
# NAME            COMPLETIONS   DURATION   AGE
# db-migration    1/1           12s        5m

# View Job logs:
kubectl logs job/db-migration

# Manually trigger a CronJob immediately (for testing):
kubectl create job --from=cronjob/database-backup manual-backup-$(date +%s)

# Delete completed Jobs (free up API server object storage):
kubectl delete jobs --field-selector status.successful=1
```

### Common Use Cases
- Database schema migrations before application deployments
- Periodic data aggregation and report generation
- Scheduled log/data archival and cleanup
- CI pipeline steps (test runners, build jobs)
- One-time cluster bootstrapping tasks

### Best Practices
- Always set `activeDeadlineSeconds` to prevent runaway Jobs
- Set `backoffLimit` appropriately — some jobs should fail fast
- Use `concurrencyPolicy: Forbid` for CronJobs that cannot overlap
- Clean up old Jobs to prevent etcd bloat (`ttlSecondsAfterFinished`)
- Use `restartPolicy: OnFailure` (not `Always`) for Job Pods

---

## 15. RBAC and Security

**What it is:** Role-Based Access Control (RBAC) restricts what users and workloads can do in Kubernetes. Roles and ClusterRoles define permissions; RoleBindings and ClusterRoleBindings grant them to subjects (users, groups, ServiceAccounts).

**Why it matters in DevOps:** The principle of least privilege is a security cornerstone. RBAC ensures that a compromised Pod can only access what it needs — not the entire cluster. It also gates human access appropriately per team or role.

### Example 1: ServiceAccount with Least-Privilege Role

```yaml
# Create a ServiceAccount for a specific application:
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: production
---
# Define what the ServiceAccount is allowed to do (namespace-scoped):
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: production
rules:
  - apiGroups: [""]                 # "" = core API group
    resources: ["pods", "pods/log"] # Can read pods and their logs
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list"]          # Can read ConfigMaps
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["app-db-secret"] # Can ONLY read this specific Secret
    verbs: ["get"]
---
# Bind the Role to the ServiceAccount:
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-role-binding
  namespace: production
subjects:
  - kind: ServiceAccount
    name: app-sa
    namespace: production
roleRef:
  kind: Role
  apiVersion: rbac.authorization.k8s.io/v1
  name: app-role
```

### Example 2: ClusterRole for Cluster-Wide Read Access

```yaml
# ClusterRoles apply cluster-wide (non-namespaced resources or all namespaces)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
  - apiGroups: [""]
    resources: ["nodes", "namespaces", "persistentvolumes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "statefulsets", "daemonsets"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["metrics.k8s.io"]
    resources: ["nodes", "pods"]
    verbs: ["get", "list"]
---
# Bind to a specific user (e.g., an SRE team member):
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: sre-cluster-reader
subjects:
  - kind: User
    name: jane.doe@example.com    # As identified by your auth provider
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

### Example 3: Assign ServiceAccount to a Pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-service
  template:
    metadata:
      labels:
        app: api-service
    spec:
      serviceAccountName: app-sa       # Use the specific SA, not default
      automountServiceAccountToken: true  # Mount the token (set false if unused)
      containers:
        - name: api
          image: api-service:2.0
```

### Example 4: Verify Permissions with auth can-i

```bash
# Check what a user can do:
kubectl auth can-i create pods --namespace=production
kubectl auth can-i delete secrets --namespace=production --as=jane.doe@example.com

# Check ServiceAccount permissions:
kubectl auth can-i list deployments \
  --as=system:serviceaccount:production:app-sa \
  --namespace=production

# List all rules for a Role:
kubectl describe role app-role -n production

# Audit who has permission to do something:
kubectl get rolebindings,clusterrolebindings -A \
  -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{range .subjects[*]}{.kind}/{.name}{"\n"}{end}{end}'
```

### Example 5: Pod Security — Restricted Profile

```yaml
# Apply a Pod Security Standard at the namespace level.
# "restricted" is the most locked-down profile.
apiVersion: v1
kind: Namespace
metadata:
  name: secure-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted    # Block violating Pods
    pod-security.kubernetes.io/audit: restricted      # Audit violations
    pod-security.kubernetes.io/warn: restricted       # Warn on violations
```

### Common Use Cases
- Scoping CI/CD system access to specific namespaces
- Granting read-only cluster monitoring access to SRE teams
- Restricting application Pods to only read their own Secrets
- Namespace-admin delegation to development teams

### Best Practices
- Set `automountServiceAccountToken: false` for Pods that don't call the K8s API
- Never use `cluster-admin` for application ServiceAccounts
- Audit RBAC bindings regularly with tools like `rbac-lookup` or `rakkess`
- Enable audit logging to capture API server interactions
- Prefer RoleBindings over ClusterRoleBindings to limit blast radius

---

## 16. Helm and Package Management

**What it is:** Helm is the package manager for Kubernetes. A **Chart** is a collection of YAML templates with values files, enabling reusable, parameterisable application packaging. Helm manages the full lifecycle: install, upgrade, rollback, and uninstall.

**Why it matters in DevOps:** Manually maintaining YAML for multiple environments is error-prone and repetitive. Helm templates enable DRY configuration, semantic versioning for deployments, and reliable upgrades/rollbacks across environments.

### Example 1: Essential Helm Commands

```bash
# Add and update chart repositories:
helm repo add stable https://charts.helm.sh/stable
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Search for charts:
helm search repo nginx
helm search hub wordpress       # Search Artifact Hub

# Install a chart:
helm install my-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.replicaCount=2

# Install with a values file:
helm install my-app ./my-chart -f values-production.yaml

# List installed releases:
helm list -A

# Upgrade a release:
helm upgrade my-nginx ingress-nginx/ingress-nginx --reuse-values \
  --set controller.replicaCount=3

# Rollback to previous revision:
helm rollback my-nginx 1

# Uninstall:
helm uninstall my-nginx -n ingress-nginx
```

### Example 2: Chart Directory Structure

```
my-app-chart/
├── Chart.yaml          # Chart metadata (name, version, description)
├── values.yaml         # Default values (overridden per environment)
├── values-dev.yaml     # Dev-specific overrides
├── values-prod.yaml    # Production-specific overrides
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl    # Named template helpers (reusable snippets)
│   └── NOTES.txt       # Shown after install
└── charts/             # Bundled sub-chart dependencies
```

### Example 3: Helm Chart Deployment Template

```yaml
# templates/deployment.yaml — parameterised with Helm values:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          ports:
            - containerPort: {{ .Values.service.targetPort }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- if .Values.env }}
          env:
            {{- range .Values.env }}
            - name: {{ .name }}
              value: {{ .value | quote }}
            {{- end }}
          {{- end }}
```

### Example 4: values.yaml and Environment Override

```yaml
# values.yaml (defaults):
replicaCount: 2
image:
  repository: my-registry/my-app
  tag: ""               # Defaults to Chart.AppVersion if empty
service:
  type: ClusterIP
  port: 80
  targetPort: 8080
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
env:
  - name: LOG_LEVEL
    value: info

---
# values-production.yaml (override for production):
replicaCount: 6
image:
  tag: "3.2.1"          # Pin exact image tag in production
service:
  type: LoadBalancer
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "2"
    memory: "2Gi"
env:
  - name: LOG_LEVEL
    value: warn
```

### Common Use Cases
- Packaging and versioning Kubernetes applications
- Multi-environment deployments with shared templates
- Installing third-party infrastructure (Prometheus, Cert-Manager, Nginx Ingress)
- GitOps pipelines using Helm releases

### Best Practices
- Always use `helm upgrade --install` in CI/CD (idempotent: installs on first run, upgrades on subsequent)
- Pin chart versions in production: `helm install x chart@1.2.3`
- Use `helm template` to preview rendered manifests before applying
- Store `values.yaml` per environment in Git (not passwords — use Helm Secrets or ESO)
- Use Helmfile or Argo CD App of Apps for managing multiple charts

---

## 17. Monitoring and Logging Patterns

**What it is:** Observability in Kubernetes comprises three pillars: **metrics** (numeric measurements over time), **logs** (timestamped event records), and **traces** (request flow across services). The standard stack is Prometheus + Grafana for metrics and EFK/Loki for logs.

**Why it matters in DevOps:** You cannot run production systems blind. Observability enables proactive alerting, rapid incident response, capacity planning, and SLO/SLA compliance. Kubernetes makes observability both more necessary (distributed systems fail in complex ways) and more powerful (rich metadata via labels).

### Example 1: Install kube-prometheus-stack with Helm

```bash
# Deploy Prometheus, Grafana, Alertmanager, and Node Exporter in one chart:
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install kube-prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=changeme \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=50Gi

# Access Grafana:
kubectl port-forward -n monitoring svc/kube-prometheus-grafana 3000:80
# Open http://localhost:3000 (admin/changeme)
```

### Example 2: Prometheus ServiceMonitor (Application Metrics Scraping)

```yaml
# Tell Prometheus to scrape your application's /metrics endpoint.
# Requires the Prometheus Operator (included in kube-prometheus-stack).
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-monitor
  namespace: monitoring
  labels:
    release: kube-prometheus    # Must match Prometheus selector
spec:
  selector:
    matchLabels:
      app: my-app               # Match the Service that exposes /metrics
  namespaceSelector:
    matchNames:
      - production
  endpoints:
    - port: metrics             # Named port on the Service
      path: /metrics
      interval: 30s             # Scrape every 30 seconds
      scrapeTimeout: 10s
```

### Example 3: PrometheusRule — Custom Alert

```yaml
# Define custom alerting rules evaluated by Prometheus:
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: app-alerts
  namespace: monitoring
  labels:
    release: kube-prometheus
spec:
  groups:
    - name: application.rules
      interval: 30s
      rules:
        # Alert if error rate exceeds 1% for 5 minutes:
        - alert: HighErrorRate
          expr: |
            sum(rate(http_requests_total{status=~"5.."}[5m])) /
            sum(rate(http_requests_total[5m])) > 0.01
          for: 5m
          labels:
            severity: critical
            team: platform
          annotations:
            summary: "High error rate detected"
            description: "Error rate is {{ $value | humanizePercentage }} for {{ $labels.service }}"

        # Alert if Pod CPU exceeds 80% of limit for 10 minutes:
        - alert: PodHighCPU
          expr: |
            (sum by (pod, namespace) (rate(container_cpu_usage_seconds_total{container!=""}[5m])) /
            sum by (pod, namespace) (kube_pod_container_resource_limits{resource="cpu"})) > 0.8
          for: 10m
          labels:
            severity: warning
          annotations:
            summary: "Pod {{ $labels.pod }} CPU usage is high"
```

### Example 4: Loki and Promtail for Log Aggregation

```bash
# Install Loki + Promtail (lightweight, Prometheus-native log aggregation):
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install loki grafana/loki-stack \
  --namespace logging \
  --create-namespace \
  --set grafana.enabled=false \      # Use existing Grafana from kube-prometheus
  --set prometheus.enabled=false \
  --set loki.persistence.enabled=true \
  --set loki.persistence.size=20Gi
```

```yaml
# Promtail DaemonSet config snippet (already included in loki-stack):
# Automatically collects logs from all containers via /var/log/pods
# and forwards to Loki with Kubernetes metadata labels.
# In Grafana, add Loki as a data source:
# URL: http://loki.logging.svc.cluster.local:3100
```

### Example 5: Application Logging Best Practices

```yaml
# Configure your application Pod for structured logging:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: structured-logger
spec:
  replicas: 2
  selector:
    matchLabels:
      app: structured-logger
  template:
    metadata:
      labels:
        app: structured-logger
      annotations:
        # Promtail pipeline hints:
        logging.grafana.com/format: logfmt   # or "json"
    spec:
      containers:
        - name: app
          image: my-app:1.0
          env:
            # Force JSON logs for better parsing in Loki:
            - name: LOG_FORMAT
              value: json
            - name: LOG_LEVEL
              value: info
          # Logs go to stdout/stderr — kubelet captures them automatically:
          # Never write logs to files inside containers in Kubernetes.
```

### Common Use Cases
- CPU/memory dashboards and alerts (Grafana + Prometheus)
- Application error rate and latency SLOs (custom metrics)
- Centralised log search and tailing (Loki / EFK)
- Distributed tracing for microservices (Jaeger / Tempo)
- Cost monitoring and chargeback per namespace/team

### Best Practices
- Write all logs to stdout/stderr — never to files inside containers
- Emit structured (JSON) logs for machine-parseable ingestion
- Instrument applications with Prometheus client libraries for custom metrics
- Define SLOs as PrometheusRules with alerting to Alertmanager
- Set appropriate retention periods: metrics (30–90 days), logs (7–30 days)
- Use Grafana dashboards from grafana.com/grafana/dashboards as starting points (e.g., dashboard ID 15760 for Kubernetes cluster overview)

---

## 📎 Quick Reference

```bash
# ─── Context and Namespace ───────────────────────────────
kubectl config get-contexts
kubectl config use-context <context>
kubectl config set-context --current --namespace=<ns>

# ─── Get Resources ───────────────────────────────────────
kubectl get all -n <namespace>
kubectl get pods -A                          # All namespaces
kubectl get pods -o wide                     # Include node and IP
kubectl get pods -o yaml                     # Full YAML output

# ─── Describe and Debug ──────────────────────────────────
kubectl describe pod <pod>
kubectl logs <pod> -c <container> -f         # Follow logs
kubectl exec -it <pod> -- /bin/sh            # Shell into container
kubectl top pods / kubectl top nodes         # Resource usage

# ─── Apply and Delete ────────────────────────────────────
kubectl apply -f <file.yaml>
kubectl apply -f <directory>/
kubectl delete -f <file.yaml>
kubectl delete pod <pod> --grace-period=0    # Force delete

# ─── Deployments ─────────────────────────────────────────
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl scale deployment/<name> --replicas=5

# ─── Port Forward ────────────────────────────────────────
kubectl port-forward pod/<pod> 8080:80
kubectl port-forward svc/<service> 8080:80

# ─── Dry-run and Diff ────────────────────────────────────
kubectl apply -f <file.yaml> --dry-run=client
kubectl diff -f <file.yaml>
```

---

*This document is part of the [DevOps Kubernetes Bootcamp](README.md). See also: [COMMANDS.md](COMMANDS.md) for a full kubectl cheat sheet, and [LABS/](LABS/) for hands-on exercises.*
