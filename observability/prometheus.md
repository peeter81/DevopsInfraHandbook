# 📄 **Fail: `observability/prometheus.md`**

```markdown
# Prometheus – Observability käsiraamat  
## (Metrics, Exporters, PromQL, Alerts, Service Discovery, Grafana)

## Ülevaade
Prometheus on tööstusstandard **metrics-based monitoring** süsteem:

- tõmbab andmeid (pull model)  
- salvestab ajareana (time series)  
- kasutab PromQL päringukeelt  
- integreerub Grafanaga  
- toetab alert’e Alertmanageri kaudu  

Prometheus on Kubernetes’e ja pilvekeskkondade de facto standard.

---

# 1. Prometheus arhitektuur

Komponendid:

- **Prometheus server** – kogub ja salvestab mõõdikuid  
- **Exporters** – muudavad süsteemi mõõdikud Prometheuse formaati  
- **Alertmanager** – saadab alert’e  
- **Grafana** – visualiseerimine  
- **Service Discovery** – leiab dünaamilised sihtmärgid (Kubernetes, EC2, Consul)

---

# 2. Metrics formaat

Prometheus kasutab **text-based exposition format**.

Näide:

```
http_requests_total{method="GET",code="200"} 1027
```

Komponendid:

- **metric name**  
- **labels**  
- **value**  

---

# 3. Exporters

Levinumad exporterid:

### 3.1 Node Exporter
Kogub OS-i mõõdikuid:

- CPU  
- RAM  
- disk  
- network  

Käivita:

```bash
./node_exporter
```

### 3.2 cAdvisor
Konteinerite mõõdikud.

### 3.3 Blackbox Exporter
HTTP/TCP/ICMP kontrollid.

### 3.4 MySQL/PostgreSQL Exporter
Andmebaasi mõõdikud.

### 3.5 SNMP Exporter
Võrguseadmed.

---

# 4. Prometheus konfiguratsioon

`prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
      - targets: ["localhost:9100"]
```

---

# 5. PromQL päringud

### 5.1 Lihtne päring

```
node_cpu_seconds_total
```

### 5.2 Filtreerimine

```
node_cpu_seconds_total{mode="idle"}
```

### 5.3 Rate (per-second)

```
rate(http_requests_total[5m])
```

### 5.4 Summeerimine

```
sum(rate(http_requests_total[5m])) by (method)
```

### 5.5 CPU kasutus %

```
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

---

# 6. Alerts (Alertmanager)

`alert.rules`:

```yaml
groups:
  - name: node-alerts
    rules:
      - alert: HighCPU
        expr: avg(rate(node_cpu_seconds_total{mode!="idle"}[5m])) > 0.8
        for: 2m
        labels:
          severity: warning
        annotations:
          description: "CPU usage > 80%"
```

---

# 7. Alertmanager konfiguratsioon

`alertmanager.yml`:

```yaml
route:
  receiver: email

receivers:
  - name: email
    email_configs:
      - to: "ops@example.com"
```

---

# 8. Grafana integratsioon

Grafana loeb Prometheust kui data source’i:

```
URL: http://prometheus:9090
```

Levinud dashboardid:

- Node Exporter Full  
- Kubernetes Cluster Monitoring  
- Blackbox Exporter  
- PostgreSQL Overview  

---

# 9. Kubernetes integratsioon

Prometheus Operator loob:

- ServiceMonitor  
- PodMonitor  
- Alertmanager  
- Prometheus CRD  

Näide ServiceMonitor:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  endpoints:
    - port: metrics
```

---

# 10. Best Practices

- Kasuta **Prometheus Operator** Kuberneteses  
- Kasuta **rate()** ja **irate()** ajapõhiste mõõdikute jaoks  
- Kasuta **labels** mõõdikute rikastamiseks  
- Ära kasuta liiga pikki scrape intervalle  
- Kasuta **recording rules** keerukate päringute kiirendamiseks  
- Kasuta **federation** suurte klastrite jaoks  
- Kasuta **Grafana dashboards** visualiseerimiseks  

---

# Kokkuvõte
Prometheus on võimas ja paindlik monitooringusüsteem, mis kogub mõõdikuid, võimaldab keerukaid päringuid ja integreerub Alertmanageri ning Grafanaga.  
See on Kubernetes’e ja pilvekeskkondade standardlahendus.
```
