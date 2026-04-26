# 📄 **Fail: `containers/kubernetes-observability.md`**

```markdown
# Kubernetes Observability – Containers käsiraamat  
## (Metrics, Logs, Traces, Prometheus, Grafana, Loki, Jaeger, OpenTelemetry)

## Ülevaade
Observability = **metrics + logs + traces**  
Kubernetes’is on see kriitiline:

- veaotsing  
- jõudluse analüüs  
- autoscaling  
- SLA/SLO jälgimine  
- turvalisus  

See fail katab kõige olulisemad tööriistad ja praktikad.

---

# 1. Metrics

Kõige olulisemad mõõdikud:

- CPU usage  
- Memory usage  
- Disk I/O  
- Network I/O  
- Pod restarts  
- Latency  
- Error rate  

Kubernetes kasutab:

- **cAdvisor** (container metrics)  
- **kube-state-metrics** (K8s objektid)  
- **Node Exporter** (node metrics)  
- **Prometheus** (scraping + storage)  

---

# 2. Prometheus

Prometheus on standardne monitoring stack.

## 2.1 Install (Helm)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prom prometheus-community/kube-prometheus-stack
```

## 2.2 Prometheus UI

```bash
kubectl port-forward svc/prom-kube-prometheus-stack-prometheus 9090:9090
```

---

# 3. Grafana

Grafana visualiseerib Prometheuse mõõdikuid.

## 3.1 Ava dashboard

```bash
kubectl port-forward svc/prom-grafana 3000:80
```

Logi sisse:

```
admin / prom-operator
```

## 3.2 Kubernetes dashboards

- Node metrics  
- Pod metrics  
- Container metrics  
- API server metrics  
- etcd metrics  

---

# 4. kube-state-metrics

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

# 5. Logs

Kubernetes EI kogu logisid ise.  
Logimine toimub:

- node tasemel (kubelet → container runtime)  
- log agent (Fluentd, Vector, Promtail)  
- tsentraliseeritud logisüsteem (ELK/EFK/Loki)  

---

# 6. Loki + Promtail + Grafana

Kaasaegne, odav ja kiire logisüsteem.

- **Promtail** kogub logid  
- **Loki** salvestab  
- **Grafana** visualiseerib  

Promtail daemonset:

```bash
kubectl get ds -n monitoring
```

---

# 7. Distributed Tracing

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

# 8. OpenTelemetry

OpenTelemetry (OTel) on standard:

- metrics  
- logs  
- traces  

Rakendused ekspordivad andmed OTel Collectori kaudu:

```yaml
exporters:
  otlp:
    endpoint: otel-collector:4317
```

---

# 9. Alerting

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

# 10. Probes (Health Checks)

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

# 11. Best Practices

- Kasuta **Prometheus + Grafana** tootmises  
- Kasuta **kube-prometheus-stack** kiireks seadistamiseks  
- Kasuta **OpenTelemetry** rakenduse mõõdikute ja tracingu jaoks  
- Kasuta **Alertmanager** teavituste jaoks  
- Kasuta **liveness/readiness probes**  
- Ära kogu liiga palju mõõdikuid (Prometheus võib täituda)  
- Kasuta **Loki** logide jaoks (odavam kui Elasticsearch)  

---

# Kokkuvõte
Kubernetes observability põhineb Prometheusel, Grafanal, Loki’l, Jaegeril ja OpenTelemetry’l.  
Metrics + logs + traces = täielik nähtavus ja töökindlus.
```

