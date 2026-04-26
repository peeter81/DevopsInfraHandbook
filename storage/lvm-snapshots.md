# 📄 **Fail: `storage/lvm-snapshots.md`**

```markdown
# LVM Snapshots – Storage käsiraamat  
## (Copy-on-write, thin snapshots, merge, restore, performance)

## Ülevaade
LVM snapshotid võimaldavad luua hetkeseisu koopiaid loogilistest volume’idest.  
Neid kasutatakse:

- backup  
- süsteemi uuenduste testimine  
- kiire rollback  
- VM‑ide ja konteinerite checkpointid  

LVM toetab kahte tüüpi snapshotte:

1. **Classic COW snapshots** (paks snapshot)  
2. **Thin snapshots** (thin pooli sees, väga efektiivsed)

---

# 1. Classic LVM snapshots (COW)

Classic snapshotid kasutavad **copy‑on‑write** mehhanismi:

- originaalblokk kirjutatakse snapshoti alale enne muutmist  
- snapshoti maht peab olema piisav muutuste jaoks  
- snapshot aeglustab kirjutamist (COW overhead)

---

# 2. Classic snapshoti loomine

```bash
lvcreate -L 5G -s -n snap1 /dev/vgdata/lvdata
```

Selgitus:
- `-s` → snapshot
- `5G` → ruum muutuste jaoks, mitte kogu LV suurus

Vaata:

```bash
lvs -a
```

---

# 3. Snapshoti mountimine

```bash
mount /dev/vgdata/snap1 /mnt
```

Snapshot on **read‑only**, kui ei kasutata `--permission rw`.

---

# 4. Snapshoti taastamine (rollback)

Rollback kirjutab snapshoti sisu tagasi originaalile.

```bash
lvconvert --merge /dev/vgdata/snap1
```

Pärast merge’i:
- reboot (kui LV on root)
- või unmount + remount (mitte-root LV puhul)

---

# 5. Snapshoti kustutamine

```bash
lvremove /dev/vgdata/snap1
```

---

# 6. Thin snapshots (thin pool)

Thin snapshots on:

- kohesed  
- ruumisäästlikud  
- väga kiired  
- sobivad VM‑idele ja konteineritele  

Loo thin snapshot:

```bash
lvcreate -s -n snapthin1 vgthin/thinvol1
```

Vaata:

```bash
lvs -a -o lv_name,origin,lv_size,data_percent,metadata_percent
```

---

# 7. Thin snapshoti taastamine

Rollback:

```bash
lvconvert --merge vgthin/snapthin1
```

Toimib ainult siis, kui origin pole vahepeal uuesti kirjutatud.

---

# 8. Snapshoti jõudlus

### Classic snapshots
- aeglustavad kirjutamist  
- sobivad väikesteks backupideks  
- mitte soovitatavad pikaajaliseks kasutuseks

### Thin snapshots
- väga kiire  
- minimaalne overhead  
- sobivad VM‑idele, konteineritele, CI/CD‑le  
- sõltuvad thin pooli metadata mahust

---

# 9. Snapshoti täitumine (classic)

Kui snapshot täitub → **kogu LV muutub read‑only**.

Kontrolli:

```bash
lvs -o lv_name,lv_size,data_percent vgdata
```

---

# 10. Best Practices

- Kasuta **thin snapshots** alati, kui võimalik  
- Ära hoia classic snapshotte pikalt  
- Jälgi thin pooli `metadata_percent`  
- Kasuta snapshotte enne süsteemi uuendusi  
- Kasuta snapshotte VM‑ide backupiks  
- Ära tee snapshotte täis ketastega (COW overhead)  

---

# Kokkuvõte
LVM snapshotid on võimas tööriist kiirete backupide, rollbackide ja testkeskkondade jaoks.  
Thin snapshots on tänapäevane standard — kiired, ruumisäästlikud ja ideaalsed virtualiseerimise jaoks.
```
