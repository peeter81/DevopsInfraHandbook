# 📄 **Fail: `containers/kind.md`**

```markdown
# kind – Kubernetes in Docker  
## (Local clusters, multi‑node setups, testing, CI integration)

## Ülevaade
**kind** (Kubernetes IN Docker) on tööriist, mis võimaldab käivitada **Kubernetes klastreid Docker konteinerites**.  
See on ideaalne:

- arenduseks  
- testimiseks  
- CI/CD pipeline’ides  
- kiireks eksperimenteerimiseks  
- mitme‑node topoloogiate loomiseks  

kind ei vaja VM‑e — kõik töötab Docker konteinerites.

---

# 1. Install

## 1.1 Linux / macOS / Windows

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
mv ./kind /usr/local/bin/kind
```

Kontrolli:

```bash
kind version
```

---

# 2. Loo Kubernetes cluster

Lihtne ühe‑node cluster:

```bash
kind create cluster
```

Kustuta:

```bash
kind delete cluster
```

---

# 3. Multi‑node cluster

Loo config fail:

`kind-config.yaml`

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Käivita:

```bash
kind create cluster --config kind-config.yaml
```

---

# 4. Kubeconfig

kind lisab kubeconfig’i automaatselt:

```bash
kubectl cluster-info --context kind-kind
```

---

# 5. Load images into kind

Kui ehitad lokaalse image’i ja tahad seda kind klastrisse:

```bash
kind load docker-image myapp:latest
```

---

# 6. Storage

kind kasutab Docker volumesid.

Vaata:

```bash
docker volume ls
```

---

# 7. Ingress tugi

Installi NGINX ingress:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

---

# 8. kind + Helm

Helm töötab kind klastris ilma probleemideta:

```bash
helm install web bitnami/nginx
```

---

# 9. kind CI/CD kasutus

kind on populaarne:

- GitHub Actions  
- GitLab CI  
- Jenkins  
- Azure DevOps  

Näide GitHub Actions:

```yaml
- name: Create kind cluster
  uses: helm/kind-action@v1.8.0
```

---

# 10. kind networking

kind loob Docker network’i:

```bash
docker network ls | grep kind
```

Node’d suhtlevad selle kaudu.

---

# 11. kind cluster reset

Kustuta ja loo uuesti:

```bash
kind delete cluster
kind create cluster
```

---

# 12. Best Practices

- Kasuta kind’i lokaalseks arenduseks  
- Kasuta multi‑node config’e testimiseks  
- Kasuta `kind load docker-image` lokaalse image testimiseks  
- Kasuta ingress‑nginx routinguks  
- Kasuta CI/CD pipeline’ides kiireks K8s testimiseks  
- Ära kasuta kind’i tootmises  

---

# Kokkuvõte
kind on kiire ja kerge viis Kubernetes klastrite loomiseks Docker konteinerites.  
See sobib suurepäraselt arenduseks, testimiseks ja CI/CD töövoogudeks, pakkudes täielikku K8s funktsionaalsust ilma virtuaalmasinate või pilveteenusteta.
```
