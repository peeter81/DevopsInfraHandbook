# 📄 **Fail: `containers/kubernetes-networking.md`**

```markdown
# Kubernetes Networking – Containers käsiraamat  
## (Pod networking, Services, DNS, Ingress, CNI, NetworkPolicies)

## Ülevaade
Kubernetes networking on üks keerukamaid, kuid olulisemaid teemasid.  
K8s võrgumudel põhineb neljal põhiprintsiibil:

1. **Iga pod saab oma IP‑aadressi**  
2. **Podid suhtlevad omavahel ilma NAT‑ita**  
3. **Node’id peavad olema täielikult routitud**  
4. **Service pakub püsiva IP ja load balancing’u**

---

# 1. Pod Networking

Iga pod saab oma IP‑aadressi CNI plugina kaudu:

- Calico  
- Cilium  
- Flannel  
- Weave  
- Kube‑router  

Podide vahel pole NAT‑i — see on Kubernetes’e võrgumudeli alus.

Vaata podi IP:

```bash
kubectl get pod -o wide
```

---

# 2. CNI (Container Network Interface)

CNI pluginad loovad:

- veth paarid  
- overlay võrgud  
- routing  
- IPAM (IP haldus)  

CNI konfiguratsioon:

```
/etc/cni/net.d/
```

Pluginad:

```
/opt/cni/bin/
```

---

# 3. Services

Service loob püsiva IP‑aadressi ja load balancing’u podide vahel.

## 3.1 ClusterIP (vaikimisi)

Ainult klastris nähtav.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - port: 80
```

## 3.2 NodePort

Avab pordi igal node’il.

```yaml
type: NodePort
```

## 3.3 LoadBalancer

Pilvepakkuja loob LB.

```yaml
type: LoadBalancer
```

## 3.4 Headless Service

DNS tagastab kõik podide IP‑d.

```yaml
clusterIP: None
```

---

# 4. DNS (CoreDNS)

Kubernetes kasutab **CoreDNS** teenust.

DNS kirje:

```
web.default.svc.cluster.local
```

Vaata DNS logisid:

```bash
kubectl -n kube-system logs -l k8s-app=kube-dns
```

---

# 5. Ingress

Ingress haldab HTTP/HTTPS liiklust.

Näide:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
spec:
  rules:
    - host: web.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

Ingress controllerid:

- NGINX  
- Traefik  
- HAProxy  
- Istio Gateway  

---

# 6. NetworkPolicies

NetworkPolicy piirab liiklust podide vahel.

Näide: keelab kõik ühendused:

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

Näide: lubab ainult ühe teenuse:

```yaml
ingress:
  - from:
      - podSelector:
          matchLabels:
            app: frontend
```

---

# 7. kube-proxy

kube-proxy haldab Service routingut:

- iptables  
- IPVS (kiirem)  

Vaata:

```bash
kubectl -n kube-system get pods -l k8s-app=kube-proxy
```

---

# 8. Overlay Networking

Kubernetes kasutab overlay võrke:

- VXLAN (Flannel, Calico)  
- BPF tunneldus (Cilium)  

Overlay võimaldab podidel suhelda üle mitme node’i.

---

# 9. Troubleshooting

### 9.1 DNS ei tööta

```bash
kubectl -n kube-system rollout restart deployment/coredns
```

### 9.2 Pod ei saa ühendust

```bash
kubectl exec -it pod -- curl http://service
```

### 9.3 CNI error

```bash
journalctl -u kubelet
ls /etc/cni/net.d/
```

### 9.4 NetworkPolicy blokib liiklust

```bash
kubectl describe networkpolicy
```

---

# 10. Best Practices

- Kasuta **Calico** või **Cilium** tootmises  
- Kasuta **NetworkPolicies** turvalisuse jaoks  
- Kasuta **Ingress** HTTP/HTTPS liikluse haldamiseks  
- Ära kasuta NodePort’i tootmises  
- Kasuta **Headless Service** StatefulSet’ide jaoks  
- Kasuta **IPVS** kube-proxy režiimi parema jõudluse jaoks  

---

# Kokkuvõte
Kubernetes networking põhineb podide IP‑del, CNI pluginate routingul, Services load balancingul, DNS‑il ja NetworkPolicies turvalisusel.  
See on orkestreerimise üks olulisemaid ja keerukamaid osi.
```
