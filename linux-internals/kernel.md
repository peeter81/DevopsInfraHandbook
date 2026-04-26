# 📄 **Fail 4: `linux-internals/kernel.md`**

```markdown
# Linux Kernel – Linux Internals käsiraamat  
## (Kernel space, Syscalls, Modules, Scheduling, Boot process)

## Ülevaade
Linux kernel on operatsioonisüsteemi süda.  
See vastutab:

- protsesside halduse eest  
- mälu halduse eest  
- failisüsteemide eest  
- võrgustiku eest  
- riistvara draiverite eest  
- turvalisuse eest  

Kernel töötab **kernel space’is**, rakendused **user space’is**.

---

# 1. Kernel space vs user space

## User space
- tavalised programmid  
- piiratud õigused  
- ei pääse otse riistvarale  

## Kernel space
- täielik ligipääs riistvarale  
- kernel, draiverid, scheduler  
- ohtlik, kui midagi läheb valesti  

---

# 2. System calls (syscalls)

User space → kernel space suhtlus toimub **syscall’ide** kaudu.

Näited:
- `read()`  
- `write()`  
- `open()`  
- `fork()`  
- `exec()`  
- `socket()`  

Vaata protsessi syscall’e:

```
strace ls
```

---

# 3. Kernel modules

Kernelit saab laiendada **moodulitega** (kooslaaditavad draiverid).

## 3.1 Lae moodul

```
modprobe vfio-pci
```

## 3.2 Eemalda moodul

```
modprobe -r vfio-pci
```

## 3.3 Laetud moodulid

```
lsmod
```

---

# 4. Kernel boot process

1. BIOS/UEFI  
2. Bootloader (GRUB)  
3. Kernel laadimine  
4. initramfs  
5. systemd (PID 1)  
6. teenuste käivitamine  

Vaata kernel logi:

```
dmesg
```

---

# 5. Kernel parameters

Kernel parameetreid saab anda GRUB-is:

`/etc/default/grub`:

```
GRUB_CMDLINE_LINUX="quiet splash intel_iommu=on iommu=pt"
```

Uuenda:

```
update-grub
```

---

# 6. Kernel scheduling

Linux kasutab:
- **CFS** (Completely Fair Scheduler) – vaikimisi  
- **FIFO** – real-time  
- **RR** – real-time  
- **Deadline** – RT süsteemidele  

Vaata:

```
chrt -p <pid>
```

---

# 7. Kernel memory management

Kernel haldab:
- page cache  
- slab allocator  
- virtual memory  
- NUMA  
- hugepages  

Vaata:

```
cat /proc/meminfo
```

---

# 8. Kernel networking stack

Linuxi võrgupinu sisaldab:
- netfilter  
- conntrack  
- routing  
- TCP/IP stack  
- XDP/eBPF  
- NIC draiverid  

Vaata:

```
ss -tuna
```

---

# 9. Kernel security

Linux kasutab:
- SELinux  
- AppArmor  
- seccomp  
- capabilities  
- namespaces  

Vaata protsessi capabilities:

```
capsh --print
```

---

# 10. Kernel buildimine (advanced)

Lae kernel:

```
git clone `https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git` [(git.kernel.org in Bing)](https://www.bing.com/search?q="https%3A%2F%2Fgit.kernel.org%2Fpub%2Fscm%2Flinux%2Fkernel%2Fgit%2Ftorvalds%2Flinux.git")
```

Konfigureeri:

```
make menuconfig
```

Kompileeri:

```
make -j$(nproc)
make modules_install
make install
```

---

# 11. Best Practices

- Ära kompileeri kernelit, kui pole vajadust  
- Kasuta LTS kerneli tootmises  
- Kasuta kernel mooduleid draiverite jaoks  
- Kasuta `dmesg` riistvara probleemide diagnoosimiseks  
- Kasuta sysctl’i kernel parameetrite muutmiseks  

---

# Kokkuvõte
Linux kernel on süsteemi süda.  
Syscall’id, moodulid, scheduler, mälu, võrgupinu ja turvamehhanismid moodustavad võimsa ja paindliku arhitektuuri, mis sobib serveritele, pilvele ja Kubernetes klastritele.
```
