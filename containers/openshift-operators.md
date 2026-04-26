# 📄 **Fail: `containers/openshift-operators.md`**

```markdown
# OpenShift Operators – Containers käsiraamat  
## (OperatorHub, OLM, CRD, lifecycle management, automation)

## Ülevaade
OpenShift Operators on mehhanism keerukate rakenduste ja teenuste automaatseks haldamiseks Kubernetes klastris.

Operatorid võimaldavad automatiseerida:

- paigaldamist  
- konfiguratsiooni  
- uuendamist  
- skaleerimist  
- varundamist  
- taastamist  

Operatorid kasutavad **CRD-sid (Custom Resource Definitions)** ja **OLM-i (Operator Lifecycle Manager)**.

---

# 1. OperatorHub

OpenShift sisaldab sisseehitatud OperatorHub’i:

- Red Hat Certified Operators  
- Community Operators  
- Marketplace Operators  

Vaata:

```bash
oc get packagemanifests -n openshift-marketplace
```

---

# 2. Operator Lifecycle Manager (OLM)

OLM haldab:

- Operatorite installi  
- Versiooniuuendusi  
- Channel’eid (stable, fast, candidate)  
- Subscription’e  

OLM komponendid:

```
openshift-operator-lifecycle-manager
openshift-marketplace
```

---

# 3. Operator install

### 3.1 Install GUI kaudu
OpenShift Console → OperatorHub → vali operator → Install.

### 3.2 Install YAML kaudu

Näide Subscription:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: my-operator
  namespace: openshift-operators
spec:
  channel: stable
  name: my-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

Loo:

```bash
oc apply -f subscription.yaml
```

---

# 4. Custom Resources (CR)

Operatorid lisavad uusi ressursitüüpe.

Näide PostgreSQL CR:

```yaml
apiVersion: postgres-operator.crunchydata.com/v1beta1
kind: PostgresCluster
metadata:
  name: mydb
spec:
  instances:
    - name: primary
      replicas: 1
```

Loo:

```bash
oc apply -f mydb.yaml
```

Operator haldab automaatselt:

- deployment’e  
- backup’e  
- failover’it  
- konfiguratsiooni  

---

# 5. Operator Channels

Operatorid võivad pakkuda erinevaid kanaleid:

- **stable** – tootmiskeskkond  
- **fast** – uuemad funktsioonid  
- **candidate** – testimiseks  

Näide Subscription:

```yaml
spec:
  channel: stable
```

---

# 6. Operator Upgrade

OLM uuendab automaatselt Operatorit, kui Subscription seda lubab.

Kontrolli:

```bash
oc get csv -n openshift-operators
```

CSV = ClusterServiceVersion

---

# 7. Operator uninstall

Eemalda Subscription:

```bash
oc delete subscription my-operator -n openshift-operators
```

Eemalda CRD-d (vajadusel):

```bash
oc delete crd <nimi>
```

---

# 8. Operator tüübid

| Tüüp | Kirjeldus |
|------|-----------|
| **Helm Operator** | lihtne, Helm chart’i wrapper |
| **Ansible Operator** | loogika Ansible playbookidega |
| **Go Operator** | kõige võimsam, täielik kontroll |

---

# 9. Levinud Operatorid

- **PostgreSQL Operator**  
- **MongoDB Operator**  
- **ElasticSearch Operator**  
- **Kafka (Strimzi) Operator**  
- **Jaeger Operator**  
- **Service Mesh Operator**  
- **GitOps Operator**  
- **OpenShift Pipelines (Tekton)**  
- **OpenShift Serverless (Knative)**  

---

# 10. Troubleshooting

### 10.1 Operator ei käivitu

```bash
oc get pods -n openshift-operators
```

### 10.2 CR ei loo ressursse

```bash
oc describe <crd> <nimi>
```

### 10.3 CSV error

```bash
oc get csv -n openshift-operators
oc describe csv <nimi>
```

---

# 11. Best Practices

- Kasuta **stable** channel’it tootmises  
- Ära muuda CRD-sid käsitsi  
- Kasuta Namespaced installi, kui Operator on projektispetsiifiline  
- Kasuta Cluster installi, kui Operator teenindab kogu klastrit  
- Kasuta OperatorHub’i, mitte manuaalseid installiskripte  
- Kontrolli CSV staatust pärast installi  

---

# Kokkuvõte
OpenShift Operators automatiseerivad keerukate teenuste elutsükli haldamise Kubernetes klastris.  
OLM, CRD-d ja Subscription’id moodustavad võimsa mehhanismi, mis muudab rakenduste haldamise usaldusväärseks ja standardiseerituks.
```
