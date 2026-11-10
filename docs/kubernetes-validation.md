# Kubernetes Deployment Validation

## Objective and Evidence Status

Document the historical Amazon EKS deployment validation and provide repeatable checks for the **current Kubernetes desired state** of the Cloud-Native DevOps Platform.

> **Updated November 10, 2026.** Earlier screenshots and successful runtime observations represent a previous EKS deployment. The November 2026 changes to frontend Service exposure, PostgreSQL storage, backend configuration, and Traefik routing have been committed and the Kustomize manifests rendered successfully, **but have not yet been validated in a newly deployed EKS cluster**. This document does not claim a live cluster currently exists.

The current architecture is:

```text
Client -> AWS Load Balancer -> Traefik IngressRoute
                                     |--> frontend ClusterIP -> frontend Pod (Nginx/React)
                                     `--> backend ClusterIP  -> FastAPI Pod
                                                                 |
                                                         postgres ClusterIP
                                                                 |
                                                           PostgreSQL Pod
                                                                 |
                                                           postgres-pvc
                                                                 |
                                                        gp3 EBS volume (CSI)
```

The EKS cluster and EBS CSI add-on are defined in the related [infrastructure repository](https://github.com/kapilkumaria/cloud-native-devops-platform-infra). The frontend/backend images and Dockerfiles are defined in the [application repository](https://github.com/kapilkumaria/cloud-native-devops-platform-app).

---

## 1. Before Runtime Validation

Make sure the correct AWS account, region, cluster and context are selected. **Do not treat the example commands as evidence of deployment.**

```bash
aws sts get-caller-identity
aws configure get region
aws eks list-clusters --region us-east-1
kubectl config current-context
```

If the intended cluster exists, configure access using its **actual** name:

```bash
aws eks update-kubeconfig --region us-east-1 --name <project-1-eks-cluster-name>
kubectl config current-context
kubectl cluster-info
```

Before applying manifests, confirm:

- EKS worker nodes are available and Ready.
- The AWS EBS CSI driver is installed and healthy; its IAM permissions and EKS Pod Identity association are configured.
- The `cloud-platform` namespace exists **before** namespace-scoped Secrets are created.
- Required PostgreSQL and backend Secrets have been securely provisioned using [`secure-secret-provisioning.md`](secure-secret-provisioning.md). No real Secret values are committed to Git.
- Traefik is installed with its required CRDs before applying IngressRoute resources.
- The frontend container image actually contains the updated `frontend/src/main.tsx` code. The historical `kappu1512/cloud-platform-frontend:v1` image may not contain the same-origin API fix; build/publish a new immutable image and update the manifest before runtime testing.
- The placeholder host `app.example.com` is replaced/configured as appropriate for the chosen test environment. Its existence in manifests does not imply working public DNS or TLS.

> **Storage migration warning:** Switching an existing PostgreSQL Deployment from `emptyDir` to a PVC does **not** migrate historical data. Plan backup, restore and cutover separately if preserving data matters.

---

## 2. Source-Level Validation (Current Manifests)

From the Kubernetes repository root:

```bash
kubectl kustomize kubernetes/ > /tmp/project1-k8s-rendered.yaml
```

This render was successfully checked after the November source changes. For a future checkout, repeat it and inspect the output:

```bash
rg -n 'kind: (Service|PersistentVolumeClaim|StorageClass|Deployment|IngressRoute)|name: (frontend|postgres-pvc|gp3-cloud-platform)' /tmp/project1-k8s-rendered.yaml
```

A successful render proves only that Kustomize can produce YAML. It does **not** prove that EBS provisioning, Pod scheduling, container startup or ingress routing will work in AWS.

Check Kubernetes schema validation **only when connected to the intended cluster**:

```bash
kubectl apply --dry-run=server -k kubernetes/
```

A server-side dry-run also does not create workloads or prove runtime health.

---

## 3. Namespace Validation

```bash
kubectl get namespace cloud-platform
```

**Expected on a deployed cluster:** the `cloud-platform` namespace exists and is `Active`.

Historical screenshot: [cloud-platform-namespace.png](screenshots/cloud-platform-namespace.png).

---

## 4. EKS Worker Nodes

```bash
kubectl get nodes -o wide
kubectl get nodes -L topology.kubernetes.io/zone
```

**Expected:** all intended workers are `Ready`; internal IPs and Kubernetes versions correspond to the target EKS deployment.

Historical screenshot: [eks-nodes.png](screenshots/eks-nodes.png).

---

## 5. Application Pods and Deployments

```bash
kubectl get deployments,pods -n cloud-platform -o wide
kubectl rollout status deployment/frontend -n cloud-platform --timeout=180s
kubectl rollout status deployment/backend -n cloud-platform --timeout=180s
kubectl rollout status deployment/postgres -n cloud-platform --timeout=180s
```

Expected application components:

| Workload | Expected healthy condition |
| --- | --- |
| `frontend` | Ready replicas, running Nginx/React container |
| `backend` | Ready replicas, FastAPI startup succeeds |
| `postgres` | Ready replica, database initialized, storage mounted |

Check actual Pod names with `kubectl get pods`; Deployment-managed Pod names include generated suffixes.

Historical screenshot: [kubernetes-pods-running.png](screenshots/kubernetes-pods-running.png).

---

## 6. Kubernetes Service Types — Current Design

```bash
kubectl get svc -n cloud-platform
kubectl get svc frontend backend postgres -n cloud-platform \
  -o custom-columns='NAME:.metadata.name,TYPE:.spec.type,PORTS:.spec.ports[*].port'
```

**Expected with the current manifests:**

| Service | Type | Reason |
| --- | --- | --- |
| `frontend` | `ClusterIP` | Internal backend for Traefik; no second public load balancer |
| `backend` | `ClusterIP` | Internal FastAPI routing target |
| `postgres` | `ClusterIP` | Internal database access only |

Traefik's own Service (in the `traefik` namespace) is the intended `LoadBalancer` entry point.

Historical screenshot: [kubernetes-services.png](screenshots/kubernetes-services.png). **This image may reflect the previous frontend `LoadBalancer` configuration; it must not be cited as validation of the current `ClusterIP` change.**

---

## 7. PostgreSQL Persistence and EBS CSI

The current manifest set defines:

- A PostgreSQL Deployment mounting `postgres-pvc` rather than an `emptyDir` database volume.
- A `ReadWriteOnce`, 5 GiB PVC requesting the `gp3-cloud-platform` StorageClass.
- A `gp3` StorageClass using the AWS EBS CSI provisioner (`ebs.csi.aws.com`), with encryption, `WaitForFirstConsumer`, expansion enabled, and **`Retain`** reclaim policy.
- A `Recreate` strategy for the single-replica PostgreSQL Deployment.

**Expected after actual deployment:**

```bash
kubectl get storageclass gp3-cloud-platform
kubectl get pvc postgres-pvc -n cloud-platform -o wide
kubectl describe pvc postgres-pvc -n cloud-platform
kubectl get pv
kubectl get pods -n kube-system | grep -i ebs
kubectl describe deployment postgres -n cloud-platform
kubectl get pods -n cloud-platform -o wide
```

A PVC with `WaitForFirstConsumer` may remain `Pending` until a Pod using it is scheduled. Investigate events before assuming a provisioner failure.

For runtime persistence validation, use a **non-production** test database, write a unique test record, restart/recreate the PostgreSQL Pod, and verify the record survives. Do not delete the PVC to test persistence, and do not perform this destructive experiment against valuable data.

> **Cost and cleanup warning:** `Retain` can leave an EBS volume after its claim is deleted. Verify retained PVs and EBS volumes during teardown; do not assume `kubectl delete` or `terraform destroy` removes every storage charge. Take a backup before deleting retained volumes.

Historical screenshots: [postgres-pod-details.png](screenshots/postgres-pod-details.png) and [postgres-pod-events.png](screenshots/postgres-pod-events.png). They reflect earlier runtime validation and do **not** establish current EBS-backed persistence.

---

## 8. Backend Secrets and Configuration

The current backend ConfigMap no longer carries a password-bearing `DATABASE_URL`. The backend constructs its connection settings from separate environment variables, with sensitive values supplied through Kubernetes Secrets.

Reference: [`secure-secret-provisioning.md`](secure-secret-provisioning.md).

Check only Secret existence and **key names**, never values:

```bash
kubectl get secret -n cloud-platform
kubectl get configmap -n cloud-platform
kubectl describe deployment backend -n cloud-platform
```

Do **not** print or commit Secret YAML, decoded values, database URLs containing passwords, or shell transcripts exposing credentials. Rotate any real credential that was previously included in published configuration or Git history.

---

## 9. Full Resource Inventory

```bash
kubectl get all -n cloud-platform
kubectl get ingressroute -n cloud-platform
kubectl get pvc -n cloud-platform
kubectl get events -n cloud-platform --sort-by=.lastTimestamp
```

`kubectl get all` does not include every resource type (for example, PVCs, Secrets, IngressRoutes, or StorageClasses). Use the dedicated commands above.

Historical screenshot: [cloud-platform-resources.png](screenshots/cloud-platform-resources.png).

---

## 10. Troubleshooting

```bash
kubectl describe pod <pod-name> -n cloud-platform
kubectl logs -n cloud-platform deployment/backend --tail=100
kubectl get events -n cloud-platform --sort-by=.lastTimestamp
kubectl describe pvc postgres-pvc -n cloud-platform
kubectl get endpoints -n cloud-platform
```

| Symptom | First checks |
| --- | --- |
| PostgreSQL PVC Pending | EBS CSI add-on, Pod Identity/IAM, StorageClass, PVC events, node AZ/scheduling |
| PostgreSQL CrashLoopBackOff | DB Secret keys, startup logs, PVC mount, data-directory permissions |
| Backend errors | Backend logs, database Service/endpoints, required Secret keys, DB readiness |
| Frontend API errors | Published frontend image, Vite build-time `VITE_API_URL`, Traefik `/api/v1` route |
| Service without endpoints | Pod readiness, Service selectors, container ports |
| Public HTTP fails | Traefik LB, host header/DNS, IngressRoute, controller logs |

---

## 11. Validation Summary

**Historical EKS deployment:** Namespace, nodes, application Pods, Services and PostgreSQL runtime were previously validated; screenshots document that earlier environment.

**November 2026 source changes:** Kustomize rendering passed locally. The revised frontend `ClusterIP`, PostgreSQL EBS persistence, EBS CSI integration, corrected routing, updated image and Secrets **still require a new end-to-end EKS runtime validation**.

Do not mark the following complete until tested on a fresh/rebuilt cluster:

```text
[ ] EBS CSI add-on operational with correct identity permissions
[ ] gp3 StorageClass installed and postgres-pvc Bound
[ ] PostgreSQL persistence proven across Pod replacement
[ ] frontend, backend and postgres Ready
[ ] frontend/backend/postgres Services are ClusterIP
[ ] New frontend image deployed
[ ] Traefik API and frontend paths return expected responses
[ ] Argo CD reconciliation remains healthy with the corrected desired state
[ ] Monitoring and logging still receive telemetry
[ ] Retained EBS storage audited during teardown
```

## Lessons Learned

- Distinguish historical runtime evidence from newly committed desired state.
- Use internal `ClusterIP` Services behind an ingress controller.
- Use CSI-managed persistent volumes for stateful databases, with explicit recovery and deletion policies.
- Understand `WaitForFirstConsumer`, EKS Pod Identity and AWS IAM permissions for dynamic EBS provisioning.
- Validate Kubernetes layers independently: render, API validation, Pods, Services, storage, and ingress.
