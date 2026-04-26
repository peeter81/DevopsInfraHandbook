# 📄 **Fail: `observability/opentelemetry.md`**

```markdown
# OpenTelemetry – Observability käsiraamat  
## (Tracing, Metrics, Logs, OTLP, Collectors, Instrumentation)

## Ülevaade
OpenTelemetry (OTel) on CNCF standard **tracingu, metrics’i ja logide** kogumiseks.  
See on vendor‑neutraalne ja töötab koos:

- Grafana Tempo  
- Jaeger  
- Prometheus  
- Loki  
- Elastic APM  
- Datadog  
- New Relic  
- Honeycomb  

OTel = universaalne observability SDK + protokoll + kollektor.

---

# 1. OpenTelemetry komponendid

### 1.1 SDK / Instrumentation  
Rakenduse kood instrumenteeritakse automaatselt või käsitsi.

### 1.2 OTLP protokoll  
Standardne protokoll (HTTP/GRPC) telemeetria edastamiseks.

### 1.3 Collector  
Võtab vastu → töötleb → saadab edasi.

Collector on OTel südames.

---

# 2. OpenTelemetry Collector

Collector koosneb:

- **Receivers** – kust andmed tulevad  
- **Processors** – töötlemine  
- **Exporters** – kuhu andmed lähevad  

Näide `otel-collector.yaml`:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  batch:

exporters:
  logging:
  otlp:
    endpoint: tempo:4317
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging, otlp]
```

---

# 3. Tracing

Tracing jälgib päringuid läbi süsteemi:

- span → üks operatsioon  
- trace → span’ide kogum  
- attributes → metadata  

Näide Python:

```python
from opentelemetry import trace
from opentelemetry.instrumentation.requests import RequestsInstrumentor

RequestsInstrumentor().instrument()

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("process_request"):
    do_work()
```

---

# 4. Metrics

OTel toetab:

- Counter  
- Gauge  
- Histogram  

Näide Go:

```go
counter, _ := meter.Int64Counter("requests_total")
counter.Add(ctx, 1)
```

---

# 5. Logs

OTel logid:

- struktureeritud  
- JSON  
- saadetakse OTLP kaudu  

Näide:

```json
{
  "timestamp": "...",
  "severity": "INFO",
  "body": "User logged in",
  "attributes": {
    "user_id": 123
  }
}
```

---

# 6. Auto-instrumentation

OTel suudab automaatselt instrumenteerida:

- Python  
- Java  
- Node.js  
- .NET  
- Go (osaliselt)  

Näide Node.js:

```bash
npm install @opentelemetry/auto-instrumentations-node
```

---

# 7. OpenTelemetry + Tempo (Tracing)

Tempo on Grafana tracing backend.

Collector → Tempo:

```yaml
exporters:
  otlp:
    endpoint: tempo:4317
    tls:
      insecure: true
```

Grafana visualiseerib traces.

---

# 8. OpenTelemetry + Prometheus (Metrics)

Collector → Prometheus:

```yaml
exporters:
  prometheus:
    endpoint: "0.0.0.0:9464"
```

Prometheus scrape’ib seda endpointi.

---

# 9. OpenTelemetry + Loki (Logs)

Collector → Loki:

```yaml
exporters:
  loki:
    endpoint: http://loki:3100/loki/api/v1/push
```

---

# 10. Kubernetes integratsioon

OTel Collector DaemonSet:

```bash
helm install opentelemetry-collector open-telemetry/opentelemetry-collector
```

Auto-instrumentation sidecar:

```yaml
annotations:
  instrumentation.opentelemetry.io/inject-java: "true"
```

---

# 11. Best Practices

- Kasuta **Collector** kui keskset telemeetria hub’i  
- Kasuta **OTLP** kui standardprotokolli  
- Kasuta **auto-instrumentation** võimalusel  
- Ära lisa liiga palju attributes (kardinaalsus!)  
- Kasuta **batch processor** jõudluse jaoks  
- Kasuta **Tempo** tracingu jaoks  
- Kasuta **Prometheus** metrics’i jaoks  
- Kasuta **Loki** logide jaoks  

---

# Kokkuvõte
OpenTelemetry on universaalne observability standard, mis ühendab metrics, logs ja traces üheks ökosüsteemiks.  
OTLP + Collector + Tempo/Prometheus/Loki = täielik vendor‑neutraalne observability stack.
```
