# Traefik Validation

## Objective

Validate application routing through Traefik.

## Routes

| Path | Target |
|---|---|
| `/` | frontend service |
| `/api` | backend service |
| `/docs` | backend Swagger UI |
| `/redoc` | backend Redoc UI |

## Validate

```
kubectl get ingressroute -n cloud-platform
kubectl get middleware -n cloud-platform
```

## Browser Validation

```
http://app.example.com/
http://app.example.com/docs
```

