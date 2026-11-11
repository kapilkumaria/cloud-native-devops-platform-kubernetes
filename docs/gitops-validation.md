# GitOps Validation

## Objective

Document declarative delivery using Git, Kustomize and Argo CD for the Cloud-Native DevOps Platform, while distinguishing earlier successful deployment evidence from validation still required for the updated manifests.

> **Evidence status — November 10, 2026:** Earlier screenshots document a deployed EKS application. The revised frontend `ClusterIP` Service, EBS-backed PostgreSQL PVC, and Traefik `/api/v1` routing have not been validated at runtime in a new Project 1 EKS deployment.

## Repository Structure

```text
kubernetes/
├── apps/
├── database/
├── ingress/
├── namespaces/
└── kustomization.yaml
```

The source of truth for application manifests is `kubernetes/kustomization.yaml` and its referenced resources. The Argo CD Application/AppProject definitions are in `argocd/`.

## Kustomize Source Validation (Does Not Require a Cluster)

Run from the repository root:

```bash
kubectl kustomize kubernetes/ > /tmp/cloud-platform-rendered.yaml
```

Inspect the rendered YAML and verify that it contains the intended current resources and **does not** contain embedded credentials. Rendering validates assembly, not the availability of AWS infrastructure or runtime health.

## Deployment Prerequisites

Before attempting a live deployment:

1. Provision EKS networking, worker nodes, required IAM permissions and the EBS CSI add-on/Pod Identity association from the infrastructure repository.
2. Install Traefik and its CRDs before applying `IngressRoute` objects.
3. Create the `cloud-platform` namespace and provision `postgres-secret` and `backend-secret` out of band; follow [Secure Secret Provisioning](secure-secret-provisioning.md).
4. Ensure the required frontend and backend image references are available; the frontend image must be rebuilt if testing the updated API base URL code.
5. Confirm the context and cluster before applying manifests.

**Kustomize does not provision the application Secrets in this repository.** The Secrets must exist before the dependent workloads can start; never claim that `kubectl apply -k` creates them.

## Optional Manual Apply

```bash
kubectl config current-context
kubectl apply -k kubernetes/
kubectl get pods,svc,pvc -n cloud-platform
kubectl get ingressroute -n cloud-platform
```

Use manual apply for troubleshooting or intentional bootstrap workflows. If Argo CD owns the same resources, manage subsequent changes through Git/Argo CD and avoid competing manual edits.

## Argo CD Reconciliation

```bash
kubectl apply -f argocd/projects/cloud-platform-project.yaml
kubectl apply -f argocd/applications/cloud-platform-app.yaml
kubectl get applications -n argocd
```

Inspect the Application's `spec.source`, `spec.destination` and `spec.syncPolicy`; auto-sync, pruning and self-healing depend on the configured policy and should not be assumed.

Expected after all prerequisites and a successful sync:

- `cloud-platform-app` reports `Synced` and `Healthy`.
- Frontend and backend are exposed internally through Kubernetes `ClusterIP` Services.
- PostgreSQL PVC is `Bound` to an appropriate EBS-backed PV.
- Traefik forwards `/api/v1` paths without removing `/api`.
- The backend receives required credentials from pre-existing Secrets.

These are **future runtime checks**, not November 2026 test results.

## Historical Evidence

- [Earlier cloud-platform resources](screenshots/cloud-platform-resources.png)
- [Earlier Argo CD application sync](screenshots/argocd-application-synced.png)
- [Earlier Argo CD sync events](screenshots/argocd-sync-events.png)

For detailed Argo CD checks, see [Argo CD Validation](argocd-validation.md).

## Benefits and Lessons Learned

- Version-controlled desired state and reviewable changes.
- Reproducible deployment specifications.
- Drift detection and policy-dependent reconciliation.
- Dependency ordering for CRDs, storage drivers, Secrets and workloads.
- Git-based configuration rollback, with extra care for stateful data.
