# 📄 **Fail: `cloud/azure.md`**

```markdown
# Azure – Cloud käsiraamat  
## (VMs, VNets, Resource Groups, Storage, AKS, IAM, Monitoring)

## Ülevaade
Microsoft Azure on üks maailma suurimaid pilveplatvorme.  
See fail annab ülevaate kõige olulisematest teenustest, mida kasutatakse DevOps, infrastruktuuri ja automatiseerimise töövoogudes.

---

# 1. Resource Groups

Resource Group = loogiline konteiner Azure’i ressurssidele.

Näited:

- VM-id  
- VNet  
- Storage kontod  
- Databases  
- AKS klaster  

CLI:

```bash
az group create --name prod-rg --location westeurope
```

---

# 2. Azure Virtual Machines (VMs)

Azure VM = virtuaalserver pilves.

### Olulised mõisted:
- **Image** (Ubuntu, Windows, custom)  
- **Size** (B2s, D2s_v3, F4s)  
- **NIC**  
- **Public IP**  
- **Network Security Group (NSG)**  
- **Managed Disks**  
- **User Data / cloud-init**  

VM loomine:

```bash
az vm create \
  --resource-group prod-rg \
  --name web01 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username peeter \
  --generate-ssh-keys
```

---

# 3. Virtual Network (VNet)

Azure VNet = privaatne virtuaalvõrk.

Komponendid:

- **Subnets**  
- **Route Tables**  
- **NSG** (tulemüür)  
- **VPN Gateway**  
- **Peering**  

Näide:

```bash
az network vnet create \
  --resource-group prod-rg \
  --name prod-vnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name web \
  --subnet-prefix 10.0.1.0/24
```

---

# 4. Network Security Groups (NSG)

NSG = tulemüür reeglitega.

Näide:

```bash
az network nsg rule create \
  --resource-group prod-rg \
  --nsg-name web-nsg \
  --name allow-http \
  --protocol tcp \
  --priority 100 \
  --destination-port-range 80 \
  --access allow
```

---

# 5. Azure Storage

Azure Storage tüübid:

- **Blob Storage** – objektisalvestus  
- **File Storage** – SMB jagamine  
- **Queue Storage** – sõnumijärjekorrad  
- **Table Storage** – NoSQL tabelid  

Blob upload:

```bash
az storage blob upload \
  --account-name mystorage \
  --container-name backups \
  --file db.sql
```

---

# 6. Azure SQL

Azure SQL on hallatud SQL Server.

Eelised:

- automaatne backup  
- automaatne patching  
- kõrge saadavus  
- skaleerimine  

---

# 7. Azure Kubernetes Service (AKS)

AKS = hallatud Kubernetes klaster.

Loomine:

```bash
az aks create \
  --resource-group prod-rg \
  --name prod-aks \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3
```

Kubeconfig:

```bash
az aks get-credentials --resource-group prod-rg --name prod-aks
```

---

# 8. Azure Load Balancer

Tüübid:

- **Basic LB**  
- **Standard LB**  
- **Application Gateway** (Layer 7)  
- **Front Door** (global routing)  

---

# 9. Azure Monitor

Azure Monitor = logid + mõõdikud + alertid.

Komponendid:

- **Log Analytics Workspace**  
- **Metrics**  
- **Alerts**  
- **Application Insights**  

---

# 10. Azure Active Directory (AAD)

AAD = identiteedihaldus.

Kasutatakse:

- kasutajad  
- grupid  
- teenuserollid  
- SSO  
- MFA  

---

# 11. Best Practices

- Kasuta **Resource Groups** loogiliseks eraldamiseks  
- Pane VM-id **private subnet**’isse  
- Kasuta **NSG** reegleid turvalisuseks  
- Kasuta **Managed Disks**  
- Kasuta **AKS** konteinerite jaoks  
- Kasuta **Azure Monitor** logide ja alertide jaoks  
- Kasuta **Terraform** Azure infrastruktuuri haldamiseks  

---

# Kokkuvõte
Azure pakub täielikku pilveplatvormi, mis katab compute, storage, võrgu, andmebaasid, monitooringu ja Kubernetes’e.  
VM + VNet + NSG + Storage + AKS = tüüpiline tootmiskeskkond Azure’is.
```
