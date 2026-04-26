**Netfilter hook’id on Linuxi kerneli kontrollpunktid, mille kaudu iga pakett läbib kindla töötlusetapi — need on PREROUTING, INPUT, FORWARD, OUTPUT ja POSTROUTING.** Need võimaldavad rakendada filtrit, NAT-i, logimist ja muid paketitöötluse mehhanisme.   [Oracle Blogs](https://blogs.oracle.com/linux/introduction-to-netfilter)

---

# network/netfilter-hooks.md  
## Netfilter Hooks – Linuxi paketitöötluse kontrollpunktid

Netfilter hook’id on **Linuxi kerneli sisemised kontrollpunktid**, mille külge saab registreerida callback‑funktsioone, et töödelda pakette erinevates võrgupinu etappides. Need moodustavad aluse iptables’ile, nftables’ile ja conntrack’ile.  
Hook’id võimaldavad rakendada **filtreerimist, NAT-i, logimist, queue’ingut ja paketimanipulatsiooni**.   [The netfilter.org project](https://www.nftables.org/index.html)

---

## 1. Netfilteri viis põhihook’i (IPv4/IPv6)

Linuxi võrgupinu töötleb pakette järgmiste hook’ide kaudu:   [Oracle Blogs](https://blogs.oracle.com/linux/introduction-to-netfilter)

### **1. PREROUTING (NF_IP_PRE_ROUTING)**  
- Käivitub kohe pärast paketi saabumist.  
- Toimub enne ruutimisotsust.  
- Kasutatakse DNAT-i, märgistamist, varajast filtrit.

### **2. INPUT (NF_IP_LOCAL_IN)**  
- Pakett on suunatud kohalikule masinale.  
- Toimub pärast ruutimist.  
- Kasutatakse lokaalse teenuse kaitsmiseks.

### **3. FORWARD (NF_IP_FORWARD)**  
- Pakett ei ole mõeldud kohalikule masinale.  
- Kasutatakse ruuterites ja gateway’de puhul.  
- Võimaldab stateful forwarding filtrit.

### **4. OUTPUT (NF_IP_LOCAL_OUT)**  
- Kohalik protsess loob paketi.  
- Esimene punkt, kus väljaminevat paketti saab muuta.  
- Kasutatakse SNAT-i, filtrit, märgistamist.

### **5. POSTROUTING (NF_IP_POST_ROUTING)**  
- Viimane etapp enne paketi väljumist liidesest.  
- Kasutatakse SNAT-i, MASQUERADE’i ja lõplikku paketimanipulatsiooni.

---

## 2. Ingress hook (varajane filtratsioon)

Linux kernel 4.2 lisas **ingress hook’i**, mis töötab *enne* PREROUTING hook’i ja on seotud konkreetse võrguliidesega.  
See võimaldab väga varajast filtrit — enne fragmentide kokkupanekut.  
- Sobib DoS kaitseks ja varajaseks droppimiseks.  
- Ei võimalda L4 päiste täielikku analüüsi fragmentide tõttu.   [wiki.netfilter.org](https://wiki.netfilter.org/wiki-nftables/index.php/Netfilter_hooks)

---

## 3. Hook’ide töövoog Linuxi võrgupinus

### **Sissetulev pakett (destineeritud kohalikule masinale)**  
1. PREROUTING  
2. INPUT  

### **Sissetulev pakett (forwarding)**  
1. PREROUTING  
2. FORWARD  
3. POSTROUTING  

### **Väljaminev pakett (kohalik protsess)**  
1. OUTPUT  
2. POSTROUTING  

Need etapid määravad, millises hook’is saab rakendada NAT-i, filtrit või queue’ingut.   [wiki.netfilter.org](https://wiki.netfilter.org/wiki-nftables/index.php/Netfilter_hooks)

---

## 4. Hook’ide prioriteedid

Igal hook’il võib olla mitu callback’i.  
Netfilter täidab need **tõusvas prioriteedijärjekorras** — väiksem number tähendab varasemat täitmist.  
Näiteks nftables võimaldab määrata base‑chain’i prioriteedi, et kontrollida reeglite täpset täitmisjärjekorda.   [wiki.netfilter.org](https://wiki.netfilter.org/wiki-nftables/index.php/Netfilter_hooks)

---

## 5. Hook’ide kasutusjuhtumid

- **NAT**:  
  - DNAT → PREROUTING  
  - SNAT/MASQUERADE → POSTROUTING  
- **Firewall**:  
  - INPUT → kaitseb hosti  
  - FORWARD → kaitseb ruuterit  
  - OUTPUT → kontrollib lokaalseid protsesse  
- **IDS/IPS**:  
  - NFQUEUE → PREROUTING/INPUT/OUTPUT  
- **QoS / shaping**:  
  - ingress hook → varajane droppimine  
  - tc → shaping  

---

## 6. Kokkuvõte

Netfilter hook’id on Linuxi võrgupinu kriitiline osa, mis võimaldab paindlikku ja jõulist paketitöötlust. Need moodustavad aluse iptables’ile, nftables’ile, conntrack’ile ja paljudele turva- ning võrgulahendustele.
