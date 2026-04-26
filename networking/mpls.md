# 📄 **Fail 18: `networking/mpls.md`**

```markdown
# MPLS – Networking käsiraamat  
## (Multiprotocol Label Switching)

## Ülevaade
MPLS on tehnoloogia, mis suunab pakette **siltide (labels)**, mitte IP‑aadresside järgi.  
See võimaldab:

- kiiremat paketiedastust  
- VPN‑e (L3VPN, L2VPN)  
- Traffic Engineering (TE)  
- QoS tagamist  
- Telco ja ISP võrkude skaleerimist  

MPLS töötab **L2 ja L3 vahel** (Layer 2.5).

---

# 1. MPLS põhimõte

Pakett saab **MPLS label’i**, mille alusel routerid (LSR‑id) teevad forwardingut.

MPLS label stack:

```
+---------------------------+
| Label | TC | S | TTL     |
+---------------------------+
```

- **Label** – suunamisteave  
- **TC** – QoS  
- **S** – stack bottom flag  
- **TTL** – time to live  

---

# 2. MPLS rollid

## 2.1 CE – Customer Edge
- kliendi router  
- ei tea MPLS‑ist midagi

## 2.2 PE – Provider Edge
- lisab ja eemaldab MPLS label’e  
- hoiab VRF‑e (L3VPN)

## 2.3 P – Provider router
- forwardib MPLS label’e  
- ei näe kliendi IP‑sid

---

# 3. MPLS forwarding

## 3.1 Label push (PE → P)
Lisatakse MPLS label.

## 3.2 Label swap (P → P)
Asendatakse label uuega.

## 3.3 Label pop (P → PE)
Eemaldatakse label.

---

# 4. MPLS L3VPN

L3VPN kasutab:
- VRF‑e  
- MP‑BGP‑d  
- MPLS label’e  

Liiklus on klienditi eraldatud.

Näide VRF:

```bash
ip vrf add customer1
ip link set eth0 master customer1
ip route add table customer1 10.1.0.0/16 via 10.1.1.1
```

---

# 5. MPLS Linuxis

Linux toetab MPLS‑i kernelis.

## 5.1 Lubamine

```bash
modprobe mpls_router
modprobe mpls_iptunnel
```

## 5.2 MPLS label route

```bash
ip -f mpls route add 200 via inet 192.168.1.2
```

## 5.3 MPLS tunnel

```bash
ip link add mplstun0 type mpls_iptunnel local 192.168.1.1 remote 192.168.1.2 label 100
```

---

# 6. MPLS Traffic Engineering (TE)

Kasutab:
- RSVP‑TE  
- explicit path’e  
- bandwidth reservation  

Sobib Telco ja suurte ISP‑de jaoks.

---

# 7. MPLS + Segment Routing (SR‑MPLS)

Segment Routing on MPLS‑i moderniseeritud versioon.

Eelised:
- ei vaja LDP‑d  
- lihtsam konfigureerida  
- skaleerub paremini  
- töötab hästi koos EVPN‑iga  

---

# 8. MPLS + EVPN

EVPN (Ethernet VPN) kasutab MPLS‑i, et pakkuda:
- L2VPN  
- L3VPN  
- MAC learning BGP kaudu  
- multi‑homingut  

Kasutatakse:
- andmekeskustes  
- Telco võrkudes  
- Kubernetes edge‑võrkudes  

---

# 9. MPLS arhitektuuri diagramm

```
CE --- PE --- P --- P --- PE --- CE
        |           |
      VRF         Label swap
```

---

# 10. Best Practices

- Kasuta VRF‑e kliendiliikluse eraldamiseks  
- Kasuta Segment Routingut uutes projektides  
- Kasuta EVPN‑i L2VPN/L3VPN jaoks  
- Dokumenteeri label’id ja VRF‑id  
- Kasuta MPLS‑i ainult siis, kui vaja Telco/ISP skaleeritavust  

---

# Kokkuvõte
MPLS on Telco ja ISP võrkude selgroog.  
See võimaldab kiiret forwardingut, VPN‑e, traffic engineeringut ja skaleeritavat multi‑tenant arhitektuuri.  
Linux toetab MPLS‑i, kuid suuremad lahendused kasutavad spetsiaalseid routereid.
```
