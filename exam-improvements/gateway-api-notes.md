# Ingress to Gateway API Migration

## Conceptual Mapping

| Ingress | Gateway API |
|---------|-------------|
| IngressClass | GatewayClass |
| Ingress (listener config) | Gateway |
| Ingress rules/paths | HTTPRoute |
| `tls` block | Gateway `listeners[].tls` |
| `rules[].http.paths` | HTTPRoute `rules[].matches` |
| `backend.service` | HTTPRoute `backendRefs` |

---

## Gateway

The Gateway defines listeners (ports, protocols, TLS, hostnames).

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    port: 443
    protocol: HTTPS
    hostname: "app.example.com"
    tls:
      mode: Terminate
      certificateRefs:
      - name: app-tls-secret
        kind: Secret
  - name: http
    port: 80
    protocol: HTTP
    hostname: "app.example.com"
```

### Key Fields

- `gatewayClassName`: References the GatewayClass (like `ingressClassName`)
- `listeners[].hostname`: Can use wildcards like `*.example.com`
- `tls.mode`: `Terminate` (common) or `Passthrough`
- `certificateRefs`: Points to TLS Secret (must be same namespace or use ReferenceGrant)

---

## HTTPRoute

The HTTPRoute defines routing rules (paths, backends, filters).

### Basic Routing

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app-route
spec:
  parentRefs:
  - name: my-gateway
  hostnames:
  - "app.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-svc
      port: 80
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-svc
      port: 80
```

### Key Fields

- `parentRefs`: Must reference the Gateway by name
- `matches[].path.type`: `PathPrefix`, `Exact`, or `RegularExpression`
- `backendRefs`: Service name and port
- **Order matters**: Put specific paths before generic ones (`/api` before `/`)

---

## Multiple Backends (Weighted Routing)

For canary deployments or traffic splitting:

```yaml
rules:
- matches:
  - path:
      type: PathPrefix
      value: /
  backendRefs:
  - name: app-v1
    port: 80
    weight: 80
  - name: app-v2
    port: 80
    weight: 20
```

---

## URL Rewrite (Replaces `rewrite-target` Annotation)

```yaml
rules:
- matches:
  - path:
      type: PathPrefix
      value: /api
  filters:
  - type: URLRewrite
    urlRewrite:
      path:
        type: ReplacePrefixMatch
        replacePrefixMatch: /
  backendRefs:
  - name: api-svc
    port: 80
```

---

## Complete Migration Example

### Before (Ingress)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

### After (Gateway + HTTPRoute)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    port: 443
    protocol: HTTPS
    hostname: "app.example.com"
    tls:
      mode: Terminate
      certificateRefs:
      - name: app-tls-secret
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app-route
spec:
  parentRefs:
  - name: my-gateway
  hostnames:
  - "app.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-svc
      port: 80
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-svc
      port: 80
```

---

## Useful Commands

```bash
# Check available GatewayClasses
kubectl get gatewayclass

# List Gateways and HTTPRoutes
kubectl get gateway
kubectl get httproute

# Verify HTTPRoute attached successfully
kubectl describe httproute my-app-route

# Get field documentation during exam
kubectl explain gateway.spec.listeners.tls
kubectl explain httproute.spec.rules.matches
```

---

## Exam Tips

1. **parentRefs** in HTTPRoute must match the Gateway name exactly
2. **pathType: Prefix** (Ingress) → **type: PathPrefix** (HTTPRoute)
3. TLS config lives in Gateway, not HTTPRoute
4. Multiple HTTPRoutes can attach to one Gateway
5. GatewayClass is usually pre-installed (`nginx`, `istio`, etc.)
6. Secrets for TLS must be in same namespace as Gateway (or use ReferenceGrant)
7. Use `kubectl explain` if you forget field names
