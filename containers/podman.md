# 📄 **Fail: `containers/podman.md`**

```markdown
# Podman – Containers käsiraamat  
## (Rootless containers, images, pods, Docker compatibility, systemd integration)

## Ülevaade
Podman on konteinerite platvorm, mis on täielikult **daemonless** ja toetab **rootless** konteinerite käivitamist.  
See on 100% CLI‑ühilduv Dockeriga:

- `docker run` → `podman run`
- `docker build` → `podman build`
- `docker ps` → `podman ps`

Podman kasutab:
- **cgroups v2**
- **user namespaces**
- **conmon** (container monitor)
- **Buildah** image ehitamiseks
- **Skopeo** image halduseks

---

# 1. Podman vs Docker

| Funktsioon | Docker | Podman |
|-----------|--------|--------|
| Daemon | jah (`dockerd`) | **ei** |
| Rootless | piiratud | **täielik tugi** |
| CLI ühilduvus | jah | jah |
| Pods | ei | **jah** (nagu Kubernetes) |
| Systemd integratsioon | manuaalne | **automaatne** |
| Turvalisus | root daemon | **daemonless, rootless** |

---

# 2. Põhikäsud

## 2.1 Image’ide nimekiri

```bash
podman images
```

## 2.2 Containerite nimekiri

```bash
podman ps
podman ps -a
```

## 2.3 Containeri käivitamine

```bash
podman run -d --name web nginx
```

## 2.4 Shell konteineris

```bash
podman exec -it web bash
```

---

# 3. Rootless containers

Podman võimaldab konteinerit käivitada ilma root‑õigusteta:

```bash
podman run -it alpine sh
```

Rootless konteinerid kasutavad:
- user namespaces
- subuid/subgid mappingut

Kontrolli:

```bash
cat /etc/subuid
cat /etc/subgid
```

---

# 4. Podman pods

Pod on konteinerite grupp (nagu Kubernetes).

## 4.1 Loo pod

```bash
podman pod create --name mypod -p 8080:80
```

## 4.2 Lisa konteiner pod’i

```bash
podman run -d --pod mypod nginx
```

## 4.3 Vaata pod’e

```bash
podman pod ps
```

---

# 5. Dockerfile ja build

Podman kasutab Buildah’t:

```bash
podman build -t myapp .
```

---

# 6. Volumes

```bash
podman volume create data
podman run -v data:/var/lib/mysql mysql
```

Bind mount:

```bash
podman run -v /host/path:/container/path nginx
```

---

# 7. Võrgud

```bash
podman network create backend
podman run --network backend nginx
```

---

# 8. Systemd integratsioon (suur eelis Dockeriga võrreldes)

Podman saab konteineri automaatselt systemd teenuseks genereerida.

## 8.1 Loo systemd unit

```bash
podman generate systemd --name web --files
```

See loob:

```
container-web.service
```

Paigalda:

```bash
mv container-web.service /etc/systemd/system/
systemctl enable --now container-web
```

---

# 9. Podman Compose

Docker Compose alternatiiv:

```bash
podman compose up -d
podman compose down
```

Toetab `docker-compose.yml` formaati.

---

# 10. Registries

Konfiguratsioon:

```
/etc/containers/registries.conf
```

Pull:

```bash
podman pull docker.io/library/nginx
```

Push:

```bash
podman push myapp:1.0 quay.io/user/myapp
```

---

# 11. Troubleshooting

## 11.1 Rootless networking probleemid

```bash
podman network inspect
```

## 11.2 Storage täis

```bash
podman system df
podman system prune -a
```

## 11.3 SELinux probleemid

```bash
setenforce 0
# testimiseks
```

---

# 12. Best Practices

- Kasuta **rootless** konteinerid turvalisuse jaoks  
- Kasuta **pods** Kubernetes‑sarnaste arhitektuuride jaoks  
- Kasuta **systemd integratsiooni** tootmises  
- Kasuta **podman compose** multi‑container stackide jaoks  
- Ära kasuta Docker daemonit, kui Podman on saadaval  
- Kasuta registries.conf failis usaldusväärseid registriesid  

---

# Kokkuvõte
Podman on turvaline, daemonless ja Docker‑ühilduv konteineriplatvorm, mis sobib eriti hästi serveritele, DevOps‑töövoogudele ja Kubernetes‑stiilis arhitektuuridele.  
Rootless töö, pods ja systemd integratsioon teevad sellest tugeva alternatiivi Dockerile.
```
