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

---

## 🐞 Debugging & Troubleshooting Commands

When things go wrong, these are the essential commands to find the root cause.

| Command | Purpose | When to use it? |
| :--- | :--- | :--- |
| `kubectl get pods` | Status check | "Is my pod running, crashing, or stuck in Pending?" |
| `kubectl describe pod <name>` | Detailed events | "Why is my pod stuck in `ImagePullBackOff` or `Pending`?" Look at the bottom 'Events' section. |
| `kubectl logs <name>` | Application logs | "My pod is Running, but the app is throwing 500 errors." |
| `kubectl logs <name> --previous` | Previous crash logs | "My pod crashed and restarted. I need to see why the *last* one died." |
| `kubectl exec -it <name> -- sh` | Shell access | "I need to look at the filesystem or test internal network inside the running container." |
| `kubectl port-forward pod/<name> 8080:80` | Local testing tunnel | "I want to access this pod from my local browser without setting up a Service." (NEVER use in production) |

---

## 🌐 Networking: Local vs. Production

Understanding how traffic flows to your pods is crucial for moving from local development to production.

### Local Development / Debugging
*   **Port-Forwarding (`kubectl port-forward`)**: Creates a temporary tunnel from your local machine (e.g., `localhost:8080`) directly to a specific Pod.
*   **Why?** It bypasses all Kubernetes routing (Services, Ingress). It is strictly for testing and debugging. If the pod dies, the connection breaks.

### Production (The Right Way)
In production, Pods are ephemeral (they die and get new IPs automatically). You never route traffic directly to a Pod.

1.  **Service (ClusterIP)**: Gives a **permanent internal IP** to a group of Pods. Microservices talk to each other using this Service name (e.g., `http://product-service:8080`).
2.  **Service (LoadBalancer)**: Tells the Cloud Provider (AWS, Azure, GCP) to create a **physical Cloud Load Balancer** with a Public IP, and points external traffic to your Pods.
3.  **Ingress**: The **API Gateway**. It usually sits behind a LoadBalancer and routes traffic based on URLs (e.g., `api.example.com` goes to the API Service, `app.example.com` goes to the Frontend Service). This is much cheaper than creating a LoadBalancer for every microservice.
