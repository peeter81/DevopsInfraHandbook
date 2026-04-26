# 📄 **Fail: `storage/device-mapper.md`**

```markdown
# Device Mapper – Storage käsiraamat  
## (dmsetup, mapping targets, LVM, cryptsetup, multipath, thin, cache)

## Ülevaade
**Device Mapper (DM)** on Linuxi kernelikiht, mis võimaldab luua virtuaalseid block‑seadmeid.  
See on alus paljudele tehnoloogiatele:

- **LVM** (LV-d, thin poolid, cache, RAID)  
- **dm-crypt** (LUKS krüpteerimine)  
- **dm-multipath** (SAN multipath)  
- **dm-thin** (thin provisioning)  
- **dm-cache** (SSD cache)  
- **dm-snapshot** (classic snapshots)  

Kõik need töötavad Device Mapperi peal.

---

# 1. Device Mapperi põhimõte

DM loob virtuaalse block‑seadme:

```
/dev/mapper/<nimi>
```

See seade on **mapping** füüsilistele block‑idele.

Mappingut juhib **target**, nt:

- linear  
- striped  
- crypt  
- thin  
- cache  
- snapshot  
- multipath  

---

# 2. dmsetup – põhikäsk

Vaata kõiki DM seadmeid:

```bash
dmsetup ls
```

Detailne info:

```bash
dmsetup info <seade>
```

Näita mappingut:

```bash
dmsetup table <seade>
```

---

# 3. Device Mapperi targets

## 3.1 linear
Lihtne 1:1 mapping.

Näide:

```
0 2097152 linear 8:1 0
```

## 3.2 striped
RAID0 stiilis striping.

## 3.3 crypt
dm-crypt (LUKS) kasutab seda.

## 3.4 snapshot
Classic LVM snapshotid.

## 3.5 thin
Thin provisioning (thin pool + thin LV).

## 3.6 cache
SSD cache HDD ees.

## 3.7 multipath
SAN multipath (failover + load balancing).

---

# 4. LVM ja Device Mapper

Iga LVM LV on tegelikult DM seade:

```bash
ls -l /dev/mapper
```

Näed:

- vg/lv → symbolic link  
- tegelik DM seade → `/dev/dm-0`, `/dev/dm-1`, jne

Vaata mappingut:

```bash
dmsetup table /dev/vgdata/lvdata
```

---

# 5. dm-crypt (LUKS)

Krüpteeritud seade on DM target:

```bash
cryptsetup luksOpen /dev/sda1 cryptroot
```

Loob:

```
/dev/mapper/cryptroot
```

Vaata:

```bash
dmsetup table cryptroot
```

---

# 6. dm-multipath

SAN keskkondades kasutatakse multipath’i:

```bash
multipath -ll
```

DM loob ühe virtuaalse seadme mitme tee peale.

---

# 7. dm-thin (thin provisioning)

Thin pool koosneb:

- data LV → dm-thin target  
- metadata LV → dm-thin metadata target  

Vaata:

```bash
dmsetup table vgthin-thinpool
```

---

# 8. dm-cache (SSD cache)

SSD cache kasutab:

- cache data LV  
- cache metadata LV  
- origin LV  

DM target: **cache**

Vaata:

```bash
dmsetup table vgcache-data
```

---

# 9. dm-snapshot

Classic LVM snapshotid kasutavad **snapshot** targetit.

Näide:

```bash
dmsetup table vgdata-snap1
```

---

# 10. Device Mapper debugging

## 10.1 Kõik DM seadmed

```bash
dmsetup ls --tree
```

## 10.2 Kernel logid

```bash
dmesg | grep dm-
```

## 10.3 DM statistika

```bash
dmsetup status
```

---

# 11. Best Practices

- Ära muuda DM mappingut käsitsi, kui kasutad LVM-i  
- Kasuta `dmsetup table` probleemide diagnoosimiseks  
- Jälgi thin pooli metadata’t (dm-thin)  
- Kasuta multipath SAN keskkondades  
- Ära kasuta dm-crypt’i ilma headeri backupita  
- Kasuta dm-cache ainult kiire SSD-ga  

---

# Kokkuvõte
Device Mapper on Linuxi storage‑arhitektuuri keskne komponent.  
LVM, krüpteerimine, multipath, thin provisioning ja SSD cache kõik töötavad DM targetite peal.  
DM mõistmine aitab diagnoosida keerukaid storage‑probleeme ja ehitada stabiilseid süsteeme.
```
