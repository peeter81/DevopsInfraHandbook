# 📄 **Fail 3: `linux-internals/filesystems.md`**

```markdown
# Linux Filesystems – Linux Internals käsiraamat  
## (EXT4, XFS, Btrfs, Inodes, Journaling, Mounting)

## Ülevaade
Linux toetab kümneid failisüsteeme, kuid kõige levinumad on:

- **EXT4** – vaikimisi, stabiilne, kiire  
- **XFS** – suurte failide ja serverite jaoks  
- **Btrfs** – snapshotid, RAID, checksums  
- **ZFS** – enterprise taseme, kuid mitte kernelis  
- **tmpfs** – RAM‑põhine failisüsteem  

Failisüsteemid kasutavad **inodes**, **journalingut**, **mount options** ja **block device’e**.

---

# 1. Inodes

Iga fail Linuxis koosneb:

- inode (metaandmed)
- data blocks (sisu)

Inode sisaldab:
- faili suurus  
- omanik  
- õigused  
- ajatemplid  
- block pointerid  

Vaata inode infot:

```
stat failinimi
```

---

# 2. EXT4

EXT4 on Linuxi kõige levinum failisüsteem.

Eelised:
- kiire  
- stabiilne  
- journaling  
- suur failitugi  

Loomine:

```
mkfs.ext4 /dev/sdb1
```

Kontroll:

```
fsck.ext4 /dev/sdb1
```

---

# 3. XFS

XFS on optimeeritud suurte failide ja serverite jaoks.

Eelised:
- väga kiire paralleelne I/O  
- hea suurte failide jaoks  
- ei toeta shrink’i  

Loomine:

```
mkfs.xfs /dev/sdb1
```

Kontroll:

```
xfs_repair /dev/sdb1
```

---

# 4. Btrfs

Btrfs on modernne CoW (copy‑on‑write) failisüsteem.

Eelised:
- snapshotid  
- checksums  
- sisseehitatud RAID  
- subvolume’d  

Loomine:

```
mkfs.btrfs /dev/sdb1
```

Snapshot:

```
btrfs subvolume snapshot /data /data-snap
```

---

# 5. ZFS (OpenZFS)

ZFS ei ole Linuxi kernelis, kuid on saadaval DKMS‑ina.

Eelised:
- checksums  
- deduplication  
- compression  
- RAID‑Z  

Loomine:

```
zpool create tank /dev/sdb
```

---

# 6. Mounting

## 6.1 Failisüsteemi mountimine

```
mount /dev/sdb1 /mnt
```

## 6.2 Unmount

```
umount /mnt
```

## 6.3 /etc/fstab

Näide:

```
/dev/sdb1  /data  ext4  defaults  0  2
```

---

# 7. tmpfs (RAM‑põhine)

```
mount -t tmpfs -o size=2G tmpfs /mnt/ramdisk
```

Kasutatakse:
- cache  
- build temp  
- Kubernetes pod tmpfs  

---

# 8. Journaling

EXT4 ja XFS kasutavad journalingut:

- metadata journaling  
- full data journaling (EXT4)  

See vähendab failisüsteemi korruptsiooni.

---

# 9. Filesystem tuning

EXT4:

```
tune2fs -l /dev/sdb1
```

XFS:

```
xfs_info /dev/sdb1
```

---

# 10. Best Practices

- EXT4 → üldine kasutus  
- XFS → serverid, logid, suured failid  
- Btrfs → snapshotid, dev/test  
- ZFS → enterprise storage  
- Kasuta `fstab` automaatseks mountimiseks  
- Kasuta `fsck` ainult siis, kui failisüsteem on unmounted  

---

# Kokkuvõte
Linuxi failisüsteemid on paindlikud ja võimsad.  
EXT4, XFS, Btrfs ja ZFS katavad kõik kasutusstsenaariumid alates desktopist kuni enterprise storage’ini.
```
