# 📄 **Fail: `containers/kubernetes-storage.md`**

```markdown
# Kubernetes Storage – Containers käsiraamat  
## (PV, PVC, StorageClass, CSI, StatefulSets, AccessModes, VolumeModes)

## Ülevaade
Kubernetes pakub paindlikku ja laiendatavat salvestusmudelit, mis sobib nii lihtsate kui ka keerukate töökoormuste jaoks.

Peamised komponendid:

- **PersistentVolume (PV)**
- **PersistentVolumeClaim (PVC)**
- **StorageClass**
- **CSI draiverid**
- **StatefulSet storage**
- **AccessModes**
- **VolumeModes**

---

# 1. PersistentVolume (PV)

PV on klastris olev püsiv salvestusruum.

Näide:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

---

# 2. PersistentVolumeClaim (PVC)

Rakendused ei kasuta PV‑sid otse — nad taotlevad PVC kaudu.

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

# 3. PVC kasutamine podis

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

# 4. StorageClass

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

# 5. CSI (Container Storage Interface)

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

# 6. AccessModes

| AccessMode | Kirjeldus |
|------------|-----------|
| **ReadWriteOnce (RWO)** | ainult üks node saab kirjutada |
| **ReadOnlyMany (ROX)** | mitu node’i saavad lugeda |
| **ReadWriteMany (RWX)** | mitu node’i saavad kirjutada |

RWX vajab tavaliselt:

- NFS  
- CephFS  
- Longhorn  
- GlusterFS  

---

# 7. VolumeModes

| VolumeMode | Kirjeldus |
|------------|-----------|
| **Filesystem** | mountitakse failisüsteemina |
| **Block** | toorblokk, ilma FS‑ita |

Block sobib:

- andmebaasidele  
- kõrge jõudlusega töökoormustele  

---

# 8. StatefulSet storage

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

# 9. Dynamic Provisioning

Kui PVC kasutab StorageClass’i, loob Kubernetes PV automaatselt.

Kontrolli:

```bash
kubectl get pv
kubectl get pvc
```

---

# 10. NFS

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

# 11. Troubleshooting

### 11.1 PVC stuck in Pending

Kontrolli StorageClass:

```bash
kubectl get storageclass
```

### 11.2 Pod ei saa mountida

```bash
kubectl describe pod <nimi>
```

### 11.3 NFS permission denied

```bash
chmod -R 777 /data
```

---

# 12. Best Practices

- Kasuta **StorageClass** automaatseks provisioninguks  
- Kasuta **CSI** tootmises  
- Ära kasuta hostPath tootmises  
- Kasuta **StatefulSet** andmebaaside jaoks  
- Kasuta **RWX** ainult NFS/Longhorn/CephFS puhul  
- Tee regulaarselt backup’e (Velero, Restic)  

---

# Kokkuvõte
Kubernetes storage põhineb PV/PVC mudelil, StorageClass’il ja CSI draiveritel.  
See võimaldab luua skaleeritavaid ja töökindlaid salvestuslahendusi igasugustele töökoormustele.
```
