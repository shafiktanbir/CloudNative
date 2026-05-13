# ☸️ Kubernetes & Minikube Cheat Sheet

---

## 🚀 Minikube

```bash
# Start / Stop / Status
minikube start
minikube stop
minikube status
minikube delete          # destroy the cluster completely

# Access a LoadBalancer service (opens browser + shows URL)
minikube service <service-name>

# Get the URL only (no browser)
minikube service <service-name> --url

# Open the Kubernetes Dashboard
minikube dashboard

# SSH into the minikube node
minikube ssh

# Check minikube IP
minikube ip

# Load a local Docker image into minikube (avoids push to registry)
minikube image load <image-name>:<tag>

# Port-forward a service to localhost (alternative to minikube service)
kubectl port-forward service/<service-name> <local-port>:<service-port>
# Example:
kubectl port-forward service/my-service 8080:80
```

---

## 📦 Pods

```bash
# List pods
kubectl get pods
kubectl get pods -o wide          # with node/IP info
kubectl get pods -w               # watch (live updates)

# Describe a pod (events, status, config)
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> -f        # follow (live)
kubectl logs <pod-name> --previous  # logs from crashed pod

# Shell into a running pod
kubectl exec -it <pod-name> -- /bin/sh
kubectl exec -it <pod-name> -- /bin/bash

# Delete a pod (Deployment will recreate it)
kubectl delete pod <pod-name>
```

---

## 🌐 Services

```bash
# List services
kubectl get services
kubectl get svc                   # shorthand

# Describe a service
kubectl describe service <service-name>

# Delete a service
kubectl delete service <service-name>

# Service types:
#   ClusterIP    → internal only (default)
#   NodePort     → exposes on node IP + random port (30000-32767)
#   LoadBalancer → external IP (use with minikube service <name>)
```

---

## 🔄 Deployments

```bash
# List deployments
kubectl get deployments
kubectl get deploy               # shorthand

# Describe a deployment
kubectl describe deployment <name>

# Apply / update from YAML
kubectl apply -f <file>.yaml

# Delete from YAML
kubectl delete -f <file>.yaml

# Scale replicas
kubectl scale deployment <name> --replicas=3

# Restart pods (pulls fresh image if imagePullPolicy: Always)
kubectl rollout restart deployment/<name>

# Check rollout progress
kubectl rollout status deployment/<name>

# Rollback to previous version
kubectl rollout undo deployment/<name>

# View rollout history
kubectl rollout history deployment/<name>
```

---

## 🗺️ ConfigMaps

```bash
# List configmaps
kubectl get configmaps
kubectl get cm                   # shorthand

# Describe a configmap
kubectl describe configmap <name>

# Create from literal
kubectl create configmap <name> --from-literal=KEY=VALUE

# Delete
kubectl delete configmap <name>
```

---

## 🔍 General / Useful

```bash
# Get all resources in namespace
kubectl get all

# Apply all YAMLs in a folder
kubectl apply -f ./k8-practice/

# Delete all from a folder
kubectl delete -f ./k8-practice/

# Get events (great for debugging)
kubectl get events --sort-by='.lastTimestamp'

# Check resource usage (needs metrics-server)
kubectl top pods
kubectl top nodes
```

---

## 🔁 My Update Workflow (ProductService)

```bash
# 1. Build new image
docker build -t ghcr.io/shafiktanbir/productservice:latest .

# 2. Push to GHCR
docker push ghcr.io/shafiktanbir/productservice:latest

# 3. Trigger rolling update
kubectl rollout restart deployment/product-deployment

# 4. Verify
kubectl rollout status deployment/product-deployment
```
