---
tags: [desktop-support, azure, cloud, L2]
aliases: [azure-backup, azure-backup]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#intermediate` `#az-104`

# AZURE-01: Azure Backup and Recovery

> [!abstract] Overview
> Azure Backup ek cloud-based backup-as-a-service (BaaS) solution hai jo virtual machines aur on-premises servers ke data ko protect karta hai. Ek support engineer ko yeh jaanna bohot zaroori hai kyunki hard drive crashes, ransomware, ya accidental deletion ke case mein data ko restore karna daily job ka part hota hai.

---
## 🧠 Concept Overview

- **What it is** — Azure Backup is a cloud-based backup-as-a-service (BaaS) solution that protects data on virtual machines in Azure and on-premises physical or virtual servers. It stores backups in Recovery Services Vaults.
- **Why it matters** — Hard drive crashes, database corruption, ransomware infections, and accidental file deletions happen. We need it to restore files, mount virtual disks, and rebuild servers.
- **Where you see this** — Restoring single files from a VM backup snapshot, monitoring daily backup jobs, configuring policies, and restoring deleted VMs.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Monitors daily backup jobs, checks for failed backups, and escalates backup agent errors. |
| **L2** | Performs file-level restores using the Azure recovery mount utility, runs VM restore tasks, and installs backup agents. |
| **L3** | Configures Recovery Services Vault architecture, designs cross-region restore (CRR) rules, manages redundancy. |

> [!tip] Seedha Simple Mein
> *Azure-Backup aapke VM ka data safe rakhta hai cloud mein. Galti se kuch delete ho jaye ya server crash ho, toh vault se sidha restore maar sakte ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Backup** is like **a Time Machine for your server** because...
>
> - You can go back to exactly how the server looked yesterday.
> - You can grab just one missing file from the past without rewriting the present.

---
## 🔬 Technical Deep Dive

### 1. Recovery Services Vault (RSV)

> [!info] Key Concept
> An RSV is an online storage entity in Azure used to hold data such as backup copies, recovery points, and backup policies.

- **Backup Policies**: Define *when* backups run (daily, weekly) and *how long* they are kept (retention range).
- **Storage Redundancy**:
  - **Locally Redundant Storage (LRS)**: Replicates backup data three times within a single datacenter. Lowest cost.
  - **Geo-Redundant Storage (GRS)**: Replicates backup data to a secondary paired region hundreds of miles away. Protects against regional disasters.

> [!danger] Common Mistake
> Trying to change vault redundancy after backing up data. You must set the storage redundancy *before* protecting any items in the vault!

### 2. Azure VM Backup & File-Level Restore

Azure backs up running VMs using a hypervisor-level snapshot. It does not require shutting down the VM:
- **Instant Restore**: Azure stores snapshots locally on the storage account for up to 5 days, allowing fast recoveries.
- **Vault Restore**: Snapshots are compressed and transferred to the RSV for long-term retention.

### 3. Soft Delete Security Feature

> [!info] Key Concept
> Soft Delete protects backup data against accidental deletion or ransomware attacks.

- **How it works**: If an admin clicks "Stop backup and delete backup data", it is not immediately erased.
- **Retention**: It is held for **14 days** at no charge and can be fully undeleted.

> [!danger] Common Mistake
> Never disable Soft Delete unless absolutely required for testing. If disabled, a compromised account could instantly wipe all historical backups!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure subscription with a running Virtual Machine.
> - Azure Az PowerShell module logged in.

### Step 1: Trigger a Manual Backup

```powershell
# Set the vault context
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "RG-Lab" -Name "Vault-Lab"
Set-AzRecoveryServicesAsVaultContext -Vault $vault

# Retrieve protected VM
$container = Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" -FriendlyName "VM-LabServer"
$item = Get-AzRecoveryServicesBackupItem -Container $container -WorkloadType "AzureVM"

# Trigger manual backup with 14 days retention
$backupJob = Backup-AzRecoveryServicesBackupItem -Item $item -ExpiryDateTimeUTC (Get-Date).AddDays(14)
```

> [!success] Expected Output
> Verify in the portal that the backup status changes from `InProgress` to `Completed` or run `Get-AzRecoveryServicesBackupJob -Job $backupJob`.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-AzRecoveryServicesVault` | Get the vault object | `Get-AzRecoveryServicesVault -Name "Vault"` |
| `Set-AzRecoveryServicesAsVaultContext` | Sets vault context for subsequent commands | `Set-AzRecoveryServicesAsVaultContext -Vault $Vault` |
| `Get-AzRecoveryServicesBackupItem` | Lists protected items (like VMs) in the vault | `Get-AzRecoveryServicesBackupItem -WorkloadType "AzureVM"` |
| `Backup-AzRecoveryServicesBackupItem` | Triggers a manual backup | `Backup-AzRecoveryServicesBackupItem -Item $Item -ExpiryDateTimeUTC <Date>` |
| `az backup job list` | CLI to list backup jobs | `az backup job list --output table` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Service connection timeout** | Network firewall or routing blocking traffic | Check network route and enable target ports (like TCP 3260 for iSCSI) |
| **Access Denied error** | User lacks permissions | Verify account access permissions in Azure RBAC |
| **Resource not found** | Object path misspelled | Verify spelling of target path or query active objects |
| **File-Level restore script fails** | Defender block or missing ports | Check if Windows Defender blocked it, or open TCP 3260/445 |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Recovering a Single Lost Excel File

> [!example] Ticket
> "I accidentally deleted our primary project tracker spreadsheet from the shared E: drive on 'VM-FileShare'. Can you restore this file from last night's backup?"

**L1 Response:** Locate the backup status of `VM-FileShare` in the RSV and confirm recovery points exist.
**Escalation Trigger:** Pass to L2 to perform the actual file restore.
**L2 Resolution:**
1. Download File Recovery script (`VolumeMount.exe`) from Azure portal.
2. RDP into `VM-FileShare`, run script (mounts backup as `G:` drive via iSCSI).
3. Copy `Project-Tracker.xlsx` from `G:` to `E:`.
4. Unmount drive (`Q` in PowerShell).

### 🎫 Scenario 2: Rebuilding a Deleted Virtual Machine

> [!example] Ticket
> "VM-AppServer-01 has been deleted from the resource group. The website is down. We need the server restored immediately."

**L1 Response:** Verify backup history and Activity logs to confirm VM deletion.
**Escalation Trigger:** Pass to L2/L3 to perform the Full VM Restore.
**L2 Resolution:**
1. Navigate to RSV -> Select the deleted VM -> Click **Restore VM**.
2. Select **Create New VM**, input original VNet and Resource Group.
3. Once restored (~20 mins), verify network interfaces and start VM.
4. Manually re-assign Static Public IP or update DNS if needed (trap!).

---
## 🎤 Interview Questions

> [!question] Q1: How does the Soft Delete feature protect Azure backups?
> **Answer:** Soft Delete preserves backup data for 14 days after an administrator or attacker attempts to delete it. It acts as a safety net against malicious deletions or ransomware.

==**Exam Tip:** Soft Delete is kept for 14 days at NO extra cost.==

> [!question] Q2: What is the difference between LRS and GRS?
> **Answer:** LRS replicates data 3 times within a single datacenter. GRS replicates data to a secondary paired region hundreds of miles away.

> [!question] Q3: If a user deletes a file on an Azure VM, do you have to restore the entire VM?
> **Answer:** No, I would use the **File Recovery** feature to generate a script that mounts the snapshot as a local drive inside the VM, allowing me to copy just the missing file.

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]] — Virtual machines protected by the recovery vaults.
- [[04-Cloud-and-Security/09-Security/BitLocker-Deep-Dive|BitLocker Deep Dive]] — Detail disk encryption key integrations.
- [[01-Foundations/01-Hardware/Storage-Deep-Dive|Storage Deep Dive]] — Outlines the core raid and redundancy architectures.
