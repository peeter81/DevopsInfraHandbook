# 📄 **Fail: `observability/rabbitmq.md`**  

```markdown
# RabbitMQ – Observability käsiraamat  
## (Metrics, Logs, Prometheus Exporter, Health Checks, Tracing)

## Ülevaade
RabbitMQ on sõnumijärjekorra süsteem (message broker), mida kasutatakse:

- mikroteenuste vaheliseks suhtluseks  
- tööjärjekordadeks (work queues)  
- pub/sub mustriteks  
- event-driven arhitektuuriks  

Observability seisukohalt on RabbitMQ oluline, sest:

- sõnumite kuhjumine mõjutab süsteemi jõudlust  
- tarbijate (consumers) aeglus tekitab latentsust  
- ühenduste arv ja kanalid võivad kasvada kontrollimatult  
- sõnumite kaod või dead-lettered sõnumid vajavad jälgimist  

---

# 1. Metrics (Prometheus Exporter)

RabbitMQ-l on ametlik Prometheus exporter.

## 1.1 Lubamine

`rabbitmq.conf`:

```
management.tcp.port = 15672
prometheus.tcp.port = 15692
```

Prometheus scrape:

```yaml
scrape_configs:
  - job_name: rabbitmq
    static_configs:
      - targets: ["rabbitmq:15692"]
```

## 1.2 Olulised mõõdikud

### Queue metrics
- `rabbitmq_queue_messages_ready`  
- `rabbitmq_queue_messages_unacked`  
- `rabbitmq_queue_messages_total`  

### Consumer metrics
- `rabbitmq_queue_consumers`  
- `rabbitmq_channel_consumers`  

### Connection metrics
- `rabbitmq_connections`  
- `rabbitmq_channels`  

### Node metrics
- `rabbitmq_node_mem_used`  
- `rabbitmq_node_disk_free`  

---

# 2. Logs

RabbitMQ logid asuvad:

```
/var/log/rabbitmq/rabbit@<hostname>.log
/var/log/rabbitmq/startup.log
```

Olulised logisõnumid:

- ühenduse katkestused  
- kanalite sulgemised  
- sõnumite tagasilükkamised (nack)  
- dead-lettered sõnumid  
- cluster node’i probleemid  

---

# 3. Health checks

RabbitMQ toetab HTTP health check’e:

```
http://rabbitmq:15672/api/health/checks/alarms
http://rabbitmq:15672/api/health/checks/local_alarms
http://rabbitmq:15672/api/health/checks/node_is_mirror_sync_critical
```

Kubernetes readiness/liveness:

```yaml
livenessProbe:
  httpGet:
    path: /api/health/checks/alarms
    port: 15672
```

---

# 4. Tracing

RabbitMQ toetab **OpenTelemetry tracingut**:

`rabbitmq.conf`:

```
opentelemetry.enabled = true
opentelemetry.traces.sampling = always
opentelemetry.exporter.otlp.endpoint = http://otel-collector:4317
```

See võimaldab jälgida:

- publish → queue → consumer teekonda  
- latentsust  
- sõnumite töötlemise aega  

---

# 5. Dead Letter Queue (DLQ) observability

DLQ on kriitiline osa jälgimisest.

Olulised mõõdikud:

- `rabbitmq_queue_messages_ready{queue="dead-letter"}`  
- `rabbitmq_queue_messages_total{queue="dead-letter"}`  

DLQ kasv = probleem tarbijas või sõnumi formaadis.

---

# 6. Grafana dashboardid

Levinud dashboardid:

- RabbitMQ Overview  
- Queue Performance  
- Consumer Latency  
- Node Health  
- Connections & Channels  

---

# 7. Best Practices

- Kasuta **Prometheus exporterit**  
- Pane alertid queue pikkuse ja unacked sõnumite peale  
- Kasuta **DLQ** vigaste sõnumite tuvastamiseks  
- Kasuta **OTel tracingut** publish → consume jälgimiseks  
- Kasuta **health check’e** Kuberneteses  
- Pane alertid node disk free ja memory alarmide peale  

---

# Kokkuvõte
RabbitMQ observability hõlmab metrics, logs, tracing ja health check’e.  
Prometheus + Grafana + OTel = täielik RabbitMQ jälgimise lahendus.
```

---

# ✔ Vastus sinu teisele küsimusele  
**Kas RabbitMQ kuulub observability kategooriasse?**

**Jah — RabbitMQ ise ei ole observability tööriist, kuid RabbitMQ *observability* on kriitiline osa süsteemi jälgimisest.**  
Seda käsitletakse tavaliselt observability kategoorias, sest:

- RabbitMQ tekitab metrics  
- RabbitMQ tekitab logs  
- RabbitMQ toetab tracingut  
- RabbitMQ seisund mõjutab kogu süsteemi tervist  

Seega RabbitMQ observability on täiesti õigustatud osa sinu dokumentatsiooni *observability* peatükist.
