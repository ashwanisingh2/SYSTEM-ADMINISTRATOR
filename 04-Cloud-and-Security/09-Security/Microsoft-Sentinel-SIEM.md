---
tags: [security, siem, sentinel, cloud, az-104]
aliases: [microsoft-sentinel-siem]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# Microsoft-Sentinel-SIEM

> [!abstract] Overview
> Microsoft Sentinel is a cloud-native Security Information and Event Management (SIEM) and Security Orchestration, Automation, and Response (SOAR) solution. It aggregates data streams across users, devices, applications, and infrastructure, utilizing machine learning algorithms and Kusto Query Language (KQL) to hunt and respond to threats.

---

## Concept Overview
- **What it is** — Sentinel is Microsoft's cloud platform for security analytics, collecting logs, correlating events, generating incident alerts, and orchestrating automated response workflows.
- **Why it matters for a support engineer** — Desktop support and sysadmins are the first line of defense. Knowing how logs are collected, how alert indicators relate to target endpoints, and how to verify security breaches is essential.
- **Where you encounter this in real job** — Connecting Entra ID logs to Sentinel, analyzing a suspicious user login location using KQL, or responding to automated system isolation alerts.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Monitors the Sentinel Incident queue, triages alerts (checks IP ownership, user roles), and escalates suspicious items.
  - **Escalation Trigger:** Escalate to L2 if a suspicious alert is validated as a potential breach (e.g., multi-factor authentication fatigue bypass attempt).
  - ****L2 Resolution:**** Configures data connectors, builds monitoring Workbooks (dashboards), and writes basic KQL log queries to trace events.
  - ****L3 Resolution:**** Writes custom Analytics detection rules, designs automated SOAR Playbooks (Logic Apps) to disable users, and hunts for advanced persistent threats.

*Seedha simple mein: Microsoft Sentinel ek Cloud-based security tool hai. Yeh pure company ke logs (Windows, Office 365, Firewall) ko scan karta hai. Agar koi hacker system hack karne ki koshish kare, toh yeh automated alert generate karta hai aur automated commands run karke attack ko rok deta hai.*

---

## Technical Deep Dive

### 1. SIEM vs SOAR
* **SIEM (Security Information and Event Management)**: Collects logs, index data, detects patterns, and alerts analysts when rules match (correlating separate logs).
* **SOAR (Security Orchestration, Automation, and Response)**: Takes SIEM alerts and automatically runs scripts or playbooks (e.g., disabling user accounts, blocking IP addresses in firewalls) to respond immediately.

### 2. Kusto Query Language (KQL)
KQL is the read-only query language used to extract information from Log Analytics workspaces.
* **Basic Syntax**:
```kql
TableName
| where TimeGenerated > ago(1d)
| where ResultType == "50126" // Authentication Failure
| summarize count() by UserPrincipalName
```

### 3. Log Source Integration: Data Connectors
Sentinel ingests data using **Data Connectors**. Common sources:
* **Out-of-the-box (Free)**: Azure Activity logs, Microsoft 365 Audit logs, Entra ID logs.
* **Log Forwarding (CEF/Syslog)**: Ingests logs from on-premises firewalls (Cisco, Fortinet) using a dedicated Linux log collector VM.
* **AMA (Azure Monitor Agent)**: Installed on servers to capture Windows Event Logs and Linux syslog.

---

## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - An active Azure Subscription.
> - A Log Analytics Workspace configured.
> - Contributor permissions on the Subscription.

### Step 1: Initialize Microsoft Sentinel
We will onboard Microsoft Sentinel onto our Log Analytics Workspace.

1. Open PowerShell as Administrator.
2. Register the Sentinel security resource provider:
```powershell
Register-AzResourceProvider -ProviderNamespace "Microsoft.SecurityInsights"
```
3. Add Sentinel to your Log Analytics Workspace:
```powershell
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName "RG-BCDR" -Name "LAW-Prod"
New-AzResource -ResourceId "$($workspace.ResourceId)/providers/Microsoft.SecurityInsights/onboardingStates/default" -ApiVersion "2020-01-01" -Force
```
**Expected Output:** Resource is successfully provisioned and Sentinel dashboard becomes available.

---

### Step 2: Connect Microsoft Entra ID Data Source
We will enable log ingestion from Azure Active Directory (Entra ID) into Sentinel.

1. Navigate to Azure Portal -> **Microsoft Sentinel** -> Select Workspace -> **Data Connectors**.
2. Search for **Microsoft Entra ID** -> Click **Open Connector Page**.
3. Under Configuration, check the logs to collect:
   * **Sign-in logs**
   * **Audit logs**
4. Click **Apply Changes**.
**Expected Output:** Connector status displays a green line showing active data flow within 15 minutes.

---

### Step 3: Write KQL Threat Hunting Queries
We will write a query to detect user logins occurring from outside our country.

1. Navigate to **Logs** in Sentinel.
2. Paste the query block into the editor:
```kql
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType == 0 // Successful Sign-ins
| where LocationDetails.countryOrRegion != "IN" // Identify logins outside India
| project TimeGenerated, UserPrincipalName, IPAddress, LocationDetails.city, AppDisplayName
| order by TimeGenerated desc
```
3. Click **Run**.
**Expected Output:** Display grid containing timestamps, users, external IPs, and locations matching the condition.

---

### Step 4: Configure an Analytics Rule (Alert Trigger)
We will create a rule to generate an Incident if a user gets blocked after entering wrong passwords repeatedly.

1. Navigate to **Analytics** -> Click **Create** -> Select **Scheduled query rule**.
2. Name: `Brute Force Blocked User`.
3. Set rule query logic:
```kql
SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType == 50053 // Account is locked out
| summarize FailureCount = count() by UserPrincipalName, IPAddress
| where FailureCount >= 1
```
4. Map entities: Set `UserPrincipalName` to **Account** and `IPAddress` to **IP**.
5. Set scheduling to run every 1 hour.
6. Under **Incident Configuration**, enable "Create incidents from alerts".
7. Save the rule.
**Expected Output:** Rule is registered. When a lockout occurs, a new incident is logged in the Incident dashboard.

---

## Common Commands / KQL Snippets

| KQL Operator | Description | Example |
|--------------|-------------|---------|
| `where` | Filters records based on condition | `| where ResultType == 0` |
| `summarize` | Aggregates and groups data | `| summarize count() by User` |
| `project` | Selects columns to display | `| project UserPrincipalName, IPAddress` |
| `join` | Merges two log tables | `SigninLogs | join SecurityEvent on ...` |

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **Data Connector status shows disconnected** | Diagnostic settings deleted or permissions changed. | Verify target diagnostic settings are sending data to the Log Analytics workspace. Ensure Sentinel has Reader/Contributor roles. |
| **High cloud cost in Sentinel** | Ingesting unnecessary logs (e.g., debug syslogs or IIS logs). | Refine data collector filters. Set daily volume cap thresholds on Log Analytics workspace to control costs. |
| **KQL query fails with syntax error** | Typos in column names or incorrect casing. | KQL is case-sensitive. Verify column name references (e.g., use `IPAddress`, not `ipaddress`). |

---

## Interview Questions

**Q1: What is the primary role of a SIEM, and how does Microsoft Sentinel fit in?**
> A: A SIEM (Security Information and Event Management) aggregates security logs from multiple disparate systems (servers, firewalls, applications), correlates events, and alerts security analysts. Microsoft Sentinel is a cloud-native SIEM that runs on top of Azure, enabling rapid cloud-scale logging and AI-driven analysis without needing server infrastructure.

**Q2: What query language is used in Microsoft Sentinel and what are its key components?**
> A: **KQL (Kusto Query Language)** is used. Its structure is built around tables (e.g., `SigninLogs`, `SecurityEvent`) piped (`|`) into operators like `where` (filtering), `summarize` (aggregation), `project` (selecting columns), and `extend` (creating custom columns).

**Q3: How does Sentinel handle automated responses to threat incidents?**
> A: It uses **SOAR** capabilities implemented via **Playbooks** (which are built on Azure Logic Apps). When an Analytics rule generates an incident, it triggers a Playbook that executes workflows, such as sending a Teams message, blocking an IP in the firewall, or resetting a user's password.

**Q4: Which log table tracks login attempts in Azure Active Directory?**
> A: The `SigninLogs` table tracks user sign-in events, including success codes, failures, devices, app IDs, and geolocation details.

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-05 Security and Compliance]]
- [[04-Cloud-and-Security/09-Security/CIA-Triad-and-Zero-Trust]]
- [[04-Cloud-and-Security/09-Security/Incident-Response-Playbook]]
