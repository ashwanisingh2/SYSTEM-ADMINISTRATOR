---
tags: [desktop-support, azure, cloud, L2]
aliases: [azure-certification-roadmaps, azure-certification-roadmaps]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

# Azure Certification Roadmaps (AZ-900 & AZ-104)

---

---
## Concept Overview
- **What it is**: The Microsoft Azure certification path validates cloud competence. **AZ-900 (Azure Fundamentals)** covers baseline cloud concepts, core services, security, privacy, and billing. **AZ-104 (Azure Administrator)** is a deep, technical certification validating skills in managing identity, governance, storage, compute, and virtual networks.
- **Why it matters for a support engineer**: Cloud literacy is mandatory for modern IT roles. AZ-900 provides L1 engineers with structural cloud vocabulary, while AZ-104 is the gold standard for L2/L3 engineers seeking to administer corporate cloud environments.
- **Where you encounter this in real job**: Studying for professional growth, matching cloud support tickets to vendor exam concepts, and applying standardized architecture patterns.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Studies and obtains the AZ-900 to understand IaaS/PaaS structures and basic portal navigation.
  - **L2**: Obtains the AZ-104 to manage subscriptions, configure virtual networks, resize VMs, and deploy storage accounts.
  - **L3**: Leverages AZ-104 (and advances to AZ-305 Solutions Architect) to design enterprise cloud migrations, governance models, and cost-control systems.

---

---
## Technical Deep Dive
### 1. Certification Breakdown: AZ-900 vs. AZ-104

| Feature | AZ-900 (Azure Fundamentals) | AZ-104 (Azure Administrator Associate) |
|---|---|---|
| **Target Audience** | Beginners, Sales, Support L1 | System Administrators, Support L2/L3 |
| **Difficulty Level** | Introductory (Conceptual) | Intermediate to Advanced (Technical/Lab-heavy) |
| **Core focus** | Cloud concepts, general services, billing | Identity, storage, VM compute, VNets, monitoring |
| **Exam Format** | Multiple choice, drag/drop (40-50 Qs) | Multiple choice, case studies, active labs (40-60 Qs) |
| **Passing Score** | 700 / 1000 | 700 / 1000 |

### 2. AZ-900 Exam Blueprint & Key Modules
The AZ-900 exam evaluates three primary areas:
1. **Describe Cloud Concepts (25-30%)**:
   - High Availability, Scalability, Elasticity, Agility, and Disaster Recovery.
   - Shared Responsibility Model (what Microsoft secures vs. what the Customer secures).
   - IaaS (Infrastructure as a Service), PaaS (Platform as a Service), SaaS (Software as a Service).
2. **Describe Azure Architecture and Services (35-40%)**:
   - Azure Resource Manager (ARM), Resource Groups, Subscriptions, and Management Groups.
   - Compute types (VMs, App Services, Containers, Azure Virtual Desktop).
   - Networking (VNets, ExpressRoute, VPN Gateways, DNS).
   - Storage classes (Blob, File, Queue, Table) and disk types.
3. **Describe Azure Management and Governance (30-35%)**:
   - Cost Management, Pricing Calculator, TCO Calculator.
   - Azure Blueprints, Azure Policy, Resource Locks, and Service Trust Portal.

### 3. AZ-104 Exam Blueprint & Key Modules
The AZ-104 exam requires hands-on technical configuration skills across five domains:
1. **Manage Azure Identities and Governance (15-20%)**:
   - Entra ID (Azure AD) user and group administration.
   - Configure Role-Based Access Control (RBAC) and Subscription policies.
   - Configure Azure Resource Locks and Resource Tags.
2. **Implement and Manage Storage (15-20%)**:
   - Storage Accounts: configure replication (LRS, GRS, ZRS), network access, and SAS tokens.
   - Configure Azure Files and Blob storage containers.
3. **Deploy and Manage Azure Compute Resources (20-25%)**:
   - Deploy VMs using ARM templates.
   - Configure VM sizing, disk attachments, and extensions.
   - Deploy Azure Container Instances (ACI) and Azure Kubernetes Service (AKS).
4. **Configure and Manage Virtual Networking (20-25%)**:
   - Deploy VNets, configure Subnets, and setup VNet Peering.
   - Manage Network Security Groups (NSG) and Application Security Groups (ASG).
   - Configure Azure DNS, Public IPs, and VPN Gateways.
5. **Monitor and Backup Azure Resources (10-15%)**:
   - Recovery Services Vault: configure backup policies and perform VM restores.
   - Configure Azure Monitor: query Log Analytics, create alerts, and check diagnostic logs.

---


## Real-World Exam Scenarios

### Scenario 1: Selecting the Right Storage Replication Option (High Availability)
**Exam Question Style:** Your company has an application that stores logs in an Azure Storage account. Corporate compliance dictates that data must remain accessible even if a primary Azure region suffers a catastrophic physical disaster (e.g., hurricane). Cost must be kept minimal. Which storage replication option do you select?
**Your First 3 Checks:**
1. Determine if Locally Redundant Storage (LRS) satisfies the regional disaster requirement (No, LRS is single datacenter).
2. Determine if Zone-Redundant Storage (ZRS) satisfies regional disaster protection (No, ZRS is across zones in a single region).
3. Evaluate Geo-Redundant Storage (GRS) vs. Read-Access Geo-Redundant Storage (RA-GRS).
**Diagnosis/Rationale:**
- **GRS**: Replicates your data to a secondary region hundreds of miles away. If the primary region fails, Microsoft triggers a failover, restoring access. This satisfies the disaster requirement.
- **RA-GRS**: Provides read-only access to the secondary region *before* failover. This is more expensive and unnecessary since the compliance rule only requires availability during disaster recovery.
**Root Cause:** Compliance requirement for geographic backup redundancy.
**Fix/Option:** Configure the Storage Account replication to **Standard_GRS** (Geo-Redundant Storage).
**Prevention:** Standardize storage templates to use GRS for all production-critical database and log storage accounts.

### Scenario 2: Troubleshooting Subnet Routing (User-Defined Routes vs Default)
**Exam Question Style:** You have two subnets inside a VNet: `Subnet-Web` (10.0.1.0/24) and `Subnet-DB` (10.0.2.0/24). You deploy an Azure Firewall in a third subnet. You need to ensure all outbound traffic from the Web subnet to the DB subnet routes through the Firewall. What do you configure?
**Your First 3 Checks:**
1. Check the default system routes created by Azure inside a VNet.
2. Determine if an Azure Route Table is needed.
3. Define the User-Defined Route (UDR) target parameters.
**Diagnosis/Rationale:**
1. By default, Azure automatically creates system routes allowing all subnets in a VNet to communicate directly.
2. To override this default behavior, you must create a custom Route Table.
3. You must write a route rule directing traffic for the destination range `10.0.2.0/24` to the **Virtual Appliance** next hop type, pointing to the Azure Firewall's private IP (e.g., `10.0.3.4`).
4. Finally, you must associate the Route Table to `Subnet-Web`.
**Root Cause:** Overriding default system routing within an Azure VNet.
**Fix/Option:**
1. Create a Route Table.
2. Add a route: Destination `10.0.2.0/24`, Next Hop: `Virtual Appliance`, Next Hop Address: `10.0.3.4`.
3. Link the Route Table to `Subnet-Web`.
**Prevention:** Avoid writing conflicting route rules in different subnets to prevent routing loops.

---


## Critical Points

> [!danger] Never Do This
> - Never attempt the AZ-104 exam without performing actual hands-on laboratory exercises.
> - The AZ-104 is known for case studies and command-line syntax questions. You cannot pass the exam by memorizing text definitions; you must understand the interface layout and command parameters.

> [!warning] Common Trap
> - Confusing **Geographic Redundancy** with **High Availability** (HA) during exams.
> - GRS protects against regional data loss, but it has a recovery time objective (RTO) delay. For active-active high availability, you must run application VMs in scale sets across multiple Availability Zones in a region using ZRS.

> [!tip] Senior Engineer Tip
> - When studying for the AZ-104, focus heavily on the **Virtual Networking** domain (20-25%). Historically, candidates fail the exam because they do not understand subnets, VNet peering, NSG rule evaluation orders, and DNS routing.

> [!success] Verification Steps
> - Use the **Microsoft Learn Assessment** free practice exams.
> - Achieve a consistent score of **80% or higher** on practice assessments before registering for the official AZ-104 exam.

> [!question] Interview Alert
> - "Explain the difference between Azure Policy and Role-Based Access Control (RBAC)."
> - Answer: "RBAC controls *who* has access to resources and *what* actions they can perform (e.g., User A can create VMs). Azure Policy controls the *properties* of the resources themselves, regardless of who deploys them (e.g., VMs must be created in the East US region, or VMs cannot use standard HDDs)."

---


## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Rushing to book the AZ-104 immediately after AZ-900 | Underestimating the technical depth | Spend 4-6 weeks building active labs in Azure before booking the AZ-104 exam. |
| Forgetting to delete lab resources after study sessions | Leaving VMs running, incurring costs | Group all lab resources into a single Resource Group (e.g., RG-Lab) and delete the group to wipe all resources at once. |
| Memorizing old dumps containing out-of-date Azure portal paths | Azure interface updates constantly | Focus on Microsoft Learn documentation, which is updated continuously. |

---


## Tags
#desktop-support #azure #certification #roadmaps #L1 #L2 #interview-topic #lab-complete #daily-use

---
## Step-by-Step Lab
**Objective:** Execute a standard AZ-104 command-line provisioning template to build a resource group, create a storage account with lifecycle management policies, and verify configuration.
**Time Required:** 30 minutes
**Environment Needed:** Azure Free Tier or Sandbox account.
**Pre-requisites:** Azure CLI installed on your PC.

**Steps:**
1. Open PowerShell/CMD. Log in to Azure:
   ```bash
   az login
   ```
2. Create the targeted Resource Group:
   ```bash
   az group create --name RG-Exam-Prep --location westus2
   ```
3. Create a Standard LRS Storage Account:
   ```bash
   az storage account create --name saexamprep104 --resource-group RG-Exam-Prep --location westus2 --sku Standard_LRS --kind StorageV2
   ```
4. Define a Lifecycle Management Policy in JSON format to delete blobs 30 days after creation:
   Create a local file `policy.json`:
   ```json
   {
     "rules": [
       {
         "enabled": true,
         "name": "DeleteAfter30Days",
         "type": "Lifecycle",
         "definition": {
           "actions": {
             "baseBlob": {
               "delete": { "daysAfterModificationGreaterThan": 30 }
             }
           },
           "filters": { "blobTypes": [ "blockBlob" ] }
         }
       }
     ]
   }
   ```
5. Apply the lifecycle policy:
   ```bash
   az storage account management-policy create --account-name saexamprep104 --resource-group RG-Exam-Prep --policy @policy.json
   ```
6. Verification: List the applied policy to confirm it is active:
   ```bash
   az storage account management-policy show --account-name saexamprep104 --resource-group RG-Exam-Prep
   ```

**Success Criteria:** The Resource Group and Storage Account are created, and the lifecycle policy is applied successfully, returning the JSON configuration in the output.
**Common Failures:** The command fails if the storage account name (`saexamprep104`) is not globally unique.

---

---
## Cheat Sheet / Quick Reference
AZ-104 questions often require selecting the correct PowerShell cmdlet or Azure CLI command:

### PowerShell
```powershell
# Create a new Storage Account with Geo-Redundant Storage (GRS)
New-AzStorageAccount -ResourceGroupName "RG-Prod" -Name "sauniquename104" -Location "eastus" -SkuName "Standard_GRS" -Kind "StorageV2"

# Create a VNet Peering connection
Add-AzVirtualNetworkPeering -Name "Peer-Hub-to-Spoke" -VirtualNetwork $HubVNet -RemoteVirtualNetworkId $SpokeVNet.Id

# Mount an Azure File Share on a local Windows PC
net use Z: \\sauniquename104.file.core.windows.net\sharename /u:AZURE\sauniquename104 [Account_Key]
```

### Azure CLI
```bash
# Create a Resource Group
az group create --name RG-Prod --location eastus

# Generate a SAS token for a storage container (expires in 2 hours)
az storage container generate-sas --account-name sauniquename104 --name blobs --permissions r --expiry 2026-06-25T18:00:00Z --output tsv
```

---

> [!info] 60-Second Summary
> **What**: The Microsoft certification path validating Azure cloud knowledge (AZ-900: Fundamentals, AZ-104: Administrator).
> **Why**: Critical for validation, career growth, and standardizing team operations.
> **How**: Study exam blueprints, build active labs (using resource groups), and master PowerShell/CLI commands.
> **Command**: `New-AzStorageAccount` / `az group create`
> **Interview Answer Starter**: "To prepare for cloud administration, I target Microsoft's AZ-104 blueprint, focusing on virtual networks, RBAC governance, and storage accounts..."

**Key Numbers to Remember:**
- AZ-104 pass score: 700 / 1000
- Networking weighting in AZ-104: 20-25%
- Cloud concepts weighting in AZ-900: 25-30%
- Free account credit window: 30 days

**3 Things Interviewer Wants to Hear:**
- Hands-on lab experience is mandatory to pass the AZ-104
- The difference between Layer 4 (Load Balancer) and Layer 7 (App Gateway) load balancing
- How to manage cloud billing using Resource Tags and Cost Management

---

---
## Troubleshooting
| Problem | Cause | Fix | Command |
|---|---|---|---|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall | `ping -c 4 <ip>` / `nc -zv <ip> <port>` |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password | N/A |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects | N/A |

---
## Interview Questions
### Basic (L1 Level)
**Q: What is the main difference between IaaS and PaaS in cloud computing?**
A: IaaS (Infrastructure as a Service) provides raw hardware like virtual machines, where you manage the OS, software updates, and configuration. PaaS (Platform as a Service) provides a managed hosting environment (like Azure App Services) where Microsoft handles the OS, server patching, and scaling, and you only manage the application code.

**Q: What are the three default levels of Azure Resource Locks, and what do they do?**
A: The two main levels are **ReadOnly** (which allows users to view a resource but not modify or delete it) and **CanNotDelete** (which allows users to read and modify a resource but blocks anyone from deleting it).

### Intermediate (L2 Level)
**Q: How does a user connect their local PC to an Azure Virtual Network to access private resources?**
A: They use a **Point-to-Site (P2S) VPN**. First, you configure a Virtual Network Gateway in the Azure VNet and upload root certificates. Next, you generate a VPN Client configuration package and install it on the user's laptop. The user logs in using their client credentials, establishing a secure connection to the VNet.

**Q: Explain how to configure Azure Backup to preserve a database server VM for 5 years while keeping storage costs low.**
A: I create a Recovery Services Vault and configure a custom Backup Policy. I set the policy to perform daily backups. Under retention, I set the daily recovery points to delete after 14 days. I then enable **Monthly Retention** for 60 months (5 years) and **Yearly Retention** for 5 years, keeping only the final backup of the month/year. This preserves historical points without storing thousands of daily snapshots, reducing costs.

### Advanced (L3/Senior Level)
**Q: You are preparing for the AZ-104 exam. A case study requires configuring a high-availability load balancing architecture. Compare when to use Azure Load Balancer, Azure Application Gateway, and Azure Front Door.**
A:
- **Situation**: Designing load balancing solutions for Azure architectures.
- **Task**: Differentiate between load balancer types based on OSI layers and scope.
- **Action**:
  - **Azure Load Balancer**: Operates at Layer 4 (TCP/UDP). It is a high-speed, low-latency load balancer used to distribute traffic within a region to VMs.
  - **Application Gateway**: Operates at Layer 7 (HTTP/HTTPS). It supports SSL termination, path-based routing (e.g., sending `/images` traffic to a different backend pool), and includes a Web Application Firewall (WAF). Scoped to a region.
  - **Azure Front Door**: A global web entry point operating at Layer 7. It uses routing protocols (Anycast) to route traffic to the closest regional endpoint, offering global high availability and CDNs.
- **Result**: Selecting the correct load balancer aligns with application scope, security requirements, and protocol layers.

### HR / Behavioral
**Q: Why are you pursuing the AZ-104 certification, and how will it help our support team?**
A: Our team is migrating more workloads to the cloud. I want to ensure my skills are aligned with Microsoft's best practices. The AZ-104 validates practical, technical skills in identity, virtual networking, and storage. Passing it ensures I can resolve complex Azure VM and network issues faster, reduce escalation rates, and help train our L1 staff to navigate the Azure Portal securely.

---

---
## Seedha Simple Mein
*Seedha simple mein: Azure-Certification-Roadmaps ke bare mein seekhta hai. Yeh azure infrastructure aur system settings ko properly implement karne aur support tickets ko runbooks ke help se standard templates me clear karne me help karta hai.*

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]] — Core domain (IAM / RBAC) tested in AZ-104.
- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]] — Core compute domain tested in AZ-104.
- [[04-Cloud-and-Security/08-Azure/Azure-Networking|Azure Networking]] — Core networking domain (VNet/NSG) tested in AZ-104.

---
