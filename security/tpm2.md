# 📄 **Fail: `security/tpm2.md`**

```markdown
# TPM 2.0 – Security käsiraamat  
## (Trusted Platform Module, PCR, Secure Boot, disk encryption, attestation)

## Ülevaade
TPM 2.0 (Trusted Platform Module) on riistvaraline turvakiip, mis pakub:

- turvalist võtmete hoiustamist  
- süsteemi tervikluse kontrolli (PCR registrid)  
- Secure Boot tuge  
- disk encryption (LUKS, BitLocker) võtmete kaitset  
- attestation (tõestus, et süsteem pole kompromiteeritud)  

TPM on kriitiline komponent kaasaegsetes turvalistes Linuxi ja Windowsi süsteemides.

---

# 1. Kontrolli, kas TPM 2.0 on olemas

```bash
tpm2_getcap -c properties-fixed
```

Või:

```bash
ls /dev/tpm*
```

---

# 2. TPM2 tööriistad

Installi:

```bash
yum install tpm2-tools
```

Või:

```bash
apt install tpm2-tools
```

---

# 3. PCR registrid (Platform Configuration Registers)

PCR-id salvestavad süsteemi käivitamise mõõtmised:

- BIOS  
- Bootloader  
- Kernel  
- Initramfs  
- Secure Boot komponendid  

Vaata PCR väärtusi:

```bash
tpm2_pcrread sha256:0,1,2,3,4,5,6,7
```

PCR-id kasutatakse:

- Secure Boot  
- Measured Boot  
- LUKS võtmete automaatseks avamiseks  
- Remote attestation  

---

# 4. TPM ja Secure Boot

TPM salvestab mõõtmised PCR registritesse.  
Secure Boot kontrollib:

- BIOS → Bootloader  
- Bootloader → Kernel  
- Kernel → Initramfs  

Kui midagi on muudetud → PCR väärtused muutuvad → süsteemi ei usaldata.

---

# 5. TPM ja LUKS (disk encryption)

TPM saab hoida LUKS võtmeid ja avada ketta automaatselt, kui süsteemi terviklikkus on korras.

## 5.1 Loo LUKS võti TPM-is

```bash
tpm2_createprimary -C o -c primary.ctx
tpm2_create -C primary.ctx -u key.pub -r key.priv
tpm2_load -C primary.ctx -u key.pub -r key.priv -c key.ctx
```

## 5.2 Seo LUKS TPM-iga

```bash
systemd-cryptenroll --tpm2-device=auto /dev/sda3
```

Kontrolli:

```bash
systemd-cryptenroll --list /dev/sda3
```

---

# 6. TPM ja systemd-cryptenroll

Systemd teeb TPM kasutamise lihtsaks.

LUKS avamine TPM abil:

```bash
systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+2+7 /dev/sda3
```

PCR-id määravad, milliste boot-komponentide muutmine keelab automaatse avamise.

---

# 7. TPM NV storage (püsiv mälu)

TPM-il on väike püsiv mälu (NV index), kuhu saab salvestada:

- võtmeid  
- konfiguratsiooni  
- attestation andmeid  

Näide:

```bash
tpm2_nvdefine 0x1500016 -C o -s 32 -a "ownerread|ownerwrite"
tpm2_nvwrite 0x1500016 -C o -i secret.bin
tpm2_nvread 0x1500016 -C o
```

---

# 8. Remote Attestation

TPM võimaldab serveril tõestada, et:

- BIOS pole muudetud  
- kernel on usaldusväärne  
- bootloader on puutumata  

Attestation tööriistad:

- Keylime  
- tpm2-tools  
- IMA (Integrity Measurement Architecture)  

---

# 9. TPM reset ja puhastamine

**HOIATUS:** See kustutab kõik TPM võtmed.

```bash
tpm2_clear
```

Vajalik näiteks:

- emaplaadi vahetusel  
- TPM korruptsiooni korral  
- uue süsteemi ettevalmistamisel  

---

# 10. Best Practices

- Kasuta TPM-i LUKS võtmete kaitsmiseks  
- Kasuta PCR 7 (Secure Boot) ja PCR 2 (kernel) sidumist  
- Ära salvesta suuri andmeid TPM NV-sse  
- Kasuta systemd-cryptenroll TPM integratsiooniks  
- Kasuta Secure Boot + TPM koos  
- Kasuta attestationit serverite terviklikkuse kontrolliks  

---

# Kokkuvõte
TPM 2.0 on kaasaegse turvalise süsteemi alus.  
See pakub riistvaralist võtmete kaitset, boot integrity kontrolli, disk encryption integratsiooni ja attestation võimalusi.  
TPM-i kasutamine koos Secure Booti ja LUKS-iga annab tugeva turvakihi nii serveritele kui tööjaamadele.
```
