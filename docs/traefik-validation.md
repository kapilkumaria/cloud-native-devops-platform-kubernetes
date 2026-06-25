# Traefik Validation

## Objective

Validate Traefik Ingress Controller deployment and routing configuration.

---

## Traefik Deployment

```bash
kubectl get pods -n traefik
```

Expected Result

Traefik pod Running.

---

## Traefik Service

```bash
kubectl get svc -n traefik
```

Expected Result

LoadBalancer Service created.

---

## Middleware Validation

```bash
kubectl get middleware -A
```

Screenshot

- docs/screenshots/traefik-middleware.png

---

## IngressRoute Validation

```bash
kubectl get ingressroute -A
```

Screenshot

- docs/screenshots/traefik-ingressroute.png

---

## Load Balancer

Traefik exposes:

- HTTP (80)
- HTTPS (443)

using an AWS Elastic Load Balancer.

---

## Routing

Traefik routes requests to:

Frontend

↓

Backend API

↓

PostgreSQL

---

## Lessons Learned

- Traefik CRDs
- IngressRoute
- Middleware
- Strip Prefix
- LoadBalancer Services
