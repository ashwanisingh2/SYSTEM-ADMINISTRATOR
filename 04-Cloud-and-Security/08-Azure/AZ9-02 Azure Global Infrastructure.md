---
tags: [desktop-support, azure, cloud, L1]
aliases: [az9-02-azure-global-infrastructure, az9-02]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#beginner` `#none`

# AZ9-02: Azure Global Infrastructure

> [!abstract] Overview
> Yeh note Azure ke global physical aur logical architecture ko explain karta hai. Isme Regions, Availability Zones, aur Subscriptions/Resource Groups ka hierarchy samjhaya gaya hai. Resources ko properly deploy aur manage karne ke liye yeh structure jaanna zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — Microsoft Azure ka physical datacenter layout aur resources ko organize karne ka logical folder structure.
- **Why it matters** — Sahi region chunne se latency kam hoti hai aur Availability Zones se downtime bachta hai.
- **Where you see this** — Jab aap portal pe VM banate hain, toh aapko "Region" aur "Resource Group" select karna hota hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Resource Groups banana aur resources ki location check karna. |
| **L2** | Proper naming conventions aur tags apply karna. |
| **L3** | Multi-region disaster recovery aur Management Groups design karna. |

> [!tip] Seedha Simple Mein
> *Azure ka infrastructure ek global logistics company jaisa hai. Region bada warehouse hub hai, Zone uske andar alag alag buildings hain, aur Resource Group woh dabbe hain jisme aapka project rakha hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Infrastructure** is like a **Worldwide Logistics Network** because...
>
> - **Geographies** are countries with strict border laws (Data Residency).
> - **Regions** are major warehouse hubs in specific cities (e.g., East US).
> - **Availability Zones** are 3 separate buildings inside that hub with independent power. If one floods, the others keep working.
> - **Resource Groups** are shipping boxes: grouping related items for one order to track and bill easily.

---
## 🔬 Technical Deep Dive

### 1. Physical Infrastructure Hierarchy

- **Geographies:** Ensure data residency compliance (e.g., EU data stays in EU).
- **Regions:** City-scope hubs. Choice based on Latency, Service Availability, and Cost.
- **Region Pairs:** Regions paired 300+ miles apart (East US/West US) for High Availability backup against regional disasters.
- **Availability Zones:** Separate datacenters *within* a region. Independent power/cooling. Deploying across zones gives `99.99%` SLA.

### 2. Azure Logical Hierarchy

> [!info] Key Concept
> Logical hierarchy manages billing and RBAC (Role-Based Access Control).

`Entra ID Tenant -> Management Groups -> Subscriptions -> Resource Groups -> Resources`

- **Subscriptions:** The primary billing boundary.
- **Resource Groups (RG):** A logical folder. A resource can only be in one RG at a time, but RGs can hold resources from different regions.

### 3. Naming Conventions & Tagging

- **Tags:** Metadata key-value pairs (e.g., `Env: Prod`, `Dept: IT`) for billing and filtering.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Access to Azure Portal.

### Step 1: Create a Resource Group

```bash
az group create --name rg-lab-infrastructure --location eastus --tags Project=Sysadmin_Lab Env=Development
```

1. Portal > **Resource groups** > **Create**.
2. Name: `rg-lab-infrastructure`, Region: **East US**.
3. Add tags: `Project : Sysadmin_Lab` and `Env : Development`.
4. Click **Review + create**.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az account list-locations` | List available Azure regions | `az account list-locations -o table` |
| `az group create` | Creates a resource group | `az group create -n rg1 -l eastus` |
| `az group list` | List all resource groups | `az group list` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Cannot deploy a specific VM size | Service not available in region | Check region service availability matrix and choose another region. |
| Resource cannot be created | Subscription quota reached | Request quota increase or delete unused resources in the subscription. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Billing Confusion

> [!example] Ticket
> "Finance wants to know which department is costing the most in our Azure bill."

**L1 Response:** Explain that tags are needed for this.
**Escalation Trigger:** Apply bulk tags via PowerShell.
**L2 Resolution:** Add mandatory `Department` tags to all Resource Groups and configure billing reports grouped by this tag.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between an Availability Zone and a Region Pair?
> **Answer:** Availability Zones are separate datacenters *within the same region* to protect against local facility failures (like a power outage). Region Pairs are entirely *different regions* at least 300 miles apart to protect against massive regional disasters (like a hurricane).

> [!question] Q2: Can a resource group contain resources from multiple regions?
> **Answer:** Yes, a Resource Group is just a logical container. It can hold resources deployed in East US, West Europe, etc.

==**Exam Tip:** Every resource must belong to exactly one Resource Group. It cannot belong to multiple.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — Base virtualization concepts.
- [[04-Cloud-and-Security/08-Azure/AZ104-01 Azure Identity and Governance|AZ104-01 Azure Identity and Governance]] — Managing subscriptions and resource locks.
