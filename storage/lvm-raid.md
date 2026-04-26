# 📄 **Fail: `storage/lvm-raid.md`**

```markdown
# LVM RAID – Storage käsiraamat  
## (RAID1, RAID5, RAID6, RAID10 LVM-is, recovery, monitoring)

## Ülevaade
LVM-il on sisseehitatud RAID-tugi, mis võimaldab luua:

- RAID1 – peegeldus  
- RAID5 – parity, ruumisäästlik  
- RAID6 – topelt parity  
- RAID10 – striping + mirroring  

LVM RAID on paindlikum kui mdadm:
- saab laiendada online  
- saab konverteerida RAID tasemeid  
- töötab LVM VG sees  
- toetab thin poolidega kombineerimist  

---

# 1. RAID1 (peegeldus)

## 1.1 Loo RAID1 LV

```bash
lvcreate -L 100G -n lv_raid1 --type raid1 -m1 vgdata
```

`-m1` → üks mirror (kokku 2 koopiat)

Vaata:

```bash
lvs -a -o +devices
```

---

# 2. RAID5

RAID5 vajab vähemalt **3 füüsilist ketast**.

## 2.1 Loo RAID5 LV

```bash
lvcreate -L 500G -n lv_raid5 --type raid5 -i3 vgdata
```

`-i3` → 3 data stripes + 1 parity

---

# 3. RAID6

RAID6 vajab vähemalt **4 ketast**.

```bash
lvcreate -L 1T -n lv_raid6 --type raid6 -i4 vgdata
```

---

# 4. RAID10

RAID10 = striping + mirroring.

```bash
lvcreate -L 500G -n lv_raid10 --type raid10 -i2 -m1 vgdata
```

Selgitus:
- `-i2` → 2 stripes  
- `-m1` → iga stripe on peegeldatud  

---

# 5. RAID staatuse vaatamine

```bash
lvs -a -o lv_name,raid_sync_action,raid_mismatch_count,devices vgdata
```

Detailne:

```bash
cat /proc/mdstat
```

Kuigi LVM RAID ei kasuta mdadm-i, näitab `/proc/mdstat` siiski sünkroniseerimise infot.

---

# 6. RAID taastamine (recovery)

Kui üks ketas läheb katki:

### 6.1 Märgi PV vigaseks

```bash
pvchange --missing /dev/sdc
```

### 6.2 Asenda ketas

```bash
pvcreate /dev/sdd
vgextend vgdata /dev/sdd
```

### 6.3 Taasta RAID

```bash
lvconvert --repair vgdata/lv_raid1
```

---

# 7. RAID laiendamine

## 7.1 Lisa uus PV

```bash
vgextend vgdata /dev/sde
```

## 7.2 Laienda LV

```bash
lvextend -L +200G vgdata/lv_raid5
```

## 7.3 Rebalance

LVM teeb seda automaatselt.

---

# 8. RAID taseme konverteerimine

## 8.1 RAID1 → RAID5

```bash
lvconvert --type raid5 -i3 vgdata/lv_raid1
```

## 8.2 RAID5 → RAID6

```bash
lvconvert --type raid6 vgdata/lv_raid5
```

## 8.3 RAID1 → RAID10

```bash
lvconvert --type raid10 -i2 vgdata/lv_raid1
```

---

# 9. RAID jõudlus

### RAID1
- hea lugemiskiirus  
- keskmine kirjutamiskiirus  

### RAID5
- hea lugemine  
- parity tõttu aeglasem kirjutamine  

### RAID6
- turvalisem kui RAID5  
- veel aeglasem kirjutamine  

### RAID10
- parim jõudlus  
- parim latentsus  
- parim sobivus andmebaasidele  

---

# 10. Best Practices

- Kasuta **RAID10** andmebaaside ja VM hostide jaoks  
- Ära kasuta **RAID5/6 SSD-dega** (parity write amplification)  
- Kasuta **RAID1** väikeste serverite jaoks  
- Jälgi `raid_sync_action` ja `raid_mismatch_count`  
- Tee regulaarselt `lvconvert --repair` test VM-is  
- Ära kasuta RAID5/6 väikeste ketastega (URE risk)  

---

# Kokkuvõte
LVM RAID on paindlik ja võimas lahendus, mis võimaldab luua RAID tasemeid otse LVM-i sees, toetades laiendamist, konverteerimist ja automaatset taastamist.  
See sobib nii serveritele, VM hostidele kui ka suuremahulistele storage-lahendustele.
```
