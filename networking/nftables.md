# networking/nftables.md  
## nftables – Networking käsiraamat

## Ülevaade

nftables on Linuxi uuem ja soovitatud firewalli raamistik, mis asendab:

- iptables  
- ip6tables  
- arptables  
- ebtables  

Eelised:

- ühtne süntaks IPv4/IPv6 jaoks  
- kiirem ja efektiivsem  
- lihtsam ja loetavam reeglistik  
- atomaarne reeglite uuendamine  
- võimalus kasutada **maps**, **sets**, **concatenations**

---

# 1. Reeglistiku vaatamine

```bash
nft list ruleset
```

Näitab kõiki tabeleid, chaine ja reegleid.

---

# 2. Tabeli loomine

```bash
nft add table inet filter
```

**inet** = töötab nii IPv4 kui IPv6 jaoks.

---

# 3. Chain’i loomine

```bash
nft add chain inet filter input { type filter hook input priority 0; policy drop; }
```

Selgitus:

- **type filter** – filtritüüp  
- **hook input** – seotakse INPUT hook’iga  
- **priority 0** – täitmisjärjekord  
- **policy drop** – vaikimisi kõik keelatud  

---

# 4. Reeglite lisamine

## 4.1 Lubame established/related ühendused

```bash
nft add rule inet filter input ct state established,related accept
```

## 4.2 Lubame loopback

```bash
nft add rule inet filter input iif "lo" accept
```

## 4.3 Lubame SSH

```bash
nft add rule inet filter input tcp dport 22 accept
```

## 4.4 Logimine

```bash
nft add rule inet filter input log prefix "DROP INPUT: "
```

---

# 5. NAT näited

## 5.1 NAT tabel

```bash
nft add table ip nat
```

## 5.2 PREROUTING chain (DNAT)

```bash
nft add chain ip nat prerouting { type nat hook prerouting priority -100; }
```

## 5.3 POSTROUTING chain (SNAT/MASQ)

```bash
nft add chain ip nat postrouting { type nat hook postrouting priority 100; }
```

## 5.4 DNAT (port forwarding)

```bash
nft add rule ip nat prerouting tcp dport 80 dnat to 192.168.1.10:8080
```

## 5.5 MASQUERADE

```bash
nft add rule ip nat postrouting oif "eth0" masquerade
```

---

# 6. Sets (väga võimas funktsioon)

## 6.1 IP-aadresside komplekt

```bash
nft add set inet filter blacklist { type ipv4_addr; }
nft add element inet filter blacklist { 10.0.0.5, 10.0.0.6 }
```

## 6.2 Reegel, mis kasutab set’i

```bash
nft add rule inet filter input ip saddr @blacklist drop
```

---

# 7. Maps (port ? tegevus)

## Map loomine

```bash
nft add map inet filter allowed_ports { type inet_service : verdict; }
nft add element inet filter allowed_ports { 22 : accept, 80 : accept, 443 : accept }
```

## Reegel, mis kasutab map’i

```bash
nft add rule inet filter input tcp dport vmap @allowed_ports
```

---

# Kokkuvõte

nftables on kaasaegne, kiire ja paindlik firewalli raamistik, mis asendab iptables’i.  
See toetab sets, maps, atomaarseid uuendusi ja ühtset süntaksit IPv4/IPv6 jaoks.

