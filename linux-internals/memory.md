# 📄 **Fail 2: `linux-internals/memory.md`**

```markdown
# Linux Memory – Linux Internals käsiraamat  
## (RAM, Swap, Paging, OOM, HugePages)

## Ülevaade
Linuxi mäluhaldus on üks maailma kõige arenenumaid.  
See kasutab:

- virtuaalmälu (virtual memory)
- lehekülgede haldust (paging)
- page cache’i
- slab allocator’it
- NUMA arhitektuuri
- OOM killer’it
- swap’i
- hugepages’e

---

# 1. Virtuaalmälu (Virtual Memory)

Igal protsessil on **oma virtuaalne aadressiruum**:

- stack  
- heap  
- text (kood)  
- shared libraries  
- mmap failid  

Virtuaalmälu → kernel mapib selle füüsilisele RAM‑ile.

---

# 2. Page Cache

Linux kasutab vaba RAM‑i **failisüsteemi cache’ina**.

Vaata:

```
free -h
```

`buff/cache` näitab page cache’i.

Page cache kiirendab:

- failide lugemist  
- failide kirjutamist  
- I/O operatsioone  

---

# 3. Swap

Swap on RAM‑i laiendus kettal.

## 3.1 Swap kasutuse vaatamine

```
swapon -s
free -h
```

## 3.2 Swap faili loomine

```bash
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

---

# 4. OOM Killer (Out Of Memory)

Kui RAM + swap saavad täis, käivitub **OOM killer**.

Vaata logisid:

```
dmesg | grep -i oom
```

OOM valib protsessi, mis:

- kasutab palju mälu  
- on madala prioriteediga  
- ei ole süsteemi jaoks kriitiline  

---

# 5. Memory cgroups

Konteinerid kasutavad cgroup’e mälu piiramiseks.

Näide:

```
docker run --memory=512m nginx
```

Vaata:

```
cat /sys/fs/cgroup/memory/memory.usage_in_bytes
```

---

# 6. NUMA (Non‑Uniform Memory Access)

Mitme CPU socketiga serverites on RAM jagatud **NUMA sõlmedeks**.

Vaata:

```
numactl --hardware
```

NUMA‑aware rakendused (DPDK, databases) töötavad kiiremini.

---

# 7. HugePages

HugePages vähendavad TLB misses’e ja parandavad jõudlust.

## 7.1 Kontrolli

```
cat /proc/meminfo | grep -i huge
```

## 7.2 Loo hugepages

```
echo 2048 > /proc/sys/vm/nr_hugepages
```

## 7.3 Mount

```
mount -t hugetlbfs nodev /mnt/huge
```

Kasutavad:
- DPDK  
- PostgreSQL  
- Oracle DB  
- virtuaalmasinad  

---

# 8. Memory leak diagnoosimine

## 8.1 top / htop
Vaata protsessi RSS kasvu.

## 8.2 /proc/<pid>/smaps
Detailne memory map.

## 8.3 valgrind
C/C++ programmide leakide leidmiseks.

---

# 9. Kernel memory allocators

Linux kasutab:

- **SLAB**
- **SLUB** (vaikimisi)
- **SLOB** (embedded)

Vaata:

```
cat /sys/kernel/slab
```

---

# 10. Best Practices

- Ära paanitse, kui RAM on “täis” — page cache on hea asi  
- Kasuta swap’i minimaalselt serverites  
- Kasuta cgroup’e konteinerite piiramiseks  
- Kasuta hugepages’e jõudluskriitilistes rakendustes  
- Jälgi OOM killeri logisid  
- NUMA‑aware paigutus annab suurt jõudlust  

---

# Kokkuvõte
Linuxi mäluhaldus on võimas ja paindlik.  
Virtuaalmälu, page cache, swap, OOM, NUMA ja hugepages on kriitilised mõisted iga DevOps/SRE jaoks.
```
