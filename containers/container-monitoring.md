# 📄 **Fail: `containers/container-monitoring.md`**

```markdown
# Container Monitoring – Containers käsiraamat  
## (Metrics, logs, traces, Prometheus, Grafana, cAdvisor, Node Exporter, Alerting)

## Ülevaade
Konteinerite ja Kubernetes’i jälgimine koosneb kolmest sambast:

- **Metrics** – CPU, RAM, I/O, latency  
- **Logs** – rakenduse ja süsteemi logid  
- **Traces** – request flow mikroteenustes  

See fail katab kõige olulisemad tööriistad ja praktikad.

---

# 1. Metrics

Kõige olulisemad mõõdikud:

- CPU usage  
- Memory usage  
- Disk I/O  
- Network I/O  
- Container restarts  
- Pod health  
- Latency  
- Error rate  

---

# 2. cAdvisor (container metrics)

cAdvisor on Google’i tööriist, mis kogub konteinerite mõõdikuid.

Kubernetesis töötab see kubelet’i sees.

Vaata cAdvisor endpoint’i:

```
http://<node-ip>:4194
```

---

# 3. Prometheus

Prometheus on standardne monitoring stack konteinerite ja Kubernetes’i jaoks.

## 3.1 Install (Helm)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prom prometheus-community/kube-prometheus-stack
```

## 3.2 Prometheus UI

```bash
kubectl port-forward svc/prom-kube-prometheus-stack-prometheus 9090:9090
```

---

# 4. Grafana

Grafana visualiseerib Prometheuse mõõdikuid.

## 4.1 Ava dashboard

```bash
kubectl port-forward svc/prom-grafana 3000:80
```

Logi sisse:

```
admin / prom-operator
```

## 4.2 Kubernetes dashboards

- Node metrics  
- Pod metrics  
- Container metrics  
- API server metrics  
- etcd metrics  

---

# 5. Node Exporter

Node Exporter kogub hosti mõõdikuid:

- CPU  
- RAM  
- Disk  
- Network  
- Filesystem  

Prometheus scrape’ib seda automaatselt kube-prometheus-stack’is.

---

# 6. Kube-State-Metrics

Kogub Kubernetes objektide mõõdikuid:

- Deployments  
- StatefulSets  
- DaemonSets  
- Pods  
- Nodes  
- PVC-d  
- Services  

Näide mõõdikust:

```
kube_pod_status_phase
```

---

# 7. Alertmanager

Alertmanager saadab teavitusi:

- Slack  
- Email  
- Teams  
- PagerDuty  
- OpsGenie  

Näide alert:

```yaml
groups:
  - name: node-alerts
    rules:
      - alert: HighCPU
        expr: node_cpu_seconds_total > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          description: "Node CPU usage is high"
```

---

# 8. Application Monitoring

Rakenduse mõõdikud:

- HTTP latency  
- HTTP error rate  
- Request throughput  
- DB query time  
- Cache hit/miss  

### Parimad teegid:

- **Prometheus client** (Go, Python, Java, Node.js)  
- **OpenTelemetry** (universaalne standard)  

Näide Prometheus endpoint:

```
/metrics
```

---

# 9. Distributed Tracing

Tracing näitab request’i teekonda läbi mikroteenuste.

Tööriistad:

- **Jaeger**  
- **Zipkin**  
- **OpenTelemetry Collector**  

Näide trace:

```
frontend → api → auth → db
```

---

# 10. Kubernetes Probes

## 10.1 Liveness probe

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

## 10.2 Readiness probe

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

## 10.3 Startup probe

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
```

---

# 11. Logging vs Metrics vs Tracing

| Tüüp | Milleks? | Näited |
|------|----------|--------|
| **Metrics** | jõudlus, alerting | CPU, RAM, latency |
| **Logs** | veaotsing | stack traces, events |
| **Traces** | request flow | distributed tracing |

Kõik kolm on vajalikud.

---

# 12. Best Practices

- Kasuta **Prometheus + Grafana** tootmises  
- Kasuta **kube-prometheus-stack** kiireks seadistamiseks  
- Kasuta **OpenTelemetry** rakenduse mõõdikute ja tracingu jaoks  
- Kasuta **Alertmanager** teavituste jaoks  
- Kasuta **liveness/readiness probes**  
- Ära kogu liiga palju mõõdikuid (Prometheus võib täituda)  

---

# Kokkuvõte
Container monitoring põhineb Prometheusel, Grafanal, cAdvisoril, Node Exporteril ja OpenTelemetry’l.  
Metrics + logs + traces = täielik nähtavus ja töökindlus.
```
