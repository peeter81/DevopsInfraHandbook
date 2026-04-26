# 📄 **Fail: `containers/docker.md`**

```markdown
# Docker – Containers käsiraamat  
## (Images, containers, volumes, networks, Dockerfile, Compose)

## Ülevaade
Docker on konteinerite platvorm, mis võimaldab:

- käivitada rakendusi isoleeritult  
- pakendada rakendusi imagesse  
- hallata sõltuvusi  
- luua reprodutseeritavaid keskkondi  
- kasutada Compose’i multi‑container stackide jaoks  

Docker kasutab:
- **cgroups** (ressursipiirangud)
- **namespaces** (isolatsioon)
- **overlayfs** (filesystem layers)

---

# 1. Põhikäsud

## 1.1 Image’ide nimekiri

```bash
docker images
```

## 1.2 Containerite nimekiri

```bash
docker ps
docker ps -a
```

## 1.3 Containeri käivitamine

```bash
docker run -d --name web nginx
```

## 1.4 Containeri logid

```bash
docker logs web
```

## 1.5 Containeri shell

```bash
docker exec -it web bash
```

---

# 2. Image’ide haldus

## 2.1 Image tõmbamine

```bash
docker pull ubuntu:22.04
```

## 2.2 Image kustutamine

```bash
docker rmi nginx
```

## 2.3 Image ehitamine Dockerfile’ist

```bash
docker build -t myapp:1.0 .
```

---

# 3. Dockerfile

Näide:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

Ehita:

```bash
docker build -t myapp .
```

---

# 4. Volumes (andmete püsivus)

## 4.1 Loo volume

```bash
docker volume create data
```

## 4.2 Kasuta volume’it

```bash
docker run -d -v data:/var/lib/mysql mysql
```

## 4.3 Bind mount

```bash
docker run -v /host/path:/container/path nginx
```

---

# 5. Võrgud

## 5.1 Loo võrk

```bash
docker network create backend
```

## 5.2 Lisa container võrku

```bash
docker run --network backend nginx
```

## 5.3 Vaata võrke

```bash
docker network ls
```

---

# 6. Ressursipiirangud

## 6.1 CPU

```bash
docker run --cpus=1.5 nginx
```

## 6.2 RAM

```bash
docker run --memory=512m nginx
```

## 6.3 PID limit

```bash
docker run --pids-limit=100 nginx
```

---

# 7. Docker Compose

Fail: `docker-compose.yml`

Näide:

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
    volumes:
      - dbdata:/var/lib/mysql

volumes:
  dbdata:
```

Käivita:

```bash
docker compose up -d
```

Seiska:

```bash
docker compose down
```

---

# 8. Registry

## 8.1 Logi sisse

```bash
docker login
```

## 8.2 Push

```bash
docker push myrepo/myapp:1.0
```

## 8.3 Pull

```bash
docker pull myrepo/myapp:1.0
```

---

# 9. Troubleshooting

## 9.1 Container ei käivitu

```bash
docker logs <container>
```

## 9.2 Pordikonflikt

```bash
docker ps -a
ss -tulpn | grep 80
```

## 9.3 Disk täis (overlayfs)

```bash
docker system df
docker system prune -a
```

---

# 10. Best Practices

- Kasuta **Dockerfile multistage build**’e  
- Ära jookse konteinerit root kasutajana  
- Kasuta **volumes** andmete jaoks  
- Kasuta **docker compose** multi‑container stackide jaoks  
- Kasuta **ressursipiiranguid** (CPU, RAM)  
- Ära hoia kasutamata image’eid ja konteinerid  
- Kasuta **healthcheck** direktiivi Dockerfile’is  

---

# Kokkuvõte
Docker on konteinerite standardplatvorm, mis võimaldab rakendusi isoleerida, pakendada ja hallata.  
Dockerfile, Compose, volumes ja võrgud moodustavad paindliku ja võimsa tööriistakomplekti nii arenduseks kui tootmiseks.
```
