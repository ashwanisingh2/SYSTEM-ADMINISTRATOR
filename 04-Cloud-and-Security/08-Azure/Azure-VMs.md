---
tags: [desktop-support, azure, cloud, L2]
aliases: [azure-vms]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#intermediate` `#az-104`

# Azure Virtual Machines

> [!abstract] Overview
> Yeh note Azure Virtual Machines (IaaS) ko cover karta hai. Ek support engineer ke liye yeh jaanna zaroori hai kyunki organizations apne servers, databases, aur remote desktops ko Azure par host karti hain. VMs ka lifecycle manage karna, disks attach karna aur connection failures troubleshoot karna daily job ka hissa hai.

---
## 🧠 Concept Overview

- **What it is** — Azure Virtual Machines on-demand, scalable, virtualized computing resources (IaaS) hain jo Microsoft ke data centers mein host hoti hain, aur Windows ya Linux chalaati hain.
- **Why it matters** — Support engineers ko VM lifecycle (start/stop), storage disks, aur Boot diagnostics manage karna aana chahiye taaki system uptime maintain rahe.
- **Where you see this** — Jab VM resize karna ho, unresponsive VMs ko Redeploy karna ho, ya nayi data disks mount karni ho.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic power cycles (Start, Stop, Restart), status check, aur connection failures ko escalate karna. |
| **L2** | VMs resize karna, virtual data disks add/format karna, Boot Diagnostics check karna, aur stuck VMs ko redeploy karna. |
| **L3** | Auto-Scaling rules design karna, Dedicated Hosts manage karna, backup retention, aur Azure Migrate se large scale migrations. |

> [!tip] Seedha Simple Mein
> *Azure VMs basically cloud mein rakhe hue computers hain. Jaise apne office ka physical server, waise hi cloud ka VM jismein OS, CPU, RAM aur Disk hota hai, bas humein iski hardware ki chinta nahi karni padti.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure VM** is like a **Rented Office Space** because...
>
> - Jab aapko zarurat hoti hai, aap space rent karte ho (VM start karte ho) aur jab band karte ho (deallocate), toh rent nahi lagta.
> - Agar aapka saman zyada ho jaye, toh aap nayi almirah laate ho (persistent Data Disk add karte ho).
> - Jo common lobby hoti hai wahan rakha saman safe nahi (Temporary D: drive), maintenance mein gayab ho sakta hai.

---
## 🔬 Technical Deep Dive

### 1. VM Sizing & Families

> [!info] Key Concept
> Azure VMs alag-alag hardware configurations mein aati hain jise 'Families' kehte hain, workload ke hisaab se.

- **A/B-Series**: Burstable, development servers ke liye.
- **D-Series**: General purpose, Active Directory, standard apps.
- **F-Series**: Compute optimized, web servers, processing.
- **E/M-Series**: Memory optimized, SQL Server, SAP HANA.

### 2. Disk Storage Architecture

> [!info] Key Concept
> Ek standard Azure VM mein 3 type ke disks hote hain: OS Disk, Temporary Disk, Data Disk.

1. **OS Disk**: Persistent storage (C: drive).
2. **Temporary Disk**: Ephemeral storage host server pe (D: drive).
3. **Data Disks**: Persistent SCSI disks (E:, F: drives).

> [!danger] Common Mistake
> Kabhi bhi apna data, backups ya databases **D: drive (Temporary Storage)** par save mat karna. Jab VM deallocate hota hai ya host maintenance hoti hai, toh yeh drive completely wipe out ho jaati hai!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure subscription / Sandbox
> - Azure Az PowerShell module logged in

### Step 1: Query VM Status

```powershell
# Get the status of a lab VM
Get-AzVM -ResourceGroupName "RG-Lab" -Name "VM-Test" -Status | Select-Object Name, PowerState
```

> [!success] Expected Output
> ```
> Name      PowerState
> ----      ----------
> VM-Test   VM running
> ```

### Step 2: Stop and Deallocate VM

```powershell
# Deallocate to release charges and prep for resize
Stop-AzVM -ResourceGroupName "RG-Lab" -Name "VM-Test" -Force -Confirm:$false
```

### Step 3: Resize the VM

```powershell
# Change VM size to Standard B2s
$VM = Get-AzVM -ResourceGroupName "RG-Lab" -Name "VM-Test"
$VM.HardwareProfile.VmSize = "Standard_B2s"
Update-AzVM -ResourceGroupName "RG-Lab" -VM $VM
```

### Step 4: Attach New Data Disk

```powershell
# Create and attach a 50GB Premium SSD
$DiskConfig = New-AzDiskConfig -Location $VM.Location -CreateOption Empty -DiskSizeGB 50 -SkuName Premium_LRS
$NewDisk = New-AzDisk -DiskName "VM-Test-Data01" -Disk $DiskConfig -ResourceGroupName "RG-Lab"
$VM = Add-AzVMDataDisk -VM $VM -Name "VM-Test-Data01" -CreateOption Attach -ManagedDiskId $NewDisk.Id -Lun 1
Update-AzVM -ResourceGroupName "RG-Lab" -VM $VM
```

### Step 5: Start VM

```powershell
# Start the VM
Start-AzVM -ResourceGroupName "RG-Lab" -Name "VM-Test"
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Start-AzVM` | VM ko start karta hai | `Start-AzVM -ResourceGroupName "RG" -Name "VM-Web-01"` |
| `Stop-AzVM -Force` | VM ko stop aur deallocate karta hai (billing stop) | `Stop-AzVM -ResourceGroupName "RG" -Name "VM-Web-01" -Force` |
| `Restart-AzVM -Redeploy` | Stuck VM ko naye hardware host par move karke boot karta hai | `Restart-AzVM -ResourceGroupName "RG" -Name "VM-Web-01" -Redeploy` |
| `az vm deallocate` | Azure CLI se VM deallocate karta hai | `az vm deallocate -g "RG" -n "VM-Web-01"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **RDP/SSH Timeout** | Network Security Group (NSG) rules ya firewall block kar raha hai. | NSG rules check karein (Port 3389/22) aur IP allow karein. Boot diagnostics dekhein. |
| **High Disk Latency** | Standard SSD/HDD I/O limit hit kar raha hai. | Premium SSD ya Ultra Disk attach karein databases ke liye. |
| **Data on D: drive missing** | VM deallocate hua tha ya host maintainance hui, aur D: ephemeral hai. | User ko educate karein aur nayi persistent Data Disk (E:) attach karke usko configure karein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Unresponsive VM (Stuck Windows Update)

> [!example] Ticket
> "I cannot connect to our testing server 'VM-Dev-04' via RDP. It times out. The Azure Portal says the VM status is 'Running', but I cannot reach it."

**L1 Response:** Check public/private IP, verify NSG rules (Port 3389). Check Boot Diagnostics in Azure Portal.
**Escalation Trigger:** Boot Diagnostics screenshot shows VM is stuck at "Working on updates 100%".
**L2 Resolution:** Wait 30 minutes. Agar phir bhi stuck hai, Azure portal se "Redeploy + reapply" blade mein jao aur **Redeploy** par click karo taaki VM naye host par shift ho aur boot properly ho.

### 🎫 Scenario 2: Data Disappears After VM Deallocation

> [!example] Ticket
> "I restarted our SQL server 'VM-SQL-Prod'. Today, our backup directory 'D:\Backups' has completely vanished."

**L1 Response:** Verify disk configuration in Azure portal. Check that D: is the Temporary Storage drive.
**Escalation Trigger:** Data cannot be recovered. Need to configure a proper data disk.
**L2 Resolution:** Explain to user that D: drive is ephemeral and data is lost. Provision a new Premium SSD (e.g., 100GB), attach it to the VM, format it inside Windows as E: drive, aur SQL backup path E: par shift karo. GPO lagao taaki users D: par data save na karein.

---
## 🎤 Interview Questions

> [!question] Q1: A Windows VM is stuck on boot and you cannot log in. How do you troubleshoot this?
> **Answer:** I check the **Boot Diagnostics** blade in the Azure Portal. This displays a screenshot of the virtual monitor, which lets me see if the server is performing updates, run checking disk utility, or has crashed with a BSOD. I also review the serial console logs.

==**Exam Tip:** Remember that Boot Diagnostics requires a storage account or managed storage to keep logs and screenshots.==

> [!question] Q2: What is the difference between "Stopping" a VM in the guest OS vs Deallocating in Azure?
> **Answer:** Shutting down inside the guest OS leaves the VM in a "Stopped (Allocated)" state where CPU/RAM are still reserved and billed. Stopping it via Azure Portal/CLI transitions it to "Stopped (Deallocated)", which stops billing charges.

> [!question] Q3: What is the "Redeploy" action in Azure, and when would you use it?
> **Answer:** Redeploy shuts down a VM, migrates its virtual disks to a different physical host server inside the Azure datacenter, and restarts it. I use it when a VM is completely unresponsive, or the underlying hardware host has failed.

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]] — RBAC roles to administer VMs.
- [[04-Cloud-and-Security/08-Azure/Azure-Networking|Azure Networking]] — Virtual subnets and NSG rules connecting VMs.
- [[04-Cloud-and-Security/09-Security/BitLocker-Deep-Dive|BitLocker Deep Dive]] — VM disk encryption keys.
