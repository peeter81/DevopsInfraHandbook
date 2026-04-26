# 📄 **Fail: `containers/minikube.md`**

```markdown
# Minikube – Kubernetes käsiraamat  
## (Local cluster, drivers, addons, ingress, loadbalancer, volumes)

## Ülevaade
Minikube on tööriist, mis võimaldab käivitada **lokaalse Kubernetes klastrit** ühes masinas.  
See sobib:

- arenduseks  
- testimiseks  
- õppimiseks  
- CI keskkondadeks  

Minikube toetab mitut virtualiseerimismootorit (drivers):

- docker  
- virtualbox  
- kvm2  
- hyperkit  
- none (bare‑metal)

---

# 1. Install

## Linux

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Kontrolli:

```bash
minikube version
```

---

# 2. Clusteri käivitamine

### Docker driver (soovitatav)

```bash
minikube start --driver=docker
```

### KVM2

```bash
minikube start --driver=kvm2
```

### Bare-metal (none)

```bash
minikube start --driver=none
```

---

# 3. Kubectl integreerimine

Minikube lisab kubeconfig’i automaatselt:

```bash
kubectl get nodes
```

---

# 4. Dashboard

Minikube sisaldab sisseehitatud dashboard’i:

```bash
minikube dashboard
```

---

# 5. Addons

Vaata addons’eid:

```bash
minikube addons list
```

Luba ingress:

```bash
minikube addons enable ingress
```

Luba metrics-server:

```bash
minikube addons enable metrics-server
```

---

# 6. LoadBalancer teenused

Minikube simuleerib LoadBalancer’it:

```bash
minikube tunnel
```

Teenuse IP muutub kättesaadavaks lokaalselt.

---

# 7. Image’ide kasutamine Minikube sees

### Variant A: Ehita image otse Minikube Dockeris

```bash
eval $(minikube docker-env)
docker build -t myapp .
```

### Variant B: Lae image Minikube sisse

```bash
minikube image load myapp:latest
```

---

# 8. Volumes

Minikube mount:

```bash
minikube mount /host/path:/container/path
```

---

# 9. Profiilid (mitu klastrit)

Loo uus profiil:

```bash
minikube start -p dev
```

Vaheta profiili:

```bash
minikube profile dev
```

---

# 10. Clusteri peatamine ja kustutamine

Peata:

```bash
minikube stop
```

Kustuta:

```bash
minikube delete
```

---

# 11. Troubleshooting

### 11.1 Driver probleemid

```bash
minikube start --driver=docker --force
```

### 11.2 Kubelet crash

```bash
minikube logs
```

### 11.3 DNS ei tööta

```bash
kubectl -n kube-system rollout restart deployment/coredns
```

---

# 12. Best Practices

- Kasuta **docker driver**’it (kõige stabiilsem)  
- Kasuta **addons** funktsionaalsuse laiendamiseks  
- Kasuta `minikube image load` lokaalse arenduse jaoks  
- Kasuta profiile mitme projekti jaoks  
- Ära kasuta Minikube’t tootmises  

---

# Kokkuvõte
Minikube on lihtne ja võimas viis lokaalse Kubernetes klastriga töötamiseks.  
See sobib suurepäraselt arenduseks, testimiseks ja õppimiseks, pakkudes täielikku K8s funktsionaalsust minimaalse ressursikuluga.
```
