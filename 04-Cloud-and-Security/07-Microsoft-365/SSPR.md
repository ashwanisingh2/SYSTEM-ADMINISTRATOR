---
tags: [desktop-support, m365, sspr, security, L2]
aliases: [sspr, sspr]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#none`

# M365-11: Self-Service Password Reset (SSPR)

> [!abstract] Overview
> Yeh note Self-Service Password Reset (SSPR) aur Password Writeback pe focus karta hai. Hybrid networks mein password sync aur helpdesk ticket reduction ke liye yeh feature zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — Feature allowing users to reset their AD/Entra passwords using MFA, without calling IT.
- **Why it matters** — Password resets make up 30% of IT tickets. SSPR automates this securely.
- **Where you see this** — Password recovery portals (`aka.ms/sspr`), writeback errors on sync servers.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Guide users to register methods, guide them to use reset portal. |
| **L2** | Configure, fix, escalate — Add users to SSPR groups, configure Windows lock screen link. |
| **L3** | Architecture, design — Configure tenant-wide policies, enable Password Writeback on Entra Connect. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: User apna password khud reset kar sakta hai Authenticator app ya SMS ke zariye. IT ko call karne ki zaroorat nahi.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Password Writeback** is like **a remote control for your house lock** because...
>
> - The cloud portal is the remote control app.
> - When you change the code on your phone (cloud), it talks to the hub (Entra Connect) to change the physical lock on the door (Local AD).

---
## 🔬 Technical Deep Dive

### 1. SSPR Verification Methods

> [!info] Key Concept
> Users must prove identity using registered methods before resetting.

- **Microsoft Authenticator**: Most secure.
- **SMS / Voice Call**: Standard.
- **Alternative Email**: Less secure.
- **Security Questions**: Vulnerable to social engineering. Do not use alone!

### 2. Password Writeback
Required in hybrid setups. Cloud password changes are sent over outbound HTTPS (port 443) via Service Bus to the Entra Connect server, which uses Windows APIs to change the local AD password.

> [!danger] Common Mistake
> Enabling SSPR for cloud users but forgetting to enable Writeback on the Entra Connect server. Users reset their cloud password but their local PC login still requires the old one.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - On-prem Entra Connect server
> - M365 Tenant Admin

### Step 1: Verify Password Writeback on Sync Server

```powershell
# Check writeback config on sync server
Import-Module ADSync
Get-ADSyncAADPasswordResetConfiguration

# Enable it if needed
Set-ADSyncAADPasswordResetConfiguration -Enable $true
```

> [!success] Expected Output
> ```
> Password writeback enabled successfully.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-ADSyncAADPasswordResetConfiguration` | Checks if writeback is active locally | `Get-ADSyncAADPasswordResetConfiguration` |
| `Set-ADSyncAADPasswordResetConfiguration` | Enables/Disables writeback | `Set-ADSyncAAD... -Enable $true` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Event ID 33006: Access Denied | Sync account lacks AD permissions | Delegate 'Reset Password' rights to MSOL account in ADUC |
| Event ID 33001: Connection failed | Outbound 443 blocked | Allow sync server outbound to Azure Service Bus |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: SSPR Portal Blocked

> [!example] Ticket
> "User gets error: 'Your organization hasn't enabled password reset for you.'"

**L1 Response:** Verify if user registered contact methods.
**Escalation Trigger:** User is registered but still blocked.
**L2 Resolution:** SSPR is scoped to a specific security group. Add the user to the `G-SSPR-Users` group.

---
## 🎤 Interview Questions

> [!question] Q1: How does SSPR Writeback communicate with on-prem AD?
> **Answer:** It sends the encrypted password securely outbound over HTTPS (port 443) via Azure Service Bus to the Entra Connect agent, avoiding inbound firewall rules.

==**Exam Tip:** SSPR Writeback requires the Entra Connect sync account to have "Reset Password", "Write lockoutTime", and "Write pwdLastSet" delegated permissions on user OUs.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]]
- [[03-Identity-and-Core-Services/06-Active-Directory/Password-Policies|Password Policies]]
- [[04-Cloud-and-Security/07-Microsoft-365/MFA|MFA]]
