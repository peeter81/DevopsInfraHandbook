# 📄 **Fail: `containers/kubernetes-advanced.md`**

```markdown
# Kubernetes Advanced – Containers käsiraamat  
## (StatefulSets, DaemonSets, Jobs, CRDs, RBAC, Taints/Tolerations, Affinity, HPA, PDB)

## Ülevaade
See fail katab Kubernetes’i edasijõudnud funktsioonid, mis on vajalikud tootmiskeskkondades:

- keerukad workloadid  
- scheduling poliitikad  
- autoscaling  
- kõrge kättesaadavus  
- turvalisus ja RBAC  
- CRD-d ja operatorid  

---

# 1. StatefulSet

StatefulSet tagab:

- püsivad nimed (pod-0, pod-1, pod-2)  
- püsivad PVC-d  
- järjekorras käivitamise ja peatamise  

Sobib:

- PostgreSQL  
- MySQL  
- Kafka  
- Zookeeper  
- ElasticSearch  

Näide:

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
```

---

# 2. DaemonSet

DaemonSet käivitab **ühe podi iga node’i kohta**.

Kasutus:

- log agent (Fluentd, Promtail)  
- monitoring agent (Node Exporter)  
- storage agent (Ceph, Longhorn)  
- CNI pluginad  

Näide:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
        - name: exporter
          image: prom/node-exporter
```

---

# 3. Jobs ja CronJobs

## 3.1 Job

Ühekordne töö:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup
spec:
  template:
    spec:
      containers:
        - name: backup
          image: alpine
          command: ["sh", "-c", "echo backup"]
      restartPolicy: Never
```

## 3.2 CronJob

Ajastatud töö:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly
spec:
  schedule: "0 3 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: job
              image: alpine
              command: ["echo", "hello"]
          restartPolicy: Never
```

---

# 4. RBAC (Role-Based Access Control)

## 4.1 Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

## 4.2 RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-reader
subjects:
  - kind: User
    name: peeter
roleRef:
  kind: Role
  name: reader
  apiGroup: rbac.authorization.k8s.io
```

---

# 5. Taints & Tolerations

Node tasemel piirangud.

## 5.1 Lisa taint

```bash
kubectl taint nodes node1 role=db:NoSchedule
```

## 5.2 Pod peab toleratsiooni omama

```yaml
tolerations:
  - key: "role"
    operator: "Equal"
    value: "db"
    effect: "NoSchedule"
```

---

# 6. Affinity & Anti-Affinity

## 6.1 Node affinity

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values: ["ssd"]
```

## 6.2 Pod anti-affinity

```yaml
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: web
      topologyKey: "kubernetes.io/hostname"
```

---

# 7. Horizontal Pod Autoscaler (HPA)

```bash
kubectl autoscale deployment web --cpu-percent=70 --min=2 --max=10
```

---

# 8. Pod Disruption Budget (PDB)

Tagab, et liiga palju pod’e ei lõpetata korraga.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```

---

# 9. Custom Resource Definitions (CRD)

CRD võimaldab luua uusi API tüüpe.

Näide:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  names:
    kind: Database
    plural: databases
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
```

CRD-d on aluseks **Operatoritele**.

---

# 10. Best Practices

- Kasuta StatefulSet’i andmebaaside jaoks  
- Kasuta DaemonSet’i agentide jaoks  
- Kasuta RBAC minimaalsete õigustega  
- Kasuta taints/tolerations töökoormuste eraldamiseks  
- Kasuta affinity’t workload placement’i kontrollimiseks  
- Kasuta HPA autoscalinguks  
- Kasuta PDB-d kõrge kättesaadavuse tagamiseks  
- Kasuta CRD-sid ja operatoreid keerukate süsteemide haldamiseks  

---

# Kokkuvõte
Kubernetes advanced funktsioonid võimaldavad luua stabiilseid, skaleeritavaid ja turvalisi tootmisklastrid.  
StatefulSets, DaemonSets, RBAC, autoscaling ja CRD-d on professionaalse K8s töövoo alused.
```
