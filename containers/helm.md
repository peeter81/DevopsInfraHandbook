# 📄 **Fail: `containers/helm.md`**

```markdown
# Helm – Kubernetes käsiraamat  
## (Charts, releases, values, templating, repositories)

## Ülevaade
Helm on Kubernetes’i paketihaldur, mis võimaldab:

- paigaldada rakendusi ühe käsuga  
- hallata versioone (releases)  
- kasutada templatingut  
- hoida konfiguratsiooni values.yaml failides  
- uuendada ja rollbackida rakendusi  

Helm = “apt/yum” Kubernetes’i jaoks.

---

# 1. Helm install

## 1.1 Linux

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

## 1.2 Kontrolli

```bash
helm version
```

---

# 2. Helm repos

## 2.1 Lisa repo

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

## 2.2 Uuenda reposid

```bash
helm repo update
```

## 2.3 Otsi chart’e

```bash
helm search repo nginx
```

---

# 3. Chart install

Näide: paigalda NGINX

```bash
helm install mynginx bitnami/nginx
```

Vaata releases:

```bash
helm list
```

---

# 4. Chart uninstall

```bash
helm uninstall mynginx
```

---

# 5. Values.yaml

Kõik konfiguratsioonid käivad values.yaml kaudu.

Näide:

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.25"

service:
  type: LoadBalancer
  port: 80
```

Install koos values failiga:

```bash
helm install webapp ./chart -f values.yaml
```

Override käsurealt:

```bash
helm install webapp ./chart --set replicaCount=5
```

---

# 6. Helm upgrade

```bash
helm upgrade webapp ./chart -f values.yaml
```

Rollback:

```bash
helm rollback webapp 1
```

---

# 7. Helm chart struktuur

```text
mychart/
  Chart.yaml
  values.yaml
  templates/
    deployment.yaml
    service.yaml
    ingress.yaml
```

### Chart.yaml

```yaml
apiVersion: v2
name: mychart
version: 0.1.0
description: Example chart
```

---

# 8. Templating

Helm kasutab Go templatingut.

Näide:

```yaml
replicas: {{ .Values.replicaCount }}
```

Tingimus:

```yaml
{{- if .Values.ingress.enabled }}
# ingress config
{{- end }}
```

Loop:

```yaml
{{- range .Values.ports }}
- containerPort: {{ . }}
{{- end }}
```

---

# 9. Helm hooks

Hooks võimaldavad käivitada tegevusi enne/peale deploy’d.

Näide:

```yaml
annotations:
  "helm.sh/hook": pre-install
```

---

# 10. Helm dependency management

Chart võib sisaldada alamchart’e.

Lisa dependency:

```yaml
dependencies:
  - name: redis
    version: 17.0.0
    repository: "https://charts.bitnami.com/bitnami"
```

Lae alla:

```bash
helm dependency update
```

---

# 11. Helm + GitOps

Helm integreerub GitOps tööriistadega:

- ArgoCD  
- FluxCD  

GitOps kasutab Helm chart’e automaatseks deploy’ks.

---

# 12. Best Practices

- Hoia kõik konfiguratsioonid **values.yaml** failis  
- Ära pane paroole values.yaml sisse → kasuta **Secrets**  
- Kasuta **helm upgrade --install** CI/CD-s  
- Kasuta **helm lint** chartide kontrolliks  
- Versiooniuuendused tee **helm diff** abil  
- Kasuta **dependencies** keerukate stackide jaoks  

---

# Kokkuvõte
Helm on Kubernetes’i paketihaldur, mis muudab rakenduste paigaldamise, uuendamise ja haldamise lihtsaks ja standardiseerituks.  
Charts, values.yaml ja templating moodustavad võimsa tööriistakomplekti skaleeritavate süsteemide haldamiseks.
```
