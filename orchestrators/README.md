# ☸️ Kubernetes — Philosophy & Architecture

> *"Kubernetes is not just a container orchestrator — it's a platform for building platforms."*
> — Kelsey Hightower

---

## 🧠 The Philosophy of Kubernetes

Kubernetes (K8s) was born out of Google's internal system called **Borg**, which managed billions of containers per week. The core philosophy is rooted in several key principles:

### 1. 🎯 Desired State / Declarative Model
You **declare what you want** (e.g., "I want 3 replicas of my app running"), and Kubernetes continuously works to make reality match that desired state. You never manually start/stop containers — you describe the outcome.

```yaml
# You say: "I want 3 nginx pods"
replicas: 3
```
Kubernetes figures out *how* to achieve it.

### 2. 🔁 Self-Healing
If a pod crashes, a node fails, or a container becomes unhealthy, Kubernetes **automatically restores** the desired state. No human intervention needed.

### 3. 📦 Immutable Infrastructure
Containers are never modified in-place. Instead, you build a **new image**, deploy it, and Kubernetes replaces old pods with new ones (rolling updates). This eliminates "works on my machine" problems.

### 4. 🧩 Everything is an API Object
Pods, Services, Deployments, ConfigMaps — everything in Kubernetes is a **first-class API resource** stored in etcd and managed through a unified API. This makes the system highly extensible.

### 5. 🔒 Cattle, Not Pets
Workloads are treated as **disposable** and **interchangeable**. If a pod dies, a new one is created. You don't name your pods and nurse them — you scale them.

### 6. 🌐 Platform for Platforms
Kubernetes provides the primitives (scheduling, networking, storage, config management) that allow teams to build their own **internal developer platforms** on top of it.

---

## 🏗️ Kubernetes Architecture

```
┌──────────────────────────────────────────────────────────┐
│                     CONTROL PLANE                        │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐ │
│  │ kube-apiserver│  │   etcd       │  │ kube-scheduler  │ │
│  └─────────────┘  └──────────────┘  └─────────────────┘ │
│  ┌───────────────────────────────────────────────────┐   │
│  │         kube-controller-manager                   │   │
│  └───────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
   ┌────────────┐  ┌────────────┐  ┌────────────┐
   │  Worker    │  │  Worker    │  │  Worker    │
   │  Node 1    │  │  Node 2    │  │  Node 3    │
   │            │  │            │  │            │
   │ kubelet    │  │ kubelet    │  │ kubelet    │
   │ kube-proxy │  │ kube-proxy │  │ kube-proxy │
   │ [pods...]  │  │ [pods...]  │  │ [pods...]  │
   └────────────┘  └────────────┘  └────────────┘
```

---

## 🧩 Core Components

### 🖥️ Control Plane (The Brain)

| Component | Role |
|-----------|------|
| **kube-apiserver** | The front door — all kubectl commands hit this REST API. Every component talks through it. |
| **etcd** | The cluster's "database" — a distributed key-value store holding ALL cluster state (desired & actual). |
| **kube-scheduler** | Watches for new pods and assigns them to the best available Node based on resources, affinity, taints, etc. |
| **kube-controller-manager** | Runs all controllers in a single process (Node controller, ReplicaSet controller, Deployment controller, etc.). |
| **cloud-controller-manager** | Bridges K8s with cloud provider APIs (AWS, GCP, Azure) for load balancers, storage, etc. |

---

### ⚙️ Worker Nodes (The Muscle)

| Component | Role |
|-----------|------|
| **kubelet** | The agent on each node. Ensures containers described in PodSpecs are running and healthy. |
| **kube-proxy** | Maintains network rules on each node. Implements the `Service` abstraction (routes traffic to pods). |
| **Container Runtime** | The engine that actually runs containers. Supports `containerd`, `CRI-O`, or Docker (via shim). |

---

## 📦 Core Kubernetes Objects (API Resources)

### Workloads

| Object | Purpose |
|--------|---------|
| **Pod** | The smallest deployable unit. One or more containers sharing network & storage. |
| **ReplicaSet** | Ensures N copies of a Pod are always running. |
| **Deployment** | Manages ReplicaSets. Enables rolling updates & rollbacks. |
| **StatefulSet** | Like Deployment but for stateful apps (databases). Pods have stable network identities. |
| **DaemonSet** | Runs a pod on **every** node (e.g., log collectors, monitoring agents). |
| **Job** | Runs a pod to completion (batch tasks). |
| **CronJob** | Runs Jobs on a schedule (like Unix cron). |

### Networking

| Object | Purpose |
|--------|---------|
| **Service** | Stable network endpoint for a group of pods. Types: `ClusterIP`, `NodePort`, `LoadBalancer`. |
| **Ingress** | HTTP/HTTPS routing rules. Maps external URLs to internal Services. |
| **NetworkPolicy** | Firewall rules for pod-to-pod communication. |

### Configuration & Storage

| Object | Purpose |
|--------|---------|
| **ConfigMap** | Store non-sensitive config data as key-value pairs. |
| **Secret** | Store sensitive data (passwords, tokens) — base64 encoded. |
| **PersistentVolume (PV)** | A piece of storage provisioned in the cluster. |
| **PersistentVolumeClaim (PVC)** | A request for storage by a pod. |
| **StorageClass** | Defines the type of storage to dynamically provision. |

### Access Control

| Object | Purpose |
|--------|---------|
| **Namespace** | Virtual cluster within a cluster — isolates resources. |
| **ServiceAccount** | Identity for pods to authenticate with the API server. |
| **Role / ClusterRole** | Defines what actions are allowed on which resources. |
| **RoleBinding / ClusterRoleBinding** | Grants a Role to a user or ServiceAccount. |

---

## 🔄 How It All Works Together — Pod Lifecycle

```
kubectl apply -f deployment.yaml
        │
        ▼
  kube-apiserver  ──── stores in ────►  etcd
        │
        ▼
  kube-controller-manager
  (Deployment Controller creates ReplicaSet → ReplicaSet creates Pod)
        │
        ▼
  kube-scheduler  ──── assigns Pod to Node
        │
        ▼
  kubelet on Node  ──── pulls image & starts container
        │
        ▼
  kube-proxy  ──── sets up network rules for Service routing
```

---

## 🛠️ Tools in This Section

| Tool | Purpose |
|------|---------|
| `minikube` | Local single-node Kubernetes cluster for development |
| `kubectl` | CLI to interact with any Kubernetes cluster |
| `helm` | Package manager for Kubernetes (like apt/npm for K8s) |

---

## 📚 Lectures Index

| Lecture | Topic |
|---------|-------|
| lecture131 | Getting started with Kubernetes & minikube |
| lecture132 | Deploying apps with kubectl & Docker |
| lecture133–145 | Core Kubernetes objects & configurations |
| lecture153–154 | Advanced topics |

---

## 🔑 Key Takeaways

- Kubernetes is **declarative** — describe the end state, not the steps
- The **control plane** manages the cluster; **worker nodes** run your workloads
- Everything is an **API object** stored in **etcd**
- Kubernetes constantly **reconciles** actual state → desired state
- Use **Deployments** for stateless apps, **StatefulSets** for databases

---

*📖 Official Docs: [kubernetes.io/docs](https://kubernetes.io/docs)*
