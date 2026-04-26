# 📄 **Fail: `containers/kubernetes-helm.md`**

```markdown
# Kubernetes Helm – Containers käsiraamat  
## (Helm charts, releases, values.yaml, templating, repositories, upgrades)

## Ülevaade
Helm on Kubernetes’i paketihaldur — nagu apt/yum, aga klastrile.  
Helm võimaldab:

- rakenduste paketiseerimist (charts)  
- versioonihaldust (releases)  
- konfiguratsiooni haldamist (values.yaml)  
- lihtsat deploy’d ja upgrade’i  
- templatingut  

Helm on standard tootmises.

---

# 1. Helm install

## 1.1 Install Helm CLI

Linux:

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

Windows (Chocolatey):

```powershell
choco install kubernetes-helm
```

---

# 2. Helm chart struktuur

```
mychart/
  Chart.yaml
  values.yaml
  templates/
  charts/
```

### Chart.yaml

```yaml
apiVersion: v2
name: mychart
version: 0.1.0
```

### values.yaml

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

---

# 3. Helm käsud

## 3.1 Install

```bash
helm install myapp ./mychart
```

## 3.2 Upgrade

```bash
helm upgrade myapp ./mychart
```

## 3.3 Uninstall

```bash
helm uninstall myapp
```

## 3.4 Vaata releases

```bash
helm list
```

---

# 4. Helm values

Override väärtusi:

```bash
helm install myapp ./mychart -f custom-values.yaml
```

Või üksik väärtus:

```bash
helm install myapp ./mychart --set replicaCount=5
```

---

# 5. Helm repos

Lisa repo:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Otsi:

```bash
helm search repo nginx
```

Installi:

```bash
helm install nginx bitnami/nginx
```

---

# 6. Helm templating

Renderda chart ilma deploy’ta:

```bash
helm template myapp ./mychart
```

Kasulik debugimiseks.

---

# 7. Helm upgrade strateegiad

## 7.1 Rolling upgrade

Vaikimisi.

## 7.2 Atomic upgrade

Kui midagi läheb valesti → rollback automaatselt.

```bash
helm upgrade myapp ./mychart --atomic
```

## 7.3 Dry-run

```bash
helm upgrade myapp ./mychart --dry-run
```

---

# 8. Helm rollback

```bash
helm rollback myapp 1
```

---

# 9. Helm hooks

Hooks võimaldavad skripte käivitada:

- enne installi  
- pärast installi  
- enne upgrade’i  
- pärast upgrade’i  

Näide:

```yaml
annotations:
  "helm.sh/hook": pre-install
```

---

# 10. Best Practices

- Kasuta Helm’i kõigi keerukamate rakenduste jaoks  
- Hoia values.yaml puhas ja loetav  
- Kasuta `--atomic` tootmises  
- Kasuta `helm template` debugimiseks  
- Ära pane paroole values.yaml faili (kasuta Secrets)  
- Versioonihalda charts’e Git’is  

---

# Kokkuvõte
Helm on Kubernetes’i paketihaldur, mis muudab rakenduste deploy, upgrade’i ja konfiguratsiooni haldamise lihtsaks ja korduvkasutatavaks.
```

