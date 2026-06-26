---
tags: [desktop-support, m365, collaboration, L1]
aliases: [mfa, mfa]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#none`

# M365-08: Multi-Factor Authentication (MFA)

> [!abstract] Overview
> Yeh note MFA (Multi-Factor Authentication) pe focus karta hai, jisme authentication methods, Temporary Access Passes (TAP), aur security troubleshooting shamil hai. Identity security ka ek major hissa.

---
## 🧠 Concept Overview

- **What it is** — MFA blocks unauthorized access by requiring multiple forms of verification (password + app/token).
- **Why it matters** — It blocks 99.9% of identity-based attacks.
- **Where you see this** — Lost phones, MFA setup for new hires, and MFA fatigue attacks.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Reset user MFA, generate Temporary Access Passes. |
| **L2** | Configure, fix, escalate — Troubleshoot blockages in Sign-in logs, audit methods. |
| **L3** | Architecture, design — Block legacy auth, design FIDO2 / WHfB baselines. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Password leak hone pe bhi account safe rakhne ka tareeqa. User ko phone app ya token se approve karna padta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **MFA** is like **a bank safe deposit box** because...
>
> - The password is the key you have.
> - The MFA prompt is the bank manager who also needs to insert their key.
> - Both keys are needed to open the box.

---
## 🔬 Technical Deep Dive

### 1. MFA Methods

> [!info] Key Concept
> Authentication methods are ranked by security strength.

- **FIDO2 / Security Keys**: Phishing-resistant.
- **Microsoft Authenticator (Number Matching)**: High security. Prevents MFA Fatigue.
- **SMS / Voice**: Low security (vulnerable to SIM-swapping).

### 2. Disabling Legacy Authentication
Legacy protocols (IMAP/POP3) DO NOT support MFA. Attackers use them to bypass MFA entirely.

> [!danger] Common Mistake
> Never disable MFA for an account to resolve a temporary login issue. Generate a Temporary Access Pass (TAP) instead.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - M365 Tenant Admin access
> - Graph PowerShell SDK

### Step 1: Generate a TAP via PowerShell

```powershell
# Connect to Graph
Connect-MgGraph -Scopes "UserAuthenticationMethod.ReadWrite.All"

# Generate TAP
$Pass = New-MgUserAuthenticationTemporaryAccessPassMethod -UserId "testuser@company.com" -LifetimeInMinutes 30 -IsUsableOnce $true
Write-Host "TAP Passcode: $($Pass.TemporaryAccessPass)"
```

> [!success] Expected Output
> ```
> TAP Passcode: aB8!xZ9p
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `New-MgUserAuthenticationTemporaryAccessPassMethod` | Generates a TAP for a user | `New-MgUser... -UserId user@com` |
| `Revoke-MgUserSignInSession` | Revokes active sessions, forces re-login | `Revoke-MgUserSignInSession -UserId user` |
| `Get-MgUserAuthenticationMethod` | Lists user's MFA methods | `Get-MgUserAuthenticationMethod` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| User lost phone, locked out | Phone is primary MFA | Generate a TAP so user can login and register new phone |
| Receiving MFA prompts constantly (Fatigue) | Password compromised | Reset password, revoke sessions, ensure Number Matching is enabled |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: New Hire Onboarding

> [!example] Ticket
> "New hire needs to setup their account securely."

**L1 Response:** Generate a Temporary Access Pass (TAP).
**Escalation Trigger:** User cannot use TAP due to policy blocks.
**L2 Resolution:** Provide TAP securely via secondary channel; assist user with registering Authenticator app at first login without a password.

---
## 🎤 Interview Questions

> [!question] Q1: What is a Temporary Access Pass (TAP)?
> **Answer:** A time-limited passcode that allows a user to sign in without their password or MFA. Used for onboarding or lost phone scenarios.

==**Exam Tip:** Legacy authentication bypasses MFA. It must be blocked using Conditional Access or Security Defaults.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]]
- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]]
