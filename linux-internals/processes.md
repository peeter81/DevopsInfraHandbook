```markdown
# Linux Processes – Linux Internals käsiraamat

## Ülevaade
Linuxis on kõik protsessid kas:
- **process** (klassikaline)
- **thread** (kergekaaluline protsess)
- **kernel thread** (kthread)

Iga protsessil on:
- PID (process ID)
- PPID (parent PID)
- UID (kasutaja ID)
- GID (grupi ID)
- nice väärtus
- scheduler policy
- memory map
- file descriptors

---

# 1. Protsesside vaatamine

## 1.1 ps
```
ps aux
ps -ef
ps -eo pid,ppid,cmd,%cpu,%mem
```

## 1.2 top / htop
Reaalajas protsesside jälgimine.

## 1.3 pstree
```
pstree -p
```
Näitab protsesside hierarhiat.

---

# 2. Protsessi elutsükkel

1. fork() – luuakse koopia
2. exec() – laaditakse uus programm
3. run() – protsess töötab
4. exit() – protsess lõpetab
5. wait() – parent korjab exit-koodi

---

# 3. Zombie ja Orphan protsessid

## Zombie
- Protsess on lõpetanud, kuid parent pole `wait()` teinud.
- Nähtav kui `Z` top-is.

## Orphan
- Parent sureb enne child’i.
- Child võtab üle `systemd` (PID 1).

---

# 4. Protsesside tapmine

```
kill -9 <pid>     # SIGKILL
kill -15 <pid>    # SIGTERM
killall nginx
pkill -f python
```

---

# 5. Nice ja renice

## Prioriteedi muutmine
```
nice -n 10 ./script.sh
renice -n -5 -p 1234
```

Vahemik:
- nice: -20 (kõrge prioriteet) kuni +19 (madal)

---

# 6. Schedulerid

Linux toetab:
- **CFS** (Completely Fair Scheduler) – vaikimisi
- **FIFO** – real-time
- **RR** – real-time round-robin
- **Deadline** – RT süsteemidele

Kontroll:
```
chrt -p <pid>
```

---

# 7. File Descriptors

Igal protsessil on FD tabel:
- 0 – stdin
- 1 – stdout
- 2 – stderr

Vaata:
```
ls -l /proc/<pid>/fd
```

---

# 8. Memory map

```
cat /proc/<pid>/maps
```

Näitab:
- heap
- stack
- shared libraries
- mmap failid

---

# 9. Protsessi namespace’id

Linux kasutab konteinerite jaoks:
- pid namespace
- net namespace
- mount namespace
- ipc namespace
- uts namespace
- user namespace

Näide:
```
unshare -p -f --mount-proc bash
```

---

# 10. Best Practices

- Kasuta `htop` protsesside diagnoosimiseks
- Väldi `kill -9` kui võimalik
- Kontrolli zombie protsesse (võib viidata bugile)
- Kasuta namespaces testimiseks ja konteinerite mõistmiseks
- Jälgi scheduler policy’t RT süsteemides

---

# Kokkuvõte
Linuxi protsessimudel on paindlik ja võimas.  
Protsesside, schedulerite, namespace’ide ja FD‑de mõistmine on kriitiline DevOps, SRE ja süsteemiadministraatori töö jaoks.
```
