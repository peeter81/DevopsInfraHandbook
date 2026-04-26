# networking/conntrack.md  
## conntrack – ühenduste jälgimise süsteem

`conntrack` on Linuxi kernelis olev **ühenduste jälgimise süsteem**, mis on osa Netfilterist.  
See jälgib kõiki ühendusi ja nende olekuid, mis on vajalik:

- NAT toimimiseks  
- stateful firewalli reeglite jaoks  
- ühenduste statistika jaoks  
- DDoS kaitseks  
- debugginguks  

Kõik modernsed firewallid (iptables, nftables) kasutavad conntrack’i.

---

## 1. Ühenduste olekud

conntrack jälgib iga ühenduse **state** väärtust:

- **NEW** – uus ühendus  
- **ESTABLISHED** – ühendus on loodud  
- **RELATED** – seotud ühendus (nt FTP data channel)  
- **INVALID** – vigane pakett  
- **UNTRACKED** – conntrack ei jälgi seda paketti  

Firewalli reeglites kasutatakse neid:

```
ct state new,established,related accept
```

---

## 2. Ühenduste vaatamine

### Kõik ühendused
```
conntrack -L
```

### Filtreerimine protokolli järgi
```
conntrack -L -p tcp
conntrack -L -p udp
```

### Filtreerimine IP järgi
```
conntrack -L | grep 192.168.1.10
```

---

## 3. Reaalajas sündmused

```
conntrack -E
```

Näed sündmusi:

- NEW  
- DESTROY  
- ASSURED  
- TIMEOUT  

Väga kasulik NAT ja firewalli probleemide debugimisel.

---

## 4. Tabeli tühjendamine

⚠️ Ohtlik – katkestab kõik ühendused.

```
conntrack -F
```

---

## 5. Ühenduste arv ja limiidid

### Aktiivsete ühenduste arv
```
cat /proc/sys/net/netfilter/nf_conntrack_count
```

### Maksimaalne lubatud arv
```
cat /proc/sys/net/netfilter/nf_conntrack_max
```

### Limite suurendamine
```
sysctl -w net.netfilter.nf_conntrack_max=262144
```

---

## 6. NAT ja conntrack

NAT **ei tööta ilma conntrackita**.

### DNAT – conntrack loob kirje:
```
original: 1.2.3.4:80 → 10.0.0.5:8080
reply:    10.0.0.5:8080 → 1.2.3.4:80
```

### SNAT – conntrack mäletab:
```
original: 10.0.0.5:12345 → 8.8.8.8:53
reply:    8.8.8.8:53 → 203.0.113.10:12345
```

---

## 7. conntrack + NAT + firewall diagramm

```
Incoming packet
        |
        v
+-----------------------+
|     PREROUTING        |  (DNAT, conntrack NEW)
+-----------------------+
        |
        v
+-----------------------+
|       ROUTING         |
+-----------------------+
     |             |
     v             v
  INPUT         FORWARD
     |             |
     v             v
 PROCESS     POSTROUTING (SNAT, conntrack ASSURED)
```

---

## 8. Debugimine

### Näha, kas NAT töötab
```
conntrack -L -p tcp | grep dport=80
```

### Näha, kas ühendus on ESTABLISHED
```
conntrack -L | grep ESTABLISHED
```

### Näha, kas ühendus aegub
```
conntrack -E | grep TIMEOUT
```

---

## 9. Best Practices

- Suurenda `nf_conntrack_max` suure liiklusega serverites  
- Kasuta firewallis `ct state established,related`  
- Ära kasuta `conntrack -F` prod-masinas  
- Debugi NAT probleeme `conntrack -E` abil  
- Kasuta nftables’i, mitte iptables’i uutes projektides  

---

## Kokkuvõte

conntrack on Linuxi võrgupinu kriitiline komponent.  
Ilma selleta ei töötaks NAT, stateful firewall ega enamik kaasaegseid võrgulahendusi.  
Selle mõistmine on hädavajalik DevOps/SRE töö jaoks.

---

### Kui soovid, võin selle faili:

- muuta veel kompaktsemaks  
- lisada näiteid nftables’i `ct` matchidega  
- lisada ASCII diagramme  
- lisada peatüki conntrack timeoutidest  
