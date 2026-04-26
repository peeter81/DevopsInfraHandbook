# devops/kubernetes.md  
## Kubernetes – DevOps käsiraamat

## Ülevaade

Kubernetes (K8s) on konteinerite orkestreerimise platvorm, mis haldab:

- deployd  
- skaleerimist  
- self‑healingut  
- networkingut  
- storage’it  
- konfiguratsioone ja secretesid  

Allpool on kõige olulisemad käsud, ressursid ja mustrid.

---

# 1. Põhikäsud

## Ressursside vaatamine

```bash
kubectl get pods
kubectl get pods -A
kubectl get svc
kubectl get deploy
kubectl get nodes
```

## Detailne info

```bash
kubectl describe pod <nimi>
kubectl describe node <nimi>
kubectl describe deploy <nimi>
```

## Logid

```bash
kubectl logs <pod>
kubectl logs <pod> -f
kubectl logs <pod> -c <container>
```

## Exec konteinerisse

```bash
kubectl exec -it <pod> -- bash
```

---

# 2. Deploymentid

## Deploymenti näide

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
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
```

## Deployment käsud

```bash
kubectl apply -f deploy.yaml
kubectl rollout status deploy/web
kubectl rollout restart deploy/web
kubectl rollout undo deploy/web
```

---

# 3. Services

## Service tüübid

- **ClusterIP** – vaikimisi, sisemine liiklus  
- **NodePort** – avab pordi igal sõlmel  
- **LoadBalancer** – pilve load balancer  
- **Headless Service** – DNS ilma load balancinguta  

## Service näide

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

---

# 4. ConfigMap & Secret

## ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  MODE: "production"
```

## Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: cGFzc3dvcmQ=
```

---

# 5. Probes

## Liveness probe

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

## Readiness probe

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

---

# 6. Autoscaling

## HPA (Horizontal Pod Autoscaler)

```bash
kubectl autoscale deploy web --min=3 --max=10 --cpu-percent=70
```

---

# 7. Namespaces

## Namespace loomine

```bash
kubectl create ns prod
```

## Namespace kasutamine

```bash
kubectl get pods -n prod
kubectl apply -f app.yaml -n prod
```

---

# 8. RBAC

## ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
```

## Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

## RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rb
subjects:
  - kind: ServiceAccount
    name: app-sa
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io
```

---

# 9. Kubernetes arhitektuuri diagramm

```
+-------------------+
|      kubectl      |
+---------+---------+
          |
          v
+-------------------+
|    API Server     |
+---------+---------+
          |
   +------+------+
   |             |
   v             v
+-------+   +-----------+
| etcd  |   | Controller|
+-------+   +-----------+
                |
                v
          +-----------+
          | Scheduler |
          +-----------+
                |
                v
          +-----------+
          |  Nodes    |
          +-----------+
```

---

# 10. Best Practices

- Kasuta **readinessProbe** + **livenessProbe**  
- Ära kasuta `latest` image tag’i  
- Lisa **resource requests/limits**  
- Kasuta **PodDisruptionBudget**  
- Kasuta **NetworkPolicy**  
- Kasuta **Kustomize** või **Helm** deploydeks  
- Ära tee `kubectl apply` otse prod’i  

---

# Kokkuvõte

Kubernetes on DevOpsi keskne tööriist.  
Selle mõistmine annab sulle kontrolli deployde, skaleerimise, võrgu, turvalisuse ja konfiguratsioonide üle.

