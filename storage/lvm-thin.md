# 📄 **Fail: `storage/lvm-thin.md`**

```markdown
# LVM Thin Provisioning – Storage käsiraamat  
## (Thin pools, thin volumes, snapshots, monitoring)

## Ülevaade
LVM Thin Provisioning võimaldab luua loogilisi volume’e, mis kasutavad füüsilist ruumi **dünaamiliselt**.  
See tähendab:

- ruumi kasutatakse ainult siis, kui andmed päriselt kirjutatakse  
- saab luua palju volume’e ilma kohe kettaruumi kulutamata  
- snapshotid on kiired ja ruumisäästlikud  
- sobib VM‑idele, konteineritele, testkeskkondadele  

---

# 1. Mõisted

### **Thin pool**
Füüsiline ruum, kust kõik thin volume’id ruumi võtavad.

### **Thin volume**
Loogiline volume, mis kasutab ruumi ainult kirjutatud andmete ulatuses.

### **Metadata volume**
Hoidla, kus LVM jälgib block mappingut.

---

# 2. Thin pool loomine

## 2.1 Loo Physical Volume ja Volume Group

```bash
pvcreate /dev/sdb
vgcreate vgthin /dev/sdb
```

## 2.2 Loo thin pool

```bash
lvcreate -L 100G -T vgthin/thinpool
```

`-T` → loob thin pooli  
`100G` → thin pooli suurus

Vaata:

```bash
lvs -a
```

---

# 3. Thin volume loomine

```bash
lvcreate -V 20G -T vgthin/thinpool -n thinvol1
```

Selgitus:
- `-V 20G` → virtuaalne suurus (kui palju kasutaja näeb)
- tegelik ruumikasutus alguses ~0

Format:

```bash
mkfs.ext4 /dev/vgthin/thinvol1
```

Mount:

```bash
mount /dev/vgthin/thinvol1 /mnt
```

---

# 4. Thin snapshotid

Thin snapshotid on **kohesed** ja **ruumisäästlikud**.

Loo snapshot:

```bash
lvcreate -s -n snap1 -V 20G vgthin/thinpool/thinvol1
```

Snapshotid kasutavad ainult delta-andmeid.

---

# 5. Thin pooli jälgimine

Vaata kasutust:

```bash
lvs -o+seg_monitor
```

Detailne:

```bash
lvs -a -o lv_name,lv_size,data_percent,metadata_percent vgthin
```

Oluline on jälgida:
- **data_percent**  
- **metadata_percent**

Kui metadata täitub → pool muutub **read‑only**.

---

# 6. Thin pooli laiendamine

## 6.1 Laienda data ruumi

```bash
lvextend -L +50G vgthin/thinpool
```

## 6.2 Laienda metadata ruumi

```bash
lvextend -L +1G vgthin/thinpool_tmeta
```

---

# 7. Thin pooli taastamine (corruption recovery)

Kui metadata saab kahjustada:

```bash
lvconvert --repair vgthin/thinpool
```

---

# 8. Thin provisioning riskid

### **1. Overprovisioning**
Kui virtuaalne ruum > füüsiline ruum, võib pool täis saada.

### **2. Metadata täitumine**
Halvem kui data täitumine — pool muutub read‑only.

### **3. Snapshotide liigne kasutamine**
Liiga palju snapshotte → metadata kasvab kiiresti.

---

# 9. Best Practices

- Jälgi alati `data_percent` ja `metadata_percent`  
- Kasuta monitoringut (Prometheus node_exporter LVM metrics)  
- Ära lase metadata’l täis saada  
- Kasuta thin provisioningut VM‑ide ja konteinerite jaoks  
- Ära kasuta thin pooli kriitilistes andmebaasides ilma hea monitoringuta  
- Tee regulaarselt snapshotte, kuid ära hoia neid liiga kaua  

---

# Kokkuvõte
LVM Thin Provisioning on võimas ja paindlik tehnoloogia, mis võimaldab dünaamilist ruumikasutust, kiireid snapshotte ja suurt paindlikkust.  
Õige monitoring ja planeerimine on kriitilised, et vältida data või metadata täitumist.
```
