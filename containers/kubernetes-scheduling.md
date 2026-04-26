# 📄 **Fail: `containers/kubernetes-scheduling.md`**

```markdown
# Kubernetes Scheduling – Containers käsiraamat  
## (Scheduler, Affinity, Anti-Affinity, Taints/Tolerations, Resource Requests, Priorities)

## Ülevaade
Kubernetes scheduler otsustab, **millisele node’ile** iga pod paigutatakse.  
See on üks olulisemaid mehhanisme töökoormuste optimeerimiseks ja stabiilsuse tagamiseks.

Scheduler arvestab:

- ressursinõudeid  
- node’i koormust  
- affinity/anti-affinity reegleid  
- taints/tolerations  
- prioriteete  
- topology spread constraints  

---

# 1. Resource Requests & Limits

## 1.1 Requests (scheduler kasutab seda)

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
```

## 1.2 Limits (runtime piirang)

```yaml
resources:
  limits:
    cpu: "1"
    memory: "512Mi"
```

**Scheduler kasutab ainult *requests* väärtusi**.

---

# 2. NodeSelector

Lihtsaim viis pod konkreetsele node’ile suunata.

Node’i märgistamine:

```bash
kubectl label node node1 disktype=ssd
```

Pod:

```yaml
nodeSelector:
  disktype: ssd
```

---

# 3. Node Affinity

Paindlikum kui nodeSelector.

## 3.1 RequiredDuringScheduling

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

## 3.2 PreferredDuringScheduling

```yaml
preferredDuringSchedulingIgnoredDuringExecution:
  - weight: 10
    preference:
      matchExpressions:
        - key: zone
          operator: In
          values: ["eu-west-1a"]
```

---

# 4. Pod Affinity & Anti-Affinity

## 4.1 Pod Affinity (hoia podid koos)

```yaml
podAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: api
      topologyKey: "kubernetes.io/hostname"
```

## 4.2 Pod Anti-Affinity (hoia podid lahus)

```yaml
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: web
      topologyKey: "kubernetes.io/hostname"
```

---

# 5. Taints & Tolerations

## 5.1 Lisa taint node’ile

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

# 6. Topology Spread Constraints

Tagab, et podid jaotatakse ühtlaselt üle node’ide või tsoonide.

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: "topology.kubernetes.io/zone"
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: web
```

---

# 7. Priority Classes

Kõrgema prioriteediga podid võivad madalamaid välja tõrjuda.

Loo priority class:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "High priority workloads"
```

Kasuta podis:

```yaml
priorityClassName: high-priority
```

---

# 8. Scheduler Profiles (advanced)

Kubernetes võimaldab luua kohandatud scheduler profiile:

- scoring plugins  
- filtering plugins  
- preemption poliitikad  

Fail:

```
/etc/kubernetes/scheduler.conf
```

---

# 9. Troubleshooting

### 9.1 Pod stuck in Pending

```bash
kubectl describe pod <nimi>
```

Kontrolli:

- kas node’il on piisavalt CPU/RAM  
- kas affinity reeglid on liiga ranged  
- kas taints takistavad schedulingu  

### 9.2 Taints probleemid

```bash
kubectl describe node node1
```

### 9.3 Resource requests liiga suured

```bash
kubectl top nodes
```

---

# 10. Best Practices

- Kasuta **requests** väärtusi täpselt  
- Ära pane liiga suuri limits’e  
- Kasuta **anti-affinity** HA tagamiseks  
- Kasuta **taints/tolerations** töökoormuste eraldamiseks  
- Kasuta **topology spread constraints** tsoonide vaheliseks jaotuseks  
- Kasuta **priority classes** kriitiliste teenuste jaoks  

---

# Kokkuvõte
Kubernetes scheduling on võimas mehhanism, mis tagab töökoormuste optimaalse ja turvalise jaotuse.  
Affinity, taints, tolerations ja priority classes võimaldavad täielikku kontrolli podide paigutuse üle.
```
