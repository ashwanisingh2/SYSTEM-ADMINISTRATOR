---
tags: [desktop-support, m365, entra-id, hybrid-identity, L2]
aliases: [microsoft-entra-id, microsoft-entra-id]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#none`

# M365-09: Microsoft Entra ID

> [!abstract] Overview
> Yeh note Microsoft Entra ID (formerly Azure AD) ke baare mein hai. Yeh cloud identity, access management, aur hybrid sync ka core system hai jo support engineer ki daily life mein baar-baar aata hai.

---
## 🧠 Concept Overview

- **What it is** — Cloud-based identity and access management service (users, groups, devices).
- **Why it matters** — Iske bina M365, Teams, aur Intune me koi access nahi ho sakta.
- **Where you see this** — Password resets, group memberships, CA policies, and sign-in troubleshooting.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Reset passwords, update contact info, check group memberships. |
| **L2** | Configure, fix, escalate — Check Hybrid Entra Join, audit sign-in logs, troubleshoot sync errors. |
| **L3** | Architecture, design — Configure Entra Connect, manage SSO, design Risk Policies. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: M365 ka Active Directory. Ye cloud system tay karta hai ki kaun login kar sakta hai aur uske paas kya permissions hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Microsoft Entra ID** is like **a global passport control system** because...
>
> - Local AD is your home country's ID card.
> - Entra ID is the global passport that lets you travel across all cloud apps.
> - It checks your passport, stamps it (tokens), and grants access to specific areas.

---
## 🔬 Technical Deep Dive

### 1. Entra ID vs. AD DS

> [!info] Key Concept
> Entra ID is flat (Tenants/Users/Groups) and uses web protocols (SAML/OAuth). AD DS is hierarchical (OUs) and uses Kerberos/LDAP.

### 2. Hybrid Identity
- **Password Hash Sync (PHS)**: Highly available, authenticates directly in the cloud.
- **Pass-Through Auth (PTA)**: Authenticates locally via an agent.
- **Entra Connect Sync**: Syncs objects every 30 minutes from on-prem to cloud.

> [!danger] Common Mistake
> Deleting an on-prem synced user and trying to recover them only in the Entra ID recycle bin. The next 30-min sync will delete them again. Always restore in local AD!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Local Domain Controller with Entra Connect
> - PowerShell `Microsoft.Graph` Module

### Step 1: Force Delta Sync to Create a User

```powershell
# Run delta sync to push local AD user to cloud
Import-Module ADSync
Start-ADSyncSyncCycle -PolicyType Delta
```

> [!success] Expected Output
> ```
> Result
> ------
> Success
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Start-ADSyncSyncCycle -PolicyType Delta` | Forces sync from local AD to Entra ID | `Start-ADSyncSyncCycle -PolicyType Delta` |
| `dsregcmd /status` | Checks Entra ID device registration | `dsregcmd /status` |
| `Get-MgUser` | Queries a user in Entra ID | `Get-MgUser -UserId user@com` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Sync fails: stopped-connectivity-failure | Port 443 blocked on firewall | Open HTTPS port 443 outbound on sync server |
| Duplicate user created in cloud | UPN or SMTP mismatch (.local suffix) | Fix local UPN to routable domain, force sync |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Password Sync Failure

> [!example] Ticket
> "User changed password on local PC, but new password doesn't work in M365."

**L1 Response:** Confirm if they are a synced user or cloud-only.
**Escalation Trigger:** Password hasn't synced after 30 mins.
**L2 Resolution:** Check ADSync Sync Service Manager on server for errors; ensure Password Hash Sync is enabled and no firewall blocks exist.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between soft-match and hard-match in Entra Connect?
> **Answer:** Soft-match links accounts based on SMTP or UPN. Hard-match links them based on a unique binary identifier (sourceAnchor).

==**Exam Tip:** IDFix tool is used to scan local AD for errors before configuring Entra Connect sync.==

---
## 🔗 Related Notes

- [[03-Identity-and-Core-Services/06-Active-Directory/Users-and-Groups|Users and Groups]]
- [[04-Cloud-and-Security/07-Microsoft-365/MFA|MFA]]
- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]]
