# 📄 **Fail: `containers/kubernetes-ingress.md`**

```markdown
# Kubernetes Ingress – Containers käsiraamat  
## (Ingress, IngressClass, NGINX, Traefik, TLS, Path routing, Host routing)

## Ülevaade
Ingress on Kubernetes’i HTTP/HTTPS liikluse haldamise mehhanism.  
Ingress võimaldab:

- domeenipõhist routingut  
- path‑põhist routingut  
- TLS sertifikaate  
- load balancingut  
- rate limitingut  
- autentimist (ingress controlleri kaudu)

Ingress **ei tööta ilma Ingress Controllerita**.

---

# 1. Ingress Controllerid

Kõige populaarsemad:

- **NGINX Ingress Controller**  
- **Traefik**  
- **HAProxy**  
- **Istio Gateway**  
- **Kong**  
- **Contour**

Install (NGINX):

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx ingress-nginx/ingress-nginx
```

---

# 2. Ingress Resource

Lihtsaim näide:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
spec:
  rules:
    - host: web.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

---

# 3. Path Routing

```yaml
paths:
  - path: /api
    pathType: Prefix
    backend:
      service:
        name: api
        port:
          number: 8080
  - path: /static
    pathType: Prefix
    backend:
      service:
        name: static
        port:
          number: 80
```

---

# 4. Host Routing

```yaml
rules:
  - host: api.example.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: api
              port:
                number: 8080
  - host: web.example.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: web
              port:
                number: 80
```

---

# 5. TLS (HTTPS)

```yaml
tls:
  - hosts:
      - web.example.com
    secretName: web-tls
```

TLS secret:

```bash
kubectl create secret tls web-tls \
  --cert=cert.pem \
  --key=key.pem
```

---

# 6. Cert-Manager (automaatne Let's Encrypt)

Install:

```bash
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager --set installCRDs=true
```

ClusterIssuer:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

Ingress automaatne TLS:

```yaml
annotations:
  cert-manager.io/cluster-issuer: "letsencrypt"
```

---

# 7. NGINX Ingress annotatsioonid

## 7.1 Rate limiting

```yaml
nginx.ingress.kubernetes.io/limit-rps: "10"
```

## 7.2 Rewrite

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
```

## 7.3 Basic auth

```yaml
nginx.ingress.kubernetes.io/auth-type: basic
nginx.ingress.kubernetes.io/auth-secret: basic-auth
```

---

# 8. Troubleshooting

### 8.1 Ingress ei tööta

```bash
kubectl get ingress
kubectl describe ingress web
```

### 8.2 DNS ei lahenda

Kontrolli domeeni A‑kirjet.

### 8.3 TLS error

```bash
kubectl describe certificate
kubectl describe challenge
```

### 8.4 NGINX crash

```bash
kubectl -n ingress-nginx logs deploy/ingress-nginx-controller
```

---

# 9. Best Practices

- Kasuta **Ingress** HTTP/HTTPS liikluse jaoks  
- Kasuta **LoadBalancer** ainult väliste teenuste jaoks  
- Kasuta **cert-manager** automaatseks TLS‑iks  
- Kasuta **rate limiting** ja **auth** annotatsioone  
- Ära kasuta NodePort tootmises  
- Kasuta **pathType: Prefix**  
- Kasuta **separate IngressClass** mitme controlleri puhul  

---

# Kokkuvõte
Ingress on Kubernetes’i HTTP/HTTPS gateway, mis võimaldab domeenipõhist routingut, TLS‑i ja liikluse kontrolli.  
Ingress Controller (NGINX, Traefik, Istio) on selle toimimiseks hädavajalik.
```
