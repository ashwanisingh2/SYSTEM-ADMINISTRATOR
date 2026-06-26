---
tags: [azure, certification, intermediate]
aliases: [azure-certification-roadmaps]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#intermediate` `#az-104`

# AZ-08: Azure Certification Roadmaps (AZ-900 & AZ-104)

> [!abstract] Overview
> Yeh note Microsoft Azure certification path (AZ-900 aur AZ-104) cover karta hai. Ek support engineer ke liye cloud literacy mandatory hai modern IT roles mein, jisse ticket handling aur architecture samajhne me aasani hoti hai.

---
## 🧠 Concept Overview

- **What it is** — The Microsoft Azure certification path validates cloud competence. **AZ-900 (Azure Fundamentals)** covers baseline cloud concepts, while **AZ-104 (Azure Administrator)** validates technical skills in managing identity, governance, storage, compute, and virtual networks.
- **Why it matters** — Cloud literacy is mandatory for modern IT roles. AZ-900 provides L1 engineers with structural cloud vocabulary, while AZ-104 is the gold standard for administering corporate cloud environments.
- **Where you see this** — Studying for professional growth, matching cloud support tickets to vendor exam concepts, and applying standardized architecture patterns.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Studies AZ-900 to understand IaaS/PaaS structures and basic portal navigation. |
| **L2** | Obtains AZ-104 to manage subscriptions, configure virtual networks, resize VMs, and deploy storage accounts. |
| **L3** | Leverages AZ-104 (and AZ-305) to design enterprise cloud migrations, governance models, and cost-control systems. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Azure certifications aapko cloud environment manage karne ke skills dete hain. AZ-900 basic level ke liye aur AZ-104 advanced level admin kaam ke liye.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Certification Path** is like **getting your driver's licenses** because...
>
> - **AZ-900 (Learner's Permit)**: You learn the rules of the road, what different road signs mean, and how to identify car parts.
> - **AZ-104 (Commercial Driver's License)**: You learn how to actually drive a heavy vehicle, maintain the engine, plan complex routes, and troubleshoot breakdowns.

---
## 🔬 Technical Deep Dive

### 1. Certification Breakdown: AZ-900 vs. AZ-104

> [!info] Key Concept
> Understanding the target audience and exam requirements is crucial before preparation.

| Feature | AZ-900 (Azure Fundamentals) | AZ-104 (Azure Administrator Associate) |
|---|---|---|
| **Target Audience** | Beginners, Sales, Support L1 | System Administrators, Support L2/L3 |
| **Difficulty Level** | Introductory (Conceptual) | Intermediate to Advanced (Technical/Lab-heavy) |
| **Core focus** | Cloud concepts, general services, billing | Identity, storage, VM compute, VNets, monitoring |

### 2. AZ-104 Exam Blueprint

The AZ-104 exam requires hands-on technical configuration skills across five domains:
1. **Manage Azure Identities and Governance**: Entra ID, RBAC, Policies.
2. **Implement and Manage Storage**: Storage Accounts, Azure Files, Blob.
3. **Deploy and Manage Azure Compute Resources**: VMs, ARM templates, AKS.
4. **Configure and Manage Virtual Networking**: VNets, Subnets, NSG, DNS.
5. **Monitor and Backup Azure Resources**: Recovery Services Vault, Azure Monitor.

> [!danger] Common Mistake
> Rushing to book the AZ-104 immediately after AZ-900 without spending 4-6 weeks building active labs in Azure.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure Free Tier or Sandbox account.
> - Azure CLI installed on your PC.

### Step 1: Execute standard AZ-104 provisioning template

```bash
# Log in to Azure
az login

# Create the targeted Resource Group
az group create --name RG-Exam-Prep --location westus2

# Create a Standard LRS Storage Account
az storage account create --name saexamprep104 --resource-group RG-Exam-Prep --location westus2 --sku Standard_LRS --kind StorageV2
```

> [!success] Expected Output
> ```json
> {
>   "id": "/subscriptions/.../resourceGroups/RG-Exam-Prep",
>   "name": "RG-Exam-Prep",
>   "properties": {
>     "provisioningState": "Succeeded"
>   }
> }
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `New-AzStorageAccount` | Naya Storage Account banata hai | `New-AzStorageAccount -ResourceGroupName "RG-Prod" -Name "sauniquename104" -Location "eastus" -SkuName "Standard_GRS" -Kind "StorageV2"` |
| `Add-AzVirtualNetworkPeering` | Do VNets ko aapas me connect karta hai (Peering) | `Add-AzVirtualNetworkPeering -Name "Peer-Hub-to-Spoke" -VirtualNetwork $HubVNet -RemoteVirtualNetworkId $SpokeVNet.Id` |
| `az group create` | Naya Resource Group banata hai | `az group create --name RG-Prod --location eastus` |
| `az storage container generate-sas` | Storage container ke liye SAS token generate karta hai | `az storage container generate-sas --account-name sauniquename104 --name blobs --permissions r --expiry 2026-06-25T18:00:00Z --output tsv` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on NSG/firewall |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions (RBAC) or reset password |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Selecting the Right Storage Replication Option (High Availability)

> [!example] Ticket
> "Need an Azure Storage account for app logs. Data must remain accessible even if a primary Azure region suffers a catastrophic physical disaster (e.g., hurricane). Cost must be kept minimal."

**L1 Response:** Check standard storage configurations and confirm regional disaster requirements.
**Escalation Trigger:** Agar exact replication topology clear nahi hai cost vs availability trade-off ke liye.
**L2 Resolution:** Evaluate LRS/ZRS/GRS/RA-GRS. Configure the Storage Account replication to **Standard_GRS** (Geo-Redundant Storage). GRS replicates data to a secondary region hundreds of miles away, keeping costs minimal while satisfying DR requirements.

---
## 🎤 Interview Questions

> [!question] Q1: Explain the difference between Azure Policy and Role-Based Access Control (RBAC).
> **Answer:** RBAC controls *who* has access to resources and *what* actions they can perform (e.g., User A can create VMs). Azure Policy controls the *properties* of the resources themselves, regardless of who deploys them (e.g., VMs must be created in the East US region).

==**Exam Tip:** Focus heavily on the Virtual Networking domain (20-25%). Historically, candidates fail the exam because they do not understand subnets, VNet peering, NSG rule evaluation orders, and DNS routing.==

> [!question] Q2: What is the main difference between IaaS and PaaS in cloud computing?
> **Answer:** IaaS provides raw hardware like virtual machines, where you manage the OS, updates, and config. PaaS provides a managed hosting environment where Microsoft handles the OS, patching, and scaling, and you only manage the app code.

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]] — Core domain (IAM / RBAC) tested in AZ-104.
- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]] — Core compute domain tested in AZ-104.
- [[04-Cloud-and-Security/08-Azure/Azure-Networking|Azure Networking]] — Core networking domain (VNet/NSG) tested in AZ-104.
