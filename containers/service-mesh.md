# 📄 **Fail: `containers/service-mesh.md`**

```markdown
# Service Mesh – Containers käsiraamat  
## (Istio, Linkerd, mTLS, sidecar, traffic shaping, observability)

## Ülevaade
Service mesh on mikroteenuste võrgukiht, mis lisab:

- turvalise suhtluse (mTLS)
- liikluse juhtimise (traffic shaping)
- retry’d, timeout’id, circuit breaking
- observability (metrics, tracing, logs)
- poliitikad ja autoriseerimine

Kõige populaarsemad:
- **Istio**
- **Linkerd**
- **Consul Connect**
- **Kuma**

---

# 1. Service Mesh arhitektuur

Service mesh koosneb kahest komponendist:

### 1.1 Control Plane
- haldab konfiguratsiooni  
- pushib poliitikaid sidecar’idele  
- jälgib mesh’i tervist  

### 1.2 Data Plane
- sidecar proxy (nt Envoy)  
- interceptib kogu liikluse  
- rakendab poliitikaid  

---

# 2. Sidecar pattern

Iga pod saab täiendava konteineri:

```
[ app container ]  <-- sinu rakendus
[ sidecar proxy ]  <-- Envoy/Linkerd-proxy
```

Sidecar:
- krüpteerib liikluse  
- mõõdab latentsust  
- rakendab retry’d  
- teeb load balancingut  

---

# 3. Istio

Kõige funktsionaalsem service mesh.

## 3.1 Install (istioctl)

```bash
istioctl install --set profile=demo -y
```

## 3.2 Namespace injection

```bash
kubectl label namespace default istio-injection=enabled
```

## 3.3 VirtualService (traffic routing)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: web
spec:
  hosts:
    - web
  http:
    - route:
        - destination:
            host: web
            subset: v1
```

## 3.4 DestinationRule (subsets)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: web
spec:
  host: web
  subsets:
    - name: v1
      labels:
        version: v1
```

---

# 4. Linkerd

Kergem ja lihtsam kui Istio.

## 4.1 Install

```bash
linkerd install | kubectl apply -f -
```

## 4.2 Inject

```bash
kubectl get deploy web -o yaml | linkerd inject - | kubectl apply -f -
```

## 4.3 Dashboard

```bash
linkerd viz dashboard
```

---

# 5. mTLS (mutual TLS)

Service mesh krüpteerib kogu liikluse automaatselt.

### Istio:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
```

### Linkerd:
mTLS on vaikimisi **alati sees**.

---

# 6. Traffic Shaping

## 6.1 Canary release

```yaml
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

## 6.2 Fault injection

```yaml
fault:
  delay:
    fixedDelay: 5s
    percentage:
      value: 50
```

---

# 7. Observability

Service mesh lisab automaatselt:

- Prometheus metrics  
- Grafana dashboards  
- Jaeger tracing  
- Kiali (Istio UI)  

Näide:

```bash
istioctl dashboard kiali
```

---

# 8. Policy & Security

## 8.1 AuthorizationPolicy

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-web
spec:
  selector:
    matchLabels:
      app: web
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/default/sa/frontend"]
```

---

# 9. Troubleshooting

### 9.1 Pod ei saa ühendust
Kontrolli sidecar injection’it:

```bash
kubectl get pod -o json | jq '.spec.containers[].name'
```

### 9.2 mTLS konflikt
Kontrolli PeerAuthentication:

```bash
kubectl get peerauthentication
```

### 9.3 Traffic routing ei tööta
Kontrolli VirtualService:

```bash
kubectl describe virtualservice web
```

---

# 10. Best Practices

- Kasuta **Linkerd** kui vajad lihtsust  
- Kasuta **Istio** kui vajad maksimaalset kontrolli  
- Luba **mTLS** kogu klastris  
- Kasuta **canary** ja **blue/green** deploy strateegiaid  
- Kasuta **Kiali** või **Linkerd Viz** jälgimiseks  
- Ära kasuta service mesh’i väikestes projektides (liigne keerukus)  

---

# Kokkuvõte
Service mesh lisab mikroteenustele turvalise, nähtava ja kontrollitava võrgukihi.  
Istio ja Linkerd on kõige populaarsemad lahendused, pakkudes mTLS‑i, traffic shapingut, observability’t ja poliitikate haldust.
```
