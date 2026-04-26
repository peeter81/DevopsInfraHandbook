# 📄 **Fail 9: `linux-internals/logging.md`**

```markdown
# Linux Logging – Linux Internals käsiraamat  
## (systemd‑journald, rsyslog, logrotate, journalctl)

## Ülevaade
Linuxi logisüsteem koosneb mitmest komponendist:

- **systemd‑journald** – kernel + teenuste logid  
- **rsyslog** – traditsiooniline logiserver  
- **logrotate** – logide arhiveerimine ja puhastamine  
- **/var/log/** – failipõhised logid  
- **journalctl** – logide päringud  

Logid on kriitilised süsteemi diagnoosimiseks, turvalisuseks ja auditeerimiseks.

---

# 1. systemd‑journald

journald kogub logisid:

- kernelilt  
- systemd teenustelt  
- stdout/stderr väljundilt  
- syslogist  

Vaata journald olekut:

```
systemctl status systemd-journald
```

---

# 2. journalctl – logide vaatamine

## 2.1 Kõik logid

```
journalctl
```

## 2.2 Viimase boot’i logid

```
journalctl -b
```

## 2.3 Reaalajas logid

```
journalctl -f
```

## 2.4 Konkreetse teenuse logid

```
journalctl -u nginx
```

## 2.5 Kernel logid

```
journalctl -k
```

## 2.6 Ajafilter

```
journalctl --since "1 hour ago"
journalctl --since yesterday --until now
```

---

# 3. rsyslog

rsyslog kirjutab logid failidesse:

```
/var/log/syslog
/var/log/auth.log
/var/log/kern.log
```

Konfiguratsioon:

```
/etc/rsyslog.conf
/etc/rsyslog.d/*.conf
```

Restart:

```
systemctl restart rsyslog
```

---

# 4. logrotate

logrotate haldab logide:

- rotatsiooni  
- arhiveerimist  
- tihendamist  
- kustutamist  

Konfiguratsioon:

```
/etc/logrotate.conf
/etc/logrotate.d/*
```

Käivita käsitsi:

```
logrotate -f /etc/logrotate.conf
```

---

# 5. /var/log kataloog

Levinud logid:

| Fail | Sisu |
|------|------|
| `/var/log/syslog` | üldlogid |
| `/var/log/auth.log` | autentimine |
| `/var/log/kern.log` | kernel |
| `/var/log/dmesg` | boot logid |
| `/var/log/nginx/*` | nginx logid |
| `/var/log/audit/audit.log` | auditd |

---

# 6. Teenuste logid (systemd)

Systemd teenused logivad journald’i.

Vaata teenuse logi:

```
journalctl -u ssh
```

Vaata teenuse viimaseid ridu:

```
journalctl -u ssh -n 50
```

---

# 7. Logide suunamine (stdout/stderr)

Systemd teenused suunavad automaatselt:

- stdout → journald  
- stderr → journald  

Näide teenuse failist:

```
[Service]
ExecStart=/usr/bin/python3 app.py
StandardOutput=journal
StandardError=journal
```

---

# 8. Logide saatmine kaugserverisse

rsyslog:

```
*.* @10.10.10.10:514
```

journald → rsyslog → SIEM (Splunk, ELK, Graylog)

---

# 9. Best Practices

- Kasuta `journalctl -f` reaalajas jälgimiseks  
- Kasuta `journalctl -u <service>` teenuste diagnoosimiseks  
- Kasuta logrotate’i, et vältida kettaruumi täitumist  
- Saada logid keskserverisse (SIEM)  
- Ära kustuta `/var/log/journal` käsitsi  
- Kasuta auditd turvalogide jaoks  

---

# Kokkuvõte
Linuxi logisüsteem on võimas ja paindlik.  
systemd‑journald, rsyslog ja logrotate moodustavad tervikliku logiarhitektuuri, mis sobib serveritele, konteineritele ja pilvekeskkondadele.
```
