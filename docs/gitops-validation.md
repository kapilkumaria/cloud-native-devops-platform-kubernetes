# GitOps Validation

## Objective

Validate that Kubernetes resources are managed declaratively using Git as the single source of truth.

---

## Repository Structure

```
kubernetes/
├── apps/
├── database/
├── namespaces/
├── ingress/
└── kustomization.yaml
```

---

## Kustomize Validation

```bash
kubectl apply -k kubernetes
```

Expected Result

- Namespace created
- Secrets created
- ConfigMaps created
- Deployments created
- Services created
- Traefik resources created

---

## Kubernetes Resources

```bash
kubectl get all -n cloud-platform
```

Screenshot

- docs/screenshots/cloud-platform-resources.png

---

## GitOps Workflow

```
Developer

↓

Git Commit

↓

GitHub Repository

↓

ArgoCD

↓

Amazon EKS

↓

Running Application
```

---

## Benefits

- Declarative infrastructure
- Version controlled manifests
- Repeatable deployments
- Easy rollback
- Git-based change management

---

## Lessons Learned

- Kustomize overlays
- Declarative Kubernetes
- GitOps principles
- Infrastructure version control
