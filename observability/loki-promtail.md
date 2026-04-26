# 📄 **Fail: `observability/loki-promtail.md`**

```markdown
# Loki + Promtail – Logide kogumise käsiraamat  
## (Promtail config, pipelines, relabeling, Kubernetes, systemd logs)

## Ülevaade
Promtail on Loki ametlik logikogur.  
See:

- loeb logisid failidest, journald’ist või Kubernetes podidest  
- lisab label’id  
- saadab logid Loki serverisse  
- toetab relabeling’ut ja pipeline protsessoreid  

Promtail + Loki = täielik logide kogumise lahendus.

---

# 1. Promtail install

## 1.1 Linux

```bash
wget https://github.com/grafana/loki/releases/download/v2.9.0/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
chmod +x promtail-linux-amd64
```

## 1.2 systemd teenus

`/etc/systemd/system/promtail.service`:

```ini
[Unit]
Description=Promtail service

[Service]
ExecStart=/usr/local/bin/promtail-linux-amd64 -config.file=/etc/promtail/promtail.yaml
Restart=always

[Install]
WantedBy=multi-user.target
```

---

# 2. Promtail põhi‑konfiguratsioon

`/etc/promtail/promtail.yaml`:

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /var/lib/promtail/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: varlogs
    static_configs:
      - targets: [localhost]
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

---

# 3. Labelid

Promtail lisab labelid, mida Loki kasutab indekseerimiseks.

Näide:

```yaml
labels:
  job: nginx
  env: prod
```

**NB!** Väldi kõrge kardinaalsusega label’eid (näiteks `user_id`, `request_id`).

---

# 4. Relabeling

Relabeling võimaldab muuta label’eid enne Loki poole saatmist.

Näide failinime labeliks:

```yaml
relabel_configs:
  - source_labels: ["__path__"]
    target_label: "filename"
```

Näide hostname lisamiseks:

```yaml
relabel_configs:
  - target_label: "host"
    replacement: "${HOSTNAME}"
```

---

# 5. Pipeline protsessorid

Promtail võimaldab logisid töödelda:

- `regex`  
- `replace`  
- `json`  
- `timestamp`  
- `drop`  

Näide JSON logide parsimisest:

```yaml
pipeline_stages:
  - json:
      expressions:
        level: level
        msg: message
  - labels:
      level:
      msg:
```

Näide ridade eemaldamisest:

```yaml
pipeline_stages:
  - drop:
      expression: ".*healthcheck.*"
```

---

# 6. Journald logid

Promtail saab lugeda systemd journald logisid:

```yaml
scrape_configs:
  - job_name: journald
    journal:
      max_age: 12h
      labels:
        job: systemd
```

---

# 7. Kubernetes integratsioon

Promtail DaemonSet:

```bash
helm install promtail grafana/promtail
```

Promtail automaatselt:

- loeb podi logisid `/var/log/pods/...`  
- lisab labelid: `namespace`, `pod`, `container`  
- lisab Kubernetes metadata  

---

# 8. Docker logid

Docker logid asuvad:

```
/var/lib/docker/containers/*/*.log
```

Promtail config:

```yaml
scrape_configs:
  - job_name: docker
    static_configs:
      - targets: [localhost]
        labels:
          job: docker
          __path__: /var/lib/docker/containers/*/*.log
```

---

# 9. Promtail + TLS

```yaml
clients:
  - url: https://loki.example.com/loki/api/v1/push
    tls_config:
      ca_file: /etc/ssl/certs/ca.crt
      cert_file: /etc/ssl/certs/client.crt
      key_file: /etc/ssl/private/client.key
```

---

# 10. Best Practices

- Kasuta **positions.yaml** logide jätkamiseks  
- Kasuta **pipeline_stages** JSON logide parsimiseks  
- Väldi **kõrge kardinaalsusega label’eid**  
- Kasuta **journald** systemd logide jaoks  
- Kasuta **Helm chart** Kuberneteses  
- Kasuta **drop** müra eemaldamiseks  
- Kasuta **TLS** tootmises  

---

# Kokkuvõte
Promtail on Loki logikogur, mis toetab faililogisid, journald’i ja Kubernetes podide logisid.  
Pipeline + relabeling + labels = paindlik ja võimas logide töötlemine.
```
