# 📄 **Fail: `observability/loki.md`**

```markdown
# Loki – Observability käsiraamat  
## (Log aggregation, LogQL, Promtail, Grafana, Retention, Indexing)

## Ülevaade
Loki on Grafana Labs’i loodud **logide kogumise ja päringute süsteem**, mis on:

- odav (minimaalne indeks)  
- skaleeruv  
- Kubernetes-sõbralik  
- Prometheuse loogikaga kooskõlas  
- integreeritud Grafanaga  

Loki ei indekseeri logide sisu — ainult **labels**.  
See muudab Loki oluliselt odavamaks kui Elasticsearch.

---

# 1. Loki arhitektuur

Komponendid:

- **Loki** – logide salvestus  
- **Promtail** – logide kogumine serveritest / konteineritest  
- **Grafana** – visualiseerimine  
- **Distributor / Ingester / Querier** (horizontaalne skaleerimine)  

---

# 2. Promtail

Promtail kogub logisid:

- failidest  
- journald’ist  
- Kubernetes podidest  

Näide `promtail.yaml`:

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

---

# 3. LogQL päringud

LogQL on Loki päringukeel (sarnane PromQL-ile).

### 3.1 Lihtne päring

```
{job="nginx"}
```

### 3.2 Filtreerimine

```
{job="nginx"} |= "error"
```

### 3.3 Regex filtreerimine

```
{job="nginx"} |~ "5.."
```

### 3.4 Logide loendamine

```
count_over_time({job="nginx"}[5m])
```

### 3.5 Error rate

```
rate({job="nginx"} |= "error" [5m])
```

---

# 4. Loki + Grafana

Grafana paneel LogQL päringuga:

```
{app="backend"} |= "timeout"
```

Visualiseerimise tüübid:

- Logs  
- Time series (kui kasutada `rate()` või `count_over_time()`)  

---

# 5. Loki konfiguratsioon

`loki-config.yaml`:

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  filesystem:
    directory: /loki/chunks
```

---

# 6. Kubernetes integratsioon

Promtail DaemonSet:

```bash
helm install promtail grafana/promtail
```

Loki:

```bash
helm install loki grafana/loki
```

Grafana:

```bash
helm install grafana grafana/grafana
```

---

# 7. Loki vs Elasticsearch

| Funktsioon | Loki | Elasticsearch |
|-----------|------|---------------|
| Hind | väga odav | kallis |
| Indekseerimine | ainult labels | kogu logi |
| Päringud | LogQL | Lucene |
| Kubernetes tugi | suurepärane | hea |
| Ressursikulu | madal | kõrge |
| Skaalumine | lihtne | keerukas |

---

# 8. Retention ja storage

Loki toetab:

- filesystem  
- S3  
- GCS  
- Azure Blob  
- MinIO  

Retention seadistamine:

```yaml
limits_config:
  retention_period: 168h   # 7 päeva
```

---

# 9. Best Practices

- Kasuta **labels** targalt (väldi kõrge kardinaalsusega label’eid)  
- Kasuta **Promtail** logide kogumiseks  
- Kasuta **Grafana** visualiseerimiseks  
- Kasuta **S3/MinIO** pikaajaliseks storage’iks  
- Kasuta **retention** kulude kontrollimiseks  
- Ära pane logisid liiga detailseks (väldi massiivseid JSON objekte)  

---

# Kokkuvõte
Loki on kerge, skaleeruv ja odav logide kogumise süsteem, mis sobib ideaalselt Kubernetes’e ja Prometheus/Grafana stack’i kõrvale.  
LogQL + Promtail + Grafana = täielik logide observability lahendus.
```
