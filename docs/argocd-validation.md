# ArgoCD Validation

## Objective

Validate GitOps deployment using ArgoCD.

## Components

- ArgoCD Project
- ArgoCD Application
- Kubernetes manifests repository
- Automated sync
- Self-healing
- Pruning

## Apply

```
kubectl apply -f argocd/projects/cloud-platform-project.yaml
kubectl apply -f argocd/applications/cloud-platform-app.yaml
```
## Validate

```
kubectl get applications -n argocd
kubectl describe application cloud-platform-app -n argocd
```

## Expected Result

The ArgoCD application should show:

```
Synced
Healthy
```
