---
tags: [desktop-support, career, certification, security, L1]
aliases: [sc-900-roadmap, roadmap-sc900]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

# SC-900: Security, Compliance, and Identity Fundamentals Roadmap

> [!note] Overview
> This roadmap note covers preparation for the Microsoft Certified: Security, Compliance, and Identity Fundamentals (SC-900) exam. It details the core concepts of security methodology (Zero Trust), Identity and Access Management (Microsoft Entra ID), Microsoft Security solutions (Defender, Sentinel), and compliance tracking (Purview).

---
## Concept Overview
- **What it is** — The SC-900 exam validates foundational-level knowledge of security, compliance, and identity (SCI) across Microsoft cloud and related services.
- **Why it matters** — Cyber security threats are at an all-time high. A support engineer must understand the core principles of identity protection, data compliance, threat intelligence, and secure access management to protect corporate assets.
- **Real job encounter** — Assisting a user with setting up Multi-Factor Authentication (MFA), troubleshooting conditional access blocking issues, and configuring data classifications.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Verify user authentication methods, assist with MFA resets, register security keys, and log basic phishing report incidents.
  - **Escalation Trigger**: Escalate when a user account is compromised, suspicious logins are flagged, or when Conditional Access policies block executive access.
  - **L2 Resolution**: Configure user risk policies, set up self-service password reset (SSPR), monitor Defender for Endpoint alerts, and audit sign-in logs.
  - **L3 Resolution**: Author Conditional Access policies, configure Microsoft Sentinel connectors, build data loss prevention (DLP) rules, and investigate system-wide breaches.

*Seedha simple mein: SC-900 exam Microsoft ke Security, Compliance, aur Identity systems ka foundation validation hai. Isme hum seekhte hain ki Zero Trust security model kya hai, Multi-Factor Authentication (MFA) kaise setup hota hai, aur Microsoft Defender aur Sentinel tools threats se kaise protect karte hain.*

---
## Technical Deep Dive

### 1. Exam Domains and Weightage
- **Describe the Concepts of Security, Compliance, and Identity (10–15%)**: Zero Trust methodology (Verify explicitly, Use least privileged access, Assume breach), Shared Responsibility, and Defense-in-Depth.
- **Describe the Capabilities of Microsoft Entra ID (25–30%)**: Entra ID identities, Authentication methods, Multi-Factor Authentication (MFA), Self-Service Password Reset (SSPR), Conditional Access, Identity Governance, and Privileged Identity Management (PIM).
- **Describe the Capabilities of Microsoft Security Solutions (35–40%)**: Microsoft Defender for Cloud, Microsoft Defender XDR (Endpoint, Identity, Office 365, Cloud Apps), and Microsoft Sentinel (SIEM/SOAR).
- **Describe the Capabilities of Microsoft Compliance Solutions (20–25%)**: Microsoft Purview (Data Protection, Data Loss Prevention/DLP, Insider Risk, eDiscovery) and Service Trust Portal.

### 2. Core Concepts: Conditional Access
Conditional Access is Microsoft's primary Zero Trust policy engine:
- **Signals**: Who is accessing (User/Group), from where (Location/IP), using what (Device state), and trying to access which application.
- **Decision**: Block access, Allow access (but require MFA), or Limit access (read-only mode).

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Microsoft Entra ID tenant (M365 Developer or Azure Free Trial).
> - Account with **Global Administrator** or **Authentication Administrator** rights.

### Step 1: Configure Self-Service Password Reset (SSPR)
SSPR allows users to reset their passwords without contacting the IT service desk.

1. Log into the Microsoft Entra admin center (`entra.microsoft.com`).
2. Navigate to **Identity** -> **Protection** -> **Password reset**.
3. Under the **Properties** blade:
   - **Self service password reset enabled**: Change from *None* to *Selected*.
   - Click **Select group**, select a test security group (e.g., `SSPR-Test-Users`), and click **Select**.
   - Click **Save** at the top.
4. Navigate to the **Authentication methods** blade:
   - Set **Number of methods required to reset** to `1`.
   - Check the boxes for **Email** and **Mobile phone (SMS only)**.
   - Click **Save**.
**Expected Result:** Users in `SSPR-Test-Users` group can now reset passwords via `passwordreset.microsoftonline.com`.

### Step 2: Set Up Multi-Factor Authentication (MFA) Registration
We will verify and reset authentication registration details for a user.

1. In the Entra Portal, navigate to **Identity** -> **Users** -> **All Users**.
2. Select a test user account (e.g., `Priya Sharma`).
3. Under the **Manage** menu, click **Authentication methods**.
4. Review the registered phone numbers or Authenticator apps.
5. If the user lost their phone, click **Require re-register multifactor authentication** at the top.
**Expected Result:** The next time the user logs in, they are prompted to configure MFA registration.

---
## Cheat Sheet / Quick Reference

| Tool / Concept | Scope | Core Purpose / Example |
|---|---|---|
| **Zero Trust** | Strategy | Security framework based on: Verify explicitly, Least privilege, Assume breach |
| **Microsoft Entra ID** | Identity | Cloud-based directory and identity access management service |
| **Conditional Access** | Policy | "If-Then" statement engine matching signals to access decisions |
| **Microsoft Defender** | XDR | Extended detection and response suite protecting endpoints and mail |
| **Microsoft Sentinel** | SIEM / SOAR| Cloud-native security information and event manager for threat tracking |
| **Microsoft Purview** | Compliance | Governance platform managing and protecting sensitive corporate data |
| **Service Trust Portal**| Audit | Microsoft portal providing independent system audit and compliance reports |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command / URL |
|---|---|---|---|
| User is blocked by Conditional Access but has the correct password and MFA. | The user's device is marked "non-compliant" or connection is originating from a blocked country IP. | Check the user's sign-in logs in Entra ID to identify the exact policy causing the block. | Navigate to Users -> Sign-in logs |
| SSPR page says: "Your administrator has not enabled password reset." | The user is not a member of the security group selected in SSPR configurations. | Add the user to the designated security group in Entra portal. | `Add-MgGroupMember` |
| MFA SMS code is not received by the user. | Local carrier routing issue, or international number format is wrong in profile. | Check that the phone number in authentication methods contains the correct country code. | E.g. `+91 9876543210` |
| Microsoft Defender agent fails to report server status. | Server lacks outbound internet access on required endpoints. | Configure local proxy settings or open firewalls on port 443 to Microsoft endpoints. | `Test-NetConnection` |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What are the three core principles of the Zero Trust security model?
> **A:** The three core principles are:
> 1. **Verify Explicitly:** Always authenticate and authorize access based on all available data points (identity, location, device health).
> 2. **Use Least Privileged Access:** Limit user access with Just-In-Time (JIT) and Just-Enough-Access (JEA) to protect data.
> 3. **Assume Breach:** Minimize damage by segmenting access, encrypting end-to-end communication, and using analytics to detect threats.

> [!question] L2 Question
> **Q:** What is the difference between a SIEM and a SOAR tool?
> **A:** A **SIEM (Security Information and Event Management)** tool (like Microsoft Sentinel) collects, aggregates, and analyzes log data across the enterprise to detect threats. A **SOAR (Security Orchestration, Automation, and Response)** tool automates the response to those threats (e.g. running a playbook to auto-block a compromised user account in Entra ID when an alert occurs).

> [!question] L3/Scenario Question
> **Q:** You notice several failed login attempts from different countries for an executive's account within minutes of each other. The account has MFA enabled. How do you respond to this, and how do you prevent future incidents?
> **A:** 
> - **Situation:** "Impossible Travel" alert suggesting a potential credential spray or brute force attack against a VIP user.
> - **Task:** Contain the threat and block unauthorized access.
> - **Action:** 
>   1. **Immediate Containment**: Revoke all active sessions for the user in the Entra portal and trigger a password reset.
>   2. **Conditional Access**: Configure a Conditional Access policy that blocks access from untrusted countries or IP ranges (Geofencing).
>   3. **Risk-Based Policy**: Enable Entra ID Protection User Risk policies that automatically block access or force a password change if a sign-in is flagged as high-risk (e.g., impossible travel).
>   4. **Investigate Logs**: Audit the sign-in logs to verify if any of the foreign connections showed "MFA completed successfully" (which would indicate MFA fatigue or SIM swapping).
> - **Result:** The account is secured, and future impossible travel logins are blocked.

---
## Seedha Simple Mein
*Seedha simple mein: SC-900 certification exam Microsoft ke security suite ke basics par focus karta hai. Isme hum Zero Trust model, self-service password reset (SSPR) setup karna, and different Defender endpoints (Endpoint, Mail, Identity) ke functions seekhte hain. Conditional Access iska sabse main logic engine hai jo signals ke basis par logins verify karta hai.*

---
## Related Notes
- [[04-Cloud-and-Security/09-Security/CIA-Triad-and-Zero-Trust|CIA-Triad-and-Zero-Trust]] — Security methodologies.
- [[04-Cloud-and-Security/09-Security/MFA-and-Identity-Protection|MFA-and-Identity-Protection]] — Dynamic MFA structures.
- [[04-Cloud-and-Security/09-Security/Microsoft-Sentinel-SIEM|Microsoft-Sentinel-SIEM]] — Sentinel SIEM analysis.

---
*Tags: #desktop-support #career #certification #security #L1*
