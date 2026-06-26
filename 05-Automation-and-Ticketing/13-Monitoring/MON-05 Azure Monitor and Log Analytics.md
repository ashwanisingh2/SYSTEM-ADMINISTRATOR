---
tags: [azure, monitoring, log-analytics, kql, az-104, cloud]
aliases: [mon-05, azure-monitor, log-analytics]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE / CLOUD**

`#complete` `#intermediate` `#az-104`

# MON-05: Azure Monitor and Log Analytics

> [!abstract] Overview
> Azure Monitor is Microsoft's cloud-native unified monitoring platform for collecting, analyzing, and acting on telemetry from Azure and on-premises resources. It provides end-to-end visibility across your entire infrastructure — from VM performance metrics to application logs, security sign-in events, and resource health. With KQL (Kusto Query Language) powered Log Analytics at its core, Azure Monitor enables proactive alerting, deep troubleshooting, compliance auditing, and automated remediation — making it the single pane of glass every cloud admin needs.

---
## 🧠 Concept Overview

- **What it is** — Azure Monitor is a unified monitoring solution for Azure resources including VMs, App Services, networking, storage, identity (Entra ID), and even on-premises servers via Azure Arc. It collects two fundamental data types: **Metrics** (numeric time-series data) and **Logs** (structured/unstructured event data stored in Log Analytics).
- **Why it matters** — Cloud resources fail silently without monitoring. Unlike on-premises servers where you can physically check blinking lights, a cloud VM can silently run out of disk or get brute-forced. Proactive alerting prevents outages and SLA breaches.
- **Where you see this** — Monitoring Azure VMs for health, tracking failed logins in Entra ID, alerting on VM CPU spikes, compliance auditing, and investigating "why is the app slow?" tickets.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Views Azure Monitor dashboards, checks VM health status, reads and acknowledges alert notifications |
| **L2** | Creates Log Analytics Workspace, writes KQL queries, configures alert rules and action groups, sets up diagnostic settings |
| **L3** | Designs enterprise monitoring architecture, creates custom workbooks, integrates Azure Monitor with Sentinel SIEM, builds automated remediation via Logic Apps |

> [!tip] Seedha Simple Mein
> *Azure Monitor ek building ka CCTV + alarm system hai cloud ke liye. Jaise CCTV cameras (metrics) har kamre ki recording karte hain, alarm (alerts) khatre pe bajta hai, aur control room (Log Analytics) mein sab kuch centrally monitor hota hai. Agar kuch bhi gadbad hoti hai, toh guard (Action Group) ko turant pata chal jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Hospital Patient Monitoring System** is like **Azure Monitor** because...
>
> - **Metrics** = Vital signs displayed on the bedside monitor (heart rate = CPU, blood pressure = memory).
> - **Logs (Log Analytics)** = Detailed patient charts and test reports.
> - **Alerts** = Nurse call button that rings when any vital drops below threshold.
> - **Action Groups** = Response team — nurse gets paged, doctor gets SMS.
> - **VM Insights** = Full body scan dashboard showing everything at once.

---
## 🔬 Technical Deep Dive

### 1. Azure Monitor Architecture

> [!info] Key Concept
> Azure Monitor collects telemetry from multiple layers into a central repository.

**Data Source Layers:**

| 📡 Layer | 🛠️ What It Collects | 📝 Example |
|-------|-----------------|---------|
| **Application** | App performance, exceptions, requests | Application Insights traces |
| **Operating System** | Guest OS metrics, event logs, syslog | CPU %, Windows Event Logs |
| **Azure Resource** | Platform metrics, resource logs | VM metrics, NSG flow logs |
| **Subscription** | Activity logs, service health | Who deleted a VM, outage alerts |
| **Tenant (Entra ID)** | Sign-in logs, audit logs | Failed logins, MFA events |

**Azure Monitor Agent (AMA) vs Legacy:**

> [!danger] Common Mistake
> Using the old legacy MMA agent. ==Microsoft officially deprecated MMA/OMS in Aug 2024. Always use AMA + DCR for new deployments!==

### 2. Log Analytics Workspace

> [!info] Key Concept
> A centralized log repository in Azure. Every log, event, and perf counter flows here. You query it using KQL.

* **Pricing:** Pay-As-You-Go (< 100 GB/day) or Commitment Tier (100+ GB/day).
* **Retention:** 30 days free, up to 730 days paid.

### 3. KQL (Kusto Query Language)

> [!tip] Pro Tip
> *KQL PowerShell piping jaisa hi hai, data filter aur format karne ke liye `|` use karte hain.*

**Top Admin Queries:**

```kql
# VM Heartbeat Check
Heartbeat
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| where LastHeartbeat < ago(5m)

# Failed Sign-Ins
SigninLogs
| where ResultType != 0
| project TimeGenerated, UserPrincipalName, IPAddress, ResultDescription
| order by TimeGenerated desc
```
==**Exam Tip:** KQL syntax is heavily tested in AZ-104. Know `where`, `summarize`, and `project`.==

### 4. Alerts and Action Groups

> [!info] Key Concept
> Alert rules evaluate conditions. Action groups define what happens when an alert fires (email, SMS, runbook).

**Alert Severity:**
* Sev 0 (Critical) to Sev 4 (Verbose).

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure subscription with Contributor access
> - Running Azure VM
> - Azure CLI installed

### Step 1: Create Log Analytics Workspace

```bash
# Workspace create karo
az monitor log-analytics workspace create \
  --resource-group MyRG \
  --workspace-name MyLogWorkspace \
  --location eastus \
  --sku PerGB2018
```

> [!success] Expected Output
> JSON output with `"provisioningState": "Succeeded"`

### Step 2: Create an Alert Rule

```bash
# CPU alert create karo
az monitor metrics alert create \
  --name "High-CPU-Alert" \
  --resource-group MyRG \
  --scopes /subscriptions/{sub-id}/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM \
  --condition "avg Percentage CPU > 85" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action /subscriptions/{sub-id}/resourceGroups/MyRG/providers/Microsoft.Insights/actionGroups/MyActionGroup \
  --severity 2
```

> [!success] Expected Output
> JSON confirming alert rule creation. Alert will fire if CPU > 85% for 5 mins.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|---------|-------------|---------|
| `az monitor log-analytics workspace create` | Create Workspace | `az monitor log-analytics workspace create -g MyRG -n MyLAW` |
| `az monitor diagnostic-settings create` | Enable diagnostics | `az monitor diagnostic-settings create --name "VM-Diag"...` |
| `az monitor metrics alert create` | Create metric alert rule | `az monitor metrics alert create --name "High-CPU"...` |
| `az monitor action-group create` | Create action group | `az monitor action-group create -g MyRG -n "AdminAlert"...` |
| `Invoke-AzOperationalInsightsQuery` | Run KQL via PowerShell | `Invoke-AzOperationalInsightsQuery -WorkspaceId $wsId -Query "Heartbeat \| take 5"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|---------|-------------|-----|
| No data in Log Analytics | AMA not installed on VM | Install AMA: VM → Extensions → Add Azure Monitor Agent |
| KQL query returns empty | Wrong table name or time range | Expand time to 24h, check exact table names (case-sensitive) |
| Alert not firing | Action Group not linked | Edit alert rule → Actions → Verify Action Group is selected |
| VM Insights shows "No data" | DCR not configured properly | Check Azure Monitor → Data Collection Rules → Verify VM association |
| High Log Analytics cost | Excessive data ingestion | Review data volume: Usage table in KQL, set daily cap |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Azure VM Not Responding

> [!example] Ticket
> "Production web server VM (PROD-WEB-01) stopped responding since 2:00 AM. Website is down, users are complaining."

**L1 Response:** Check VM status in Portal. Review Azure Service Health. Check recent Sev 0/1 alerts.
**Escalation Trigger:** If VM shows "Running" but is unresponsive and no heartbeat for 5+ mins.
**L2 Resolution:**
- Query `Heartbeat` table.
- Check VM Insights Performance tab for CPU/Memory spikes before crash.
- Review Boot Diagnostics.
- Restart VM via CLI: `az vm restart -g ProdRG -n PROD-WEB-01`.

### 🎫 Scenario 2: Multiple Users Can't Login

> [!example] Ticket
> "50+ users reporting 'Access Denied' error when accessing company apps since 9:00 AM today."

**L1 Response:** Check Service Health for M365/Entra ID outages. Collect exact error codes from users.
**Escalation Trigger:** If no global outage is reported and it affects specific users.
**L2 Resolution:**
- Query `SigninLogs` for failed authentications `ResultType != 0`.
- Check Conditional Access Policies for recent changes via `AuditLogs`.
- Verify MFA status. Adjust policies if legitimate access is blocked.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Azure Monitor Metrics and Azure Monitor Logs?
> **Answer:** **Metrics** are lightweight, numeric, time-series data points (like CPU%). They are near real-time (~1 min) and retained for 93 days. **Logs** are rich event records stored in a Log Analytics Workspace, queried using KQL, with a 5-15 min ingestion delay and retention up to 730 days.
> *Simple mein: Metrics = thermometer reading (quick number), Logs = full medical report (detailed info).*

> [!question] Q2: Write a KQL query to find all VMs with CPU usage above 90% in the last 1 hour.
> **Answer:**
> ```kql
> Perf
> | where TimeGenerated > ago(1h)
> | where CounterName == "% Processor Time" and InstanceName == "_Total"
> | summarize AvgCPU = avg(CounterValue) by Computer
> | where AvgCPU > 90
> | order by AvgCPU desc
> ```

> [!question] Q3: What is an Action Group?
> **Answer:** An Action Group defines what happens when an alert fires. It contains notification preferences (Email, SMS) and automated actions (Logic App, Webhook, Runbook). You can reuse one Action Group across multiple alert rules.

> [!question] Q4: How do you enable end-to-end monitoring on a new VM?
> **Answer:** Create Log Analytics Workspace → Enable Diagnostic Settings → Install AMA → Create DCR → Enable VM Insights → Create Alert Rules → Create Action Group.

==**Exam Tip:** Memorize the flow: Workspace -> DCR -> AMA -> Alerts -> Action Group. Extremely common in AZ-104!==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ104-06 Azure Monitor and Backup|AZ104-06 Azure Monitor and Backup]] — Core Azure backup strategies
- [[04-Cloud-and-Security/09-Security/Microsoft-Sentinel-SIEM|Microsoft Sentinel SIEM]] — Built on top of Log Analytics
- [[05-Automation-and-Ticketing/13-Monitoring/MON-04 Windows Performance Monitor|MON-04 Windows Performance Monitor]] — On-prem monitoring equivalent
