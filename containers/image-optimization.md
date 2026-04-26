# 📄 **Fail: `containers/image-optimization.md`**

```markdown
# Container Image Optimization – Containers käsiraamat  
## (Small images, multi‑stage builds, caching, layers, security)

## Ülevaade
Optimeeritud konteinerimage’id on:

- väiksemad  
- kiiremini allalaetavad  
- odavamad salvestada  
- turvalisemad (vähem pakette → väiksem rünnepind)  
- parema cache‑kasutusega  

See fail katab kõik võtmetehnikad, mida kasutatakse professionaalsetes DevOps/Kubernetes keskkondades.

---

# 1. Väikeste base image’ide kasutamine

### Soovitatavad:
- **alpine** (väike, kiire, musl libc)
- **distroless** (ainult runtime, pole shelli)
- **ubi‑micro** (RHEL‑põhine, enterprise)
- **scratch** (täiesti tühi)

### Näited:

```dockerfile
FROM alpine:3.19
```

```dockerfile
FROM gcr.io/distroless/base
```

---

# 2. Multi‑stage builds

Kõige olulisem optimeerimistehnika.

### Näide: Go rakendus

```dockerfile
FROM golang:1.22 AS build
WORKDIR /app
COPY . .
RUN go build -o app

FROM alpine:3.19
COPY --from=build /app/app /usr/local/bin/app
CMD ["app"]
```

Tulemus:
- build image ~1GB  
- final image ~10–20MB  

---

# 3. Layerite optimeerimine

### Reegel: **iga RUN, COPY, ADD → uus layer**

### Soovitus:
- kombineeri käske üheks RUN käsuks
- puhasta cache samas RUN käsus

Näide halb:

```dockerfile
RUN apt update
RUN apt install -y curl
RUN apt clean
```

Näide hea:

```dockerfile
RUN apt update && apt install -y curl && apt clean
```

---

# 4. Cache kasutamine

### COPY järjestus on kriitiline

Halb:

```dockerfile
COPY . .
RUN npm install
```

Hea:

```dockerfile
COPY package*.json .
RUN npm install
COPY . .
```

Tulemus:
- `npm install` cache’tub  
- rebuild on palju kiirem  

---

# 5. Failide välistamine (.dockerignore)

Failid, mida EI tohiks image’i kopeerida:

```
.git
node_modules
*.log
tmp/
.env
```

Fail: `.dockerignore`

---

# 6. Distroless images (turvalisus)

Distroless image’id sisaldavad ainult:

- runtime  
- sinu rakendus  

Ei sisalda:
- shelli  
- package managerit  
- debug tööriistu  

Näide:

```dockerfile
FROM gcr.io/distroless/nodejs:18
COPY app.js .
CMD ["app.js"]
```

Turvalisus paraneb märgatavalt.

---

# 7. Scratch images (maksimaalne minimalism)

Sobib:
- Go  
- Rust  
- C/C++ statically linked binary

Näide:

```dockerfile
FROM scratch
COPY app /
CMD ["/app"]
```

Image suurus: **kõigest mõni MB**

---

# 8. Security scanning

Kasuta:
- **Trivy**
- **Grype**
- **Docker Scout**
- **Anchore**

Näide:

```bash
trivy image myapp:latest
```

---

# 9. Metadata lisamine (labels)

```dockerfile
LABEL maintainer="Peeter"
LABEL version="1.0"
LABEL org.opencontainers.image.source="https://github.com/project"
```

---

# 10. Healthcheck

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
```

---

# 11. Best Practices

- Kasuta **multi‑stage builds**  
- Kasuta **alpine**, **distroless** või **scratch**  
- Kasuta `.dockerignore`  
- Ära hoia build‑tööriistu final image’is  
- Kasuta **healthcheck**  
- Kasuta **Trivy** või **Grype** turvakontrolliks  
- Kasuta **layer caching** optimeerimiseks  
- Ära lisa image’i tarbetuid faile  

---

# Kokkuvõte
Container image optimeerimine vähendab suurust, parandab turvalisust ja kiirendab CI/CD pipeline’e.  
Multi‑stage builds, väiksed base image’id ja caching on professionaalse konteineritöövoo alused.
```

