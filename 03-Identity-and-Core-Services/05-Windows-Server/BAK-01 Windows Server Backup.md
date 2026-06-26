---
tags: [windows-server, backup, disaster-recovery, backup-restore]
aliases: [windows-server-backup]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# BAK-01 Windows Server Backup

> [!abstract] Overview
> Windows Server Backup (WSB) is a built-in feature in Windows Server 2022 that provides backup and recovery options for server environments. It supports full server backups (Bare Metal Recovery), specific volumes, system state backup (essential for Active Directory Domain Controllers), and file-level recovery.

---

## Concept Overview
- **What it is** — WSB is a local backup engine that utilizes Volume Shadow Copy Service (VSS) to capture block-level images of volumes, supporting full, incremental, and system state backups.
- **Why it matters for a support engineer** — Servers fail due to hardware crashes, ransomware attacks, or user error. Knowing how to quickly schedule and restore from backups is critical to minimizing business downtime.
- **Where you encounter this in real job** — Setting up nightly automated backups of Active Directory Domain Controllers, restoring an accidentally deleted file share, or performing a Bare Metal Recovery (BMR) on replacement hardware.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Monitors backup logs in Event Viewer, checks destination storage availability, and restores single files or folders.
  - **Escalation Trigger:** Escalate to L2 if backups consistently fail with VSS errors, or if a Bare Metal Recovery or System State restore is required.
  - ****L2 Resolution:**** Installs WSB features, configures backup schedules to local disks or network shares, runs `wbadmin` commands, and performs System State restores.
  - ****L3 Resolution:**** Establishes enterprise retention policies, troubleshoots deep VSS writer corruptions, and plans Bare Metal Recovery protocols across diverse hardware.

*Seedha simple mein: Windows Server Backup ek built-in backup utility hai jo pure server ki image ya selected data ka backup leti hai. Iska use karke system crash hone par hum asani se system state ya files ko restore kar sakte hain.*

---

## Technical Deep Dive

### 1. Backup Types in Windows Server
* **Full Server Backup**: Backs up all volumes on the server. Necessary for Bare Metal Recovery (BMR) to rebuild the entire server on clean hardware.
* **System State Backup**: Backs up boot files, Active Directory database (`ntds.dit`), SYSVOL folder, registry, and IIS metabase. Required for Domain Controller recovery.
* **Volume-level Backup**: Backs up selected drives (e.g., `D:\` drive containing file shares).
* **Incremental Backup**: Only backs up data blocks that have changed since the last backup, saving network and storage space.

### 2. Volume Shadow Copy Service (VSS)
WSB relies heavily on VSS. VSS coordinates between:
* **VSS Requester**: The backup software (WSB).
* **VSS Writer**: The applications (like AD, SQL, Exchange) which flush transactions to disk before the backup is captured.
* **VSS Provider**: The OS kernel which creates the snapshot on disk.

### 3. Command Line Backup: `wbadmin`
Administrators use `wbadmin` to automate backups via scripts or PowerShell.

---

## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - A Windows Server 2022 instance.
> - An additional dedicated hard disk (`Disk 1`) initialized but unformatted for backup storage, or a configured network share path.

### Step 1: Install Windows Server Backup Feature
WSB is not installed by default on Windows Server.

1. Open **PowerShell** as Administrator.
2. Run the installation command:
```powershell
Install-WindowsFeature Windows-Server-Backup -IncludeAllSubFeature
```
**Expected Output:** `Success : True`, `Restart Needed : No`.

---

### Step 2: Configure a Scheduled System State Backup using wbadmin
We will schedule a nightly backup of the server's System State to a secondary disk (`E:` drive).

1. List available backup targets:
```powershell
wbadmin get disks
```
2. Schedule a daily System State backup at 11:00 PM:
```powershell
wbadmin enable backup -addtarget:E: -schedule:23:00 -systemstate -quiet
```
**Expected Output:** `The backup schedule has been successfully created.`

---

### Step 3: Run an Ad-hoc Full Server Backup
We will run a one-time full backup of the server immediately, preparing it for Bare Metal Recovery.

1. Initiate the backup to an external drive (e.g., `F:` drive):
```powershell
wbadmin start backup -backupTarget:F: -allCritical -quiet
```
2. Monitor backup progress:
```powershell
wbadmin get status
```
**Expected Output:** Status shows progress percentage until completion message: `The backup operation successfully completed.`

---

### Step 4: Perform a File-Level Recovery
We will restore a single deleted file from our backup set.

1. Identify available versions of backups:
```powershell
wbadmin get versions
```
2. Start the recovery wizard command (specify the backup version date/time):
```powershell
wbadmin start recovery -version:06/25/2026-23:00 -itemType:File -items:C:\Shares\Report.xlsx -recursive -recoveryTarget:C:\RestoredData -quiet
```
**Expected Output:** `The recovery operation successfully completed.`

---

## Common Commands / Cheat Sheet

| Command | Description | Example |
|---------|-------------|---------|
| `wbadmin start backup` | Run a one-time backup | `wbadmin start backup -backupTarget:E: -include:C:` |
| `wbadmin stop job` | Cancel a running backup job | `wbadmin stop job` |
| `wbadmin get versions` | List details of backups stored on target | `wbadmin get versions` |
| `wbadmin get items` | List items included in a specific backup version | `wbadmin get items -version:timestamp` |
| `wbadmin start systemstaterecovery` | Perform a recovery of the system state | `wbadmin start systemstaterecovery -version:timestamp` |

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **Backup fails with "VSS writer failed" error** | An application writer is in a failed state. | Run `vssadmin list writers` to identify the failed writer. Restart the corresponding service (e.g., Volume Shadow Copy service or application service). |
| **Scheduled backup to network share overwrites old backups** | WSB only supports a single backup version history when backing up to network shares. | Use dedicated local disks for multiple versions history, or script separate target folders on network shares. |
| **Bare Metal Recovery (BMR) option grayed out** | Not all boot/system critical partitions were backed up. | Ensure `-allCritical` switch is used in `wbadmin`, which automatically adds System Reserved and C: drives. |

---

## Interview Questions

**Q1: What is System State Backup in Windows Server and what does it contain?**
> A: System State Backup captures critical system components required to restore a server. For a Domain Controller, it contains the Active Directory Database (`ntds.dit`), the SYSVOL folder (Group Policies), the system registry, IIS Metabase, boot files, and COM+ class registration databases.

**Q2: What is Bare Metal Recovery (BMR)?**
> A: BMR is a recovery process that allows you to restore a server from scratch on a new hardware machine with no operating system pre-installed. It restores the OS, configuration, applications, and all data directly from the system critical image backup.

**Q3: How do you verify the health of VSS writers in Windows Server?**
> A: Open PowerShell or Command Prompt as administrator and run `vssadmin list writers`. Check if all listed writers show `State: [1] Stable` and `Last error: No error`.

**Q4: Can you store multiple historical backup versions on a network share using Windows Server Backup?**
> A: No, by default, when Windows Server Backup targets a remote shared folder, only the latest backup is kept. Any subsequent backup overwrites the previous backup image file. To store history, you must back up to a dedicated local physical or virtual disk.

---


---
## Real-World Scenario: File Server Crash (500+ Users Affected)
Agar company ka main file server (500+ users connected) crash ho jaye, toh as a Senior Sysadmin aapka immediate action plan yeh hoga:
1. **Identify & Isolate**: Check if the server is pingable. Verify hardware status via Dell iDRAC / HPE iLO or check virtual machine status in vCenter/Hyper-V.
2. **Access Check & Lock**: If the storage volume is corrupt or unmapped, stop DFS Namespace referrals to the crashed target to redirect users to secondary replica servers if active.
3. **Trigger Disaster Recovery (DR)**: Start restoring the crashed system drive or volumes from the latest VSS snapshot or Azure Backup RSV restore point.
4. **Communication**: Broadcast status updates to the Incident Commander and helpdesk teams to manage incoming support volume.

### PowerShell Automation Snippet: Verify File Shares and Access Permissions
```powershell
# Get all active file shares on the server
Get-SmbShare | Format-Table -AutoSize

# Test local DFS Namespace server connection health
Test-DFSNamespaceTarget -Path "\\corp.local\DFSRoot\Public"
```

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-01 Windows Server 2022 Introduction]]
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-11 Storage — Disk Management and RAID]]
- [[04-Cloud-and-Security/08-Azure/Azure-Backup]]
