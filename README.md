# Cloud-Native DevOps Platform - Kubernetes Repository

This repository contains the Kubernetes, GitOps, ingress, monitoring, and logging configuration for the Cloud-Native DevOps Platform.

## Purpose

This repository represents the platform operations layer.

It is responsible for:

- Kubernetes application deployment
- GitOps delivery with ArgoCD
- Traefik ingress routing
- Monitoring structure
- Logging structure
- Platform validation documentation

## Related Repositories

### Application Repository

`cloud-native-devops-platform-app`

Contains:

- React frontend
- FastAPI backend
- PostgreSQL local Compose setup
- Dockerfiles
- Docker Compose
- Application documentation

### Infrastructure Repository

`cloud-native-devops-platform-infra`

Contains:

- Terraform
- AWS infrastructure
- Ansible automation
- Docker host provisioning
- Automated deployment validation

## Repository Structure

```
cloud-native-devops-platform-kubernetes/
│
├── kubernetes/
│   ├── apps/
│   ├── database/
│   ├── ingress/
│   └── namespaces/
│
├── argocd/
│   ├── applications/
│   └── projects/
│
├── traefik/
│   ├── ingressroutes/
│   ├── middlewares/
│   └── tls/
│
├── monitoring/
│   ├── prometheus/
│   ├── grafana/
│   ├── alertmanager/
│   └── dashboards/
│
├── logging/
│   ├── loki/
│   └── promtail/
│
└── docs/
    ├── diagrams/
    ├── screenshots/
    ├── argocd-validation.md
    └── traefik-validation.md
```

## GitOps Workflow

```
Developer
   ↓
GitHub Repository
   ↓
ArgoCD
   ↓
Kubernetes Cluster
   ↓
Traefik
   ↓
Frontend / Backend / PostgreSQL
```


