# Cloud-Native DevOps Platform - Kubernetes Repository

This repository contains the Kubernetes platform layer for the Cloud-Native DevOps Platform project.

It demonstrates Amazon EKS, Kubernetes workloads, GitOps with ArgoCD, Traefik ingress routing, Prometheus monitoring, Grafana dashboards, Alertmanager, Loki, and Promtail.

---

## Architecture Diagram

![Kubernetes Platform Architecture](docs/diagrams/Cloud-Native-DevOps-Platform%20-%20Kubernetes-Platform-Architecture.png)

---

## Project Role

This is repository 3 of the flagship DevOps platform project.

| Repository | Purpose |
|---|---|
| cloud-native-devops-platform-app | Application source code, Dockerfiles, Docker Compose |
| cloud-native-devops-platform-infra | Terraform, Ansible, AWS infrastructure automation |
| cloud-native-devops-platform-kubernetes | Kubernetes, GitOps, ingress, monitoring, logging |

---

## What This Repository Demonstrates

- Amazon EKS workload deployment
- Kubernetes Deployments, Services, ConfigMaps, Secrets, and Namespaces
- Kustomize-based manifest deployment
- ArgoCD GitOps application delivery
- Traefik IngressRoute and Middleware routing
- Prometheus monitoring
- Grafana dashboards
- Alertmanager alerting layer
- Loki log aggregation
- Promtail log collection
- Kubernetes validation and operational evidence

---

## Platform Architecture

GitHub Repository
        |
        v
      ArgoCD
        |
        v
   Amazon EKS Cluster
        |
        v
  Traefik Ingress Controller
        |
        v
Frontend ----> Backend ----> PostgreSQL

Monitoring:
Prometheus + Grafana + Alertmanager

Logging:
Loki + Promtail

---

## Repository Structure

- argocd/
  - applications/
  - projects/
- kubernetes/
  - apps/
  - database/
  - ingress/
  - namespaces/
  - kustomization.yaml
- monitoring/
  - prometheus/
  - grafana/
  - alertmanager/
  - dashboards/
- logging/
  - loki/
  - promtail/
- traefik/
  - ingressroutes/
  - middlewares/
  - tls/
- docs/
  - diagrams/
  - screenshots/
  - validation documents

---

## Deployment Workflow

1. Terraform provisions the EKS cluster from the infrastructure repository.
2. kubectl is configured using aws eks update-kubeconfig.
3. Traefik is installed with Helm.
4. Application manifests are deployed using Kustomize.
5. ArgoCD is installed.
6. ArgoCD watches this repository.
7. ArgoCD syncs Kubernetes resources into the EKS cluster.
8. Prometheus, Grafana, and Alertmanager are installed for monitoring.
9. Loki and Promtail are installed for logging.

---

## Kubernetes Deployment

Apply application manifests manually:

    kubectl apply -k kubernetes

Validate:

    kubectl get pods -n cloud-platform
    kubectl get svc -n cloud-platform
    kubectl get all -n cloud-platform

---

## ArgoCD GitOps Deployment

Apply ArgoCD project and application:

    kubectl apply -f argocd/projects/cloud-platform-project.yaml
    kubectl apply -f argocd/applications/cloud-platform-app.yaml

Validate:

    kubectl get appprojects -n argocd
    kubectl get applications -n argocd

Expected result:

    cloud-platform-app   Synced   Healthy

---

## Traefik Routing

Validate Traefik:

    kubectl get svc -n traefik
    kubectl get ingressroute -n cloud-platform
    kubectl get middleware -n cloud-platform

Test routes:

    TRAEFIK_LB=$(kubectl get svc traefik -n traefik -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

    curl -I -H "Host: app.example.com" http://$TRAEFIK_LB/
    curl -I -H "Host: app.example.com" http://$TRAEFIK_LB/docs

---

## Monitoring Stack

Validate monitoring resources:

    kubectl get all -n monitoring

Prometheus validation:

- Target health page
- up PromQL query
- Kubernetes scrape targets

Grafana validation:

- Kubernetes dashboard access
- Cluster metrics visibility

Alertmanager validation:

- Alertmanager UI access
- Alerting service running

---

## Logging Stack

Validate logging resources:

    kubectl get all -n logging

Loki readiness:

    curl http://localhost:3100/ready

Loki internal components:

    curl http://localhost:3100/services

---

## Validation Evidence

Runtime validation screenshots are available under:

    docs/screenshots/

### Amazon EKS

| Screenshot | Description |
|---|---|
| [eks-nodes.png](docs/screenshots/eks-nodes.png) | Amazon EKS worker nodes in Ready state |
| [full-cluster.png](docs/screenshots/full-cluster.png) | Complete Kubernetes cluster runtime |
| [kube-system-resources.png](docs/screenshots/kube-system-resources.png) | Kubernetes system components |

### Kubernetes Application

| Screenshot | Description |
|---|---|
| [cloud-platform-namespace.png](docs/screenshots/cloud-platform-namespace.png) | Application namespace created |
| [cloud-platform-resources.png](docs/screenshots/cloud-platform-resources.png) | All application resources running |
| [kubernetes-pods-running.png](docs/screenshots/kubernetes-pods-running.png) | Frontend, backend and PostgreSQL pods |
| [kubernetes-services.png](docs/screenshots/kubernetes-services.png) | Kubernetes Services |
| [postgres-pod-details.png](docs/screenshots/postgres-pod-details.png) | PostgreSQL pod description |
| [postgres-pod-events.png](docs/screenshots/postgres-pod-events.png) | PostgreSQL events |

### Traefik

| Screenshot | Description |
|---|---|
| [traefik-loadbalancer.png](docs/screenshots/traefik-loadbalancer.png) | Traefik AWS LoadBalancer |
| [traefik-resources.png](docs/screenshots/traefik-resources.png) | Traefik Kubernetes resources |
| [traefik-pods.png](docs/screenshots/traefik-pods.png) | Traefik pods |
| [traefik-ingressroute.png](docs/screenshots/traefik-ingressroute.png) | Traefik IngressRoute |
| [traefik-middleware.png](docs/screenshots/traefik-middleware.png) | Traefik Middleware |
| [traefik-frontend-route.png](docs/screenshots/traefik-frontend-route.png) | Frontend routing validation |
| [traefik-backend-docs-route.png](docs/screenshots/traefik-backend-docs-route.png) | Backend Swagger routing validation |

### ArgoCD GitOps

| Screenshot | Description |
|---|---|
| [argocd-resources.png](docs/screenshots/argocd-resources.png) | ArgoCD resources |
| [argocd-services.png](docs/screenshots/argocd-services.png) | ArgoCD services |
| [argocd-pods-running.png](docs/screenshots/argocd-pods-running.png) | ArgoCD pods |
| [argocd-application-synced.png](docs/screenshots/argocd-application-synced.png) | GitOps application synced |
| [argocd-application-details.png](docs/screenshots/argocd-application-details.png) | Application details |
| [argocd-sync-events.png](docs/screenshots/argocd-sync-events.png) | Synchronization history |

### Monitoring

| Screenshot | Description |
|---|---|
| [monitoring-resources.png](docs/screenshots/monitoring-resources.png) | Monitoring namespace resources |
| [monitoring-services.png](docs/screenshots/monitoring-services.png) | Monitoring services |
| [monitoring-pods-running.png](docs/screenshots/monitoring-pods-running.png) | Monitoring pods |
| [prometheus-targets.png](docs/screenshots/prometheus-targets.png) | Prometheus target health |
| [prometheus-up-query.png](docs/screenshots/prometheus-up-query.png) | PromQL query validation |
| [prometheus-runtime.png](docs/screenshots/prometheus-runtime.png) | Prometheus runtime validation |
| [grafana-dashboard.png](docs/screenshots/grafana-dashboard.png) | Grafana dashboard validation |
| [alertmanager-ui.png](docs/screenshots/alertmanager-ui.png) | Alertmanager UI validation |

### Logging

| Screenshot | Description |
|---|---|
| [logging-resources.png](docs/screenshots/logging-resources.png) | Loki and Promtail resources |
| [loki-promtail-running.png](docs/screenshots/loki-promtail-running.png) | Loki and Promtail running |
| [loki-ready.png](docs/screenshots/loki-ready.png) | Loki readiness endpoint |
| [loki-components-running.png](docs/screenshots/loki-components-running.png) | Loki internal services running |

---

## Documentation

Detailed validation notes are available in:

- [Kubernetes Validation](docs/kubernetes-validation.md)
- [GitOps Validation](docs/gitops-validation.md)
- [ArgoCD Validation](docs/argocd-validation.md)
- [Traefik Validation](docs/traefik-validation.md)
- [Monitoring Validation](docs/monitoring-validation.md)
- [Logging Validation](docs/logging-validation.md)

---

## Current Status

Completed:

- EKS cluster deployment
- Kubernetes application deployment
- Traefik ingress routing
- ArgoCD GitOps sync
- Prometheus monitoring
- Grafana dashboard access
- Alertmanager validation
- Loki logging backend
- Promtail log collector
- Runtime screenshots
- Validation documentation
- Architecture diagram

---

## Skills Demonstrated

- Kubernetes platform engineering
- Amazon EKS operations
- GitOps delivery with ArgoCD
- Helm-based platform installation
- Ingress routing with Traefik
- Observability with Prometheus and Grafana
- Alerting with Alertmanager
- Logging with Loki and Promtail
- Kubernetes troubleshooting
- Platform validation and documentation

