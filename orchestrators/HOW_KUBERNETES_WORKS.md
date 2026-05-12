# ⚙️ How Kubernetes Works — Deep Dive

> Kubernetes doesn't manage containers. It manages **desired state** and uses containers as the mechanism to achieve it.

---

## 🧭 The Big Picture

When you deploy an app to Kubernetes, here's the simplified mental model:

```
You (Developer)
    │
    │  kubectl apply -f app.yaml
    ▼
┌─────────────────────────────────────────────┐
│              KUBERNETES CLUSTER             │
│                                             │
│   ┌──────────────┐     ┌─────────────────┐ │
│   │ Control Plane│────▶│   Worker Nodes  │ │
│   │  (The Brain) │     │  (The Muscle)   │ │
│   └──────────────┘     └─────────────────┘ │
└─────────────────────────────────────────────┘
```

The **Control Plane** decides WHAT runs WHERE.  
The **Worker Nodes** actually RUN the containers.

---

## 🔄 The Core Concept: The Reconciliation Loop

This is the **heart** of Kubernetes. Everything in K8s follows this pattern:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   DESIRED STATE          ACTUAL STATE               │
│   (what you want)        (what's running)           │
│                                                     │
│   replicas: 3    ──▶  [reconcile]  ──▶  3 pods up  │
│                                                     │
│   if actual ≠ desired → Kubernetes FIXES IT         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Example:**
- You say: `replicas: 3`
- A node crashes → only 2 pods running
- K8s detects: actual (2) ≠ desired (3)
- K8s automatically starts a new pod on another node ✅

This loop runs **forever**, every few seconds.

---

## 📋 Step-by-Step: What Happens When You Run `kubectl apply`

### Step 1 — You Submit a Manifest
```bash
kubectl apply -f deployment.yaml
```
`kubectl` sends a **REST API call** (HTTP POST/PATCH) to the `kube-apiserver`.

---

### Step 2 — API Server Validates & Stores
```
kubectl ──HTTP──▶ kube-apiserver ──writes──▶ etcd
```
- The API server **validates** your YAML (correct schema? proper permissions?)
- If valid, it **stores the desired state** in `etcd`
- `etcd` is the single source of truth for the entire cluster

---

### Step 3 — Controllers Watch for Changes
```
etcd ──event──▶ Deployment Controller
                    │
                    ▼
             "I see a new Deployment!
              Let me create a ReplicaSet"
                    │
                    ▼
             ReplicaSet Controller
                    │
                    ▼
             "I see a new ReplicaSet!
              Let me create 3 Pods"
```

Controllers run inside `kube-controller-manager`. Each controller has ONE job:
- `Deployment Controller` → manages Deployments
- `ReplicaSet Controller` → ensures correct number of pods
- `Node Controller` → monitors node health
- `Job Controller` → manages one-off tasks

---

### Step 4 — Scheduler Assigns Pods to Nodes

New pods are created but have **no node assigned yet**. The scheduler watches for these "unscheduled" pods.

```
kube-scheduler watches for: pods with nodeName = ""
        │
        ▼
   Scoring Algorithm:
   - How much CPU/RAM does the pod need?
   - Which nodes have enough resources?
   - Any affinity/anti-affinity rules?
   - Any taints/tolerations?
        │
        ▼
   Best node selected → writes nodeName to pod spec in etcd
```

---

### Step 5 — Kubelet Starts the Container

The `kubelet` on the selected node is always watching etcd for pods assigned to ITS node.

```
etcd ──event──▶ kubelet on Node-2
                    │
                    ▼
             "A pod is assigned to me!
              Let me start it."
                    │
                    ▼
             Talks to Container Runtime (containerd)
                    │
                    ▼
             Container Runtime pulls image from registry
                    │
                    ▼
             Container starts running 🚀
                    │
                    ▼
             kubelet reports status back to API Server
```

---

### Step 6 — kube-proxy Sets Up Networking

Once a pod is running, `kube-proxy` on each node updates **iptables/IPVS rules** so that traffic can reach the pod via its Service.

```
Service (ClusterIP: 10.96.0.1:80)
        │
        ▼ (iptables rule by kube-proxy)
Pod A (192.168.1.5:8080)  OR
Pod B (192.168.1.6:8080)  OR
Pod C (192.168.1.7:8080)
```

---

## 🌐 Kubernetes Networking — How Pods Talk

### Rule 1: Every Pod Gets Its Own IP
No NAT between pods. Pod-to-pod communication is **flat and direct**.

```
Pod A (10.244.1.2) ──────▶ Pod B (10.244.2.5)
        Direct connection, no proxy needed
```

### Rule 2: Services Provide Stable IPs
Pods are ephemeral — they can die and get new IPs. **Services** give you a stable endpoint.

```
Client ──▶ Service (stable IP: 10.96.0.100)
                │
                ├──▶ Pod 1 (10.244.1.2)
                ├──▶ Pod 2 (10.244.2.5)
                └──▶ Pod 3 (10.244.3.8)
```

### Types of Services

| Type | Access Scope | Use Case |
|------|-------------|----------|
| `ClusterIP` | Inside cluster only | Internal microservices |
| `NodePort` | Via node's IP:Port | Dev/testing external access |
| `LoadBalancer` | External cloud LB | Production external access |
| `ExternalName` | DNS alias | Point to external service |

---

## 💾 How Kubernetes Handles Storage

Pods are stateless by default — if a pod dies, its data is gone.

```
PersistentVolumeClaim (PVC)  ──binds──▶  PersistentVolume (PV)
        │                                        │
        │                                        ▼
        │                               Actual Storage
        │                         (disk, NFS, cloud block storage)
        │
    Pod mounts PVC as a volume
    → data survives pod restarts ✅
```

**Flow:**
1. Admin creates a `PersistentVolume` (or StorageClass auto-provisions it)
2. Developer creates a `PersistentVolumeClaim` (requests X GB of storage)
3. K8s binds PVC → PV
4. Pod mounts the PVC — data is now persistent

---

## 🔄 Rolling Updates — Zero Downtime Deployments

When you update your image (e.g., `v1` → `v2`):

```
Initial State:  [Pod v1] [Pod v1] [Pod v1]

Step 1:         [Pod v1] [Pod v1] [Pod v2]  ← new pod created
Step 2:         [Pod v1] [Pod v2] [Pod v2]  ← old pod removed
Step 3:         [Pod v2] [Pod v2] [Pod v2]  ← update complete ✅
```

Kubernetes ensures:
- `maxUnavailable: 1` → never take down more than 1 pod at a time
- `maxSurge: 1` → never have more than 1 extra pod
- Health checks pass before old pods are removed

**Rollback is instant:**
```bash
kubectl rollout undo deployment/my-app
```

---

## 🏥 Health Checks — How K8s Knows if Your App is OK

```yaml
livenessProbe:   # Is the container ALIVE? (restart if fails)
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

readinessProbe:  # Is the container READY to receive traffic?
  httpGet:
    path: /ready
    port: 8080

startupProbe:    # Is the app done starting up? (for slow apps)
  httpGet:
    path: /started
    port: 8080
```

| Probe | Fails → Action |
|-------|---------------|
| **Liveness** | Container is restarted |
| **Readiness** | Pod removed from Service endpoints (no traffic sent) |
| **Startup** | Liveness/Readiness disabled until startup passes |

---

## ⚖️ How the Scheduler Picks a Node

The scheduler uses a **two-phase algorithm**:

### Phase 1 — Filtering (Hard Rules)
Eliminates nodes that CANNOT run the pod:
- Not enough CPU/RAM
- Node has a taint the pod doesn't tolerate
- Node affinity rules don't match
- Volume zone constraints

### Phase 2 — Scoring (Soft Rules)
Ranks remaining nodes:
- **LeastRequestedPriority** → prefer nodes with more free resources
- **BalancedResourceAllocation** → balance CPU vs memory usage
- **NodeAffinityPriority** → prefer nodes matching affinity hints

```
All Nodes: [Node1] [Node2] [Node3] [Node4]
After Filter: [Node2] [Node3]        ← Node1 & Node4 eliminated
After Score:  Node3 wins (score: 90) ← Pod assigned to Node3
```

---

## 🔐 How Kubernetes Handles Configuration & Secrets

```
ConfigMap (non-sensitive)          Secret (sensitive)
─────────────────────────          ──────────────────
DB_HOST=postgres-svc               DB_PASSWORD=s3cr3t
APP_ENV=production                 API_KEY=abc123
LOG_LEVEL=info                     TLS_CERT=...
        │                                  │
        └──────────────┬───────────────────┘
                       ▼
               Pod consumes as:
               - Environment variables
               - Mounted files/volumes
```

**Secrets are base64-encoded** (NOT encrypted by default). Use **Sealed Secrets** or **Vault** for production.

---

## 📊 Full End-to-End Flow Summary

```
Developer
   │ kubectl apply -f deployment.yaml
   ▼
kube-apiserver ──validates──▶ etcd (stores desired state)
   │
   ▼ (watch event)
kube-controller-manager
   ├── Deployment Controller → creates ReplicaSet
   └── ReplicaSet Controller → creates 3 Pods (unscheduled)
   │
   ▼ (watch event)
kube-scheduler
   └── scores nodes → assigns nodeName to each pod
   │
   ▼ (watch event on each node)
kubelet (on assigned node)
   └── tells container runtime to pull image & start container
   │
   ▼
kube-proxy
   └── updates iptables rules so Service routes traffic to new pods
   │
   ▼
🎉 Your app is LIVE and receiving traffic!
```

---

## 🆚 Kubernetes vs Traditional Deployment

| Aspect | Traditional (VM/Bare Metal) | Kubernetes |
|--------|----------------------------|------------|
| Scaling | Manual — SSH in, start process | `kubectl scale --replicas=10` |
| Failure recovery | Manual — ops team paged | Automatic — K8s restarts pod |
| Deployment | Downtime window needed | Rolling update, zero downtime |
| Config management | SSH + env files | ConfigMap + Secrets |
| Service discovery | Hardcoded IPs / DNS | Built-in DNS via CoreDNS |
| Load balancing | Nginx/HAProxy config | Service object built-in |

---

## 🎯 Key Mental Models to Remember

1. **Watch → Act** — Every K8s component watches etcd and acts on changes
2. **Level-triggered, not edge-triggered** — K8s doesn't respond to events once; it continuously checks state
3. **Optimistic concurrency** — Multiple controllers can watch the same object; `resourceVersion` prevents conflicts
4. **Everything has a controller** — Every API object type has a controller loop managing it
5. **etcd is the source of truth** — If it's not in etcd, K8s doesn't know about it

---

*📖 Further Reading:*
- *[Kubernetes Internals](https://github.com/shubheksha/kubernetes-internals)*
- *[The Kubernetes Book](https://nigelpoulton.com/books/)*
- *[Official Concepts](https://kubernetes.io/docs/concepts/)*
