# Kubernetes Validation

## Objective

Validate that the application stack can be deployed to Kubernetes using declarative manifests.

## Components

- Frontend Deployment and Service
- Backend Deployment and Service
- PostgreSQL Deployment, Service, Secret, and PVC
- Namespace
- ArgoCD Application

## Validation Commands

```bash
kubectl apply -f apps/base/namespace.yaml
kubectl apply -f apps/postgres/
kubectl apply -f apps/backend/
kubectl apply -f apps/frontend/

## Verification

```bash
kubectl get pods -n cloud-platform
kubectl get svc -n cloud-platform
kubectl get pvc -n cloud-platform
```
## Expected Result

All application pods should reach Running status and services should be created successfully.
