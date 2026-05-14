# ☸️ Kubernetes Orchestration — Cheat Sheet & Learning Guide

> *"Kubernetes is not just a container orchestrator — it's a platform for building platforms."*
> — Kelsey Hightower

This file is a **self-contained reference** distilled from all lectures in the `orchestrators/` folder.
It covers every concept from first principles to Helm chart deployment.

---

## 📚 Table of Contents

1. [What is a Container Orchestrator?](#1-what-is-a-container-orchestrator)
2. [Core Philosophy](#2-core-philosophy)
3. [Architecture: Brain vs Muscle](#3-architecture-brain-vs-muscle)
4. [The Reconciliation Loop (Most Important Concept)](#4-the-reconciliation-loop)
5. [Kubernetes Objects Quick Reference](#5-kubernetes-objects-quick-reference)
6. [How a Deployment Actually Works (Step-by-Step)](#6-how-a-deployment-actually-works)
7. [Networking Deep Dive](#7-networking-deep-dive)
8. [Configuration & Secrets](#8-configuration--secrets)
9. [Scaling](#9-scaling)
10. [Health Checks (Probes)](#10-health-checks-probes)
11. [Storage (PV / PVC / StorageClass)](#11-storage)
12. [Helm — The Package Manager for K8s](#12-helm)
13. [Essential kubectl Commands](#13-essential-kubectl-commands)
14. [Real YAML Examples from This Project](#14-real-yaml-examples)
15. [Decision Tree: Which Object Do I Use?](#15-decision-tree)
16. [Debugging Playbook](#16-debugging-playbook)
17. [Learning Roadmap](#17-learning-roadmap)

---

## 1. What is a Container Orchestrator?

Before Kubernetes, running containers at scale meant:
- Manually SSHing into machines to start/stop containers
- Writing custom scripts to restart crashed processes
- No built-in load balancing or service discovery

**Container orchestration** solves this by automating:
- **Scheduling** — deciding which machine runs which container
- **Self-healing** — restarting crashed containers automatically
- **Scaling** — adding/removing instances based on load
- **Rolling updates** — deploying new versions without downtime
- **Service discovery** — letting services find each other by name

Kubernetes (K8s) was born from Google's internal **Borg** system, which ran billions of containers/week.

---

## 2. Core Philosophy

| Principle | What it means |
|-----------|---------------|
| **Declarative Model** | You describe *what you want* (e.g. "3 replicas"), not *how to do it*. K8s figures out the steps. |
| **Self-Healing** | If a pod crashes or a node dies, K8s automatically restores the desired state. |
| **Immutable Infrastructure** | Never modify a running container. Build a new image → roll it out. Old pods are replaced. |
| **Everything is an API Object** | Pods, Services, Deployments — all are resources stored in `etcd` and managed via a REST API. |
| **Cattle, Not Pets** | Pods are disposable. Never give them special names and nurse them. Scale them horizontally. |

---

## 3. Architecture: Brain vs Muscle

```
┌──────────────────────────────────────────┐
│           CONTROL PLANE (Brain)          │
│                                          │
│  kube-apiserver  ← Entry point (REST)    │
│  etcd            ← Source of truth (DB)  │
│  kube-scheduler  ← Assigns pods→nodes   │
│  controller-mgr  ← Runs all controllers  │
└──────────────────┬───────────────────────┘
                   │ watches & directs
       ┌───────────┼───────────┐
       ▼           ▼           ▼
 ┌──────────┐ ┌──────────┐ ┌──────────┐
 │ Worker 1 │ │ Worker 2 │ │ Worker 3 │
 │ kubelet  │ │ kubelet  │ │ kubelet  │  ← runs containers
 │ kube-proxy│ │kube-proxy│ │kube-proxy│  ← handles networking
 │ [pods]   │ │ [pods]   │ │ [pods]   │
 └──────────┘ └──────────┘ └──────────┘
```

### Control Plane Components

| Component | Role |
|-----------|------|
| `kube-apiserver` | Front door. Every `kubectl` command hits this REST API. |
| `etcd` | Distributed key-value store — holds ALL cluster state. |
| `kube-scheduler` | Watches unscheduled pods → assigns them to the best node. |
| `kube-controller-manager` | Runs all controllers (Deployment, ReplicaSet, Node, Job…). |
| `cloud-controller-manager` | Talks to cloud providers (AWS/GCP/Azure) for LBs & storage. |

### Worker Node Components

| Component | Role |
|-----------|------|
| `kubelet` | Agent on every node. Ensures pod specs are running & healthy. |
| `kube-proxy` | Maintains iptables/IPVS rules for Service routing. |
| `containerd` / `CRI-O` | Container runtime that actually runs the containers. |

---

## 4. The Reconciliation Loop

> **This is the heart of Kubernetes.** Everything else follows from it.

```
DESIRED STATE              ACTUAL STATE
(what you declared)        (what's running)

  replicas: 3    ──────▶  [reconcile]  ──────▶  3 pods UP

  if actual ≠ desired  →  Kubernetes FIXES IT
```

This loop runs **continuously**, every few seconds, forever.

**Concrete example:**
1. You declare `replicas: 3`
2. A node crashes → only 2 pods running
3. K8s detects: actual (2) ≠ desired (3)
4. K8s starts a new pod on another node ✅

Every K8s component follows the **Watch → Act** pattern:
- Watch `etcd` for changes to their object type
- Act to reconcile actual state toward desired state

---

## 5. Kubernetes Objects Quick Reference

### Workloads

| Object | When to use | Key trait |
|--------|-------------|-----------|
| **Pod** | Never directly in prod — use a controller | Smallest deployable unit; 1+ containers sharing network/storage |
| **Deployment** | Stateless apps (APIs, web servers, workers) | Manages rolling updates & rollbacks via ReplicaSets |
| **StatefulSet** | Databases (Postgres, Mongo, Kafka) | Pods get stable IDs (0, 1, 2) and persistent storage |
| **DaemonSet** | Log collectors, monitoring agents | Runs exactly ONE pod per node |
| **Job** | Batch tasks (DB migrations, reports) | Runs to completion |
| **CronJob** | Scheduled tasks (nightly backups) | Job on a cron schedule |

### Networking

| Object | Purpose | Scope |
|--------|---------|-------|
| **Service (ClusterIP)** | Internal load balancer | Inside cluster only |
| **Service (NodePort)** | Opens port on every node | Dev/test external access |
| **Service (LoadBalancer)** | Provisions cloud LB | Production external access |
| **Ingress** | L7 HTTP/S routing (path & hostname rules) | External, routes to Services |
| **NetworkPolicy** | Pod-level firewall | Internal traffic control |

### Configuration & Storage

| Object | Purpose |
|--------|---------|
| **ConfigMap** | Non-sensitive key-value config (log level, app env) |
| **Secret** | Sensitive data: passwords, API keys (base64 encoded) |
| **PersistentVolume (PV)** | A physical piece of storage |
| **PersistentVolumeClaim (PVC)** | A request for storage by a pod |
| **StorageClass** | Defines the type of storage available (SSD, HDD, cloud) |

### Access Control (RBAC)

| Object | Purpose |
|--------|---------|
| **Namespace** | Virtual cluster — isolates resources by team/env |
| **ServiceAccount** | Identity for a pod talking to the API server |
| **Role / ClusterRole** | Defines allowed actions on resources |
| **RoleBinding** | Links a Role to a ServiceAccount/User |

---

## 6. How a Deployment Actually Works

```
kubectl apply -f deployment.yaml
         │
         ▼
  kube-apiserver  ──validates──▶  etcd  (stores desired state)
         │
         ▼  (watch event)
  kube-controller-manager
    ├─ Deployment Controller → creates ReplicaSet
    └─ ReplicaSet Controller → creates N Pods (unscheduled)
         │
         ▼  (watch event)
  kube-scheduler
    └─ scores nodes → writes nodeName to each pod
         │
         ▼  (watch event on assigned node)
  kubelet
    └─ tells containerd: pull image & start container
         │
         ▼
  kube-proxy
    └─ updates iptables → Service routes traffic to new pods
         │
         ▼
🎉 App is LIVE
```

### Rolling Update (Zero Downtime)

```yaml
# Default update strategy in a Deployment
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1   # never remove > 1 pod at once
    maxSurge: 1         # never have > 1 extra pod
```

```
v1 → v2 update:
  [Pod v1] [Pod v1] [Pod v1]
  [Pod v1] [Pod v1] [Pod v2]   ← new pod healthy
  [Pod v1] [Pod v2] [Pod v2]   ← old pod removed
  [Pod v2] [Pod v2] [Pod v2]   ← done ✅
```

---

## 7. Networking Deep Dive

### Rule 1: Every Pod Gets Its Own IP
No NAT. Pod-to-pod communication is flat and direct.

### Rule 2: Never Route Traffic Directly to Pods
Pods die and get new IPs. Always route via a **Service**.

```
Client
  │
  ▼
Service (stable IP: 10.96.0.100)   ← ClusterIP (internal)
  │
  ├──▶ Pod 1 (10.244.1.2)
  ├──▶ Pod 2 (10.244.2.5)
  └──▶ Pod 3 (10.244.3.8)
```

### Traffic Flow: Local Dev vs Production

```
LOCAL DEV:
  localhost:8080  ──port-forward──▶  Pod  (temporary tunnel, bypasses Service)

PRODUCTION (correct way):
  Internet
    └──▶ Cloud LoadBalancer (public IP)
           └──▶ Ingress Controller (nginx)
                  └──▶ Service (ClusterIP)
                         └──▶ Pods
```

### Ingress: The API Gateway

Ingress lets you serve multiple services under one IP with path/host routing:

```
product.local/api/products   ──▶  product-service:8080
product.local/admin          ──▶  admin-service:3000
```

Requires an **Ingress Controller** (nginx is most common). In minikube:
```bash
minikube addons enable ingress
```

Then update `/etc/hosts`:
```
192.168.49.2  product.local   # use: minikube ip → get the IP
```

---

## 8. Configuration & Secrets

### ConfigMap — Non-Sensitive Config

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: log-level-configmap
data:
  log_level: "Information"
```

### Secret — Sensitive Config

Encode value first:
```bash
echo -n 'my-api-key' | base64
# → bXktYXBpLWtleQ==
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: api-key-secret
type: Opaque
data:
  api_key: bXktYXBpLWtleQ==   # base64 encoded
```

> ⚠️ Secrets are only base64, NOT encrypted. Use **Sealed Secrets** or **HashiCorp Vault** in production.

### Injecting into Pods as Environment Variables

```yaml
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: log-level-configmap
        key: log_level
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: api-key-secret
        key: api_key
```

### Reading in .NET

```csharp
var logLevel = Environment.GetEnvironmentVariable("LOG_LEVEL");
var apiKey   = Environment.GetEnvironmentVariable("API_KEY");
```

---

## 9. Scaling

### Manual Scaling

```bash
# Scale up
kubectl scale --replicas=5 deployment/product

# Scale back down
kubectl scale --replicas=1 deployment/product
```

Deleting a pod managed by a Deployment just causes K8s to recreate it:
```bash
kubectl delete pod product-abc123   # K8s immediately creates a replacement
```

### Horizontal Pod Autoscaler (HPA)

Automatically scales based on CPU/memory usage:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: product-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: product
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

### KEDA — Event-Driven Autoscaling

Scale based on Kafka queue length, RabbitMQ depth, or any custom metric:
```yaml
kind: ScaledObject
spec:
  scaleTargetRef:
    name: consumer-deployment
  triggers:
    - type: kafka
      metadata:
        topic: orders
        lagThreshold: "10"
```

---

## 10. Health Checks (Probes)

```yaml
livenessProbe:     # Is the container ALIVE? → restarts if fails
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

readinessProbe:    # Ready to receive traffic? → removed from Service if fails
  httpGet:
    path: /ready
    port: 8080

startupProbe:      # Slow-starting apps → disables liveness until app starts
  httpGet:
    path: /started
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

| Probe | Failure Action |
|-------|----------------|
| **Liveness** | Container is killed and restarted |
| **Readiness** | Pod removed from Service endpoints (traffic stops) |
| **Startup** | Liveness/Readiness paused until startup succeeds |

---

## 11. Storage

Pods are stateless by default — data is lost when a pod dies.

**Flow:**
```
StorageClass (defines type)
    └──▶ PersistentVolume / dynamic provisioning
              └──▶ PersistentVolumeClaim (your request)
                        └──▶ Pod mounts PVC as a volume
                                  → data survives pod restarts ✅
```

```yaml
# PVC example
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

Use **StatefulSet** (not Deployment) when pods need their own dedicated storage.

---

## 12. Helm

Helm is the **package manager for Kubernetes** — like `apt` or `npm`, but for K8s manifests.

### Core Concepts

| Term | Meaning |
|------|---------|
| **Chart** | A packaged K8s application (collection of templates + defaults) |
| **Release** | A deployed instance of a chart in your cluster |
| **Repository** | A collection of charts (like ArtifactHub) |
| **Values** | Variables that customize a chart at install time |

### Common Commands

```bash
# Add a chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo wordpress

# Install a chart (creates a "release")
helm install my-release bitnami/wordpress

# Install with custom values
helm install my-release bitnami/wordpress \
  --set wordpressUsername=admin \
  --set wordpressPassword=secret

# List installed releases
helm list

# Upgrade an existing release
helm upgrade my-release bitnami/wordpress

# Uninstall a release (deletes all K8s resources)
helm uninstall my-release

# Create a new chart scaffold
helm create productservice
```

### Helm Chart Structure

```
productservice/
  Chart.yaml          ← chart metadata (name, version, description)
  values.yaml         ← default configuration values
  templates/
    deployment.yaml   ← templated K8s manifests
    service.yaml
    ingress.yaml
    hpa.yaml
    _helpers.tpl      ← reusable template functions
```

### Template Syntax

```yaml
# templates/deployment.yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
replicas: {{ .Values.replicaCount }}
```

```yaml
# values.yaml
replicaCount: 3
image:
  repository: ghcr.io/shafiktanbir/productservice
  tag: "latest"
```

---

## 13. Essential kubectl Commands

### Cluster & Context

```bash
kubectl config get-contexts         # list clusters
kubectl config use-context <name>   # switch cluster
kubectl cluster-info                # cluster endpoint info
```

### Pods

```bash
kubectl get pods                    # list pods (default namespace)
kubectl get pods -A                 # all namespaces
kubectl get pods -o wide            # include node/IP info
kubectl get pods -w                 # watch (live updates)

kubectl describe pod <name>         # full details + events (best for debugging)
kubectl logs <name>                 # application logs
kubectl logs <name> -f              # follow logs live
kubectl logs <name> --previous      # logs from crashed container

kubectl exec -it <name> -- /bin/sh  # shell into running pod
kubectl delete pod <name>           # delete (controller recreates it)
```

### Deployments

```bash
kubectl get deployments
kubectl describe deployment <name>

kubectl apply -f deployment.yaml    # create or update
kubectl delete -f deployment.yaml   # delete

kubectl scale deployment <name> --replicas=5
kubectl rollout restart deployment/<name>   # rolling restart
kubectl rollout status deployment/<name>    # watch rollout progress
kubectl rollout undo deployment/<name>      # rollback to previous version
kubectl rollout history deployment/<name>   # view revision history
```

### Services & Networking

```bash
kubectl get services                # list services
kubectl get svc                     # shorthand
kubectl describe service <name>

# Create a temporary tunnel (dev only)
kubectl port-forward pod/<name> 8080:8080
kubectl port-forward service/<name> 7080:8080
```

### Config & Secrets

```bash
kubectl get configmaps
kubectl get cm                      # shorthand
kubectl describe configmap <name>
kubectl create configmap <name> --from-literal=KEY=VALUE

kubectl get secrets
kubectl describe secret <name>
```

### General Debugging

```bash
kubectl get all                              # everything in namespace
kubectl get all -A                           # everything cluster-wide
kubectl get events --sort-by='.lastTimestamp'  # sorted events (very useful)
kubectl top pods                             # resource usage (needs metrics-server)
kubectl top nodes

# Run a temporary debug pod
kubectl run debug --image=busybox --rm -it --restart=Never -- sh
```

### Minikube Specific

```bash
minikube start / stop / status / delete
minikube dashboard                  # open K8s dashboard in browser
minikube ip                         # get cluster IP
minikube service <name>             # open service in browser (tunnels LoadBalancer)
minikube service <name> --url       # just print the URL
minikube addons enable ingress      # enable nginx ingress controller
minikube image load <image>:<tag>   # load local image (skips registry push)
```

---

## 14. Real YAML Examples

All YAML files below come from the `orchestrators/lecture142/k8s/product.yaml` file in this repo.

### Complete Deployment + Service + Ingress + ConfigMap + Secret

```yaml
# Deployment — runs 3 replicas of the product microservice
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product
spec:
  replicas: 3
  selector:
    matchLabels:
      app: product              # must match pod template labels
  template:
    metadata:
      labels:
        app: product
    spec:
      containers:
      - name: product
        image: ghcr.io/shafiktanbir/productservice:latest
        ports:
        - containerPort: 8080
        env:
        - name: LOG_LEVEL       # injected from ConfigMap
          valueFrom:
            configMapKeyRef:
              name: log-level-configmap
              key: log_level
        - name: API_KEY         # injected from Secret
          valueFrom:
            secretKeyRef:
              name: api-key-secret
              key: api_key
---
# Service — stable internal endpoint for the pods
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  selector:
    app: product               # routes to pods with this label
  ports:
    - protocol: TCP
      port: 8080               # port the Service listens on
      targetPort: 8080         # port the container listens on
---
# Ingress — routes external HTTP traffic to the Service
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: product-ingress
spec:
  rules:
  - host: product.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: product-service
            port:
              number: 8080
---
# ConfigMap — non-sensitive config
apiVersion: v1
kind: ConfigMap
metadata:
  name: log-level-configmap
data:
  log_level: "Information"
---
# Secret — sensitive config (base64 encoded)
apiVersion: v1
kind: Secret
metadata:
  name: api-key-secret
type: Opaque
data:
  api_key: cHJvZHVjdC1hcGkta2V5   # echo -n 'product-api-key' | base64
```

### Apply & Verify

```bash
kubectl apply -f product.yaml
kubectl get all
kubectl get ingress
```

---

## 15. Decision Tree

**"Which K8s object do I need?"**

```
Is it a long-running process?
  ├─ Yes → Is it stateless?               → Deployment
  ├─ Yes → Does it need unique ID/storage? → StatefulSet
  └─ Yes → Must run on every node?         → DaemonSet

Is it a one-off or scheduled task?
  ├─ Run once → Job
  └─ On a schedule → CronJob

Does it need to be reachable?
  ├─ From inside the cluster only?    → Service (ClusterIP)
  ├─ From outside via HTTP/HTTPS?     → Ingress (+ ClusterIP Service)
  └─ From outside via TCP/UDP?        → Service (LoadBalancer)

How do I pass configuration?
  ├─ Non-sensitive (log level, env)?  → ConfigMap
  └─ Sensitive (password, API key)?   → Secret

Does the pod need persistent storage?
  └─ Yes → PersistentVolumeClaim + StatefulSet

Do I want to scale automatically?
  ├─ Based on CPU/memory?             → HorizontalPodAutoscaler (HPA)
  └─ Based on queue depth or events?  → KEDA ScaledObject
```

---

## 16. Debugging Playbook

### Pod Won't Start

```bash
kubectl get pods              # check STATUS (Pending, CrashLoopBackOff, ImagePullBackOff)
kubectl describe pod <name>   # look at the EVENTS section at the bottom
kubectl logs <name>           # app logs
kubectl logs <name> --previous  # logs from last crash
```

| Status | Likely cause |
|--------|-------------|
| `Pending` | Not enough resources on any node, or PVC not bound |
| `ImagePullBackOff` | Wrong image name/tag, or registry auth missing |
| `CrashLoopBackOff` | App is crashing at startup — check logs |
| `OOMKilled` | Container exceeded memory limit |

### Service Not Reachable

```bash
kubectl get endpoints <service-name>   # check if pods are listed
kubectl describe service <name>        # verify selector matches pod labels
```

If `Endpoints` is empty: the Service selector doesn't match any pod labels.

### General Investigation Order

```
1. kubectl get pods          → Is it running?
2. kubectl describe pod      → Events, resource limits, node assignment
3. kubectl logs <pod>        → App-level errors
4. kubectl get endpoints     → Is Service routing correctly?
5. kubectl get events        → Cluster-wide events sorted by time
```

---

## 17. Learning Roadmap

Use this as a structured path to go from zero to production-ready K8s.

### 🟢 Beginner (Week 1–2)

| Topic | How to practise | Lecture in this repo |
|-------|-----------------|----------------------|
| Install minikube & kubectl | `minikube start` | lecture131 |
| Run your first pod | `kubectl apply -f pod.yaml` | lecture134 |
| Understand Deployments | Scale up/down | lecture137 |
| Expose with a Service | `kubectl port-forward` | lecture138–139 |
| Combine in one YAML | Multi-resource `---` files | lecture140 |

### 🟡 Intermediate (Week 3–4)

| Topic | How to practise | Lecture in this repo |
|-------|-----------------|----------------------|
| Ingress & custom DNS | Enable nginx addon, update `/etc/hosts` | lecture141 |
| ConfigMaps & Secrets | Inject env vars from ConfigMap/Secret | lecture142 |
| Manual scaling & self-healing | `kubectl scale` + delete pods | lecture143 |
| Probes (liveness/readiness) | Add probes to your Deployment | HOW_KUBERNETES_WORKS.md |
| Persistent Storage | PVC + StatefulSet | K8S_OBJECTS_MENTAL_MAP.md |

### 🔴 Advanced (Week 5+)

| Topic | Resources |
|-------|-----------|
| Helm charts (install & create) | lecture153–154 |
| HPA (CPU-based autoscaling) | `kubectl autoscale deployment` |
| KEDA (event-driven scaling) | lecture in `scalability/` folder |
| Istio service mesh | `orchestrators/` + Istio docs |
| GitOps with ArgoCD | `devops_cicd/` folder |
| Observability (Prometheus, Grafana, Jaeger) | `monitoring/` folder |

### 📖 Recommended External Resources

| Resource | URL | Best for |
|----------|-----|----------|
| Official Kubernetes Docs | https://kubernetes.io/docs | Reference |
| Play with Kubernetes | https://labs.play-with-k8s.com | Free browser cluster |
| Kubernetes the Hard Way | https://github.com/kelseyhightower/kubernetes-the-hard-way | Deep internals |
| ArtifactHub | https://artifacthub.io | Finding Helm charts |
| CNCF Landscape | https://landscape.cncf.io | Ecosystem overview |
| Nigel Poulton — The Kubernetes Book | https://nigelpoulton.com/books/ | Best beginner book |

---

## 🔑 Key Mental Models (Memorise These)

1. **Watch → Act** — Every K8s component watches `etcd` and acts on changes
2. **Describe the outcome, not the steps** — Declarative > imperative
3. **Pods are cattle, not pets** — They die, they restart, they get new IPs
4. **Services give stable addresses** — Never hardcode pod IPs
5. **Controllers are the real workers** — `kubectl` just writes to `etcd`; controllers do the actual work
6. **etcd is the single source of truth** — If it's not in etcd, K8s doesn't know about it
7. **Namespaces are soft boundaries** — Use them to separate environments (dev/staging/prod)

---

*📂 Explore the `orchestrators/` folder for hands-on YAML files and step-by-step lecture notes.*
*📊 See `graphify-out/GRAPH_REPORT.md` for a knowledge graph of the entire project.*
