# 📄 **Fail: `security/fips.md`**

```markdown
# FIPS Mode – Security käsiraamat  
## (FIPS 140-2/140-3, crypto policies, OpenSSL, kernel, compliance)

## Ülevaade
FIPS (Federal Information Processing Standards) on USA valitsuse krüptograafiline standard.  
Linuxis tähendab **FIPS mode**, et süsteem kasutab ainult FIPS‑sertifitseeritud krüptograafilisi algoritme ja mooduleid.

FIPS on nõutud:
- valitsusasutustes  
- finantssektoris  
- kõrge turvalisusega keskkondades  
- FedRAMP / DoD / NIST nõuete täitmisel  

---

# 1. Mis muutub FIPS režiimis?

Kui FIPS mode on lubatud:

- OpenSSL kasutab ainult FIPS‑sertifitseeritud algoritme  
- keelatakse MD5, RC4, DES, SHA‑1 (osaliselt)  
- kernel kasutab FIPS‑sertifitseeritud crypto mooduleid  
- SSH, IPsec, TLS, VPN-id kasutavad ainult FIPS‑lubatud ciphers’eid  
- /dev/random ja /dev/urandom käitumine muutub rangemaks  

---

# 2. Kontrolli, kas FIPS on lubatud

```bash
cat /proc/sys/crypto/fips_enabled
```

0 = FIPS disabled  
1 = FIPS enabled

---

# 3. FIPS lubamine (RHEL, Rocky, Alma, CentOS Stream)

## 3.1 Installi FIPS moodulid

```bash
yum install dracut-fips
```

## 3.2 Luba FIPS

```bash
fips-mode-setup --enable
```

See teeb:
- kernel command line → `fips=1`
- uuendab initramfs
- kontrollib OpenSSL FIPS moodulit

## 3.3 Reboot

```bash
reboot
```

---

# 4. FIPS lubamine (Ubuntu)

Ubuntu kasutab **crypto-policies** süsteemi.

```bash
sudo update-crypto-policies --set FIPS
```

Reboot:

```bash
sudo reboot
```

---

# 5. Kontrolli OpenSSL FIPS olekut

```bash
openssl version
```

FIPS režiimis näed:

```
OpenSSL 3.x FIPS
```

Kontrolli FIPS providerit:

```bash
openssl list -providers
```

---

# 6. Millised algoritmid on lubatud?

### Lubatud:
- AES  
- SHA‑256 / SHA‑384 / SHA‑512  
- RSA (>= 2048 bit)  
- ECDSA  
- HMAC  
- DRBG (Deterministic Random Bit Generator)  

### Keelatud:
- MD5  
- SHA‑1 (signing keelatud)  
- DES / 3DES  
- RC4  
- Blowfish  
- DSA  

---

# 7. FIPS ja SSH

SSH kasutab ainult FIPS‑lubatud ciphers’eid.

Vaata:

```bash
ssh -Q cipher
```

FIPS režiimis näed ainult:
- aes256-ctr  
- aes192-ctr  
- aes128-ctr  
- chacha20-poly1305 (sõltub distro FIPS poliitikast)

---

# 8. FIPS ja TLS (Apache, Nginx)

OpenSSL piirab automaatselt ciphers’eid.

Kontrolli:

```bash
openssl ciphers -v
```

FIPS režiimis näed ainult AES‑põhiseid komplekte.

---

# 9. FIPS ja VPN-id

IPsec, OpenVPN, StrongSwan, Libreswan — kõik kasutavad ainult FIPS‑lubatud algoritme.

Näiteks StrongSwan:

```
ike=aes256-sha256-modp2048
esp=aes256-sha256
```

---

# 10. FIPS režiimi keelamine

RHEL/Alma/Rocky:

```bash
fips-mode-setup --disable
reboot
```

Ubuntu:

```bash
update-crypto-policies --set DEFAULT
reboot
```

---

# 11. FIPS režiimi probleemid

- mõned rakendused ei tööta (vajavad MD5 või SHA‑1)  
- Git ei saa SHA‑1 signatuure kasutada  
- VPN-id võivad vajada uuendatud konfiguratsiooni  
- konteinerid võivad kasutada mitte‑FIPS OpenSSL versioone  
- jõudlus võib langeda (rangem crypto)  

---

# 12. Best Practices

- Kasuta FIPS ainult siis, kui see on nõutud  
- Testi rakendusi enne FIPS lubamist  
- Kasuta FIPS‑sertifitseeritud OpenSSL versiooni  
- Kontrolli, et kõik konteinerid kasutavad FIPS‑ühilduvat crypto’t  
- Kasuta `fips-mode-setup` (RHEL) või `update-crypto-policies` (Ubuntu)  
- Ära lülita FIPS režiimi välja tootmises ilma mõju hindamata  

---

# Kokkuvõte
FIPS mode tagab, et Linux kasutab ainult sertifitseeritud krüptograafilisi algoritme ja mooduleid.  
See on vajalik kõrge turvalisusega keskkondades, kuid võib piirata rakenduste ühilduvust ja jõudlust.  
Õige planeerimine ja testimine on kriitilised.
```

