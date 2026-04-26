# 📄 **Fail 17: `networking/vrf.md`**

```markdown
# VRF – Networking käsiraamat  
## (Virtual Routing and Forwarding)

## Ülevaade
VRF võimaldab ühel Linuxi või võrguseadme instantsil omada **mitut eraldatud routing‑tabelit**.  
See tähendab, et üks server või router saab käituda nagu mitu loogilist routerit.

VRF-i kasutatakse:
- multi‑tenant keskkondades  
- turvaliseks võrgu eraldamiseks  
- Kubernetes node’idel (storage, mgmt, pod network eraldus)  
- Telco ja ISP lahendustes  

---

# 1. VRF põhimõte

Ilma VRF‑ita on Linuxis **üks routing‑tabel**.  
VRF lisab mitu eraldatud tabelit:

```
VRF-A → routing table 100
VRF-B → routing table 200
VRF-C → routing table 300
```

Iga VRF:
- omab oma gateway’d  
- omab oma route’e  
- ei näe teiste VRF‑ide liiklust  

---

# 2. VRF loomine Linuxis

## 2.1 Loo VRF

```bash
ip link add vrf-blue type vrf table 100
ip link set vrf-blue up
```

## 2.2 Seo NIC VRF‑iga

```bash
ip link set eth0 master vrf-blue
```

## 2.3 Lisa IP aadress

```bash
ip addr add 10.10.10.10/24 dev eth0
```

## 2.4 Lisa gateway VRF‑i tabelisse

```bash
ip route add default via 10.10.10.1 table 100
```

---

# 3. VRF‑i kontroll

```bash
ip link show vrf-blue
ip route show table 100
```

---

# 4. VRF + ping

VRF‑i sees pingimiseks:

```bash
ip vrf exec vrf-blue ping 10.10.10.1
```

---

# 5. VRF + systemd (teenused VRF‑is)

Teenuse käivitamine VRF‑is:

```bash
ip vrf exec vrf-blue systemctl start nginx
```

või:

```bash
ip vrf exec vrf-blue curl http://10.10.10.20
```

---

# 6. VRF + Kubernetes

VRF‑e kasutatakse Kubernetes node’idel:
- storage‑võrgu eraldamiseks  
- management‑võrgu eraldamiseks  
- pod‑võrgu eraldamiseks  
- Telco CNI‑des (Multus, SR‑IOV, Calico Enterprise)  

Näide:
- vrf-mgmt → table 100  
- vrf-storage → table 200  
- vrf-pods → table 300  

---

# 7. VRF + routing policy (RPDB)

VRF kasutab **policy routingut**:

`/etc/iproute2/rt_tables`:

```
100 vrf-blue
200 vrf-red
```

Policy:

```bash
ip rule add iif eth0 table 100
ip rule add iif eth1 table 200
```

---

# 8. VRF arhitektuuri diagramm

```
+---------------------------+
|        Linux Host         |
+---------------------------+
      |           |
      v           v
+-----------+   +-----------+
|  VRF-A    |   |  VRF-B    |
| table 100 |   | table 200 |
+-----------+   +-----------+
      |               |
      v               v
   eth0            eth1
```

---

# 9. Best Practices

- Kasuta VRF‑e multi‑tenant ja turvalistes keskkondades  
- Ära sega storage, mgmt ja workload liiklust  
- Lisa VRF‑id systemd teenustele `ip vrf exec` abil  
- Kontrolli, et iga VRF‑i routing‑tabel oleks korrektne  
- Kuberneteses kasuta VRF‑e Telco ja HPC workload’ide jaoks  

---

# Kokkuvõte
VRF võimaldab Linuxil käituda nagu mitu eraldatud routerit.  
See on kriitiline tehnoloogia turvaliste, skaleeritavate ja multi‑tenant võrkude ehitamisel — eriti Kuberneteses, Telco ja enterprise keskkondades.
```
