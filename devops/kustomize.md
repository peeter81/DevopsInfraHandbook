# 📄 **Fail 4: `devops/kustomize.md` (PUHASTATUD)**

```markdown
# Kustomize – DevOps käsiraamat

## Ülevaade
Kustomize on Kubernetes’i natiivne tööriist, mis võimaldab:

- luua korduvkasutatavaid baaskonfiguratsioone
- rakendada keskkonnaspetsiifilisi muudatusi (dev / stage / prod)
- hallata pildi versioone, patche ja konfiguratsioone ilma templatinguta

Kustomize on sisse ehitatud `kubectl` käsusse:

```
kubectl apply -k <kaust>
kubectl kustomize <kaust>
```

---

# 1. `kustomization.yaml` põhistruktuur

```yaml
resources:
  - deployment.yaml
  - service.yaml

images:
  - name: nginx
    newTag: "1.27"

configMapGenerator:
  - name: app-config
    literals:
      - MODE=production

secretGenerator:
  - name: db-secret
    literals:
      - PASSWORD=supersecret

patchesStrategicMerge:
  - patch-replicas.yaml
```

---

# 2. Baas + overlay struktuur

```
app/
  base/
    deployment.yaml
    service.yaml
    kustomization.yaml

  overlays/
    dev/
      kustomization.yaml
    stage/
      kustomization.yaml
    prod/
      kustomization.yaml
```

---

# 3. Näide: base `kustomization.yaml`

```yaml
resources:
  - deployment.yaml
  - service.yaml

images:
  - name: myapp
    newTag: "1.0.0"
```

---

# 4. Näide: prod overlay

`overlays/prod/kustomization.yaml`:

```yaml
bases:
  - ../../base

patchesStrategicMerge:
  - replicas-prod.yaml

images:
  - name: myapp
    newTag: "1.0.5"
```

`replicas-prod.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 5
```

---

# 5. ConfigMapGenerator

```yaml
configMapGenerator:
  - name: app-config
    literals:
      - LOG_LEVEL=info
      - FEATURE_X=true
```

Kustomize genereerib automaatselt hash’i:

```
app-config-7d8f9c7c9b
```

See tagab, et ConfigMap’i muutus → podid restarditakse.

---

# 6. SecretGenerator

```yaml
secretGenerator:
  - name: db-secret
    literals:
      - USER=admin
      - PASSWORD=pass123
```

NB: **ära hoia paroole Git’is** → kasuta Sealed Secrets või External Secrets.

---

# 7. Piltide override

```yaml
images:
  - name: backend
    newName: registry.example.com/backend
    newTag: "2.3.1"
```

---

# 8. patchesStrategicMerge

Näide patchist, mis muudab ainult ühe välja:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  template:
    spec:
      containers:
        - name: backend
          resources:
            limits:
              cpu: "500m"
```

---

# 9. patchesJson6902 (täpsem patching)

```yaml
patchesJson6902:
  - target:
      group: apps
      version: v1
      kind: Deployment
      name: backend
    path: patch.json
```

`patch.json`:

```json
[
  {
    "op": "replace",
    "path": "/spec/replicas",
    "value": 4
  }
]
```

---

# 10. Kustomize arhitektuuri diagramm

```
        +---------------------------+
        |        Base YAML          |
        +-------------+-------------+
                      |
                      v
        +---------------------------+
        |         Overlays          |
        |   (dev / stage / prod)    |
        +-------------+-------------+
                      |
                      v
        +---------------------------+
        |     Kustomize Build       |
        |  (rendered manifests)     |
        +-------------+-------------+
                      |
                      v
        +---------------------------+
        |     kubectl apply         |
        +---------------------------+
```

---

# 11. Best Practices

- Kasuta **base + overlays** struktuuri
- Ära dubleeri YAML’e → kasuta patche
- Kasuta `configMapGenerator` ja `secretGenerator`
- Ära kasuta `latest` tag’i
- Kasuta GitOps tööriistu (ArgoCD / FluxCD)

---

# Kokkuvõte
Kustomize võimaldab hallata Kubernetes konfiguratsioone puhtalt, korduvkasutatavalt ja keskkonnaspetsiifiliselt.  
See on GitOps’i ja enterprise‑taseme K8s töövoogude lahutamatu osa.
```
