# networking/ovs.md  
## Open vSwitch (OVS) – Networking käsiraamat

## Ülevaade

Open vSwitch (OVS) on virtualiseeritud switch, mida kasutatakse:

- Linuxi serverites  
- Kubernetes CNI-des (OVN‑Kubernetes)  
- OpenStackis  
- SDN lahendustes  
- VXLAN / Geneve overlay‑võrkudes  

OVS toetab:

- L2 switching  
- L3 routing  
- VXLAN, Geneve, GRE tunneldamist  
- OpenFlow protokolli  
- QoS, ACL, mirroring  

---

## 1. OVS põhikäskud

### 1.1 OVS konfiguratsiooni vaatamine

```bash
ovs-vsctl show
```

### 1.2 Bridge’i loomine

```bash
ovs-vsctl add-br br0
```

### 1.3 Bridge’i kustutamine

```bash
ovs-vsctl del-br br0
```

### 1.4 Pordi lisamine

```bash
ovs-vsctl add-port br0 eth1
```

### 1.5 Pordi eemaldamine

```bash
ovs-vsctl del-port br0 eth1
```

---

## 2. OVS + VXLAN

### 2.1 VXLAN tunneli loomine

```bash
ovs-vsctl add-port br0 vxlan100 \
    -- set interface vxlan100 type=vxlan options:remote_ip=192.168.1.20 options:key=100
```

Selgitus:

- **remote_ip** – teise hosti IP  
- **key=100** – VNI (VXLAN Network Identifier)  

### 2.2 Multicast VXLAN

```bash
ovs-vsctl add-port br0 vxlan100 \
    -- set interface vxlan100 type=vxlan options:group=239.1.1.1 options:key=100
```

---

## 3. OVS + Geneve (OVN‑Kubernetes)

Geneve on VXLANi uuem alternatiiv.

```bash
ovs-vsctl add-port br0 geneve0 \
    -- set interface geneve0 type=geneve options:remote_ip=192.168.1.20 options:key=200
```

---

## 4. OVS Flow’d (OpenFlow)

### 4.1 Flow’de vaatamine

```bash
ovs-ofctl dump-flows br0
```

### 4.2 Lihtne flow‑reegel

```bash
ovs-ofctl add-flow br0 "priority=100,in_port=1,actions=output:2"
```

### 4.3 Kõik flow’d kustutada

```bash
ovs-ofctl del-flows br0
```

---

## 5. OVS + Linux bridge võrdlus

| Funktsioon | Linux Bridge | OVS |
|-----------|--------------|-----|
| VXLAN | piiratud | täielik |
| Geneve | ei | jah |
| OpenFlow | ei | jah |
| SDN integratsioon | nõrk | tugev |
| Kubernetes tugi | keskmine | tugev (OVN) |
| Performance | hea | väga hea |

---

## 6. OVS arhitektuuri diagramm

```
+-------------------------------------------------------+
|                     ovs-vsctl                         |
|                (konfiguratsioon)                      |
+---------------------------+---------------------------+
                            |
                            v
+-------------------------------------------------------+
|                 OVSDB Server                          |
+---------------------------+---------------------------+
                            |
                            v
+-------------------------------------------------------+
|                 ovs-vswitchd                          |
|          (datapath controller)                        |
+---------------------------+---------------------------+
                            |
                            v
+-------------------------------------------------------+
|                 Kernel datapath                       |
+-------------------------------------------------------+
```

---

## 7. Debugimine

### 7.1 Logid

```bash
journalctl -u openvswitch-switch
```

### 7.2 Interface info

```bash
ovs-vsctl list interface vxlan100
```

### 7.3 Bridge info

```bash
ovs-vsctl list bridge br0
```

---

## 8. Best Practices

- Kasuta **OVN‑Kubernetes**, kui soovid enterprise‑taseme K8s võrku  
- Kasuta **Geneve**, kui võimalik (parem kui VXLAN)  
- Hoia OVS konfiguratsioon Git’is  
- Kasuta **OpenFlow** ainult vajadusel (keerukas)  
- Kontrolli MTU‑d (VXLAN / Geneve lisavad overhead’i)  

---

## Kokkuvõte

Open vSwitch on võimas ja paindlik virtualiseeritud switch, mis sobib nii serveritesse, SDN‑võrkudesse kui ka Kubernetes klastritesse.  
OVS on standardtööriist kaasaegsetes pilve‑ ja andmekeskuse võrkudes.
