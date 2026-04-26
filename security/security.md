# 📄 **Fail: `security/selinux.md`**

```markdown
# SELinux – Security käsiraamat  
## (Modes, contexts, booleans, policies, troubleshooting)

## Ülevaade
SELinux (Security-Enhanced Linux) on **Mandatory Access Control (MAC)** süsteem, mis lisab Linuxile tugeva turvakihi.  
Iga protsess ja fail saab **turvakonteksti**, mis määrab, mida see protsess tohib teha.

SELinuxi komponendid:
- **policies** (reeglid)
- **contexts** (failide ja protsesside sildid)
- **booleans** (lülitatavad reeglid)
- **audit logid** (AVC denials)
- **modes** (enforcing, permissive, disabled)

---

# 1. SELinux režiimid

Vaata režiimi:

```bash
getenforce
```

Muuda ajutiselt:

```bash
setenforce 0   # permissive
setenforce 1   # enforcing
```

Püsiv seadistus:

```
/etc/selinux/config
```

Režiimid:
- **enforcing** – reeglid kehtivad  
- **permissive** – logib rikkumised, ei blokeeri  
- **disabled** – välja lülitatud  

---

# 2. SELinux kontekstid

Igal failil ja protsessil on kontekst:

```bash
ls -Z
ps -eZ
```

Näide kontekstist:

```
system_u:object_r:httpd_sys_content_t:s0
```

Konteksti komponendid:
- **user**  
- **role**  
- **type** (kõige olulisem)  
- **level**  

---

# 3. Failide konteksti muutmine

Ajutine muutmine:

```bash
chcon -t httpd_sys_content_t /var/www/html/index.html
```

Püsiv muutmine (policy reegel):

```bash
semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html
```

---

# 4. SELinux booleans

Booleans lubavad või keelavad teatud käitumise.

Näita kõiki:

```bash
getsebool -a
```

Näide: lubada Apache’l võrku kasutada:

```bash
setsebool -P httpd_can_network_connect on
```

---

# 5. SELinux logid (AVC denials)

Kõik rikkumised logitakse:

```
/var/log/audit/audit.log
```

Leia viimased rikkumised:

```bash
ausearch -m avc
```

Või:

```bash
journalctl -t setroubleshoot
```

---

# 6. setroubleshoot (diagnoos)

Installi:

```bash
yum install setroubleshoot
```

Analüüsi rikkumist:

```bash
sealert -a /var/log/audit/audit.log
```

See annab:
- põhjuse  
- soovitatud lahenduse  
- vajaliku booleani või fcontext reegli  

---

# 7. SELinux policy tüübid

Levinud policy’d:
- **targeted** (ainult teenused on piiratud)  
- **mls** (kõrge turvatase, harva kasutatav)  

Vaata:

```bash
sestatus
```

---

# 8. SELinux ja teenused

## 8.1 Apache

Lubada HTTPD-l võrku kasutada:

```bash
setsebool -P httpd_can_network_connect on
```

Lubada HTTPD-l lugeda teist kataloogi:

```bash
semanage fcontext -a -t httpd_sys_content_t "/data/www(/.*)?"
restorecon -Rv /data/www
```

## 8.2 NFS

```bash
setsebool -P nfs_export_all_rw on
```

## 8.3 Docker / Podman

```bash
setsebool -P container_manage_cgroup on
```

---

# 9. SELinux troubleshooting flow

1. Kontrolli režiimi  
   ```
   getenforce
   ```

2. Vaata audit logi  
   ```
   ausearch -m avc
   ```

3. Kasuta sealert  
   ```
   sealert -a /var/log/audit/audit.log
   ```

4. Rakenda soovitatud boolean või fcontext  
   ```
   setsebool -P ...
   semanage fcontext ...
   restorecon -Rv ...
   ```

5. Ära lülita SELinux’i välja  
   (see on halb praktika)

---

# 10. Best Practices

- Hoia SELinux **enforcing** režiimis  
- Kasuta `semanage fcontext` püsivate reeglite jaoks  
- Ära kasuta `chcon` püsivaks lahenduseks  
- Kasuta `setroubleshoot` probleemide diagnoosimiseks  
- Ära keela SELinux’i, kui teenus ei tööta — lahenda kontekstid  
- Kasuta booleane ainult vajadusel  

---

# Kokkuvõte
SELinux on võimas turvasüsteem, mis kaitseb servereid kompromiteerimise eest.  
Õige kasutamine nõuab kontekstide, booleanide ja audit logide mõistmist, kuid pakub tugevat kaitset teenuste ja failide tasemel.
```
