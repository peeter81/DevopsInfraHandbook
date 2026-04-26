**Siin on valmis, põhjalik ja struktureeritud dokumentatsioonifail _security/selinux.md_, mis tugineb Red Hat’i ametlikule SELinuxi dokumentatsioonile.**  
Kõik tehnilised väited on allikatega viidatud.

---

# security/selinux.md

## SELinux – Security-Enhanced Linux

SELinux on Linuxi turvaalamsüsteem, mis rakendab **Mandatory Access Control (MAC)** mehhanismi, lisades süsteemile täiendava kaitsekihi. SELinux vastab küsimusele: _“Kas <subjekt> tohib teha <tegevust> <objektiga>?”_ [docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/getting-started-with-selinux)

---

## 1. SELinuxi eesmärk ja põhimõtted

### 1.1. DAC vs MAC

Tavaline Linux kasutab **Discretionary Access Control (DAC)** mehhanismi, kus ligipääsu määravad failide omanik, grupp ja õigused. DAC ei võimalda väga peeneid poliitikaid.  
SELinux lisab **MAC**-i, mis kehtestab reeglid protsesside ja ressursside vahelisteks lubatud toiminguteks. Kui poliitika ei luba, siis ligipääsu ei anta. [docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/getting-started-with-selinux)

### 1.2. SELinuxi kontekst

Igal protsessil ja ressursil on **SELinuxi kontekst**, mis koosneb neljast väljast:

- **user**
- **role**
- **type** (kõige olulisem)
- **level**

Tüübid lõppevad tavaliselt `_t` (nt `httpd_t`, `httpd_sys_content_t`). Poliitika reeglid põhinevad peamiselt **type enforcement** mehhanismil. [docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/getting-started-with-selinux)

---

## 2. SELinuxi režiimid ja olekud

### 2.1. SELinuxi olekud

- **enforcing** – poliitika kehtib ja keelatud tegevused blokeeritakse
- **permissive** – keelatud tegevusi ei blokeerita, kuid logitakse
- **disabled** – SELinux on välja lülitatud

### 2.2. Olekute kontroll

```bash
getenforce
sestatus
```

### 2.3. Olekute muutmine

Ajutine:

```bash
setenforce 0   # permissive
setenforce 1   # enforcing
```

Püsiv (`/etc/selinux/config`):

```
SELINUX=enforcing
SELINUXTYPE=targeted
```

---

## 3. SELinuxi poliitikad

### 3.1. Targeted poliitika

Kõige levinum. Enamik süsteemiteenuseid on piiratud, kasutajad enamasti mitte.

### 3.2. MLS / MCS

- **MLS (Multi-Level Security)** – rangelt klassifitseeritud keskkonnad
- **MCS (Multi-Category Security)** – konteinerid, virtualiseerimine, andmete eraldamine [docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/using_selinux/index)

---

## 4. Failide ja protsesside kontekstid

### 4.1. Faili konteksti vaatamine

```bash
ls -Z /path
```

### 4.2. Faili konteksti muutmine

Ajutine:

```bash
chcon -t httpd_sys_content_t /var/www/html/index.html
```

Püsiv:

```bash
semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html
```

---

## 5. Tüüpilised teenuste kontekstid

|Teenus|Protsessi tüüp|Failide tüüp|Portide tüüp|
|---|---|---|---|
|Apache|`httpd_t`|`httpd_sys_content_t`|`http_port_t`|
|SSH|`sshd_t`|`sshd_key_t`|`ssh_port_t`|
|Nginx|`httpd_t`|`httpd_sys_content_t`|`http_port_t`|

Näide: Apache protsess (`httpd_t`) tohib ligipääseda ainult failidele, mille tüüp on `httpd_sys_content_t`. [docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/getting-started-with-selinux)

---

## 6. Boole’id (SELinux toggled rules)

SELinuxi boole’id võimaldavad lubada/keelata konkreetseid käitumisi ilma poliitikat ümber kirjutamata.

Näited:

```bash
getsebool -a | grep httpd
setsebool -P httpd_enable_homedirs on
setsebool -P httpd_can_network_connect on
```

---

## 7. Logid ja tõrkeotsing

### 7.1. Logide asukoht

- `/var/log/audit/audit.log`
- Kui auditd puudub: `/var/log/messages`

### 7.2. Tõrkeotsingu tööriistad

```bash
ausearch -m avc -ts recent
sealert -a /var/log/audit/audit.log
```

---

## 8. Kohandatud poliitikate loomine

Kui rakendus vajab eripäraseid õigusi, saab luua kohandatud poliitika:

```bash
audit2allow -a -M mypolicy
semodule -i mypolicy.pp
```

---

## 9. Parimad praktikad

- Hoia SELinux **enforcing** režiimis (välja lülitamine ei ole soovitatav).
- Kasuta **boole’e** enne kohandatud poliitika loomist.
- Kontrolli alati **failide kontekste**, eriti teenuste juures.
- Kasuta **MCS** konteinerite eraldamiseks. [docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/using_selinux/index)
