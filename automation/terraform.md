# 📄 **Fail: `automation/terraform.md`**

```markdown
# Terraform – Automation käsiraamat  
## (IaC, Providers, Resources, Variables, Modules, State, Workspaces)

## Ülevaade
Terraform on HashiCorp’i **Infrastructure as Code (IaC)** tööriist, mis võimaldab:

- pilve infrastruktuuri loomist (AWS, Azure, GCP)  
- võrgu, VM‑ide, konteinerite, load balancer’ite haldamist  
- korduvkasutatavaid moduleid  
- deklaratiivset konfiguratsiooni  
- muutuste planeerimist (terraform plan)  
- state’i haldust  

Terraform on DevOps standard.

---

# 1. Terraform install

Linux:

```bash
sudo apt install terraform
```

Windows:

```powershell
choco install terraform
```

---

# 2. Terraform projektistruktuur

```
main.tf
variables.tf
outputs.tf
terraform.tfvars
modules/
```

---

# 3. Provider

Näide AWS providerist:

```hcl
provider "aws" {
  region = "eu-west-1"
}
```

---

# 4. Resource

Näide EC2 instantsi loomisest:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t3.micro"
}
```

---

# 5. Variables

`variables.tf`:

```hcl
variable "instance_type" {
  default = "t3.micro"
}
```

Kasutamine:

```hcl
instance_type = var.instance_type
```

`terraform.tfvars`:

```hcl
instance_type = "t3.small"
```

---

# 6. Outputs

```hcl
output "public_ip" {
  value = aws_instance.web.public_ip
}
```

Vaata:

```bash
terraform output
```

---

# 7. Terraform käsud

## 7.1 Init

```bash
terraform init
```

## 7.2 Plan

```bash
terraform plan
```

## 7.3 Apply

```bash
terraform apply
```

## 7.4 Destroy

```bash
terraform destroy
```

---

# 8. State

Terraform salvestab infrastruktuuri seisu:

```
terraform.tfstate
```

Soovitatav: **remote state** (S3 + DynamoDB lock):

```hcl
backend "s3" {
  bucket = "tf-state"
  key    = "prod/terraform.tfstate"
  region = "eu-west-1"
  dynamodb_table = "tf-lock"
}
```

---

# 9. Modules

Module = korduvkasutatav Terraformi üksus.

Struktuur:

```
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
```

Kasutamine:

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}
```

---

# 10. Data sources

Loe olemasolevat infot:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal*"]
  }
}
```

---

# 11. Workspaces

Erinevad keskkonnad:

```bash
terraform workspace new dev
terraform workspace new prod
terraform workspace list
```

---

# 12. Terraform + Packer

Packer loob image’i → Terraform kasutab seda:

```hcl
data "aws_ami" "golden" {
  most_recent = true
  owners      = ["self"]
  filter {
    name   = "name"
    values = ["ubuntu-golden-*"]
  }
}
```

---

# 13. Best Practices

- Kasuta **remote state** (S3 + DynamoDB)  
- Kasuta **modules** korduvkasutuseks  
- Ära hoia paroole Git’is (kasuta Vault)  
- Kasuta **terraform fmt** ja **terraform validate**  
- Kasuta **variables** paindlikkuseks  
- Kasuta **plan** enne iga apply’d  
- Versioonihalda providerite versioone  

---

# Kokkuvõte
Terraform on võimas IaC tööriist, mis võimaldab hallata kogu infrastruktuuri deklaratiivselt ja korduvkasutatavalt.  
Modules + State + Plan = stabiilne ja kontrollitud infrastruktuur.
```
