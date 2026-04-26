# 📄 **Fail: `cloud/docker.md`**

```markdown
# Docker – Containers käsiraamat  
## (Images, Containers, Volumes, Networks, Dockerfile, Compose)

## Ülevaade
Docker on konteinerite platvorm, mis võimaldab rakendusi pakendada, käivitada ja jagada isoleeritud keskkondades.  
See on DevOps-i ja Kubernetes’e vundament.

---

# 1. Docker põhimõisted

### Image
- read-only mall, millest konteinerid luuakse  
- sisaldab rakendust + sõltuvusi  

### Container
- image’i käivitatud instants  
- isoleeritud protsess  

### Registry
- koht, kus image’id asuvad (Docker Hub, GitHub Container Registry, ECR, ACR)

---

# 2. Docker käsud

## 2.1 Image’i tõmbamine

```bash
docker pull nginx
```

## 2.2 Konteineri käivitamine

```bash
docker run -d -p 80:80 nginx
```

## 2.3 Konteinerite vaatamine

```bash
docker ps
docker ps -a
```

## 2.4 Konteineri peatamine ja kustutamine

```bash
docker stop web
docker rm web
```

## 2.5 Image’i kustutamine

```bash
docker rmi nginx
```

---

# 3. Dockerfile

Näide:

```dockerfile
FROM ubuntu:22.04

RUN apt update && apt install -y nginx

COPY index.html /var/www/html/

CMD ["nginx", "-g", "daemon off;"]
```

Build:

```bash
docker build -t mynginx .
```

Run:

```bash
docker run -p 8080:80 mynginx
```

---

# 4. Volumes

Volumes = püsiv storage konteineritele.

Loo volume:

```bash
docker volume create data
```

Kasuta:

```bash
docker run -v data:/var/lib/mysql mysql
```

---

# 5. Networks

Loo võrk:

```bash
docker network create appnet
```

Käivita konteiner võrgus:

```bash
docker run --network appnet nginx
```

---

# 6. Docker Compose

Compose võimaldab mitut konteinerit koos käivitada.

`docker-compose.yml`:

```yaml
version: "3.9"
services:
  web:
    image: nginx
    ports:
      - "80:80"
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
```

Käivita:

```bash
docker compose up -d
```

Peata:

```bash
docker compose down
```

---

# 7. Docker registrid

### Docker Hub

```bash
docker login
docker push myimage
```

### AWS ECR

```bash
aws ecr get-login-password | docker login --username AWS --password-stdin <repo>
```

### Azure ACR

```bash
az acr login --name myregistry
```

---

# 8. Docker vs Kubernetes

| Funktsioon | Docker | Kubernetes |
|-----------|--------|------------|
| Konteinerite käivitamine | ✔ | ✔ |
| Orkestreerimine | ✖ | ✔ |
| Load balancing | ✖ | ✔ |
| Self-healing | ✖ | ✔ |
| Skaalumine | ✖ | ✔ |

Docker = konteinerid  
Kubernetes = konteinerite orkestreerimine

---

# 9. Best Practices

- Kasuta **väikseid base image’e** (alpine)  
- Ära pane paroole image’i  
- Kasuta `.dockerignore`  
- Kasuta **multi-stage build**  
- Kasuta **volumes** püsiva storage jaoks  
- Ära jookse root kasutajana konteineris  
- Kasuta **healthcheck**  

---

# Kokkuvõte
Docker on konteinerite standard, mis võimaldab rakendusi kiiresti ja isoleeritult käivitada.  
Dockerfile + Compose + Registry = täielik konteinerite töövoog.
```
