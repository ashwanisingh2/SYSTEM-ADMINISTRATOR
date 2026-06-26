---
tags: [desktop-support, m365, collaboration, L1]
aliases: [licensing, licensing]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#none`

# M365 Licensing: Microsoft 365 Licensing

> [!abstract] Overview
> Yeh note Microsoft 365 Licensing ke baare mein hai, jo framework hai cloud services (Outlook, Teams, SharePoint) aur local software ko user accounts se assign karne ke liye. License errors productivity rokti hain, isliye inhe samajhna zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — Tarika jisse users ko features ka access milta hai (via SKUs jaise E3, E5, Business Premium).
- **Why it matters** — Galat license assign karne se features nahi chalte (jaise "Unlicensed Product" in Office), aur extra licenses se company ka paisa waste hota hai.
- **Where you see this** — Naye employee ko access dena, Office activation errors fix karna, aur Group-Based licensing handle karna.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Users ko M365 Admin Center mein license assign/remove karna, local Office activation tokens reset karna. |
| **L2** | Group-Based licensing errors fix karna, Shared Computer Activation (SCA) configure karna. |
| **L3** | Microsoft Enterprise Agreements manage karna, Tenant-to-Tenant migrations aur global API script assignments handle karna. |

> [!tip] Seedha Simple Mein
> *Licensing ke bina aapka user koi M365 app use nahi kar sakta. SKUs like E3 or E5 grant different levels of access, and Group-based licensing automates this using Entra groups.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **M365 Licensing** is like **Gym Memberships** because...
>
> - **Basic Plan (Business Basic)**: Sirf track aur common areas (Web apps) use kar sakte ho.
> - **Premium Plan (Business Premium)**: Machines bhi use kar sakte ho, plus personal locker milega (Desktop apps + Security).
> - **Shared Access (SCA)**: Ek equipment jispe sab log aake apna card tap karke exercise kar sakte hain bina apne personal limit ko touch kiye.

---
## 🔬 Technical Deep Dive

### 1. Group-Based Licensing

> [!info] Key Concept
> Manual assignment error-prone hai. Group-based licensing mein Entra ID Security Group pe license lagaya jata hai.

- Group banayein aur E3/E5 license assign karein.
- User ko group mein daalein = License automatically mil jayega.
- Dynamic Groups (e.g. Dept: Sales) se zero-touch assignment ho sakta hai.

### 2. Shared Computer Activation (SCA)

Default Office 5 personal devices allow karta hai. Shared environments (RDS, hospital terminals) mein SCA use hota hai jisse Office user ki identity check karke temporary token download karta hai, na ki device limit use karta hai.

> [!danger] Common Mistake
> Never delete a user's license immediately after termination if you need to run an email archive migration. Removing it immediately deletes their mailbox after 30 days. Convert to shared mailbox first!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - M365 Test Tenant
> - Microsoft Graph PowerShell modules

### Step 1: Query Licenses via PowerShell

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Directory.Read.All", "User.ReadWrite.All", "Organization.Read.All"

# Query all available License SKUs
Get-MgSubscribedSku | Select-Object SkuId, SkuPartNumber, ActiveUnits, ConsumedUnits
```

> [!success] Expected Output
> ```
> SkuPartNumber            ActiveUnits ConsumedUnits
> -------------            ----------- -------------
> ENTERPRISEPACK           250         150
> SPE_E5                   100         50
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-MgSubscribedSku` | Tenant mein available licenses dikhata hai | `Get-MgSubscribedSku` |
| `cscript ospp.vbs /dstatus` | Local Office activation license ka status dikhata hai | `cd "C:\Program Files\Microsoft Office\Office16" ; cscript ospp.vbs /dstatus` |
| `cscript ospp.vbs /unpkey:XXXXX` | Corrupted product key ko hatata hai | `cscript ospp.vbs /unpkey:VMFT9` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Office shows "Unlicensed Product" on a shared terminal | Shared Computer Activation (SCA) enabled nahi hai | Reg key `SharedComputerLicensing` ko `1` set karein under ClickToRun\Configuration. |
| Entra shows "License assignment failed: Conflict detected" | Group license aur manual license aapas mein takra rahe hain | User pe jake manually assigned conflicting license ko remove karke 'Reprocess' karein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Unlicensed Product Error

> [!example] Ticket
> "I opened Word on the lab computer and it says Unlicensed Product and I can't type anything."

**L1 Response:** Verify karo ki user ke paas M365 Business Standard/Premium ya E3/E5 license hai. Client machine pe `ospp.vbs /dstatus` chalao.
**Escalation Trigger:** Agar license hai but client PC activate nahi ho raha kyonki device limit cross ho gayi.
**L2 Resolution:** Shared lab PC pe Registry mein `SharedComputerLicensing` ko `1` set karke `%localappdata%\Microsoft\Office\16.0\Licensing` delete karna aur Word dubara open karna.

---
## 🎤 Interview Questions

> [!question] Q1: What happens to a user's data when you remove their Microsoft 365 license?
> **Answer:** User account enters a 30-day grace period. Is time mein data accessible hota hai aur license reassign karke recover ho sakta hai. 30 din baad, data permanently delete ho jata hai (unless litigation hold active ho).

==**Exam Tip:** Shared Mailboxes ko alag se license nahi chahiye hota jab tak unka size 50 GB se kam ho aur unhe in-place archive ki zaroorat na ho.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]] — The directory where groups and users are licensed.
- [[04-Cloud-and-Security/07-Microsoft-365/Exchange-Online|Exchange Online]] — Relies on licenses to provision mailbox quotas.
- [[06-Career-Growth/14-Certifications/MD-102|MD-102 Roadmap]] — Covers licensing objectives for modern endpoints.
