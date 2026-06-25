# Logging Validation

## Objective

This document validates the centralized logging platform deployed on Amazon EKS.

The logging stack consists of:

* Loki
* Promtail

---

# Logging Namespace

Logging components are deployed into the dedicated namespace.

```bash
kubectl get all -n logging
kubectl get pods -n logging
kubectl get svc -n logging
```

The namespace contains:

* Loki StatefulSet
* Promtail DaemonSet
* Loki ClusterIP Service

---

# Loki Validation

Port-forward Loki.

```bash
kubectl port-forward svc/loki \
-n logging 3100:3100
```

Verify readiness.

```bash
curl http://localhost:3100/ready
```

Expected output:

```text
ready
```

Verify Loki internal services.

```bash
curl http://localhost:3100/services
```

Expected running services include:

* distributor
* ingester
* querier
* query-frontend
* compactor
* store
* query-scheduler
* memberlist
* ring

---

# Promtail Validation

Verify Promtail deployment.

```bash
kubectl get daemonset -n logging
```

Verify Promtail pods.

```bash
kubectl get pods -n logging
```

Promtail is deployed as a DaemonSet to ensure every Kubernetes worker node forwards container logs to Loki.

---

# Validation Screenshots

| Screenshot                  | Description                             |
| --------------------------- | --------------------------------------- |
| logging-resources.png       | Logging namespace resources             |
| loki-promtail-running.png   | Loki StatefulSet and Promtail DaemonSet |
| loki-ready.png              | Loki readiness endpoint                 |
| loki-components-running.png | Loki internal services                  |

---

# Result

The centralized logging platform was successfully deployed on Amazon EKS.

Loki is operational, Promtail is collecting logs from Kubernetes worker nodes, and the logging infrastructure is ready for integration with Grafana.
