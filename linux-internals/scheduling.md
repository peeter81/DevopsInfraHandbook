# 📄 **Fail 10: `linux-internals/scheduling.md`**

```markdown
# Linux Scheduling – Linux Internals käsiraamat  
## (CFS, vruntime, runqueue, load balancing, preemption, affinity)

## Ülevaade
Linuxi scheduler otsustab, milline protsess saab CPU‑aega.  
Vaikimisi scheduler on **CFS – Completely Fair Scheduler**, mis püüab anda igale protsessile “õiglase” osa CPU‑st.

Scheduler töötab:
- igal CPU‑l eraldi  
- kasutades runqueue’sid  
- kasutades vruntime’i  
- tehes load balancingut  
- toetades preemption’it  
- arvestades CPU affinity’t  

---

# 1. Runqueue

Igal CPU‑l on oma **runqueue**:

- runnable protsessid  
- sorted red‑black tree  
- kõige väiksema vruntime’iga protsess valitakse esimesena  

Vaata CPU runqueue infot:

```
cat /proc/sched_debug
```

---

# 2. CFS (Completely Fair Scheduler)

CFS ei kasuta time‑slice’e.  
Selle asemel kasutab ta **vruntime’i**.

### CFS eesmärk:
> Iga protsess saab võrdselt CPU aega, sõltuvalt nice väärtusest.

---

# 3. vruntime (virtual runtime)

Igal protsessil on **virtuaalne runtime**, mis kasvab:

- kiiremini → kui protsess on madala prioriteediga  
- aeglasemalt → kui protsess on kõrge prioriteediga  

Scheduler valib **kõige väiksema vruntime’iga** protsessi.

---

# 4. Red‑Black Tree

CFS hoiab runnable protsesse **RB‑tree’s**:

- O(log n) insert  
- O(log n) delete  
- O(log n) next task  

Kõige vasakpoolsem element → järgmine protsess.

---

# 5. Preemption

Linux on **preemptive multitasking** OS.

Kernel võib protsessi katkestada, kui:
- saabub IRQ  
- kõrgema prioriteediga protsess muutub runnable’iks  
- scheduler tick toimub  

Vaata preemption mode:

```
grep PREEMPT /boot/config-$(uname -r)
```

---

# 6. Load balancing

Kui üks CPU on ülekoormatud, scheduler:

- liigutab protsesse teisele CPU‑le  
- arvestab NUMA topoloogiat  
- arvestab cache locality’t  

Vaata:

```
cat /proc/schedstat
```

---

# 7. CPU affinity

Affinity määrab, millistel CPU‑del protsess võib töötada.

Vaata:

```
taskset -p <pid>
```

Muuda:

```
taskset -cp 0,1 <pid>
```

---

# 8. Nice ja priority

Nice väärtus mõjutab vruntime’i kasvu.

Vahemik:
- **-20** → kõrge prioriteet  
- **+19** → madal prioriteet  

Muuda:

```
renice -n -5 -p 1234
```

---

# 9. Real‑time schedulerid

Linux toetab RT scheduler’eid:

- **SCHED_FIFO**  
- **SCHED_RR**  
- **SCHED_DEADLINE**  

Vaata:

```
chrt -p <pid>
```

Muuda:

```
chrt -f -p 1 <pid>
```

---

# 10. Scheduler debugging

## 10.1 sched_debug

```
cat /proc/sched_debug
```

## 10.2 ftrace

```
echo function > /sys/kernel/debug/tracing/current_tracer
```

## 10.3 perf sched

```
perf sched record
perf sched latency
```

---

# 11. Best Practices

- Ära kasuta RT schedulerit ilma vajaduseta  
- Kasuta CPU affinity’t jõudluskriitilistes rakendustes  
- Jälgi runqueue pikkust (`/proc/sched_debug`)  
- Kasuta `perf sched` latency analüüsiks  
- Kasuta nice väärtusi batch‑tööde jaoks  

---

# Kokkuvõte
Linuxi scheduler on keerukas ja võimas.  
CFS, vruntime, runqueue, preemption ja load balancing tagavad, et protsessid saavad CPU‑aega õiglaselt ja efektiivselt.  
Scheduler’i mõistmine on kriitiline jõudluse optimeerimisel ja süsteemi diagnoosimisel.
```
