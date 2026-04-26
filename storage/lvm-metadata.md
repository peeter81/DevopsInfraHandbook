# 📄 **Fail: `storage/lvm-metadata.md`**

```markdown
# LVM Metadata – Storage käsiraamat  
## (Metadata role, corruption, backup, restore, thin pool metadata)

## Ülevaade
LVM kasutab metadata’t, et hoida infot:

- PV, VG ja LV struktuuri kohta  
- segmentide ja extents’ide kohta  
- RAID, thin poolide ja cache konfiguratsiooni kohta  
- snapshotide ja mappingute kohta  

Metadata on **kriitiline** — kui see saab kahjustada, võib kogu LVM struktuur muutuda kasutamatuks.

---

# 1. Kus LVM metadata asub?

### PV metadata
Iga Physical Volume sisaldab metadata’t:

```
pvdisplay -m
```

### VG metadata
Volume Group metadata hoitakse:

- PV alguses  
- /etc/lvm/backup/ kataloogis (automaatsed koopiad)

### Thin pool metadata
Thin poolidel on eraldi metadata LV:

```
vgthin/thinpool_tmeta
```

---

# 2. Metadata varukoopiad

LVM teeb automaatselt varukoopiaid:

```
/etc/lvm/backup/<vgname>
/etc/lvm/archive/<timestamped files>
```

Vaata:

```bash
ls /etc/lvm/backup
ls /etc/lvm/archive
```

---

# 3. Metadata käsitsi backup

```bash
vgcfgbackup vgdata
```

See loob faili:

```
/etc/lvm/backup/vgdata
```

---

# 4. Metadata taastamine

Kui VG metadata saab kahjustada:

```bash
vgcfgrestore vgdata
```

Võid valida konkreetse arhiveeritud versiooni:

```bash
vgcfgrestore -f /etc/lvm/archive/vgdata_00012-123456.vg vgdata
```

---

# 5. Thin pool metadata

Thin poolidel on **data** ja **metadata** LV:

Näide:

```
lvs -a -o lv_name,lv_size,metadata_percent,data_percent vgthin
```

Metadata täitumine on kriitiline:

- kui metadata täitub → **thin pool muutub read‑only**
- kui metadata saab kahjustada → kogu thin pool võib muutuda kasutamatuks

---

# 6. Thin pool metadata laiendamine

```bash
lvextend -L +1G vgthin/thinpool_tmeta
```

Pärast seda:

```bash
lvconvert --repair vgthin/thinpool
```

---

# 7. Metadata corruption recovery

### 7.1 Kontrolli metadata’t

```bash
lvconvert --repair vgthin/thinpool
```

### 7.2 Taasta metadata varukoopiast

```bash
vgcfgrestore vgthin
```

### 7.3 Kui metadata LV on katki

Loo uus metadata LV:

```bash
lvcreate -L 1G -n thinpool_tmeta_new vgthin
lvconvert --thinpool vgthin/thinpool --poolmetadata vgthin/thinpool_tmeta_new
```

---

# 8. Metadata ja snapshotid

Classic snapshotid kasutavad metadata’t COW block mappingute jaoks.

Thin snapshots kasutavad thin pool metadata’t:

- iga snapshot lisab metadata koormust  
- liiga palju snapshotte → metadata täitub kiiresti  

---

# 9. Best Practices

- Tee regulaarselt `vgcfgbackup`  
- Hoia /etc/lvm/backup ja /etc/lvm/archive turvalises kohas  
- Jälgi thin pooli `metadata_percent`  
- Ära lase metadata’l täis saada  
- Kasuta kiiret SSD-d thin pool metadata jaoks  
- Ära hoia liiga palju snapshotte korraga  

---

# Kokkuvõte
LVM metadata on kogu LVM struktuuri alus.  
Selle korruptsioon võib muuta kõik LV-d kasutamatuks, seega on varukoopiad ja regulaarne jälgimine kriitilise tähtsusega — eriti thin poolide puhul.
```
