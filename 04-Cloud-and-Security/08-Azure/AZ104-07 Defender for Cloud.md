---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-07-defender-for-cloud, az104-07]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#intermediate` `#az-104`

# AZ104-07: Microsoft Defender for Cloud

> [!abstract] Overview
> Yeh note Microsoft Defender for Cloud, ek cloud-native application protection platform (CNAPP) ke baare mein hai. Isme CSPM, Secure Score, Just-in-Time (JIT) VM Access, aur multi-cloud security cover kiya gaya hai. Yeh resources ki security scan karta hai aur auto-remediation playbooks chalata hai.

---
## 🧠 Concept Overview

- **What it is** — Ek central security dashboard jo aapke Azure, AWS, GCP, aur on-premise servers ko scan karta hai, threat detect karta hai aur Secure Score batata hai.
- **Why it matters** — Cloud mein choti si misconfiguration (jaise open SSH port) se bada data breach ho sakta hai. Yeh automatic audits deta hai.
- **Where you see this** — Jab kisi public VM par brute force attack ho raha ho, ya security compliance audit pass karna ho.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Secure Score dashboard monitor karna, aur low-severity security recommendations dekhna. |
| **L2** | JIT VM access configure karna, NSG block rules lagana, aur standard security fixes apply karna. |
| **L3** | Multi-cloud connectors design karna (AWS/GCP), Logic Apps playbooks banana, custom policies likhna. |

> [!tip] Seedha Simple Mein
> *Defender for Cloud cloud assets ka security guard hai. Yeh dekhta hai kahan darwaze (ports) khule hain aur hume alert karta hai. JIT Access ek special temporary pass hai jo kaam khatam hone par wapas block ho jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Defender for Cloud** is like a **Smart Security Company** for your office building because...
>
> - **CSPM (Posture Management)**: The daytime auditor who points out "Window 3 is unlocked, fix it." Gives you a Secure Score checklist.
> - **CWPP (Workload Protection)**: The armed guard in the control room who rings the alarm if someone actively tries to break in.
> - **Multi-Cloud**: It monitors the main Azure office, but also rented AWS warehouses and local branches.

---
## 🔬 Technical Deep Dive

### 1. CSPM vs. CWPP

> [!info] Key Concept
> Posture Management (CSPM) is for proactive auditing, Workload Protection (CWPP) is for active threat defense.

- **CSPM:** Included in Free Tier. Generates Secure Score based on configuration audits and compliance policies.
- **CWPP:** Paid plans (Defender for Servers/Storage/DB). Generates active Security Alerts for brute force attacks, malware, etc.

### 2. Understanding Secure Score

- Represented as a percentage. Groups recommendations into Security Controls (e.g. "Enable MFA").
- You must complete *all* recommendations within a control to get the points.

### 3. Just-in-Time (JIT) VM Access

- Blocks management ports (22, 3389) by default via NSG deny rules.
- User requests access. If approved, Defender temporarily adds an allow rule for their specific IP for a set time (e.g., 3 hours).
- After the timer expires, the rule is auto-deleted.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure Subscription with target VM `VM-PROD-WEB` and associated NSG.

### Step 1: Enable Defender for Cloud Plans

1. Azure Portal > Microsoft Defender for Cloud > **Environment settings**.
2. Select your subscription > Toggle **Defender for Servers** to **On** > Save.

### Step 2: Configure Just-in-Time (JIT) VM Access

1. Defender for Cloud > **Workload protections** > **Just-in-Time VM access**.
2. Select `VM-PROD-WEB` > **Enable JIT on 1 VM**.
3. Set Port **3389**, Allowed source IPs: `Per-request IP`, Max request time: `3 hours` > Save.
4. Try connecting RDP (fails). Click **Request Access**, open ports, try again (succeeds).

### Step 3: Remediate an Open Storage Account

1. Go to **Recommendations** in Defender.
2. Locate *"Storage accounts should restrict network access..."*.
3. Select the target storage account and click **Remediate**, or manually change its Networking settings to restrict access.
4. Secure Score will update in next scan (12-24 hours).

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az security alert list` | Lists active security alerts | `az security alert list --state Active` |
| `az security task list` | Retrieves active recommendations | `az security task list` |
| `az security pricing create` | Configures the pricing tier plan | `az security pricing create -n Servers --tier Standard` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Secure Score not updating after fix | Scheduled scan hasn't run yet | Wait 12-24 hours for the next evaluation cycle. |
| Cannot configure JIT VM access | Missing NSG write permissions | Assign `Network Contributor` role or custom `networkSecurityGroups/write` permission. |
| JIT access button greyed out | VM has no NSG attached | Associate an NSG with the VM's NIC or parent subnet. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Vendor needs Temporary Access

> [!example] Ticket
> "A third-party vendor needs RDP access to the DB server for exactly 2 hours to patch software."

**L1 Response:** Verify approval and guide them to use JIT.
**Escalation Trigger:** If JIT request fails due to missing permissions.
**L2 Resolution:** Use JIT VM Access. Select the DB server, click Request Access, input the vendor's source IP, set max time to 2 hours, and open port 3389.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Security Recommendations and Security Alerts?
> **Answer:** Recommendations are proactive suggestions (CSPM) to fix weak configurations. Alerts are reactive notifications (CWPP) of active attacks or threats on running systems.

> [!question] Q2: How does JIT VM Access secure a machine?
> **Answer:** It denies inbound administrative traffic by default. When access is requested and approved, it temporarily alters the VM's Network Security Group (NSG) to allow traffic from the requester's IP for a limited duration.

==**Exam Tip:** Secure Score points are only awarded when *all* recommendations within a specific Security Control are fully remediated.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Creating and managing Azure VMs JIT applies to.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — Detailing Network Security Groups (NSGs).
- [[04-Cloud-and-Security/09-Security/Microsoft-Sentinel-SIEM|Microsoft-Sentinel-SIEM]] — Integrating Defender alerts into Sentinel.
