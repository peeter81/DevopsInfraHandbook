# 📄 **Fail 11: `linux-internals/namespaces.md`**

```markdown
# Linux Namespaces – Linux Internals käsiraamat  
## (pid, net, mount, uts, ipc, user, cgroup)

## Ülevaade
Namespaces on Linuxi mehhanism, mis isoleerib protsessid üksteisest.  
See on konteinerite (Docker, LXC, Kubernetes) **põhitehnoloogia**.

Iga namespace loob *oma lokaalse reaalsuse*:
- oma protsessitabel  
- oma võrgustik  
- oma failisüsteem  
- oma hostinimi  
- oma kasutajad  
- oma IPC ressursid  

---

# 1. Namespaces tüübid

| Namespace | Failisüsteemi nimi | Eesmärk |
|----------|--------------------|---------|
| pid      | pid                | protsesside isolatsioon |
| net      | net                | võrguliideste isolatsioon |
| mount    | mnt                | mount‑punktide isolatsioon |
| uts      | uts                | hostname ja domainname |
| ipc      | ipc                | shared memory, semaphores |
| user     | user               | UID/GID map |
| cgroup   | cgroup             | ressursipiirangud |

---

# 2. Namespace loomine käsitsi

## 2.1 Uus pid namespace

```
unshare -p -f --mount-proc bash
```

Uues shellis:

```
ps aux
```

Näed ainult *enda* protsesse.

---

# 3. Network namespace

## 3.1 Loo uus netns

```
ip netns add testns
```

## 3.2 Käivita käsk namespace’is

```
ip netns exec testns ip link
```

## 3.3 Lisa virtuaalne veth paar

```
ip link add veth0 type veth peer name veth1
ip link set veth1 netns testns
```

---

# 4. Mount namespace

Iga konteiner saab oma mount‑puu.

Näide:

```
unshare -m bash
mount -t tmpfs tmpfs /mnt
```

See mount ei mõjuta hosti.

---

# 5. UTS namespace

Muuda hostname’i:

```
unshare -u bash
hostname container1
```

Hosti hostname ei muutu.

---

# 6. IPC namespace

Isolatsioon:
- shared memory  
- message queues  
- semaphores  

Näide:

```
unshare -i bash
```

---

# 7. User namespace

User namespace võimaldab:
- hostis **mitte‑root**  
- konteineris **root**

Näide:

```
unshare -U bash
```

Vaata UID map:

```
cat /proc/self/uid_map
```

---

# 8. Cgroup namespace

Isolatsioon:
- CPU  
- RAM  
- I/O  
- PID limiidid  

Näide:

```
unshare -C bash
```

---

# 9. Namespaces konteinerites

Docker kasutab:

| Namespace | Kasutus |
|----------|---------|
| pid      | konteineri protsessid |
| net      | virtuaalne võrk |
| mount    | overlayfs |
| uts      | hostname |
| ipc      | shm |
| user     | rootless containers |
| cgroup   | limiidid |

Kubernetes kasutab sama mehhanismi iga podi jaoks.

---

# 10. Kontrolli namespace’e

Vaata protsessi namespace linke:

```
ls -l /proc/<pid>/ns
```

Näed:

```
ipc  mnt  net  pid  user  uts
```

---

# 11. Best Practices

- Kasuta user namespace’e rootless konteinerite jaoks  
- Kasuta netns testimiseks (routing, firewall, VPN)  
- Kasuta mount namespace’e chroot alternatiivina  
- Ära sega IPC namespace’e mitme konteineri vahel  
- Kasuta pid namespace’i turvalisuse suurendamiseks  

---

# Kokkuvõte
Namespaces on Linuxi konteinerite alus.  
Need võimaldavad isoleerida protsesse, võrke, mount‑punkte, kasutajaid ja ressursse — luues turvalise ja kergekaalulise virtualiseerimise mudeli.
```
