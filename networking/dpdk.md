# networking/dpdk.md  
## DPDK – Networking käsiraamat  
### (Data Plane Development Kit)

## Ülevaade

DPDK (Data Plane Development Kit) on kõrge jõudlusega paketitöötluse raamistik, mis võimaldab:

- mööduda Linuxi kernelist (kernel bypass)  
- töödelda pakette otse user space’is  
- saavutada miljoneid pakette sekundis (Mpps)  
- kasutada NIC‑i queue’sid ja CPU pinningut  

DPDK-d kasutatakse:

- NFV (Network Function Virtualization)  
- SDN dataplane  
- Telco 5G Core  
- High‑performance load balancerid  
- Firewallid ja IDS/IPS süsteemid  

---

## 1. DPDK arhitektuur

DPDK kasutab:

- **hugepages** (2 MB või 1 GB)  
- **poll mode driver (PMD)** – ei kasuta interrupts’e  
- **NUMA‑aware memory allocation**  
- **CPU pinning** (isolcpus)  

---

## 2. HugePages seadistamine

### 2.1 Kontrolli olemasolevaid hugepage’e

```bash
cat /proc/meminfo | grep Huge
```

### 2.2 Lisa hugepage’d

```bash
echo 2048 > /proc/sys/vm/nr_hugepages
```

### 2.3 Mount hugepage FS

```bash
mkdir -p /mnt/huge
mount -t hugetlbfs nodev /mnt/huge
```

---

## 3. NIC‑i sidumine DPDK driveriga

DPDK kasutab tavaliselt:

- **vfio-pci** (soovitatud)  
- **igb_uio** (vanem)  

### 3.1 Lae vfio-pci

```bash
modprobe vfio-pci
```

### 3.2 Leia NIC PCI aadress

```bash
lspci | grep Eth
```

### 3.3 Seo NIC DPDK driveriga

```bash
dpdk-devbind.py --bind=vfio-pci 0000:03:00.0
```

---

## 4. DPDK testpmd (benchmark)

### 4.1 testpmd käivitamine

```bash
dpdk-testpmd -l 1-3 -n 4 -- --port-topology=chained --forward-mode=io --auto-start
```

Selgitus:

- **-l 1-3** → CPU core’id  
- **-n 4** → memory channels  
- **forward-mode=io** → lihtne forwarding  

### 4.2 testpmd käsud

```
show port stats all
start
stop
quit
```

---

## 5. NUMA ja CPU pinning

DPDK jõudlus sõltub NUMA‑st.

### 5.1 Kontrolli NUMA topoloogiat

```bash
lscpu | grep NUMA
```

### 5.2 Pin CPU core’id

```bash
taskset -c 2-5 dpdk-testpmd ...
```

---

## 6. DPDK + OVS (OVS‑DPDK)

Open vSwitch saab kasutada DPDK dataplane’i.

### 6.1 OVS‑DPDK lubamine

```bash
ovs-vsctl set Open_vSwitch . other_config:dpdk-init=true
```

### 6.2 DPDK memory seadistamine

```bash
ovs-vsctl set Open_vSwitch . other_config:dpdk-socket-mem="1024,1024"
```

### 6.3 DPDK port lisamine

```bash
ovs-vsctl add-port br0 dpdk0 -- set Interface dpdk0 type=dpdk
```

---

## 7. DPDK paketivoo diagramm

```
+-------------------------------------------------------+
|                 NIC RX Queue                          |
+---------------------------+---------------------------+
                            |
                            v
+-------------------------------------------------------+
|           Poll Mode Driver (PMD)                      |
+---------------------------+---------------------------+
                            |
                            v
+-------------------------------------------------------+
|           DPDK User Space (fast packet processing)    |
+---------------------------+---------------------------+
                            |
                            v
+-------------------------------------------------------+
|                 NIC TX Queue                          |
+-------------------------------------------------------+
```

---

## 8. Best Practices

- Kasuta **vfio-pci** (turvalisem ja kiirem)  
- Kasuta **isolcpus** kernel boot parameetrit  
- Kasuta **hugepages** (vähem TLB misses)  
- Kasuta **NUMA‑aware** CPU ja memory paigutust  
- Väldi interrupts’e – PMD on palju kiirem  
- Kasuta **OVS‑DPDK**, kui vajad SDN dataplane’i  

---

## Kokkuvõte

DPDK on kriitiline tehnoloogia kõrge jõudlusega võrgusüsteemides.  
See võimaldab user space’is töödelda pakette äärmiselt kiiresti, möödudes kernelist ja saavutades telco‑taseme jõudluse.
