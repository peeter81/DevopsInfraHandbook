# 📄 **Fail: `security/luks-tpm.md`**

```markdown
# LUKS + TPM2 – Security käsiraamat  
## (TPM‑põhine ketta avamine, PCR sidumine, systemd-cryptenroll, auto‑unlock)

## Ülevaade
LUKS (Linux Unified Key Setup) on Linuxi standardne kettakrüpteerimise süsteem.  
TPM2 (Trusted Platform Module) võimaldab LUKS võtmeid **turvaliselt riistvaras hoida** ja ketta automaatselt avada, kui süsteemi boot‑ahel on usaldusväärne.

LUKS + TPM2 eelised:
- automaatne ketta avamine ilma paroolita  
- võtmed ei lahku kunagi TPM‑ist  
- PCR sidumine tagab, et ketas avaneb ainult siis, kui boot‑ahel pole muudetud  
- sobib serveritele, tööjaamadele, kiosk‑süsteemidele  

---

# 1. Eeldused

- TPM 2.0 olemas (`/dev/tpm0`)  
- Secure Boot soovitatav  
- systemd ≥ 248 (sisaldab TPM integratsiooni)  
- LUKS2 konteiner  

Kontrolli TPM-i:

```bash
ls /dev/tpm*
tpm2_getcap -c properties-fixed
```

---

# 2. LUKS ketta ettevalmistamine

Kui ketas pole veel krüpteeritud:

```bash
cryptsetup luksFormat /dev/sda3
cryptsetup open /dev/sda3 cryptroot
mkfs.ext4 /dev/mapper/cryptroot
```

Kui LUKS juba olemas, jätka järgmise sammuga.

---

# 3. LUKS TPM2 enroll (systemd-cryptenroll)

Systemd teeb TPM integratsiooni väga lihtsaks.

Lisa TPM2 võtme‑slot:

```bash
systemd-cryptenroll --tpm2-device=auto /dev/sda3
```

Kontrolli:

```bash
systemd-cryptenroll --list /dev/sda3
```

Näed midagi sellist:

```
tpm2: enrolled
```

---

# 4. PCR sidumine (Measured Boot)

PCR-id määravad, milliste boot‑komponentide muutmine keelab automaatse avamise.

Levinud PCR-id:

| PCR | Sisu |
|-----|------|
| 0 | BIOS/firmware |
| 2 | kernel + initramfs |
| 7 | Secure Boot policy |

Soovitatav:

```bash
systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+2+7 /dev/sda3
```

Kui keegi muudab kernelit, initramfs’i või Secure Boot võtmeid → ketas ei avane automaatselt.

---

# 5. Automaatne avamine bootimisel

Kui TPM enrollment on tehtud, systemd avab ketta automaatselt:

- initramfs loeb TPM-ist võtme  
- kontrollib PCR väärtusi  
- kui kõik sobib → ketas avaneb  
- kui ei sobi → küsitakse LUKS parooli  

Kontrolli journalist:

```bash
journalctl -b | grep -i tpm
```

---

# 6. TPM võtme eemaldamine

Kui soovid TPM-i LUKS-ist lahti siduda:

```bash
systemd-cryptenroll --wipe-slot=tpm2 /dev/sda3
```

---

# 7. TPM võtme uuesti loomine

Kui PCR-id muutusid (kernel uuendus, Secure Boot muudatus):

```bash
systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+2+7 --recreate-key /dev/sda3
```

---

# 8. LUKS + TPM2 + Secure Boot

Parim turvakombinatsioon:

- Secure Boot kontrollib allkirju  
- TPM mõõdab boot‑ahela PCR‑idesse  
- LUKS avatakse ainult siis, kui PCR väärtused sobivad  

See kaitseb:
- evil‑maid rünnakute eest  
- initramfs manipulatsiooni eest  
- kernel patchimise eest  
- bootloaderi muutmise eest  

---

# 9. LUKS + TPM2 + Network Unlock (serverid)

Serverites saab kasutada:

- TPM2 auto‑unlock  
- fallback parool  
- remote unlock SSH kaudu  
- Tang / Clevis (võrgupõhine unlock)

Näide Clevis + TPM:

```bash
clevis luks bind -d /dev/sda3 tpm2 '{}'
```

---

# 10. Best Practices

- Kasuta PCR 0 + 2 + 7 sidumist  
- Kasuta Secure Boot + TPM koos  
- Ära salvesta TPM võtmeid plaintext kujul  
- Kasuta LUKS2 (mitte LUKS1)  
- Testi auto‑unlock pärast kernel uuendusi  
- Kasuta fallback parooli serverites  

---

# Kokkuvõte
TPM2 võimaldab LUKS krüpteeritud kettaid turvaliselt ja automaatselt avada, sidudes võtme süsteemi boot‑ahela terviklikkusega.  
See on üks tugevamaid ja mugavamaid lahendusi nii tööjaamade kui serverite turvamiseks.
```
