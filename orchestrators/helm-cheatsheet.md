# ⛵ Helm Cheat Sheet

> **Helm = The package manager for Kubernetes**
> Think of it like `apt` / `brew` / `npm` — but for Kubernetes applications.

---

## 🧠 Philosophy of Helm

### What problem does Helm solve?
Without Helm, deploying an app to Kubernetes means writing and managing many raw YAML files:
- `deployment.yaml`
- `service.yaml`
- `configmap.yaml`
- `ingress.yaml`
- `hpa.yaml`
- `serviceaccount.yaml`
- ...and more

These files have **hardcoded values**, making it painful to:
- Deploy the same app to different environments (dev/staging/prod)
- Upgrade or rollback to a previous version
- Share configurations with others

### Helm's answer → Charts + Values + Releases

```
Chart = Reusable template (the blueprint)
Values = Your configuration (what you inject)
Release = A running instance of a Chart in your cluster
```

### The 3 core abstractions

| Concept | Analogy | What it is |
|---------|---------|------------|
| **Chart** | Docker Image | A versioned, reusable package of Kubernetes templates |
| **Values** | Environment Variables | The input configuration that customizes a Chart |
| **Release** | Running Container | A deployed instance of a Chart with a specific set of Values |

### Helm's Design Philosophy
1. **Declarative over imperative** — describe *what* you want, not *how* to get there
2. **Versioned and auditable** — every install/upgrade creates a numbered revision
3. **Parameterized templates** — one chart, many environments via `values.yaml`
4. **Atomic upgrades** — either the whole upgrade succeeds, or Helm rolls back
5. **Release-scoped ownership** — Helm tracks which objects belong to which release

---

## 📦 Installation & Setup

```bash
# Install Helm (Linux)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version

# Add official stable chart repo
helm repo add stable https://charts.helm.sh/stable

# Add Bitnami repo (most popular)
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update all repos
helm repo update

# List all repos
helm repo list
```

---

## 🔍 Search & Inspect Charts

```bash
# Search Helm Hub (Artifact Hub)
helm search hub wordpress

# Search within added repos
helm search repo bitnami/wordpress

# Show all versions of a chart
helm search repo bitnami/wordpress --versions

# Show chart details (metadata, keywords, etc.)
helm show chart bitnami/wordpress

# Show default values
helm show values bitnami/wordpress

# Show all info (chart + values + readme)
helm show all bitnami/wordpress
```

---

## 🚀 Install / Deploy

```bash
# Basic install
helm install <release-name> <chart>
helm install my-wordpress bitnami/wordpress

# Install into a specific namespace (create ns if not exists)
helm install my-wordpress bitnami/wordpress \
  --namespace wordpress \
  --create-namespace

# Install with custom values file
helm install my-wordpress bitnami/wordpress \
  -f my-values.yaml

# Install with inline value overrides
helm install my-wordpress bitnami/wordpress \
  --set service.type=NodePort \
  --set wordpressUsername=admin

# Install a local chart folder
helm install productservice-release ./productservice

# Dry run (preview what would be deployed — NO cluster changes)
helm install my-wordpress bitnami/wordpress --dry-run --debug

# Render templates locally without connecting to cluster
helm template my-wordpress bitnami/wordpress
helm template my-wordpress ./productservice -f values.yaml
```

---

## ♻️ Upgrade

```bash
# Upgrade an existing release
helm upgrade <release-name> <chart>
helm upgrade my-wordpress bitnami/wordpress

# Upgrade with custom values
helm upgrade my-wordpress bitnami/wordpress -f my-values.yaml

# Upgrade or install if release doesn't exist yet
helm upgrade --install my-wordpress bitnami/wordpress

# Upgrade and wait for all pods to be ready before marking success
helm upgrade my-wordpress bitnami/wordpress --wait

# Upgrade with a timeout (default is 5m0s)
helm upgrade my-wordpress bitnami/wordpress --wait --timeout 10m

# Force resource replacement (use with caution)
helm upgrade my-wordpress bitnami/wordpress --force
```

---

## ⏪ Rollback

```bash
# View release history (shows revision numbers)
helm history my-wordpress

# Rollback to the previous revision
helm rollback my-wordpress

# Rollback to a specific revision number
helm rollback my-wordpress 2

# Dry run rollback (preview what would change)
helm rollback my-wordpress 2 --dry-run
```

---

## 🗑️ Uninstall

```bash
# Remove a release (deletes all Kubernetes objects created by Helm)
helm uninstall my-wordpress

# Uninstall from a specific namespace
helm uninstall my-wordpress -n wordpress

# Keep release history after uninstall (allows future rollback)
helm uninstall my-wordpress --keep-history

# Clean up orphaned PVCs after uninstall (Helm does NOT delete PVCs automatically)
kubectl delete pvc -l app.kubernetes.io/instance=my-wordpress -n default
```

---

## 📋 List & Inspect Releases

```bash
# List all releases in current namespace
helm list

# List releases in all namespaces
helm list -A

# List only deployed releases
helm list --deployed

# List only failed releases
helm list --failed

# Show detailed status of a release
helm status my-wordpress

# Show computed values for a running release
helm get values my-wordpress

# Show all values (including defaults)
helm get values my-wordpress --all

# Show manifest (what Kubernetes YAML was actually applied)
helm get manifest my-wordpress

# Show release notes
helm get notes my-wordpress
```

---

## 🛠️ Create & Develop Charts

```bash
# Scaffold a new chart
helm create my-chart

# Chart structure
# my-chart/
# ├── Chart.yaml        # Chart metadata (name, version, appVersion)
# ├── values.yaml       # Default values
# ├── .helmignore       # Files to exclude from packaging
# └── templates/        # Go-templated Kubernetes manifests
#     ├── deployment.yaml
#     ├── service.yaml
#     ├── ingress.yaml
#     ├── _helpers.tpl  # Reusable template snippets
#     └── NOTES.txt     # Post-install instructions shown to user

# Lint a chart (validate syntax)
helm lint ./my-chart

# Lint with custom values
helm lint ./my-chart -f values-prod.yaml

# Package a chart into a .tgz archive
helm package ./my-chart

# Inspect a packaged chart
helm show chart my-chart-0.1.0.tgz
```

---

## 🔬 Debugging

```bash
# Dry run with full debug output
helm install my-release ./my-chart --dry-run --debug

# Render templates locally (no cluster needed)
helm template my-release ./my-chart

# Render with specific values file
helm template my-release ./my-chart -f values-prod.yaml

# Check what Helm actually sent to the cluster
helm get manifest my-release

# Describe pod crashing in a Helm-managed deployment
kubectl describe pod <pod-name>

# Check pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # logs from crashed/previous container

# Watch all resources for a release live
watch kubectl get all -l app.kubernetes.io/instance=my-release
```

---

## 🌐 Environment Patterns (Multi-env with Helm)

```bash
# Recommended project structure
# helm/
# ├── my-chart/           # The chart (shared template)
# │   ├── Chart.yaml
# │   ├── values.yaml     # Defaults
# │   └── templates/
# ├── values-dev.yaml     # Dev overrides
# ├── values-staging.yaml # Staging overrides
# └── values-prod.yaml    # Prod overrides

# Deploy to dev
helm upgrade --install my-app ./my-chart -f values-dev.yaml -n dev --create-namespace

# Deploy to staging
helm upgrade --install my-app ./my-chart -f values-staging.yaml -n staging --create-namespace

# Deploy to prod
helm upgrade --install my-app ./my-chart -f values-prod.yaml -n prod --create-namespace
```

---

## ⚠️ Common Pitfalls

| Pitfall | Explanation | Fix |
|---------|-------------|-----|
| **Probe port mismatch** | Default chart probes check port `80` (nginx), your app may run on a different port | Update `livenessProbe.httpGet.port` in `values.yaml` |
| **CrashLoopBackOff after install** | Kubernetes probes fail → container killed | Run `kubectl describe pod` + `kubectl logs` to diagnose |
| **PVCs not deleted on uninstall** | Helm protects data by design | Manually delete PVCs with `kubectl delete pvc` |
| **`--set` overrides lost on upgrade** | Helm doesn't remember `--set` flags between runs | Always use `-f values.yaml` for persistent config |
| **Wrong resource type in kubectl** | `kubectl describe <pod-name>` fails | Must include resource type: `kubectl describe pod <pod-name>` |
| **`minikube service` with pod name** | `minikube service` needs a Service name, not a Pod name | Run `kubectl get service` to find the Service name |

---

## 📐 Key Mental Model

```
kubectl (raw YAML)          Helm (packaged + versioned)
─────────────────────       ────────────────────────────
kubectl apply -f *.yaml  →  helm install / helm upgrade
kubectl delete -f *.yaml →  helm uninstall
(manual history)         →  helm history / helm rollback
(hardcoded values)       →  values.yaml + --set overrides
(no versioning)          →  Chart.yaml version + revision tracking
```

---

*Last updated: 2026-05-16 | CloudNative learning notes*
