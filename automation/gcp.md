# 📄 **Fail: `cloud/gcp.md`**

```markdown
# Google Cloud Platform (GCP) – Cloud käsiraamat  
## (Compute Engine, VPC, IAM, Cloud Storage, Cloud SQL, GKE, Monitoring)

## Ülevaade
Google Cloud Platform (GCP) on Google’i pilveplatvorm, mis on eriti tugev:

- Kubernetes (GKE)  
- masinõpe  
- globaalsed võrgud  
- skaleeritav storage  
- DevOps tööriistad  

See fail annab ülevaate kõige olulisematest teenustest.

---

# 1. Compute Engine (VM-id)

Compute Engine = virtuaalserverid GCP-s.

### Olulised mõisted:
- **Machine type** (e2-medium, n2-standard-2)  
- **Image** (Ubuntu, Container-Optimized OS)  
- **Firewall rules**  
- **Service Account**  
- **Startup Script** (cloud-init)  

VM loomine CLI-ga:

```bash
gcloud compute instances create web01 \
  --zone=europe-west1-b \
  --machine-type=e2-medium \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud
```

---

# 2. VPC (Virtual Private Cloud)

GCP VPC on **globaalse ulatusega**, erinevalt AWS/Azure regionaalsetest VPC-dest.

Komponendid:

- **Subnets** (regionaalsed)  
- **Firewall rules**  
- **Routes**  
- **VPC Peering**  
- **Cloud NAT**  

VPC loomine:

```bash
gcloud compute networks create prod-vpc --subnet-mode=custom
```

Subnet:

```bash
gcloud compute networks subnets create web \
  --network=prod-vpc \
  --range=10.0.1.0/24 \
  --region=europe-west1
```

---

# 3. IAM (Identity and Access Management)

IAM haldab kasutajaid, rolle ja teenusekontosid.

### Olulised mõisted:
- **User**  
- **Service Account**  
- **Role** (primitive, predefined, custom)  
- **Policy**  

Näide rolli andmisest:

```bash
gcloud projects add-iam-policy-binding myproject \
  --member="user:peeter@example.com" \
  --role="roles/viewer"
```

---

# 4. Cloud Storage (GCS)

GCS = Google’i objektisalvestus (S3 analoog).

Bucket loomine:

```bash
gcloud storage buckets create gs://mybucket --location=europe-west1
```

Faili upload:

```bash
gcloud storage cp file.txt gs://mybucket
```

---

# 5. Cloud SQL (Managed Databases)

Cloud SQL toetab:

- PostgreSQL  
- MySQL  
- SQL Server  

Eelised:

- automaatne backup  
- automaatne patching  
- kõrge saadavus (HA)  
- read replicas  

---

# 6. GKE (Google Kubernetes Engine)

GKE on maailma kõige küpsem hallatud Kubernetes.

Klaster:

```bash
gcloud container clusters create prod-aks \
  --num-nodes=3 \
  --zone=europe-west1-b
```

Kubeconfig:

```bash
gcloud container clusters get-credentials prod-aks --zone=europe-west1-b
```

---

# 7. Load Balancing

GCP Load Balancer on **globaalne**, mitte regionaalne.

Tüübid:

- **HTTP(S) Global Load Balancer**  
- **TCP/UDP Load Balancer**  
- **Internal Load Balancer**  

---

# 8. Cloud Logging & Monitoring

Stackdriver (Cloud Operations Suite):

- Metrics  
- Logs  
- Alerts  
- Dashboards  

Logide vaatamine:

```bash
gcloud logging read "resource.type=gce_instance" --limit=20
```

---

# 9. Cloud Functions (Serverless)

Cloud Functions = serverless funktsioonid.

Näide:

```python
def hello(request):
    return "Hello from GCP"
```

Deploy:

```bash
gcloud functions deploy hello --runtime python311 --trigger-http
```

---

# 10. Best Practices

- Kasuta **Service Accounts**, mitte kasutajakontosid  
- Pane VM-id **private subnet**’isse + Cloud NAT  
- Kasuta **GKE** konteinerite jaoks  
- Kasuta **Cloud Storage** logide ja backup’ide jaoks  
- Kasuta **IAM minimal permissions** põhimõtet  
- Kasuta **Terraform** GCP haldamiseks  
- Kasuta **Cloud Monitoring** alertide jaoks  

---

# Kokkuvõte
GCP on tugev Kubernetes’e, globaalse võrgu ja DevOps tööriistade poolest.  
Compute Engine + VPC + IAM + GCS + Cloud SQL + GKE = tüüpiline tootmiskeskkond GCP-s.
```
