# 📄 **Fail: `security/ssh-hardening.md`**

```markdown
# SSH Hardening – Security käsiraamat  
## (Ciphers, MACs, key types, config hardening, fail2ban, 2FA)

## Ülevaade
SSH on serverite kõige kriitilisem ligipääsukanal.  
SSH hardening tähendab konfiguratsiooni karmistamist, et vältida:

- brute‑force rünnakuid  
- nõrku võtmeformaate  
- vananenud krüptograafiat  
- privilege escalation võimalusi  
- paroolipõhist autentimist  

---

# 1. SSH konfiguratsioonifail

Peamine konfiguratsioon:

```
/etc/ssh/sshd_config
```

Pärast muudatusi:

```bash
systemctl restart sshd
```

---

# 2. Keela parooliga sisselogimine

Kõige olulisem samm:

```bash
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
```

Ainult võtmega autentimine.

---

# 3. Keela root login

```bash
PermitRootLogin no
```

Soovitatav kasutada sudo’t.

---

# 4. Lubatud võtmetüübid

Keela vanad võtmed:

```bash
HostKeyAlgorithms ssh-ed25519,ssh-rsa
PubkeyAcceptedAlgorithms ssh-ed25519,ssh-rsa
```

Soovitus:
- **ed25519** → kiire, turvaline  
- **rsa 4096** → fallback  

Keela DSA ja ECDSA:

```bash
HostKeyAlgorithms -ssh-dss,-ecdsa-sha2-nistp256
```

---

# 5. Ciphers ja MACs

Keela nõrgad algoritmid:

```bash
Ciphers aes256-ctr,aes192-ctr,aes128-ctr,chacha20-poly1305@openssh.com
MACs hmac-sha2-512,hmac-sha2-256
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
```

---

# 6. Lubatud kasutajad ja grupid

Piira ligipääsu:

```bash
AllowUsers peeter deploy
AllowGroups sshusers admins
```

---

# 7. Idle timeout

```bash
ClientAliveInterval 300
ClientAliveCountMax 2
```

5 minutit → ühendus katkestatakse.

---

# 8. Porti muutmine (valikuline)

```bash
Port 2222
```

Ei ole turvameede, kuid vähendab müra logides.

---

# 9. Fail2ban

Installi:

```bash
apt install fail2ban
```

Või:

```bash
yum install fail2ban
```

SSH filter:

```
/etc/fail2ban/jail.local
```

Näide:

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
maxretry = 5
bantime = 1h
```

---

# 10. 2FA (Google Authenticator)

Installi:

```bash
apt install libpam-google-authenticator
```

Luba PAM:

```
/etc/pam.d/sshd
```

Lisa:

```
auth required pam_google_authenticator.so
```

SSH config:

```bash
ChallengeResponseAuthentication yes
```

---

# 11. SSH banner (legal notice)

```
/etc/issue.net
```

SSH config:

```bash
Banner /etc/issue.net
```

---

# 12. Logimine ja audit

Vaata logisid:

```bash
journalctl -u ssh
```

Või:

```bash
grep sshd /var/log/auth.log
```

---

# 13. Best Practices

- Kasuta ainult võtmega autentimist  
- Keela root login  
- Kasuta ed25519 võtmeid  
- Keela nõrgad ciphers ja MACs  
- Kasuta fail2ban brute‑force kaitseks  
- Kasuta 2FA kriitilistes süsteemides  
- Piira kasutajaid ja gruppe  
- Kasuta auditd või journald logide jälgimiseks  

---

# Kokkuvõte
SSH hardening tugevdab serveri turvalisust, eemaldades nõrgad algoritmid, piirates ligipääsu ja lisades kaitsekihte nagu fail2ban ja 2FA.  
See on üks olulisemaid samme turvalise Linuxi serveri ülesehitamisel.
```

