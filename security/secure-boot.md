# 📄 **Fail: `security/secure-boot.md`**

```markdown
# Secure Boot – Security käsiraamat  
## (UEFI, signatures, shim, kernel validation, key management)

## Ülevaade
Secure Boot on UEFI funktsioon, mis tagab, et süsteem käivitab **ainult allkirjastatud ja usaldusväärseid** boot‑komponente.

Secure Boot kontrollib:
- UEFI firmware  
- bootloader (shim, GRUB2)  
- kernel  
- kernel modules  
- initramfs  

Kui mõni komponent on muudetud või allkiri ei sobi → käivitamine peatatakse.

---

# 1. Secure Boot komponendid

### 1.1 UEFI firmware
Hoidub võtmeid ja kontrollib allkirju.

### 1.2 Shim
Microsofti allkirjastatud väike loader, mis lubab Linuxi käivitada.

### 1.3 GRUB2
Bootloader, mis peab olema allkirjastatud.

### 1.4 Kernel
Iga kernel peab olema allkirjastatud.

### 1.5 Kernel modules
Secure Boot režiimis:
- unsigned mooduleid ei saa laadida  
- DKMS moodulid peavad olema allkirjastatud  

---

# 2. Kontrolli Secure Boot olekut

```bash
mokutil --sb-state
```

Väljund:
- **SecureBoot enabled**
- **SecureBoot disabled**

Vaata UEFI infot:

```bash
dmesg | grep -i secure
```

---

# 3. MOK (Machine Owner Key)

Linux kasutab MOK süsteemi, et lubada kasutaja enda võtmeid.

Vaata MOK võtmeid:

```bash
mokutil --list-enrolled
```

Lisa uus võti:

```bash
mokutil --import mykey.der
```

Pärast rebooti UEFI MOK Manager küsib kinnitust.

---

# 4. Kernel module signing

Secure Boot režiimis peavad kõik moodulid olema allkirjastatud.

### 4.1 Loo võti

```bash
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=Custom Kernel Modules/"
```

### 4.2 Allkirjasta moodul

```bash
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 MOK.priv MOK.der mymodule.ko
```

### 4.3 Lisa võti MOK süsteemi

```bash
mokutil --import MOK.der
```

---

# 5. DKMS ja Secure Boot

DKMS moodulid (nt NVIDIA, VirtualBox) tuleb allkirjastada automaatselt.

Loo skript:

```
/etc/dkms/post-build.d/sign-modules
```

Sisu:

```bash
#!/bin/bash
for module in $(find /var/lib/dkms/ -name '*.ko'); do
    /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 /root/MOK.priv /root/MOK.der "$module"
done
```

Muuda õigused:

```bash
chmod +x /etc/dkms/post-build.d/sign-modules
```

---

# 6. Secure Boot ja TPM

Secure Boot + TPM = **Measured Boot**

- Secure Boot kontrollib allkirju  
- TPM salvestab PCR registritesse mõõtmised  
- LUKS saab siduda PCR väärtustega  

See tagab, et ketas avaneb ainult siis, kui boot‑ahel on puutumata.

---

# 7. Secure Boot keelamine

UEFI menüüst:
- **Security → Secure Boot → Disable**

Linuxist seda teha ei saa.

---

# 8. Secure Boot probleemid

- NVIDIA draiverid vajavad allkirjastamist  
- VirtualBox moodulid ei lae  
- custom kernelid ei käivitu ilma allkirjata  
- mõned live‑USB-d ei tööta  
- vanad Linuxi distrod ei toeta shim’i  

---

# 9. Best Practices

- Hoia Secure Boot **enabled** tootmises  
- Kasuta MOK võtmeid custom moodulite jaoks  
- Kasuta TPM + Secure Boot LUKS automaatseks avamiseks  
- Ära allkirjasta tundmatuid mooduleid  
- Kasuta ainult usaldusväärseid võtmeid  
- Testi kernel uuendusi enne tootmisesse viimist  

---

# Kokkuvõte
Secure Boot tagab, et süsteem käivitab ainult usaldusväärseid ja allkirjastatud komponente.  
Koos TPM‑iga moodustab see tugeva turvakihi, mis kaitseb boot‑ahelat ja võimaldab turvalist disk encryption integratsiooni.
```
