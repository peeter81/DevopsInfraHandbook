# 📄 **Fail: `containers/container-logging.md`**

```markdown
# Container Logging – Containers käsiraamat  
## (stdout/stderr, log drivers, log rotation, Kubernetes logging, EFK/ELK)

## Ülevaade
Konteinerite logimine on kriitiline:

- veaotsing  
- auditeerimine  
- jõudluse jälgimine  
- turvalisus  
- tootmiskeskkonna analüütika  

Konteinerid on lühiealised → logid peavad olema tsentraliseeritud ja püsivad.

---

# 1. Docker Logging

Docker konteinerid logivad vaikimisi:

```
stdout
stderr
```

Vaata logisid:

```bash
docker logs web
```

---

# 2. Docker log drivers

Vaata driverit:

```bash
docker info | grep "Logging Driver"
```

Levinud driverid:

| Driver | Kirjeldus |
|--------|-----------|
| json-file | vaikimisi, logid hostis |
| syslog | saadab syslog’i |
| journald | systemd journal |
| fluentd | saadab Fluentd agenti |
| awslogs | AWS CloudWatch |
| gelf | Graylog |

Näide:

```bash
docker run --log-driver=syslog nginx
```

---

# 3. Log rotation

Vaikimisi Docker EI tee log rotation’it.

Lisa `/etc/docker/daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Restart:

```bash
systemctl restart docker
```

---

# 4. containerd / CRI‑O logging

containerd logid:

```
/var/log/containers/
```

CRI logid:

```
/var/log/pods/
```

Vaata:

```bash
crictl logs <container>
```

---

# 5. Kubernetes Logging

Kubernetes EI kogu logisid ise.  
Logimine toimub:

- node tasemel (kubelet → container runtime)  
- log agent (Fluentd, Vector, Promtail)  
- tsentraliseeritud logisüsteem (ELK/EFK/Loki)  

---

# 6. Node-level logs

Pod logid:

```bash
kubectl logs web-123
```

Kõik konteinerid podis:

```bash
kubectl logs web-123 --all-containers
```

---

# 7. Sidecar logging pattern

Kui rakendus ei logi stdout’i, kasutatakse sidecar’i:

```yaml
containers:
  - name: app
    image: myapp
  - name: log-shipper
    image: fluentd
    volumeMounts:
      - name: logs
        mountPath: /var/log/app
```

---

# 8. EFK Stack (Elasticsearch + Fluentd + Kibana)

Kõige populaarsem Kubernetes logisüsteem.

### Fluentd → Elasticsearch → Kibana

Fluentd kogub logid:

```bash
kubectl -n logging get pods
```

Kibana UI:

```
http://kibana.example.com
```

---

# 9. Loki + Promtail + Grafana

Kaasaegne, odav ja kiire logisüsteem.

- **Promtail** kogub logid  
- **Loki** salvestab  
- **Grafana** visualiseerib  

Promtail daemonset:

```bash
kubectl get ds -n monitoring
```

---

# 10. Cloud Logging

Pilvepakkujad:

| Pilv | Logisüsteem |
|------|--------------|
| AWS | CloudWatch Logs |
| Azure | Log Analytics / Monitor |
| GCP | Cloud Logging |

Kubernetes saadab logid automaatselt cloud agenti.

---

# 11. Application-level logging

Soovitused:

- logi **JSON** formaadis  
- lisa **trace ID** ja **request ID**  
- ära logi paroole ega isikuandmeid  
- kasuta struktureeritud logimist (Zap, Logrus, Serilog)  

Näide JSON log:

```json
{
  "level": "info",
  "msg": "user logged in",
  "user": "peeter",
  "trace_id": "abc123"
}
```

---

# 12. Troubleshooting

### 12.1 Logid ei ilmu

Kontrolli:

```bash
kubectl logs <pod>
```

### 12.2 Fluentd crash

```bash
kubectl -n logging logs fluentd-0
```

### 12.3 Log rotation ei tööta

Kontrolli Docker daemon.json.

---

# 13. Best Practices

- Logi **stdout/stderr** → Kubernetes kogub automaatselt  
- Kasuta **JSON** logiformaati  
- Kasuta **log rotation**’it  
- Kasuta **EFK** või **Loki** tootmises  
- Ära logi tundlikke andmeid  
- Kasuta **trace ID** ja **correlation ID**  
- Kasuta sidecar’i, kui rakendus ei logi stdout’i  

---

# Kokkuvõte
Container logging hõlmab Docker log drivers, containerd/CRI‑O logisid, Kubernetes log agent’e ja tsentraliseeritud logisüsteeme nagu EFK või Loki.  
Struktureeritud logimine ja log rotation on kriitilised tootmiskeskkonnas.
```
