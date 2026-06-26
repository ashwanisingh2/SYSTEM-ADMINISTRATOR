---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-09-intune-monitoring-and-reporting, int-09]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#advanced` `#md-102`

# INT-09: Intune Monitoring and Reporting

> [!abstract] Overview
> Yeh note Intune ki monitoring, auditing, aur reporting tools ke baare mein hai. Isme compliance dashboards, Endpoint Analytics, Audit logs, aur Graph API se reporting nikalna cover kiya gaya hai.

---
## 🧠 Concept Overview

- **What it is** — Intune ka reporting engine jo batata hai ki devices compliant hain ya nahi, apps install hue ya fail hue, aur system performance kaisa hai.
- **Why it matters** — Bina reports ke administrator ko pata nahi chalega ki policies theek se apply ho rahi hain ya fail ho rahi hain.
- **Where you see this** — Jab management pooche ki kitne laptops compliant hain, ya phir troubleshoot karna ho ki kis admin ne policy change kardi.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | App installation status dashboard check karna aur user ko update dena. |
| **L2** | Audit logs check karna, compliance report failures investigate karna, aur Endpoint Analytics dashboard monitor karna. |
| **L3** | Graph API se PowerShell scripts banakar custom reports generate karna aur Azure Log Analytics integration setup karna. |

> [!tip] Seedha Simple Mein
> *Intune reporting aur monitoring tools ke zariye aap devices ki health, app install status, system boot performance aur admins ke kiye gaye changes (audit) check karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Intune Reporting** is like **an airport control tower dashboard** because...
>
> - Compliance Reports bataate hain ki kaunsi flight (device) safety rules follow kar rahi hai.
> - App Installation Reports bataate hain ki luggage (software) properly load hua ya nahi.
> - Audit Logs "black box" hain jo ATC controllers (Admins) ki har movement record karte hain.

---
## 🔬 Technical Deep Dive

### 1. Endpoint Analytics

> [!info] Key Concept
> Endpoint Analytics user experience measure karta hai. Yeh batata hai ki boot times kitne slow hain aur kaunse apps sabse zyada crash ho rahe hain.

- Startup Performance: Hardware initialization aur Group Policy processing time measure karta hai.
- App Reliability: Apps ke crash rates measure karke reliability score deta hai.

### 2. Audit Logs & Graph API

Intune Audit logs saari admin activities record karte hain (jaise profile creation/deletion).
Graph API allow karta hai scripts ke through bulk data extract karna custom reports ke liye.

> [!danger] Common Mistake
> Interpreting an "Error" status in the App dashboard as a software failure without checking the detection rules. Agar app successfully install ho gayi par detection rule galat hai, toh Intune error hi show karega.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Intune Admin Center access
> - Graph PowerShell module

### Step 1: Query Device Info via Graph API

```powershell
# Authenticate to Graph
Connect-MgGraph -Scopes "DeviceManagementManagedDevices.Read.All"

# Fetch devices and export to CSV
$devices = Get-MgDeviceManagementManagedDevice -Select "displayName,operatingSystem,complianceState"
$devices | Export-Csv "C:\IntuneInventory.csv" -NoTypeInformation
```

> [!success] Expected Output
> ```
> Successfully exported IntuneInventory.csv with device status details.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Connect-MgGraph` | Microsoft Graph API se authenticate karta hai | `Connect-MgGraph -Scopes "DeviceManagementManagedDevices.Read.All"` |
| `Get-MgDeviceManagementManagedDevice` | Saare Intune devices ki details fetch karta hai | `Get-MgDeviceManagementManagedDevice` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| App install fails with `0x87D300C9` | Installation timed out. Installer prompt ke liye wait kar raha hai | Ensure silent switches like `/qn` or `/S` are included in the Intune app configuration. |
| Unknown configuration changes breaking connectivity | Kisi aur admin ne policy modify kar di hai | Intune Audit Logs mein Device Configuration > Create/Update filter lagakar check karein ki kisne kiya. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Sudden Spike in App Failures

> [!example] Ticket
> "Many users are reporting their laptops are very slow to boot up in the morning."

**L1 Response:** Check Task Manager for startup apps on one machine.
**Escalation Trigger:** Agar issue widespread hai aur individual machine fix se kaam nahi ban raha.
**L2 Resolution:** Endpoint Analytics > Startup Performance check karna. Identify karna ki kya naya Group Policy ya security software boot process ko delay kar raha hai.

---
## 🎤 Interview Questions

> [!question] Q1: How do you track who changed an Intune policy?
> **Answer:** Intune console mein `Tenant administration` -> `Audit logs` mein jaake Activity category ko 'Device Configuration' pe filter karte hain. Isse admin account aur modified settings pata chal jati hain.

==**Exam Tip:** Intune operational logs history purge ho jati hai, long term retention ke liye ise Azure Log Analytics se connect (Diagnostic settings) karna padta hai!==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — MDM authority and baseline setup.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Details compliance states.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details device configuration templates.
