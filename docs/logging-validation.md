# Logging Validation

## Objective

Record the earlier centralized logging deployment on Amazon EKS and provide repeatable checks for a future deployment of the Cloud-Native DevOps Platform.

> **Evidence status — November 10, 2026:** The historical screenshots and successful Loki/Promtail observations below belong to the earlier EKS deployment. The revised Project 1 configuration has not been redeployed or revalidated at runtime.

## Previously Deployed Components

- Loki logging backend (historically a StatefulSet).
- Promtail agent (historically a DaemonSet).
- Loki Service (historically internal `ClusterIP`).

The exact component layout can vary with Helm chart versions; inspect the deployed resources before assuming names or ports.

## Namespace and Resource Checks

```bash
kubectl get ns logging
kubectl get pods -n logging
kubectl get svc -n logging
kubectl get statefulset,daemonset -n logging
```

## Loki Readiness

Verify the Service name and ports:

```bash
kubectl get svc -n logging
```

For the historical `loki` Service:

```bash
kubectl port-forward -n logging svc/loki 3100:3100
```

In a separate terminal:

```bash
curl -fsS http://localhost:3100/ready
curl -fsS http://localhost:3100/services
```

The `/ready` endpoint should indicate readiness when Loki is operational. The `/services` endpoint is diagnostic; component names depend on Loki deployment mode and version, so do not require every possible component to appear in all installations.

## Promtail Agent Checks

```bash
kubectl get daemonsets -n logging
kubectl get pods -n logging -o wide
kubectl logs -n logging daemonset/promtail --tail=100
```

The final command assumes the historical DaemonSet is named `promtail`; use `kubectl get daemonsets -n logging` first to confirm.

Promtail was deployed as a DaemonSet to collect Kubernetes node/container logs. Verify desired versus ready pod counts and any node selector, affinity or toleration limitations rather than assuming one agent runs on every node.

## End-to-End Log Validation

A running Promtail DaemonSet and a ready Loki endpoint do not by themselves prove that application logs are being ingested. On a rebuilt cluster, use Grafana Explore or an authenticated Loki query API to confirm recent logs from a known application pod and record the query/observed timestamps.

## Historical Validation Screenshots

| Screenshot | Previously recorded evidence |
| --- | --- |
| [logging-resources.png](screenshots/logging-resources.png) | Logging namespace resources |
| [loki-promtail-running.png](screenshots/loki-promtail-running.png) | Loki and Promtail resources running |
| [loki-ready.png](screenshots/loki-ready.png) | Loki readiness result |
| [loki-components-running.png](screenshots/loki-components-running.png) | Loki internal services |

## Historical Result and Current Status

The earlier deployment recorded Loki readiness and running Promtail resources. This documentation preserves those results while leaving verification of current log ingestion and revised EKS application behavior explicitly **pending**.
