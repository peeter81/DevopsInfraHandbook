# 📄 **Fail 12: `linux-internals/cgroups.md`**

```markdown
# Linux Cgroups – Linux Internals käsiraamat  
## (CPU, Memory, I/O, PIDs, cgroup v1 vs v2, systemd)

## Ülevaade
Control Groups (cgroups) võimaldavad Linuxil piirata ja jälgida protsesside ressursikasutust:

- CPU  
- RAM  
- I/O  
- PID arv  
- Network (tc + cgroups)  

Cgroups on konteinerite (Docker, Kubernetes, LXC) **põhitehnoloogia** koos namespaces’idega.

---

# 1. Cgroup v1 vs v2

## Cgroup v1
- iga ressurss oma hierarhias  
- keerulisem  
- Docker algselt kasutas v1  

## Cgroup v2
- üks ühtne hierarhia  
- lihtsam  
- parem accounting  
- Kubernetes ja systemd eelistavad v2  

Kontrolli versiooni:

```
ls /sys/fs/cgroup
```

Kui näed `cgroup.controllers` → v2.

---

# 2. Cgroups systemd kaudu

Iga systemd teenus töötab oma cgroup’is.

Vaata teenuse cgroup’i:

```
systemctl status nginx
```

Vaata ressursipiiranguid:

```
systemctl show nginx | grep -i cgroup
```

---

# 3. CPU piiramine

## 3.1 systemd teenusele

```
systemctl set-property nginx.service CPUQuota=50%
```

See piirab teenuse CPU kasutuse 50% peale.

---

# 4. Memory piiramine

```
systemctl set-property nginx.service MemoryMax=500M
```

Vaata:

```
cat /sys/fs/cgroup/system.slice/nginx.service/memory.current
```

---

# 5. I/O piiramine

```
systemctl set-property nginx.service IOReadBandwidthMax=/dev/sda 10M
systemctl set-property nginx.service IOWriteBandwidthMax=/dev/sda 5M
```

---

# 6. PIDs piiramine

Piira protsesside arvu:

```
systemctl set-property nginx.service TasksMax=200
```

---

# 7. Cgroups käsitsi (advanced)

## 7.1 Loo cgroup

```
mkdir /sys/fs/cgroup/mygroup
```

## 7.2 Piira CPU

```
echo 50000 > /sys/fs/cgroup/mygroup/cpu.max
```

## 7.3 Lisa protsess

```
echo <pid> > /sys/fs/cgroup/mygroup/cgroup.procs
```

---

# 8. Docker ja cgroups

Docker kasutab cgroup’e:

- `--cpus=1.5`  
- `--memory=512m`  
- `--pids-limit=100`  

Näide:

```
docker run --cpus=1 --memory=256m nginx
```

---

# 9. Kubernetes ja cgroups

Kubernetes kasutab cgroup’e podide piiramiseks:

```
resources:
  limits:
    cpu: "2"
    memory: "1Gi"
  requests:
    cpu: "500m"
    memory: "256Mi"
```

Kubelet töötab systemd cgroup driver’iga.

---

# 10. Cgroup tööriistad

## 10.1 systemd-cgls

Näita cgroup’i puu:

```
systemd-cgls
```

## 10.2 systemd-cgtop

Reaalajas cgroup ressursikasutus:

```
systemd-cgtop
```

---

# 11. Best Practices

- Kasuta cgroup v2 uutes süsteemides  
- Kasuta systemd property’sid teenuste piiramiseks  
- Ära anna konteineritele piiramatuid ressursse  
- Kasuta CPUQuota ja MemoryMax serverite stabiliseerimiseks  
- Kuberneteses kasuta alati resource requests/limits  

---

# Kokkuvõte
Cgroups on Linuxi ressursihalduse alus.  
Need võimaldavad piirata CPU, RAM, I/O ja PID kasutust — ning on kriitilised konteinerite, serverite ja Kubernetes klastrite töökindluse tagamiseks.
```
