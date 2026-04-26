# 📄 **Fail: `observability/alertmanager.md`**

```markdown
# Alertmanager – Alerts käsiraamat  
## (Routing, Receivers, Inhibition, Silences, Templates)

## Ülevaade
Alertmanager on Prometheuse alertide haldussüsteem.  
See:

- võtab vastu Prometheuse alertid  
- rühmitab need  
- rakendab reegleid (routing, inhibition)  
- saadab teavitused (email, Slack, Teams, webhook)  
- võimaldab silences (ajutine vaigistamine)

---

# 1. Alertmanager konfiguratsioon

`alertmanager.yml`:

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: default
  group_by: ["alertname", "instance"]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h

receivers:
  - name: default
    email_configs:
      - to: "ops@example.com"
```

---

# 2. Routing

Routing määrab, millised alertid kuhu saadetakse.

Näide:

```yaml
route:
  receiver: default
  routes:
    - match:
        severity: critical
      receiver: pagerduty
    - match:
        severity: warning
      receiver: slack
```

---

# 3. Receivers

### Email

```yaml
email_configs:
  - to: "ops@example.com"
```

### Slack

```yaml
slack_configs:
  - channel: "#alerts"
    api_url: https://hooks.slack.com/services/XXX
```

### Webhook

```yaml
webhook_configs:
  - url: http://alert-handler:8080/alert
```

---

# 4. Inhibition

Inhibition = ära saada madalama taseme alerti, kui kõrgema taseme alert on juba aktiivne.

Näide:

```yaml
inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ["instance"]
```

---

# 5. Silences

Silence = ajutine alerti vaigistamine.

CLI:

```bash
amtool silence add alertname=HighCPU --duration=2h
```

Kõik silences:

```bash
amtool silence query
```

---

# 6. Alertmanager + Prometheus

Prometheus konfiguratsioon:

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager:9093"]
```

---

# 7. Alert templating

Alertmanager kasutab Go templatingut.

Näide Slacki teate mall:

```yaml
slack_configs:
  - channel: "#alerts"
    text: |
      *Alert:* {{ .CommonLabels.alertname }}
      *Instance:* {{ .CommonLabels.instance }}
      *Description:* {{ .CommonAnnotations.description }}
```

---

# 8. High availability

Alertmanager toetab HA klastrit:

```yaml
--cluster.listen-address=0.0.0.0:9094
--cluster.peer=alertmanager-1:9094
--cluster.peer=alertmanager-2:9094
```

---

# 9. Best Practices

- Kasuta **grouping** müra vähendamiseks  
- Kasuta **inhibition** duplikaatide vältimiseks  
- Kasuta **silences** hooldustööde ajal  
- Kasuta **Slack/Webhook** kiireks reageerimiseks  
- Pane Alertmanager **HA klastrisse**  
- Ära saada liiga palju alert’e (alert fatigue)  

---

# Kokkuvõte
Alertmanager on Prometheuse alertide keskne haldussüsteem, mis võimaldab routingut, groupingut, inhibition’it ja teavitusi.  
Prometheus + Alertmanager = täielik alerting stack.
```
