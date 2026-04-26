**Siin on täielik ja korrektselt allikatega viidatud dokumentatsioonifail *network/netfilter.md*. See põhineb Linuxi Netfilteri ametlikul dokumentatsioonil ja Wikipedia tehnilistel kirjeldustel.**
# 📄 **Fail 1x: `networking/netfilter.md`**

## Netfilter – Linuxi kernelitaseme paketitöötluse raamistik

**Netfilter on Linuxi kerneli sisemine raamistik, mis võimaldab paketifiltreerimist, NAT-i, logimist ja paketitöötlust läbi erinevate hook’ide.**  
See on alus nii *iptables*, *nftables* kui ka *conntrack* süsteemidele.  
  [The netfilter.org project](https://netfilter.org/?ref=linux-handbook)

---

## 1. Netfilteri roll Linuxi võrgupinu sees

Netfilter lisab kernelisse **hook-punktid**, mille külge saavad moodulid registreerida callback-funktsioone. Iga pakett, mis läbib võrgupinu, käivitatakse vastava hook’i juures.  
  [The netfilter.org project](https://netfilter.org/?ref=linux-handbook)

### 1.1. Peamised hook’id (IPv4/IPv6)
- **PREROUTING** – enne ruutimist  
- **INPUT** – pakett, mis on suunatud lokaalsele masinale  
- **FORWARD** – pakett, mis läbib masinat  
- **OUTPUT** – lokaalne protsess saadab paketi välja  
- **POSTROUTING** – pärast ruutimist, enne väljumist  

Need hook’id võimaldavad rakendada filtrit, NAT-i ja muud paketitöötlust.  
  [Wikipedia](https://en.wikipedia.org/wiki/Netfilter)

---

## 2. Netfilteri funktsionaalsus

Netfilter toetab järgmisi funktsioone:

### 2.1. Paketifiltreerimine  
- **stateless filtering** (IPv4/IPv6)  
- **stateful filtering** koos *conntrack* süsteemiga  
  [The netfilter.org project](https://netfilter.org/?ref=linux-handbook)

### 2.2. NAT (Network Address Translation)  
- SNAT  
- DNAT  
- Masquerading  
- Port forwarding  
  [The netfilter.org project](https://netfilter.org/?ref=linux-handbook)

### 2.3. Paketilogimine  
- logimine kernelisse  
- ulogd kaudu kasutajaruumi logimine  

### 2.4. Paketijärjekorrad (NFQUEUE)  
Võimaldab saata pakette kasutajaruumi töötlemiseks.  
  [The netfilter.org project](https://netfilter.org/?ref=linux-handbook)

### 2.5. Paketimanipulatsioon  
- märgistamine  
- TTL muutmine  
- muud madalama taseme modifikatsioonid  

---

## 3. Netfilteri komponendid

### 3.1. iptables  
Ajalooline kasutajaruumi tööriist, mis manipuleerib Netfilteri tabelitega.  
  [The netfilter.org project](https://netfilter.org/?ref=linux-handbook)

### 3.2. nftables  
iptables’i järeltulija, mis pakub:
- paremat jõudlust  
- ühtset käsusüntaksit IPv4/IPv6 jaoks  
- kompaktsemaid reegleid  
  [The netfilter.org project](https://netfilter.org/?ref=linux-handbook)

### 3.3. conntrack  
Netfilteri ühenduste jälgimise süsteem, mis võimaldab stateful filtrit.  
  [The netfilter.org project](https://netfilter.org/documentation/index.html)

### 3.4. libnetfilter-* teegid  
Netfilteri alamkomponendid:
- libnetfilter_conntrack  
- libnetfilter_queue  
- libnftnl  
- libnfnetlink  
  [The netfilter.org project](https://netfilter.org/?ref=linux-handbook)

---

## 4. Netfilteri arhitektuur

### 4.1. Tabelid ja chain’id  
Netfilter kasutab tabelite ja chain’ide süsteemi:

| Tabel | Eesmärk |
|-------|---------|
| **filter** | Paketifiltreerimine |
| **nat** | NAT operatsioonid |
| **mangle** | Paketimanipulatsioon |
| **raw** | Conntracki vältimine |
| **security** | LSM integratsioon |

### 4.2. Reegli struktuur  
Iga reegel koosneb:
- **match** (tingimused)  
- **target** (tegevus: ACCEPT, DROP, DNAT, SNAT jne)  

  [The netfilter.org project](https://netfilter.org/?ref=linux-handbook)

---

## 5. Netfilteri kasutusjuhtumid

- Linuxi tulemüürid (iptables, nftables)  
- NAT ruuterid  
- VPN gateway’d  
- Kubernetes kube-proxy (iptables/nftables backend)  
- IDS/IPS süsteemid NFQUEUE kaudu  

---

## 6. Netfilteri areng ja ajalugu

- Projekt algas 1998 (Rusty Russell).  
- Netfilter lisati Linuxi kernelisse versioonis 2.3.15 (1999).  
- ipchains ja ipfwadm asendati Netfilteriga.  
  [Wikipedia](https://en.wikipedia.org/wiki/Netfilter)

---

## 7. Parimad praktikad

- Kasuta **nftables** uute konfiguratsioonide jaoks.  
- Vähenda reeglite arvu – iga reegel mõjutab jõudlust.  
- Kasuta **sets** (nftables) suure hulga IP-de haldamiseks.  
- Logi ainult vajalikku (väldi logi üleujutamist).  
- Kasuta **conntrack** stateful filtriks.  
