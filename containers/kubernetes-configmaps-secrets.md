# 📄 **Fail: `containers/kubernetes-configmaps-secrets.md`**

```markdown
# Kubernetes ConfigMaps & Secrets – Containers käsiraamat  
## (ConfigMaps, Secrets, envFrom, volumes, encryption, best practices)

## Ülevaade
ConfigMaps ja Secrets on Kubernetes’i konfiguratsioonihalduse kaks põhikomponenti.

- **ConfigMap** – konfiguratsioon ilma tundlike andmeteta  
- **Secret** – paroolid, võtmed, tokenid (base64 kodeeritud)  

Need võimaldavad hoida konfiguratsiooni konteineritest eraldi.

---

# 1. ConfigMap

ConfigMap salvestab konfiguratsiooni, mis EI sisalda paroole.

## 1.1 Loo ConfigMap YAML-iga

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  MODE: production
  TIMEOUT: "30"
```

## 1.2 Loo ConfigMap käsurealt

```bash
kubectl create configmap app-config --from-literal=MODE=production
```

---

# 2. ConfigMap kasutamine

## 2.1 envFrom (kõik võtmed keskkonnamuutujatena)

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

## 2.2 Üksik väärtus

```yaml
env:
  - name: MODE
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: MODE
```

## 2.3 Mount failina

```yaml
volumeMounts:
  - name: config
    mountPath: /etc/config

volumes:
  - name: config
    configMap:
      name: app-config
```

---

# 3. Secret

Secret on mõeldud tundlike andmete jaoks:

- paroolid  
- API võtmed  
- sertifikaadid  
- tokenid  

## 3.1 Loo Secret käsurealt

```bash
kubectl create secret generic db --from-literal=password=123
```

## 3.2 Loo Secret YAML-iga

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db
type: Opaque
data:
  password: MTIz   # base64("123")
```

---

# 4. Secret kasutamine

## 4.1 Keskkonnamuutujana

```yaml
env:
  - name: DB_PASS
    valueFrom:
      secretKeyRef:
        name: db
        key: password
```

## 4.2 Failina mountimine

```yaml
volumeMounts:
  - name: secret
    mountPath: /etc/secret

volumes:
  - name: secret
    secret:
      secretName: db
```

---

# 5. Secret tüübid

| Tüüp | Kirjeldus |
|------|-----------|
| **Opaque** | suvalised võtmed |
| **kubernetes.io/dockerconfigjson** | private registry |
| **tls** | TLS sert + privaatvõti |
| **bootstrap token** | kubeadm tokenid |

---

# 6. Base64 kodeerimine

Secret väärtused peavad olema base64 kodeeritud.

Kodeeri:

```bash
echo -n "123" | base64
```

Dekodeeri:

```bash
echo -n "MTIz" | base64 --decode
```

---

# 7. ConfigMap vs Secret

| Funktsioon | ConfigMap | Secret |
|-----------|-----------|--------|
| Tundlik info | ✖ | ✔ |
| Krüpteerimine | ✖ | võimalik (etcd encryption) |
| Base64 | ✖ | ✔ |
| Mount failina | ✔ | ✔ |
| envFrom | ✔ | ✔ |

---

# 8. Etcd Encryption (soovitatav)

Secrets salvestatakse etcd-s **base64**, mitte krüpteeritult.  
Krüpteerimiseks:

`/etc/kubernetes/encryption.yaml`:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <base64-key>
      - identity: {}
```

Käivita API server selle konfiguratsiooniga.

---

# 9. Rolling updates ConfigMap/Secret muutmisel

Deployment ei uuene automaatselt, kui ConfigMap/Secret muutub.

Lahendused:

- lisa annotation, mis muutub  
- kasuta checksum’i  
- kasuta reloaderit (Stakater Reloader)

Näide annotation:

```yaml
annotations:
  config-hash: "123abc"
```

---

# 10. Best Practices

- Kasuta **ConfigMap** konfiguratsiooni jaoks  
- Kasuta **Secret** tundlike andmete jaoks  
- Krüpteeri Secrets **etcd encryption** abil  
- Ära hoia paroole image’is  
- Ära pane Secrets versioonihaldusesse  
- Kasuta **RBAC** Secret’itele ligipääsu piiramiseks  
- Kasuta **checksum annotations** automaatseks restartiks  

---

# Kokkuvõte
ConfigMaps ja Secrets on Kubernetes’i konfiguratsioonihalduse vundament.  
ConfigMap hoiab mittetundlikku infot, Secret tundlikku infot — ja mõlemad võimaldavad konfiguratsiooni konteineritest eraldi hoida.
```
