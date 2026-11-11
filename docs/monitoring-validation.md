# Monitoring Validation

## Objective

Record the earlier Amazon EKS monitoring deployment and provide verification commands for future deployments of the Cloud-Native DevOps Platform.

> **Evidence status — November 10, 2026:** The successful monitoring results and screenshots below are from the previous EKS runtime. The updated Project 1 EKS application configuration is not currently running and has not been revalidated end to end.

## Monitoring Components

The previously validated monitoring stack included:

- Prometheus
- Grafana
- Alertmanager
- Prometheus Operator
- kube-state-metrics
- node-exporter

The installation/Service names below reflect the earlier deployment; confirm actual Helm release and Service names on any rebuilt cluster.

## Namespace and Resource Checks

```bash
kubectl get ns monitoring
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl get all -n monitoring
```

Expected when deployed: relevant monitoring components are present, scheduled and ready. `kubectl get all` does not include every Kubernetes resource type or Prometheus Operator CRD.

## Prometheus

First check the actual Service names:

```bash
kubectl get svc -n monitoring
```

For the historical `kube-prometheus-stack` Service naming convention:

```bash
kubectl port-forward -n monitoring svc/kube-prometheus-stack-prometheus 9090:9090
```

Open `http://localhost:9090`, check **Status > Targets**, and query:

```promql
up
```

Confirm intended scrape targets are discovered and healthy. An `up` result of `1` indicates a successful scrape of that target at query time; it does not by itself validate all application functionality.

## Grafana

```bash
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3001:80
```

Open `http://localhost:3001`. Validate the configured Prometheus data source, dashboard panels and time series. Use securely managed Grafana credentials; do not publish passwords or session tokens in this repository.

## Alertmanager

```bash
kubectl port-forward -n monitoring svc/kube-prometheus-stack-alertmanager 9093:9093
```

Open `http://localhost:9093`. Check Alertmanager health and active configuration. UI availability alone does **not** establish that alert notifications are delivered: verify alert rules, routing, receiver configuration and notification delivery separately if claimed.

## Historical Validation Screenshots

| Screenshot | Previously recorded evidence |
| --- | --- |
| [monitoring-resources.png](screenshots/monitoring-resources.png) | Monitoring namespace resources |
| [monitoring-services.png](screenshots/monitoring-services.png) | Monitoring Services |
| [monitoring-pods-running.png](screenshots/monitoring-pods-running.png) | Running monitoring pods |
| [prometheus-targets.png](screenshots/prometheus-targets.png) | Prometheus target state |
| [prometheus-up-query.png](screenshots/prometheus-up-query.png) | `up` query |
| [prometheus-runtime.png](screenshots/prometheus-runtime.png) | Prometheus runtime |
| [grafana-dashboard.png](screenshots/grafana-dashboard.png) | Grafana dashboard |
| [alertmanager-ui.png](screenshots/alertmanager-ui.png) | Alertmanager UI |

## Historical Result and Current Status

The earlier EKS deployment demonstrated accessible Prometheus, Grafana dashboards and Alertmanager UI, supported by the linked screenshots. This is historical evidence, **not** a claim that the November revised manifests or alert delivery have been verified in a running cluster.
