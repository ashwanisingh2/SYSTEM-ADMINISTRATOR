---
tags: [desktop-support, microsoft-365, identity, ad-connect, L2]
aliases: [azure-ad-connect, entra-connect]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#advanced` `#az-104`

# M365-07: Azure AD Connect

> [!abstract] Overview
> Yeh note Microsoft Entra Connect (Azure AD Connect) ke baare mein hai. Yeh local AD aur Entra ID (cloud) ke beech users, passwords, aur groups ko sync karta hai.

---
## 🧠 Concept Overview

- **What it is** — Entra Connect is a sync tool that replicates AD objects to the cloud.
- **Why it matters** — Provides Hybrid Identity, letting users login everywhere with one password.
- **Where you see this** — Password changes, user provisioning, ADSync troubleshooting.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Verify sync status in portal, reset basic passwords. |
| **L2** | Configure, fix, escalate — Force manual sync, check Sync Service errors, resolve duplicate UPNs. |
| **L3** | Architecture, design — Install Entra Connect, Password Writeback, manage Staging servers. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Azure AD Connect office ke local AD server aur Microsoft 365 cloud ke beech data match (sync) karta hai, taaki single password chale.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure AD Connect** is like **a mirror reflecting a room** because...
>
> - What happens in the local room (Active Directory) is reflected in the mirror (Entra ID).
> - If you change a shirt locally, the mirror updates.
> - But it's usually one-way unless you enable features like Password Writeback.

---
## 🔬 Technical Deep Dive

### 1. Hybrid Authentication Models

> [!info] Key Concept
> Defines how authentication happens for synced users.

- **PHS (Password Hash Sync)**: Syncs password hashes. Auth happens in the cloud.
- **PTA (Pass-through Auth)**: Auth requests are forwarded to on-prem agents for local validation.
- **ADFS**: Federation. Complex, legacy on-prem infrastructure.

### 2. The Synchronization Loop
Sync stages: Import -> Sync (Metaverse) -> Export. Delta runs every 30 mins.

### 3. Password Writeback
Essential for Self-Service Password Reset (SSPR). Writes cloud password changes back to local AD.

> [!danger] Common Mistake
> Installing Azure AD Connect on a second server in active mode without enabling Staging Mode. This causes directory corruption and object duplication.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - On-prem DC and a dedicated sync server.
> - Enterprise Admin + Cloud Global Admin.

### Step 1: Trigger Manual Sync

```powershell
# Run delta sync cycle
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
| `Start-ADSyncSyncCycle -PolicyType Delta` | Forces immediate delta sync | `Start-ADSyncSyncCycle -PolicyType Delta` |
| `Start-ADSyncSyncCycle -PolicyType Initial` | Forces a full complete sync | `Start-ADSyncSyncCycle -PolicyType Initial` |
| `Get-ADSyncScheduler` | Displays sync schedules | `Get-ADSyncScheduler` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `AttributeValueMustBeUnique` Error | Duplicate UPN or SMTP locally | Fix the duplicate value in ADUC |
| Password writes fail | Missing ADSync permissions | Grant "Replicating Directory Changes" in AD to ADSync account |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Changes Not Showing

> [!example] Ticket
> "I updated user's job title in AD, but it's not showing in Teams after 2 hours."

**L1 Response:** Verify local AD has the updated title.
**Escalation Trigger:** 30 min sync cycle passed, still not updated.
**L2 Resolution:** Force a Delta sync via PowerShell and check the Synchronization Service Manager for export errors on that user object.

---
## 🎤 Interview Questions

> [!question] Q1: How do you force an Entra Connect sync?
> **Answer:** Run `Start-ADSyncSyncCycle -PolicyType Delta` in PowerShell on the sync server.

==**Exam Tip:** Staging mode is used to install a backup sync server. It imports and syncs, but does NOT export changes.==

---
## 🔗 Related Notes

- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]]
- [[04-Cloud-and-Security/07-Microsoft-365/SSPR|SSPR]]
