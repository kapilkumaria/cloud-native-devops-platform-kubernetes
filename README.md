# Cloud-Native DevOps Platform - Kubernetes Repository

This repository contains the Kubernetes platform layer for the Cloud-Native DevOps Platform project.

It demonstrates Amazon EKS, Kubernetes workloads, GitOps with ArgoCD, Traefik ingress routing, Prometheus monitoring, Grafana dashboards, Alertmanager, Loki, and Promtail.

## Project Role

This is repository 3 of the flagship DevOps platform project.

| Repository | Purpose |
|---|---|
| cloud-native-devops-platform-app | Application source code, Dockerfiles, Docker Compose |
| cloud-native-devops-platform-infra | Terraform, Ansible, AWS infrastructure automation |
| cloud-native-devops-platform-kubernetes | Kubernetes, GitOps, ingress, monitoring, logging |

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

## Architecture Overview

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
  - screenshots/
  - diagrams/
  - validation documents

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

## Kubernetes Deployment

Apply application manifests manually:

    kubectl apply -k kubernetes

Validate:

    kubectl get pods -n cloud-platform
    kubectl get svc -n cloud-platform
    kubectl get all -n cloud-platform

## ArgoCD GitOps Deployment

Apply ArgoCD project and application:

    kubectl apply -f argocd/projects/cloud-platform-project.yaml
    kubectl apply -f argocd/applications/cloud-platform-app.yaml

Validate:

    kubectl get appprojects -n argocd
    kubectl get applications -n argocd

Expected result:

    cloud-platform-app   Synced   Healthy

## Traefik Routing

Validate Traefik:

    kubectl get svc -n traefik
    kubectl get ingressroute -n cloud-platform
    kubectl get middleware -n cloud-platform

Test routes:

    TRAEFIK_LB=$(kubectl get svc traefik -n traefik -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

    curl -I -H "Host: app.example.com" http://$TRAEFIK_LB/
    curl -I -H "Host: app.example.com" http://$TRAEFIK_LB/docs

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

## Logging Stack

Validate logging resources:

    kubectl get all -n logging

Loki readiness:

    curl http://localhost:3100/ready

Loki internal components:

    curl http://localhost:3100/services

## Validation Evidence

Screenshots are stored in:

    docs/screenshots/

Important screenshots include:

| Screenshot | Purpose |
|---|---|
| eks-nodes.png | EKS worker nodes ready |
| cloud-platform-namespace.png | Application namespace validation |
| cloud-platform-resources.png | Application resources running |
| kubernetes-pods-running.png | Frontend, backend, and PostgreSQL pods running |
| kubernetes-services.png | Kubernetes Services validation |
| traefik-ingressroute.png | Traefik IngressRoute validation |
| traefik-middleware.png | Traefik Middleware validation |
| argocd-application-synced.png | ArgoCD application synced and healthy |
| argocd-sync-events.png | ArgoCD sync event history |
| monitoring-resources.png | Monitoring stack resources |
| prometheus-targets.png | Prometheus target health |
| prometheus-up-query.png | PromQL query validation |
| grafana-dashboard.png | Grafana dashboard validation |
| alertmanager-ui.png | Alertmanager UI validation |
| logging-resources.png | Loki and Promtail resources |
| loki-components-running.png | Loki internal services running |
| loki-promtail-running.png | Loki and Promtail runtime validation |

## Documentation

Detailed validation notes are available in:

- docs/kubernetes-validation.md
- docs/gitops-validation.md
- docs/argocd-validation.md
- docs/traefik-validation.md
- docs/monitoring-validation.md
- docs/logging-validation.md

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

Pending:

- Final professional architecture diagram

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
