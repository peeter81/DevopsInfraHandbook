# networking/vxlan.md  
## VXLAN – Networking käsiraamat

## Ülevaade

VXLAN (Virtual Extensible LAN) on L2‑võrgu laiendus üle L3‑võrgu.  
Seda kasutatakse:

- Kubernetes CNI-des (Flannel, Calico, Cilium)  
- Pilvevõrkudes  
- Suurtes andmekeskustes  
- Overlay‑võrkude loomiseks  

VXLAN kapseldab Etherneti kaadrid UDP pakettidesse (port 4789).

---

## 1. VXLAN põhimõte

VXLAN loob **virtuaalse L2 võrgu** üle L3 võrgu.

- Iga VXLAN segment = **VNI** (VXLAN Network Identifier)  
- Iga host loob **VXLAN tunneli** teiste hostidega  
- Kapseldamine toimub UDP kaudu  

---

## 2. VXLAN käsud Linuxis

### 2.1 VXLAN liidese loomine

```bash
ip link add vxlan100 type vxlan id 100 group 239.1.1.1 dev eth0 dstport 4789
```

Selgitus:

- **id 100** → VNI 100  
- **group 239.1.1.1** → multicast grupp  
- **dev eth0** → füüsiline NIC  
- **dstport 4789** → VXLAN UDP port  

### 2.2 Liidese aktiveerimine

```bash
ip link set up dev vxlan100
```

### 2.3 IP aadressi lisamine

```bash
ip addr add 10.10.10.1/24 dev vxlan100
```

---

## 3. VXLAN + bridge

VXLAN liidest kasutatakse tihti Linux bridge’i sees.

```bash
ip link add br0 type bridge
ip link set vxlan100 master br0
ip link set eth1 master br0
ip link set br0 up
```

---

## 4. Unicast VXLAN (EVPN / static)

Kui multicast ei ole saadaval → kasutatakse **static FDB** kirjeid.

### 4.1 FDB kirje lisamine

```bash
bridge fdb add 00:00:00:00:00:00 dev vxlan100 dst 192.168.1.20
```

### 4.2 FDB kirje vaatamine

```bash
bridge fdb show dev vxlan100
```

---

## 5. VXLAN paketistruktuur

```
+-------------------------------------------------------+
|                 Outer Ethernet                        |
+-------------------------------------------------------+
|                 Outer IP                              |
+-------------------------------------------------------+
|                 UDP Header                            |
+-------------------------------------------------------+
|                 VXLAN Header                          |
+-------------------------------------------------------+
|                 Inner Ethernet                        |
+-------------------------------------------------------+
|                 Inner Payload                         |
+-------------------------------------------------------+
```

---

## 6. VXLAN kasutus Kuberneteses

### Flannel (VXLAN backend)

- Loob VXLAN tunneli iga node vahel  
- Lihtne ja stabiilne  
- Kasutab UDP porti 8472  

### Calico (VXLAN mode)

- Võimaldab L3 routingut + VXLAN overlay  
- Toetab BGP + EVPN  

### Cilium

- Toetab VXLAN + Geneve  
- eBPF dataplane  

---

## 7. VXLAN arhitektuuri diagramm

```
Node A                                               Node B
-----------------                                   -----------------
|     Pod       |                                   |     Pod       |
-----------------                                   -----------------
       |                                                   |
       |  L2 traffic                                       |  L2 traffic
       v                                                   v
+-----------------+                           +-----------------+
|   vxlan100      | <==== UDP Tunnel =====>   |   vxlan100      |
+-----------------+                           +-----------------+
       |                                                   |
       v                                                   v
+-----------------+                           +-----------------+
|     eth0        |                           |     eth0        |
+-----------------+                           +-----------------+
```

---

## 8. Best Practices

- Kasuta **unicast FDB** kui multicast pole saadaval  
- Kuberneteses eelista **Cilium** või **Calico** VXLAN-i  
- Kontrolli MTU-d (VXLAN lisab ~50+ baiti)  
- Kasuta **jumbo frames** kui võimalik  
- Logi ja jälgi `bridge fdb` kirjeid  

---

## Kokkuvõte

VXLAN on kaasaegsete overlay‑võrkude alus.  
See võimaldab luua skaleeruvaid L2 võrke üle L3 infrastruktuuri ning on kriitiline komponent Kubernetes CNI-des ja pilvevõrkudes.
