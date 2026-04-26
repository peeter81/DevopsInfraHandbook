# 📄 **Fail: `observability/elastic.md`**

```markdown
# Elasticsearch – Observability käsiraamat  
## (Logs, Search, Indexing, Kibana, Beats, Ingest Pipelines)

## Ülevaade
Elasticsearch on hajutatud otsingu- ja analüütikamootor, mida kasutatakse:

- logide kogumiseks  
- täisteksti otsinguks  
- analüütikaks  
- observability stack’i osana (ELK: Elasticsearch + Logstash + Kibana)  

Elasticsearch on võimas, kuid ressursinõudlik ja keerukam kui Loki.

---

# 1. Elasticsearch arhitektuur

Komponendid:

- **Cluster** – kogu süsteem  
- **Node** – üks server  
- **Index** – loogiline andmekogu  
- **Shard** – indeksi tükk  
- **Replica** – varukoopia  

Tüüpiline tootmisklaster:

```
3x master nodes
3x data nodes
1x ingest node
1x kibana
```

---

# 2. Indeksid ja dokumendid

Elasticsearch salvestab JSON dokumente.

Näide dokument:

```json
{
  "timestamp": "2026-04-25T12:00:00Z",
  "level": "error",
  "message": "Database connection failed"
}
```

Indeksi loomine:

```bash
PUT /logs-2026-04-25
```

Dokumendi lisamine:

```bash
POST /logs-2026-04-25/_doc
{
  "message": "Hello world"
}
```

---

# 3. Kibana

Kibana on Elasticsearchi UI:

- dashboards  
- visualiseerimine  
- Discover (logide otsing)  
- Dev Tools (päringud)  
- Alerts  

---

# 4. Beats

Beats on kergekaalulised agentid:

### 4.1 Filebeat  
Kogub logifaile.

### 4.2 Metricbeat  
Kogub süsteemi mõõdikuid.

### 4.3 Packetbeat  
Võrguliikluse analüüs.

### 4.4 Heartbeat  
Uptime monitoring.

Filebeat konfiguratsioon:

```yaml
filebeat.inputs:
  - type: log
    paths:
      - /var/log/*.log

output.elasticsearch:
  hosts: ["http://elasticsearch:9200"]
```

---

# 5. Logstash

Logstash on logide töötlemise pipeline:

- input  
- filter  
- output  

Näide:

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "apache-%{+YYYY.MM.dd}"
  }
}
```

---

# 6. Ingest Pipelines

Elasticsearch võib logisid töödelda ilma Logstashita.

Näide pipeline:

```json
PUT _ingest/pipeline/nginx
{
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{COMBINEDAPACHELOG}"]
      }
    }
  ]
}
```

Kasutamine:

```bash
POST logs/_doc?pipeline=nginx
```

---

# 7. Päringud (Query DSL)

### 7.1 Match query

```json
{
  "query": {
    "match": {
      "message": "error"
    }
  }
}
```

### 7.2 Range query

```json
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "now-1h"
      }
    }
  }
}
```

### 7.3 Bool query

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "level": "error" } }
      ],
      "filter": [
        { "range": { "timestamp": { "gte": "now-5m" } } }
      ]
    }
  }
}
```

---

# 8. Elasticsearch vs Loki

| Funktsioon | Elasticsearch | Loki |
|-----------|---------------|------|
| Hind | kallis | odav |
| Indekseerimine | kogu logi | ainult labels |
| Päringud | Query DSL | LogQL |
| Ressursikulu | kõrge | madal |
| Skaalumine | keerukas | lihtne |
| Kasutusjuht | otsing + analüütika | logid + observability |

---

# 9. Best Practices

- Kasuta **ilmtingimata** 3 master node’i  
- Kasuta **ilmtingimata** snapshot’e (S3, GCS)  
- Ära pane liiga palju indekseid (shard explosion)  
- Kasuta **ILM** (Index Lifecycle Management)  
- Kasuta **Beats** logide kogumiseks  
- Kasuta **Ingest Pipelines** lihtsate transformatsioonide jaoks  
- Ära hoia logisid liiga kaua (kulukas)  

---

# Kokkuvõte
Elasticsearch on võimas otsingu- ja logianalüüsi mootor, mis sobib suurtele süsteemidele, kus on vaja keerukaid päringuid ja analüütikat.  
ELK stack (Elasticsearch + Logstash + Kibana) on klassikaline observability lahendus.
```
