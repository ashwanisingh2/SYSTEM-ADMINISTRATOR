---
tags: [desktop-support, azure, cloud, L1]
aliases: [az9-03-azure-compute-services, az9-03]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#beginner` `#none`

# AZ9-03: Azure Compute Services

> [!abstract] Overview
> Yeh note Azure ke compute options cover karta hai. Isme Virtual Machines (IaaS), App Services (PaaS), Functions (Serverless), aur Containers (ACI/AKS) shamil hain. Kis task ke liye kaunsa compute service best hai, yeh is note mein samjhaya gaya hai.

---
## 🧠 Concept Overview

- **What it is** — Cloud mein code run karne ke alag-alag tareeqe, from full OS control (VMs) to code-only execution (Functions).
- **Why it matters** — Sahi compute service chunne se paisa aur management time dono bachta hai.
- **Where you see this** — Web hosting ke liye App Service use karna, ya legacy apps ke liye Windows VM banana.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | VM start/stop karna aur basic status check karna. |
| **L2** | App Services deploy karna aur Scale Sets configure karna. |
| **L3** | Kubernetes (AKS) clusters design karna aur serverless microservices architecture banana. |

> [!tip] Seedha Simple Mein
> *Virtual Machine khud gadi chalane jaisa hai (maintenance aapka), App Service taxi jaisa hai (bas destination batao), aur Functions ek delivery drone hai jo sirf kaam aane par start hota hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Compute Options** are like choosing a **Commercial Vehicle** because...
>
> - **Azure VMs (IaaS)** is buying a cargo van. You maintain the engine and drive it, but control everything inside.
> - **Azure App Service (PaaS)** is hiring a taxi. You don't care about maintenance, just give the driver your code and pay for the ride.
> - **Azure Functions (Serverless)** is renting an automated drone. It wakes up when a delivery request triggers, flies, lands, and turns off. You pay per second of flight.

---
## 🔬 Technical Deep Dive

### 1. Azure Virtual Machines (VMs)

- **Series:** B (Burstable, cheap), D (General, web), E (Memory-optimized, DBs), F (Compute-optimized, batch).
- **Naming Example:** `Standard_D4s_v5` (D-Series, 4 cores, SSD support, v5 hardware).

### 2. High Availability: Fault vs. Update Domains

> [!info] Key Concept
> Protect VMs using Availability Sets.

- **Fault Domain (FD):** Physical racks with common power/network. Prevents hardware crash impact.
- **Update Domain (UD):** Logical groups for Microsoft patching reboots.

### 3. VM Scale Sets (VMSS)

- Auto-scaling identical VMs horizontally based on CPU/RAM load.

### 4. Azure App Service (PaaS)

- Managed web hosting for .NET, Java, Node.js, PHP, Docker.

### 5. Serverless & Containers

- **Functions:** Event-driven, pay-per-execution code.
- **ACI (Container Instances):** Fastest way to run a single container.
- **AKS (Kubernetes Service):** Complex multi-container orchestration.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure Portal access.

### Step 1: Create a Linux Virtual Machine

1. Portal > **Virtual machine** > **Create**.
2. RG: `rg-lab-infrastructure`, Name: `vm-web-dev`, Region: East US.
3. Image: Ubuntu 22.04 LTS. Size: `Standard_B1s`.
4. Authentication: SSH public key, Username: `azureuser`.
5. Allow Ports: SSH (22), HTTP (80).
6. Click **Review + create**. Download the `.pem` key file.

### Step 2: Verify SSH Access

```bash
ssh -i ~/Downloads/vm-web-dev_key.pem azureuser@[VM_Public_IP]
```
> [!success] Expected Output
> You are logged into the Ubuntu bash shell.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az vm create` | Provisions a VM | `az vm create -n MyVm -g MyResourceGroup --image Ubuntu2204` |
| `az vm start / stop` | Manage VM power state | `az vm stop -n MyVm -g MyResourceGroup` |
| `az webapp up` | Deploy code to App Service | `az webapp up -n MyWebApp -l eastus` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| SSH connection timeout | NSG rule missing or wrong IP | Ensure Network Security Group allows Port 22 from your public IP. |
| App Service HTTP 500 | Application code crash | Go to App Service > Diagnostic Logs, turn on Application Logging and view stream. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Web App Scaling

> [!example] Ticket
> "Our application server is crashing every Friday afternoon due to high traffic."

**L1 Response:** Check CPU metrics in Azure Portal.
**Escalation Trigger:** Transition to Scale Sets or App Service.
**L2 Resolution:** If it's a VM, migrate to VM Scale Sets (VMSS) with rules to scale out on Friday afternoons. If App Service, configure Auto-Scale rules based on CPU > 80%.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between a Fault Domain and an Update Domain?
> **Answer:** Fault Domains map to physical hardware racks (power/network); they protect against unexpected crashes. Update Domains map to logical segments that Microsoft reboots one by one during planned maintenance/patching.

> [!question] Q2: Which service is best for running a script that fires only when a user uploads a photo?
> **Answer:** Azure Functions. It's event-driven and you only pay for the few seconds the script runs, saving money over a 24/7 VM.

==**Exam Tip:** App Services are PaaS, VMs are IaaS, Functions are Serverless.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — Service model comparisons.
- [[04-Cloud-and-Security/08-Azure/AZ9-04 Azure Storage and Networking|AZ9-04 Azure Storage and Networking]] — Virtual networks for VM interfaces.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Advanced sizing and ARM template deployments.
