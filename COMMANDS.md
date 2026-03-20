# kubectl Command Reference

A comprehensive reference for `kubectl` commands used throughout this Kubernetes bootcamp.
Each section includes syntax, flags, real-world examples, and expected output.

---

## Table of Contents

1. [kubectl Basics](#1-kubectl-basics)
2. [Creating Resources](#2-creating-resources)
3. [Updating Resources](#3-updating-resources)
4. [Deleting Resources](#4-deleting-resources)
5. [Debugging & Troubleshooting](#5-debugging--troubleshooting)
6. [Context & Config](#6-context--config)
7. [Rollouts](#7-rollouts)
8. [Labels & Annotations](#8-labels--annotations)
9. [Namespace Operations](#9-namespace-operations)
10. [Advanced Operations](#10-advanced-operations)

---

## 1. kubectl Basics

### `kubectl version`

Displays the client and server version of kubectl.

**Syntax:**
```bash
kubectl version [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `--client` | Show only the client version |
| `--output`, `-o` | Output format: `json`, `yaml`, `short` |
| `--short` | Print just the version number |

**Examples:**

```bash
# Show both client and server versions
kubectl version

# Show only the kubectl client version
kubectl version --client

# Show version in short format
kubectl version --client --short

# Output version as JSON
kubectl version -o json
```

**Expected Output:**
```
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.2
```

---

### `kubectl cluster-info`

Displays the address of the Kubernetes control plane and cluster services.

**Syntax:**
```bash
kubectl cluster-info [flags]
```

**Examples:**

```bash
# Show cluster endpoint info
kubectl cluster-info

# Dump full cluster diagnostic info
kubectl cluster-info dump

# Dump cluster info to a directory
kubectl cluster-info dump --output-directory=/tmp/cluster-dump
```

**Expected Output:**
```
Kubernetes control plane is running at https://127.0.0.1:6443
CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

---

### `kubectl get`

Lists one or many resources in the cluster.

**Syntax:**
```bash
kubectl get <resource> [name] [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `-o wide` | Show additional columns (node, IP, etc.) |
| `-o yaml` | Output as YAML |
| `-o json` | Output as JSON |
| `-o name` | Output only resource names |
| `-n`, `--namespace` | Target namespace |
| `-A`, `--all-namespaces` | List across all namespaces |
| `--watch`, `-w` | Watch for changes in real time |
| `-l`, `--selector` | Filter by label selector |
| `--show-labels` | Show all labels as a last column |
| `--field-selector` | Filter by field value |
| `--sort-by` | Sort list by a JSONPath expression |

**Examples:**

```bash
# List all pods in the current namespace
kubectl get pods

# List all pods across all namespaces
kubectl get pods -A

# List pods with extra info (IP, node)
kubectl get pods -o wide

# Watch pods update in real time
kubectl get pods -w

# Get a specific pod by name
kubectl get pod my-nginx-abc123

# Get pods with a specific label
kubectl get pods -l app=nginx

# Get pods and show their labels
kubectl get pods --show-labels

# Get multiple resource types at once
kubectl get pods,services,deployments

# List all deployments in the kube-system namespace
kubectl get deployments -n kube-system

# Get output as YAML for a specific pod
kubectl get pod my-nginx-abc123 -o yaml

# Get all resources in a namespace
kubectl get all -n production

# Sort pods by creation time
kubectl get pods --sort-by=.metadata.creationTimestamp

# Filter by field selector (running pods only)
kubectl get pods --field-selector=status.phase=Running

# Get nodes and their status
kubectl get nodes

# Get nodes with extra details
kubectl get nodes -o wide
```

**Expected Output (`kubectl get pods -o wide`):**
```
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx-6799fc88d8-9k4z2   1/1     Running   0          2m    10.244.0.5   minikube   <none>           <none>
nginx-6799fc88d8-xvp78   1/1     Running   0          2m    10.244.0.6   minikube   <none>           <none>
```

---

### `kubectl describe`

Shows detailed information about a resource or group of resources.

**Syntax:**
```bash
kubectl describe <resource> [name] [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `-n`, `--namespace` | Target namespace |
| `-l`, `--selector` | Describe resources matching a label selector |
| `-f`, `--filename` | Describe from a file |

**Examples:**

```bash
# Describe a specific pod
kubectl describe pod my-nginx-abc123

# Describe all pods
kubectl describe pods

# Describe a deployment
kubectl describe deployment nginx

# Describe a node
kubectl describe node minikube

# Describe a service
kubectl describe service nginx-svc

# Describe pods matching a label
kubectl describe pods -l app=nginx

# Describe a persistent volume claim
kubectl describe pvc my-storage-claim

# Describe a config map
kubectl describe configmap app-config

# Describe a secret (values are base64-encoded)
kubectl describe secret my-secret
```

**Expected Output (truncated):**
```
Name:         nginx-6799fc88d8-9k4z2
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Mon, 01 Jan 2024 10:00:00 +0000
Labels:       app=nginx
              pod-template-hash=6799fc88d8
Status:       Running
IP:           10.244.0.5
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m    default-scheduler  Successfully assigned default/nginx-... to minikube
  Normal  Pulled     2m    kubelet            Container image "nginx:latest" already present on machine
  Normal  Created    2m    kubelet            Created container nginx
  Normal  Started    2m    kubelet            Started container nginx
```

---

### `kubectl explain`

Lists the fields for supported API resources. Essential for writing YAML manifests.

**Syntax:**
```bash
kubectl explain <resource>[.field] [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `--recursive` | Print all fields and sub-fields |
| `--api-version` | Get fields for a specific API version |

**Examples:**

```bash
# Explain the Pod resource
kubectl explain pod

# Explain the spec field of a Pod
kubectl explain pod.spec

# Explain containers in pod spec
kubectl explain pod.spec.containers

# Explain resource limits
kubectl explain pod.spec.containers.resources

# Explain Deployment spec
kubectl explain deployment.spec

# Recursively show all pod fields
kubectl explain pod --recursive

# Explain a Service spec
kubectl explain service.spec

# Explain PersistentVolumeClaim
kubectl explain pvc.spec

# Explain node affinity
kubectl explain pod.spec.affinity.nodeAffinity
```

---

## 2. Creating Resources

### `kubectl apply`

Apply a configuration to a resource by file or stdin. Creates or updates resources.
**Preferred approach** for declarative management.

**Syntax:**
```bash
kubectl apply -f <filename|directory|URL> [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `-f`, `--filename` | File, directory, or URL |
| `-R`, `--recursive` | Process directories recursively |
| `--dry-run=client` | Preview changes without applying |
| `--dry-run=server` | Validate against server without persisting |
| `--prune` | Remove resources no longer in the file |
| `--record` | Record the command in resource annotation |
| `-o`, `--output` | Output format after apply |

**Examples:**

```bash
# Apply a single YAML file
kubectl apply -f deployment.yaml

# Apply all YAML files in a directory
kubectl apply -f ./k8s/

# Apply recursively
kubectl apply -R -f ./manifests/

# Apply from a URL
kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook/all-in-one/guestbook-all-in-one.yaml

# Dry run to preview what would change
kubectl apply -f deployment.yaml --dry-run=client

# Server-side dry run (validates against API)
kubectl apply -f deployment.yaml --dry-run=server

# Apply and output the result as YAML
kubectl apply -f service.yaml -o yaml

# Apply multiple files
kubectl apply -f deployment.yaml -f service.yaml

# Apply with pruning (remove stale resources)
kubectl apply -f ./k8s/ --prune -l app=myapp
```

**Expected Output:**
```
deployment.apps/nginx created
service/nginx-svc created
```

---

### `kubectl create`

Create a resource from a file or stdin. Fails if the resource already exists.
Use `apply` for idempotent operations.

**Syntax:**
```bash
kubectl create <resource> [flags]
kubectl create -f <filename>
```

**Examples:**

```bash
# Create from a YAML file
kubectl create -f pod.yaml

# Create a namespace
kubectl create namespace staging

# Create a deployment imperatively
kubectl create deployment nginx --image=nginx:1.25

# Create a deployment with 3 replicas
kubectl create deployment web --image=nginx:1.25 --replicas=3

# Create a service to expose a deployment
kubectl create service nodeport nginx --tcp=80:80

# Create a ConfigMap from literal values
kubectl create configmap app-config --from-literal=ENV=production --from-literal=LOG_LEVEL=info

# Create a ConfigMap from a file
kubectl create configmap nginx-conf --from-file=nginx.conf

# Create a Secret from literal values
kubectl create secret generic db-creds --from-literal=user=admin --from-literal=password=s3cr3t

# Create a Secret from files
kubectl create secret generic tls-certs --from-file=tls.crt --from-file=tls.key

# Create a ServiceAccount
kubectl create serviceaccount my-app-sa

# Dry run to generate YAML without creating
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

**Expected Output:**
```
deployment.apps/nginx created
namespace/staging created
configmap/app-config created
```

---

### `kubectl run`

Run a particular image in a pod. Useful for quick testing.

**Syntax:**
```bash
kubectl run <name> --image=<image> [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `--image` | Container image to run |
| `--port` | Port to expose |
| `--env` | Environment variables |
| `--labels`, `-l` | Labels for the pod |
| `--rm` | Delete pod after it exits |
| `-it` | Keep stdin open and allocate a tty |
| `--restart` | Restart policy (Always, OnFailure, Never) |
| `--command` | Override ENTRYPOINT with custom command |

**Examples:**

```bash
# Run an nginx pod
kubectl run nginx --image=nginx:1.25

# Run a temporary busybox pod interactively
kubectl run -it busybox --image=busybox --rm --restart=Never -- sh

# Run a temporary pod to test DNS resolution
kubectl run -it dns-test --image=busybox --rm --restart=Never -- nslookup kubernetes.default

# Run a curl pod to test an internal service
kubectl run curl-test --image=curlimages/curl --rm -it --restart=Never -- curl http://nginx-svc

# Run a pod with environment variables
kubectl run myapp --image=myapp:v1 --env="ENV=prod" --env="PORT=8080"

# Run a pod on a specific port
kubectl run webapp --image=nginx --port=80

# Generate pod YAML without running
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Run a one-off job
kubectl run pi --image=perl:5 --restart=Never -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

---

## 3. Updating Resources

### `kubectl edit`

Edit a resource in your default editor. Opens the live resource YAML for modification.

**Syntax:**
```bash
kubectl edit <resource> <name> [flags]
```

**Examples:**

```bash
# Edit a deployment
kubectl edit deployment nginx

# Edit a service
kubectl edit service nginx-svc

# Edit a configmap
kubectl edit configmap app-config

# Edit using a different editor
KUBE_EDITOR="nano" kubectl edit deployment nginx

# Edit a resource in a specific namespace
kubectl edit deployment nginx -n production
```

> **Tip:** Set `KUBE_EDITOR=nano` or `KUBE_EDITOR=code --wait` in your shell profile for a preferred editor.

---

### `kubectl patch`

Update fields of a resource in place using a strategic merge, JSON merge, or JSON patch.

**Syntax:**
```bash
kubectl patch <resource> <name> -p '<patch>' [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `-p`, `--patch` | The patch content |
| `--type` | Patch type: `strategic` (default), `merge`, `json` |

**Examples:**

```bash
# Patch a deployment's replica count
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'

# Patch using JSON patch syntax
kubectl patch deployment nginx --type=json -p '[{"op":"replace","path":"/spec/replicas","value":3}]'

# Patch a pod's container image
kubectl patch pod nginx-abc123 -p '{"spec":{"containers":[{"name":"nginx","image":"nginx:1.26"}]}}'

# Add a label to a node
kubectl patch node minikube -p '{"metadata":{"labels":{"disk":"ssd"}}}'

# Patch a service type from ClusterIP to NodePort
kubectl patch service nginx-svc -p '{"spec":{"type":"NodePort"}}'

# Mark a node as unschedulable (cordon effect)
kubectl patch node minikube -p '{"spec":{"unschedulable":true}}'
```

---

### `kubectl scale`

Scale the number of replicas for a deployment, replica set, or stateful set.

**Syntax:**
```bash
kubectl scale <resource> <name> --replicas=<count> [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `--replicas` | New desired number of replicas |
| `--current-replicas` | Only scale if current replica count matches |
| `--timeout` | Wait this long before giving up |

**Examples:**

```bash
# Scale a deployment to 3 replicas
kubectl scale deployment nginx --replicas=3

# Scale down to 0 (stops all pods without deleting the deployment)
kubectl scale deployment nginx --replicas=0

# Scale only if currently at 2 replicas
kubectl scale deployment nginx --replicas=5 --current-replicas=2

# Scale a stateful set
kubectl scale statefulset mysql --replicas=3

# Scale multiple deployments at once
kubectl scale deployment nginx redis --replicas=2

# Scale using a YAML file
kubectl scale -f deployment.yaml --replicas=4
```

**Expected Output:**
```
deployment.apps/nginx scaled
```

---

### `kubectl set image`

Update the image of a container in a pod template (triggers rolling update).

**Syntax:**
```bash
kubectl set image <resource>/<name> <container>=<image> [flags]
```

**Examples:**

```bash
# Update nginx deployment to a new image version
kubectl set image deployment/nginx nginx=nginx:1.26

# Update with --record (deprecated but still seen)
kubectl set image deployment/nginx nginx=nginx:1.26 --record

# Update all containers in a deployment
kubectl set image deployment/web *=myrepo/myapp:v2

# Update a DaemonSet image
kubectl set image daemonset/fluentd fluentd=fluentd:v1.16

# Update and watch the rollout
kubectl set image deployment/nginx nginx=nginx:1.26 && kubectl rollout status deployment/nginx
```

---

### `kubectl replace`

Replace a resource by filename or stdin. The resource must already exist.

**Syntax:**
```bash
kubectl replace -f <filename> [flags]
```

**Examples:**

```bash
# Replace a resource from file
kubectl replace -f deployment.yaml

# Force replace (delete then re-create — use with caution)
kubectl replace --force -f pod.yaml

# Replace from stdin
kubectl get deployment nginx -o yaml | sed 's/replicas: 2/replicas: 4/' | kubectl replace -f -
```

> **Warning:** `kubectl replace --force` deletes and recreates the resource. There will be downtime unless replicas are managed by a higher-level controller.

---

## 4. Deleting Resources

### `kubectl delete`

Delete resources by file, name, label selector, or type.

**Syntax:**
```bash
kubectl delete <resource> <name|--all|-l selector> [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `-f`, `--filename` | Delete resources described in file |
| `--all` | Delete all resources of the given type |
| `-l`, `--selector` | Delete by label selector |
| `--grace-period` | Seconds before forced deletion (0 = immediate) |
| `--force` | Immediately remove resource (skips graceful shutdown) |
| `--cascade` | Also delete dependent resources (default: true) |
| `--wait` | Wait for deletion to complete |
| `-n`, `--namespace` | Target namespace |

**Examples:**

```bash
# Delete a pod by name
kubectl delete pod my-nginx-abc123

# Delete a deployment by name
kubectl delete deployment nginx

# Delete from a YAML file
kubectl delete -f deployment.yaml

# Delete all resources defined in a directory
kubectl delete -f ./k8s/

# Delete all pods in the current namespace
kubectl delete pods --all

# Delete pods matching a label
kubectl delete pods -l app=nginx

# Delete all resources in a namespace
kubectl delete all --all -n staging

# Force delete a stuck pod immediately
kubectl delete pod stuck-pod --grace-period=0 --force

# Delete a namespace (and all resources within it)
kubectl delete namespace staging

# Delete multiple resource types at once
kubectl delete pod,service,deployment -l app=myapp

# Delete with no waiting
kubectl delete pod nginx --wait=false
```

**Expected Output:**
```
pod "my-nginx-abc123" deleted
deployment.apps "nginx" deleted
```

> **Tip:** Deleting a pod managed by a Deployment will cause it to be recreated. To permanently remove pods, delete the Deployment.

---

## 5. Debugging & Troubleshooting

### `kubectl logs`

Print the logs for a container in a pod.

**Syntax:**
```bash
kubectl logs <pod> [-c container] [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `-c`, `--container` | Container name (for multi-container pods) |
| `-f`, `--follow` | Stream logs in real time |
| `--tail` | Number of recent lines to show |
| `--since` | Show logs since duration (e.g., `1h`, `5m`) |
| `--since-time` | Show logs since RFC3339 timestamp |
| `--previous`, `-p` | Show logs from previous (crashed) container |
| `--timestamps` | Include timestamps |
| `-l`, `--selector` | Fetch logs from pods matching label |
| `--all-containers` | Get logs from all containers |

**Examples:**

```bash
# View pod logs
kubectl logs my-nginx-abc123

# Stream (follow) logs in real time
kubectl logs -f my-nginx-abc123

# Show last 100 lines
kubectl logs --tail=100 my-nginx-abc123

# Show logs from last 30 minutes
kubectl logs --since=30m my-nginx-abc123

# Show logs from a specific container in a multi-container pod
kubectl logs my-pod -c sidecar-container

# Show logs from a crashed previous container instance
kubectl logs my-pod --previous

# Show logs with timestamps
kubectl logs my-pod --timestamps

# Stream logs from all pods with a label (great for deployments)
kubectl logs -f -l app=nginx --all-containers

# Show logs for all containers in a pod
kubectl logs my-pod --all-containers=true

# Logs since a specific time
kubectl logs my-pod --since-time=2024-01-01T10:00:00Z
```

---

### `kubectl exec`

Execute a command inside a running container.

**Syntax:**
```bash
kubectl exec <pod> [-c container] -- <command> [args]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `-c`, `--container` | Target container in multi-container pod |
| `-i`, `--stdin` | Pass stdin to the container |
| `-t`, `--tty` | Allocate a TTY |

**Examples:**

```bash
# Get a shell inside a running pod
kubectl exec -it my-nginx-abc123 -- /bin/bash

# Get a shell (use sh if bash not available)
kubectl exec -it my-nginx-abc123 -- /bin/sh

# Run a single command without interactive shell
kubectl exec my-nginx-abc123 -- ls /usr/share/nginx/html

# Check environment variables inside a container
kubectl exec my-nginx-abc123 -- env

# Check which processes are running
kubectl exec my-nginx-abc123 -- ps aux

# Execute in a specific container of a multi-container pod
kubectl exec -it my-pod -c sidecar -- /bin/sh

# Test internal DNS resolution
kubectl exec -it my-nginx-abc123 -- nslookup kubernetes.default

# Check network connectivity
kubectl exec -it my-nginx-abc123 -- curl http://my-service:8080/health

# View a file inside the container
kubectl exec my-nginx-abc123 -- cat /etc/nginx/nginx.conf

# Run a database query in a mysql pod
kubectl exec -it mysql-pod -- mysql -u root -p -e "SHOW DATABASES;"
```

---

### `kubectl port-forward`

Forward one or more local ports to a port on a pod or service. Useful for local development and debugging.

**Syntax:**
```bash
kubectl port-forward <pod|service|deployment> <localPort>:<remotePort> [flags]
```

**Examples:**

```bash
# Forward local port 8080 to pod port 80
kubectl port-forward pod/my-nginx-abc123 8080:80

# Forward to a service
kubectl port-forward service/nginx-svc 8080:80

# Forward to a deployment
kubectl port-forward deployment/nginx 8080:80

# Use the same port on both ends
kubectl port-forward pod/my-nginx-abc123 80:80

# Forward multiple ports
kubectl port-forward pod/myapp 8080:8080 9090:9090

# Listen on all interfaces (not just localhost)
kubectl port-forward --address 0.0.0.0 pod/my-nginx-abc123 8080:80

# Port forward in a specific namespace
kubectl port-forward -n monitoring pod/prometheus-abc123 9090:9090
```

---

### `kubectl top`

Display resource (CPU/memory) usage for pods or nodes. Requires Metrics Server.

**Syntax:**
```bash
kubectl top <pods|nodes> [flags]
```

**Examples:**

```bash
# Show resource usage for all pods
kubectl top pods

# Show resource usage for pods in a specific namespace
kubectl top pods -n kube-system

# Sort pods by CPU usage
kubectl top pods --sort-by=cpu

# Sort pods by memory usage
kubectl top pods --sort-by=memory

# Show resource usage for all nodes
kubectl top nodes

# Show containers within pods
kubectl top pods --containers
```

**Expected Output:**
```
NAME                     CPU(cores)   MEMORY(bytes)
nginx-6799fc88d8-9k4z2   1m           6Mi
nginx-6799fc88d8-xvp78   1m           6Mi
```

---

### `kubectl events` / `kubectl get events`

View events in the cluster. Events are the first place to look when diagnosing failures.

**Examples:**

```bash
# List all events in the current namespace
kubectl get events

# List events sorted by time (most recent first)
kubectl get events --sort-by=.lastTimestamp

# Watch events in real time
kubectl get events -w

# List events in all namespaces
kubectl get events -A

# List only warning events
kubectl get events --field-selector type=Warning

# List events for a specific object
kubectl get events --field-selector involvedObject.name=my-nginx-abc123

# Show events with a custom output format
kubectl get events -o custom-columns='TIMESTAMP:.lastTimestamp,REASON:.reason,OBJECT:.involvedObject.name,MESSAGE:.message'
```

---

## 6. Context & Config

### `kubectl config` Commands

Manage kubeconfig files, contexts, clusters, and users.

**Examples:**

```bash
# View the current kubeconfig
kubectl config view

# View kubeconfig with secrets revealed
kubectl config view --minify --raw

# List all available contexts
kubectl config get-contexts

# Show the current context
kubectl config current-context

# Switch to a different context
kubectl config use-context my-cluster-context

# Switch context to minikube
kubectl config use-context minikube

# Rename a context
kubectl config rename-context old-name new-name

# Delete a context
kubectl config delete-context old-context

# Set the default namespace for the current context
kubectl config set-context --current --namespace=production

# Set a new cluster in kubeconfig
kubectl config set-cluster my-cluster --server=https://1.2.3.4:6443

# Set credentials for a user
kubectl config set-credentials my-user --token=mytoken123

# Create a new context
kubectl config set-context my-new-context --cluster=my-cluster --user=my-user --namespace=default

# Merge two kubeconfig files
KUBECONFIG=~/.kube/config:~/new-cluster.yaml kubectl config view --flatten > ~/.kube/merged-config
```

---

## 7. Rollouts

### `kubectl rollout`

Manage rollouts of deployments, daemonsets, and statefulsets.

**Syntax:**
```bash
kubectl rollout <subcommand> <resource> [flags]
```

**Subcommands:** `status`, `history`, `undo`, `pause`, `resume`, `restart`

**Examples:**

```bash
# Check the status of a rollout
kubectl rollout status deployment/nginx

# Watch rollout status until complete
kubectl rollout status deployment/nginx --watch

# View rollout history
kubectl rollout history deployment/nginx

# View details of a specific revision
kubectl rollout history deployment/nginx --revision=3

# Roll back to the previous version
kubectl rollout undo deployment/nginx

# Roll back to a specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# Pause a rolling update
kubectl rollout pause deployment/nginx

# Resume a paused rollout
kubectl rollout resume deployment/nginx

# Restart a deployment (triggers rolling update with same image)
kubectl rollout restart deployment/nginx

# Check rollout status of a DaemonSet
kubectl rollout status daemonset/fluentd

# Undo a DaemonSet update
kubectl rollout undo daemonset/fluentd
```

**Expected Output (`kubectl rollout status`):**
```
Waiting for deployment "nginx" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
deployment "nginx" successfully rolled out
```

---

## 8. Labels & Annotations

### `kubectl label`

Add, update, or remove labels from resources.

**Syntax:**
```bash
kubectl label <resource> <name> <key>=<value> [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `--overwrite` | Overwrite existing label |
| `-l` | Select resources by label to update |
| `--all` | Select all resources |

**Examples:**

```bash
# Add a label to a pod
kubectl label pod my-nginx-abc123 environment=production

# Add a label to a node
kubectl label node worker-1 disk=ssd

# Update an existing label (requires --overwrite)
kubectl label pod my-nginx-abc123 environment=staging --overwrite

# Remove a label (use key- syntax)
kubectl label pod my-nginx-abc123 environment-

# Label all pods with a specific selector
kubectl label pods -l app=nginx tier=frontend

# Label all pods in the namespace
kubectl label pods --all region=us-east-1

# Add label to a deployment
kubectl label deployment nginx release=stable

# Verify the label was applied
kubectl get pods --show-labels
```

---

### `kubectl annotate`

Add, update, or remove annotations from resources.

**Syntax:**
```bash
kubectl annotate <resource> <name> <key>=<value> [flags]
```

**Examples:**

```bash
# Add an annotation to a pod
kubectl annotate pod my-nginx-abc123 description="Primary nginx pod"

# Add an annotation to a deployment
kubectl annotate deployment nginx team=backend-team

# Update an existing annotation
kubectl annotate deployment nginx team=platform-team --overwrite

# Remove an annotation
kubectl annotate deployment nginx team-

# Annotate with a URL
kubectl annotate deployment nginx runbook="https://wiki.example.com/nginx-runbook"

# Add annotation to all pods
kubectl annotate pods --all maintainer=ops-team

# Add a prometheus scrape annotation to a service
kubectl annotate service my-svc prometheus.io/scrape="true" prometheus.io/port="8080"
```

---

## 9. Namespace Operations

Namespaces provide logical isolation of resources within a cluster.

**Examples:**

```bash
# List all namespaces
kubectl get namespaces

# Get resources in a specific namespace
kubectl get pods -n kube-system

# Get resources in ALL namespaces
kubectl get pods -A

# Create a namespace
kubectl create namespace development

# Delete a namespace (deletes ALL resources inside it)
kubectl delete namespace development

# Set default namespace for your current context
kubectl config set-context --current --namespace=production

# View which namespace is currently default
kubectl config view --minify | grep namespace

# Get all resources in a namespace
kubectl get all -n staging

# Copy a secret from one namespace to another
kubectl get secret my-secret -n source-ns -o yaml | \
  sed 's/namespace: source-ns/namespace: target-ns/' | \
  kubectl apply -f -

# Count pods per namespace
kubectl get pods -A --no-headers | awk '{print $1}' | sort | uniq -c | sort -rn

# List all resource quotas across namespaces
kubectl get resourcequota -A

# Check LimitRanges in a namespace
kubectl get limitrange -n production
```

> **Tip:** Install `kubens` (from the `kubectx` project) for fast namespace switching: `kubens production`

---

## 10. Advanced Operations

### `kubectl autoscale`

Create a HorizontalPodAutoscaler (HPA) to automatically scale a deployment.

**Syntax:**
```bash
kubectl autoscale <resource> <name> --min=<n> --max=<n> [--cpu-percent=<n>]
```

**Examples:**

```bash
# Autoscale a deployment between 2 and 10 replicas based on CPU
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# View the HPA
kubectl get hpa

# Describe the HPA (shows current metrics)
kubectl describe hpa nginx

# Delete the HPA
kubectl delete hpa nginx
```

---

### `kubectl cordon` / `kubectl uncordon`

Mark a node as unschedulable (cordon) or schedulable again (uncordon). New pods will not be scheduled on a cordoned node.

**Examples:**

```bash
# Cordon a node (prevent new pods from scheduling)
kubectl cordon worker-node-1

# Verify the node is cordoned
kubectl get nodes

# Uncordon a node (allow scheduling again)
kubectl uncordon worker-node-1
```

**Expected Output (`kubectl get nodes` after cordon):**
```
NAME            STATUS                     ROLES    AGE   VERSION
worker-node-1   Ready,SchedulingDisabled   <none>   5d    v1.28.2
```

---

### `kubectl drain`

Safely evict all pods from a node before maintenance. Combines cordon + eviction.

**Syntax:**
```bash
kubectl drain <node> [flags]
```

**Common Flags:**
| Flag | Description |
|------|-------------|
| `--ignore-daemonsets` | Ignore DaemonSet-managed pods |
| `--delete-emptydir-data` | Delete pods using emptyDir volumes |
| `--force` | Force eviction of unmanaged pods |
| `--grace-period` | Override pod termination grace period |
| `--timeout` | Max time to wait for eviction |

**Examples:**

```bash
# Drain a node before maintenance
kubectl drain worker-node-1 --ignore-daemonsets

# Drain and delete emptyDir data
kubectl drain worker-node-1 --ignore-daemonsets --delete-emptydir-data

# Drain with force (for pods not managed by a controller)
kubectl drain worker-node-1 --ignore-daemonsets --force

# After maintenance, uncordon to bring node back
kubectl uncordon worker-node-1
```

---

### `kubectl taint`

Add, update, or remove taints on nodes. Taints repel pods that don't have matching tolerations.

**Syntax:**
```bash
kubectl taint node <name> <key>=<value>:<effect>
```

**Effects:** `NoSchedule`, `PreferNoSchedule`, `NoExecute`

**Examples:**

```bash
# Taint a node to prevent scheduling unless pod tolerates it
kubectl taint node worker-node-1 dedicated=gpu:NoSchedule

# Apply NoExecute taint (evicts existing pods too)
kubectl taint node worker-node-1 maintenance=true:NoExecute

# Remove a taint (append -)
kubectl taint node worker-node-1 dedicated:NoSchedule-

# View taints on nodes
kubectl describe node worker-node-1 | grep -A3 Taints

# Taint control plane nodes (common in managed clusters)
kubectl taint node master-node node-role.kubernetes.io/control-plane:NoSchedule
```

---

### `kubectl certificate`

Approve or deny certificate signing requests (CSRs). Used with TLS bootstrapping.

**Examples:**

```bash
# List pending certificate signing requests
kubectl get csr

# Approve a CSR
kubectl certificate approve node-csr-abc123

# Deny a CSR
kubectl certificate deny node-csr-abc123

# View the CSR details
kubectl describe csr node-csr-abc123
```

---

### Additional Useful Commands

```bash
# Copy files to/from a container
kubectl cp my-pod:/var/log/app.log ./app.log
kubectl cp ./config.json my-pod:/app/config.json

# Copy from a specific container
kubectl cp my-pod:/var/log/app.log ./app.log -c my-container

# Wait for a pod to be ready
kubectl wait --for=condition=ready pod/my-nginx-abc123 --timeout=60s

# Wait for all pods in a deployment to be available
kubectl wait --for=condition=available deployment/nginx --timeout=120s

# Apply a resource quota to a namespace
kubectl create quota dev-quota --hard=cpu=2,memory=4Gi,pods=10 -n development

# Get raw API output
kubectl get --raw /api/v1/namespaces/default/pods

# Proxy to the Kubernetes API server
kubectl proxy --port=8001

# Run kubectl with verbose output (great for debugging)
kubectl get pods -v=8

# Output all API resources available in the cluster
kubectl api-resources

# Check which API versions are available
kubectl api-versions

# List all supported resource types and their short names
kubectl api-resources --namespaced=true

# Check your permissions
kubectl auth can-i create pods
kubectl auth can-i delete deployments --namespace=production
kubectl auth can-i '*' '*' --all-namespaces
```

---

## Quick Reference Cheat Sheet

| Task | Command |
|------|---------|
| List pods | `kubectl get pods` |
| List pods (all namespaces) | `kubectl get pods -A` |
| Describe a pod | `kubectl describe pod <name>` |
| Follow pod logs | `kubectl logs -f <pod>` |
| Shell into a pod | `kubectl exec -it <pod> -- /bin/sh` |
| Apply a manifest | `kubectl apply -f file.yaml` |
| Scale a deployment | `kubectl scale deploy/<name> --replicas=3` |
| Update an image | `kubectl set image deploy/<name> <c>=<img>:<tag>` |
| Roll back | `kubectl rollout undo deploy/<name>` |
| Port forward | `kubectl port-forward <pod> 8080:80` |
| Switch namespace | `kubectl config set-context --current --namespace=<ns>` |
| Switch context | `kubectl config use-context <ctx>` |
| Delete all in namespace | `kubectl delete all --all -n <ns>` |
| Drain a node | `kubectl drain <node> --ignore-daemonsets` |
| Watch resources | `kubectl get pods -w` |
| Get resource YAML | `kubectl get pod <name> -o yaml` |
| Dry run apply | `kubectl apply -f file.yaml --dry-run=client` |
| Generate manifest | `kubectl create deploy nginx --image=nginx --dry-run=client -o yaml` |

---

*This reference covers the most common kubectl operations. For the full documentation, run `kubectl help` or visit [kubernetes.io/docs](https://kubernetes.io/docs/reference/kubectl/).*
