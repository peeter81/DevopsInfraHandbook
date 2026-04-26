# 📄 **Fail: `automation/packer.md`**

```markdown
# Packer – Automation käsiraamat  
## (Golden Images, Builders, Provisioners, Variables, Templates, HCP Packer)

## Ülevaade
Packer on HashiCorp’i tööriist, mis loob **automaatseid serveri- ja VM‑image’e**:

- AWS AMI  
- Azure Image  
- Google Cloud Image  
- VMware  
- VirtualBox  
- Docker image  
- Proxmox (pluginiga)

Packer võimaldab luua **golden image’e**, mida kasutatakse CI/CD, Kubernetes node’ide, VM‑ide ja serverite standardiseerimiseks.

---

# 1. Packer põhimõte

Packer töötab kolmes etapis:

1. **Builder** – loob VM/instance’i  
2. **Provisioner** – seadistab selle (Ansible, shell, jne)  
3. **Artifact** – salvestab lõpliku image’i  

---

# 2. Packer install

Linux:

```bash
sudo apt install packer
```

Windows:

```powershell
choco install packer
```

---

# 3. Packer template struktuur

Packer kasutab JSON või HCL2 formaati (soovitatav).

Näide `ubuntu.pkr.hcl`:

```hcl
variable "aws_region" {
  default = "eu-west-1"
}

source "amazon-ebs" "ubuntu" {
  region        = var.aws_region
  instance_type = "t3.micro"
  ami_name      = "ubuntu-golden-{{timestamp}}"
  source_ami_filter {
    filters = {
      name                = "ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    owners      = ["099720109477"]
    most_recent = true
  }
}

build {
  name    = "ubuntu-golden"
  sources = ["source.amazon-ebs.ubuntu"]

  provisioner "shell" {
    inline = [
      "apt update",
      "apt install -y nginx",
      "systemctl enable nginx"
    ]
  }
}
```

---

# 4. Packer käsud

## 4.1 Kontrolli süntaksit

```bash
packer validate ubuntu.pkr.hcl
```

## 4.2 Build

```bash
packer build ubuntu.pkr.hcl
```

## 4.3 Muutujate kasutamine

```bash
packer build -var "aws_region=eu-central-1" ubuntu.pkr.hcl
```

---

# 5. Provisioners

### 5.1 Shell provisioner

```hcl
provisioner "shell" {
  script = "setup.sh"
}
```

### 5.2 Ansible provisioner

```hcl
provisioner "ansible" {
  playbook_file = "playbook.yml"
}
```

### 5.3 File provisioner

```hcl
provisioner "file" {
  source      = "config.cfg"
  destination = "/tmp/config.cfg"
}
```

---

# 6. Builders

Levinumad builderid:

- **amazon-ebs** – AWS AMI  
- **azure-arm** – Azure image  
- **googlecompute** – GCP image  
- **vmware-iso** – VMware VM  
- **virtualbox-iso** – VirtualBox VM  
- **docker** – Docker image  

Näide Docker builder:

```hcl
source "docker" "nginx" {
  image  = "ubuntu:20.04"
  commit = true
}
```

---

# 7. Variables

```hcl
variable "instance_type" {
  default = "t3.micro"
}
```

Kasutamine:

```hcl
instance_type = var.instance_type
```

---

# 8. HCP Packer (registry)

HashiCorp Cloud Platform võimaldab:

- image versioonihaldust  
- automaatset deprecate’imist  
- CI/CD integratsiooni  

Näide:

```hcl
hcp_packer_registry {
  bucket_name = "golden-images"
}
```

---

# 9. Packer + Terraform

Packer loob image’i → Terraform kasutab seda:

Terraform:

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

# 10. Best Practices

- Kasuta **HCL2**, mitte JSON  
- Kasuta **variables** paindlikkuseks  
- Kasuta **Ansible** keerukamate seadistuste jaoks  
- Kasuta **HCP Packer** image versioonihalduseks  
- Ära pane paroole template’i (kasuta Vault)  
- Tee image’id **immutably** (ära tee runtime konfiguratsiooni)  

---

# Kokkuvõte
Packer loob automaatselt standardiseeritud serveri‑ ja VM‑image’e.  
See on ideaalne golden image’ide, CI/CD ja infrastruktuuri automatiseerimise jaoks.
```

