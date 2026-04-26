# 📄 **Fail: `containers/kubernetes-daemonsets.md`**

```markdown
# Kubernetes DaemonSets – Containers käsiraamat  
## (Node-level workloads, agents, monitoring, logging, CNI, storage daemons)

## Ülevaade
DaemonSet tagab, et **iga node** (või valitud node’id) käivitavad **täpselt ühe podi**.  
See on kriitiline node‑tasemel agentide ja infrastruktuurikomponentide jaoks.

Kasutusjuhtumid:

- logiagent (Fluentd, Promtail)  
- monitoring agent (Node Exporter)  
- CNI pluginad (Calico, Cilium)  
- storage agendid (Ceph, Longhorn)  
- node‑tasemel sidecarid  
- turvaagendid (Falco)

---

# 1. DaemonSet näide

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

# 2. NodeSelector

DaemonSet võib töötada ainult teatud node’idel.

```yaml
nodeSelector:
  node-role.kubernetes.io/worker: ""
```

---

# 3. Tolerations

DaemonSet’id vajavad sageli toleratsioone, et töötada ka master‑node’idel.

```yaml
tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
```

---

# 4. Update Strategy

## 4.1 RollingUpdate (vaikimisi)

```yaml
updateStrategy:
  type: RollingUpdate
```

## 4.2 OnDelete

```yaml
updateStrategy:
  type: OnDelete
```

---

# 5. DaemonSet vs Deployment

| Funktsioon | DaemonSet | Deployment |
|-----------|-----------|------------|
| Üks pod iga node’i kohta | ✔ | ✖ |
| Sobib agentidele | ✔ | ✖ |
| Sobib stateless teenustele | ✖ | ✔ |
| Rolling updates | ✔ | ✔ |
| Node‑tasemel töö | ✔ | ✖ |

---

# 6. Levimad DaemonSet’id tootmises

### 6.1 Monitoring
- Node Exporter  
- Promtail  
- Datadog Agent  
- New Relic Infra Agent  

### 6.2 Logging
- Fluentd  
- Vector  
- Filebeat  

### 6.3 Networking
- Calico  
- Cilium  
- Weave  

### 6.4 Storage
- Longhorn  
- Ceph OSD/Mon  

### 6.5 Security
- Falco  
- AppArmor loader  

---

# 7. Troubleshooting

### 7.1 Pod ei käivitu mõnel node’il

```bash
kubectl describe daemonset <nimi>
kubectl describe node <node>
```

### 7.2 Taints takistavad käivitamist

Kontrolli:

```bash
kubectl describe node
```

### 7.3 Image pull error

```bash
kubectl get events
```

---

# 8. Best Practices

- Kasuta DaemonSet’e **ainult node‑tasemel agentide** jaoks  
- Lisa vajalikud **tolerations**, et töötada kõikidel node’idel  
- Kasuta **RollingUpdate** strateegiat  
- Ära pane DaemonSet’i pod’e käsitsi kustutama — scheduler taastab need kohe  
- Kasuta **resource requests** vältimaks node’i ülekoormust  

---

# Kokkuvõte
DaemonSet on Kubernetes’i mehhanism, mis tagab ühe podi igal node’il.  
See on hädavajalik monitoringu, logimise, võrgu ja storage agentide jaoks.
```
