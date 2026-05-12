# 🧠 Kubernetes Objects: Senior Engineer's Mental Map

This guide serves as a high-level reference for architecting and troubleshooting Kubernetes environments. It categorizes objects by their **operational intent** rather than just their technical definition.

---

## 1. Workload Orchestration (The "Running Code")

| Object | Mental Model | When to use it? |
| :--- | :--- | :--- |
| **Pod** | The atomic unit. A "logical host" for containers. | Never directly in production. Use a controller. |
| **Deployment** | **Stateless.** Disposable, interchangeable replicas. | Your standard API, Web App, or Worker. |
| **StatefulSet** | **Identity Matters.** Stable network IDs (0, 1, 2) and persistent storage mapping. | Databases (Postgres, Mongo), Message Brokers (Kafka). |
| **DaemonSet** | **Node-bound.** Exactly one copy per node. | Logging agents (Fluentbit), Monitoring (Prometheus Node Exporter). |
| **Job / CronJob** | **Run to Completion.** Batch processing or scheduled tasks. | Database migrations, nightly backups, report generation. |

> [!TIP]
> **Senior Insight:** Always prefer `Deployments` unless you have a specific reason for pod identity (`StatefulSet`) or node locality (`DaemonSet`).

---

## 2. Service Discovery & Connectivity (The "Traffic Flow")

| Object | Mental Model | Interaction Pattern |
| :--- | :--- | :--- |
| **Service (ClusterIP)** | **Internal Load Balancer.** Stable IP/DNS for internal pods. | Default for microservice-to-microservice talk. |
| **Service (NodePort)** | **External Hole.** Opens a port on every node. | Legacy or simple dev environments. Avoid in Cloud. |
| **Service (LoadBalancer)**| **Cloud Integration.** Provisions a Cloud Provider LB. | Exposing a service directly to the internet. |
| **Ingress** | **Smart Gateway.** L7 Routing (HTTP/HTTPS), SSL termination, Path routing. | Routing `api.myapp.com/v1` to Service A and `/v2` to Service B. |
| **Endpoint / EndpointSlice** | **The Directory.** The actual list of Pod IPs behind a Service. | Troubleshooting: "Service exists but no pods are listed in endpoints." |

---

## 3. Configuration & State (The "Environment")

| Object | Mental Model | Best Practice |
| :--- | :--- | :--- |
| **ConfigMap** | **Non-sensitive Config.** Env vars, config files. | Mount as volumes for large configs; Env vars for simple flags. |
| **Secret** | **Sensitive Config.** API keys, DB passwords, TLS certs. | Never commit to Git. Use external managers (Vault/SealedSecrets). |
| **Namespace** | **Virtual Isolation.** Soft multi-tenancy. | Group by project or environment (dev, staging, prod). |

---

## 4. Persistent Storage (The "Memory")

| Object | Mental Model | Analogy |
| :--- | :--- | :--- |
| **StorageClass (SC)** | **The Menu.** Defines *what kind* of storage is available. | "I want SSD storage from AWS." |
| **PersistentVolume (PV)** | **The Physical Disk.** A specific piece of storage. | "Here is a 20GB EBS volume." |
| **PersistentVolumeClaim (PVC)** | **The Ticket.** A request for storage by a user/pod. | "I need 10GB of SSD storage." |

> [!NOTE]
> **The Flow:** SC → (Dynamic Provisioning) → PV → (Bound to) → PVC → (Mounted to) → Pod.

---

## 5. Security & Governance (The "Guardrails")

| Object | Mental Model | Question it answers |
| :--- | :--- | :--- |
| **ServiceAccount** | **Pod Identity.** The "User" account for a process. | "Who is this pod talking to the API?" |
| **Role / ClusterRole** | **Permissions.** A list of "Allowed Actions". | "Can I list pods? Can I delete secrets?" |
| **RoleBinding** | **The Link.** Connects an Identity to a Role. | "Give the 'Viewer' role to 'Pod-A's ServiceAccount'." |
| **NetworkPolicy** | **Pod Firewall.** L3/L4 traffic rules. | "Only the Backend can talk to the Database." |

---

## 🏗️ Senior Engineering Decision Tree

1.  **Is it a long-running process?**
    *   Yes → Is it stateless? → **Deployment**
    *   Yes → Does it need a unique ID/Storage? → **StatefulSet**
    *   Yes → Does it need to run on every node? → **DaemonSet**
2.  **Does it need to be reachable?**
    *   Inside the cluster? → **Service (ClusterIP)**
    *   Outside the cluster (HTTP)? → **Ingress**
    *   Outside the cluster (TCP/UDP)? → **Service (LoadBalancer)**
3.  **How do I pass config?**
    *   Static/Public? → **ConfigMap**
    *   Sensitive? → **Secret**
    *   Large file? → **ConfigMap (Mounted as Volume)**

---

## 🛠️ The "Swiss Army Knife" Command (Reference)

```bash
# The 'Explain' command is a senior's best friend to find field definitions
kubectl explain pod.spec.containers.resources

# Check all resources across all namespaces
kubectl get all -A

# Identify "Leaking" or "Pending" storage
kubectl get pvc,pv
```
