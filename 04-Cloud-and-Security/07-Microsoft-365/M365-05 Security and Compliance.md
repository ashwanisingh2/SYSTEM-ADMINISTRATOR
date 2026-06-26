---
tags: [desktop-support, m365, collaboration, security, L1]
aliases: [m365-05-security-and-compliance, m365-05]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#advanced` `#none`

# M365-05: Security and Compliance

> [!abstract] Overview
> Yeh note Microsoft 365 security aur compliance tools cover karta hai, jisme Defender for Office 365, Data Loss Prevention (DLP), Information Protection (Sensitivity Labels), eDiscovery, aur Conditional Access shamil hain. Ek support engineer ke liye in tools ki samajh hona zaroori hai taaki data aur identity secure rahein.

---
## 🧠 Concept Overview

- **What it is** — M365 Security and Compliance is a suite of tools that protect data, email, and identities from cyber threats.
- **Why it matters** — Real job mein data leakage ko rokna aur unauthorized access prevent karna sabse crucial task hai.
- **Where you see this** — Email phishing alerts, blocked external sharing links, and conditional access MFA prompts.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check user alerts, verify blocked emails, guide users on MFA prompts. |
| **L2** | Configure, fix, escalate — Release false-positive quarantined emails, manage basic DLP rules. |
| **L3** | Architecture, design — Design CA policies, enterprise DLP strategies, and compliance frameworks. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: M365 Security aur Compliance system identities (Conditional Access, MFA), data channels (Defender, Safe Links), aur document boundaries (DLP, Sensitivity Labels) ko protect karne ka ek unified framework hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **M365 Security and Compliance** is like **a high-security corporate facility** because...
>
> - **Conditional Access** is the security guards checking IDs at the gate.
> - **Defender for Office 365** is the automated scan tunnel checking mail packages for explosives.
> - **DLP** is the exit guard making sure you don't walk out with confidential blueprints.
> - **Sensitivity Labels** stamp physical documents with "SECRET" restricting who can open them.

---
## 🔬 Technical Deep Dive

### 1. Microsoft Defender for Office 365

> [!info] Key Concept
> Protects email, files, and collaboration channels (Teams, SharePoint, OneDrive) from cyber threats via Safe Attachments, Safe Links, and Anti-Phishing.

- **Plan 1**: Core prevention. Includes Safe Attachments, Safe Links, and Anti-Phishing.
- **Plan 2**: Active post-breach investigation (AIR, Threat Tracker, Attack Simulation).

### 2. Data Loss Prevention (DLP)
Identifies, monitors, and protects sensitive information (like SSNs, Credit Cards) across M365 locations.

### 3. Microsoft Purview Information Protection (Sensitivity Labels)
Classification and encryption of documents. Rights Management (RMS) stays embedded inside the file metadata.

### 4. Identity Protection: Conditional Access (CA)
Evaluates signals (user location, device health, risk) before granting access.

> [!danger] Common Mistake
> Enabling Conditional Access policies targeting "All Users" to require MFA without excluding a "Break-Glass" (Emergency Access) account, permanently locking out the tenant!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Microsoft 365 Tenant with Global Administrator
> - Access to Microsoft Purview portal

### Step 1: Create a Conditional Access Policy via PowerShell

```powershell
# Install and Connect using Microsoft Graph
Install-Module Microsoft.Graph -Force
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess"

# Define Policy Parameters
$policy = @{
    DisplayName = "Require MFA for Admins"
    State = "enabledForReportingButNotEnforced"
    Conditions = @{
        Users = @{ IncludeRoles = @("62e90394-69f5-4237-9190-012177145e10") }
        Applications = @{ IncludeApplications = @("All") }
    }
    GrantControls = @{ BuiltInControls = @("mfa") }
}

# Create the Policy
New-MgIdentityConditionalAccessPolicy -BodyParameter $policy
```

> [!success] Expected Output
> ```
> Id                                   DisplayName
> --                                   -----------
> abc-123...                           Require MFA for Admins
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-DlpCompliancePolicy` | Retrieves all active DLP policies | `Get-DlpCompliancePolicy` |
| `Get-RetentionCompliancePolicy` | Lists data retention policies | `Get-RetentionCompliancePolicy` |
| `Get-MgIdentityConditionalAccessPolicy` | Lists Entra CA policies | `Get-MgIdentityConditionalAccessPolicy` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Users cannot open legitimate links (Safe Links block) | Defender for Office 365 flagged destination | Add URL to Tenant Allow/Block Lists in Defender Portal |
| eDiscovery search results empty | Incorrect KQL syntax or mailbox not in scope | Correct KQL syntax and verify location scope |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Blocked External Sharing

> [!example] Ticket
> "User cannot share a spreadsheet with an external vendor. Getting a policy block error."

**L1 Response:** Verify if the spreadsheet contains sensitive info like Credit Cards.
**Escalation Trigger:** File does not contain sensitive info, but is blocked (False Positive).
**L2 Resolution:** Review DLP alerts in Purview, override the block if authorized, or adjust the DLP rule sensitivity.

---
## 🎤 Interview Questions

> [!question] Q1: Explain the difference between SPF, DKIM, and DMARC.
> **Answer:** SPF lists authorized IPs. DKIM adds cryptographic signature to email header. DMARC instructs receiving server what to do with failures (none, quarantine, reject).

==**Exam Tip:** Conditional Access policies process simultaneously. If any policy results in "Block", access is blocked regardless of other policies.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Tenant-wide user scopes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-02 Exchange Online Administration|M365-02 Exchange Online Administration]] — Mail flow routing
- [[04-Cloud-and-Security/07-Microsoft-365/INT-07 Endpoint Security in Intune|INT-07 Endpoint Security in Intune]] — Windows Defender
