# 📄 **Fail 14: `networking/xdp.md`**

```markdown
# XDP – Networking käsiraamat  
## (eXpress Data Path, eBPF dataplane)

## Ülevaade
XDP on Linuxi kernelis olev **ülikiire paketitöötluse mehhanism**, mis töötab enne netfilterit, enne iptables’i ja isegi enne kernel networking stack’i.

XDP võimaldab:
- töödelda pakette NIC driveri tasemel
- saavutada 20–40 Mpps (miljonit paketti sekundis)
- luua firewall’e, load balancereid, DDoS kaitset
- kasutada eBPF programme

XDP on **kiirem kui iptables, nftables, OVS ja isegi DPDK** mõnes kasutusstsenaariumis.

---

# 1. XDP töörežiimid

## 1.1 XDP native mode
- töötab NIC driveri sees  
- maksimaalne jõudlus  
- vajab NIC driveri tuge

## 1.2 XDP generic mode
- töötab kernel networking stack’i peal  
- aeglasem, kuid töötab igal NIC’il

## 1.3 XDP offload mode
- programm jookseb NIC‑i riistvaras  
- väga kiire  
- vajab spetsiaalset NIC‑i (nt Mellanox)

---

# 2. XDP programmi struktuur (eBPF)

Lihtne XDP drop programm:

```c
SEC("xdp")
int xdp_drop(struct xdp_md *ctx) {
    return XDP_DROP;
}
```

Tagastuskoodid:
- **XDP_DROP** – pakett visatakse ära
- **XDP_PASS** – pakett läheb kernelisse edasi
- **XDP_TX** – pakett saadetakse tagasi samale NIC‑ile
- **XDP_REDIRECT** – pakett suunatakse teisele NIC‑ile või AF_XDP socketisse

---

# 3. XDP programmi kompileerimine

```bash
clang -O2 -target bpf -c xdp_prog.c -o xdp_prog.o
```

---

# 4. XDP programmi laadimine NIC‑ile

```bash
ip link set dev eth0 xdp obj xdp_prog.o sec xdp
```

Eemaldamine:

```bash
ip link set dev eth0 xdp off
```

---

# 5. XDP + eBPF maps

XDP kasutab **maps** struktuure, et salvestada:
- loendureid
- IP‑aadresside tabeleid
- musta- ja valgeliste
- load balancing backend’e

Näide map’i defineerimisest:

```c
struct bpf_map_def SEC("maps") blacklist = {
    .type = BPF_MAP_TYPE_HASH,
    .key_size = sizeof(__u32),
    .value_size = sizeof(__u32),
    .max_entries = 1024,
};
```

---

# 6. XDP + AF_XDP (user‑space zero‑copy)

AF_XDP võimaldab:
- suunata XDP pakette user‑space’i
- ilma kernel networking stack’ita
- ilma DPDK‑ta

See annab:
- väga madala latentsuse
- väga kõrge läbilaskevõime

---

# 7. XDP kasutusvaldkonnad

## 7.1 DDoS kaitse
- XDP_DROP suudab ära visata miljoneid pakette sekundis

## 7.2 Load balancer
- XDP_REDIRECT → teisele NIC‑ile või queue’le

## 7.3 Firewall
- IP mustad nimekirjad
- Portide filtreerimine

## 7.4 Telemeetria
- eBPF maps → Prometheus eksport

## 7.5 Kubernetes CNI
- Cilium kasutab XDP’d dataplane’is

---

# 8. XDP paketivoo diagramm

```
Incoming packet
      |
      v
+---------------------------+
|       XDP Program         |  (NIC driver level)
+---------------------------+
   |      |       |      |
   v      v       v      v
DROP    PASS     TX   REDIRECT
```

---

# 9. XDP tööriistad

## 9.1 bpftool

```bash
bpftool prog show
bpftool map show
```

## 9.2 xdp-loader (xdp-tools)

```bash
xdp-loader load -m skb -s xdp xdp_prog.o eth0
```

## 9.3 xdpdump (tcpdump XDP tasemel)

```bash
xdpdump -i eth0
```

---

# 10. Best Practices

- Kasuta **native mode** kui NIC toetab
- Kasuta **maps** struktuure dünaamiliste reeglite jaoks
- Ära tee XDP programmis liiga palju loogikat (hoia kiire)
- Kasuta **bpftool** debugimiseks
- Kuberneteses eelista **Cilium** (kasutab XDP’d ja eBPF’i)

---

# Kokkuvõte
XDP on Linuxi kõige kiirem dataplane tehnoloogia.  
See töötab NIC driveri tasemel, võimaldades luua ülikiireid firewall’e, load balancereid ja DDoS kaitset.  
XDP + eBPF on kaasaegse võrgumaailma üks võimsamaid tööriistu.
```
