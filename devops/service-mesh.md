# devops/service-mesh.md  
## Service Mesh – DevOps käsiraamat  
### (Istio, Linkerd, Envoy)

## Ülevaade

Service mesh lahendab mikroteenuste vahelise suhtluse keerukuse.  
See lisab:

- **Traffic management** (canary, blue‑green, A/B)
- **Security** (mTLS, RBAC, policy)
- **Observability** (metrics, logs, tracing)
- **Reliability** (retry, timeout, circuit breaker)

Kõik see toimub **sidecar‑proxy** (nt Envoy) kaudu.

---

# 1. Istio

Istio on kõige võimekam service mesh — enterprise standard.

## Olulised ressursid

### VirtualService  
Routing reeglid (host, path, headers).

### DestinationRule  
Subsets, load balancing, connection pool, circuit breaking.

### Gateway  
Ingress / egress L7 gateway.

### PeerAuthentication  
mTLS poliitika.

### AuthorizationPolicy  
RBAC‑stiilis ligipääs.

---

## 1.1 VirtualService näide (canary release)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web
spec:
  hosts:
    - web.example.com
  http:
    - route:
        - destination:
            host: web
            subset: v1
          weight: 90
        - destination:
            host: web
            subset: v2
          weight: 10
```

---

## 1.2 DestinationRule näide

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: web
spec:
  host: web
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

---

## 1.3 Istio käsud

```
istioctl install
istioctl proxy-status
istioctl analyze
```

---

# 2. Linkerd

Linkerd on kergem, lihtsam ja vaikimisi mTLS‑iga.

## Olulised käsud

```
linkerd check
linkerd viz install | kubectl apply -f -
linkerd inject deployment.yaml
linkerd tap deploy/web
```

## Eelised

- Väga lihtne paigaldada  
- Väike overhead  
- Väga stabiilne  
- Hea observability  

---

# 3. Envoy

Envoy on L7 proxy, mida Istio ja paljud meshid kasutavad sidecarina.

## Envoy rollid

- HTTP/1.1, HTTP/2, gRPC proxy  
- mTLS, JWT auth  
- Retry, timeout, circuit breaking  
- Metrics (Prometheus)  
- Access logid  

---

## 3.1 Envoy standalone näide

```yaml
static_resources:
  listeners:
    - name: listener_0
      address:
        socket_address: { address: 0.0.0.0, port_value: 8080 }
      filter_chains:
        - filters:
            - name: envoy.filters.network.http_connection_manager
              typed_config:
                "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
                stat_prefix: ingress_http
                route_config:
                  name: local_route
                  virtual_hosts:
                    - name: backend
                      domains: ["*"]
                      routes:
                        - match: { prefix: "/" }
                          route: { cluster: backend }
  clusters:
    - name: backend
      connect_timeout: 0.25s
      type: logical_dns
      load_assignment:
        cluster_name: backend
        endpoints:
          - lb_endpoints:
              - endpoint:
                  address:
                    socket_address: { address: backend, port_value: 80 }
```

---

# 4. Service Mesh mustrid

## Canary release  
- 90% → v1  
- 10% → v2  
- Automaatne rollback  

## Blue‑green  
- Kaks täisversiooni  
- Lülitus DNS/VirtualService tasemel  

## Traffic shadowing  
- Kopeerib liikluse uuele teenusele  
- Vastust ei kasutata  

## mTLS everywhere  
- Teenused autentivad üksteist  
- Krüpteeritud side  

---

# 5. Service Mesh arhitektuuri diagramm

```
          +-------------------+
          |   IngressGateway  |
          +---------+---------+
                    |
                    v
          +-------------------+
          |   VirtualService  |
          +---------+---------+
                    |
                    v
          +-------------------+
          |     Service       |
          +---------+---------+
                    |
        +-----------+-----------+
        |                       |
        v                       v
+---------------+       +---------------+
|   Pod v1      |       |   Pod v2      |
| +-----------+ |       | +-----------+ |
| |  app      | |       | |  app      | |
| +-----------+ |       | +-----------+ |
| | sidecar   | |       | | sidecar   | |
| +-----------+ |       | +-----------+ |
+---------------+       +---------------+
```

---

# 6. Best Practices

- Kasuta **mTLS** kõikjal  
- Kasuta **traffic shifting** (canary)  
- Kasuta **retry + timeout** poliitikaid  
- Kasuta **circuit breaking**  
- Ära kasuta `latest` image tag’i  
- Kasuta **Kustomize** või **Helm** mesh konfiguratsioonideks  
- Kasuta **ArgoCD / FluxCD** GitOps deploydeks  

---

# Kokkuvõte

Service mesh lahendab mikroteenuste vahelise suhtluse keerukuse, lisades turvalisuse, nähtavuse ja kontrolli.  
Istio on kõige võimekam, Linkerd kõige lihtsam ja Envoy kõige paindlikum komponent.

