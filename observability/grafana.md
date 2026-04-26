# 📄 **Fail: `observability/grafana.md`**

```markdown
# Grafana – Observability käsiraamat  
## (Dashboards, Data Sources, Panels, Alerts, Variables, Provisioning)

## Ülevaade
Grafana on tööstusstandard **visualiseerimise ja observability** platvorm:

- toetab kümneid andmeallikaid (Prometheus, Loki, Elasticsearch, InfluxDB, PostgreSQL)  
- võimaldab luua dünaamilisi dashboards  
- toetab alert’e  
- sobib ideaalselt Prometheuse ja Loki kõrvale  

---

# 1. Data Sources

Grafana loeb andmeid erinevatest allikatest.

### Levinumad:
- **Prometheus** (metrics)  
- **Loki** (logid)  
- **Elasticsearch**  
- **InfluxDB**  
- **PostgreSQL / MySQL**  
- **Tempo** (tracing)  

Lisa Prometheus:

```
URL: http://prometheus:9090
```

---

# 2. Dashboards

Dashboard koosneb:

- **Panels**  
- **Rows**  
- **Variables**  
- **Data Sources**  

Paneelide tüübid:

- Graph  
- Time series  
- Gauge  
- Stat  
- Table  
- Logs  
- Heatmap  

---

# 3. Panels

Näide PromQL päringust paneelis:

```
rate(http_requests_total[5m])
```

Paneeli komponendid:

- Query  
- Visualization  
- Legend  
- Thresholds  
- Transformations  

---

# 4. Variables

Variables muudavad dashboardid dünaamiliseks.

Näide labeli põhine variable:

```yaml
Name: instance
Query: label_values(node_cpu_seconds_total, instance)
```

Kasutamine paneelis:

```
rate(node_cpu_seconds_total{instance="$instance"}[5m])
```

---

# 5. Alerts

Grafana toetab:

- **Grafana Alerting** (uus)  
- **Legacy Alerting** (vana)  

Näide alert reeglist:

```
Condition: avg() OF query(A, 5m) IS ABOVE 80
```

Notification channels:

- Email  
- Slack  
- Teams  
- PagerDuty  
- Webhook  

---

# 6. Loki + Grafana (logs)

Loki on Prometheuse analoog logidele.

Näide LogQL päringust:

```
{job="nginx"} |= "error"
```

---

# 7. Provisioning (automaatne seadistus)

Grafana võimaldab automaatset seadistamist:

- datasources  
- dashboards  
- alert rules  

Näide datasource provisioning:

`/etc/grafana/provisioning/datasources/prometheus.yaml`:

```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    access: proxy
    isDefault: true
```

---

# 8. Dashboard provisioning

`/etc/grafana/provisioning/dashboards/default.yaml`:

```yaml
apiVersion: 1
providers:
  - name: default
    folder: "Default"
    type: file
    options:
      path: /var/lib/grafana/dashboards
```

---

# 9. Grafana + Kubernetes

Grafana helm chart:

```bash
helm install grafana grafana/grafana
```

Parool:

```bash
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

---

# 10. Best Practices

- Kasuta **variables** dashboardide paindlikkuseks  
- Kasuta **folders** dashboardide organiseerimiseks  
- Kasuta **provisioning** automaatseks seadistamiseks  
- Kasuta **Grafana Alerting** uue süsteemi jaoks  
- Kasuta **Loki** logide jaoks  
- Kasuta **Tempo** tracingu jaoks  
- Ära tee liiga keerulisi päringuid paneelides (kasuta recording rules)  

---

# Kokkuvõte
Grafana on võimas observability tööriist, mis võimaldab visualiseerida metrics, logs ja traces.  
Prometheus + Loki + Grafana = täielik observability stack.
```
