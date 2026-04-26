# 📄 **Fail 7: `linux-internals/security.md`**

```markdown
# Linux Security – Linux Internals käsiraamat  
## (SELinux, AppArmor, Capabilities, seccomp, PAM, auditd)

## Ülevaade
Linuxi turvamudel koosneb mitmest kihist:

- discretionary access control (DAC) – klassikalised failide õigused  
- mandatory access control (MAC) – SELinux, AppArmor  
- capabilities – peenhäälestatud root õigused  
- seccomp – syscall filtrid  
- namespaces – konteinerite isolatsioon  
- cgroups – ressursside piiramine  
- auditd – turvalogid  
- PAM – autentimine  

Need moodustavad tugeva ja paindliku turva-arhitektuuri.

---

# 1. Failiõigused (DAC)

Linux kasutab:
- owner  
- group  
- others  

Õigused:
- r (read)  
- w (write)  
- x (execute)  

Vaata:

```
ls -l
```

Muuda:

```
chmod 750 fail
chown user:group fail
```

---

# 2. SELinux (Mandatory Access Control)

SELinux on väga range turvasüsteem.

Režiimid:
- **enforcing** – reeglid kehtivad  
- **permissive** – logib, kuid ei blokeeri  
- **disabled** – välja lülitatud  

Vaata olekut:

```
getenforce
sestatus
```

Muuda:

```
setenforce 0
```

Logid:

```
/var/log/audit/audit.log
```

---

# 3. AppArmor

Lihtsam kui SELinux, profiilipõhine.

Vaata profiile:

```
aa-status
```

Luba profiil:

```
aa-enforce /etc/apparmor.d/usr.bin.nginx
```

---

# 4. Linux Capabilities

Root õigused saab jagada väiksemateks osadeks.

Näited:
- `CAP_NET_ADMIN` – võrguseaded  
- `CAP_SYS_ADMIN` – väga lai  
- `CAP_DAC_OVERRIDE` – ignoreeri failide õigusi  

Vaata protsessi capabilities:

```
capsh --print
```

Lisa konteinerile:

```
docker run --cap-add=NET_ADMIN ubuntu
```

---

# 5. seccomp (syscall filter)

seccomp piirab, milliseid syscall’e protsess võib teha.

Näide Dockeris:

```
docker run --security-opt seccomp=default.json nginx
```

---

# 6. Namespaces

Linux kasutab isolatsiooniks:

- pid  
- net  
- mount  
- ipc  
- uts  
- user  

Näide:

```
unshare -n bash
```

---

# 7. PAM (Pluggable Authentication Modules)

PAM kontrollib autentimist.

Failid:

```
/etc/pam.d/*
```

Näide sudo reegel:

```
auth required pam_wheel.so use_uid
```

---

# 8. auditd

auditd logib turvasündmusi.

Käivita:

```
systemctl start auditd
```

Vaata logisid:

```
ausearch -m avc
```

---

# 9. Failide terviklikkuse kontroll (AIDE)

Loo baseline:

```
aide --init
```

Kontrolli:

```
aide --check
```

---

# 10. Best Practices

- Kasuta SELinux’i tootmises (enforcing)  
- Kasuta AppArmorit Ubuntu serverites  
- Väldi `CAP_SYS_ADMIN` andmist konteineritele  
- Kasuta seccomp’i ja namespaces’e konteinerite turvamiseks  
- Kasuta auditd turvaintsidentide uurimiseks  
- Kasuta PAM-i tugevaks autentimiseks  
- Ära jaga root SSH ligipääsu  

---

# Kokkuvõte
Linuxi turvamudel on kihiline ja võimas.  
SELinux/AppArmor, capabilities, seccomp, namespaces ja auditd moodustavad tugeva aluse turvaliste serverite ja konteinerite ehitamiseks.
```
