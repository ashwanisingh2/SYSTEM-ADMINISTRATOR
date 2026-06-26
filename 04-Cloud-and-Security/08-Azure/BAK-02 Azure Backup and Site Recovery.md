---
tags: [desktop-support, azure, cloud, L2]
aliases: [bak-02-azure-backup-and-site-recovery, bak-02]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# BAK-02 Azure Backup and Site Recovery

> [!abstract] Overview
> Azure Backup and Azure Site Recovery (ASR) form the pillars of Microsoft's Business Continuity and Disaster Recovery (BCDR) strategy. Azure Backup is a secure service to back up on-premises and cloud workloads. Azure Site Recovery is a orchestration service that replicates virtual machines to ensure applications remain online during regional outages.

---

---
## Concept Overview
- **What it is** — Azure Backup is a cloud-native backup solution that retains historical data points. Azure Site Recovery replicates workloads in real-time, allowing immediate failover to a recovery site during disasters.
- **Why it matters for a support engineer** — A modern sysadmin must protect company infrastructure from localized hardware failures, ransomware, and natural disasters. Knowing how to restore servers and initiate failover systems is critical.
- **Where you encounter this in real job** — Configuring a daily backup policy for Azure Virtual Machines, installing the MARS agent to back up an on-premises file server to Azure, or executing a disaster recovery test drill.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Monitors backup logs in the Recovery Services Vault, verifies replication health in the portal, and restores files using the Azure Recovery instant mount.
  - **Escalation Trigger:** Escalate to L2 if backup jobs fail consistently, replication synchronization is stuck, or if a VM failover needs to be executed.
  - ****L2 Resolution:**** Deploys Recovery Services Vaults, configures backup policies, installs and registers the MARS agent on-premises, and performs full VM restores.
  - ****L3 Resolution:**** Designs ASR replication patterns, configures Recovery Plans with custom failover scripts, manages network mapping between sites, and sets RTO/RPO metrics.


---

---
## Technical Deep Dive
### 1. BCDR Metrics: RTO and RPO
* **Recovery Time Objective (RTO)**: The maximum acceptable duration of downtime before service is restored (How fast do we need to recover?).
* **Recovery Point Objective (RPO)**: The maximum acceptable data loss measured in time (How much data can we afford to lose? e.g., 4 hours of data).

### 2. Azure Backup Methods
* **Azure VM Backup**: Backs up entire Azure VMs using snapshots. No agent needed inside the VM.
* **MARS (Microsoft Azure Recovery Services) Agent**: Installed on-premises servers. Backs up files, folders, and system state directly to Azure Recovery Services Vault.
* **MABS (Microsoft Azure Backup Server)**: On-premises backup server that caches backups locally before sending them to Azure, supporting SQL, Exchange, and SharePoint.

### 3. Recovery Services Vault (RSV)
The RSV is the online storage entity in Azure. It stores backup data and holds recovery points. Storage redundancy options:
* **LRS (Locally Redundant Storage)**: Three copies in a single datacenter.
* **GRS (Geo-Redundant Storage)**: Replicated to a secondary paired region (Default and recommended for safety).
* **ZRS (Zone-Redundant Storage)**: Replicated across Availability Zones.

---

---
## Step-by-Step Lab
> [!warning] Pre-requisites
> - An active Azure Subscription.
> - An active Azure VM (`VM-Production`) running in `East US`.
> - A Resource Group named `RG-BCDR`.

### Step 1: Create a Recovery Services Vault
We will initialize the RSV that will house our backups and replication sets.

1. Install Az PowerShell module (if not installed):
```powershell
Install-Module -Name Az -AllowClobber -Force
```
2. Connect to Azure Account:
```powershell
Connect-AzAccount
```
3. Create the Recovery Services Vault in the Resource Group:
```powershell
New-AzRecoveryServicesVault -Name "RSV-ProdBackup" -ResourceGroupName "RG-BCDR" -Location "East US"
```
4. Set storage redundancy to Geo-Redundant (GRS):
```powershell
$vault = Get-AzRecoveryServicesVault -Name "RSV-ProdBackup" -ResourceGroupName "RG-BCDR"
Set-AzRecoveryServicesBackupProperty -Vault $vault -BackupStorageRedundancy GeoRedundant
```
**Expected Output:** The vault is created with properties showing `BackupStorageRedundancy : GeoRedundant`.

---

### Step 2: Configure Azure VM Backup Policy and Enable Backup
We will create a policy to back up our VM daily at 2:00 AM and retain it for 30 days.

1. Set vault context:
```powershell
Get-AzRecoveryServicesVault -Name "RSV-ProdBackup" | Set-AzRecoveryServicesVaultContext
```
2. Enable backup protection for `VM-Production`:
```powershell
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"
Enable-AzRecoveryServicesBackupProtection -ResourceGroupName "RG-BCDR" -Name "VM-Production" -Policy $policy
```
**Expected Output:** Backup protection enabled status: `Passed`.

---

### Step 3: Configure Azure Site Recovery (ASR) Replication
We will configure replication of our East US VM to a secondary region (West US) for disaster recovery.

1. Navigate to the Azure Portal -> **Recovery Services Vaults** -> **RSV-ProdBackup**.
2. Click **Site Recovery** under Getting Started -> Click **Enable Replication**.
3. Select Source: **Azure Virtual Machines** -> Source Location: **East US**.
4. Select Target Region: **West US** -> Resource Group: `RG-BCDR-Replica`.
5. Select Replication Policy -> Click **Enable Replication**.
**Expected Output:** Azure begins replication. Under "Replicated Items", the VM status changes to `Healthy` once the initial replication sync completes.

---

### Step 4: Perform a Test Failover (Disaster Recovery Drill)
We will verify that our target VM starts up in the West US region without affecting the production VM in East US.

1. Under **Replicated Items**, click on the VM.
2. Select **Test Failover** from the top menu bar.
3. Select the Recovery Point (e.g., *Latest Application-Consistent*).
4. Choose the target Azure Virtual Network (must be isolated from production).
5. Click **OK**.
**Expected Output:** Azure creates a test VM in West US. Under "Jobs", status shows test failover completed successfully. After verifying, run **Cleanup Test Failover** to delete the test resources.

---

---
## Cheat Sheet / Quick Reference
| Command | Description | Example |
|---------|-------------|---------|
| `New-AzRecoveryServicesVault` | Create an RSV vault | `New-AzRecoveryServicesVault -Name "VaultName" -ResourceGroupName "RG" -Location "EastUS"` |
| `Get-AzRecoveryServicesBackupJob` | List backup jobs in vault | `Get-AzRecoveryServicesBackupJob -Status Failed` |
| `Backup-AzRecoveryServicesBackupItem` | Trigger an ad-hoc VM backup | `Backup-AzRecoveryServicesBackupItem -Item $backupItem` |
| `Restore-AzRecoveryServicesBackupItem` | Restore VM disks from restore points | `Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp -StorageAccountName "storage"` |

---

---
## Troubleshooting
| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **VM Backup fails with Guest Agent status Unavailable** | The Azure VM Agent is not installed, outdated, or service stopped. | Install/Update Azure VM Agent inside VM OS. Ensure `Windows Azure Guest Agent` service is running. |
| **ASR replication state remains Critical** | Network connectivity issues or storage write churn is too high. | Verify network bandwidth. Ensure source VM disk write rates do not exceed limits (e.g., 10MB/s per disk for standard SSDs). |
| **MARS backup job fails with network timeout** | Firewall or Proxy server blocking Azure backup endpoints. | Allow outbound connections to Azure backup URLs (e.g., `*.backup.windowsazure.com`) over port 443. |

---

---
## Interview Questions
**Q1: What is the main structural difference between Azure Backup and Azure Site Recovery?**
> A: **Azure Backup** captures and stores backups of data over long periods, offering point-in-time recovery points (typically used for data corruption, file deletion, and compliance audits). **Azure Site Recovery (ASR)** replicates virtual machines in real-time to a secondary region, maintaining an near-live state to allow rapid failover and restore application availability during a disaster.

**Q2: What is the difference between RTO and RPO?**
> A: **RTO (Recovery Time Objective)** is the maximum target time allowed to restore service after an outage (how long the system can be offline). **RPO (Recovery Point Objective)** is the maximum target age of data that can be lost (how much data change can be lost, e.g., if backups run every 4 hours, RPO is 4 hours).

**Q3: How does the Azure VM backup run without affecting virtual machine operations?**
> A: It uses the Azure VM extension to interact with host-level VSS (Windows) or fsfreeze (Linux) to quiesce the VM filesystem. It takes a storage-level snapshot of the VM's managed disks. Once the snapshot is completed, the VM resumes full write operations while the data is copied to the Recovery Services Vault in the background.

**Q4: What is a Test Failover in Azure Site Recovery and why is it important?**
> A: A Test Failover is a disaster recovery drill. It launches copies of replicated VMs in an isolated virtual network in the recovery region. This allows you to verify that the virtual machines start up and applications run correctly without interrupting the live production VMs or the ongoing data replication.

---

---
## Seedha Simple Mein
*Seedha simple mein: Azure Backup server data ke historical backups leta hai taaki files delete hone par recovery ho sake. Azure Site Recovery (ASR) server ki copy run-time me doosre Azure region me save rakhta hai, taaki agar main datacenter band ho jaye toh backup server turant start ho sake.*

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ104-06 Azure Monitor and Backup]]
- [[04-Cloud-and-Security/08-Azure/Azure-Backup]]
- [[03-Identity-and-Core-Services/05-Windows-Server/BAK-01 Windows Server Backup]]
