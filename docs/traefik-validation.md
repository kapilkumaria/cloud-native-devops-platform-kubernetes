# Traefik Validation

## Objective and Evidence Status

Document earlier Traefik runtime validation and define the correct validation procedure for the **current Cloud-Native DevOps Platform Kubernetes manifests**.

> **Updated November 10, 2026.** The earlier screenshots show a previous EKS/Traefik configuration, including middleware that was used historically. The current `IngressRoute` configuration no longer applies the old StripPrefix middleware to `/api` requests. The revised routing configuration has been rendered locally but **has not yet been retested against a running Project 1 EKS cluster**.

---

## 1. Current Request Flow

```text
Client
  |
  | HTTP Host: app.example.com (placeholder)
  v
AWS Load Balancer
  |
  v
Traefik ingress controller
  |
  +-- Host + PathPrefix(`/api`) --> backend ClusterIP:8000 --> FastAPI
  |
  `-- Host + PathPrefix(`/`)    --> frontend ClusterIP:80 --> Nginx/React
```

The frontend is **not** meant to create its own public AWS LoadBalancer. Traefik's Service is the intended ingress entry point.

`app.example.com` is a manifest placeholder, not proof that a DNS record or production HTTPS certificate exists.

---

## 2. Installation Prerequisites

Before runtime tests, verify:

- The intended EKS cluster exists and `kubectl` is connected to the correct context.
- Traefik is installed and its CRDs (including `IngressRoute`) are available.
- The `cloud-platform` namespace, application Deployments and internal Services are healthy.
- A frontend image built from the corrected application source has been published and deployed. Historical image tag `kappu1512/cloud-platform-frontend:v1` may not include the same-origin API fallback.
- The intended hostname and routing strategy are defined for the test environment.

```bash
kubectl config current-context
kubectl get pods -n traefik
kubectl get svc -n traefik
kubectl get crd | grep -i ingressroute
kubectl get deploy,pods,svc -n cloud-platform
```

---

## 3. Traefik Controller and Public Service

```bash
kubectl get pods -n traefik -o wide
kubectl get svc -n traefik -o wide
kubectl get endpoints -n traefik
```

**Expected on a rebuilt cluster:** the Traefik controller is Ready and the intended `LoadBalancer` Service has an external hostname/address.

The original deployment exposed Traefik HTTP/HTTPS entry points on ports 80 and 443. The actual listeners, TLS configuration and external address must be checked on each new deployment; a Service port 443 alone does not prove valid HTTPS certificates.

Historical screenshots: [traefik-loadbalancer.png](screenshots/traefik-loadbalancer.png), [traefik-resources.png](screenshots/traefik-resources.png), [traefik-pods.png](screenshots/traefik-pods.png).

---

## 4. IngressRoute — Current Desired State

The current manifest is:

`kubernetes/ingress/traefik/ingressroutes/cloud-platform-ingressroute.yaml`

Verify its rendered form before applying:

```bash
kubectl kustomize kubernetes/ > /tmp/project1-k8s-rendered.yaml
kubectl get ingressroute -n cloud-platform
kubectl describe ingressroute -n cloud-platform
```

The **intended** routes are:

| Match | Service | Port | Behavior |
| --- | --- | --- | --- |
| `Host(app.example.com)` and `/api` prefix | `backend` | `8000` | Preserve the original `/api/v1/...` request path |
| `Host(app.example.com)` and `/` prefix | `frontend` | `80` | Serve React application through Nginx |

The router should prioritize the more specific `/api` match as intended by the manifest. Confirm behavior with actual HTTP tests before declaring the routing correct.

Historical screenshot: [traefik-ingressroute.png](screenshots/traefik-ingressroute.png).

---

## 5. StripPrefix Middleware — Historical Versus Current

**Historical implementation:** A Traefik StripPrefix middleware was previously used, and the earlier validation screenshot documents that design:

[traefik-middleware.png](screenshots/traefik-middleware.png).

**Current implementation:** Do **not** attach the historical StripPrefix middleware to the backend `/api` route. FastAPI already exposes API paths under `/api/v1`; stripping `/api` would forward a different path and can return `404`.

A middleware object remaining in a historical file or screenshot is not itself the problem: **what matters is whether the active IngressRoute references it**. Verify the current route references rather than assuming middleware is being applied.

```bash
kubectl get ingressroute -n cloud-platform -o yaml
kubectl get middleware -n cloud-platform
```

The presence or absence of other middleware resources is not evidence that StripPrefix is active on this particular backend route.

---

## 6. Runtime HTTP Tests (After EKS Deployment)

First discover the actual Traefik LoadBalancer Service name:

```bash
kubectl get svc -n traefik
```

If the Service is named `traefik`, obtain its AWS hostname:

```bash
TRAEFIK_LB="$(kubectl get svc traefik -n traefik \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')"

printf 'Traefik LB: %s\n' "$TRAEFIK_LB"
```

Use the correct host header while testing the example-host route:

```bash
# Frontend: an HTML response is expected when the frontend and route are healthy.
curl -i -H 'Host: app.example.com' "http://${TRAEFIK_LB}/"

# Backend OpenAPI: the current API prefix must be preserved.
curl -i -H 'Host: app.example.com' "http://${TRAEFIK_LB}/api/v1/openapi.json"
```

**Use GET** (`curl -i`) for these functional checks: `curl -I` sends HEAD and can produce misleading results if an endpoint is GET-only.

Do not use the previous public test path `/docs` as the main ingress validation. The current backend route matches `/api`; a direct `/docs` request is not guaranteed to reach the FastAPI service. For Swagger testing, access the backend via a controlled port-forward when appropriate:

```bash
kubectl -n cloud-platform port-forward svc/backend 18000:8000
# In another terminal: open http://127.0.0.1:18000/docs
```

**Expected, not yet reverified for November 2026:**

- Frontend root responds with the application HTML, typically HTTP 200.
- `/api/v1/openapi.json` returns a successful OpenAPI JSON response if that path is served by the running backend image.
- API requests do not lose the `/api` prefix.
- Browser API calls use the same origin when `VITE_API_URL` was unset **at frontend image build time**.

When a real public hostname is configured, verify DNS and TLS **separately** rather than treating HTTP routing as proof of HTTPS readiness.

Historical screenshots: [traefik-frontend-route.png](screenshots/traefik-frontend-route.png), [traefik-backend-docs-route.png](screenshots/traefik-backend-docs-route.png). These are historical, not November route-validation evidence.

---

## 7. Troubleshooting

```bash
kubectl get ingressroute -n cloud-platform
kubectl describe ingressroute -n cloud-platform
kubectl get svc,endpoints -n cloud-platform
kubectl logs -n traefik deployment/traefik --tail=100
kubectl logs -n cloud-platform deployment/backend --tail=100
kubectl logs -n cloud-platform deployment/frontend --tail=100
```

| Symptom | Check |
| --- | --- |
| Traefik Service has no external address | Controller Service type, AWS load-balancer provisioning and events |
| Frontend responds but API returns 404 | API route match, `/api/v1` path preservation, middleware references |
| API is routed to frontend | IngressRoute match rules, priority/specificity and host header |
| Traefik returns 502/503 | Backend/frontend Service endpoints, readiness, Pod logs |
| Frontend browser calls fail while direct API succeeds | Vite build-time URL, old frontend image, browser network tab, CORS/same-origin behavior |
| `Host` mismatch | IngressRoute's configured host and request Host header |
| HTTPS fails | Actual entry point/TLS configuration, DNS and certificate provisioning; do not assume enabled |

---

## 8. Validation Status

**Historically validated:** Traefik controller, AWS LoadBalancer Service, IngressRoute and middleware/routing were tested in an earlier EKS deployment. Refer to the screenshots for that point-in-time evidence.

**Current source configuration:** The `/api` route was changed to avoid StripPrefix. Source manifests passed Kustomize rendering.

**Still pending on a rebuilt EKS cluster:**

```text
[ ] Traefik controller/CRDs installed and healthy
[ ] Traefik LoadBalancer external address available
[ ] frontend/backend Services are ClusterIP with endpoints
[ ] Live IngressRoute preserves /api/v1
[ ] GET / returns expected frontend HTML
[ ] GET /api/v1/openapi.json returns expected API schema
[ ] New frontend image uses the intended same-origin API configuration
[ ] Hostname/DNS and HTTPS verified separately if configured
[ ] Argo CD synchronization does not reintroduce old routing
```

## Lessons Learned

- Keep public ingress at the controller boundary rather than on each application Service.
- Preserve FastAPI's actual `/api/v1` path unless a deliberate rewrite is implemented and tested.
- Test GET semantics with GET requests, not only HEAD requests.
- Distinguish historical screenshots from validation of newly committed manifests.
- Diagnose ingress layer by layer: DNS/Host header → load balancer → Traefik → IngressRoute → Service endpoints → Pod.
