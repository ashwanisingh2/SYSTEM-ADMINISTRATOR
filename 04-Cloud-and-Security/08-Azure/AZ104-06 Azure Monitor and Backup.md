---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-06-azure-monitor-and-backup, az104-06]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

# AZ104-06: Azure Monitor and Backup

> [!abstract] Overview
> This note covers Azure monitoring, logging, and disaster recovery. It details metrics, Log Analytics workspaces, Kusto Query Language (KQL) queries, Action Group notifications, Service/Resource Health checks, and Recovery Services Vault restores.

---

---
## Concept Overview
Think of monitoring and backups in Azure like the health monitoring and backup black box on a commercial spaceship:
- **Metrics** are the real-time gauges on the dashboard (speed, temperature, engine load). They update every few seconds so you can react immediately if something goes red (Metric Alerts).
- **Logs** are the detailed black-box logbooks. Every action, error message, and warning is recorded in a massive database ledger (**Log Analytics Workspace**). 
- **KQL (Kusto Query Language)** is your search computer: you write queries to scan millions of log lines and find the exact second a specific system crashed.
- **Recovery Services Vault** is the escape pod and emergency backup bank. If the spaceship gets hit by space debris (ransomware or hardware failure), you use the vault to restore a complete, healthy copy of the ship to its exact last-saved coordinates.


---

---
## Technical Deep Dive
### 1. Azure Monitor: Metrics vs. Logs
- **Metrics:** Numerical values representing the state of a resource at a point in time (e.g., CPU percentage, network out bytes). Collected at regular intervals, stored in a time-series database. Ideal for real-time alerting.
- **Logs:** Contain different kinds of data organized into records with different sets of properties (e.g., system event logs, application logs). Stored in a Log Analytics Workspace. Ideal for complex historical analysis.

### 2. Log Analytics Workspace & KQL Basics
A Log Analytics Workspace is a logical storage container for log data. To query this data, use **KQL (Kusto Query Language)**.
- **KQL Syntax Structure:**
  `TableName | Filter Condition | Grouping/Aggregation | Projection`

#### Useful KQL Queries for Sysadmins
```kusto
// Query 1: Find all virtual machines with CPU usage greater than 90%
Perf
| where CounterName == "% Processor Time" and CounterValue > 90
| summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)

// Query 2: Find all failed login attempts in the security logs
SecurityEvent
| where EventID == 4625
| summarize count() by TargetAccount, Computer, bin(TimeGenerated, 1d)
| order by count_ desc

// Query 3: Find heartbeat failures (virtual machines that stopped communicating)
Heartbeat
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| where LastHeartbeat < ago(15m)
```

### 3. Metric Alerts and Action Groups
- **Metric Alerts:** Triggers notifications when resource metrics cross thresholds.
  - *Static Threshold:* Compares metric against a set value (e.g., CPU $> 90\%$).
  - *Dynamic Threshold:* Uses machine learning to calculate a historical baseline, triggering alerts if behavior deviates from normal patterns.
- **Action Groups:** Define *who* gets notified and *what* automated action occurs. Actions include: Email, SMS, Voice, Push notifications, Azure Function triggers, Webhooks (to trigger ServiceNow/Slack), or Automation Runbooks (to auto-restart a VM).

### 4. Service Health vs. Resource Health
- **Service Health:** Alerts you to outages, planned maintenance, and advisories affecting **Azure services globally** within your region (e.g., "Virtual Machines are experiencing provisioning issues in East US").
- **Resource Health:** Focuses on the status of your **individual resources** (e.g., "Your VM 'vm-web-dev' is unavailable due to an underlying host hardware failure").

### 5. Recovery Services Vault (VM Backup & Restore)
- **Backup Methods:**
  - **Azure VM Backup:** Agentless, application-consistent backups using VSS (Windows) or fsfreeze (Linux) snapshots.
  - **MARS Agent (Microsoft Azure Recovery Services):** Installed inside Windows clients/servers. Backs up files, folders, and system state directly to the vault.
- **Restore Options:**
  - *Create New VM:* Restores the disk and builds a new VM configuration.
  - *Restore Disks:* Restores the `.vhdx` disk files to a storage account, allowing you to attach them to existing VMs.
  - *File Recovery:* Mounts the backup snapshot disk directly as a local drive letter on a recovery server, allowing you to browse and copy individual files.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Access to the Azure Portal, a deployed VM `vm-web-dev`, and a Recovery Services Vault `rsv-lab-backup`.

### Step 1: Create a Metric Alert
1. Log into the Azure Portal. Go to the virtual machine page for `vm-web-dev`.
2. Under Monitoring in the left panel, click **Alerts** -> **+ Create** -> **Alert rule**.
3. Condition: Select signal **Percentage CPU**.
4. Alert Logic:
   - Threshold: **Static** | Operator: **Greater than**
   - Threshold value: **85**
   - Aggregation granularity: **5 minutes** | Frequency of evaluation: **1 minute**.
5. Click Next: **Actions**. Click **Create action group**.
   - Action group name: `ag-sysadmin-alerts`.
   - Notifications: Select **Email/SMS message/Push/Voice**. Check Email and enter `sysadmin@company.com`.
   - Click Create.
6. Click Next: **Details**. Alert rule name: `Alert_High_CPU_vm-web-dev`. Severity: **Sev 3 (Informational)**.
7. Click **Review + create**, then click **Create**.

### Step 2: Trigger a Manual Backup
1. Go to your Recovery Services Vault `rsv-lab-backup`.
2. Click **Backup items** -> **Azure Virtual Machine**.
3. Click `vm-web-dev` -> click **Backup now**.
4. Set retention date to 30 days. Click OK.
5. Go to **Backup Jobs** to monitor progress. Once complete, a recovery point is registered.

### Step 3: Perform File Recovery (Restore Test)
1. Go to `rsv-lab-backup` -> **Backup items** -> **Azure Virtual Machine** -> click `vm-web-dev`.
2. Click **File Recovery** at the top menu.
3. Select recovery point. Click **Download Script**.
4. Copy the password displayed in the portal.
5. Run the downloaded script (PowerShell or Bash) on your local recovery machine, entering the password.
6. **Verify:** The script mounts the backup virtual disk locally. Open File Explorer. Verify you can browse the guest OS files and copy `data.txt` out, proving backup integrity.
7. Click **Unmount Disks** in the Azure Portal when finished.

---

---
## Cheat Sheet / Quick Reference
| Command / Configuration | Scope | Purpose / Example |
|---|---|---|
| `systemctl status <service>` | Linux | Check status of system service |
| `ip address show` | Linux | Display local interface network details |
| `Get-Service` | PowerShell | Verify service status on Windows hosts |
| `Test-NetConnection` | PowerShell | Check network path connectivity to target ports |

---
## Troubleshooting
| Problem | Cause | Fix | Command |
|---|---|---|---|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall | `ping -c 4 <ip>` / `nc -zv <ip> <port>` |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password | N/A |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects | N/A |

---
## Interview Questions
> [!question] L1 Question
> **Q:** How do you verify if the target service is running?
> **A:** On Linux, I would execute `systemctl status <service-name>`. On Windows, I would run `Get-Service <service-name>` in PowerShell or check Services.msc.

> [!question] L2 Question
> **Q:** Explain how you would troubleshoot a network connectivity issue to a remote server.
> **A:** I would verify local IP configuration, test routing gateway using `ping`, trace hops using `traceroute` or `tracert`, and check port accessibility using `telnet` or `Test-NetConnection` on target port.

---
## Seedha Simple Mein
*Seedha simple mein: Azure Monitor metrics (real-time telemetry) aur logs (historical data) collect karta hai. Log analysis ke liye hum KQL queries use karte hain. Disaster recovery aur VM backups ko manage karne ke liye Recovery Services Vault use hota hai.*

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Base backup policy mappings.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Diagnostic logs for NSG flows.
