# 📄 **Fail: `containers/container-orchestration.md`**

```markdown
# Container Orchestration – Containers käsiraamat  
## (Kubernetes, Swarm, Nomad, autoscaling, scheduling, HA, workloads)

## Ülevaade
Container orchestration tähendab konteinerite automaatset haldamist:

- deploy  
- scale  
- heal  
- restart  
- networking  
- storage  
- secrets  
- rolling updates  

Kõige populaarsemad orkestreerijad:

- **Kubernetes** (tööstusstandard)
- **Docker Swarm** (lihtne)
- **HashiCorp Nomad** (kerge, multi‑runtime)
- **OpenShift** (enterprise Kubernetes)

---

# 1. Miks on orkestreerimist vaja?

Konteinerid on:

- lühiealised  
- ebapüsivad  
- võivad crash’ida  
- vajavad networkingut  
- vajavad storage’t  
- vajavad load balancingut  

Orkestreerija lahendab kõik need probleemid automaatselt.

---

# 2. Kubernetes (tööstusstandard)

Kubernetes pakub:

- automaatne scheduling  
- self‑healing  
- rolling updates  
- autoscaling  
- service discovery  
- secrets & config  
- storage provisioning  
- network policies  

### Kubernetes komponendid:

- **API Server**
- **Scheduler**
- **Controller Manager**
- **etcd**
- **Kubelet**
- **Kube‑proxy**
- **Container runtime (containerd/CRI‑O)**

---

# 3. Docker Swarm (lihtne orkestreerija)

Plussid:
- väga lihtne  
- Docker CLI‑ga ühilduv  
- kiire seadistada  

Miinused:
- piiratud funktsionaalsus  
- pole nii skaleeritav kui Kubernetes  
- vähe enterprise funktsioone  

---

# 4. HashiCorp Nomad

Nomad on:

- kerge  
- kiire  
- töötab ilma Kubernetes’i keerukuseta  
- toetab mitut workload’i: Docker, exec, Java, QEMU, raw binary  

Plussid:
- väga lihtne  
- väga stabiilne  
- integreerub Consul + Vault’iga  

---

# 5. Orchestration Features

## 5.1 Scheduling

Scheduler otsustab:
- millisele node’ile pod läheb  
- arvestab CPU/RAM nõudeid  
- arvestab taustakoormust  
- arvestab affinity/anti‑affinity  

## 5.2 Self‑healing

Kui pod crash’ib:
- Kubernetes käivitab uue  
- Swarm käivitab uue  
- Nomad käivitab uue  

## 5.3 Rolling updates

Näide Kubernetes:

```bash
kubectl rollout restart deployment web
```

## 5.4 Autoscaling

Kubernetes HPA:

```bash
kubectl autoscale deployment web --cpu-percent=70 --min=2 --max=10
```

## 5.5 Service discovery

DNS:

```
web.default.svc.cluster.local
```

## 5.6 Secrets management

Kubernetes:

```bash
kubectl create secret generic db --from-literal=password=123
```

---

# 6. Workload Types

### Kubernetes:
- Deployment  
- StatefulSet  
- DaemonSet  
- Job  
- CronJob  

### Nomad:
- service  
- batch  
- system  
- periodic  

---

# 7. High Availability (HA)

Orkestreerija tagab:

- mitme node’i töö  
- automaatne failover  
- leader election  
- replikatsioon  

Kubernetes HA:
- 3+ master node’i  
- etcd klaster  

---

# 8. Networking

Orkestreerija loob:

- overlay networks  
- service load balancing  
- ingress / gateway  
- network policies  

---

# 9. Storage

Orkestreerija haldab:

- PVC/PV (Kubernetes)  
- CSI draiverid  
- automaatne provisioning  

---

# 10. Observability

Orkestreerija integreerub:

- Prometheus  
- Grafana  
- Loki  
- Jaeger  
- OpenTelemetry  

---

# 11. Best Practices

- Kasuta Kubernetes’t tootmises  
- Kasuta Swarm’i väikestes projektides  
- Kasuta Nomad’it kui vajad lihtsust  
- Kasuta autoscalingut  
- Kasuta rolling updates  
- Kasuta network policies  
- Kasuta CSI storage’t  
- Kasuta observability stack’i  

---

# Kokkuvõte
Container orchestration automatiseerib konteinerite deploy, scaling, networking ja storage halduse.  
Kubernetes on tööstusstandard, Swarm on lihtne ja Nomad on kerge alternatiiv.
```
