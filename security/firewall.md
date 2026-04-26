# 📄 **Fail: `security/firewall.md`**

```markdown
# Linux Firewall – Security käsiraamat  
## (firewalld, nftables, iptables, zones, services, rich rules)

## Ülevaade
Linuxi tulemüürid põhinevad kernelis oleval **netfilter** raamistikul.  
Kõige levinumad kasutajaliidesed:

- **firewalld** (RHEL, Rocky, Alma, Fedora – vaikimisi)
- **nftables** (uus standard, asendab iptables’i)
- **iptables** (vana, kuid endiselt kasutusel)

Firewalld kasutab sisemiselt **nftables**-i.

---

# 1. Firewalld

Firewalld kasutab **zone**-põhist mudelit.

Vaata olekut:

```bash
systemctl status firewalld
```

Vaata aktiivset zone’i:

```bash
firewall-cmd --get-active-zones
```

---

# 2. Zones

Levinud zone’id:

| Zone | Kirjeldus |
|------|-----------|
| public | vaikimisi, avatud ainult valitud teenused |
| home | lubab rohkem teenuseid |
| internal | usaldatud võrk |
| dmz | piiratud teenused |
| drop | kõik paketid visatakse |
| block | kõik paketid blokitakse ICMP teatega |

Vaheta zone:

```bash
firewall-cmd --set-default-zone=public
```

---

# 3. Teenuste lubamine

Firewalld kasutab teenuse profiile:

```bash
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
```

Vaata teenuseid:

```bash
firewall-cmd --list-services
```

---

# 4. Portide lubamine

```bash
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

---

# 5. Rich rules (täpsemad reeglid)

Näide: luba SSH ainult kindlast IP-st:

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.5" service name="ssh" accept'
firewall-cmd --reload
```

Blokeeri IP:

```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="1.2.3.4" drop'
```

---

# 6. Masquerading (NAT)

```bash
firewall-cmd --add-masquerade --permanent
```

---

# 7. Forwarding

```bash
firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080 --permanent
```

---

# 8. nftables (uus standard)

Vaata tabelid:

```bash
nft list tables
```

Näita reegleid:

```bash
nft list ruleset
```

Näide reeglist:

```bash
nft add rule inet filter input tcp dport 22 accept
```

---

# 9. iptables (vana, kuid endiselt kasutusel)

Vaata reegleid:

```bash
iptables -L -n -v
```

Näide:

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

---

# 10. Firewalld vs nftables vs iptables

| Lahendus | Staatus | Märkused |
|----------|---------|----------|
| firewalld | vaikimisi RHEL/Fedora | kasutab nftables’i |
| nftables | uus standard | soovitatav uutes süsteemides |
| iptables | legacy | asendatakse järk-järgult |

---

# 11. Firewalld debugging

Vaata logisid:

```bash
journalctl -u firewalld
```

Testi reegleid:

```bash
firewall-cmd --query-port=22/tcp
```

---

# 12. Best Practices

- Kasuta **firewalld** RHEL-põhistes süsteemides  
- Kasuta **nftables** uutes projektides  
- Ära kasuta iptables’i uutes süsteemides  
- Piira SSH ligipääsu IP-põhiselt  
- Kasuta zone’e erinevate võrkude jaoks  
- Kasuta rich rules täpsemaks kontrolliks  
- Luba ainult vajalikud teenused ja pordid  

---

# Kokkuvõte
Linuxi tulemüürid põhinevad netfilteril, kuid kasutajaliidesed varieeruvad.  
Firewalld on mugav ja zone-põhine, nftables on uus standard, iptables on legacy.  
Õige konfiguratsioon tagab turvalise ja kontrollitud võrguliikluse.
```

