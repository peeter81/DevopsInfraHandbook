# 📄 **Fail: `containers/kubernetes-basics.md`**

```markdown
# Kubernetes Basics – Containers käsiraamat  
## (Pods, Deployments, Services, ConfigMaps, Secrets, Namespaces)

## Ülevaade
Kubernetes (K8s) on konteinerite orkestreerimise tööstusstandard.  
See haldab:

- konteinerite deploy  
- skaleerimist  
- self‑healing  
- networking  
- storage  
- konfiguratsiooni  
- turvalisust  

See fail katab kõige olulisemad põhikontseptsioonid.

---

# 1. Kubernetes arhitektuur

## 1.1 Control Plane
- API Server  
- Scheduler  
- Controller Manager  
- etcd  

## 1.2 Worker Nodes
- Kubelet  
- Kube‑proxy  
- Container runtime (containerd/CRI‑O)  

---

# 2. Pod

Pod on väikseim deploy‑ühik.

Näide:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
    - name: nginx
      image: nginx
```

Loo:

```bash
kubectl apply -f pod.yaml
```

---

# 3. Deployment

Deployment haldab podide elutsüklit.

Näide:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx
```

---

# 4. Service

Service annab püsiva IP ja load balancing’u.

## 4.1 ClusterIP (vaikimisi)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - port: 80
```

## 4.2 NodePort

```yaml
type: NodePort
```

## 4.3 LoadBalancer

Pilvepakkuja loob LB.

---

# 5. ConfigMap

Konfiguratsioon ilma paroolideta.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  MODE: production
```

Mount:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

---

# 6. Secret

Paroolid ja võtmed.

```bash
kubectl create secret generic db --from-literal=password=123
```

Kasuta:

```yaml
env:
  - name: DB_PASS
    valueFrom:
      secretKeyRef:
        name: db
        key: password
```

---

# 7. Namespaces

Loo namespace:

```bash
kubectl create ns dev
```

Vaheta:

```bash
kubectl config set-context --current --namespace=dev
```

---

# 8. Kubectl käsud

Vaata:

```bash
kubectl get pods
kubectl get svc
kubectl get deploy
kubectl get nodes
```

Detailid:

```bash
kubectl describe pod web
```

Logid:

```bash
kubectl logs web
```

---

# 9. Rolling updates

```bash
kubectl rollout restart deployment web
```

Rollback:

```bash
kubectl rollout undo deployment web
```

---

# 10. Health checks

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

---

# 11. Best Practices

- Kasuta Deployments, mitte üksikuid pod’e  
- Kasuta Services load balancinguks  
- Kasuta ConfigMap + Secret konfiguratsiooniks  
- Kasuta Namespaces projektide eraldamiseks  
- Kasuta readiness/liveness probe’e  
- Ära hoia paroole image’is  

---

# Kokkuvõte
Kubernetes põhineb podidel, deploymentidel, teenustel ja konfiguratsioonihalduse mehhanismidel.  
Need moodustavad kogu orkestreerimise vundamendi.
```
