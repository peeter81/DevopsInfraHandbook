# 📄 **Fail: `storage/zfs.md`**

```markdown
# ZFS – Storage käsiraamat  
## (Pools, datasets, snapshots, ARC/L2ARC, RAID-Z, send/receive)

## Ülevaade
ZFS on enterprise‑taseme failisüsteem ja volume manager ühes paketis.  
See pakub:

- checksums (andmete terviklikkus)  
- copy‑on‑write (CoW)  
- snapshotid ja clone’id  
- sisseehitatud RAID-Z  
- deduplication  
- compression  
- send/receive replikatsioon  
- automaatne self‑healing  

ZFS ei ole Linuxi kernelis — kasutatakse **OpenZFS** versiooni.

---

# 1. Zpool (storage pool)

ZFS töötab **zpool** tasemel, mitte LVM‑i või mdadm‑i peal.

## 1.1 Loo zpool

```bash
zpool create tank /dev/sdb
```

## 1.2 RAID-Z

### RAID-Z1 (1 parity)

```bash
zpool create tank raidz1 /dev/sdb /dev/sdc /dev/sdd
```

### RAID-Z2 (2 parity)

```bash
zpool create tank raidz2 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

### RAID-Z3 (3 parity)

```bash
zpool create tank raidz3 /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf
```

---

# 2. Datasets

Datasets on nagu alamfailisüsteemid.

## 2.1 Loo dataset

```bash
zfs create tank/data
```

## 2.2 Compression

```bash
zfs set compression=lz4 tank/data
```

## 2.3 Quota

```bash
zfs set quota=100G tank/data
```

---

# 3. Snapshots

Snapshotid on kohesed ja ruumisäästlikud.

## 3.1 Loo snapshot

```bash
zfs snapshot tank/data@snap1
```

## 3.2 Snapshotide nimekiri

```bash
zfs list -t snapshot
```

## 3.3 Snapshoti taastamine

```bash
zfs rollback tank/data@snap1
```

---

# 4. Clone

Clone on kirjutatav snapshot.

```bash
zfs clone tank/data@snap1 tank/data_clone
```

---

# 5. Send / Receive (replikatsioon)

ZFS võimaldab andmeid saata teisele masinale.

## 5.1 Täis replikatsioon

```bash
zfs send tank/data@snap1 | ssh server zfs receive backup/data
```

## 5.2 Inkremendiline replikatsioon

```bash
zfs send -i snap1 snap2 tank/data | ssh server zfs receive backup/data
```

---

# 6. ARC ja L2ARC

ZFS kasutab agressiivset cachingut:

### ARC (RAM cache)
- väga kiire  
- ZFS kasutab RAM‑i maksimaalselt  

### L2ARC (SSD cache)
Lisa SSD cache:

```bash
zpool add tank cache /dev/nvme0n1
```

---

# 7. ZIL / SLOG (write log)

ZIL = ZFS Intent Log  
SLOG = eraldi SSD write log

Lisa SLOG:

```bash
zpool add tank log /dev/nvme1n1
```

Sobib:
- andmebaasid  
- NFS serverid  

---

# 8. Deduplication

Dedup säästab ruumi, kuid vajab palju RAM‑i.

Lülita sisse:

```bash
zfs set dedup=on tank/data
```

Soovitus:
- kasuta ainult siis, kui tead, mida teed  
- soovitatav RAM: **1–5 GB per 1 TB dedupitud andmeid**

---

# 9. Tervisekontroll

## 9.1 Pooli staatus

```bash
zpool status
```

## 9.2 Scrub (vigade parandamine)

```bash
zpool scrub tank
```

---

# 10. Best Practices

- Kasuta **lz4 compression** alati  
- Ära kasuta deduplication’it ilma piisava RAM‑ita  
- Kasuta RAID-Z2 või RAID-Z3 suurte ketastega  
- Tee regulaarselt `zpool scrub`  
- Kasuta SLOG‑i ainult kiire NVMe SSD‑ga  
- Ära kasuta ZFS‑i koos hardware RAID‑iga (kasuta JBOD)  
- Kasuta send/receive backup’iks  

---

# Kokkuvõte
ZFS on üks võimsamaid failisüsteeme maailmas.  
See ühendab RAID‑i, snapshotid, replikatsiooni, caching’u ja andmete terviklikkuse ühte terviklikku lahendusse, mis sobib nii serveritele, NAS‑idele kui ka enterprise storage’ile.
```
