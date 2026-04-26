# 📄 **Fail: `storage/lvm-cache.md`**

```markdown
# LVM Cache – Storage käsiraamat  
## (SSD cache HDD‑dele, cache modes, cache pool, performance)

## Ülevaade
LVM Cache võimaldab kasutada **SSD‑d HDD kiirendamiseks**.  
See töötab nii:

- HDD → aeglane, suur maht  
- SSD → kiire, väike maht  
- LVM cache → SSD toimib HDD ees cache kihina  

Sobib:
- VM hostidele  
- andmebaasidele  
- failiserveritele  
- suurtele LVM volume’idele  

---

# 1. Mõisted

### **Cache pool**
SSD‑st loodud LVM pool, mis hoiab cache block’e.

### **Origin volume**
HDD‑l olev aeglane LVM volume, mida kiirendatakse.

### **Cache mode**
- **writethrough** – ohutu, aeglasem  
- **writeback** – kiire, riskantsem (andmed võivad olla ainult SSD‑l)  
- **passthrough** – cache on välja lülitatud  

---

# 2. Eeldused

- üks SSD (või NVMe) → cache  
- üks HDD → origin  
- mõlemad LVM PV‑d

Näide:

```
/dev/sda → HDD
/dev/nvme0n1 → SSD
```

---

# 3. Cache pool loomine

## 3.1 Loo PV ja VG

```bash
pvcreate /dev/sda /dev/nvme0n1
vgcreate vgcache /dev/sda /dev/nvme0n1
```

## 3.2 Loo origin volume (HDD)

```bash
lvcreate -L 500G -n data vgcache /dev/sda
```

## 3.3 Loo cache pool (SSD)

```bash
lvcreate -L 50G -n cachepool vgcache /dev/nvme0n1
lvconvert --type cache-pool --poolmetadata vgcache/cachepool vgcache/cachepool
```

---

# 4. Lisa cache origin volume’ile

```bash
lvconvert --type cache --cachepool vgcache/cachepool vgcache/data
```

Vaata:

```bash
lvs -a -o +devices
```

---

# 5. Cache mode muutmine

## 5.1 writethrough (ohutu)

```bash
lvchange --cachemode writethrough vgcache/data
```

## 5.2 writeback (kiire)

```bash
lvchange --cachemode writeback vgcache/data
```

## 5.3 passthrough (cache välja lülitatud)

```bash
lvchange --cachemode passthrough vgcache/data
```

---

# 6. Cache statistika

Vaata:

```bash
lvs -o lv_name,cache_total_blocks,cache_used_blocks,cache_dirty_blocks vgcache
```

Detailne:

```bash
dmsetup status /dev/mapper/vgcache-data
```

---

# 7. Cache eemaldamine

```bash
lvconvert --uncache vgcache/data
```

Cache pool jääb alles, kuid origin töötab ilma cache’ita.

---

# 8. Cache pooli laiendamine

```bash
lvextend -L +20G vgcache/cachepool
```

---

# 9. Best Practices

- Kasuta **writethrough** serverites, kus andmekadu ei ole lubatud  
- Kasuta **writeback** test- ja dev‑keskkondades  
- Ära kasuta liiga väikest cache pooli (<10% origin mahust)  
- Kasuta NVMe SSD‑sid parema jõudluse jaoks  
- Jälgi cache dirty block’e writeback režiimis  
- Kasuta monitoringut (Prometheus node_exporter LVM metrics)  

---

# Kokkuvõte
LVM Cache võimaldab kasutada SSD‑d HDD kiirendamiseks, pakkudes suurt jõudluse kasvu minimaalse kuluga.  
Õige cache mode ja piisav SSD maht on kriitilised stabiilsuse ja jõudluse tagamiseks.
```
