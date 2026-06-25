# Kubernetes Deployment Validation

## Objective

Validate that the Cloud-Native DevOps Platform application was successfully deployed to an Amazon EKS Kubernetes cluster.

---

## Namespace Validation

```bash
kubectl get ns
```

Expected Result

- Namespace `cloud-platform` exists.
- Namespace status is `Active`.

Screenshot

- docs/screenshots/cloud-platform-namespace.png

---

## EKS Worker Nodes

```bash
kubectl get nodes -o wide
```

Expected Result

- All worker nodes are Ready.
- Internal IP addresses assigned.
- Kubernetes version matches cluster version.

Screenshot

- docs/screenshots/eks-nodes.png

---

## Application Pods

```bash
kubectl get pods -n cloud-platform -o wide
```

Expected Result

| Pod | Status |
|------|--------|
| frontend | Running |
| backend | Running |
| postgres | Running |

Screenshot

- docs/screenshots/kubernetes-pods-running.png

---

## Kubernetes Services

```bash
kubectl get svc -n cloud-platform
```

Expected Result

| Service | Type |
|----------|------|
| frontend | LoadBalancer |
| backend | ClusterIP |
| postgres | ClusterIP |

Screenshot

- docs/screenshots/kubernetes-services.png

---

## Complete Kubernetes Resources

```bash
kubectl get all -n cloud-platform
```

Expected Result

Resources created successfully:

- Pods
- Services
- Deployments
- ReplicaSets

Screenshot

- docs/screenshots/cloud-platform-resources.png

---

## PostgreSQL Pod Validation

```bash
kubectl describe pod -n cloud-platform <postgres-pod>
```

Validation includes:

- Pod Running
- Readiness probe
- Mounted volume
- Container image
- Scheduler events

Screenshots

- docs/screenshots/postgres-pod-details.png
- docs/screenshots/postgres-pod-events.png

---

## Validation Summary

✔ Namespace created

✔ EKS cluster operational

✔ Worker nodes Ready

✔ Application deployed

✔ Pods Running

✔ Services created

✔ PostgreSQL operational

✔ Deployments healthy

✔ ReplicaSets created

---

## Lessons Learned

- Deploying applications using Kubernetes manifests
- Namespace isolation
- Deployments and ReplicaSets
- ClusterIP vs LoadBalancer services
- Pod scheduling
- Readiness probes
- Kubernetes troubleshooting using kubectl
