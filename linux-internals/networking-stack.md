# 📄 **Fail 5: `linux-internals/networking-stack.md`**

```markdown
# Linux Networking Stack – Linux Internals käsiraamat  
## (OSI vs Linux stack, NIC → Kernel → Sockets → Apps)

## Ülevaade
Linuxi võrgupinu on kihiline arhitektuur, mis töötleb pakette teekonnal:

**NIC → Kernel → Netfilter → Routing → Socket → Application**

See on üks maailma kõige paindlikumaid ja võimsamaid võrgustikke, mida kasutavad:
- serverid  
- pilveplatvormid  
- Kubernetes  
- SDN  
- Telco  

---

# 1. OSI mudel vs Linux networking stack

| OSI kiht | Linux komponent |
|---------|-----------------|
| L1 – Physical | NIC, PHY |
| L2 – Data Link | MAC, Ethernet driver |
| L3 – Network | IP stack, routing |
| L4 – Transport | TCP, UDP, SCTP |
| L5–7 – Session/App | Sockets, libraries, applications |

Linux ei järgi OSI mudelit 1:1, kuid funktsioonid on samad.

---

# 2. Paketivoo teekond (RX – incoming)

```
NIC → Driver → XDP → Netfilter PREROUTING → Routing → 
  → INPUT (local) → Socket → Application
  → FORWARD (routing) → POSTROUTING → NIC
```

Selgitus:

- **NIC** – võtab paketi vastu  
- **Driver** – annab kernelile  
- **XDP** – ultra‑kiire eBPF dataplane  
- **Netfilter** – firewall, NAT, conntrack  
- **Routing** – otsustab, kuhu pakett läheb  
- **Socket** – rakenduse ühendus  
- **Application** – nginx, sshd, curl jne  

---

# 3. Paketivoo teekond (TX – outgoing)

```
Application → Socket → Kernel → Routing → Netfilter POSTROUTING → NIC
```

---

# 4. Netfilter hook’id

Linuxi firewall ja NAT töötavad järgmiste hook’ide kaudu:

- **PREROUTING** – DNAT  
- **INPUT** – kohalikule masinale  
- **FORWARD** – läbi masina  
- **OUTPUT** – kohalikud paketid  
- **POSTROUTING** – SNAT  

---

# 5. Routing

Routing otsustab, kuhu pakett läheb.

Vaata routing tabelit:

```
ip route show
```

Lisa route:

```
ip route add 10.10.0.0/16 via 192.168.1.1
```

---

# 6. TCP/IP stack

Linuxi TCP/IP stack sisaldab:

- ARP  
- IPv4 / IPv6  
- ICMP  
- TCP  
- UDP  
- SCTP  

Vaata ühendusi:

```
ss -tuna
```

---

# 7. Socketid

Rakendused kasutavad socket’e:

- TCP socket  
- UDP socket  
- RAW socket  
- UNIX domain socket  

Vaata protsessi socket’e:

```
ls -l /proc/<pid>/fd
```

---

# 8. NIC queue’d ja IRQ’d

Modernsed NIC‑id kasutavad:

- **RSS** (Receive Side Scaling)  
- **Multiple RX/TX queues**  
- **IRQ pinning**  

Vaata IRQ infot:

```
cat /proc/interrupts
```

---

# 9. eBPF ja XDP

Linuxi võrgupinu saab laiendada eBPF programmidega:

- XDP (NIC driveri tasemel)  
- TC eBPF (traffic control)  
- Socket eBPF (SOCKMAP, SKB)  

Vaata eBPF programme:

```
bpftool prog show
```

---

# 10. Kubernetes ja Linux networking stack

Kubernetes kasutab Linuxi võrgupinu:

- CNI pluginad (Calico, Cilium, Flannel)  
- iptables / nftables  
- VXLAN / Geneve  
- eBPF dataplane (Cilium)  
- conntrack  

Kõik podide võrgud töötavad Linuxi kernelis.

---

# 11. Best Practices

- Kasuta `ss` TCP ühenduste diagnoosimiseks  
- Kasuta `tcpdump` paketipüüdmise jaoks  
- Kasuta `bpftool` eBPF programmide kontrollimiseks  
- Ära kasuta iptables’i uutes projektides → eelista nftables  
- Kasuta XDP’d jõudluskriitilistes lahendustes  
- Kontrolli IRQ ja NIC queue’de tasakaalu (RSS)  

---

# Kokkuvõte
Linuxi võrgupinu on võimas ja paindlik arhitektuur, mis võimaldab ehitada servereid, pilveplatvorme, Kubernetes klastri võrke ja Telco taseme lahendusi.  
NIC → Kernel → Netfilter → Routing → Socket → Application teekonna mõistmine on kriitiline DevOps/SRE töö jaoks.
```
