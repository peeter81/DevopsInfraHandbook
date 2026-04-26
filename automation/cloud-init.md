# 📄 **Fail: `cloud-init.md`**

```markdown
# cloud-init – Automation käsiraamat  
## (User data, cloud-config, packages, users, files, runcmd, write_files)

## Ülevaade
cloud-init on tööriist, mis konfigureerib VM‑e automaatselt esimesel käivitamisel.  
Seda kasutavad:

- AWS EC2  
- Azure VM  
- Google Compute Engine  
- OpenStack  
- Proxmox (cloud-init tugi)  
- VMware (cloud-init image’iga)

cloud-init loeb **user-data** faili ja teeb:

- pakettide installi  
- kasutajate loomise  
- failide kirjutamise  
- skriptide käivitamise  
- hostname seadistamise  
- SSH võtmete lisamise  

---

# 1. cloud-init formaadid

cloud-init toetab kolme formaati:

1. **cloud-config** (YAML) → kõige levinum  
2. shell script (`#!/bin/bash`)  
3. MIME multi-part (kombineeritud)

---

# 2. cloud-config põhinäide

```yaml
#cloud-config
package_update: true
packages:
  - nginx

users:
  - name: peeter
    groups: sudo
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAA...

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
```

---

# 3. Hostname seadistamine

```yaml
#cloud-config
hostname: web01
fqdn: web01.example.com
```

---

# 4. Pakettide install

```yaml
packages:
  - htop
  - curl
  - git
```

---

# 5. Failide kirjutamine (write_files)

```yaml
write_files:
  - path: /etc/motd
    permissions: "0644"
    content: |
      Tere tulemast serverisse!
```

---

# 6. Skriptide käivitamine (runcmd)

```yaml
runcmd:
  - echo "Start script"
  - apt install -y docker.io
  - systemctl enable docker
```

**NB!** `runcmd` käivitub *pärast* boot’i.

---

# 7. Shell script user-data

```bash
#!/bin/bash
apt update
apt install -y nginx
echo "Hello" > /var/www/html/index.html
```

---

# 8. Multi-part MIME (advanced)

Kombineerib mitu faili:

```
Content-Type: multipart/mixed; boundary="==BOUNDARY=="
MIME-Version: 1.0

--==BOUNDARY==
Content-Type: text/cloud-config

#cloud-config
packages:
  - nginx

--==BOUNDARY==
Content-Type: text/x-shellscript

#!/bin/bash
echo "Hello" > /var/www/html/index.html

--==BOUNDARY==--
```

---

# 9. cloud-init logid

Logid asuvad:

```
/var/log/cloud-init.log
/var/log/cloud-init-output.log
```

Kontrolli staatust:

```bash
cloud-init status
```

---

# 10. cloud-init uuesti käivitamine

```bash
sudo cloud-init clean
sudo cloud-init init
```

---

# 11. Proxmox cloud-init

Proxmoxis:

1. Loo cloud-init template (Ubuntu cloud image)  
2. Lisa VM‑ile Cloud-Init drive  
3. Täida väljad:  
   - user  
   - password  
   - SSH key  
   - network config  

cloud-init failid asuvad:

```
/etc/cloud/cloud.cfg
/etc/cloud/cloud.cfg.d/
```

---

# 12. Best Practices

- Kasuta **cloud-config** formaati  
- Ära pane paroole user-data sisse (kasuta SSH võtmeid)  
- Kasuta `write_files` konfiguratsioonifailide jaoks  
- Kasuta `runcmd` teenuste käivitamiseks  
- Kontrolli logisid `/var/log/cloud-init.log`  
- Kasuta **cloud-init clean** testimisel  
- Kasuta **immutable image + cloud-init** kombinatsiooni  

---

# Kokkuvõte
cloud-init võimaldab VM‑e automaatselt konfigureerida esimesel käivitamisel.  
See on kriitiline tööriist pilveautomaatikas, CI/CD-s ja golden image strateegiates.
```
