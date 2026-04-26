# 📄 **Fail: `containers/kubernetes-operators.md`**

```markdown
# Kubernetes Operators – Containers käsiraamat  
## (CRD, Controller, Reconciliation Loop, Operator SDK, Helm Operators)

## Ülevaade
Operatorid on Kubernetes’i kõige võimsam mehhanism keerukate rakenduste automatiseerimiseks.

Operator =  
**CRD (Custom Resource Definition) + Controller (reconciliation loop)**

Operatorid võimaldavad:

- automaatset deploy’d  
- automaatset backup’i  
- automaatset skaleerimist  
- automaatset self‑healingut  
- versiooniuuendusi  
- klastrite loomist (Kafka, Elastic, Redis, PostgreSQL)

Operatorid on nagu “mini‑Kubernetes Kubernetes’i sees”.

---

# 1. CRD (Custom Resource Definition)

CRD lisab Kubernetes’ile uue API objekti.

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

Pärast CRD loomist saab kasutada:

```bash
kubectl get databases
```

---

# 2. Custom Resource (CR)

CR on CRD instants.

Näide:

```yaml
apiVersion: example.com/v1
kind: Database
metadata:
  name: mydb
spec:
  size: small
  replicas: 3
```

Operator loeb seda ja teeb vajalikud muudatused.

---

# 3. Controller

Controller jälgib CR-i ja tagab, et tegelik seis vastab soovitud seisule.

See on **reconciliation loop**:

```
desired state → controller → actual state
```

Kui midagi muutub:

- pod crashib  
- PVC kaob  
- config muutub  

→ Operator parandab selle automaatselt.

---

# 4. Operator SDK

Operator SDK võimaldab kirjutada operatoreid:

- Go Operator  
- Helm Operator  
- Ansible Operator  

Näide uue operatori loomisest:

```bash
operator-sdk init --domain example.com --owner "Peeter"
```

---

# 5. Helm Operator

Helm chart → automaatne operator.

Loo:

```bash
operator-sdk create api --group apps --version v1 --kind WebApp --helm-chart ./chart
```

Operator haldab chart’i automaatselt.

---

# 6. Ansible Operator

Operatori loogika kirjutatakse Ansible playbook’idesse.

Sobib:

- lihtsamate operatsioonide jaoks  
- kui ei soovi Go koodi kirjutada  

---

# 7. Operator kasutusjuhtumid

### 7.1 Andmebaasid
- PostgreSQL Operator  
- MySQL Operator  
- MongoDB Operator  

### 7.2 Message brokers
- Kafka Operator  
- RabbitMQ Operator  

### 7.3 Storage
- Ceph Rook Operator  
- Longhorn Operator  

### 7.4 Observability
- Prometheus Operator  
- Loki Operator  

### 7.5 Security
- Vault Operator  
- Cert-Manager Operator  

---

# 8. Operator vs Helm Chart

| Funktsioon | Helm Chart | Operator |
|-----------|------------|----------|
| Deploy | ✔ | ✔ |
| Upgrade | ✔ | ✔ |
| Automaatne self‑healing | ✖ | ✔ |
| Automaatne backup | ✖ | ✔ |
| Automaatne skaleerimine | ✖ | ✔ |
| Reconciliation loop | ✖ | ✔ |
| Sobib keerukatele süsteemidele | ✖ | ✔ |

---

# 9. Troubleshooting

### 9.1 CR ei tööta

```bash
kubectl describe crd <nimi>
```

### 9.2 Operator crashib

```bash
kubectl logs deploy/<operator>
```

### 9.3 Reconciliation loop ei tööta

Kontrolli events:

```bash
kubectl get events
```

---

# 10. Best Practices

- Kasuta operatoreid **keerukate** süsteemide jaoks  
- Ära kirjuta operatorit, kui Helm chart piisab  
- Kasuta **Operator SDK** uute operatorite loomiseks  
- Kasuta **CRD versioonimist**  
- Lisa **status** väli CR‑idele  
- Kasuta **finalizers** ressursside korrektseks kustutamiseks  

---

# Kokkuvõte
Operatorid laiendavad Kubernetes’i API‑t ja võimaldavad täielikult automatiseerida keerukaid rakendusi.  
CRD + Controller + Reconciliation Loop = autonoomne süsteem, mis haldab ennast ise.
```
