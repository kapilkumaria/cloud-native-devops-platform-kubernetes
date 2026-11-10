# Cloud-Native DevOps Platform — Kubernetes Repository

Kubernetes platform configuration for the **Cloud-Native DevOps Platform**, including Amazon EKS workloads, Kustomize, Traefik ingress, Argo CD GitOps, Prometheus/Grafana/Alertmanager monitoring, and Loki/Promtail logging.

> **Implementation and evidence status (November 10, 2026):** This repository was updated to use a frontend `ClusterIP` Service, a PostgreSQL PersistentVolumeClaim (PVC) backed by an EBS `gp3` StorageClass, corrected Traefik API routing, and safer backend configuration. The updated manifests passed a local `kubectl kustomize kubernetes/` render. **There is currently no Project 1 EKS cluster running**, and the revised EBS persistence, ingress behavior, and full-stack deployment have **not yet been validated at runtime**. Screenshots and older validation notes document an earlier cluster configuration.

---

## Architecture Diagram

![Kubernetes Platform Architecture](docs/diagrams/Cloud-Native-DevOps-Platform%20-%20Kubernetes-Platform-Architecture.png)

Editable source: `docs/diagrams/kubernetes-platform-architecture.drawio`.

> The diagram may depict an earlier deployment. For the current desired-state networking and storage configuration, use the diagram and explanations below and the manifests under `kubernetes/`.

## Project Role and Related Repositories

This is **repository 3** of the Cloud-Native DevOps Platform project.

| Repository | Responsibility |
| --- | --- |
| [cloud-native-devops-platform-infra](https://github.com/kapilkumaria/cloud-native-devops-platform-infra) | Terraform-based AWS infrastructure, EKS and EBS CSI configuration; separate EC2/Ansible deployment path |
| [cloud-native-devops-platform-app](https://github.com/kapilkumaria/cloud-native-devops-platform-app) | React/Vite frontend, FastAPI backend, PostgreSQL, Dockerfiles and Docker Compose |
| **cloud-native-devops-platform-kubernetes** | Kubernetes manifests, Traefik routing, Argo CD applications, monitoring and logging |

## What This Repository Demonstrates

- EKS application workloads with Kubernetes Deployments, Services, Namespace, ConfigMaps and separately provisioned Secrets.
- Kustomize-based resource composition.
- Traefik as the intended external HTTP entry point; the frontend and backend use internal `ClusterIP` Services.
- PostgreSQL PVC backed by an EBS `gp3` StorageClass, with the AWS EBS CSI add-on configured separately in the infrastructure repository.
- Argo CD GitOps configuration and historical reconciliation validation.
- Prometheus, Grafana and Alertmanager observability configuration.
- Loki and Promtail log collection configuration.
- Historical validation notes and screenshots, explicitly distinguished from the latest source changes.

---

## Current Intended EKS Architecture

```text
Browser / HTTP client
       |
       | DNS + HTTP Host header (example: app.example.com)
       v
AWS Load Balancer exposing Traefik
       |
       v
Traefik IngressRoute
       | \
       |  \-- Host + PathPrefix(`/api`) --> backend ClusterIP :8000
       |                                     |
       |                                     v
       |                                 FastAPI Pod
       |                                     |
       |                                     v
       |                              postgres ClusterIP
       |                                     |
       |                                     v
       |                               PostgreSQL Pod
       |                                     |
       |                               postgres-pvc
       |                                     |
       |                          EBS CSI driver + gp3 EBS volume
       |
       \----- Host + PathPrefix(`/`) -----> frontend ClusterIP :80
                                             |
                                             v
                                        Nginx / React Pod
```

The infrastructure repository defines the EKS add-on configuration for the EBS CSI driver and its IAM access through **EKS Pod Identity**. This add-on must be installed and operational before a dynamically provisioned EBS-backed PVC can bind successfully.

The **example hostname `app.example.com` is a placeholder**. Configure an actual hostname and appropriate DNS/IngressRoute settings before claiming public browser access. The current description does not claim production TLS/HTTPS is implemented or validated for that hostname.

### Why the frontend is ClusterIP

The previous frontend Service was a `LoadBalancer`, creating a second potential public entry point even though Traefik already provided ingress. The current desired state exposes the frontend internally through `ClusterIP`; Traefik is the intended external ingress boundary. This architecture avoids an unnecessary frontend-specific AWS load balancer.

### Traefik API routing

- Requests matching the configured host and `/api` prefix are forwarded to the backend Service on port `8000`.
- Frontend requests matching the host and `/` prefix are forwarded to the frontend Service on port `80`.
- **Do not strip `/api`** from backend requests. The FastAPI application uses `/api/v1` routes, and stripping the prefix breaks those paths.
- An older middleware manifest or screenshot may still be present as historical evidence; the current Kustomize render does not attach the old StripPrefix middleware to the API route.

### Frontend API base URL

The application repository now contains:

```typescript
OpenAPI.BASE = import.meta.env.VITE_API_URL || window.location.origin
```

`VITE_API_URL` is a **Vite build-time variable**, not a runtime Nginx/Kubernetes environment override. If no explicit URL is set when compiling the frontend, browser API requests use the frontend's current origin, allowing Traefik to forward `/api/v1/...` to FastAPI.

The old frontend runtime ConfigMap has been removed. **An existing frontend Docker image does not update when application source changes**: build and publish an updated frontend image and update its Kubernetes image reference before testing the new behavior in EKS. The previously referenced `kappu1512/cloud-platform-frontend:v1` should not be assumed to contain the fix.

### PostgreSQL persistence

The current PostgreSQL Deployment mounts a PVC named `postgres-pvc` rather than using `emptyDir` for database data. The associated manifest defines a `gp3` StorageClass provisioned by `ebs.csi.aws.com`, with `WaitForFirstConsumer`, encryption and volume expansion enabled.

The PostgreSQL Deployment uses a `Recreate` strategy, suitable for this single-replica example to avoid two Pods concurrently trying to mount the same `ReadWriteOnce` volume during a rollout. **This is not database high availability, and the PVC is not a backup.** EBS volumes are Availability Zone-bound; proper backup/restore and disaster-recovery planning are still required.

The StorageClass uses `reclaimPolicy: Retain`. Deleting a PVC may therefore leave its underlying EBS volume behind **and continue to incur AWS storage costs**. Review retained PVs/EBS volumes during teardown before deleting anything that contains data you need.

> **Migration warning:** Switching an existing PostgreSQL Deployment from `emptyDir` to a PVC does **not** transfer existing database contents. Back up and restore any important data before making this change on a running environment. No currently running Project 1 EKS PostgreSQL workload was identified at the November documentation checkpoint.

---

## Repository Structure

```text
cloud-native-devops-platform-kubernetes/
├── argocd/
│   ├── applications/
│   └── projects/
├── kubernetes/
│   ├── apps/
│   │   ├── backend/
│   │   └── frontend/
│   ├── database/postgres/
│   │   ├── deployment.yaml
│   │   ├── pvc.yaml
│   │   └── storageclass.yaml
│   ├── ingress/traefik/ingressroutes/
│   ├── namespaces/
│   └── kustomization.yaml
├── monitoring/
├── logging/
├── docs/
│   ├── diagrams/
│   ├── screenshots/
│   ├── kubernetes-validation.md
│   ├── traefik-validation.md
│   ├── argocd-validation.md
│   ├── gitops-validation.md
│   ├── monitoring-validation.md
│   ├── logging-validation.md
│   └── secure-secret-provisioning.md
└── README.md
```

Use the actual Git tree as the authority for the complete manifests and chart configuration. The directory listing above is a high-level map, not an exhaustive file inventory.

---

## Deployment Prerequisites and Safe Sequence

1. Provision the VPC, EKS cluster and worker nodes from the infrastructure repository; inspect a fresh Terraform plan before `apply`.
2. Provision and verify the AWS EBS CSI driver add-on, its EKS Pod Identity agent/association and IAM permissions, using the configuration under `terraform/eks/` in the infrastructure repository.
3. Configure `kubectl` against the **intended** cluster and verify its context before applying resources.
4. Install and validate the Traefik controller and its required CRDs using a compatible Helm chart. The Traefik install is a separate prerequisite; the application Kustomization does not install the controller itself.
5. Create the `cloud-platform` Namespace, then securely provision the required Kubernetes Secrets **without committing secret values to Git**. Follow [secure-secret-provisioning.md](docs/secure-secret-provisioning.md).
6. Build, scan and publish the application images, including a **new frontend image** incorporating the same-origin API fallback. Update the Deployment image references to the intended immutable tag or digest before deployment.
7. Render and inspect the Kubernetes desired state; verify there are no unexpected external `LoadBalancer` Services, stale image references or credentials in ConfigMaps.
8. Apply the Kubernetes manifests in a controlled deployment and validate Pods, Services, PVC binding and application behavior.
9. Install/configure Argo CD and verify the repository, `targetRevision`, path and prospective prune set **before** enabling automated synchronization.
10. Deploy or reconnect monitoring and logging using their repository-specific configuration; verify actual queries/log ingestion, not just Running Pods.
11. Record runtime results and safely tear down cloud resources when testing is complete.

**GitOps safety:** An Argo CD Application may be configured to track `main` with automated sync/prune/self-heal. Review the source revision and live cluster before applying the Application. A push to Git can trigger reconciliation when a cluster is running and connected.

**Status:** These steps are the **recommended sequence for the next controlled rebuild**, not evidence that the latest configuration has been deployed successfully.

---

## Local Manifest Validation (No EKS Required)

From the repository root:

```bash
kubectl kustomize kubernetes/ > /tmp/cloud-platform-rendered.yaml
```

Review the rendered manifest:

```bash
grep -nE 'kind: (StorageClass|PersistentVolumeClaim|Service|IngressRoute)' \
  /tmp/cloud-platform-rendered.yaml
```

Verify the source references:

```bash
grep -Rni 'postgres-pvc\|gp3-cloud-platform\|ebs.csi.aws.com' \
  kubernetes/database/postgres kubernetes/kustomization.yaml

grep -Rni 'kind: Service\|type: ClusterIP' \
  kubernetes/apps/frontend/service.yaml

grep -n 'PathPrefix' \
  kubernetes/ingress/traefik/ingressroutes/cloud-platform-ingressroute.yaml
```

A successful Kustomize render was observed after the November changes. **Rendering is not equivalent to API-server validation or runtime verification.** The Kubernetes resources still require the relevant CRDs, Secrets, controller installation and AWS infrastructure to function.

---

## EKS Runtime Validation (Required on Next Deployment)

Confirm the context and resource inventory:

```bash
kubectl config current-context
kubectl get nodes -o wide
kubectl get pods -n kube-system
kubectl get pods -n cloud-platform -o wide
kubectl get deployments,services -n cloud-platform
kubectl get storageclass
kubectl get pvc -n cloud-platform
kubectl get pv
```

**Expected only after provisioning:** EKS worker nodes `Ready`; EBS CSI controller/agent healthy; PostgreSQL PVC `Bound`; application Pods `Ready`; frontend Service `ClusterIP`; Traefik has the intended external entry point.

Inspect a PVC if it remains `Pending`:

```bash
kubectl describe pvc postgres-pvc -n cloud-platform
kubectl get events -n cloud-platform --sort-by=.lastTimestamp
```

Check EBS CSI add-on status from AWS (adjust the cluster name to the actual deployment):

```bash
aws eks describe-addon \
  --cluster-name '<EKS_CLUSTER_NAME>' \
  --addon-name aws-ebs-csi-driver \
  --region us-east-1
```

Inspect PostgreSQL rollout and mounts:

```bash
kubectl rollout status deployment/postgres -n cloud-platform
kubectl describe deployment/postgres -n cloud-platform
```

Before claiming data persistence, create a known non-sensitive test record, recreate the PostgreSQL Pod through a controlled procedure, and verify the record remains. Do not use destructive testing against important data; keep a backup and recovery procedure. **This persistence test has not yet been executed for the revised configuration.**

---

## Traefik Routing Validation

Verify controller and application routes:

```bash
kubectl get svc -n traefik
kubectl get ingressroute -n cloud-platform
kubectl get svc -n cloud-platform
```

Resolve the Traefik Service's external hostname (verify the actual Service name first):

```bash
TRAEFIK_LB="$(kubectl get svc traefik -n traefik \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')"
```

The example `Host` matcher in the checked-in IngressRoute is `app.example.com`. With that placeholder host and **only after Traefik and the app are running**, test through the load balancer using the appropriate HTTP Host header:

```bash
curl -i -H 'Host: app.example.com' "http://${TRAEFIK_LB}/"
curl -i -H 'Host: app.example.com' "http://${TRAEFIK_LB}/api/v1/openapi.json"
```

Expected behavior once deployed: frontend response at `/`; FastAPI OpenAPI response at `/api/v1/openapi.json` **without stripping `/api`**. The test does not claim HTTPS or a valid public DNS name. A `curl -I` request uses `HEAD`, which may not be supported by every FastAPI route; `curl -i` performs a normal `GET` request.

The old README's `http://<TRAEFIK_LB>/docs` example is not a reliable verification of the `/api` routing rule: `/docs` matches the frontend catch-all and does not prove backend routing. Use the `/api/v1/...` endpoint for this check.

---

## Argo CD GitOps Deployment

The repository includes Argo CD Project and Application definitions:

```text
argocd/projects/cloud-platform-project.yaml
argocd/applications/cloud-platform-app.yaml
```

After installing Argo CD and verifying the intended Git source and cluster, inspect these files before applying them:

```bash
kubectl apply -f argocd/projects/cloud-platform-project.yaml
kubectl apply -f argocd/applications/cloud-platform-app.yaml

kubectl get appprojects -n argocd
kubectl get applications -n argocd
```

Check the Application's actual synchronization and health statuses rather than assuming `Synced / Healthy`. Historical screenshots show a previously validated Argo CD deployment; the latest manifest revisions have not yet been reconciled in EKS.

---

## Monitoring and Logging

The repository contains monitoring and logging configuration previously exercised on an EKS cluster.

### Monitoring

```bash
kubectl get all -n monitoring
```

Validate Prometheus target health and PromQL (`up`), Grafana dashboards and Alertmanager configuration after the monitoring stack has been installed. Resources appearing in `kubectl get` do not establish that metrics are queryable or alerts are delivered.

### Logging

```bash
kubectl get all -n logging
```

If Loki is configured to listen on port `3100`, use an appropriate local port-forward before calling `localhost`:

```bash
# Confirm actual Service name and namespace before running:
# kubectl -n logging port-forward svc/<LOKI_SERVICE_NAME> 3100:3100
curl -fsS http://localhost:3100/ready
```

Check that recent workload logs can be queried from Loki, rather than relying only on Pod health. The `localhost` URL works only while a suitable port-forward (or equivalent local listener) is running.

---

## Kubernetes Secret Handling

The repository does not intentionally store live Kubernetes Secret values in tracked manifests. Follow [Secure Secret Provisioning](docs/secure-secret-provisioning.md) for the authoritative provisioning procedure.

Documented Kubernetes Secret names and expected key names:

| Secret | Required keys |
| --- | --- |
| `backend-secret` | `FIRST_SUPERUSER`, `FIRST_SUPERUSER_PASSWORD`, `POSTGRES_PASSWORD`, `SECRET_KEY` |
| `postgres-secret` | `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` |

The backend consumes its normal PostgreSQL connection settings through environment variables and Secret values, rather than a credential-containing `DATABASE_URL` in its ConfigMap. Database passwords must match the intended application/database configuration.

- Create the Namespace before provisioning namespaced Secrets.
- Avoid exposing literal secrets in shell history, screenshots or terminal transcripts.
- Do not commit plaintext Secret YAML, Terraform state files or private keys.
- A Kubernetes Secret is not automatically equivalent to end-to-end secret encryption and governance. For production, consider AWS Secrets Manager with External Secrets Operator, appropriate IAM identity controls and Kubernetes secret-encryption configuration.
- Historical demo credentials appearing in prior Git revisions should be considered exposed and rotated if ever used outside isolated demonstrations.

---

## Validation Evidence — Historical Runtime Screenshots

The screenshots below are **historical validation evidence from a prior EKS deployment**, not evidence that the November 2026 networking/PVC revisions have been deployed and tested. In particular, older screenshots can display the previous frontend LoadBalancer Service or Traefik middleware.

### Amazon EKS

| Screenshot | Historical evidence |
| --- | --- |
| [eks-nodes.png](docs/screenshots/eks-nodes.png) | EKS worker nodes in Ready state |
| [full-cluster.png](docs/screenshots/full-cluster.png) | Cluster resources |
| [kube-system-resources.png](docs/screenshots/kube-system-resources.png) | Kubernetes system components |

### Kubernetes Application

| Screenshot | Historical evidence |
| --- | --- |
| [cloud-platform-namespace.png](docs/screenshots/cloud-platform-namespace.png) | Application namespace |
| [cloud-platform-resources.png](docs/screenshots/cloud-platform-resources.png) | Application resource inventory |
| [kubernetes-pods-running.png](docs/screenshots/kubernetes-pods-running.png) | Frontend, backend and PostgreSQL Pods |
| [kubernetes-services.png](docs/screenshots/kubernetes-services.png) | Services (may include the former frontend LoadBalancer) |
| [postgres-pod-details.png](docs/screenshots/postgres-pod-details.png) | Historical PostgreSQL Pod details; **not** PVC persistence proof |
| [postgres-pod-events.png](docs/screenshots/postgres-pod-events.png) | Historical PostgreSQL events |

### Traefik

| Screenshot | Historical evidence |
| --- | --- |
| [traefik-loadbalancer.png](docs/screenshots/traefik-loadbalancer.png) | Traefik load balancer |
| [traefik-resources.png](docs/screenshots/traefik-resources.png) | Traefik resources |
| [traefik-pods.png](docs/screenshots/traefik-pods.png) | Traefik Pods |
| [traefik-ingressroute.png](docs/screenshots/traefik-ingressroute.png) | Earlier IngressRoute configuration |
| [traefik-middleware.png](docs/screenshots/traefik-middleware.png) | **Old** middleware evidence; not current API routing behavior |
| [traefik-frontend-route.png](docs/screenshots/traefik-frontend-route.png) | Earlier frontend route test |
| [traefik-backend-docs-route.png](docs/screenshots/traefik-backend-docs-route.png) | Earlier backend docs route test |

### Argo CD GitOps

| Screenshot | Historical evidence |
| --- | --- |
| [argocd-resources.png](docs/screenshots/argocd-resources.png) | Argo CD resources |
| [argocd-services.png](docs/screenshots/argocd-services.png) | Argo CD Services |
| [argocd-pods-running.png](docs/screenshots/argocd-pods-running.png) | Argo CD Pods |
| [argocd-application-synced.png](docs/screenshots/argocd-application-synced.png) | Previously synchronized application |
| [argocd-application-details.png](docs/screenshots/argocd-application-details.png) | Application details |
| [argocd-sync-events.png](docs/screenshots/argocd-sync-events.png) | Historical synchronization events |

### Monitoring

| Screenshot | Historical evidence |
| --- | --- |
| [monitoring-resources.png](docs/screenshots/monitoring-resources.png) | Monitoring resource inventory |
| [monitoring-services.png](docs/screenshots/monitoring-services.png) | Monitoring Services |
| [monitoring-pods-running.png](docs/screenshots/monitoring-pods-running.png) | Monitoring Pods |
| [prometheus-targets.png](docs/screenshots/prometheus-targets.png) | Prometheus scrape targets |
| [prometheus-up-query.png](docs/screenshots/prometheus-up-query.png) | PromQL query example |
| [prometheus-runtime.png](docs/screenshots/prometheus-runtime.png) | Prometheus runtime |
| [grafana-dashboard.png](docs/screenshots/grafana-dashboard.png) | Grafana dashboard |
| [alertmanager-ui.png](docs/screenshots/alertmanager-ui.png) | Alertmanager UI |

### Logging

| Screenshot | Historical evidence |
| --- | --- |
| [logging-resources.png](docs/screenshots/logging-resources.png) | Logging resources |
| [loki-promtail-running.png](docs/screenshots/loki-promtail-running.png) | Loki and Promtail |
| [loki-ready.png](docs/screenshots/loki-ready.png) | Loki readiness |
| [loki-components-running.png](docs/screenshots/loki-components-running.png) | Loki internal components |

---

## Documentation

- [Kubernetes Validation](docs/kubernetes-validation.md)
- [GitOps Validation](docs/gitops-validation.md)
- [Argo CD Validation](docs/argocd-validation.md)
- [Traefik Validation](docs/traefik-validation.md)
- [Monitoring Validation](docs/monitoring-validation.md)
- [Logging Validation](docs/logging-validation.md)
- [Secure Secret Provisioning](docs/secure-secret-provisioning.md)

Existing validation notes may refer to earlier configuration; compare their dates and commands to the current manifests before using them as a rebuild procedure.

## Status and Outstanding Work

| Area | Evidence/status |
| --- | --- |
| EKS, Argo CD, monitoring and logging | Historical runtime screenshots and validation documents exist |
| Frontend Service changed to `ClusterIP` | Implemented and merged to `main`; not yet runtime retested |
| PostgreSQL PVC and gp3 StorageClass | Implemented and Kustomize-rendered; AWS volume binding/data persistence not yet tested |
| EBS CSI support | Terraform configuration added in infrastructure repository; AWS add-on installation not yet tested |
| Traefik API routing without StripPrefix | Implemented and Kustomize-rendered; browser/API runtime test pending |
| Backend ConfigMap secret hygiene | Credential-bearing `DATABASE_URL` removed; actual Secret provisioning still required |
| Frontend same-origin API fallback | Application source compiled successfully; new container image must be built/published |
| EKS cluster | Not currently deployed |
| HTTPS/TLS for placeholder host | Not claimed as implemented or verified |
| Database backup/restore and high availability | Not implemented by a single PVC/Deployment |
| Full runtime validation | Pending controlled rebuild |

**Next rebuild acceptance criteria:** EBS CSI add-on healthy; `postgres-pvc` bound; PostgreSQL record survives a controlled Pod recreation; frontend and backend Pods ready; updated frontend image in use; frontend `/` and API `/api/v1/...` served successfully through Traefik; Argo CD reconciliation verified; monitoring/logging queryable; retained EBS volumes audited during teardown.

## Skills Demonstrated

- Amazon EKS workload and networking architecture
- Kubernetes Services, storage and workload troubleshooting
- Terraform-managed EKS add-on and IAM design
- Amazon EBS CSI and PVC/StorageClass configuration
- Helm and Traefik ingress architecture
- Kustomize-based Kubernetes configuration
- Argo CD GitOps delivery concepts and historical validation
- Prometheus, Grafana and Alertmanager monitoring
- Loki/Promtail log aggregation
- Secret-handling principles and evidence-based technical documentation

---

*The manifests in this repository express the current intended configuration. Historical runtime evidence should not be mistaken for validation of changes introduced after the cluster was torn down.*
