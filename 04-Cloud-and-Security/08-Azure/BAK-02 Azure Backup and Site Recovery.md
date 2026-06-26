---
tags: [desktop-support, azure, cloud, L2]
aliases: [bak-02-azure-backup-and-site-recovery, bak-02]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#advanced` `#az-104`

# BAK-02: Azure Backup and Site Recovery

> [!abstract] Overview
> Is note mein hum samjhenge Azure Backup aur Azure Site Recovery (ASR) ko, jo Microsoft ke Business Continuity and Disaster Recovery (BCDR) strategy ke pillars hain. Ek support engineer ke liye yeh janna bahut zaroori hai ki disaster (like ransomware, hardware fail) ke time par server ko cloud se kaise restore karein aur applications ko online kaise rakhein.

---
## 🧠 Concept Overview

- **What it is** — Azure Backup is a cloud-native backup solution that retains historical data points. Azure Site Recovery (ASR) replicates workloads in real-time, allowing immediate failover to a recovery site during disasters.
- **Why it matters** — A modern sysadmin must protect company infrastructure from localized hardware failures, ransomware, and natural disasters. Knowing how to restore servers and initiate failover systems is critical.
- **Where you see this** — Configuring a daily backup policy for Azure Virtual Machines, installing the MARS agent to back up an on-premises file server to Azure, or executing a disaster recovery test drill.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Monitors backup logs in the Recovery Services Vault, verifies replication health in the portal, and restores files using the Azure Recovery instant mount. |
| **L2** | Deploys Recovery Services Vaults, configures backup policies, installs/registers MARS agent on-premises, and performs full VM restores. Escalate kab karta hai? Jab replication synchronization stuck ho ya VM failover execute karna ho. |
| **L3** | Designs ASR replication patterns, configures Recovery Plans with custom failover scripts, manages network mapping between sites, and sets RTO/RPO metrics. |

> [!tip] Seedha Simple Mein
> *Azure Backup server data ke historical backups leta hai taaki files delete hone par recovery ho sake. Azure Site Recovery (ASR) server ki copy run-time me doosre Azure region me save rakhta hai, taaki agar main datacenter band ho jaye toh backup server turant start ho sake.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Backup & Site Recovery** is like **Car Insurance vs. Spare Tire** because...
>
> - **Azure Backup** is like Car Insurance — it helps you recover your car if it's completely destroyed, but it takes some time to process the claim and get a new one (RTO is longer).
> - **Azure Site Recovery** is like a Spare Tire — if you get a flat tire, you can immediately switch to the spare and keep driving with almost zero delay (RTO is very short).

---
## 🔬 Technical Deep Dive

### 1. BCDR Metrics: RTO and RPO

> [!info] Key Concept
> These two metrics define your disaster recovery limits and SLA.

- **Recovery Time Objective (RTO)**: The maximum acceptable duration of downtime before service is restored (How fast do we need to recover?).
- **Recovery Point Objective (RPO)**: The maximum acceptable data loss measured in time (How much data can we afford to lose? e.g., 4 hours of data).

### 2. Azure Backup Methods

- **Azure VM Backup**: Backs up entire Azure VMs using snapshots. No agent needed inside the VM.
- **MARS (Microsoft Azure Recovery Services) Agent**: Installed on-premises servers. Backs up files, folders, and system state directly to Azure Recovery Services Vault.
- **MABS (Microsoft Azure Backup Server)**: On-premises backup server that caches backups locally before sending them to Azure, supporting SQL, Exchange, and SharePoint.

### 3. Recovery Services Vault (RSV)

> [!info] Key Concept
> The RSV is the online storage entity in Azure. It stores backup data and holds recovery points.

Storage redundancy options:
- **LRS (Locally Redundant Storage)**: Three copies in a single datacenter.
- **GRS (Geo-Redundant Storage)**: Replicated to a secondary paired region (Default and recommended for safety).
- **ZRS (Zone-Redundant Storage)**: Replicated across Availability Zones.

> [!danger] Common Mistake
> Creating the Recovery Services Vault in a different region than the VMs you want to back up. The RSV must be in the same region as the resources!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Azure Subscription.
> - An active Azure VM (`VM-Production`) running in `East US`.
> - A Resource Group named `RG-BCDR`.

### Step 1: Create a Recovery Services Vault

```powershell
# Step 1.1: Install Az PowerShell module (if not installed)
Install-Module -Name Az -AllowClobber -Force

# Step 1.2: Connect to Azure Account
Connect-AzAccount

# Step 1.3: Create the Recovery Services Vault in the Resource Group
New-AzRecoveryServicesVault -Name "RSV-ProdBackup" -ResourceGroupName "RG-BCDR" -Location "East US"

# Step 1.4: Set storage redundancy to Geo-Redundant (GRS)
$vault = Get-AzRecoveryServicesVault -Name "RSV-ProdBackup" -ResourceGroupName "RG-BCDR"
Set-AzRecoveryServicesBackupProperty -Vault $vault -BackupStorageRedundancy GeoRedundant
```

> [!success] Expected Output
> The vault is created with properties showing `BackupStorageRedundancy : GeoRedundant`.

### Step 2: Configure Azure VM Backup Policy and Enable Backup

```powershell
# Step 2.1: Set vault context
Get-AzRecoveryServicesVault -Name "RSV-ProdBackup" | Set-AzRecoveryServicesVaultContext

# Step 2.2: Enable backup protection for VM-Production
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"
Enable-AzRecoveryServicesBackupProtection -ResourceGroupName "RG-BCDR" -Name "VM-Production" -Policy $policy
```

> [!success] Expected Output
> Backup protection enabled status: `Passed`.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `New-AzRecoveryServicesVault` | Naya RSV vault banata hai | `New-AzRecoveryServicesVault -Name "VaultName" -ResourceGroupName "RG" -Location "EastUS"` |
| `Get-AzRecoveryServicesBackupJob` | Vault mein backup jobs list karta hai | `Get-AzRecoveryServicesBackupJob -Status Failed` |
| `Backup-AzRecoveryServicesBackupItem` | Ad-hoc VM backup trigger karta hai | `Backup-AzRecoveryServicesBackupItem -Item $backupItem` |
| `Restore-AzRecoveryServicesBackupItem` | Restore points se VM disks restore karta hai | `Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp -StorageAccountName "storage"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **VM Backup fails with Guest Agent status Unavailable** | The Azure VM Agent is not installed, outdated, or service stopped. | Install/Update Azure VM Agent inside VM OS. Ensure `Windows Azure Guest Agent` service is running. |
| **ASR replication state remains Critical** | Network connectivity issues or storage write churn is too high. | Verify network bandwidth. Ensure source VM disk write rates do not exceed limits (e.g., 10MB/s per disk for standard SSDs). |
| **MARS backup job fails with network timeout** | Firewall or Proxy server blocking Azure backup endpoints. | Allow outbound connections to Azure backup URLs (e.g., `*.backup.windowsazure.com`) over port 443. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Failed Backup Job Alert

> [!example] Ticket
> "Alert: VM-Production backup failed with error 'Guest Agent is unavailable'. Please resolve to maintain compliance."

**L1 Response:** Check the VM in Azure portal and verify if the VM status is running. Check VM extension status.
**Escalation Trigger:** If the VM is accessible but the agent cannot be restarted or reinstalled easily.
**L2 Resolution:** Connect to the VM via RDP/SSH. Check the `Windows Azure Guest Agent` service. If it's stopped, start it. If corrupted, reinstall the Azure VM Agent manually.

### 🎫 Scenario 2: Need File Restored from Yesterday

> [!example] Ticket
> "User accidentally deleted the 2025 Financial Report from the shared drive on VM-Finance. We need it restored from yesterday's backup."

**L1 Response:** Go to RSV -> Backup Items -> VM-Finance. Select 'File Recovery' for yesterday's restore point. Download and run the executable to mount the recovery disk locally, copy the file, and unmount.
**Escalation Trigger:** If the file recovery mount fails or entire disk restore is needed.
**L2 Resolution:** Perform a full disk restore or investigate why the recovery script is failing to mount.

---
## 🎤 Interview Questions

> [!question] Q1: What is the main structural difference between Azure Backup and Azure Site Recovery?
> **Answer:** **Azure Backup** captures and stores backups of data over long periods, offering point-in-time recovery points. **Azure Site Recovery (ASR)** replicates virtual machines in real-time to a secondary region, maintaining a near-live state to allow rapid failover.

==**Exam Tip:** Backup is for data retention (RPO). Site Recovery is for business continuity and disaster failover (RTO).==

> [!question] Q2: How does the Azure VM backup run without affecting virtual machine operations?
> **Answer:** It uses the Azure VM extension to interact with host-level VSS (Windows) or fsfreeze (Linux) to quiesce the VM filesystem. It takes a storage-level snapshot of the VM's managed disks, and then resumes full write operations while copying data to RSV.

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ104-06 Azure Monitor and Backup|AZ104-06 Azure Monitor and Backup]]
- [[04-Cloud-and-Security/08-Azure/Azure-Backup|Azure-Backup]]
- [[03-Identity-and-Core-Services/05-Windows-Server/BAK-01 Windows Server Backup|BAK-01 Windows Server Backup]]
