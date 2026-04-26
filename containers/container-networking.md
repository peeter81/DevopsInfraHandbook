# 📄 **Fail: `containers/container-networking.md`**

```markdown
# Container Networking – Containers käsiraamat  
## (Bridge, host, none, overlay, macvlan, CNI, DNS, service discovery)

## Ülevaade
Konteinerite võrgundus on kriitiline osa konteinerplatvormidest.  
See fail katab:

- Docker networking  
- containerd/CRI‑O CNI  
- overlay võrgud  
- macvlan  
- DNS ja service discovery  
- Kubernetes võrgumudel  

---

# 1. Docker Networking

Vaata võrke:

```bash
docker network ls
```

### 1.1 bridge (vaikimisi)

- iga konteiner saab privaatse IP  
- NAT hosti kaudu  
- sobib lokaalseks arenduseks

Loo:

```bash
docker network create mynet
```

Käivita konteiner võrgus:

```bash
docker run --network=mynet nginx
```

---

### 1.2 host

- konteiner jagab hosti võrku  
- pole isolatsiooni  
- kiireim, kuid ebaturvaline

```bash
docker run --network=host nginx
```

---

### 1.3 none

- konteineril pole võrku  
- kasutatakse sandboxinguks

```bash
docker run --network=none nginx
```

---

### 1.4 overlay (Swarm/Kubernetes)

- mitme hosti vahel  
- kasutab VXLAN tunneldust  
- vajalik klastrites

```bash
docker network create -d overlay backend
```

---

### 1.5 macvlan

- konteiner saab **päris MAC‑aadressi**  
- näeb välja nagu eraldi masin  
- sobib IoT, legacy süsteemidele

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 macvlan0
```

---

# 2. containerd / CRI‑O networking (CNI)

Kubernetes kasutab **CNI pluginaid**:

- flannel  
- calico  
- cilium  
- weave  
- kube‑router  

CNI konfiguratsioon:

```
/etc/cni/net.d/
```

Vaata CNI pluginaid:

```
/opt/cni/bin/
```

---

# 3. Kubernetes Networking

Kubernetes võrgumudel:

- **iga pod saab oma IP**  
- podid suhtlevad ilma NAT‑ita  
- kõik node’d peavad olema routitud  
- Service annab püsiva IP  

---

# 4. Kubernetes Services

### 4.1 ClusterIP (vaikimisi)

- sisemine teenus  
- ainult klastris nähtav

### 4.2 NodePort

- avab pordi igal node’il  
- piiratud kasutus

### 4.3 LoadBalancer

- pilvepakkuja loob LB  
- tootmises standard

### 4.4 Headless Service

```yaml
clusterIP: None
```

- DNS tagastab **kõik podide IP‑d**  
- vajalik StatefulSet’idele

---

# 5. DNS ja Service Discovery

Kubernetes kasutab **CoreDNS**.

Näide DNS kirjest:

```
web.default.svc.cluster.local
```

Vaata DNS logisid:

```bash
kubectl -n kube-system logs -l k8s-app=kube-dns
```

---

# 6. NetworkPolicies

Lubab piirata liiklust podide vahel.

Näide: keelab kõik:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

---

# 7. Service Mesh (advanced)

Service mesh lisab:

- mTLS  
- traffic shaping  
- observability  
- retries  
- circuit breaking  

Populaarsed:

- Istio  
- Linkerd  
- Consul Connect  

---

# 8. Troubleshooting

### 8.1 DNS ei tööta

```bash
kubectl -n kube-system rollout restart deployment/coredns
```

### 8.2 Pod ei saa ühendust

```bash
kubectl exec -it pod -- curl http://service
```

### 8.3 CNI error

```bash
journalctl -u kubelet
ls /etc/cni/net.d/
```

---

# 9. Best Practices

- Kasuta overlay võrke klastrites  
- Kasuta NetworkPolicies turvalisuse jaoks  
- Ära kasuta host networkingut tootmises  
- Kasuta macvlan ainult spetsiaalsetel juhtudel  
- Kontrolli alati DNS ja CNI konfiguratsiooni  
- Kasuta service mesh’i mikroteenuste puhul  

---

# Kokkuvõte
Container networking hõlmab Docker bridge võrke, CNI pluginaid, overlay tunneldust, Kubernetes Services ja NetworkPolicies.  
Õige võrgumudel tagab skaleeritavuse, turvalisuse ja töökindluse.
```
