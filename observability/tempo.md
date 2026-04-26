# 📄 **Fail: `observability/tempo.md`**

```markdown
# Tempo – Distributed Tracing käsiraamat  
## (Tracing backend, OTLP, Storage, Grafana, Scaling)

## Ülevaade
Grafana Tempo on **odav, skaleeruv ja lihtne tracing backend**, mis töötab ideaalselt koos:

- OpenTelemetry (OTel)
- Grafana
- Loki
- Prometheus

Tempo on optimeeritud massiivsete trace-mahtude jaoks, kasutades **object storage** (S3, GCS, Azure Blob, MinIO).

---

# 1. Tempo arhitektuur

Komponendid:

- **Distributor** – võtab vastu trace’d
- **Ingester** – kirjutab trace’d object storage’i
- **Querier** – loeb trace’d
- **Query Frontend** – caching + paralleelsus
- **Compactor** – optimeerib storage’i

Tempo ei vaja indekseid → väga odav.

---

# 2. OpenTelemetry → Tempo

OTel Collector konfiguratsioon:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

exporters:
  otlp:
    endpoint: tempo:4317
    tls:
      insecure: true

processors:
  batch:

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp]
```

---

# 3. Tempo konfiguratsioon

`tempo.yaml`:

```yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
        http:

storage:
  trace:
    backend: s3
    s3:
      bucket: tempo-traces
      endpoint: s3.amazonaws.com
      access_key: xxx
      secret_key: yyy
```

---

# 4. Tempo + Grafana

Grafana datasource:

```
Type: Tempo
URL: http://tempo:3200
```

Grafana võimaldab:

- trace visualiseerimist
- span detailide vaatamist
- trace → logs → metrics pivot’it (Loki + Prometheus)

---

# 5. Tempo + Loki + Prometheus (Full Observability)

Tempo → traces  
Loki → logs  
Prometheus → metrics  

Grafana ühendab need üheks:

- **From trace to logs**
- **From logs to trace**
- **From trace to metrics**

See on observability “kolmainsus”.

---

# 6. Tempo + Kubernetes

Helm install:

```bash
helm install tempo grafana/tempo
```

OTel Collector:

```bash
helm install otel-collector open-telemetry/opentelemetry-collector
```

---

# 7. Tempo vs Jaeger

| Funktsioon | Tempo | Jaeger |
|-----------|--------|--------|
| Storage | Object storage | ES, Cassandra, Badger |
| Hind | väga odav | kallim |
| Skaalumine | väga lihtne | keerukam |
| UI | Grafana | Jaeger UI |
| OTel tugi | suurepärane | suurepärane |

Tempo sobib massiivseks tracinguks, Jaeger analüütikaks.

---

# 8. Best Practices

- Kasuta **OTLP** protokolli  
- Kasuta **object storage** (S3/MinIO)  
- Kasuta **batch processor** Collector’is  
- Ära lisa liiga palju attributes (kardinaalsus!)  
- Kasuta **Grafana** visualiseerimiseks  
- Kasuta **Tempo** kui soovid odavat ja skaleeruvat tracingut  

---

# Kokkuvõte
Tempo on lihtne, odav ja skaleeruv tracing backend, mis sobib ideaalselt OpenTelemetry ja Grafana ökosüsteemi.  
OTLP → Collector → Tempo → Grafana = täielik tracing pipeline.
```
