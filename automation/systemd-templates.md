# 📄 **Fail: `systemd-templates.md`**

```markdown
# systemd Templates – Linux käsiraamat  
## (Template units, @-units, instance names, overrides, environment files)

## Ülevaade
systemd **template units** võimaldavad luua korduvkasutatavaid teenuseid, mis käivituvad erinevate instantsinimedega.  
See on ideaalne:

- mitme sama rakenduse instantsi jaoks  
- mitme konteineri jaoks  
- mitme worker-protsessi jaoks  
- mitme VPN-tunneli jaoks  
- mitme mount’i jaoks  

Template unit = fail, mille nimi sisaldab **@** sümbolit.

Näide:

```
myservice@.service
```

---

# 1. Template unit struktuur

`/etc/systemd/system/myapp@.service`:

```ini
[Unit]
Description=My App instance %i

[Service]
ExecStart=/usr/local/bin/myapp --instance %i
Restart=always

[Install]
WantedBy=multi-user.target
```

### `%i` = instantsi nimi  
Kui käivitad `myapp@worker1.service`, siis `%i = worker1`.

---

# 2. Template instantsi käivitamine

```bash
systemctl start myapp@worker1
systemctl start myapp@worker2
```

Vaata:

```bash
systemctl status myapp@worker1
```

---

# 3. Template enable/disable

```bash
systemctl enable myapp@worker1
systemctl disable myapp@worker1
```

---

# 4. Environment failid instantsidele

Loo instantsipõhine environment fail:

`/etc/myapp/worker1.env`:

```
PORT=8081
MODE=worker
```

Template:

```ini
EnvironmentFile=/etc/myapp/%i.env
```

---

# 5. Override ainult ühele instantsile

```bash
systemctl edit myapp@worker1
```

Luuakse:

```
/etc/systemd/system/myapp@worker1.service.d/override.conf
```

Näide override:

```ini
[Service]
Environment="DEBUG=true"
```

---

# 6. Template koos multiple instances

Näide 3 workerit:

```bash
systemctl enable --now myapp@worker{1,2,3}
```

---

# 7. Template socket units

Template võib olla ka socket:

`myapp@.socket`:

```ini
[Socket]
ListenStream=/run/myapp-%i.sock

[Install]
WantedBy=sockets.target
```

---

# 8. Template mount units

`mnt-data@.mount`:

```ini
[Mount]
What=/dev/disk/by-label/%i
Where=/mnt/%i
Type=ext4
```

Käivita:

```bash
systemctl start mnt-data@backup
```

---

# 9. Template service + timer

`backup@.service`:

```ini
[Service]
ExecStart=/usr/local/bin/backup --target %i
```

`backup@.timer`:

```ini
[Timer]
OnCalendar=daily
Unit=backup@%i.service
```

Käivita:

```bash
systemctl enable --now backup@server1.timer
```

---

# 10. Kasulikud % muutujad

| Muutuja | Kirjeldus |
|--------|-----------|
| `%i` | instantsi nimi |
| `%I` | täielik instantsi nimi (escape’itud) |
| `%n` | unit’i nimi |
| `%p` | unit’i prefiks |
| `%H` | hostname |
| `%u` | kasutaja |

---

# 11. Best Practices

- Kasuta template’e korduvate teenuste jaoks  
- Kasuta `%i` instantsipõhiste seadete jaoks  
- Kasuta **EnvironmentFile** instantsipõhiste konfiguratsioonide jaoks  
- Kasuta `systemctl edit` override’de jaoks  
- Ära kopeeri sama teenust kümnesse faili — kasuta template’i  
- Kasuta timer + template kombinatsiooni mitme serveri backup’iks  

---

# Kokkuvõte
systemd template units võimaldavad luua paindlikke ja korduvkasutatavaid teenuseid, mis töötavad mitme instantsina.  
See on ideaalne worker-protsesside, konteinerite, mount’ide ja automaatsete backup’ide jaoks.
```

