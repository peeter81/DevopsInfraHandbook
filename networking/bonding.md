# 📄 **Fail 15: `networking/bonding.md`**

---

```markdown
# Network Bonding – Networking käsiraamat  
## (LACP, Active‑Backup, Balance‑RR)

## Ülevaade
Network bonding (Linuxis ka *teaming*) võimaldab mitut füüsilist NIC‑i ühendada üheks loogiliseks liideseks.  
Eesmärgid:

- suurem töökindlus (failover)
- suurem läbilaskevõime (load balancing)
- link redundancy
- serverite ja storage’i kõrge saadavus

Linux toetab mitut bonding mode’i.

---

# 1. Bonding mode’id

## 1.1 mode=1 — Active‑Backup (HA)
- Üks NIC aktiivne, teine ooterežiimis  
- Kõige stabiilsem  
- Ei vaja switchi konfiguratsiooni  

```
Primary NIC → Active  
Secondary NIC → Standby
```

## 1.2 mode=4 — 802.3ad (LACP)
- Link Aggregation Control Protocol  
- Load balancing + redundancy  
- Vajab switchi LACP konfi  
- Kõige populaarsem tootmises  

## 1.3 mode=0 — Balance‑RR
- Round‑robin  
- Kõrgeim throughput  
- Võib põhjustada paketite järjekorra segadust  
- Vajab switchi tuge  

## 1.4 mode=2 — Balance‑XOR
- Hashipõhine load balancing  
- Vajab switchi tuge  

## 1.5 mode=5 — Balance‑TLB
- Transmit load balancing  
- Ei vaja switchi tuge  

## 1.6 mode=6 — Balance‑ALB
- Adaptive load balancing  
- Ei vaja switchi tuge  
- Toetab RX load balancingut  

---

# 2. Bonding konfiguratsioon Linuxis

## 2.1 Paigalda bonding moodul

```bash
modprobe bonding
```

## 2.2 Kontrolli, kas moodul on laaditud

```bash
lsmod | grep bonding
```

---

# 3. Bonding konfiguratsioon (Netplan – Ubuntu)

Näide LACP (mode 4):

```yaml
network:
  version: 2
  renderer: networkd
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: 802.3ad
        mii-monitor-interval: 100
      dhcp4: no
      addresses:
        - 10.0.0.10/24
      gateway4: 10.0.0.1
```

---

# 4. Bonding konfiguratsioon (ifcfg – RHEL/CentOS)

`/etc/sysconfig/network-scripts/ifcfg-bond0`:

```ini
DEVICE=bond0
TYPE=Bond
BONDING_OPTS="mode=802.3ad miimon=100"
BOOTPROTO=none
IPADDR=10.0.0.10
PREFIX=24
```

`/etc/sysconfig/network-scripts/ifcfg-eth0`:

```ini
DEVICE=eth0
MASTER=bond0
SLAVE=yes
```

---

# 5. Bonding staatuse kontroll

```bash
cat /proc/net/bonding/bond0
```

Näed:

- aktiivne liides  
- slave NIC‑id  
- LACP partner info  
- link state  

---

# 6. LACP switchi konfiguratsioon (näide)

Cisco:

```
interface range Gi1/0/1-2
  channel-group 1 mode active
```

Juniper:

```
set interfaces ge-0/0/0 ether-options 802.3ad ae0
set interfaces ge-0/0/1 ether-options 802.3ad ae0
```

---

# 7. Bonding + VLAN

```bash
ip link add link bond0 name bond0.100 type vlan id 100
ip addr add 10.10.100.10/24 dev bond0.100
ip link set bond0.100 up
```

---

# 8. Bonding arhitektuuri diagramm

```
+-----------+     +-----------+
|   eth0    |     |   eth1    |
+-----+-----+     +-----+-----+
      \               /
       \             /
        +-----------+
        |  bond0    |
        +-----------+
              |
              v
        Linux Networking
```

---

# 9. Best Practices

- Kui vajad **HA**, kasuta *active‑backup* (mode 1)  
- Kui vajad **läbilaskevõimet**, kasuta *LACP* (mode 4)  
- Kontrolli, et switchi mõlemad pordid oleksid samas LAG‑is  
- Ära sega erineva kiirusega NIC‑e samas bondis  
- Kontrolli MTU ühtlust  
- Kasuta `miimon=100` link monitoringut  

---

# Kokkuvõte
Network bonding suurendab töökindlust ja läbilaskevõimet, ühendades mitu NIC‑i üheks loogiliseks liideseks.  
Mode 1 sobib kõrge saadavuse jaoks, mode 4 (LACP) aga tootmiskeskkonna jõudluse ja stabiilsuse jaoks.
```
