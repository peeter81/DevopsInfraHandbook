# 📄 **Fail: `containers/crio.md`**

```markdown
# CRI‑O – Containers käsiraamat  
## (Kubernetes CRI runtime, OCI images, CNI networking, configuration)

## Ülevaade
CRI‑O on Kubernetes’i jaoks loodud **kerge ja turvaline konteinerite runtime**, mis järgib:

- **CRI (Container Runtime Interface)** standardit  
- **OCI image** formaati  
- **CNI** võrgustiku pluginaid  

CRI‑O on alternatiiv containerd‑le ja Dockerile Kubernetes klastrites.

---

# 1. CRI‑O arhitektuur

CRI‑O koosneb:

- **crio** daemon  
- **runc** või **crun** runtime  
- **conmon** (container monitor)  
- **CNI** võrgustik  
- **OCI images** tugi  

Kubernetes suhtleb CRI‑O-ga läbi CRI API.

---

# 2. Install

### RHEL / Rocky / Alma

```bash
yum install cri-o
systemctl enable --now crio
```

### Ubuntu

```bash
apt install cri-o cri-o-runc
systemctl enable --now crio
```

Kontrolli:

```bash
crio --version
```

---

# 3. CRI‑O konfiguratsioon

Peamine fail:

```
/etc/crio/crio.conf
```

Loo vaikekonfiguratsioon:

```bash
crio config > /etc/crio/crio.conf
```

Olulised sektsioonid:

- `[crio.runtime]` – runtime valik (runc/crun)  
- `[crio.image]` – registrid, pull poliitikad  
- `[crio.network]` – CNI konfiguratsioon  

---

# 4. CRI‑O ja Kubernetes

Kubelet peab kasutama CRI‑O runtime’i:

```
/var/lib/kubelet/kubeadm-flags.env
```

Näide:

```
--container-runtime=remote
--container-runtime-endpoint=unix:///var/run/crio/crio.sock
```

Kontrolli ühendust:

```bash
crictl ps
crictl images
```

---

# 5. Images

CRI‑O kasutab OCI image formaati.

### Pull:

```bash
crictl pull nginx
```

### List:

```bash
crictl images
```

### Inspect:

```bash
crictl inspecti nginx
```

---

# 6. Containers

### Käivita container (madala taseme test):

```bash
crictl runp pod.json
crictl create podID container.json pod.json
crictl start containerID
```

### Peata:

```bash
crictl stop containerID
```

---

# 7. CNI networking

CRI‑O kasutab CNI pluginaid:

- flannel  
- calico  
- cilium  
- weave  

CNI konfiguratsioon:

```
/etc/cni/net.d/
```

Vaata:

```bash
crictl inspectp <pod>
```

---

# 8. Runtime valik (runc vs crun)

### runc
- vaikimisi  
- stabiilne  
- laialt kasutatud  

### crun
- kiirem  
- väiksem footprint  
- parem cgroups v2 tugi  

Vaheta runtime:

```
[crio.runtime]
default_runtime = "crun"
```

Restart:

```bash
systemctl restart crio
```

---

# 9. Storage

CRI‑O kasutab overlayfs snapshotterit.

Vaata storage katalooge:

```
/var/lib/containers/storage/
```

Puhasta:

```bash
crictl rmi --prune
```

---

# 10. Troubleshooting

### 10.1 CRI‑O ei käivitu

```bash
journalctl -u crio
```

### 10.2 CNI probleemid

```bash
crictl inspectp <pod>
```

### 10.3 Image pull error

Kontrolli registries.conf:

```
/etc/containers/registries.conf
```

---

# 11. Best Practices

- Kasuta CRI‑O Kubernetes klastrites (kerge ja turvaline)  
- Kasuta **crun** parema jõudluse jaoks  
- Hoia CNI konfiguratsioon korras  
- Kasuta `crictl` diagnostikaks  
- Ära sega CRI‑O ja Dockerit samas klastris  
- Kasuta ainult OCI‑ühilduvaid image’eid  

---

# Kokkuvõte
CRI‑O on Kubernetes’i jaoks optimeeritud konteinerite runtime, mis on kerge, turvaline ja CRI‑standardiga täielikult ühilduv.  
See on ideaalne valik Kubernetes klastritele, kus stabiilsus ja jõudlus on kriitilised.
```
