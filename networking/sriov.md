# networking/sriov.md  
## SR‑IOV – Networking käsiraamat  
### (Single Root I/O Virtualization)

## Ülevaade

SR‑IOV on tehnoloogia, mis võimaldab füüsilise NIC-i jagada mitmeks **virtuaalseks funktsiooniks (VF)**.  
See annab VM‑idele või konteineritele **otse ligipääsu NIC‑ile**, möödudes virtuaalsest switchist.

Eelised:

- Väga kõrge jõudlus (madal latency, kõrge throughput)  
- Kernel bypass (vähem CPU overhead’i)  
- Ideaalne NFV, Telco, HPC ja Kubernetes workload’idele  

---

## 1. SR‑IOV komponendid

### PF – Physical Function
- Täielik NIC-i funktsionaalsus  
- Haldus (admin)  

### VF – Virtual Function
- Kerge virtuaalne NIC  
- Antakse VM‑ile või Pod‑ile  

---

## 2. Kontrolli, kas NIC toetab SR‑IOV

```bash
lspci | grep -i ether
```

Näide PCI aadressist:

```
0000:03:00.0 Ethernet controller: Intel Corporation X710
```

Kontrolli SR‑IOV tuge:

```bash
lspci -vvv -s 0000:03:00.0 | grep -i sriov
```

---

## 3. VF‑ide loomine Linuxis

### 3.1 Loo 8 VF‑i

```bash
echo 8 > /sys/class/net/eth0/device/sriov_numvfs
```

### 3.2 Kontrolli VF‑e

```bash
ip link show eth0
```

Näed ridu:

```
vf 0 MAC ...
vf 1 MAC ...
...
```

---

## 4. VF‑i MAC ja VLAN seadistamine

### 4.1 Määra VF‑ile MAC

```bash
ip link set eth0 vf 0 mac 00:11:22:33:44:55
```

### 4.2 Määra VF‑ile VLAN

```bash
ip link set eth0 vf 0 vlan 100
```

---

## 5. VF‑i andmine VM‑ile (libvirt)

`domain.xml`:

```xml
<interface type='hostdev'>
    <source>
        <address type='pci' domain='0x0000' bus='0x03' slot='0x10' function='0x0'/>
    </source>
</interface>
```

---

## 6. SR‑IOV Kuberneteses

SR‑IOV kasutatakse Kuberneteses **DPDK**, **Telco 5G**, **HPC** ja **ultra‑low‑latency** workload’ide jaoks.

### 6.1 SR‑IOV Network Operator (OpenShift / upstream)

Komponendid:

- **sriov-network-operator**  
- **sriov-device-plugin**  
- **sriov-cni**  

### 6.2 Device Plugin näide

```yaml
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetworkNodePolicy
metadata:
  name: sriov-nic
spec:
  resourceName: sriovnic
  numVfs: 8
  nicSelector:
    pfNames: ["eth0"]
```

### 6.3 Pod, mis kasutab SR‑IOV VF‑i

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sriov-pod
spec:
  containers:
    - name: test
      image: centos
      command: ["sleep", "3600"]
      resources:
        limits:
          sriovnic: 1
```

---

## 7. SR‑IOV + DPDK

SR‑IOV VF‑id saab siduda DPDK driveritega:

```bash
dpdk-devbind.py --bind=vfio-pci 0000:03:10.0
```

See annab VM‑ile või Pod‑ile **DPDK‑võimelise NIC‑i**.

---

## 8. SR‑IOV arhitektuuri diagramm

```
+-------------------------------------------------------+
|                 Physical NIC (PF)                     |
+---------------------------+---------------------------+
                            |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
+-------------+     +-------------+     +-------------+
|    VF 0     |     |    VF 1     |     |    VF 2     |
+-------------+     +-------------+     +-------------+
        |                   |                   |
        v                   v                   v
    VM / Pod           VM / Pod           VM / Pod
```

---

## 9. Best Practices

- Kasuta **vfio-pci** drivereid (turvaline ja kiire)  
- Ära loo liiga palju VF‑e (NIC performance langeb)  
- Kasuta **NUMA‑aware** paigutust  
- Kuberneteses kasuta **SR‑IOV Network Operatorit**  
- Kontrolli MTU‑d (DPDK + SR‑IOV vajavad tihti suuremat MTU‑d)  
- Ära sega SR‑IOV‑i ja virtuaalseid switche (OVS) samal VF‑il  

---

## Kokkuvõte

SR‑IOV võimaldab NIC‑i jagada mitmeks virtuaalseks funktsiooniks, pakkudes VM‑idele ja Pod‑idele otsest, kõrge jõudlusega ligipääsu võrguriistvarale.  
See on kriitiline tehnoloogia Telco, NFV ja HPC maailmas.
