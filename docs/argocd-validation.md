# Argo CD Validation

## Objective

Document historical Argo CD GitOps validation for the Cloud-Native DevOps Platform, and provide checks for a future deployment of the updated Kubernetes manifests.

> **Evidence status — November 10, 2026:** The screenshots and successful synchronization described below reflect the earlier EKS deployment. The current frontend Service, PostgreSQL storage, routing, and Secret configuration have **not** yet been validated through a new Argo CD sync in a running Project 1 EKS cluster.

## Prerequisites

- A running, accessible EKS cluster and a working `kubectl` context.
- Argo CD installed in the `argocd` namespace, including its CRDs.
- The Argo CD AppProject and Application definitions under `argocd/`.
- Traefik and required CRDs installed before syncing resources that use `IngressRoute`.
- EBS CSI driver and Pod Identity configured before PostgreSQL attempts to bind its EBS-backed PVC.
- `postgres-secret` and `backend-secret` provisioned separately in namespace `cloud-platform` as described in [Secure Secret Provisioning](secure-secret-provisioning.md).
- A frontend image built from the updated application source and referenced by the Deployment if testing the corrected same-origin API configuration.

## Argo CD Installation Checks

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl get crd applications.argoproj.io appprojects.argoproj.io
```

Expected: relevant Argo CD components are healthy. A Service existing does not alone mean the UI is publicly accessible; access depends on the Service type and any port-forward, ingress, or access controls.

## Application Definitions

From the root of this repository, inspect the definitions before applying them:

```bash
cat argocd/projects/cloud-platform-project.yaml
cat argocd/applications/cloud-platform-app.yaml
```

Once Argo CD is installed, apply the AppProject first, then the Application:

```bash
kubectl apply -f argocd/projects/cloud-platform-project.yaml
kubectl apply -f argocd/applications/cloud-platform-app.yaml
```

Check the source repository, revision, path, destination and sync policy against the actual YAML. Do not assume automatic sync, pruning or self-healing is enabled without checking `spec.syncPolicy`.

## Synchronization Validation

```bash
kubectl get appprojects -n argocd
kubectl get applications -n argocd
kubectl describe application cloud-platform-app -n argocd
kubectl get application cloud-platform-app -n argocd \
  -o jsonpath='{.status.sync.status}{"\n"}{.status.health.status}{"\n"}'
```

Target status after prerequisites are met:

```text
Synced
Healthy
```

These are **target** results for the updated manifests, not a claim of a successful November 2026 sync.

If resources are stuck, inspect application conditions and sync events, then check dependent workloads, PVCs and Secrets:

```bash
kubectl get application cloud-platform-app -n argocd -o yaml
kubectl get pods,pvc,svc -n cloud-platform
kubectl get events -n cloud-platform --sort-by=.metadata.creationTimestamp
```

## GitOps Flow

```text
Developer change
      |
      v
Git commit / push
      |
      v
GitHub repository
      |
      v
Argo CD comparison against desired state
      |
      v
Sync (automatic only if configured; otherwise manual)
      |
      v
Kubernetes resources and health checks
```

Argo CD can detect drift. Automated self-healing requires the appropriate sync policy; a Git revert can be used to restore a previous declarative configuration, but safe rollback may require special handling for database schema and persistent data.

## Historical Validation Evidence

The screenshots captured during the prior deployment are available at:

- [Argo CD resources](screenshots/argocd-resources.png)
- [Argo CD services](screenshots/argocd-services.png)
- [Running Argo CD pods](screenshots/argocd-pods-running.png)
- [Application synced](screenshots/argocd-application-synced.png)
- [Application details](screenshots/argocd-application-details.png)
- [Sync events](screenshots/argocd-sync-events.png)

These are historic validation records and do not establish that the newly revised manifests have been reconciled successfully.

## Lessons Learned

- GitOps and declarative Kubernetes delivery.
- Separating platform prerequisites from application reconciliation.
- Argo CD Application/AppProject configuration and observability.
- Understanding the difference between `Synced` and `Healthy`.
- Diagnosing reconciliation issues caused by missing CRDs, Secrets, storage drivers or images.
