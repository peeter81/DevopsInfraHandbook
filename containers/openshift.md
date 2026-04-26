# 📄 **Fail: `containers/openshift.md`**

```markdown
# OpenShift – Containers käsiraamat  
## (OKD, Operators, Routes, Projects, SCC, DeploymentConfigs)

## Ülevaade
OpenShift on Red Hat’i Kubernetes‑põhine platvorm, mis lisab:

- turvalisuse (SCC – Security Context Constraints)
- arendaja‑sõbraliku workflow
- sisseehitatud registry
- automaatse buildi (BuildConfig)
- DeploymentConfig ressursid
- OperatorHub
- Routes (Ingress alternatiiv)
- RBAC + Projects (namespaces)

OpenShift = Kubernetes + enterprise funktsioonid.

---

# 1. OpenShift CLI (oc)

Install:

```bash
curl -LO https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz
tar -xvf oc.tar.gz
mv oc /usr/local/bin/
```

Kontrolli:

```bash
oc version
```

Login:

```bash
oc login https://api.cluster.example:6443
```

---

# 2. Projects (namespaces)

Loo projekt:

```bash
oc new-project demo
```

Vaheta projekti:

```bash
oc project demo
```

---

# 3. DeploymentConfig (OpenShift spetsiifiline)

Näide:

```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: web
spec:
  replicas: 2
  selector:
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
oc apply -f web-dc.yaml
```

---

# 4. Routes (OpenShift Ingress)

Route eksponeerib teenuse välismaailmale.

Näide:

```bash
oc expose svc/web
```

Vaata:

```bash
oc get routes
```

Route URL:

```
http://web-demo.apps.cluster.example.com
```

---

# 5. BuildConfig (automaatne build)

OpenShift suudab image’eid ise ehitada.

Näide:

```bash
oc new-build https://github.com/user/app.git --name app
```

Vaata build’e:

```bash
oc get builds
```

---

# 6. ImageStreams

OpenShift kasutab ImageStream’e image versioonide haldamiseks.

Vaata:

```bash
oc get is
```

Triggerid:

- BuildConfig → ImageStream → DeploymentConfig

---

# 7. Security Context Constraints (SCC)

OpenShift on **rangem kui Kubernetes**.

Vaata SCC:

```bash
oc get scc
```

Levinud SCC-d:

| SCC | Kirjeldus |
|-----|-----------|
| restricted | vaikimisi, ei luba root’i |
| anyuid | lubab root kasutaja |
| privileged | täielik ligipääs |

Lisa kasutaja SCC-sse:

```bash
oc adm policy add-scc-to-user anyuid -z default
```

---

# 8. Operators ja OperatorHub

Operatorid haldavad keerukaid rakendusi:

- PostgreSQL  
- MongoDB  
- Kafka  
- ElasticSearch  
- Service Mesh  
- GitOps  

Install Operator:

```bash
oc apply -f operator.yaml
```

---

# 9. OpenShift Console

Graafiline UI:

- Projects  
- Pods  
- Builds  
- Routes  
- Operators  
- Metrics  
- Logs  

Ava:

```bash
oc whoami --show-console
```

---

# 10. OpenShift vs Kubernetes

| Funktsioon | Kubernetes | OpenShift |
|-----------|------------|-----------|
| Deployment | jah | jah |
| DeploymentConfig | ei | **jah** |
| Ingress | jah | jah |
| Route | ei | **jah** |
| SCC | piiratud | **väga range** |
| BuildConfig | ei | **jah** |
| ImageStream | ei | **jah** |
| Registry | manuaalne | **sisseehitatud** |
| Operators | osaliselt | **täielik tugi** |

---

# 11. Troubleshooting

### 11.1 Pod ei käivitu (SCC probleem)

```bash
oc describe pod <nimi>
```

### 11.2 Build ebaõnnestub

```bash
oc logs build/<nimi>
```

### 11.3 Route ei tööta

```bash
oc get routes
oc describe route <nimi>
```

---

# 12. Best Practices

- Kasuta **DeploymentConfig** kui vajad OpenShift build‑pipeline’i  
- Kasuta **Routes** HTTP/HTTPS eksponeerimiseks  
- Kasuta **SCC** turvalisuse kontrollimiseks  
- Kasuta **ImageStream** versioonihalduseks  
- Kasuta **Operators** keerukate teenuste haldamiseks  
- Ära jookse konteinerit root kasutajana (restricted SCC)  

---

# Kokkuvõte
OpenShift laiendab Kubernetes’t enterprise funktsioonidega: SCC, Routes, BuildConfig, ImageStreams ja OperatorHub.  
See muudab rakenduste haldamise, turvalisuse ja CI/CD töövood oluliselt lihtsamaks ja standardiseeritumaks.
```
