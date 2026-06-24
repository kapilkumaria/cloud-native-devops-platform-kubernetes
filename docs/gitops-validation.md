# GitOps Validation

## Objective

Validate application deployment using ArgoCD.

## ArgoCD Application

```bash
kubectl apply -f argocd/application-app.yaml
```

## Verification
```
kubectl get applications -n argocd
```
## Expected status:

```
Synced
Healthy
```

## Result

ArgoCD continuously reconciles the Kubernetes cluster state with this Git repository.
