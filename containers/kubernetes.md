# 📄 **Fail: `containers/kubernetes.md`**

```markdown
# Kubernetes – Containers käsiraamat  
## (Pods, Deployments, Services, Ingress, Volumes, ConfigMaps, Secrets)

## Ülevaade
Kubernetes (K8s) on konteinerite orkestreerimise platvorm, mis haldab:

- konteinerite käivitamist  
- skaleerimist  
- võrke  
- salvestust  
- konfiguratsiooni  
- tervisekontrolle  
- automaatset taastumist  

Kubernetes on tööstusstandard suurte ja kriitiliste süsteemide haldamiseks.

---

# 1. Põhimõisted

### **Pod**
Väikseim deploy‑ühik. Sisaldab ühte või mitut konteinerit.

### **Deployment**
Deklaratiivne podide haldus (rolling updates, rollback).

### **Service**
Püsiv võrguaadress podidele.

### **Ingress**
HTTP/HTTPS routing välismaailmast.

### **ConfigMap**
Rakenduse konfiguratsioon.

### **Secret**
Krüpteeritud konfiguratsioon (paroolid, võtmed).

### **Volume**
Püsiv andmesalvestus.

---

# 2. Kubectl põhitöö

## 2.1 Kontrolli klastrit

```bash
kubectl get nodes
kubectl cluster-info
```

## 2.2 Ressursside vaatamine

```bash
kubectl get pods
kubectl get deployments
kubectl get svc
kubectl get ingress
```

Detailne:

```bash
kubectl describe pod <nimi>
```

---

# 3. Deployment

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

Loo:

```bash
kubectl apply -f web.yaml
```

---

# 4. Service

### ClusterIP (vaikimisi)

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
      targetPort: 80
```

### NodePort

```yaml
type: NodePort
ports:
  - port: 80
    nodePort: 30080
```

### LoadBalancer (pilves)

```yaml
type: LoadBalancer
```

---

# 5. Ingress

Näide:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
spec:
  rules:
  - host: example.com
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

# 6. ConfigMaps

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: production
```

Mount konteinerisse:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

---

# 7. Secrets

```bash
kubectl create secret generic db-pass --from-literal=password=salajane
```

Kasuta:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-pass
        key: password
```

---

# 8. Volumes

### EmptyDir (ajutine)

```yaml
volumes:
  - name: cache
    emptyDir: {}
```

### HostPath (hosti kataloog)

```yaml
hostPath:
  path: /data
```

### PersistentVolumeClaim (püsiv storage)

```yaml
volumeMounts:
  - mountPath: /var/lib/mysql
    name: data
```

---

# 9. Health checks

### Liveness probe

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
```

### Readiness probe

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
```

---

# 10. Autoscaling

```bash
kubectl autoscale deployment web --min=3 --max=10 --cpu-percent=70
```

---

# 11. Rolling updates ja rollback

### Uuenda image

```bash
kubectl set image deployment/web nginx=nginx:1.25
```

### Rollback

```bash
kubectl rollout undo deployment/web
```

---

# 12. Best Practices

- Kasuta **Deployments**, mitte üksikuid pod’e  
- Kasuta **Services** podide vaheliseks suhtluseks  
- Kasuta **Ingress** HTTP routinguks  
- Hoia konfiguratsioon **ConfigMap**’ides  
- Hoia paroolid **Secrets**’ites  
- Kasuta **liveness** ja **readiness** probe’e  
- Kasuta **autoscaling**’ut koormuse järgi  
- Ära hoia andmeid konteineris — kasuta **Volumes**  

---

# Kokkuvõte
Kubernetes on võimas orkestreerimisplatvorm, mis haldab konteinerite elutsüklit, võrke, salvestust ja skaleerimist.  
Deployments, Services, Ingress, ConfigMaps ja Secrets moodustavad Kubernetes’i põhikomponendid, mis võimaldavad ehitada skaleeruvaid ja töökindlaid süsteeme.
```
