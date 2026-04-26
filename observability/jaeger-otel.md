# 📄 **Fail: `observability/jaeger-otel.md`**

```markdown
# Jaeger + OpenTelemetry – Distributed Tracing käsiraamat  
## (Traces, Spans, Sampling, OTLP, Collector, UI, Deployment)

## Ülevaade
Jaeger on CNCF projekt, mis pakub **distributed tracing** lahendust.  
See sobib ideaalselt mikroteenuste arhitektuuride jälgimiseks.

OpenTelemetry (OTel) on standard, mis kogub traces → Jaeger salvestab ja visualiseerib need.

---

# 1. Jaeger arhitektuur

Komponendid:

- **Jaeger Collector** – võtab vastu OTLP või Jaeger protokolli  
- **Jaeger Query** – API päringuteks  
- **Jaeger UI** – trace’ide visualiseerimine  
- **Storage** – Elasticsearch, Cassandra, Badger, Tempo  

Tüüpiline arhitektuur:

```
App → OTel SDK → OTel Collector → Jaeger Collector → Storage → Jaeger UI
```

---

# 2. OpenTelemetry → Jaeger

OTel Collector konfiguratsioon:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

exporters:
  jaeger:
    endpoint: jaeger-collector:14250
    tls:
      insecure: true

processors:
  batch:

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [jaeger]
```

---

# 3. Tracing põhimõisted

### Span  
Üks operatsioon (nt HTTP päring, DB query).

### Trace  
Span’ide kogum, mis moodustab päringu teekonna.

### Attributes  
Key-value metadata.

### Context propagation  
Trace ID liigub teenuste vahel HTTP headerites:

```
traceparent: 00-<trace_id>-<span_id>-01
```

---

# 4. Auto-instrumentation

OTel suudab automaatselt instrumenteerida:

### Java

```bash
java -javaagent:opentelemetry-javaagent.jar \
  -Dotel.exporter.otlp.endpoint=http://otel-collector:4317 \
  -jar app.jar
```

### Python

```bash
pip install opentelemetry-distro
opentelemetry-bootstrap -a install
opentelemetry-instrument python app.py
```

### Node.js

```bash
npm install @opentelemetry/auto-instrumentations-node
```

---

# 5. Jaeger UI

Jaeger UI võimaldab:

- otsida traces teenuse järgi  
- vaadata span’e  
- näha latentsust  
- analüüsida sõltuvusi  
- võrrelda trace’e  

URL tavaliselt:

```
http://localhost:16686
```

---

# 6. Sampling

Sampling määrab, kui palju trace’e kogutakse.

### AlwaysOn

```yaml
sampler:
  type: always_on
```

### ParentBased + TraceIDRatio

```yaml
sampler:
  type: parentbased_traceidratio
  argument: 0.1   # 10%
```

---

# 7. Jaeger + Kubernetes

Helm install:

```bash
helm install jaeger jaegertracing/jaeger
```

OTel Collector:

```bash
helm install otel-collector open-telemetry/opentelemetry-collector
```

---

# 8. Jaeger vs Tempo

| Funktsioon | Jaeger | Tempo |
|-----------|--------|--------|
| Storage | ES, Cassandra, Badger | Object storage (S3, GCS) |
| Hind | kallim | odav |
| Skaalumine | keerukam | lihtne |
| UI | Jaeger UI | Grafana |
| OTel tugi | väga hea | väga hea |

Tempo sobib odavaks massiivseks tracinguks, Jaeger sobib analüütikaks.

---

# 9. Best Practices

- Kasuta **OTLP** protokolli  
- Kasuta **batch processor** jõudluse jaoks  
- Kasuta **sampling** kulude kontrollimiseks  
- Kasuta **auto-instrumentation** võimalusel  
- Ära lisa liiga palju attributes (kardinaalsus!)  
- Kasuta **Jaeger UI** latentsuse analüüsiks  
- Kasuta **Tempo** kui soovid odavamat storage’it  

---

# Kokkuvõte
Jaeger + OpenTelemetry on võimas tracing stack, mis võimaldab jälgida päringute teekonda läbi mikroteenuste.  
OTLP → Collector → Jaeger = täielik tracing pipeline.
```
