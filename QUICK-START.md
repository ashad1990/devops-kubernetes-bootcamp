# Kubernetes Quick Start Guide

**Zero to a running app in 10 minutes.**

This guide walks you through deploying, exposing, scaling, updating, and cleaning up
a real application on Kubernetes — step by step, with every command and expected output.

---

## Table of Contents

1. [Prerequisites Check](#step-1-prerequisites-check)
2. [Start Minikube](#step-2-start-minikube)
3. [Deploy Your First App](#step-3-deploy-your-first-app)
4. [Expose With a Service](#step-4-expose-with-a-service)
5. [Access the App](#step-5-access-the-app)
6. [Scale the Deployment](#step-6-scale-the-deployment)
7. [Update the App](#step-7-update-the-app)
8. [View Logs](#step-8-view-logs)
9. [Inspect Resources](#step-9-inspect-resources)
10. [Clean Up](#step-10-clean-up)

---

## Step 1: Prerequisites Check

Before you start, verify that both `kubectl` and `minikube` are installed and available in your `PATH`.

### Check kubectl

```bash
kubectl version --client --short
```

**Expected Output:**
```
Client Version: v1.28.3
```

If you see `command not found`, install kubectl by following the
[official installation guide](https://kubernetes.io/docs/tasks/tools/).

### Check minikube

```bash
minikube version
```

**Expected Output:**
```
minikube version: v1.31.2
commit: fd7ecd9c4599bef9f04c0986c4a0187f98a4396e
```

If minikube is not installed, follow the
[minikube installation guide](https://minikube.sigs.k8s.io/docs/start/).

### Why this matters

`kubectl` is the command-line tool that communicates with the Kubernetes API server.
`minikube` runs a single-node Kubernetes cluster inside a virtual machine or container
on your local machine — perfect for learning and development without needing a cloud provider.

---

## Step 2: Start Minikube

Start your local Kubernetes cluster with minikube.

### Start the cluster

```bash
minikube start
```

**Expected Output:**
```
😄  minikube v1.31.2 on Linux (amd64)
✨  Automatically selected the docker driver
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.28.2 on Docker 24.0.5 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

> **Tip:** On first run, minikube downloads a base image (~300 MB). This may take a few minutes.
> Subsequent starts are much faster.

### Verify the cluster is running

```bash
kubectl cluster-info
```

**Expected Output:**
```
Kubernetes control plane is running at https://127.0.0.1:49234
CoreDNS is running at https://127.0.0.1:49234/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

### Check the node status

```bash
kubectl get nodes
```

**Expected Output:**
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   1m    v1.28.2
```

The node status must be `Ready` before proceeding. If it shows `NotReady`, wait 30 seconds
and try again — the cluster may still be initializing.

### Why this matters

`minikube start` provisions a full Kubernetes cluster locally. The `kubectl cluster-info`
command confirms that `kubectl` can reach the API server and that the cluster is healthy.
`kubectl get nodes` shows all machines (nodes) that make up the cluster.

---

## Step 3: Deploy Your First App

We'll deploy nginx — a popular web server — using a Kubernetes **Deployment**.
A Deployment is a higher-level resource that manages Pods and ensures the desired
number of replicas are always running.

### Create the deployment manifest

Save the following YAML as `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "250m"
            memory: "128Mi"
```

```bash
# Create the file
cat > nginx-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "250m"
            memory: "128Mi"
EOF
```

### Apply the deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

**Expected Output:**
```
deployment.apps/nginx created
```

### Verify the deployment is running

```bash
kubectl get deployments
```

**Expected Output:**
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           30s
```

### Verify the pod is running

```bash
kubectl get pods
```

**Expected Output:**
```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-9k4z2   1/1     Running   0          35s
```

If you see `ContainerCreating`, the image is still being pulled. Wait a few seconds and run the command again.

### Why this matters

The **Deployment** manifest is the declarative description of your desired state.
Kubernetes continuously reconciles the actual state of the cluster to match this desired state.
The `app: nginx` label connects the Deployment to its Pods via the `selector`.
Resource `requests` and `limits` ensure the scheduler places Pods on nodes with sufficient capacity
and that containers can't starve other workloads of resources.

---

## Step 4: Expose With a Service

A Pod has an IP address, but it's ephemeral — when a Pod restarts, it gets a new IP.
A **Service** provides a stable network endpoint (DNS name + IP) that routes traffic
to matching Pods.

We'll use a `NodePort` service, which opens a port on the node itself so we can reach
it from our local machine.

### Create the service manifest

```bash
cat > nginx-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
EOF
```

### Apply the service

```bash
kubectl apply -f nginx-service.yaml
```

**Expected Output:**
```
service/nginx-svc created
```

### Verify the service

```bash
kubectl get services
```

**Expected Output:**
```
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        5m
nginx-svc    NodePort    10.98.31.102   <none>        80:30080/TCP   10s
```

### Why this matters

The Service's `selector: app: nginx` matches the label on our Pods.
All traffic to the Service is automatically load-balanced across matching Pods.
`NodePort` exposes the service on port `30080` of every node in the cluster.
The `port: 80` is the Service's internal cluster port, and `targetPort: 80` is the container port.

---

## Step 5: Access the App

Now let's actually open the application in our browser or via curl.

### Option A: Use minikube service (easiest)

```bash
minikube service nginx-svc
```

**Expected Output:**
```
|-----------|-----------|-------------|---------------------------|
| NAMESPACE |   NAME    | TARGET PORT |            URL            |
|-----------|-----------|-------------|---------------------------|
| default   | nginx-svc |          80 | http://192.168.49.2:30080 |
|-----------|-----------|-------------|---------------------------|
🎉  Opening service default/nginx-svc in default browser...
```

This command automatically finds the minikube node IP, constructs the URL, and opens it in your browser.

### Get just the URL

```bash
minikube service nginx-svc --url
```

**Expected Output:**
```
http://192.168.49.2:30080
```

### Option B: Use kubectl port-forward

If minikube service doesn't work in your environment (e.g., running in a remote VM),
use port-forward to tunnel traffic to your local machine:

```bash
kubectl port-forward service/nginx-svc 8080:80
```

**Expected Output:**
```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

Then in another terminal or browser, open `http://localhost:8080`.

### Verify with curl

```bash
# Using minikube service URL
curl $(minikube service nginx-svc --url)

# Or via port-forward (while it's running)
curl http://localhost:8080
```

**Expected Output:**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

### Why this matters

The **Welcome to nginx!** page confirms that traffic is flowing correctly from your local
machine → minikube node → Service → Pod → nginx container.
`kubectl port-forward` is invaluable for debugging services during development without
needing to expose them publicly.

---

## Step 6: Scale the Deployment

One replica is fine for development, but production applications need multiple replicas
for availability and load distribution. Let's scale to 3.

### Scale up to 3 replicas

```bash
kubectl scale deployment nginx --replicas=3
```

**Expected Output:**
```
deployment.apps/nginx scaled
```

### Watch the new pods appear in real time

```bash
kubectl get pods -w
```

**Expected Output (live stream):**
```
NAME                     READY   STATUS              RESTARTS   AGE
nginx-6799fc88d8-9k4z2   1/1     Running             0          3m
nginx-6799fc88d8-xvp78   0/1     ContainerCreating   0          2s
nginx-6799fc88d8-r2k9q   0/1     ContainerCreating   0          2s
nginx-6799fc88d8-xvp78   1/1     Running             0          5s
nginx-6799fc88d8-r2k9q   1/1     Running             0          6s
```

Press `Ctrl+C` to stop watching.

### Verify the deployment shows 3/3 ready

```bash
kubectl get deployment nginx
```

**Expected Output:**
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           4m
```

### Verify pods are spread (note the pod names)

```bash
kubectl get pods -o wide
```

**Expected Output:**
```
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx-6799fc88d8-9k4z2   1/1     Running   0          4m    10.244.0.5   minikube   <none>           <none>
nginx-6799fc88d8-xvp78   1/1     Running   0          1m    10.244.0.6   minikube   <none>           <none>
nginx-6799fc88d8-r2k9q   1/1     Running   0          1m    10.244.0.7   minikube   <none>           <none>
```

### Why this matters

Scaling to 3 replicas means the Service now load-balances traffic across 3 Pods.
If any single Pod crashes, Kubernetes automatically restarts it and keeps 2 Pods serving traffic
while the third recovers. The Deployment controller continuously ensures the actual replica count
matches the desired count.

---

## Step 7: Update the App

Kubernetes supports **rolling updates**: it replaces pods one at a time, ensuring there's
always some capacity serving traffic. Let's update from nginx 1.25 to 1.26.

### Trigger the update

```bash
kubectl set image deployment/nginx nginx=nginx:1.26
```

**Expected Output:**
```
deployment.apps/nginx image updated
```

### Watch the rolling update in real time

```bash
kubectl rollout status deployment/nginx
```

**Expected Output:**
```
Waiting for deployment "nginx" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
deployment "nginx" successfully rolled out
```

### Watch pods being replaced

In a separate terminal, you can also run `kubectl get pods -w` during the update
to see old pods terminating and new ones starting.

### Verify the new image is in use

```bash
kubectl describe deployment nginx | grep Image
```

**Expected Output:**
```
    Image:      nginx:1.26
```

### View the rollout history

```bash
kubectl rollout history deployment/nginx
```

**Expected Output:**
```
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

### Roll back if something goes wrong

```bash
# Roll back to the previous version
kubectl rollout undo deployment/nginx

# Confirm the rollback completed
kubectl rollout status deployment/nginx
```

**Expected Output:**
```
deployment "nginx" successfully rolled out
```

### Why this matters

Rolling updates ensure **zero-downtime deployments**. Kubernetes respects the `maxUnavailable`
and `maxSurge` settings in the deployment strategy (both default to 25%) to control how many
Pods can be down or over-replicated during an update. The rollout history lets you audit
changes and instantly revert to any previous revision if the new version has problems.

---

## Step 8: View Logs

Logs are your primary debugging tool. Let's look at what nginx is logging.

### Get logs from one pod

First, grab a pod name:

```bash
kubectl get pods
```

Then view its logs:

```bash
kubectl logs nginx-6799fc88d8-9k4z2
```

**Expected Output:**
```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
...
2024/01/01 10:00:00 [notice] 1#1: start worker processes
```

### Follow logs in real time

```bash
kubectl logs -f nginx-6799fc88d8-9k4z2
```

Curl the service in another terminal while logs are streaming:
```bash
curl $(minikube service nginx-svc --url)
```

You'll see access log lines appear in the log stream:
```
192.168.49.1 - - [01/Jan/2024:10:05:30 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
```

Press `Ctrl+C` to stop following.

### Stream logs from ALL replicas at once

```bash
kubectl logs -f -l app=nginx --all-containers
```

This aggregates logs from all 3 nginx Pods into one stream — essential for debugging
load-balanced applications.

### Show only recent logs

```bash
# Last 20 lines
kubectl logs --tail=20 nginx-6799fc88d8-9k4z2

# Logs from the last 5 minutes
kubectl logs --since=5m nginx-6799fc88d8-9k4z2
```

### View logs from a crashed container

If a container has crashed and restarted, its current logs may not show the error.
Use `--previous` to retrieve logs from the last terminated container instance:

```bash
kubectl logs nginx-6799fc88d8-9k4z2 --previous
```

### Why this matters

Logs reveal what's happening inside your containers at runtime.
The `-f` flag is essential for watching live traffic or debugging issues as they occur.
The `-l` label selector lets you aggregate logs across all pods of a deployment without
needing to know individual pod names — critical when you have many replicas.

---

## Step 9: Inspect Resources

Before moving to cleanup, let's explore the full picture of what we've deployed.

### See everything in the default namespace

```bash
kubectl get all
```

**Expected Output:**
```
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-9k4z2   1/1     Running   0          8m
pod/nginx-6799fc88d8-xvp78   1/1     Running   0          5m
pod/nginx-6799fc88d8-r2k9q   1/1     Running   0          5m

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        15m
service/nginx-svc    NodePort    10.98.31.102   <none>        80:30080/TCP   7m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           8m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6799fc88d8   3         3         3       8m
```

This shows every resource `kubectl get all` tracks: Pods, Services, Deployments, and ReplicaSets.

### Deep dive into a pod

```bash
# Pick any of your pod names from kubectl get pods
kubectl describe pod nginx-6799fc88d8-9k4z2
```

**Expected Output (key sections):**
```
Name:             nginx-6799fc88d8-9k4z2
Namespace:        default
Node:             minikube/192.168.49.2
Start Time:       Mon, 01 Jan 2024 10:00:00 +0000
Labels:           app=nginx
                  pod-template-hash=6799fc88d8
Status:           Running
IP:               10.244.0.5
Controlled By:    ReplicaSet/nginx-6799fc88d8
Containers:
  nginx:
    Image:          nginx:1.26
    Port:           80/TCP
    State:          Running
      Started:      Mon, 01 Jan 2024 10:00:05 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     250m
      memory:  128Mi
    Requests:
      cpu:     100m
      memory:  64Mi
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  8m    default-scheduler  Successfully assigned default/nginx-... to minikube
  Normal  Pulled     8m    kubelet            Container image "nginx:1.26" already present
  Normal  Created    8m    kubelet            Created container nginx
  Normal  Started    8m    kubelet            Started container nginx
```

### Inspect the deployment in YAML

```bash
kubectl get deployment nginx -o yaml
```

This shows the complete state of the Deployment as stored in etcd —
the same format used in manifest files.

### View events for the namespace

```bash
kubectl get events --sort-by=.lastTimestamp
```

Events are the cluster's activity log — the first place to look when something isn't working.

### Check resource usage (requires Metrics Server)

```bash
# Enable metrics-server addon in minikube
minikube addons enable metrics-server

# Wait ~30 seconds for it to start, then:
kubectl top pods
```

**Expected Output:**
```
NAME                     CPU(cores)   MEMORY(bytes)
nginx-6799fc88d8-9k4z2   1m           6Mi
nginx-6799fc88d8-xvp78   1m           6Mi
nginx-6799fc88d8-r2k9q   1m           6Mi
```

### Why this matters

`kubectl get all` gives you a complete inventory of your workloads at a glance.
`kubectl describe` shows the full lifecycle of a resource including **Events** — these events
are invaluable for debugging: they tell you why a Pod failed to schedule, why an image failed
to pull, or why a container exited unexpectedly.

---

## Step 10: Clean Up

Let's remove everything we created and stop minikube.

### Delete the service

```bash
kubectl delete -f nginx-service.yaml
```

**Expected Output:**
```
service "nginx-svc" deleted
```

### Delete the deployment

```bash
kubectl delete -f nginx-deployment.yaml
```

**Expected Output:**
```
deployment.apps "nginx" deleted
```

Deleting the Deployment automatically deletes its ReplicaSet and all managed Pods.
Watch them disappear:

```bash
kubectl get pods -w
```

**Expected Output:**
```
NAME                     READY   STATUS        RESTARTS   AGE
nginx-6799fc88d8-9k4z2   1/1     Terminating   0          10m
nginx-6799fc88d8-xvp78   1/1     Terminating   0          7m
nginx-6799fc88d8-r2k9q   1/1     Terminating   0          7m
```

After a few seconds all pods disappear and the watch returns to empty.

### Verify everything is gone

```bash
kubectl get all
```

**Expected Output:**
```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   20m
```

Only the built-in `kubernetes` service remains — that's normal.

### Clean up local manifest files

```bash
rm nginx-deployment.yaml nginx-service.yaml
```

### Stop minikube

```bash
minikube stop
```

**Expected Output:**
```
✋  Stopping node "minikube"  ...
🛑  Powering off "minikube" via SSH ...
🛑  1 node stopped.
```

Stopping minikube preserves your cluster state — the next `minikube start` resumes where you left off.

### Optionally delete the cluster entirely

```bash
minikube delete
```

**Expected Output:**
```
🔥  Deleting "minikube" in docker ...
🔥  Removing /home/user/.minikube/machines/minikube ...
💀  Removed all traces of the "minikube" cluster.
```

This permanently deletes the cluster VM/container and all its data.
Use this for a truly clean slate.

### Why this matters

Always clean up lab resources — especially in cloud environments where running workloads
cost money. Deleting a Deployment cascades to its ReplicaSets and Pods.
`minikube stop` (not delete) is ideal between sessions: your cluster state is preserved,
and startup is near-instant next time.

---

## What You Accomplished

In 10 minutes you have:

| Step | What You Did | Key Concept |
|------|-------------|-------------|
| ✅ 1 | Verified tools are installed | kubectl, minikube |
| ✅ 2 | Started a local Kubernetes cluster | Control plane, nodes |
| ✅ 3 | Deployed an app with a Deployment | Pods, ReplicaSets, Deployments |
| ✅ 4 | Exposed the app with a Service | ClusterIP, NodePort, selectors |
| ✅ 5 | Accessed the running app | Port-forward, minikube service |
| ✅ 6 | Scaled to 3 replicas | Horizontal scaling, load balancing |
| ✅ 7 | Updated the image with zero downtime | Rolling updates, rollback |
| ✅ 8 | Streamed and inspected logs | Observability, debugging |
| ✅ 9 | Inspected resources in detail | Events, describe, get all |
| ✅ 10 | Cleaned up all resources | Resource lifecycle |

---

## Next Steps

Now that you've got the fundamentals, explore these topics next:

- **ConfigMaps & Secrets** — Externalize configuration from your container images
- **Persistent Volumes** — Give your pods durable storage that survives restarts
- **Ingress** — Route external HTTP/S traffic to multiple services with a single IP
- **Namespaces** — Isolate environments (dev, staging, prod) within one cluster
- **Resource Quotas & Limits** — Govern how much CPU and memory workloads can consume
- **Liveness & Readiness Probes** — Tell Kubernetes when a container is healthy
- **HorizontalPodAutoscaler** — Auto-scale based on CPU/memory metrics
- **RBAC** — Control who can do what inside the cluster
- **Helm** — Package and deploy complex multi-resource applications

See [`COMMANDS.md`](./COMMANDS.md) for a comprehensive reference of all `kubectl` commands.

---

*For help at any time, run `kubectl help` or `kubectl <command> --help`.*
