# Monitoring Validation

## Objective

This document validates the monitoring platform deployed on Amazon EKS for the Cloud-Native DevOps Platform.

The monitoring stack consists of:

* Prometheus
* Grafana
* Alertmanager
* kube-state-metrics
* node-exporter
* Prometheus Operator

---

# Monitoring Namespace

The monitoring components are deployed into the dedicated namespace.

```bash
kubectl get all -n monitoring
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

The monitoring namespace contains:

* Prometheus
* Grafana
* Alertmanager
* Prometheus Operator
* kube-state-metrics
* node-exporter

---

# Prometheus Validation

Port-forward Prometheus.

```bash
kubectl port-forward svc/kube-prometheus-stack-prometheus \
-n monitoring 9090:9090
```

Open:

```
http://localhost:9090
```

Validation performed:

* Prometheus UI is accessible.
* Kubernetes targets are discovered.
* Metrics are successfully scraped.
* PromQL queries execute successfully.

Example query:

```text
up
```

---

# Grafana Validation

Port-forward Grafana.

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana \
-n monitoring 3001:80
```

Open:

```
http://localhost:3001
```

Validation performed:

* Grafana UI loads successfully.
* Dashboards are available.
* Kubernetes metrics are visualized.
* Prometheus datasource is functioning correctly.

---

# Alertmanager Validation

Port-forward Alertmanager.

```bash
kubectl port-forward svc/kube-prometheus-stack-alertmanager \
-n monitoring 9093:9093
```

Open:

```
http://localhost:9093
```

Validation performed:

* Alertmanager UI is accessible.
* Alertmanager service is healthy.
* Alert routing configuration is loaded.

---

# Validation Screenshots

| Screenshot                  | Description                    |
| --------------------------- | ------------------------------ |
| monitoring-resources.png    | Monitoring namespace resources |
| monitoring-services.png     | Monitoring services            |
| monitoring-pods-running.png | Monitoring pods                |
| prometheus-targets.png      | Prometheus target health       |
| prometheus-up-query.png     | PromQL query validation        |
| prometheus-runtime.png      | Runtime metrics                |
| grafana-dashboard.png       | Grafana dashboard              |
| alertmanager-ui.png         | Alertmanager UI                |

---

# Result

The monitoring platform was successfully deployed on Amazon EKS.

Prometheus is collecting cluster metrics, Grafana is visualizing metrics through dashboards, and Alertmanager is operational for alert management.
