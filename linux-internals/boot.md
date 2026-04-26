# 📄 **Fail 8: `linux-internals/boot.md`**

```markdown
# Linux Boot Process – Linux Internals käsiraamatu osa  
## (BIOS/UEFI → Bootloader → Kernel → initramfs → systemd)

## Ülevaade
Linuxi käivitumisprotsess koosneb mitmest etapist:

1. BIOS / UEFI  
2. Bootloader (GRUB)  
3. Kernel laadimine  
4. initramfs  
5. systemd (PID 1)  
6. Teenuste käivitamine  

Iga samm on kriitiline — kui üks neist ebaõnnestub, süsteem ei käivitu.

---

# 1. BIOS / UEFI

BIOS/UEFI teeb:
- riistvara initsialiseerimise  
- CPU, RAM, diskide tuvastamise  
- boot device’i valimise  

UEFI kasutab **EFI System Partition (ESP)**.

Vaata boot mode’i:

```
ls /sys/firmware/efi
```

Kui kataloog on olemas → UEFI.

---

# 2. Bootloader (GRUB)

GRUB ülesanded:
- kernelivaliku kuvamine  
- kernelile parameetrite edastamine  
- initramfs laadimine  

Konfiguratsioon:

```
/etc/default/grub
```

Uuenda:

```
update-grub
```

---

# 3. Kernel laadimine

GRUB laadib kernelifaili:

```
/boot/vmlinuz-<versioon>
```

Kernel:
- tuvastab riistvara  
- mountib root filesystemi  
- käivitab initramfs’i  

Kernel logid:

```
dmesg
```

---

# 4. initramfs

initramfs on väike ajutine root filesystem, mis sisaldab:
- draivereid  
- krüpteeritud ketaste avamist  
- LVM tuge  
- RAID tuge  

Vaata sisu:

```
lsinitramfs /boot/initrd.img-$(uname -r)
```

---

# 5. systemd (PID 1)

Kui kernel on valmis, käivitab ta:

```
/sbin/init → systemd
```

systemd:
- käivitab teenused  
- mountib failisüsteemid  
- haldab logisid  
- haldab target’eid  

Vaata boot logi:

```
journalctl -b
```

---

# 6. systemd targets

Sarnane runlevelitele.

Näited:
- `multi-user.target`  
- `graphical.target`  
- `rescue.target`  
- `emergency.target`  

Vaheta target:

```
systemctl isolate multi-user.target
```

---

# 7. Boot probleemide diagnoosimine

## 7.1 GRUB shell
Kui GRUB ei leia kernelit:

```
ls
set prefix=(hd0,1)/boot/grub
set root=(hd0,1)
insmod normal
normal
```

## 7.2 initramfs shell
Kui root filesystem ei mounti, saad `/bin/sh` shelli.

Kontrolli:

```
ls /dev
blkid
```

## 7.3 systemd boot logid

```
journalctl -xb
```

---

# 8. Kernel parameetrid

Kernelile saab anda parameetreid GRUB-is:

```
GRUB_CMDLINE_LINUX="quiet splash systemd.unified_cgroup_hierarchy=1"
```

Levinud parameetrid:
- `quiet` – vähem logi  
- `nomodeset` – GPU probleemide lahendamiseks  
- `intel_iommu=on` – IOMMU lubamine  
- `root=/dev/sda2` – root FS määramine  

---

# 9. Boot kiirendamine

- Keela mittevajalikud teenused  
- Kasuta `systemd-analyze`  
- Kasuta SSD-d  
- Kasuta paralleelset teenuste käivitamist  

Vaata boot aega:

```
systemd-analyze
systemd-analyze blame
```

---

# 10. Best Practices

- Hoia mitu kerneliversiooni /boot all  
- Ära kustuta initramfs’i  
- Kasuta UEFI-d uutes süsteemides  
- Kontrolli `journalctl -b` boot-probleemide korral  
- Kasuta kernel parameetreid riistvaraprobleemide diagnoosimiseks  

---

# Kokkuvõte
Linuxi boot-protsess on mitmeastmeline ja paindlik.  
BIOS/UEFI → GRUB → kernel → initramfs → systemd → teenused moodustavad tervikliku käivitusahela, mille mõistmine on kriitiline DevOps/SRE töö jaoks.
```
