---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-03-azure-virtual-machines, az104-03]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#advanced` `#az-104`

# AZ104-03: Azure Virtual Machines

> [!abstract] Overview
> Yeh note Azure VMs ke management, disk SKUs, proximity placement, custom extensions, autoscale, aur ARM template deployments ko cover karta hai.

---
## 🧠 Concept Overview

- **What it is** — Advanced configuration and management of Azure Virtual Machines.
- **Why it matters** — Optimizing performance, cost, and availability for IaaS workloads.
- **Where you see this** — Resizing VMs, troubleshooting VM performance, configuring backups.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Restart VMs, verify VM status in portal. |
| **L2** | Configure, fix, escalate — Resize disks, configure backups in Recovery Services Vault. |
| **L3** | Architecture, design — Configure Autoscale, write ARM templates, use Proximity Placement Groups. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Cloud mein servers (VMs) banana, unke hard disks ki speed set karna, aur unko auto-scale aur backup karna.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Virtual Machines** is like **leasing office space** because...
>
> - **Managed Disks** are your filing cabinets (SSD for fast access, HDD for archives).
> - **Proximity Placement Groups** ensure your team sits in the same room (low latency).
> - **ARM Templates** are the blueprints to build the exact office anywhere instantly.

---
## 🔬 Technical Deep Dive

### 1. Managed Disk SKUs
- **Standard HDD**: Low cost, archives.
- **Premium SSD**: High IOPS for databases.
- **Ultra Disk**: Sub-millisecond latency, independently scalable IOPS/throughput.

### 2. VMSS and Autoscale
Virtual Machine Scale Sets (VMSS) allow you to load balance identical VMs and automatically scale out/in based on metrics like CPU usage.

### 3. Proximity Placement Groups (PPG)
Forces VMs to be physically close in the same datacenter to minimize network latency.

### 4. Custom Script Extensions
Executes PowerShell or Bash scripts automatically inside the guest OS after deployment.

> [!danger] Common Mistake
> Trying to use Azure Backup on Ultra Disks. Standard Azure Backup doesn't support them.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Active Azure Subscription
> - Linux VM deployed

### Step 1: Deploy Custom Script Extension

```bash
# Upload script to install NGINX
#!/bin/bash
apt-get update && apt-get install -y nginx
```

> [!success] Expected Output
> ```
> NGINX welcome page displays when navigating to VM's public IP.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `systemctl status <service>` | Check Linux service status | `systemctl status nginx` |
| `Get-Service` | Check Windows service status | `Get-Service w3svc` |
| `Test-NetConnection` | Check network port connectivity | `Test-NetConnection -Port 80` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| VM running slow/high IO latency | Disk throttling | Upgrade to Premium SSD or increase disk size for more IOPS |
| Cannot RDP/SSH into VM | NSG blocking port | Add inbound security rule for port 3389/22 in Network Security Group |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: VM Disk Space Full

> [!example] Ticket
> "Database VM C: drive is 99% full, need more space."

**L1 Response:** Verify disk space in OS.
**Escalation Trigger:** Needs Azure portal disk expansion.
**L2 Resolution:** Stop VM, expand Managed Disk size in Azure portal, start VM, and extend volume in Windows Disk Management.

---
## 🎤 Interview Questions

> [!question] Q1: What is a Proximity Placement Group?
> **Answer:** A logical grouping that ensures VMs are physically placed close together in the same datacenter to achieve the lowest possible network latency.

==**Exam Tip:** ARM templates are written in JSON and consist of parameters, variables, resources, and outputs.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-03 Azure Compute Services|AZ9-03 Azure Compute Services]]
- [[04-Cloud-and-Security/08-Azure/AZ104-06 Azure Monitor and Backup|AZ104-06 Azure Monitor and Backup]]
