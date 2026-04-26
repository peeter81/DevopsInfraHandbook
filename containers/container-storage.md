# 📄 **Fail: `containers/container-storage.md`**

```markdown
# Container Storage – Containers käsiraamat  
## (Volumes, bind mounts, overlayfs, CSI, persistent storage, Kubernetes)

## Ülevaade
Konteinerid on **ephemeral** — nende failisüsteem kaob koos konteineriga.  
Seetõttu on püsiv salvestus kriitiline:

- andmebaasid  
- logid  
- konfiguratsioon  
- kasutajaandmed  

See fail katab Docker, containerd/CRI‑O ja Kubernetes storage’i.

---

# 1. Docker Storage

## 1.1 Bind mount (host → container)

Konteiner kasutab hosti kataloogi:

```bash
docker run -v /host/data:/data nginx
```

Plussid:
- lihtne  
- kiire  
- hea arenduseks  

Miinused:
- sõltub hosti failisüsteemist  
- pole kaasaskantav  

---

## 1.2 Docker volumes

Docker haldab ise storage’t:

```bash
docker volume create data
docker run -v data:/var/lib/mysql mysql
```

Vaata:

```bash
docker volume ls
docker volume inspect data
```

Plussid:
- kaasaskantav  
- turvalisem  
- parem tootmises  

---

## 1.3 tmpfs (RAM storage)

```bash
docker run --tmpfs /cache nginx
```

Kasutus:
- caching  
- kiire, kuid mitte püsiv  

---

# 2. containerd / CRI‑O storage

Konteinerite storage asub:

```
/var/lib/containerd/
/var/lib/containers/storage/
```

Snapshotterid:
- overlayfs (vaikimisi)  
- btrfs  
- zfs  

Vaata containerd storage’t:

```bash
nerdctl system df
```

---

# 3. OverlayFS

Kõik konteineriplatvormid kasutavad overlayfs‑i:

- **lowerdir** – image kihid  
- **upperdir** – konteineri kirjutuskihid  
- **merged** – lõplik failisüsteem  

See võimaldab:
- väikseid image’eid  
- kiireid build’e  
- copy‑on‑write  

---

# 4. Kubernetes Storage

Kubernetes kasutab **PersistentVolume (PV)** ja **PersistentVolumeClaim (PVC)** mehhanismi.

---

## 4.1 PersistentVolumeClaim (PVC)

Näide:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

---

## 4.2 Kasutamine podis

```yaml
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data

volumeMounts:
  - mountPath: /var/lib/mysql
    name: data
```

---

# 5. StorageClass

StorageClass määrab, kuidas PV luuakse.

Näide (AWS EBS):

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```

PVC kasutab seda:

```yaml
storageClassName: gp2
```

---

# 6. CSI (Container Storage Interface)

CSI on standard, mis võimaldab kasutada mis tahes storage’i:

- Ceph RBD  
- NFS  
- iSCSI  
- AWS EBS  
- Azure Disk  
- Google Persistent Disk  
- Longhorn  
- OpenEBS  

CSI draiverid töötavad podidena.

---

# 7. StatefulSet storage

StatefulSet loob **unikaalsed PVC‑d** iga podi jaoks:

```
data-myapp-0
data-myapp-1
data-myapp-2
```

Sobib:
- PostgreSQL  
- MySQL  
- Kafka  
- ElasticSearch  

---

# 8. NFS

Lihtne ja universaalne storage:

```yaml
nfs:
  server: 192.168.1.10
  path: /data
```

Plussid:
- lihtne  
- odav  

Miinused:
- aeglane  
- pole ideaalne andmebaasidele  

---

# 9. Local Persistent Volumes

Kasutab hosti ketast:

```yaml
hostPath:
  path: /mnt/data
```

Plussid:
- kiire  

Miinused:
- pole HA  
- pod peab jooksma samal node’il  

---

# 10. Troubleshooting

### 10.1 PVC stuck in Pending

Kontrolli StorageClass:

```bash
kubectl get storageclass
```

### 10.2 Pod ei saa mountida

```bash
kubectl describe pod <nimi>
```

### 10.3 NFS permission denied

Kontrolli:

```bash
chmod -R 777 /data
```

---

# 11. Best Practices

- Kasuta **StorageClass** automaatseks provisioninguks  
- Kasuta **CSI** tootmises  
- Ära kasuta hostPath tootmises  
- Kasuta **StatefulSet** andmebaaside jaoks  
- Kasuta **ReadWriteMany** (RWX) ainult NFS/Gluster/Longhorn  
- Kasuta **backup** lahendusi (Velero, Restic)  

---

# Kokkuvõte
Container storage hõlmab Docker volumesid, containerd snapshottereid, Kubernetes PVC/PV mehhanismi ja CSI draivereid.  
Õige storage lahendus sõltub töökoormusest, jõudlusest ja kõrgest kättesaadavusest.
```
