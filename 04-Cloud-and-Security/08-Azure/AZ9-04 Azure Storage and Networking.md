---
tags: [desktop-support, azure, cloud, L1]
aliases: [az9-04-azure-storage-and-networking, az9-04]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#beginner` `#none`

# AZ9-04: Azure Storage and Networking

> [!abstract] Overview
> Yeh note Azure ke core storage aur networking services ke baare mein hai. Isme Storage redundancy (LRS, ZRS, GRS), Blob Tiers, VNet architecture, NSG firewalls aur hybrid connectivity (VPN/ExpressRoute) ki basics shamil hain.

---
## 🧠 Concept Overview

- **What it is** — Cloud mein data save karne ke tareeqe aur isolated virtual networks banane ke tools.
- **Why it matters** — Secure data storage aur VMs ke beech secure communication ensure karne ke liye.
- **Where you see this** — Backup files store karna, ya VMs ke liye private subnet banana.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Blob storage me files upload karna aur ping checks karna. |
| **L2** | NSG firewall rules banana, VNet peering setup karna aur file shares mount karna. |
| **L3** | ExpressRoute architecture design karna aur Geo-redundant storage configure karna. |

> [!tip] Seedha Simple Mein
> *VNet ek private campus ki boundary hai aur Subnet uske andar ke kamre. NSG security guard hai jo check karta hai kaun andar aa raha hai. Storage account apka cloud hard drive hai jiske Azure khud 3 backup rakhta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Networking & Storage** is like a **Secure Corporate Campus** because...
>
> - **VNet** is the physical fence around the property.
> - **Subnets** are specific rooms (e.g., HR room vs Public Lobby).
> - **NSG (Firewall)** is the badge reader on each door.
> - **Storage Replication (LRS/ZRS/GRS)** is your filing system backup: Keep 3 copies in one room (LRS), in 3 different buildings (ZRS), or ship one to another city (GRS).

---
## 🔬 Technical Deep Dive

### 1. Storage Account Replication

- **LRS (Locally Redundant):** 3 copies in one physical datacenter. Low cost.
- **ZRS (Zone Redundant):** 3 copies across 3 Availability Zones. Protects against building outage.
- **GRS (Geo-Redundant):** Replicates to a paired region 300+ miles away. Protects against regional disaster.

### 2. Blob Access Tiers

> [!info] Key Concept
> Tiers optimize cost based on access frequency.

- **Hot:** Frequently accessed. High storage cost, low read cost.
- **Cool:** Infrequent (30+ days). Lower storage, higher read cost.
- **Archive:** Rare (180+ days). Lowest storage, highest read cost (takes hours to rehydrate).

### 3. Azure Virtual Networks (VNets) & NSGs

- **VNet:** Logical isolated network (`10.0.0.0/16`).
- **Subnet:** Smaller segments (`10.0.1.0/24`).
- **NSG (Network Security Group):** Stateful firewall evaluating Source, Destination, Port, Protocol based on Priority (lowest number evaluated first).

### 4. Inter-Network Connectivity

- **VNet Peering:** Connects two VNets over Azure private backbone.
- **VPN Gateway:** Site-to-Site tunnel over the public internet.
- **ExpressRoute:** Dedicated private physical fiber connection to Azure.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Active Azure Portal session.

### Step 1: Create a Storage Account and Blob

1. Portal > **Storage accounts** > **Create**.
2. Name: `saalphadevlab01` (globally unique), Region: East US, Redundancy: **LRS**.
3. Once created, go to **Containers** > **+ Container**. Name: `images` (Private).
4. Click container > **Upload** a test image file.

### Step 2: Configure an NSG Rule

1. Portal > **Network security groups**. Select your NSG.
2. **Inbound security rules** > **+ Add**.
3. Source: Any, Port: `*`
4. Destination: Any, Port: `8080`, Action: Allow, Priority: 150.
5. Click **Add**.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az storage account create` | Creates a storage account | `az storage account create -n mystorage123 -g myRG -l eastus --sku Standard_LRS` |
| `az network vnet create` | Creates a Virtual Network | `az network vnet create -g myRG -n myVNet --address-prefix 10.0.0.0/16` |
| `az network nsg rule create` | Creates a firewall rule | `az network nsg rule create -g myRG --nsg-name myNSG -n Allow80 --priority 100 --destination-port-ranges 80` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Cannot RDP into VM | NSG blocking port 3389 | Check effective security rules and add Inbound allow for 3389 from your IP. |
| Slow file download from blob | Blob is in Archive tier | Change tier to Hot, wait for rehydration process to finish. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Website Unreachable

> [!example] Ticket
> "I just deployed a new web VM but I cannot reach the homepage on port 80."

**L1 Response:** Ping IP and check basic VM status.
**Escalation Trigger:** VM is running but port unresponsive.
**L2 Resolution:** Check the Network Security Group (NSG) attached to the VM's subnet or NIC. Ensure there is an Inbound rule allowing TCP Port 80 from Source Any.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between LRS and GRS storage replication?
> **Answer:** LRS keeps 3 copies of data within a single datacenter. GRS replicates data to a secondary region hundreds of miles away, protecting against a full regional disaster.

> [!question] Q2: How does a Network Security Group (NSG) evaluate rules?
> **Answer:** NSGs evaluate rules based on priority numbers (from 100 to 65000). The lowest priority number is evaluated first, and once a match is found, further processing stops.

==**Exam Tip:** ExpressRoute never uses the public internet; VPN Gateway always uses the public internet.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-02 Azure Global Infrastructure|AZ9-02 Azure Global Infrastructure]] — Availability Zone designs.
- [[04-Cloud-and-Security/08-Azure/AZ9-03 Azure Compute Services|AZ9-03 Azure Compute Services]] — Mounting network disks to VMs.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Advanced hub-spoke VNet peering structures.
