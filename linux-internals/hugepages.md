**Siin on täielik _linux-internals/hugepages.md_ dokumentatsioonifail, mis põhineb Linuxi ametlikul kernelidokumentatsioonil.**  Kõik tehnilised faktid on viidatud Linux Kernel Archives’i dokumentatsioonile. [The Linux Kernel Archives](https://www.kernel.org/doc/html/latest/admin-guide/mm/hugetlbpage.html) [The Linux Kernel Archives](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt)

---

# hugepages.md

## Linux Huge Pages (HugeTLB & THP)

Hugepages on Linuxi mehhanism, mis võimaldab kasutada tavapärasest suuremaid mälulehti, et vähendada TLB (Translation Lookaside Buffer) koormust ja parandada jõudlust suurte mälumahtudega rakendustes.

Linux toetab kahte mehhanismi:

- **HugeTLB / hugetlbfs** – staatiline, admini poolt ette reserveeritud hugepage’id
- **Transparent Huge Pages (THP)** – dünaamiline, kernel haldab automaatselt

---

## 1. Miks hugepages?

**TLB on väga piiratud ressurss**, mis salvestab virtuaal- ja füüsilise mälu vastendusi.  
Suuremad lehed → vähem TLB kirjeid → vähem TLB miss’e → **kiirem mälu ligipääs**.  
See on eriti oluline:

- andmebaasid (PostgreSQL, Oracle, MySQL)
- virtualiseerimine (KVM, nested paging)
- HPC ja teadusarvutused
- suured in-memory rakendused (Redis, JVM, Elasticsearch)

Linuxi arhitektuurid toetavad erinevaid lehesuurusi, nt x86: **4K, 2M, 1G**. [The Linux Kernel Archives](https://www.kernel.org/doc/html/latest/admin-guide/mm/hugetlbpage.html)

---

## 2. HugeTLB (hugetlbfs)

### 2.1. Eeldused kernelis

Kernel peab olema kompileeritud valikutega:

- `CONFIG_HUGETLBFS`
- `CONFIG_HUGETLB_PAGE` (aktiveerub automaatselt) [The Linux Kernel Archives](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt)

### 2.2. Kontroll, kas hugetlbfs on saadaval

```bash
cat /proc/filesystems | grep hugetlbfs
```

### 2.3. Staatiliste hugepage’ide haldus

#### Kontrolli süsteemi seisu

```bash
cat /proc/meminfo | grep -i huge
```

Näited väljadest:

- **HugePages_Total** – reserveeritud hugepage’ide koguarv
- **HugePages_Free** – vabad hugepage’id
- **HugePages_Rsvd** – reserveeritud, kuid veel mitte allokeeritud
- **HugePages_Surp** – üleliigsed hugepage’id
- **Hugepagesize** – lehe suurus (nt 2048 kB)
- **Hugetlb** – kogu tarbitud mälu hugepage’ide poolt [The Linux Kernel Archives](https://www.kernel.org/doc/html/latest/admin-guide/mm/hugetlbpage.html)

#### Reserveeri hugepage’e

```bash
echo 128 > /proc/sys/vm/nr_hugepages
```

#### Püsiv seadistus `/etc/sysctl.conf`

```
vm.nr_hugepages = 128
```

### 2.4. hugetlbfs mountimine

```bash
mkdir /mnt/huge
mount -t hugetlbfs none /mnt/huge
```

### 2.5. Kasutamine rakendustes

Rakendused saavad hugepage’e kasutada:

- `mmap()` kaudu
- SysV shared memory (`shmget`, `shmat`) kaudu [The Linux Kernel Archives](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt)

---

## 3. Transparent Huge Pages (THP)

THP on automaatne mehhanism, mis proovib tavalisi 4K lehti koondada suuremateks (nt 2M).  
Eelis: ei vaja eelnevat reserveerimist.  
Puudus: võib tekitada latentsuspiike (page fault → 2M puhastamine).

### 3.1. Kontrolli THP olekut

```bash
cat /sys/kernel/mm/transparent_hugepage/enabled
```

### 3.2. Režiimid

- **always** – kernel proovib alati kasutada THP
- **madvise** – ainult kui rakendus küsib
- **never** – THP keelatud

### 3.3. THP eelised

- vähem page fault’e
- vähem TLB miss’e
- parem jõudlus virtualiseerimisel (KVM + nested paging)
- toetab ka multi-size THP (16K, 32K, 64K jne) [The Linux Kernel Archives](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html)

---

## 4. Võrdlus: HugeTLB vs THP

|Omadus|HugeTLB|THP|
|---|---|---|
|Reserveerimine|Staatiline, admin määrab|Dünaamiline|
|Jõudlus|Kõrgeim, stabiilne|Hea, kuid võib kõikuda|
|Sobib|DB, HPC, KVM|Üldkasutus, JVM, desktop|
|Lehesuurus|2M, 1G (kindel)|2M (PMD), mTHP 16K–64K|
|Failisüsteem|hugetlbfs|tmpfs/shmem, anon mälu|
|Page fault overhead|Väike|Võib olla suur (2M clear)|

---

## 5. Parimad praktikad

### Kui kasutada HugeTLB?

- PostgreSQL / Oracle / MySQL
- Redis / Memcached
- KVM guest memory
- Latentsustundlikud HPC rakendused

### Kui kasutada THP?

- JVM rakendused (Elasticsearch, Kafka, Spark)
- Üldine serverikasutus
- Kui ei ole vaja rangeid garantiisid

### Soovitused

- Ära kasuta THP-d andmebaasidega → kasuta HugeTLB
- JVM puhul kasuta `madvise` režiimi
- Virtualiseerimisel kasuta 1G hugepage’e, kui võimalik

---

Kui soovid, võin selle faili lisada ka sinu olemasolevasse _linux-internals_ sisukorda või genereerida järgmise teema.