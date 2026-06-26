---
tags: [desktop-support, azure, cloud, L1]
aliases: [az9-01-cloud-computing-fundamentals, az9-01]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#beginner` `#none`

# AZ9-01: Cloud Computing Fundamentals

> [!abstract] Overview
> Yeh note cloud computing ke basic concepts ko cover karta hai. Isme CapEx vs OpEx, IaaS/PaaS/SaaS models, Public/Private/Hybrid clouds aur Shared Responsibility Model bataya gaya hai. Naye engineer ke liye cloud ka basic logic samajhne ke liye yeh zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — Cloud computing matlab IT resources (servers, storage, databases) ko internet ke through rent par lena.
- **Why it matters** — Isse upfront cost (CapEx) bachta hai aur aap demand ke hisaab se instantly resources scale kar sakte hain.
- **Where you see this** — Jab aap physical server kharidne ke bajaye cloud par virtual machine banate hain.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic cloud terms aur service models identify karna. |
| **L2** | Alag-alag services deploy karte waqt IaaS aur PaaS ke differences samajhna. |
| **L3** | Enterprise level cloud migration strategy (CapEx to OpEx) design karna. |

> [!tip] Seedha Simple Mein
> *Cloud computing electricity grid jaisa hai. Aap khud ka generator nahi banate, bas wall mein plug lagate ho aur jitna use karte ho utna bill bharte ho (OpEx).*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Cloud Computing** is like using the **City Electricity Grid** because...
>
> - **On-Premises** is buying your own massive generator. You pay upfront (CapEx) and manage maintenance, regardless of whether you turn on a lightbulb.
> - **Cloud Computing** is plugging into the wall. Azure manages the generators. You only pay for what you consume (OpEx), and you can draw more power instantly if needed (Elasticity).

---
## 🔬 Technical Deep Dive

### 1. Cloud Infrastructure Benefits

- **High Availability (HA):** Minimal downtime with redundant datacenters.
- **Scalability:** Add resources when load increases (Scale Up = Bigger VM, Scale Out = More VMs).
- **Elasticity:** The ability to auto-scale dynamically based on load (e.g., adding VMs on Black Friday and removing them later).

### 2. Financial Models: CapEx vs. OpEx

> [!info] Key Concept
> CapEx requires large upfront hardware purchases. OpEx is a pay-as-you-go model.

- **CapEx (Capital Expenditure):** Physical infrastructure. Must amortize value over years.
- **OpEx (Operational Expenditure):** Cloud subscriptions. No upfront cost, tax-deductible in the same year.

### 3. Cloud Service Models

- **IaaS:** You rent hardware. You manage OS, middleware, and apps (e.g., Azure VMs).
- **PaaS:** Provider manages hardware and OS. You manage apps and DBs (e.g., Azure App Services).
- **SaaS:** Fully managed software. You just use it (e.g., Microsoft 365).

### 4. Shared Responsibility Model

> [!danger] Common Mistake
> No matter the model (IaaS, PaaS, SaaS), the **Customer** is ALWAYS responsible for Data and Access/Identity permissions.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure fundamental theoretical understanding.

### Step 1: Identify Service Models

1. Analyze requirement: A company wants to host a website without managing the underlying OS.
2. Solution: Choose **PaaS** (Azure App Service).

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `IaaS` | Infrastructure as a Service | Virtual Machines, VNet |
| `PaaS` | Platform as a Service | App Services, SQL DB |
| `SaaS` | Software as a Service | M365, Teams |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Wasted cost on unused VMs | VMs running 24/7 without elastic rules | Configure Auto-scaling to shutdown instances when CPU is low. |
| Confusion over OS patching in PaaS | Expecting manual control | PaaS removes OS control; use IaaS if you need registry access. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Access Management in SaaS

> [!example] Ticket
> "Can Microsoft restore a deleted user email because we use M365 SaaS?"

**L1 Response:** Explain that under the Shared Responsibility Model, data and identities are the customer's responsibility.
**Escalation Trigger:** If a backup solution needs to be implemented.
**L2 Resolution:** Set up third-party or native M365 backups for user data going forward.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Scalability and Elasticity?
> **Answer:** Scalability is the capacity to handle load by adding resources (Scale Out/Up). Elasticity is automating this process to add or remove resources dynamically based on metrics to save costs.

> [!question] Q2: Who is responsible for data security in a SaaS application?
> **Answer:** The Customer is always 100% responsible for their data and identity access, even in SaaS.

==**Exam Tip:** CapEx = Upfront Cost. OpEx = Pay as you go.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-02 Azure Global Infrastructure|AZ9-02 Azure Global Infrastructure]] — Regions and Availability Zones.
- [[04-Cloud-and-Security/08-Azure/AZ9-03 Azure Compute Services|AZ9-03 Azure Compute Services]] — Azure VM (IaaS) and App Service (PaaS) profiles.
- [[04-Cloud-and-Security/08-Azure/AZ9-06 Azure Cost and Governance|AZ9-06 Azure Cost and Governance]] — Cloud billing and pricing models.
