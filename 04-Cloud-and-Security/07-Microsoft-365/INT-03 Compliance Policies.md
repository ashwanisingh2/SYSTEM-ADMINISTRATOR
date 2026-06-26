---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-03-compliance-policies, int-03]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#md-102`

# INT-03: Compliance Policies

> [!abstract] Overview
> This note covers Intune Device Compliance Policies, detailing how to evaluate device health parameters (BitLocker, OS updates, passcode strength), set up non-compliance actions, and integrate with Conditional Access to block non-compliant devices.

---
## 🧠 Concept Overview

- **What it is** — Device Compliance Policies act as a health check for enrolled devices, enforcing rules like required OS versions or encryption.
- **Why it matters** — They ensure only secure, trusted devices access corporate data, protecting the organization from threats.
- **Where you see this** — Used when configuring Intune to evaluate devices and block unmanaged or insecure devices via Conditional Access.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check compliance status in Intune and verify if devices meet requirements. |
| **L2** | Create and assign compliance policies, set non-compliance actions, and configure notifications. |
| **L3** | Design zero-trust architectures integrating Intune Compliance with Entra ID Conditional Access. |

> [!tip] Seedha Simple Mein
> *Compliance Policies woh rules hain jinse device ki security health measure ki jaati hai (jaise antivirus enabled hai ya nahi). Agar rules fail hote hain toh device "Not Compliant" mark hota hai aur user ka corporate login block kar diya jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Compliance Policies** are like **a health check for cars parking in a secure corporate garage** because...
>
> - The security guard has a checklist: the car must have working brakes, an active alarm system (BitLocker), a secure steering wheel lock (passcode), and registration tags that aren't expired (OS updates).
> - If a car fails the check, it is flagged as "Non-compliant." The guard might give them a 3-day warning ticket (Grace Period), but if they don't fix the issues, they are blocked from entering the garage (Conditional Access Block).

---
## 🔬 Technical Deep Dive

### 1. Device Compliance Framework

> [!info] Key Concept
> Compliance is a binary status: a device is either **Compliant** or **Not Compliant**.

- By default, devices that have not received a compliance policy can be marked as Compliant or Non-Compliant based on the tenant's global settings (the secure option is to mark them as Non-Compliant).
- Compliance status is sent directly to Microsoft Entra ID, allowing **Conditional Access** policies to evaluate this signal before granting access to M365 apps.

### 2. Core Compliance Settings

- **System Security**: Requires secure boot, TPM version (minimum 2.0), BitLocker encryption, Antivirus enabled, and Antispyware active.
- **Device Properties**: Minimum and maximum OS version limits (e.g., block Windows builds older than 22H2).
- **Password Policies**: Require a PIN or passcode to unlock the device, specify minimum password length, and define inactivity locks.
- **Microsoft Defender for Endpoint**: Integrates machine risk scores (Low, Medium, High) from Defender for Endpoint to determine compliance.

### 3. Actions for Non-Compliance

When a device fails evaluation, you can trigger sequential actions:
- **Mark device non-compliant**: Instantly or after a delay (Grace Period, e.g., 3 days).
- **Send email to end-user**: Automatically email the user explaining why their device is out of compliance and how to remediate it.
- **Remotely lock the device**: Forces a screen lock to protect data.
- **Retire the non-compliant device**: Wipes corporate data from the device if it remains out of compliance for an extended period (e.g., 30 days).

> [!danger] Common Mistake
> Deploying a Conditional Access policy requiring device compliance before deploying any compliance policies in Intune. This marks all devices as non-compliant and blocks all users from signing in. Setting the grace period to `0` days for critical updates without warning users immediately blocks users, flooding the helpdesk. Forgetting to exclude break-glass accounts from Conditional Access policies risks a lockout.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Microsoft Intune Admin Center with Intune Administrator permissions.
> - An enrolled Windows 11 device.

### Step 1: Create a Windows 10/11 Compliance Policy

```bash
# Step 1: In the UI
# Navigate to Devices -> Compliance -> Create Policy -> Windows 10 and later
```

1. Open the **Microsoft Intune Admin Center** -> `Devices` -> `Compliance`.
2. Click **Create policy** -> Platform: **Windows 10 and later** -> Click **Create**.
3. Name: `Corp Windows Compliance Policy`.
4. Under **Settings**:
   - **Device Health**: Set **Require BitLocker** to **Require**. Set **Secure Boot** to **Require**.
   - **System Security**: Set **Firewall** to **Require**. Set **Antivirus** to **Require**.
   - **Device Properties**: Set **Minimum OS version** to `10.0.19045` (Windows 10 22H2).
5. Click **Next** to proceed to actions for non-compliance.

### Step 2: Configure Actions for Non-Compliance & Grace Period

1. Action 1: **Mark device non-compliant** -> Schedule (days after non-compliance): `3`. (This grants a 3-day grace period).
2. Action 2: Click Add -> Choose **Send email to end user** -> Select an email template. Set Schedule to `0` (immediate notification).
3. Click **Next** -> Assign the policy to the `All Devices` group -> Click **Create**.

### Step 3: Integrate with Conditional Access

1. Open the **Intune Admin Center** -> `Endpoint Security` -> `Conditional Access` -> Click **Create new policy**.
2. Name: `Block Access from Non-Compliant Devices`.
3. Targets:
   - Users: **All users** (Exclude your break-glass admin account).
   - Target resources: **All cloud apps**.
4. Conditions:
   - Device platforms: Include **Windows**.
5. Grant Controls:
   - Select **Grant access** -> Check **Require device to be marked as compliant**.
6. Set Enable policy to **Report-only** (to test first) and click **Create**.

### Step 4: Simulate Non-Compliance on Client

1. Log into your test Windows 11 device.
2. Turn off BitLocker encryption or disable the local Windows Defender Firewall.
3. Open the **Company Portal** app -> Click on your device -> Click **Check Status**.
4. The status will update to **Not compliant**, showing which settings failed the health check.

> [!success] Expected Output
> ```
> Device status changes to "Not compliant" in Company Portal.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-ChildItem -Path "HKLM:\..."` | Check client local MDM policy diagnostics registry path | `Get-ChildItem -Path "HKLM:\SOFTWARE\Microsoft\Provisioning\Diagnostics\Autopilot"` |
| `Get-ScheduledTask ... \| Start-ScheduledTask` | Force manual sync via command line on client machine | `Get-ScheduledTask -TaskPath "\Microsoft\Windows\EnterpriseMgmt\*" \| Start-ScheduledTask` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| All newly enrolled Windows devices report as "Error" or "Not Applicable" for BitLocker check | Policy evaluating BitLocker before encryption completes, or TPM disabled in BIOS | Verify TPM 2.0 in BIOS; use Configuration Profile to automate BitLocker encryption activation before compliance checks run |
| Device non-compliant due to outdated OS, but user updated Windows | Client device hasn't performed an MDM check-in (sync) since updates installed | Instruct user to open Company Portal and Sync, or trigger remote Sync from Intune Admin Center |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Executive Blocked from Outlook

> [!example] Ticket
> "I am traveling and haven't run Windows Updates for 30 days. Now I cannot access Outlook on my laptop."

**L1 Response:** Verify compliance status and the reason for non-compliance in Intune portal.
**Escalation Trigger:** Escalate to L2 to temporarily restore access while ensuring updates apply.
**L2 Resolution:** Add the executive's user account to a pre-configured "Temporary Compliance Bypass" Entra ID group excluded from the main CA policy. Remote sync the device to restore access. Instruct user to install updates, and remove them from the bypass group once the machine is compliant.

---
## 🎤 Interview Questions

> [!question] Q1: How do Intune Compliance Policies work with Entra ID Conditional Access to protect corporate resources?
> **Answer:** Intune Compliance Policies evaluate device health parameters (e.g., BitLocker, OS version). If the device meets requirements, it is marked "Compliant", and this status syncs to Entra ID. Conditional Access acts as the gatekeeper, blocking login if the policy requires a compliant device but the device is flagged as "Not Compliant".

> [!question] Q2: What are the security risks of leaving the default Intune setting "Mark devices with no compliance policy assigned as" set to "Compliant"?
> **Answer:** Any newly enrolled device (or personal device without security policies) is automatically flagged as compliant, allowing it to bypass Conditional Access checks and access sensitive corporate data, posing a huge security risk.

==**Exam Tip:** Set the global tenant compliance policy to "Mark devices with no compliance policy assigned as: **Not Compliant**". This ensures that any rogue or unmanaged device is blocked from accessing data until an admin applies a policy.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes identity joins and console controls.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-02 Device Enrollment|INT-02 Device Enrollment]] — Controls how devices are onboarded before evaluation.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details configuration policies like BitLocker settings.
