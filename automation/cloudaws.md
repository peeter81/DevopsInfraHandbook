# 📄 **Fail: `cloud/aws.md`**

```markdown
# AWS – Cloud käsiraamat  
## (EC2, VPC, IAM, S3, RDS, ELB, Autoscaling, CloudWatch)

## Ülevaade
Amazon Web Services (AWS) on maailma suurim pilveplatvorm.  
See fail annab ülevaate kõige olulisematest teenustest, mida kasutatakse DevOps, infrastruktuuri ja automatiseerimise töövoogudes.

---

# 1. EC2 (Virtual Machines)

EC2 = virtuaalserverid AWS-is.

### Olulised mõisted:
- **AMI** – serveri image  
- **Instance type** – riistvara (t2.micro, t3.medium, m5.large)  
- **Security Group** – tulemüür  
- **Key Pair** – SSH võti  
- **User Data** – cloud-init skript  

### EC2 loomine CLI-ga:

```bash
aws ec2 run-instances \
  --image-id ami-123456 \
  --instance-type t3.micro \
  --key-name mykey \
  --security-group-ids sg-123 \
  --subnet-id subnet-123
```

---

# 2. VPC (Virtual Private Cloud)

VPC = privaatne virtuaalvõrk AWS-is.

### Komponendid:
- **Subnets** (public/private)  
- **Route Tables**  
- **Internet Gateway**  
- **NAT Gateway**  
- **Security Groups**  
- **Network ACLs**  

### Tüüpiline arhitektuur:

```
VPC
 ├── Public Subnet (EC2, Load Balancer)
 └── Private Subnet (EC2, RDS)
```

---

# 3. IAM (Identity and Access Management)

IAM haldab kasutajaid, rolle ja õigusi.

### Olulised mõisted:
- **User** – inimene  
- **Role** – teenus või server  
- **Policy** – JSON õiguste komplekt  
- **MFA** – kahefaktoriline autentimine  

Näide IAM poliitikast:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::mybucket"]
    }
  ]
}
```

---

# 4. S3 (Object Storage)

S3 = odav ja skaleeruv objektisalvestus.

### Kasutusalad:
- backup  
- logid  
- staatilised veebilehed  
- artefaktid  
- big data  

CLI:

```bash
aws s3 cp file.txt s3://mybucket/
```

---

# 5. RDS (Managed Databases)

RDS haldab andmebaase:

- PostgreSQL  
- MySQL  
- MariaDB  
- SQL Server  
- Oracle  
- Aurora  

### Eelised:
- automaatne backup  
- automaatne patching  
- Multi-AZ kõrge saadavus  
- read replicas  

---

# 6. Load Balancers (ELB)

Tüübid:

- **ALB** – Application Load Balancer (HTTP/HTTPS)  
- **NLB** – Network Load Balancer (TCP/UDP)  
- **CLB** – Classic Load Balancer (vana)  

ALB näide:

```
client → ALB → target group → EC2 instances
```

---

# 7. Auto Scaling

Auto Scaling Group (ASG) skaleerib EC2 servereid automaatselt.

### Reeglid:
- CPU > 70% → lisa server  
- CPU < 30% → eemalda server  

---

# 8. CloudWatch

CloudWatch = monitooring + logid.

### Funktsioonid:
- Metrics  
- Logs  
- Alarms  
- Dashboards  

Näide alarmist:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name HighCPU \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --period 300 \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:...
```

---

# 9. Lambda (Serverless Functions)

Lambda käivitab koodi ilma serverit haldamata.

### Näide:

```python
def handler(event, context):
    return "Hello from Lambda"
```

---

# 10. CloudFormation (IaC)

CloudFormation = AWS-i natiivne IaC.

Näide:

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
```

---

# 11. Best Practices

- Kasuta **IAM rolle**, mitte IAM kasutajaid serverites  
- Kasuta **Security Group** reegleid, mitte NACL-e  
- Pane EC2 serverid **private subnet**’isse  
- Kasuta **ALB** HTTP/HTTPS jaoks  
- Kasuta **S3** logide ja backup’ide jaoks  
- Kasuta **RDS Multi-AZ** tootmises  
- Kasuta **CloudWatch alarms** monitooringuks  
- Kasuta **Terraform** või **CloudFormation** IaC jaoks  

---

# Kokkuvõte
AWS pakub täielikku pilveplatvormi, mis katab compute, storage, võrgu, andmebaasid, monitooringu ja automatiseerimise.  
EC2 + VPC + IAM + S3 + RDS + ALB + CloudWatch = tüüpiline tootmiskeskkond.
```
