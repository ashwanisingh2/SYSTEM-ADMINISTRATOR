---
tags: [azure, monitoring, cloud, sysadmin]
aliases: [mon-05, azure-monitor, log-analytics]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> 📊 **MONITORING**

`#complete` `#intermediate` `#az-104`

# MON-05: Azure Monitor and Log Analytics

> [!abstract] Overview
> Azure Monitor is a comprehensive solution for collecting, analyzing, and acting on telemetry from your cloud and on-premises environments. It helps sysadmins and cloud engineers maximize the availability and performance of applications. At its core, it uses a **Log Analytics Workspace (LAW)** to store logs and metrics, which are queried using Kusto Query Language (KQL). It also handles Alerts, Action Groups, and specialized Insights like VM Insights. *Yeh ek complete solution hai Azure infrastructure ko monitor aur manage karne ke liye.*

---
## 🧠 Concept Overview

- **What it is** — Azure Monitor is the native monitoring service in Azure. It collects two fundamental types of data: **Metrics** (numerical values at a point in time, like CPU %) and **Logs** (records with different properties, like Event logs or Syslogs). 
- **Why it matters** — In a cloud environment, you don't have physical access to servers. You rely entirely on telemetry. Azure Monitor provides a single pane of glass to view the health of VMs, Web Apps, Databases, and Networks. *Cloud mein physical servers nahi hote, toh metrics aur logs hi apka ekmatra sahara hain.*
- **Where you see this** — Setting up CPU alerts for a web server, querying past logs to find out why a server rebooted unexpectedly, or enabling diagnostic settings on a Key Vault to see who accessed a secret.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Checks basic Azure portal metrics (CPU, Memory, Disk), acknowledges triggered alerts in the portal. *Basic resources ka health check karta hai.* |
| **L2** | Writes KQL queries in Log Analytics, configures Diagnostic Settings, sets up Action Groups (email/SMS alerts). *Queries likhta hai aur alerts configure karta hai.* |
| **L3** | Designs enterprise Log Analytics Workspace architecture (cost optimization), integrates with Azure Sentinel, automates remediation via Logic Apps. *Architecture design aur automation handle karta hai.* |

> [!tip] Seedha Simple Mein
> *Azure Monitor cloud ka CCTV camera aur DVR system hai. Metrics tumhari live CCTV feed hai jo batati hai abhi kya chal raha hai, aur Log Analytics Workspace tumhara DVR hai jahan pura purana record save hota hai, jisko tum KQL use karke search kar sakte ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Modern Hospital System** is like **Azure Monitor** because...
>
> - **Metrics (Heart Rate Monitor)** = Real-time data. It just shows numbers. If heart rate crosses 120, a machine beeps. *Jaise patient ka live heart rate.*
> - **Logs (Patient Medical File)** = Detailed history. It has paragraphs of text about what medicine was given and when. *Jaise patient ki poori medical history file.*
> - **Log Analytics Workspace (LAW)** = The hospital's central records room where every patient's file is kept safely. *Woh record room jahan sabhi files safe rehti hain.*
> - **Kusto Query Language (KQL)** = The fast search system the receptionist uses to find "All patients who had a fever yesterday". *Ek advanced search tool files dhoondhne ke liye.*
> - **Action Groups** = The emergency pager system that automatically calls the doctor when a critical alert triggers. *Emergency alarm jo seedha doctor ko bulata hai.*

---
## 🔬 Technical Deep Dive

### 1. Log Analytics Workspace (LAW)

> [!info] Key Concept
> The foundation of Azure logging. When a VM or an Azure Service generates logs, it needs a place to store them. 

- You must create a LAW before you can collect any logs. *Bina LAW banaye logs store nahi ho sakte.*
- Data retention can be configured (from 30 days up to 730 days).
- **Cost Warning:** You are billed per GB of data ingested. Collecting *everything* will result in a massive Azure bill.

> [!danger] Common Mistake
> Retaining too many unneeded logs for too long will drastically inflate your cloud bill. Always customize retention policies.

### 2. Diagnostic Settings

By default, Azure services do not send their logs to a LAW. You must enable **Diagnostic Settings** on the resource.
- E.g., On an Azure SQL Database, you configure Diagnostic Settings to send `SQLSecurityAuditEvents` to your centralized Workspace.

### 3. Kusto Query Language (KQL)

KQL is a read-only query language used to process and analyze telemetry data. It is similar to SQL but uses a pipeline (`|`) structure similar to PowerShell.
- It is incredibly fast, capable of searching billions of records in seconds. *Yeh karodon logs ko kuch seconds mein search kar sakta hai.*

### 4. VM Insights & Dependency Agent

VM Insights provides a pre-built, visually rich dashboard for your virtual machines.
- It requires the **Azure Monitor Agent (AMA)** and the **Dependency Agent**.
- It visualizes CPU, RAM, and Disk, but more importantly, it maps out network dependencies (e.g., this VM talks to that Database on Port 1433).

### 5. Alerts and Action Groups

- **Alert Rule**: The condition (e.g., if CPU > 90% for 5 minutes). *Kyun alert bhejna hai, uski condition.*
- **Action Group**: What to do when the rule triggers (e.g., Email the sysadmin team, send an SMS, trigger a Webhook to ServiceNow). *Alert trigger hone pe kya action lena hai.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Azure Subscription.
> - Minimum `Contributor` access to a Resource Group.
> - At least one running Azure Virtual Machine to monitor.

### Step 1: Create a Log Analytics Workspace

1. Go to the Azure Portal search bar and type **Log Analytics workspaces**.
2. Click **Create**.
3. Select your Subscription and Resource Group.
4. Name: `LAW-Enterprise-Monitoring`.
5. Region: Same as your VMs (e.g., East US).
6. Click **Review + Create**, then **Create**.

> [!success] Expected Output
> ```
> Deployment succeeded. The workspace is now ready to receive logs.
> ```

### Step 2: Enable VM Insights and Connect the VM

1. Go to your Virtual Machine in the Azure Portal.
2. Under the **Monitoring** section on the left blade, click **Insights**.
3. Click **Enable**.
4. Select the `LAW-Enterprise-Monitoring` workspace you created in Step 1.
5. Click **Configure**. This will push the Azure Monitor Agent (AMA) to the VM.

> [!success] Expected Output
> ```
> After 5-10 minutes, the Insights tab will populate with rich performance charts and a visual Map of network connections.
> ```

### Step 3: Create an Alert Rule with an Action Group

1. Navigate to **Azure Monitor** -> **Alerts** -> **Create** -> **Alert rule**.
2. Select your VM as the Scope.
3. Under Condition, search for "Percentage CPU". 
4. Set threshold to `Greater than 85` over `5 minutes`.
5. Under Actions, click **Create action group**.
6. Name it `AG-Sysadmin-Critical`.
7. Add a notification type: Email/SMS. Enter your email address.
8. Save the Action Group and finalize the Alert Rule.

> [!success] Expected Output
> ```
> If the VM's CPU spikes above 85% for 5 minutes, you will receive an automated email from Azure.
> ```

---
## ⌨️ Command Cheat Sheet

> [!info] Key Concept
> In Azure Monitor, the "commands" are actually KQL queries executed in the Log Analytics portal.

| ⌨️ KQL Query | 🛠️ Kya karta hai | 📝 Example Use Case |
|-----------|-----------------|-----------|
| `Heartbeat \| summarize LastCall = max(TimeGenerated) by Computer` | Shows the last time each server checked in. *Dikhaata hai server last kab online tha.* | Finding offline/dead VMs |
| `Event \| where EventLog == "System" and EventLevelName == "Error"` | Pulls all System Error logs. *System ke saare error logs nikaalta hai.* | Investigating a recent Windows crash |
| `Perf \| where CounterName == "% Processor Time" \| summarize avg(CounterValue) by Computer` | Calculates average CPU per server. *Server ka average CPU dikhaata hai.* | Reporting monthly performance |
| `SecurityEvent \| where EventID == 4625` | Finds failed Windows logon attempts. *Failed login attempts dhoondhta hai.* | Security investigation for brute force |
| `Syslog \| search "error" \| take 10` | Finds the word "error" in Linux logs. *Linux logs mein error shabd khojta hai.* | Quick Linux troubleshooting |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|---------|-------|
| VM is not showing up in Log Analytics / KQL queries return nothing | The Azure Monitor Agent (AMA) is not installed, or Data Collection Rule (DCR) is missing. *Agent install nahi hai ya policy missing hai.* | Go to the VM -> Extensions. Ensure AMA is installed. Check DCR associations in Azure Monitor. |
| Alert triggers but no email is received | Action Group misconfiguration or Email blocked by spam filter. *Action Group mein galti hai ya spam mein email ja raha hai.* | Run a "Test action group" from the portal. Whitelist `azure-noreply@microsoft.com` in Exchange. |
| Log Analytics Workspace bill is extremely high ($1000+) | Collecting verbose logging (like Information level Windows Events or all IIS logs). *Faltu ke saare logs bhi jama ho rahe hain.* | Modify the Data Collection Rule to only collect "Error" and "Critical" logs. Reduce retention period from 730 days to 30 days. |
| Dependency Map in VM Insights is blank | Dependency Agent is missing or port 443 is blocked outbound. *Dependency agent load nahi hua ya network blocked hai.* | Install the Dependency Agent extension. Ensure VM can reach Azure Monitor endpoints over TCP 443. |
| KQL Query is timing out | Querying too much data (e.g., looking at 90 days of logs without summarizing). *Bohot saara data ek saath search ho raha hai.* | Use the Time Range picker to restrict to "Last 24 hours" before running heavy `search` or `join` commands. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: The "Invisible" Server Crash

> [!example] Ticket
> "The Accounting VM (ACCT-VM-01) went offline yesterday around 3:00 AM and rebooted. We need to know why."

**L1 Response:** Check the Azure portal Activity Log to see if a user manually initiated a restart. No manual restart found.
**Escalation Trigger:** Requires OS-level log investigation via KQL.
**L2 Resolution:**
1. Open Log Analytics Workspace.
2. Run this KQL Query: 
   ```kusto
   Event 
   | where Computer == "ACCT-VM-01" 
   | where EventID == 1074 or EventID == 6008 
   | project TimeGenerated, RenderedDescription
   ```
3. The query reveals a `6008` Unexpected Shutdown event, followed by a BugCheck (Blue Screen) error code.
4. Export the results and attach them to the ticket for the Server Admin team to analyze the memory dump.

### 🎫 Scenario 2: High Azure Consumption Costs

> [!example] Ticket
> "Finance noticed our Azure Monitor bill doubled this month. Please investigate and reduce costs immediately."

**L1 Response:** Direct ticket to Azure Cloud Administrator.
**Escalation Trigger:** Cost analysis requires Workspace admin rights and KQL knowledge.
**L2/L3 Resolution:**
1. Navigate to the Log Analytics Workspace -> **Usage and estimated costs**.
2. Run a query to find out which data type is consuming the most GB:
   ```kusto
   Usage 
   | where TimeGenerated > ago(30d) 
   | summarize TotalGB = sum(Quantity)/1000 by DataType 
   | sort by TotalGB desc
   ```
3. Discover that the `SecurityEvent` table is consuming 500GB because someone enabled "All Events" instead of "Common Events" in Microsoft Defender for Cloud.
4. Reconfigure the Data Collection Rule to only gather minimal security events. Cost drops by 60% the next day.

### 🎫 Scenario 3: Application Dependency Mapping

> [!example] Ticket
> "We are migrating a legacy web server to a new subnet, but the developers don't know which database servers it communicates with. Can we find out before we break it?"

**L1 Response:** Escalate to Cloud Infrastructure team.
**Escalation Trigger:** Requires advanced dependency mapping.
**L2 Resolution:**
1. Ensure VM Insights and the Dependency Agent are installed on the legacy web server.
2. Wait 24 hours for traffic to be analyzed.
3. Open Azure Monitor -> Virtual Machines -> select the legacy VM -> click **Map**.
4. The visual map clearly shows outgoing TCP traffic on port 1433 to `SQL-PRD-05` and TCP 443 to an external payment gateway.
5. Provide this map to the network team to ensure firewall rules are created for the new subnet.

---
## 🎤 Interview Questions

> [!question] Q1: What is a Log Analytics Workspace and why is it important?
> **Answer:** A Log Analytics Workspace is a logical storage container in Azure where log and metric data from multiple resources is collected, aggregated, and stored. It is important because it provides a centralized location to run KQL queries for troubleshooting, security auditing, and performance monitoring across the entire cloud environment. *Yeh ek central storage aur analyzer hai apke saare cloud data ke liye.*

> [!question] Q2: In Azure Monitor, what is the difference between a Metric and a Log?
> **Answer:** **Metrics** are numerical values collected at regular intervals (like CPU usage at 50%). They are lightweight, near real-time, and great for triggering fast alerts. **Logs** contain rich, structured text data (like a JSON payload of an error event) that happened at a specific time. Logs are used for deep troubleshooting and root cause analysis. *Metrics live numbers hote hain, logs poori story batate hain.*

> [!question] Q3: How do you trigger an email to the support team when a VM goes offline?
> **Answer:** I would create an **Alert Rule** based on a custom log query (checking the `Heartbeat` table for servers that haven't reported in 5 minutes). Then, I would attach an **Action Group** to this alert rule. The Action Group is configured with the support team's email address or SMS number. *Alert rule decide karta hai problem kab aayi, aur Action Group decide karta hai kisko notify karna hai.*

> [!question] Q4: Write a simple KQL query to find all Warning events in the System log over the last 24 hours.
> **Answer:** 
> ```kusto
> Event
> | where TimeGenerated > ago(24h)
> | where EventLog == "System"
> | where EventLevelName == "Warning"
> ```

> [!question] Q5: How do you monitor an on-premises physical server using Azure Monitor?
> **Answer:** Azure Monitor is not restricted to the cloud. You can install the **Azure Monitor Agent (AMA)** on any on-premises Windows or Linux machine using Azure Arc. Once connected, the on-prem server sends its telemetry securely over port 443 directly to your Log Analytics Workspace in Azure. *Azure Arc aur AMA ka use karke on-premise servers ko connect kiya jaa sakta hai.*

==**Exam Tip:** For AZ-104, remember that "Action Groups" define *who* to notify and *how* (Email, SMS, Webhook), while "Alert Rules" define *when* to notify (the CPU threshold or KQL query result).==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/10-Monitoring-and-Backup/MON-01 Windows Performance Monitor|MON-01 Windows Performance Monitor]] — On-premises performance tracking.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Core VM management in Azure.
- [[04-Cloud-and-Security/08-Azure/AZ104-06 Azure Monitor and Backup|AZ104-06 Azure Monitor and Backup]] — Comprehensive cloud data protection.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-05 PowerShell for Azure|PS-05 PowerShell for Azure]] — Automating workspace creation via Az module.
