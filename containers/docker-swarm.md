# 📄 **Fail: `containers/docker-swarm.md`**

```markdown
# Docker Swarm – Containers käsiraamat  
## (Swarm mode, services, stacks, overlay networks, scaling)

## Ülevaade
Docker Swarm on Dockeri sisseehitatud orkestreerimisplatvorm, mis võimaldab:

- klastrit (manager + worker nodes)
- teenuste (services) haldust
- automaatset skaleerimist
- rolling updates
- overlay võrgustikke
- stacks (Compose failid tootmises)

Swarm on lihtsam kui Kubernetes, kuid vähem funktsionaalne.

---

# 1. Swarm cluster

## 1.1 Swarm mode aktiveerimine

```bash
docker swarm init
```

Kui masinal on mitu IP‑d:

```bash
docker swarm init --advertise-addr 192.168.1.10
```

## 1.2 Worker node liitmine

Manager annab käsu:

```
docker swarm join --token <token> <manager-ip>:2377
```

Vaata tokenit uuesti:

```bash
docker swarm join-token worker
```

---

# 2. Nodes

Vaata node’e:

```bash
docker node ls
```

Node detailid:

```bash
docker node inspect <node>
```

---

# 3. Services (Swarm deployment unit)

## 3.1 Loo teenus

```bash
docker service create --name web -p 80:80 nginx
```

## 3.2 Vaata teenuseid

```bash
docker service ls
docker service ps web
```

## 3.3 Uuenda teenust

```bash
docker service update --image nginx:1.25 web
```

## 3.4 Skaalumine

```bash
docker service scale web=5
```

---

# 4. Stacks (Compose tootmises)

Swarm kasutab `docker-compose.yml` faile, kuid käivitatakse stack’ina.

Näide `stack.yml`:

```yaml
version: "3.9"
services:
  web:
    image: nginx
    ports:
      - "80:80"
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
```

Käivita stack:

```bash
docker stack deploy -c stack.yml mystack
```

Vaata:

```bash
docker stack ls
docker stack services mystack
docker stack ps mystack
```

Kustuta:

```bash
docker stack rm mystack
```

---

# 5. Overlay networks

Swarm loob klastris overlay‑võrgud.

Loo võrk:

```bash
docker network create -d overlay backend
```

Kasuta teenuses:

```yaml
networks:
  - backend
```

---

# 6. Secrets

Loo secret:

```bash
echo "mypassword" | docker secret create db_pass -
```

Kasuta teenuses:

```yaml
secrets:
  - db_pass
```

---

# 7. Rolling updates

Swarm teeb rolling updates automaatselt.

Näide:

```bash
docker service update --image myapp:2.0 --update-parallelism 1 --update-delay 10s web
```

---

# 8. Healthchecks

Dockerfile:

```dockerfile
HEALTHCHECK CMD curl -f http://localhost/ || exit 1
```

Swarm kasutab seda teenuse tervise jälgimiseks.

---

# 9. High availability

Swarm managerid:

- soovitatav 3 või 5 manager node’i  
- Raft konsensus  
- automaatne failover  

Vaheta manager:

```bash
docker node promote <node>
```

---

# 10. Troubleshooting

### 10.1 Node down

```bash
docker node ls
docker node rm <node>
```

### 10.2 Service ei käivitu

```bash
docker service ps web
docker service logs web
```

### 10.3 Overlay network ei tööta

Kontrolli:

```bash
docker network inspect backend
```

---

# 11. Best Practices

- Kasuta **3+ manager node’i**  
- Kasuta stacks, mitte üksikuid teenuseid  
- Kasuta overlay‑võrke teenuste vahel  
- Kasuta secrets paroolide jaoks  
- Kasuta rolling updates tootmises  
- Ära kasuta Swarm’i väga suurtes klastrites (Kubernetes sobib paremini)  

---

# Kokkuvõte
Docker Swarm on lihtne ja kiire orkestreerimisplatvorm, mis sobib väiksematele ja keskmise suurusega klastritele.  
Services, stacks, overlay networks ja rolling updates moodustavad Swarm’i põhifunktsionaalsuse.
```
