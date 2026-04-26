# 📄 **Fail: `containers/containerd.md`**

```markdown
# containerd – Containers käsiraamat  
## (CRI, runtimes, namespaces, images, containers, nerdctl)

## Ülevaade
containerd on madala taseme konteinerite runtime, mida kasutavad:

- Docker (kasutab containerd‑d taustal)
- Kubernetes (CRI plugin: cri‑containerd)
- Nerdctl (Docker‑ühilduv CLI containerd jaoks)

containerd on:
- kiire  
- stabiilne  
- daemonless (üks teenus, mitte Docker‑daemon)  
- CRI‑ühilduv  

---

# 1. containerd teenus

Kontrolli olekut:

```bash
systemctl status containerd
```

Taaskäivita:

```bash
systemctl restart containerd
```

Konfiguratsioon:

```
/etc/containerd/config.toml
```

Loo uus config:

```bash
containerd config default > /etc/containerd/config.toml
```

---

# 2. containerd namespaces

containerd kasutab **namespaces** konteinerite eraldamiseks.

Vaata namespaces:

```bash
ctr ns list
```

Vaheta namespace:

```bash
ctr -n k8s.io containers list
```

Levinud namespaces:
- `default`  
- `k8s.io` (Kubernetes)  
- `moby` (Docker)  

---

# 3. ctr – containerd CLI

ctr on madala taseme CLI (keerulisem kui Docker CLI).

## 3.1 Image pull

```bash
ctr images pull docker.io/library/nginx:latest
```

## 3.2 Image list

```bash
ctr images list
```

## 3.3 Container run

```bash
ctr run -d --name web docker.io/library/nginx:latest
```

## 3.4 Container stop

```bash
ctr tasks kill web
```

---

# 4. nerdctl – Docker‑ühilduv CLI containerd jaoks

Nerdctl teeb containerd kasutamise sama lihtsaks kui Docker.

Install:

```bash
apt install nerdctl
```

või:

```bash
yum install nerdctl
```

### Docker‑stiilis käsud:

```bash
nerdctl run -d --name web nginx
nerdctl ps
nerdctl logs web
nerdctl exec -it web bash
```

### Build (BuildKit)

```bash
nerdctl build -t myapp .
```

---

# 5. containerd ja Kubernetes

Kubernetes kasutab containerd‑d CRI kaudu.

Kontrolli CRI olekut:

```bash
crictl ps
crictl images
```

containerd CRI plugin töötab namespace’s:

```
k8s.io
```

---

# 6. containerd storage

Vaata storage katalooge:

```
/var/lib/containerd/
/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/
```

Puhasta:

```bash
nerdctl system prune
```

---

# 7. containerd runtimes

containerd toetab mitut runtime’i:

| Runtime | Kirjeldus |
|---------|-----------|
| runc | vaikimisi, OCI standard |
| crun | kiirem, cgroups v2 tugi |
| gVisor | sandboxed, turvaline |
| Kata Containers | lightweight VM |

Vaheta runtime:

```
/etc/containerd/config.toml
```

Näide:

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
runtime_type = "io.containerd.runc.v2"
```

---

# 8. containerd + CNI networking

Kubernetes kasutab CNI pluginaid:

- flannel  
- calico  
- cilium  
- weave  

CNI kataloog:

```
/etc/cni/net.d/
```

---

# 9. Troubleshooting

## 9.1 containerd ei käivitu

```bash
journalctl -u containerd
```

## 9.2 CRI probleemid

```bash
crictl info
```

## 9.3 Image pull error

```bash
ctr images pull --hosts-dir /etc/containerd/certs.d
```

---

# 10. Best Practices

- Kasuta **nerdctl** kui soovid Docker‑stiilis CLI‑t  
- Kasuta **crun** parema jõudluse jaoks  
- Kasuta **gVisor** sandboxinguks  
- Ära muuda containerd config’i ilma restartita  
- Kubernetes keskkonnas kasuta alati `crictl` diagnostikaks  
- Hoia CNI konfiguratsioon korras  

---

# Kokkuvõte
containerd on kiire, stabiilne ja Kubernetes‑sõbralik konteinerite runtime.  
Nerdctl muudab selle kasutamise sama lihtsaks kui Docker, samas kui ctr pakub madala taseme kontrolli.  
containerd on tänapäeva konteineriplatvormide vaikimisi standard.
```
