---
tags: [azure, cloud, monitoring, backup, az-104]
aliases: [azure-monitor, azure-backup, recovery-services-vault]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE / CLOUD**

`#complete` `#intermediate` `#az-104`

# AZ104-13: Azure Monitoring and Backup

> [!abstract] Overview
> Azure Monitoring aur Backup infrastructure ki health track karne aur data loss se bachane ke liye crucial hain. Ek support engineer ko pata hona chahiye ki alerts kaise configure karein, logs kahan check karein, aur VM ya data delete hone par usko recover kaise karein. Is chapter mein hum Azure Monitor, Log Analytics, Recovery Services Vault aur Azure Backup architecture ko samjhenge.

---
## 🧠 Concept Overview

- **What it is** — Azure Monitor ek comprehensive solution hai jo cloud aur on-premises environments se telemetry data collect, analyze aur act karta hai. Azure Backup ek native service hai jo data ka backup aur recovery provide karti hai cloud mein securely aur scalable tarike se.
- **Why it matters** — Bina monitoring ke aapko issues ka pata tab chalega jab users complain karenge (reactive approach). Bina backup ke data loss hone par business band ho sakta hai. High availability aur disaster recovery ke liye dono essential hain taaki business continuity bani rahe.
- **Where you see this** — Jab CPU 90% cross kare aur aapko SMS alert aaye, ya jab koi accidental VM delete kar de aur aapko usko pichle din ke snapshot se restore karna ho, ya ransomware attack ke baad data wapas lana ho.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Alerts monitor karna, backup job failure check karna, dashboards view karna, basic portal navigation. |
| **L2** | Configure, fix, escalate — Log Analytics me KQL queries likhna, alert rules create karna, failed backups ko re-trigger karna aur VMs ya files restore karna. |
| **L3** | Architecture, design, enterprise-level — Multi-region backup strategy banana, Azure Lighthouse se cross-tenant monitoring setup karna, custom metrics aur advanced automation using Logic Apps configure karna. |

> [!tip] Seedha Simple Mein
> *Monitoring matlab CCTV camera aur speedometer jo server pe nazar rakhta hai, aur Backup matlab time machine jo data ko wapas past mein le ja kar restore kar sakta hai jab kuch kharab ho jaye.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Monitor and Backup** is like **Health Checkups and Health Insurance** because...
>
> - **Monitor (Health Checkup):** Yeh lagatar aapki pulse, BP aur sugar check karta hai (metrics) aur agar kuch ajeeb ho to doctor ko bulata hai (alerts). Agar BP badh raha hai to aapko pehle hi pata chal jata hai.
> - **Backup (Health Insurance):** Jab accident (data loss ya corruption) ho jata hai, tab yeh insurance aapko wapas apne pairo par khada karta hai (restore). Yeh ensure karta hai ki ek incident ki wajah se life khatam na ho jaye.

---
## 🔬 Technical Deep Dive

### 1. Azure Monitor Components

> [!info] Key Concept
> Azure Monitor collects two fundamental types of data: **Metrics** (numerical values at a point in time) and **Logs** (records of events with detailed context).

**Log Analytics Workspace:**
Yeh wo central repository hai jahan saara log data store hota hai. Kusto Query Language (KQL) use karke aap in logs ko query kar sakte ho.
*Isme data tables ke format mein save hota hai, jaise Event, Syslog, Perf. Yeh backbone hai Azure Monitor ka.*

**Application Insights:**
Yeh developers ke liye APM (Application Performance Management) tool hai. Yeh code-level telemetry deta hai jaise page load time, database query time, aur exceptions.
*Agar app slow hai, to Application Insights batayega ki konsi SQL query slow chal rahi hai.*

**Alerts and Action Groups:**
Alerts tab trigger hote hain jab koi specific condition meet hoti hai (e.g., CPU > 80% for 5 mins). Alert rule me 3 cheezein hoti hain:
1. Target Resource (VM, App Service, Storage)
2. Condition (Metric limit ya Log query output)
3. Action Group (Email bhejna, SMS karna, Webhook trigger karna)

> [!danger] Common Mistake
> Action Group configure karna bhool jana. Alert rule to ban gaya aur fire bhi ho raha hai, lekin kisi ko email nahi gaya. *Hamesha check karo ki Action Group me sahi email ID aur webhook mapped hai ya nahi.*

### 2. Azure Backup Architecture

> [!info] Key Concept
> **Recovery Services Vault (RSV)** ek storage entity hai jo Azure me backup copies, recovery points aur backup policies ko store karta hai. Yeh AES-256 se encrypted hota hai.

**Backup Policy:**
Policy define karti hai ki backup kab hoga (schedule) aur kitne din tak save rahega (retention).
*Example: Daily raat 2 baje backup lo aur 30 din tak save rakho. Weekly backup ko 52 weeks tak rakho.*

**Soft Delete and Security:**
Agar koi maliciously ya galti se backup delete kar de, toh Soft Delete usko 14 din tak bachakar rakhta hai. Is dauran data wapas laya ja sakta hai. Multi-User Authorization (MUA) se hum Resource Guard laga sakte hain taaki akele koi admin backup delete na kar sake.
*Yeh ek strong safety net ki tarah kaam karta hai ransomware attacks ke against.*

**Azure Site Recovery (ASR):**
Backup sirf data save karta hai, par ASR poore environment ka replica banata hai doosre region mein (Disaster Recovery).
*Agar East US data center doob gaya, to ASR aapke servers West US me start kar dega.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure Subscription with Admin access
> - Ek running Virtual Machine (VM)
> - Log Analytics aur Backup ki basic understanding

### Step 1: Create a Log Analytics Workspace

```bash
# Resource Group banayein monitoring ke liye
az group create --name RG-Monitoring --location eastus

# Log Analytics Workspace banayein
az monitor log-analytics workspace create \
  --resource-group RG-Monitoring \
  --workspace-name MyCentralWorkspace
```

> [!success] Expected Output
> JSON format mein newly created workspace ki details show hongi, jisme `customerId` (Workspace ID) aur provision state include hoga.

### Step 2: Configure a CPU Alert on a VM

```bash
# Alert rule create karein for CPU > 80%
az monitor metrics alert create \
  --name "HighCPUAlert" \
  --resource-group RG-Monitoring \
  --scopes /subscriptions/SUB_ID/resourceGroups/RG-VM/providers/Microsoft.Compute/virtualMachines/MyVM \
  --condition "avg Percentage CPU > 80" \
  --description "Alert triggers when CPU averages over 80% for 5 mins"
```

### Step 3: Create a Recovery Services Vault and Backup Policy

```bash
# Recovery Services Vault banayein
az backup vault create \
  --resource-group RG-Backup \
  --name MyVault \
  --location eastus

# Default policy ke sath VM ka backup enable karein
az backup protection enable-for-vm \
  --resource-group RG-VM \
  --vault-name MyVault \
  --vm MyVM \
  --policy-name DefaultPolicy
```

### Step 4: Run a simple KQL Query

```kusto
// Log Analytics me yeh query run karein
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az monitor log-analytics workspace create` | Naya workspace banata hai logs store karne ke liye | `az monitor log-analytics workspace create -g RG -n Workspc1` |
| `az monitor metrics alert create` | Metric based alert rule banata hai | `az monitor metrics alert create -n CPUAlert -g RG ...` |
| `az monitor action-group create` | Action group (Email/SMS logic) banata hai | `az monitor action-group create -g RG -n MyActionGrp ...` |
| `az backup vault create` | Recovery Services Vault (RSV) banata hai | `az backup vault create -g RG -n MyVault -l eastus` |
| `az backup protection enable-for-vm` | VM ka schedule backup start karta hai | `az backup protection enable-for-vm -g RG -v MyVault --vm VM1 -p Policy1` |
| `az backup item list` | Backup items aur unki health status ki list dikhata hai | `az backup item list -g RG -v MyVault` |
| `az backup restore restore-disks` | Backup se VM disks ko naye disks ki tarah restore karta hai | `az backup restore restore-disks -g RG -v MyVault ...` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **VM Backup Failing** | VM Agent not responding, outdated ya VM off state me hai bina managed disk ke | VM ke andar Azure VM Agent ko update/restart karein. *Agent offline hone par communication break ho jata hai.* |
| **No Alerts Received** | Action Group mein email ID galat hai ya alert rule disable hai | Action Group ki settings verify karein aur portal se test email send karke dekhein. |
| **Log Search taking too long** | Query optimize nahi hai ya time range bahut bada (like 90 days) set kiya hai | Kusto (KQL) query mein time filter (`TimeGenerated`) add karein choti range ke liye. |
| **Cannot delete Backup Vault** | Vault mein abhi bhi backup items ya soft-deleted data items hain | Pehle saare backup items ki protection stop karein (delete data ke sath), aur agar soft delete on hai to 14 days wait karein. |
| **VM Extension Provisioning Failed** | NSG ya Firewall Azure Monitor IP addresses ko block kar raha hai | VNet/NSG me check karein ki outbound internet ya port 443 open hai ya AzureMonitor Service Tag allowed hai. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: CPU Alert Triggered in Middle of Night

> [!example] Ticket
> "Critical Alert: We received an automated pager alert that WebServer-VM01 CPU usage is constantly at 95% for the last 30 minutes. The website is very slow. Please investigate immediately."

**L1 Response:** Azure portal me Metrics tab check karna aur confirm karna ki CPU waqai high hai aur false positive nahi hai. Basic health dashboard check karna.
**Escalation Trigger:** Agar VM pe RDP/SSH login na ho pa raha ho, ya application bilkul down ho aur L1 access ke bahar ho.
**L2 Resolution:** VM me login karke Task Manager / `top` command se culprit process find karna. Agar rogue process hai to kill karna. Agar genuine load hai toh server restart karna ya VM ka scale-up karna ya VMSS me instance count badhana.

### 🎫 Scenario 2: Accidental Database Server Deletion

> [!example] Ticket
> "P1 Incident: A junior developer accidentally deleted the staging database VM along with its disks. We need it restored immediately from last night's backup."

**L1 Response:** Recovery Services Vault check karna ki VM ka backup available hai ya nahi, aur latest recovery point kab ka hai.
**Escalation Trigger:** VM restore ki process start karna usually L2 ko pass ki jaati hai taaki networking override na ho.
**L2 Resolution:** Vault me ja kar "Restore VM" option select karna. Naya VM banate time existing resource group aur VNet me configuration match karna, ya keval disks restore karke unhe naye VM ke sath attach karna. *Network settings (IP, NSG) re-configure karna bahut zaroori hota hai.*

### 🎫 Scenario 3: Log Analytics Agent Disconnected

> [!example] Ticket
> "Audit Issue: We are not seeing any security event logs from FileServer-02 in our Log Analytics workspace for the past 48 hours."

**L1 Response:** VM Extensions me check karna ki Microsoft Monitoring Agent (MMA) ya Azure Monitor Agent (AMA) installed aur 'Provisioning succeeded' state me hai ya nahi.
**Escalation Trigger:** Agar agent remove karke re-install karne par bhi logs na aayein ya agent provisioning fail ho raha ho.
**L2 Resolution:** Server ke andar ja kar event logs (Application log) check karna. Firewalls (port 443) ya network NSG rules verify karna. Kai baar proxy settings ki wajah se agent log data bhej nahi pata, usko `proxyconfig` command se fix karna padta hai.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Azure Monitor Metrics and Logs?
> **Answer:** Metrics numerical data points hote hain jo system ki performance (like CPU %, memory) ko dikhate hain in real-time. Logs events ka detail record hote hain (like errors, trace logs, audit info) jinhe analyze karne ke liye KQL (Kusto Query Language) use kiya jata hai. Metrics fast aur lightweight hote hain (triggering alerts quickly), jabki logs deep troubleshooting ke liye hote hain.

==**Exam Tip:** Metrics = Numeric/Fast/Real-time (CPU). Logs = Detailed/Queryable/Complex (Event Viewer).==

> [!question] Q2: How can you protect backup data from accidental deletion by a rogue admin?
> **Answer:** Soft Delete aur Multi-User Authorization (MUA) use karke. Soft delete backup data ko delete hone ke baad bhi 14 din tak retain karta hai, jisse us duration me recovery possible hoti hai. *Resource Guard (MUA) ke through critical operations (like deleting backups ya lowering retention) ke liye doosre security admin ki permission mandate ki ja sakti hai.*

> [!question] Q3: What are Action Groups in Azure Alerts?
> **Answer:** Action Group ek collection of notification preferences hai. Jab alert fire hota hai, toh yeh define karta hai ki response kya dena hai (e.g., Email bhejna, SMS karna, Webhook call karna, ITSM tool me ticket open karna, ya Logic App trigger karna). Ek Action Group banakar multiple alert rules me reuse kiya ja sakta hai.

> [!question] Q4: How do you query logs in Azure Monitor?
> **Answer:** Logs ko query karne ke liye Kusto Query Language (KQL) ka use kiya jata hai Log Analytics Workspace me. Yeh SQL se milta-julta syntax hai lekin massive log analysis ke liye highly optimized hai. *For example: `Event | where EventLevelName == "Error" | take 10`.*

==**Exam Tip:** KQL is the standard language used in Log Analytics, Application Insights, and Sentinel. Read up on basic `where`, `project`, `extend`, and `summarize` operators.==

> [!question] Q5: You want to delete a Recovery Services Vault but the portal is throwing an error. Why does this happen?
> **Answer:** Ek vault tab tak delete nahi ho sakta jab tak uske andar koi protected backup items (servers) ya soft-deleted data baaki ho. Pehle sabhi items ka backup stop karke unka data delete karna padta hai. Agar Soft Delete enabled hai (jo default hota hai), toh data delete karne ke baad bhi aapko 14 din wait karna padega vault delete karne se pehle, ya soft delete disable karke dobara try karna padega.

> [!question] Q6: What is the difference between Log Analytics Workspace and Application Insights?
> **Answer:** Log Analytics ek general-purpose log store hai jahan infrastructure, OS, aur services ke logs (Event Viewer, Syslog) aate hain. Application Insights specifically custom software applications ki performance aur usage ko track karta hai (APM - App Performance Monitoring), jaise page load times, code exceptions, aur dependency tracking (SQL queries kitna time le rahi hain). App Insights ka backend data bhi Log Analytics me hi store hota hai (Workspace-based App Insights).

---
## 🔗 Related Notes

- [[AZ104-01 Azure Virtual Machines|AZ104-01 Azure VMs]] — Monitor aur Backup inhi VMs par prominently apply hote hain.
- [[AZ104-04 Azure Storage|AZ104-04 Storage]] — Log Analytics backend me indirectly storage accounts ka use karta hai aur Backup khud ek storage mechanism hai.
- [[AZ104-05 Azure Networking|AZ104-05 Networking]] — Network Security Groups aur Firewalls jo backup/monitor agent ka outbound traffic block kar sakte hain.
- [[AZ104-07 Azure Site Recovery|AZ104-07 Disaster Recovery]] — Backup data recovery ke liye hota hai jabki ASR business continuity aur instant failover ke liye hota hai.
