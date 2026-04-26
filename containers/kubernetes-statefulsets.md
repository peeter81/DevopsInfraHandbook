# 📄 **Fail: `containers/kubernetes-statefulsets.md`**

```markdown
# Kubernetes StatefulSets – Containers käsiraamat  
## (Stateful workloads, stable identities, persistent storage, ordered startup/shutdown)

## Ülevaade
StatefulSet on Kubernetes’i töökoormuse tüüp, mis on mõeldud **püsiva olekuga** rakendustele:

- andmebaasid  
- sõnumijärjekorrad  
- klastrid (Kafka, Zookeeper, Redis, ElasticSearch)  
- kõik, mis vajab stabiilset identiteeti ja salvestusruumi  

StatefulSet tagab:

- **püsivad nimed** (pod-0, pod-1, pod-2)  
- **püsivad PVC-d** iga podi jaoks  
- **järjekorras käivitamise ja peatamise**  
- **stabiilse võrguidentiteedi**  

---

# 1. StatefulSet arhitektuur

StatefulSet koosneb:

- **StatefulSet objektist**  
- **Headless Service’ist** (DNS identiteet)  
- **PVC-dest** (püsiv storage)  

Headless Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  clusterIP: None
  selector:
    app: db
```

---

# 2. StatefulSet näide

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: "db"
  replicas: 3
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: postgres
          image: postgres
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

---

# 3. Püsivad nimed ja DNS

StatefulSet loob podid:

```
db-0
db-1
db-2
```

DNS kirjed:

```
db-0.db.default.svc.cluster.local
db-1.db.default.svc.cluster.local
db-2.db.default.svc.cluster.local
```

Igal podil on **unikaalne hostname**, mis ei muutu.

---

# 4. volumeClaimTemplates

Iga pod saab **oma PVC**:

```
data-db-0
data-db-1
data-db-2
```

PVC-d EI kustutata automaatselt, isegi kui StatefulSet kustutada.

---

# 5. Ordered startup & shutdown

StatefulSet käivitab podid järjekorras:

1. `db-0`
2. `db-1`
3. `db-2`

Ja peatab vastupidises järjekorras.

See on kriitiline:

- Zookeeper  
- Kafka  
- Redis Cluster  
- ElasticSearch  

---

# 6. Update Strategies

## 6.1 RollingUpdate (vaikimisi)

```yaml
updateStrategy:
  type: RollingUpdate
```

## 6.2 OnDelete

Podid uuenevad ainult siis, kui need käsitsi kustutada:

```yaml
updateStrategy:
  type: OnDelete
```

Sobib andmebaasidele, kus automaatne restart võib olla ohtlik.

---

# 7. StatefulSet vs Deployment

| Funktsioon | StatefulSet | Deployment |
|-----------|-------------|------------|
| Püsiv identiteet | ✔ | ✖ |
| Püsiv storage iga podi jaoks | ✔ | ✖ |
| Järjestatud käivitus | ✔ | ✖ |
| Järjestatud peatamine | ✔ | ✖ |
| Sobib andmebaasidele | ✔ | ✖ |
| Sobib stateless teenustele | ✖ | ✔ |

---

# 8. Troubleshooting

### 8.1 Pod ei käivitu

```bash
kubectl describe pod db-0
```

### 8.2 PVC ei teki

Kontrolli StorageClass:

```bash
kubectl get storageclass
```

### 8.3 DNS ei tööta

Kontrolli headless service:

```bash
kubectl get svc db
```

### 8.4 Rolling update jäi kinni

```bash
kubectl rollout status statefulset/db
```

---

# 9. Best Practices

- Kasuta StatefulSet’i **ainult** stateful rakenduste jaoks  
- Kasuta **Headless Service** DNS identiteediks  
- Kasuta **volumeClaimTemplates** püsiva storage jaoks  
- Ära kasuta RWX storage’t andmebaaside jaoks  
- Kasuta **OnDelete** update strateegiat kriitiliste DB-de puhul  
- Tee regulaarseid backup’e (Velero, Restic)  

---

# Kokkuvõte
StatefulSet on Kubernetes’i mehhanism püsiva olekuga töökoormuste jaoks.  
See tagab stabiilse identiteedi, püsiva storage’i ja järjekorras käivitamise — kõik, mida andmebaasid ja klastrid vajavad.
```
