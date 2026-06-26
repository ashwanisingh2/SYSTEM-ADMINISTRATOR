---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-06-azure-monitor-and-backup, az104-06]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#advanced` `#az-104`

# AZ104-06: Azure Monitor and Backup

> [!abstract] Overview
> Yeh note Azure monitoring, logging, aur disaster recovery ke baare mein hai. Isme Azure Monitor Metrics, Log Analytics, KQL (Kusto Query Language), Alerts, Service Health, aur Recovery Services Vault (VM Backup/Restore) cover kiye gaye hain. Ek system administrator ke liye issues detect aur fix karne mein yeh foundation hai.

---
## 🧠 Concept Overview

- **What it is** — Azure platform ke health aur performance ko track karna aur data loss bachane ke liye regular backups configure karna.
- **Why it matters** — System crash ya ransomware attack ke baad business continuity maintain karne ke liye.
- **Where you see this** — Jab CPU 90% cross kare toh alert bhejna ho, ya galti se delete hui file ko backup se recover karna ho.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Alerts receive karna, basic metric graphs dekhna, aur backup job status check karna. |
| **L2** | KQL queries run karna, Alert rules aur Action Groups banana, aur VM/file level restore perform karna. |
| **L3** | Enterprise level monitoring architecture aur cross-region disaster recovery (ASR) setup karna. |

> [!tip] Seedha Simple Mein
> *Azure Monitor aapki car ke dashboard jaisa hai jo speed (metrics) aur error logs dikhata hai. Recovery Vault aapki time-machine hai jo crash hone par purani state wapas laati hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Monitor and Backup** is like the systems on a **Commercial Spaceship** because...
>
> - **Metrics** are the real-time gauges (speed, engine heat) updating constantly.
> - **Logs** are the detailed black-box recorders capturing every event.
> - **KQL** is the search computer finding the exact moment of failure in millions of log lines.
> - **Recovery Services Vault** is the escape pod / replication system that restores the ship to its exact last-saved coordinates if hit by a meteor.

---
## 🔬 Technical Deep Dive

### 1. Azure Monitor: Metrics vs. Logs

> [!info] Key Concept
> Metrics are numerical time-series data for real-time alerting. Logs are detailed event records stored in a Log Analytics Workspace for complex analysis.

### 2. Log Analytics Workspace & KQL Basics

- **Structure:** `TableName | Filter Condition | Grouping/Aggregation`
- **Example KQL for failed logins:**
```kusto
SecurityEvent
| where EventID == 4625
| summarize count() by TargetAccount, Computer
```

### 3. Metric Alerts and Action Groups

- **Metric Alerts:** Static (fixed threshold, e.g., >85%) vs Dynamic (machine learning baseline).
- **Action Groups:** Defines *who* is notified (Email/SMS/Webhook) and *what* action is taken (Runbook/Function).

### 4. Service Health vs. Resource Health

- **Service Health:** Global Azure outages affecting multiple customers (e.g., East US VM provisioning down).
- **Resource Health:** Issues specific to your individual resources (e.g., your VM hardware failed).

### 5. Recovery Services Vault (VM Backup & Restore)

- **Azure VM Backup:** Agentless snapshot backups.
- **Restore Options:**
  - Create New VM.
  - Restore Disks (attach `.vhdx` to existing VM).
  - File Recovery (mount snapshot as a local drive to copy files).

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A deployed VM (`vm-web-dev`)
> - Recovery Services Vault (`rsv-lab-backup`)

### Step 1: Create a Metric Alert

1. Portal > VM `vm-web-dev` > **Alerts** > **+ Create**.
2. Signal: **Percentage CPU**, Threshold: Static > 85.
3. Actions: Create Action Group `ag-sysadmin-alerts` with Email notification.
4. Review + Create.

### Step 2: Trigger a Manual Backup

1. Go to Vault `rsv-lab-backup` > **Backup items** > Azure Virtual Machine > `vm-web-dev`.
2. Click **Backup now**, set retention to 30 days.

### Step 3: Perform File Recovery

1. Vault > `vm-web-dev` > **File Recovery**.
2. Select recovery point > **Download Script**.
3. Run script locally, use provided password.
4. The backup disk mounts as a local drive. Copy files out, then click **Unmount Disks** in portal.

> [!success] Expected Output
> The exact deleted file from yesterday is restored to your local machine via the mounted virtual disk.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az monitor metrics list` | List metrics for a resource | `az monitor metrics list --resource <VM_ID>` |
| `az backup protection backup-now` | Trigger on-demand backup | `az backup protection backup-now --item-name vm1` |
| `Get-AzRecoveryServicesBackupItem` | PowerShell get backup status | `Get-AzRecoveryServicesBackupItem -VaultId $vault` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Backup job failed with "ExtensionNotResponsive" | VM agent is hung or offline | Restart the Azure Guest Agent service inside the VM. |
| Metric Alert not firing | Threshold not met for required duration | Check evaluation granularity (e.g., CPU must be >90% for a full 5 minutes). |
| Cannot mount file recovery script | iSCSI initiator service stopped | Start the `MSiSCSI` service on the local Windows recovery machine. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Accidental File Deletion

> [!example] Ticket
> "I accidentally deleted the critical config file from the web server. Can we recover it from yesterday?"

**L1 Response:** Confirm the server name and file path.
**Escalation Trigger:** None, L1/L2 should handle File Recovery.
**L2 Resolution:** Open the VM's backup vault, select File Recovery, download the script, mount the drive on a jump server, copy the file, and place it back on the target server. Click unmount.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Service Health and Resource Health in Azure?
> **Answer:** Service Health reports on broad Microsoft Azure datacenter outages or maintenance. Resource Health reports on issues affecting only your specific provisioned resources.

> [!question] Q2: How can you recover a single deleted file without restoring the entire VM?
> **Answer:** By using the "File Recovery" option in the Recovery Services Vault, which downloads an iSCSI script to mount the recovery point as a local drive letter.

==**Exam Tip:** KQL summarize operator uses `bin(TimeGenerated, 1h)` to group log data into 1-hour chunks.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Base backup policy mappings.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Diagnostic logs for NSG flows.
