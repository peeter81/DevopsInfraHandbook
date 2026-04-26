# 📄 **Fail 6: `linux-internals/storage.md`**

```markdown
# Linux Storage – Linux Internals käsiraamat  
## (Block devices, LVM, RAID, I/O schedulers, udev)

## Ülevaade
Linuxi storage‑arhitektuur koosneb:
- block device’idest  
- partitsioonidest  
- failisüsteemidest  
- LVM‑ist  
- RAID‑ist  
- I/O scheduleritest  
- udev seadmepuudest  

Storage on üks olulisemaid Linuxi administreerimise valdkondi.

---

# 1. Block devices

Vaata block device’e:

```
lsblk
```

Näed:
- disk (nt /dev/sda)
- part (nt /dev/sda1)
- lvm (nt /dev/mapper/vg0-lv0)

Detailne info:

```
blkid
```

---

# 2. Partitsioonid

Loo uus partitsioon:

```
fdisk /dev/sdb
```

GPT jaoks:

```
gdisk /dev/sdb
```

---

# 3. LVM (Logical Volume Manager)

LVM võimaldab:
- dünaamilist storage’i  
- snapshot’e  
- laiendamist ja vähendamist  
- RAID tasemeid  

## 3.1 Physical Volume (PV)

```
pvcreate /dev/sdb1
```

## 3.2 Volume Group (VG)

```
vgcreate vgdata /dev/sdb1
```

## 3.3 Logical Volume (LV)

```
lvcreate -L 50G -n lvdata vgdata
```

## 3.4 LV laiendamine

```
lvextend -L +10G /dev/vgdata/lvdata
resize2fs /dev/vgdata/lvdata
```

---

# 4. RAID (mdadm)

Linuxi tarkvaraline RAID.

## 4.1 RAID1 loomine

```
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

## 4.2 RAID staatuse vaatamine

```
cat /proc/mdstat
```

---

# 5. I/O schedulers

Linux toetab mitut I/O schedulerit:

- **none** – NVMe jaoks  
- **mq-deadline** – serverid  
- **bfq** – desktop  
- **kyber** – madal latentsus  

Vaata:

```
cat /sys/block/sda/queue/scheduler
```

Muuda:

```
echo mq-deadline > /sys/block/sda/queue/scheduler
```

---

# 6. udev

udev loob automaatselt device failid `/dev` all.

Vaata seadmepuud:

```
udevadm info /dev/sda
```

Trigger:

```
udevadm trigger
```

---

# 7. Storage performance tööriistad

## 7.1 fio (benchmark)

```
fio --name=test --size=1G --rw=randread --bs=4k --numjobs=4
```

## 7.2 iostat

```
iostat -x 1
```

## 7.3 dstat

```
dstat -d
```

---

# 8. Multipath (SAN storage)

Multipath võimaldab:
- mitut tee’d sama LUN‑ini  
- failover  
- load balancing  

Vaata:

```
multipath -ll
```

---

# 9. Best Practices

- Kasuta LVM‑i paindlikkuse jaoks  
- Kasuta RAID1/10 kriitiliste andmete jaoks  
- Kasuta `mq-deadline` serverites  
- Kasuta `fio` jõudlustestideks  
- Dokumenteeri storage’i topoloogia  
- Ära kasuta RAID5/6 väikeste SSD‑dega (write hole risk)  

---

# Kokkuvõte
Linuxi storage‑arhitektuur on paindlik ja võimas.  
LVM, RAID, I/O schedulerid ja udev moodustavad tugeva aluse serverite ja pilvekeskkondade storage’i haldamiseks.
```
