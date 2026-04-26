# 📄 **Fail: `containers/kubernetes-jobs.md`**

```markdown
# Kubernetes Jobs – Containers käsiraamat  
## (Jobs, CronJobs, completions, parallelism, backoffLimit, TTL, cleanup)

## Ülevaade
Kubernetes **Job** on töö, mis peab lõppema edukalt.  
Erinevalt Deployment’ist ei hoia Job pod’e pidevalt töös — see käivitab need, ootab lõpetamist ja lõpetab.

Kasutusjuhtumid:

- backup  
- batch processing  
- migratsioonid  
- failide töötlemine  
- cron-tööd (CronJob)

---

# 1. Job

Lihtne Job, mis käivitab ühe korra käsu:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
        - name: hello
          image: busybox
          command: ["echo", "Hello from Job"]
      restartPolicy: Never
```

Loo:

```bash
kubectl apply -f job.yaml
```

Vaata:

```bash
kubectl get jobs
kubectl get pods
```

---

# 2. completions & parallelism

## 2.1 completions  
Mitu korda töö peab edukalt lõppema.

```yaml
spec:
  completions: 5
```

## 2.2 parallelism  
Mitu pod’i võib korraga töötada.

```yaml
spec:
  parallelism: 2
```

Näide: 5 tööd, 2 paralleelselt.

---

# 3. backoffLimit

Mitu korda Kubernetes proovib uuesti, kui pod crashib.

```yaml
spec:
  backoffLimit: 3
```

Vaikimisi: **6**.

---

# 4. activeDeadlineSeconds

Aeg, mille jooksul Job peab lõppema.

```yaml
spec:
  activeDeadlineSeconds: 300
```

Kui aeg täis → Job lõpetatakse.

---

# 5. TTL (auto cleanup)

TTL kustutab Job’i ja selle pod’id pärast lõpetamist.

```yaml
spec:
  ttlSecondsAfterFinished: 60
```

---

# 6. CronJob

CronJob käivitab Job’e ajastatult.

Näide: käivita iga päev kell 03:00.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly
spec:
  schedule: "0 3 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: alpine
              command: ["sh", "-c", "echo backup"]
          restartPolicy: Never
```

Vaata:

```bash
kubectl get cronjobs
```

---

# 7. CronJob ajastuse süntaks

| Sümbol | Tähendus |
|--------|----------|
| * | iga väärtus |
| */5 | iga 5 minuti tagant |
| 0 3 * * * | iga päev kell 03:00 |
| 0 */2 * * * | iga 2 tunni tagant |
| 0 3 * * 1 | iga esmaspäev kell 03:00 |

---

# 8. CronJob concurrencyPolicy

## 8.1 Allow (vaikimisi)  
Lubab mitut paralleelset Job’i.

```yaml
concurrencyPolicy: Allow
```

## 8.2 Forbid  
Ei käivita uut, kui eelmine pole lõpetanud.

```yaml
concurrencyPolicy: Forbid
```

## 8.3 Replace  
Tühistab vana ja käivitab uue.

```yaml
concurrencyPolicy: Replace
```

---

# 9. CronJob startingDeadlineSeconds

Kui CronJob jäi käivitamata (nt node oli maas), siis:

```yaml
startingDeadlineSeconds: 200
```

Kubernetes proovib seda hiljem käivitada.

---

# 10. Troubleshooting

### 10.1 Job stuck in "BackoffLimitExceeded"

Kontrolli pod’i logisid:

```bash
kubectl logs <pod>
```

### 10.2 CronJob ei käivitu

Kontrolli ajastust:

```bash
kubectl describe cronjob <nimi>
```

### 10.3 Job ei lõpeta

- kas command jääb ootama?  
- kas restartPolicy on vale?  

---

# 11. Best Practices

- Kasuta **restartPolicy: Never** Job’ide puhul  
- Kasuta **ttlSecondsAfterFinished** cleanup’iks  
- Kasuta **Forbid** concurrencyPolicy, kui töö ei tohi kattuda  
- Kasuta **activeDeadlineSeconds** runaway job’ide vältimiseks  
- Kasuta CronJob’e ainult ajastatud töödeks, mitte pidevaks tööks  

---

# Kokkuvõte
Jobs ja CronJobs võimaldavad Kubernetes’is käivitada ühekordseid ja ajastatud töökoormusi.  
Need on hädavajalikud backup’ide, batch-töötluse ja regulaarsete hoolduste jaoks.
```
