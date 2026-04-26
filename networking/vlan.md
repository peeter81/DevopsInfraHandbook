# 📄 **Fail 16: `networking/vlan.md`**

```markdown
# VLAN – Networking käsiraamat  
## (802.1Q, Trunk, Access, Tagging)

## Ülevaade
VLAN (Virtual LAN) võimaldab jagada ühe füüsilise võrgu mitmeks loogiliseks võrguks.  
Eesmärgid:

- võrgu segmenteerimine  
- turvalisus  
- broadcast domeenide vähendamine  
- multi‑tenant lahendused  
- Kubernetes node’ide ja serverite eraldamine  

VLAN-id põhinevad standardil **IEEE 802.1Q**.

---

# 1. VLAN põhimõtted

## 1.1 Access port
- kuulub **ühele VLAN‑ile**
- tag’e ei lisata
- kasutatakse tavaliselt lõppseadmetele (server, PC)

## 1.2 Trunk port
- kannab **mitut VLAN‑i**
- lisab 802.1Q tag’i
- kasutatakse switch ↔ switch ja switch ↔ server ühendustes

## 1.3 Native VLAN
- VLAN, mis liigub **ilma tag’ita**
- Cisco maailmas vaikimisi VLAN 1

---

# 2. VLAN konfiguratsioon Linuxis

## 2.1 VLAN liidese loomine

```bash
ip link add link eth0 name eth0.100 type vlan id 100
```

## 2.2 Liidese aktiveerimine

```bash
ip link set eth0.100 up
```

## 2.3 IP aadressi lisamine

```bash
ip addr add 10.10.100.10/24 dev eth0.100
```

---

# 3. VLAN konfiguratsioon Netplanis (Ubuntu)

```yaml
network:
  version: 2
  vlans:
    vlan100:
      id: 100
      link: eth0
      addresses:
        - 10.10.100.10/24
```

---

# 4. VLAN konfiguratsioon RHEL/CentOS

`/etc/sysconfig/network-scripts/ifcfg-eth0.100`:

```ini
DEVICE=eth0.100
BOOTPROTO=none
ONBOOT=yes
VLAN=yes
IPADDR=10.10.100.10
PREFIX=24
```

---

# 5. Switchi konfiguratsioon

## 5.1 Cisco – Access port

```
interface Gi1/0/1
  switchport mode access
  switchport access vlan 100
```

## 5.2 Cisco – Trunk port

```
interface Gi1/0/2
  switchport mode trunk
  switchport trunk allowed vlan 100,200,300
  switchport trunk native vlan 1
```

## 5.3 Juniper

```
set interfaces ge-0/0/1 unit 0 family ethernet-switching vlan members 100
```

---

# 6. VLAN + Bonding

```bash
ip link add link bond0 name bond0.200 type vlan id 200
ip addr add 10.10.200.10/24 dev bond0.200
ip link set bond0.200 up
```

---

# 7. VLAN + Bridge

```bash
ip link add br0 type bridge
ip link set eth0.100 master br0
ip link set br0 up
```

---

# 8. VLAN arhitektuuri diagramm

```
+---------------------------+
|         Switch            |
+---------------------------+
   | VLAN 100 | VLAN 200 |
   |          |          |
   v          v          v
+------+   +------+   +------+
| VM1  |   | VM2  |   | VM3  |
| .100 |   | .200 |   | .200 |
+------+   +------+   +------+
```

---

# 9. Best Practices

- Kasuta **trunk** porte serveritele ja hypervisoritele  
- Ära kasuta VLAN 1 tootmises  
- Dokumenteeri VLAN‑id (ID, nimi, otstarve)  
- Kontrolli MTU‑d (VLAN lisab 4 baiti)  
- Kasuta VLAN‑e turvapiiride loomiseks (prod, stage, mgmt)  
- Kuberneteses kasuta VLAN‑e node’ide ja storage’i eraldamiseks  

---

# Kokkuvõte
VLAN-id võimaldavad luua loogilisi võrke ühe füüsilise infrastruktuuri peale.  
Need on kriitilised nii serveriruumides, pilves kui ka Kubernetes klastrites, kus on vaja turvalist ja skaleeritavat võrgu segmentatsiooni.
```

