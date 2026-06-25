---
tags: [desktop-support, azure, backup, disaster-recovery, L2]
aliases: [azure-backup-guide, recovery-services, file-restore]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

# Azure Backup and Recovery

---

## Concept Overview
- **What it is**: Azure Backup is a cloud-based backup-as-a-service (BaaS) solution that protects data on virtual machines in Azure and on-premises physical or virtual servers. It stores backups in **Recovery Services Vaults**, managing encryption, retention, and restorations.
- **Why it matters for a support engineer**: Hard drive crashes, database corruption, ransomware infections, and accidental file deletions happen. A support engineer must know how to restore files, mount virtual disks from backup points, and rebuild servers from vault snapshots.
- **Where you encounter this in real job**: Restoring single files from a VM backup snapshot, monitoring daily backup job reports, configuring backup policies, and restoring deleted virtual machines.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Monitors daily backup jobs, checks for failed backups, and escalates backup agent errors.
  - **L2**: Performs file-level restores using the Azure recovery mount utility, runs VM restore tasks, and installs backup agents on local servers.
  - **L3**: Configures Recovery Services Vault architecture, designs cross-region restore (CRR) rules, manages backup storage redundancy, and implements Soft Delete overrides.

---

## Technical Deep Dive

### 1. Recovery Services Vault (RSV)
An RSV is an online storage entity in Azure used to hold data such as backup copies, recovery points, and backup policies:
- **Backup Policies**: Define *when* backups run (daily, weekly) and *how long* they are kept (retention range).
- **Storage Redundancy**:
  - **Locally Redundant Storage (LRS)**: Replicates backup data three times within a single datacenter. Lowest cost.
  - **Geo-Redundant Storage (GRS)**: Replicates backup data to a secondary paired region hundreds of miles away. Protects against regional disasters.
  - *Note*: You must set the storage redundancy *before* protecting any items in the vault. Once backup data is stored, the redundancy setting cannot be changed.

### 2. Azure VM Backup & File-Level Restore
Azure backs up running VMs using a hypervisor-level snapshot. It does not require shutting down the VM:
- **Instant Restore**: Azure stores snapshots locally on the storage account for up to 5 days, allowing fast recoveries.
- **Vault Restore**: Snapshots are compressed and transferred to the RSV for long-term retention.
- **File Recovery (Item-Level)**: If a user only deleted a single Excel file, you don't need to restore the entire VM.
  1. Go to the vault -> Click **File Recovery**.
  2. Select the recovery point. Azure generates a temporary executable script (`.exe` for Windows, `.sh` for Linux) and a passcode.
  3. Run the script inside the target VM Guest OS. It mounts the backup recovery point as a local network drive (e.g., `G:`).
  4. Copy the deleted file from the `G:` drive to the local disk.
  5. Unmount the drive.

### 3. Soft Delete Security Feature
Soft Delete protects backup data against accidental deletion or ransomware attacks:
- **How it works**: If a rogue administrator (or compromised account) clicks **Stop backup and delete backup data**, the backup is not immediately erased.
- **Retention**: The backup state is shifted to a "Soft Deleted" status and held for **14 days** at no charge. It can be fully undeleted during this window.

---

## Commands & Syntax

### PowerShell
Backup management uses the `Az.RecoveryServices` module.
```powershell
# Connect to Azure
Connect-MgGraph -Scopes "Directory.Read.All" # For account queries if needed
Connect-AzAccount

# Set the vault context
$Vault = Get-AzRecoveryServicesVault -ResourceGroupName "RG-Prod-Backup" -Name "RSV-Main-Vault"
Set-AzRecoveryServicesAsVaultContext -Vault $Vault

# Query all backup items (VMs) registered in the vault
Get-AzRecoveryServicesBackupItem -WorkloadType "AzureVM" | Select-Object FriendlyName, ProtectionStatus, HealthStatus

# Trigger a manual backup job for a specific VM
$Container = Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" -FriendlyName "VM-Web-01"
$Item = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType "AzureVM"
Backup-AzRecoveryServicesBackupItem -Item $Item -ExpiryDateTimeUTC (Get-Date).AddDays(30)
```

### Azure CLI
```bash
# Set default vault config
az backup vault env set --g RG-Prod-Backup --v RSV-Main-Vault

# Check the list of active backup jobs
az backup job list --output table
```

### GUI Path
- **Restore & Recovery**: Azure Portal -> Search **Recovery Services Vaults** -> Select Vault -> Click **Backup items** -> Select **Azure Virtual Machine** -> Click target VM -> Select **Restore VM** or **File Recovery**.

---

## Real-World Scenarios

### Scenario 1: Recovering a Single Lost Excel File from a VM Backup
**User Complaint:** A project coordinator reports: *"I accidently deleted our primary project tracker spreadsheet (Project-Tracker.xlsx) from the shared E: drive on our server 'VM-FileShare'. I emptied the local Recycle Bin. Can you restore this file from last night's backup?"*
**Your First 3 Checks:**
1. Locate the backup status of `VM-FileShare` in the Recovery Services Vault.
2. Check the list of available recovery points (snapshots) from the previous night.
3. Download the Azure File Recovery script to the VM.
**Diagnosis Steps:**
1. Open the Azure Portal. Navigate to the `RSV-Main-Vault` -> Click **Backup items** -> **Azure Virtual Machine** -> Select `VM-FileShare`.
2. Click **File Recovery** -> Select the Recovery Point from `Yesterday, 11:00 PM`.
3. Click **Download Executable** (saves a script called `VolumeMount.exe`) and copy the generated password.
4. RDP into `VM-FileShare` as administrator. Copy `VolumeMount.exe` onto the desktop.
5. Open PowerShell as Administrator. Run `.\VolumeMount.exe`.
6. Enter the password when prompted.
7. *Under the hood*: The script connects to the Azure recovery vault using iSCSI and mounts the VM's backup virtual disk locally.
8. In File Explorer, note that a new drive letter (`G:`) has appeared.
**Root Cause:** Accidental user deletion of a document on a persistent server volume.
**Fix:**
1. Open the `G:` drive (the backup mount). Go to the folder directory where the spreadsheet was located.
2. Copy `Project-Tracker.xlsx` from the `G:` drive.
3. Paste the file back into the active `E:` drive.
4. Go back to the PowerShell window on the server and press `Q` to unmount the backup disk volume.
5. Confirm the user can access their recovered document.
**Prevention:** Configure Windows Shadow Copies (VSS) on the local server drives to allow users to self-restore previous versions without helpdesk tickets.
**Ticket Close Note:** "Mounted backup snapshot locally on VM-FileShare. Restored the lost file to E: drive. Unmounted backup volume. User verified file integrity. Closed."

### Scenario 2: Rebuilding a Deleted Virtual Machine (Full VM Restore)
**User Complaint:** A helpdesk alert triggers: *"VM-AppServer-01 has been deleted from the resource group 'RG-Prod-App'. The application website is down. We need the server restored immediately."*
**Your First 3 Checks:**
1. Verify the backup history of the deleted VM in the Recovery Services Vault.
2. Check the Activity logs to confirm who deleted the VM.
3. Determine if we should restore the entire VM or just the virtual disks.
**Diagnosis Steps:**
1. Navigate to the `RSV-Main-Vault` -> **Backup items** -> Select the deleted `VM-AppServer-01` (it is still listed in the vault because the backup data was not deleted).
2. Click **Restore VM**.
3. Select the latest Recovery Point (from 4 hours ago).
4. Review the Restore Options:
   - **Create New VM**: Azure creates the VM, hooks up the disks, and configures the virtual CPU/RAM automatically.
   - **Restore Disks**: Azure only restores the `.vhd` disk files to a storage account, allowing you to manually attach them to a VM.
5. *Decision*: To restore service as fast as possible, select **Create New VM**. Input the original configuration parameters (Virtual Network, Subnet, and Resource Group).
**Root Cause:** A VM was deleted from a resource group, causing a service outage.
**Fix:**
1. In the restore wizard, select the VNet and target RG.
2. Click **Restore**. Track progress in the **Backup jobs** tab.
3. The restore finishes in 20 minutes (using the Instant Restore snapshot tier).
4. Open the VM properties, verify the new network interfaces are bound, and start the VM.
5. Verify the web application resumes normal operations.
**Prevention:** Place a `CanNotDelete` Resource Lock on all production resource groups.
**Ticket Close Note:** "Restored VM-AppServer-01 from the latest vault recovery point. Virtual machine successfully recreated and powered on. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never disable **Soft Delete** in your Recovery Services Vault properties unless absolutely required for testing.
> - If an attacker gains access to your credentials, disabling Soft Delete allows them to immediately delete all historical backups, leaving you with zero recovery options during a ransomware attack.

> [!warning] Common Trap
> - Assuming that a VM restore task automatically preserves the original public IP address of the virtual machine.
> - When you restore a VM, Azure assigns a new dynamic public IP address. If your DNS records or external firewall rules point to the old IP, services will remain broken. You must manually assign a Static Public IP or update DNS entries after the restore.

> [!tip] Senior Engineer Tip
> - When running File-Level restores on Windows Server VMs, if the download script fails to run or block ports, check if Windows Defender is blocking the download, or if the server lacks outbound TCP ports 3260 (iSCSI) and 445 (SMB) to mount the cloud drive.

> [!success] Verification Steps
> - Run `Get-AzRecoveryServicesBackupJob -Status InProgress` to monitor active restores.
> - Verify in the portal that the backup status is listed as `Protected (Excellent)`.

> [!question] Interview Alert
> - "How does the Soft Delete feature protect Azure backups?"
> - Answer: "Soft Delete preserves backup data for 14 days after an administrator or attacker attempts to delete it. During these 14 days, the backup data is marked as soft-deleted but is not actually deleted. It can be recovered without additional cost, protecting the organization against malicious deletions or ransomware."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Trying to change vault redundancy after backing up data | Unaware of block rule | Configure storage redundancy (LRS vs GRS) immediately after creating the vault before adding items. |
| Restoring the entire VM when only one file is missing | Deeming it the simplest UI action | Use the File Recovery mount script to extract the single file in 5 minutes, avoiding VM downtime. |
| Forgetting to monitor failed backup jobs | Assuming automated systems never fail | Configure automated email notifications for failed backup alerts in Azure Monitor. |

---

## Lab Exercise

**Objective:** Use PowerShell to locate a Recovery Services Vault, check backup items, trigger a manual backup job, and track the backup status.
**Time Required:** 30 minutes
**Environment Needed:** Azure subscription with a running Virtual Machine.
**Pre-requisites:** Azure Az PowerShell module logged in.

**Steps:**
1. Open PowerShell. Set the vault context:
   ```powershell
   $vault = Get-AzRecoveryServicesVault -ResourceGroupName "RG-Lab" -Name "Vault-Lab"
   Set-AzRecoveryServicesAsVaultContext -Vault $vault
   ```
2. Retrieve the protected VM item details:
   ```powershell
   $container = Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" -FriendlyName "VM-LabServer"
   $item = Get-AzRecoveryServicesBackupItem -Container $container -WorkloadType "AzureVM"
   ```
3. Trigger a manual backup with a retention policy of 14 days:
   ```powershell
   $backupJob = Backup-AzRecoveryServicesBackupItem -Item $item -ExpiryDateTimeUTC (Get-Date).AddDays(14)
   ```
4. Query the status of the triggered job:
   ```powershell
   Get-AzRecoveryServicesBackupJob -Job $backupJob
   ```
5. Verification: Monitor the status until it changes from `InProgress` to `Completed`.

**Success Criteria:** The manual backup job is initiated, and the job status returns `Completed` successfully.
**Common Failures:** The backup fails if the Azure VM Guest Agent inside the VM is stopped, blocking hypervisor snapshot communications.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a Recovery Services Vault in Azure?**
A: A Recovery Services Vault is an online storage container in Azure used to hold backup data, virtual machine recovery points, and backup policies, helping protect servers from data loss.

**Q: If a user deletes a file on an Azure VM, do you have to restore the entire virtual machine?**
A: No. I use the **File Recovery** feature in the vault. This generates a script that mounts the backup snapshot as a local drive (e.g., G: drive) inside the running VM, allowing me to copy the single file back without any downtime.

### Intermediate (L2 Level)
**Q: What is Soft Delete in Azure Backup and how long does it retain data?**
A: Soft Delete is a security feature that protects backup data from malicious or accidental deletion. When a backup is deleted, Azure retains the backup data in a soft-deleted state for 14 days at no charge, allowing administrators to restore the backup if needed.

**Q: What is the difference between LRS and GRS storage redundancy for a Recovery Services Vault?**
A: LRS (Locally Redundant Storage) replicates your backup data three times within a single physical data center, protecting against local drive failures. GRS (Geo-Redundant Storage) replicates your data to a secondary paired region hundreds of miles away, protecting your data against major regional disasters or data center outages.

### Advanced (L3/Senior Level)
**Q: You are planning backup protection for a series of virtual machines running SQL Server. The VM disks are encrypted using Azure Disk Encryption (BitLocker). How do you configure backups?**
A:
- **Situation**: Backing up encrypted virtual machines containing SQL workloads.
- **Task**: Secure the data in transit and at rest while preserving encryption keys.
- **Action**: First, I create the Recovery Services Vault. Before enabling backup, I grant the backup service permissions to access the Azure Key Vault hosting the BitLocker keys (Key Vault Access Policy -> Key & Secret Management). When configuring the backup policy, I select the SQL application-consistent snapshot backup type. I verify that during restore, Azure can retrieve both the VM disk snapshot and the encryption keys to decrypt the volume.
- **Result**: The encrypted VMs are backed up successfully, and key integration allows restoration without manual decryption.

### HR / Behavioral
**Q: Tell me about a time you had to perform a recovery under intense pressure, such as a major server failure during business hours.**
A: Our primary application server VM was corrupted due to a bad update patch right before a peak sales hour. The web application went down. I logged into our Recovery Services Vault and reviewed the recovery points. I decided against a file restore, as the OS was corrupted. I chose the latest snapshot, ran the "Create New VM" restore wizard, and pointed it to our production virtual network. In 20 minutes, the VM was rebuilt and running. I updated the public DNS settings, and the website was back online. I then wrote a post-mortem report and placed a Resource Lock on the VM to prevent similar issues.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The backup-as-a-service (BaaS) system protecting Azure VMs and local servers.
> **Why**: Recovers data from accidental deletions, hardware crashes, and ransomware attacks.
> **How**: Configure Recovery Services Vaults, apply backup policies, mount snapshots for file recovery, and monitor jobs.
> **Command**: `Backup-AzRecoveryServicesBackupItem` / `Set-AzRecoveryServicesAsVaultContext`
> **Interview Answer Starter**: "To implement cloud recovery baselines, I deploy Recovery Services Vaults with Geo-Redundant storage, enforcing Soft Delete and using File Recovery for item restores..."

**Key Numbers to Remember:**
- Soft Delete retention window: 14 days
- Max instant restore snapshot retention: 5 days
- Redundancy choices: LRS (Local) / GRS (Geo)
- iSCSI Mount port requirement: TCP 3260

**3 Things Interviewer Wants to Hear:**
- Soft Delete blocks ransomware deletion attempts for 14 days
- Storage redundancy must be set before backing up items
- File Recovery mounts backups as local drives via iSCSI loopback

---

## Related Notes
- [[04-Cloud-and-Security/08-Azure/Azure-VMs|Azure VMs]] — Virtual machines protected by the recovery vaults.
- [[04-Cloud-and-Security/09-Security/BitLocker-Deep-Dive|BitLocker Deep Dive]] — Detail disk encryption key integrations.
- [[01-Foundations/01-Hardware/Storage-Deep-Dive|Storage Deep Dive]] — Outlines the core raid and redundancy architectures.

---

## Tags
#desktop-support #azure #backup #disaster-recovery #L2 #interview-topic #lab-complete #daily-use

