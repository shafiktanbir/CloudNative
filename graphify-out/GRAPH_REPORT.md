# Graph Report - .  (2026-05-14)

## Corpus Check
- Large corpus: 248 files · ~63,232 words. Semantic extraction will be expensive (many Claude tokens). Consider running on a subfolder, or use --no-semantic to run AST-only.

## Summary
- 85 nodes · 78 edges · 24 communities (10 shown, 14 thin omitted)
- Extraction: 54% EXTRACTED · 46% INFERRED · 0% AMBIGUOUS · INFERRED: 36 edges (avg confidence: 0.89)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Distributed Tracing & Container Images|Distributed Tracing & Container Images]]
- [[_COMMUNITY_CICD & DevOps Tooling|CI/CD & DevOps Tooling]]
- [[_COMMUNITY_ProductService Deployment Variants|ProductService Deployment Variants]]
- [[_COMMUNITY_Helm Chart Templates|Helm Chart Templates]]
- [[_COMMUNITY_Event-Driven Autoscaling (KEDA + Kafka)|Event-Driven Autoscaling (KEDA + Kafka)]]
- [[_COMMUNITY_Istio Service Mesh & Bookinfo|Istio Service Mesh & Bookinfo]]
- [[_COMMUNITY_ProductService Application Core|ProductService Application Core]]
- [[_COMMUNITY_.NET Build Artifacts|.NET Build Artifacts]]
- [[_COMMUNITY_Node.js Backing Services|Node.js Backing Services]]
- [[_COMMUNITY_Product Data Model|Product Data Model]]
- [[_COMMUNITY_Course Overview|Course Overview]]
- [[_COMMUNITY_Agents & AI Config|Agents & AI Config]]
- [[_COMMUNITY_Istio Gateway Config|Istio Gateway Config]]
- [[_COMMUNITY_CockroachDB StatefulSet|CockroachDB StatefulSet]]
- [[_COMMUNITY_Kubernetes ClusterIP Services|Kubernetes ClusterIP Services]]
- [[_COMMUNITY_Nginx Service|Nginx Service]]
- [[_COMMUNITY_Kubernetes Service (Practice)|Kubernetes Service (Practice)]]
- [[_COMMUNITY_Kubernetes Service (Alt)|Kubernetes Service (Alt)]]
- [[_COMMUNITY_Cloud-Native Course Slides|Cloud-Native Course Slides]]
- [[_COMMUNITY_Persistent Volume Claims|Persistent Volume Claims]]
- [[_COMMUNITY_Nginx Ingress Controller|Nginx Ingress Controller]]
- [[_COMMUNITY_Swagger  OpenAPI Docs|Swagger / OpenAPI Docs]]

## God Nodes (most connected - your core abstractions)
1. `Commands: communications/lecture187` - 10 edges
2. `Observability Stack` - 10 edges
3. `Helm Chart Templating` - 10 edges
4. `Kubernetes Replica Scaling` - 5 edges
5. `Apache Kafka Message Broker` - 5 edges
6. `Kubernetes Orchestration` - 5 edges
7. `ASP.NET Core Web API` - 4 edges
8. `GitHub Container Registry (GHCR)` - 3 edges
9. `Istio Bookinfo Demo App` - 3 edges
10. `Deployment: skywalking-oap` - 3 edges

## Surprising Connections (you probably didn't know these)
- `ASP.NET Core Web API` --shares_data_with--> `Apache Kafka Message Broker`  [INFERRED]
  microservices/lecture69/ProductService/Program.cs → scalability/lecture319/consumer-deployment.yaml
- `HorizontalPodAutoscaler: {{ include "productservice.fullname" . }}` --implements--> `Helm Chart Templating`  [INFERRED]
  orchestrators/lecture154/helm/productservice/templates/hpa.yaml → communications/lecture187/addons/grafana.yaml
- `Ingress: {{ $fullName }}` --implements--> `Helm Chart Templating`  [INFERRED]
  orchestrators/lecture154/helm/productservice/templates/ingress.yaml → communications/lecture187/addons/grafana.yaml
- `Unknown: productservice` --implements--> `Helm Chart Templating`  [INFERRED]
  orchestrators/lecture154/helm/productservice/Chart.yaml → communications/lecture187/addons/grafana.yaml
- `Unknown: ""` --implements--> `Helm Chart Templating`  [INFERRED]
  orchestrators/lecture154/helm/productservice/values.yaml → communications/lecture187/addons/grafana.yaml

## Hyperedges (group relationships)
- **Istio Observability Stack (Prometheus, Grafana, Jaeger, Kiali)** — jaeger_deployment, kiali_serviceaccount, grafana_serviceaccount, prometheus_serviceaccount, prometheus_operator_podmonitor, zipkin_deployment [EXTRACTED 1.00]
- **ProductService Deployment Pipeline** — program_cs, product_cs, productservice_assemblyinfo_cs, productservice_globalusings_g_cs, product_deploy_deployment, product_pod_pod [INFERRED 0.85]
- **Kubernetes Workload Manifests** — bookinfo_service, jaeger_deployment, zipkin_deployment, skywalking_deployment, nginx_deployment_deployment, product_deploy_deployment [INFERRED 0.95]

## Communities (24 total, 14 thin omitted)

### Community 0 - "Distributed Tracing & Container Images"
Cohesion: 0.15
Nodes (13): Container Image: apache/skywalking-oap-server:9.1.0, Container Image: apache/skywalking-ui:9.1.0, Container Image: "docker.io/jaegertracing/all-in-one:1.35", Container Image: openzipkin/zipkin-slim:2.23.14, Deployment: jaeger, ServiceAccount: kiali, Observability Stack, PodMonitor: envoy-stats-monitor (+5 more)

### Community 1 - "CI/CD & DevOps Tooling"
Cohesion: 0.22
Nodes (10): ArgoCD GitOps, Commands: communications/lecture187, Docker CLI, GitHub Container Registry (GHCR), Helm CLI, Ingress: {{ $fullName }}, K8S Cheatsheet, Kubernetes Ingress (+2 more)

### Community 2 - "ProductService Deployment Variants"
Cohesion: 0.24
Nodes (10): Container Image: 179510827425.dkr.ecr.us-east-2.amazonaws.com/productservice:latest, Container Image: ghcr.io/shafiktanbir/productservice:latest, Container Image: mehmetozkaya/productservice:latest, Container Image: nginx:latest, Kubernetes Replica Scaling, Deployment: nginx-deployment, NGINX Reverse Proxy, Deployment: product (+2 more)

### Community 3 - "Helm Chart Templates"
Cohesion: 0.2
Nodes (10): Unknown: productservice, Deployment: {{ include "productservice.fullname" . }}, ServiceAccount: grafana, Helm Chart Templating, Container Image: busybox, Container Image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}", Service: {{ include "productservice.fullname" . }}, ServiceAccount: {{ include "productservice.serviceAccountName" . }} (+2 more)

### Community 4 - "Event-Driven Autoscaling (KEDA + Kafka)"
Cohesion: 0.22
Nodes (9): Deployment: consumer-deployment, Horizontal Pod Autoscaler (HPA), HorizontalPodAutoscaler: {{ include "productservice.fullname" . }}, Container Image: bitnami/kafka:latest, Apache Kafka Message Broker, ScaledObject: kafka-consumer-scaledobject, KEDA Event-Driven Autoscaler, HorizontalPodAutoscaler: nginx-hpa (+1 more)

### Community 5 - "Istio Service Mesh & Bookinfo"
Cohesion: 0.29
Nodes (7): Gateway: bookinfo-gateway, Service: details, How Kubernetes Works, Istio Bookinfo Demo App, Istio Service Mesh, K8S Objects Mental Map, Kubernetes Orchestration

### Community 6 - "ProductService Application Core"
Cohesion: 0.5
Nodes (5): ASP.NET Core Web API, Minimal API Endpoints, MongoDB Database, Program (.cs), Redis Cache

### Community 7 - ".NET Build Artifacts"
Cohesion: 0.67
Nodes (3): .NET Build Artifacts, ProductService.AssemblyInfo (.cs), ProductService.GlobalUsings.g (.cs)

## Knowledge Gaps
- **42 isolated node(s):** `index (.js)`, `Node.js / Express HTTP Server`, `Minimal API Endpoints`, `Product (.cs)`, `Product Data Model` (+37 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **14 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `Helm Chart Templating` connect `Helm Chart Templates` to `CI/CD & DevOps Tooling`, `Event-Driven Autoscaling (KEDA + Kafka)`, `Istio Service Mesh & Bookinfo`?**
  _High betweenness centrality (0.259) - this node is a cross-community bridge._
- **Why does `Observability Stack` connect `Distributed Tracing & Container Images` to `Helm Chart Templates`, `Istio Service Mesh & Bookinfo`?**
  _High betweenness centrality (0.207) - this node is a cross-community bridge._
- **Why does `Commands: communications/lecture187` connect `CI/CD & DevOps Tooling` to `Event-Driven Autoscaling (KEDA + Kafka)`, `Istio Service Mesh & Bookinfo`, `ProductService Application Core`?**
  _High betweenness centrality (0.186) - this node is a cross-community bridge._
- **Are the 10 inferred relationships involving `Observability Stack` (e.g. with `Deployment: jaeger` and `ServiceAccount: kiali`) actually correct?**
  _`Observability Stack` has 10 INFERRED edges - model-reasoned connections that need verification._
- **Are the 10 inferred relationships involving `Helm Chart Templating` (e.g. with `ServiceAccount: grafana` and `Unknown: productservice`) actually correct?**
  _`Helm Chart Templating` has 10 INFERRED edges - model-reasoned connections that need verification._
- **Are the 5 inferred relationships involving `Kubernetes Replica Scaling` (e.g. with `Deployment: nginx-deployment` and `Deployment: product`) actually correct?**
  _`Kubernetes Replica Scaling` has 5 INFERRED edges - model-reasoned connections that need verification._
- **Are the 2 inferred relationships involving `Apache Kafka Message Broker` (e.g. with `RabbitMQ Message Broker` and `ASP.NET Core Web API`) actually correct?**
  _`Apache Kafka Message Broker` has 2 INFERRED edges - model-reasoned connections that need verification._