# 📄 **Fail: `containers/kubernetes-security.md`**

```markdown
# Kubernetes Security – Containers käsiraamat  
## (RBAC, ServiceAccounts, NetworkPolicies, PodSecurity, Secrets, Admission Controllers)

## Ülevaade
Kubernetes turvalisus koosneb mitmest kihist:

- autentimine & autoriseerimine  
- namespace isolatsioon  
- võrgupoliitikad  
- podi õigused ja piirangud  
- secrets & config  
- admission controllers  
- audit logid  

See fail katab kõige olulisemad turvamehhanismid tootmiskeskkonnas.

---

# 1. RBAC (Role-Based Access Control)

RBAC kontrollib, mida kasutajad ja teenused teha tohivad.

## 1.1 Role (namespace-scope)

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

## 1.2 RoleBinding

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

## 1.3 ClusterRole (cluster-scope)

```yaml
kind: ClusterRole
rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list"]
```

---

# 2. ServiceAccounts

Podid kasutavad ServiceAccount’e API‑ga suhtlemiseks.

Loo:

```bash
kubectl create sa app
```

Kasuta podis:

```yaml
serviceAccountName: app
```

---

# 3. NetworkPolicies

NetworkPolicy piirab liiklust podide vahel.

Näide: keelab kõik ühendused:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Näide: lubab ainult frontend → backend:

```yaml
ingress:
  - from:
      - podSelector:
          matchLabels:
            app: frontend
```

---

# 4. Pod Security (Pod Security Standards)

Kubernetes pakub kolme taset:

| Tase | Kirjeldus |
|------|-----------|
| **Privileged** | kõik lubatud, mitte turvaline |
| **Baseline** | enamik ohtlikke funktsioone keelatud |
| **Restricted** | kõige rangem, tootmises soovitatav |

Namespace’i märgistamine:

```bash
kubectl label ns dev pod-security.kubernetes.io/enforce=restricted
```

---

# 5. SecurityContext

## 5.1 Ära jookse root kasutajana

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

## 5.2 Failisüsteem read-only

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

## 5.3 Capabilities

Eemalda kõik:

```yaml
capabilities:
  drop: ["ALL"]
```

Lisa ainult vajalikud:

```yaml
add: ["NET_BIND_SERVICE"]
```

---

# 6. Secrets

Ära hoia paroole image’is.

Loo secret:

```bash
kubectl create secret generic db --from-literal=password=123
```

Kasuta:

```yaml
env:
  - name: DB_PASS
    valueFrom:
      secretKeyRef:
        name: db
        key: password
```

---

# 7. Admission Controllers

Admission controllerid valideerivad ja muudavad request’e.

Olulised:

- **NamespaceLifecycle**  
- **LimitRanger**  
- **ServiceAccount**  
- **NodeRestriction**  
- **PodSecurity**  
- **MutatingAdmissionWebhook**  
- **ValidatingAdmissionWebhook**  

Need võimaldavad:

- poliitikate jõustamist  
- automaatset injection’it (nt sidecar)  
- konfiguratsiooni valideerimist  

---

# 8. Image Security

- kasuta privaatseid registriesid  
- kasuta image pull secrets  
- kasuta allkirjastamist (cosign)  
- skanni image’eid (Trivy, Grype)  

Näide pull secret:

```bash
kubectl create secret docker-registry regcred \
  --docker-username=user \
  --docker-password=pass
```

---

# 9. API Server Security

- kasuta HTTPS  
- kasuta RBAC  
- kasuta audit logisid  
- piira API serveri ligipääsu firewalliga  

Audit logid:

```
/var/log/kubernetes/audit.log
```

---

# 10. Node Security

- ära jookse konteinerit root node’is  
- kasuta taints/tolerations  
- kasuta node isolation’i  
- uuenda containerd/CRI‑O regulaarselt  

---

# 11. Best Practices

- Kasuta **restricted** Pod Security taset  
- Kasuta **RBAC** minimaalsete õigustega  
- Kasuta **NetworkPolicies** vaikimisi deny-all  
- Ära jookse root kasutajana  
- Kasuta **readOnlyRootFilesystem**  
- Kasuta **Secrets**, mitte ConfigMap’e paroolide jaoks  
- Skanni image’eid regulaarselt  
- Kasuta privaatseid registriesid  
- Kasuta audit logisid  
- Kasuta admission webhooke poliitikate jõustamiseks  

---

# Kokkuvõte
Kubernetes turvalisus põhineb RBAC‑il, NetworkPolicies’il, PodSecurity standarditel, secrets haldusel ja admission controlleritel.  
Need mehhanismid tagavad, et töökoormused on isoleeritud, kontrollitud ja kaitstud.
```
