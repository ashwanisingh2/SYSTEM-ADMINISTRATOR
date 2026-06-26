---
tags: [desktop-support, career, certification, azure, L1]
aliases: [az-900-roadmap, roadmap-az900]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #az-104
---

# AZ-900: Azure Fundamentals Roadmap

> [!note] Overview
> This roadmap note covers preparation for the Microsoft Certified: Azure Fundamentals (AZ-900) exam. It details the core cloud concepts, Azure services, management, governance, and security models required to pass the exam and start your cloud engineering journey.

---
## Concept Overview
- **What it is** — The AZ-900 exam validates foundational-level knowledge of cloud services and how Microsoft Azure provides those services. It is the entry point for Microsoft Cloud certifications.
- **Why it matters** — Traditional local support roles are shifting to cloud environments. Understanding public cloud architectures, pricing structures, and resource hierarchies is necessary for any modern system administrator.
- **Real job encounter** — Explaining cloud concepts to non-technical managers, estimating costs for a new Azure project using the pricing calculator, and navigating the Azure Portal.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Navigate the Azure portal, verify resource groups, read basic cost dashboard graphs, and check VM status.
  - **Escalation Trigger**: Escalate when subscriptions incur unexpected costs, resource creation fails due to quota limits, or when role-based access restricts access.
  - **L2 Resolution**: Provision simple Azure VMs and Virtual Networks, configure backup vaults, set up resource tags, and manage storage account permissions.
  - **L3 Resolution**: Design landing zones, manage enterprise subscriptions, configure Azure Blueprints, build Azure policies to enforce governance, and optimize cloud spend.

*Seedha simple mein: AZ-900 exam Microsoft Azure Cloud seekhne ki shuruaat hai. Ye certification prove karta hai ki aapko cloud technology ke basics (Virtual Machines, Storage, Cost, Resource Group, security model) ki knowledge hai aur aap cloud infra ko manage kar sakte hain.*

---
## Technical Deep Dive

### 1. Exam Domains and Weightage
- **Describe Cloud Concepts (25–30%)**: High availability, scalability, elasticity, agility, fault tolerance, CapEx vs. OpEx, IaaS, PaaS, SaaS, and Shared Responsibility Model.
- **Describe Azure Architecture and Services (35–40%)**: Resource groups, subscriptions, management groups, regions, availability zones, Virtual Networks, Virtual Machines, Azure Container Instances, Storage Services, and Azure Databases.
- **Describe Azure Management and Governance (30–35%)**: Cost Management, Azure Service Level Agreements (SLAs), Azure Advisor, Azure Portal, Azure PowerShell, Azure CLI, Azure Cloud Shell, Azure Resource Manager (ARM templates), Azure Policy, and Microsoft Purview.

### 2. Core Concepts: Shared Responsibility Model
Understanding where security boundaries lie is critical:
- **On-Premises**: Customer manages everything (networking, storage, servers, OS, application, data, identity).
- **Infrastructure as a Service (IaaS)**: Cloud provider manages hypervisor and hardware. Customer manages OS, runtime, application, data, and access.
- **Platform as a Service (PaaS)**: Cloud provider manages OS and runtime. Customer manages only the code/application and data.
- **Software as a Service (SaaS)**: Cloud provider manages everything. Customer only configures access and data permissions.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A web browser.
> - A Microsoft Account (Outlook/Hotmail).
> - An active credit/debit card for Azure Free Account validation (charges are reversed immediately).

### Step 1: Create an Azure Free Account
1. Open your browser and navigate to `https://azure.microsoft.com/free/`.
2. Click **Start free**.
3. Sign in with your Microsoft Account and complete the registration (phone and card identity verification).
**Expected Result:** You receive a tenant with $200 free credit valid for 30 days and access to various free services.

### Step 2: Create a Resource Group and Virtual Machine
We will provision a lightweight Linux VM.

1. Log into the Azure Portal (`portal.azure.com`).
2. Search for `Resource Groups` in the top search bar and click on it.
3. Click **+ Create**:
   - **Subscription**: Select your free trial subscription.
   - **Resource Group Name**: Type `AZ900-Lab-RG`.
   - **Region**: Select `East US`.
   - Click **Review + Create**, then click **Create**.
4. Once created, search for `Virtual Machines` and click **+ Create** -> **Azure virtual machine**:
   - **Resource Group**: Select `AZ900-Lab-RG`.
   - **Virtual Machine Name**: `AZ900-Linux-VM`.
   - **Region**: `East US`.
   - **Availability Options**: No infrastructure redundancy required.
   - **Image**: `Ubuntu Server 22.04 LTS - x64 Gen2`.
   - **Size**: Select a cheap standard size (e.g., `Standard_B1s` - 1 vCPU, 1 GiB memory).
   - **Authentication Type**: Select **Password** (for testing simplicity). Enter username `azureuser` and a strong password.
   - **Public inbound ports**: Select **Allow selected ports** and check **SSH (22)** and **HTTP (80)**.
5. Click **Review + Create**, then click **Create**. Wait for the deployment to finish.
**Expected Output:** Deployment succeeded screen appears, and you are assigned a public IP address.

### Step 3: Delete Resources to Avoid Cost
In the cloud, always clean up your environment when done.

1. Navigate back to your Resource Group `AZ900-Lab-RG`.
2. Click **Delete resource group** at the top.
3. Type the name of the resource group to confirm and click **Delete**.
**Expected Result:** All resources inside the group (VM, Network Interface, Public IP, Disk) are deleted simultaneously.

---
## Cheat Sheet / Quick Reference

| Azure Service | Category | Core Purpose / Example |
|---|---|---|
| **Resource Group** | Management | Logical container grouping related resources together |
| **Azure Advisor** | Management | Analyzes resource configuration and suggests cost/security fixes |
| **Virtual Network (VNet)**| Networking | Isolated private network enabling communication between VMs |
| **Blob Storage** | Storage | Object storage optimized for unstructured flat files (images, backups) |
| **Azure Files** | Storage | Fully managed cloud file shares accessible via SMB/NFS |
| **Azure Policy** | Governance | Enforces organizational rules (e.g. "Only allow East US VMs") |
| **Pricing Calculator**| Finance | Web tool used to estimate monthly Azure consumption costs |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command / URL |
|---|---|---|---|
| Card verification fails during free account creation. | Virtual/prepaid cards are not accepted, or address details mismatch. | Use a physical credit or debit card; ensure address matches billing statement. | N/A |
| Cannot create VM in East US region: "Quota exceeded." | Free trial accounts have strict limits on total CPU cores in specific regions. | Select an alternative region (e.g., East US 2, West US, or Central US) to deploy. | N/A |
| Cannot connect to the VM via SSH. | Security group is blocking port 22, or Public IP is misspelled. | Verify the inbound security rule for port 22 is active in the VM Network tab. | `ssh azureuser@<VM-Public-IP>` |
| Subscription shows charges despite using free services. | Deployed resources exceeded free tier sizes (e.g., Standard SSD instead of HDD). | Always delete the entire Resource Group after completing labs to stop billing. | N/A |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between IaaS and PaaS?
> **A:** **IaaS (Infrastructure as a Service)** provides raw computing resources like virtual machines, storage, and networking. The user must configure and manage the operating system, runtimes, and applications. **PaaS (Platform as a Service)** provides a managed runtime environment where the cloud provider manages the OS, web server, and updates, allowing the customer to focus entirely on writing and deploying their application code (e.g., Azure App Services).

> [!question] L2 Question
> **Q:** What is the difference between an Azure Region and an Availability Zone?
> **A:** An **Azure Region** is a geographical area containing one or more datacenters connected through a low-latency network (e.g., "East US"). An **Availability Zone** is a unique physical location within a region, made up of one or more datacenters equipped with independent power, cooling, and networking. Deploying across Availability Zones protects applications from local datacenter failures.

> [!question] L3/Scenario Question
> **Q:** Your company wants to migrate an on-premises file share to Azure. The users must access it via SMB from Windows laptops without establishing a VPN. Which Azure service should you recommend, and what is a potential security blocker?
> **A:** 
> - **Situation:** Cloud file share migration with direct Internet SMB access.
> - **Task:** Identify the correct service and address security constraints.
> - **Action:** 
>   1. **Service Recommendation**: Recommend **Azure Files**. It provides fully managed cloud file shares accessible via standard SMB (Server Message Block) protocol.
>   2. **Identify Security Blocker**: The main blocker is that standard SMB (port 445) is blocked by default by almost all residential ISPs (Internet Service Providers) and consumer firewalls due to security vulnerabilities like DoublePulsar/EternalBlue.
>   3. **Provide Solution**: To bypass the port 445 blocker, laptops must access Azure Files using Azure File Sync, use a VPN connection (P2S VPN), or use Azure Files over HTTPS via the REST API or mounting via SMB over QUIC (port 443).
> - **Result:** The business requirement is met, and firewall limitations are securely resolved.

---
## Seedha Simple Mein
*Seedha simple mein: AZ-900 exam Azure Cloud ka basic introduction hai. Isme hum seekhte hain ki resource group kya hote hain, pricing calculate kaise hoti hai, and standard IaaS (VMs), PaaS (App Services), aur SaaS (M365) models me responsibility kaise share hoti hai. Cloud resources run karne ke baad unhe delete karna critical hai billing se bachne ke liye.*

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — Deep dive into cloud models.
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Identity and Governance|AZ104-01 Azure Identity and Governance]] — Advanced governance concepts.
- [[06-Career-Growth/14-Certifications/SC-900-Roadmap|SC-900-Roadmap]] — Security and compliance fundamentals certification.

---
*Tags: #desktop-support #career #certification #azure #L1*
