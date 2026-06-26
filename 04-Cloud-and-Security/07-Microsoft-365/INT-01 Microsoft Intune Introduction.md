---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-01-microsoft-intune-introduction, int-01]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#beginner` `#md-102`

# INT-01: Microsoft Intune Introduction

> [!abstract] Overview
> Yeh note Microsoft Intune ko introduce karta hai, jisme MDM (Mobile Device Management) vs. MAM (Mobile Application Management), enrollment modes, identity joins aur co-management setups cover hote hain. Ek support engineer ko devices ko securely cloud se manage karne ke liye yeh pata hona chahiye.

---
## 🧠 Concept Overview

- **What it is** — Microsoft Intune ek cloud-based endpoint management solution hai jo user access manage karta hai aur applications/devices ko protect karta hai.
- **Why it matters** — Aaj kal hybrid work mein, IT ko company data protect karna hota hai bina devices ko manually touch kiye. Intune policies remotely push karta hai.
- **Where you see this** — Jab company new laptop bhejti hai jo auto-setup (Autopilot) hota hai, ya jab employee apne personal phone par company email access karta hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check enrollment status, guide users to install Company Portal app. |
| **L2** | Configure basic compliance policies, App Protection (MAM), troubleshoot sync issues. |
| **L3** | Architecture design, Co-management with SCCM setup, Autopilot profiles, advanced conditional access. |

> [!tip] Seedha Simple Mein
> *Intune corporate aur personal devices ko securely configure aur control karne ka central platform hai, jise MDM (device-level) aur MAM (app-data level) ke zaraye control kiya jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A High-Security Building** is like **Microsoft Intune** because...
>
> - **MDM (Company-Owned):** Aapko poori building ka access aur control milta hai. Aap jab chahein locks change kar sakte hain ya poori building empty karwa sakte hain.
> - **MAM (BYOD/Personal):** Building kisi aur ki hai, par aapne apna ek secure locker wahan rakha hai. Aap sirf locker (company app) ko control karte hain, baaki building (personal phone) se aapko koi matlab nahi.

---
## 🔬 Technical Deep Dive

### 1. MDM vs. MAM Comparison

> [!info] Key Concept
> - **MDM (Mobile Device Management)**: Enrolls the entire device under organization authority. Best for company-owned devices.
> - **MAM (Mobile Application Management)**: Enrolls only the application context. Best for BYOD (Bring Your Own Device) scenarios.

- **MDM** grants control over device settings, hardware (disable camera, USB ports), and OS upgrades. Allows full remote wipe.
- **MAM** prevents data copy/paste between corporate apps (e.g., Outlook) and personal apps (e.g., WhatsApp). Allows selective wipe of organizational data.

### 2. Enrollment Methods Comparison

| OS Platform | Enrollment Method | Device Ownership | Description |
|-------------|-------------------|------------------|-------------|
| **Windows** | Windows Autopilot | Corporate | Out-of-box automated configuration directly from the manufacturer. |
| **Windows** | GPO / Co-management| Corporate | Hybrid environments synced from on-premises AD and SCCM. |
| **iOS/Android** | BYOD (User Enrollment)| Personal | Installs Company Portal app, enforces MAM and basic PIN locks. |
| **iOS** | Apple ADE (ABM/ASM) | Corporate | Device Enrollment Program; locks device into MDM management. |
| **Android** | Android Enterprise | Corporate/Personal | Dedicated corporate device or Work Profile partition on personal. |

### 3. Entra ID Joins & Co-management

- **Azure AD Join (Entra Joined)**: Device is joined only to Entra ID. Perfect for cloud-only environments.
- **Hybrid Azure AD Join**: Device is joined to both on-prem AD DS and Entra ID. Best for transitioning to cloud.
- **Co-management**: Bridges on-premises SCCM and cloud Intune. You deploy SCCM client and enroll in Intune, then shift workloads (e.g., Endpoint Protection via Intune, updates via SCCM).

> [!danger] Common Mistake
> Setting both MDM User Scope and MAM User Scope to "All" for Windows clients. This causes Windows devices to fail auto-enrollment or register incorrectly as MAM-only, blocking GPO and configuration policies.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Microsoft 365 Tenant with Intune licenses active.
> - Access to the Microsoft Intune Admin Center.

### Step 1: Verify Tenant-Wide MDM Authority

```powershell
# You can verify MDM Authority status visually in the portal
# Navigate to Tenant administration -> Tenant status.
```

> [!success] Expected Output
> **MDM authority** should be set to **Microsoft Intune**.

### Step 2: Configure Company Portal Branding

1. Navigate to `Tenant administration` -> `Customization`.
2. Edit the **Default Policy**.
3. Set branding values:
   - Organization Name: `Contoso IT`
   - Support Phone/Email.
   - Theme Color (`#0078D4`).
4. Click **Save** to publish.

### Step 3: Configure Auto-Enrollment in Entra ID

1. Open **Microsoft Entra Portal** -> `Identity` -> `Devices` -> `Registration` -> `Mobility (MDM and MAM)`.
2. Click **Microsoft Intune**.
3. Set **MDM User scope** to **All**.
4. Set **MAM User scope** to **None** (to avoid conflicts on Windows).
5. Click **Save**.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `dsregcmd /status` | Checks Entra ID Join status (look for AzureAdJoined : YES) | `dsregcmd /status` |
| `dsregcmd /join` | Initiates manual register/join process | `dsregcmd /join` |
| `Get-Service "IntuneManagementExtension"` | Verifies Intune agent service is running | `Get-Service -Name "IntuneManagementExtension"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Windows 11 enrollment fails `0x801c0003` | Max device limit reached or user not in MDM scope | Check "Maximum devices per user" in Entra ID Device Settings. Verify MDM User Scope. |
| Android BYOD Company Portal blocked | Enrollment Restrictions block personal Android | Edit Enrollment device limit restrictions -> Platform Settings -> Allow Android (BYOD/Personal). |
| Windows device not getting Intune policies | MDM & MAM both set to "All" | Set MAM User Scope to "None" in Entra ID Mobility (MDM and MAM). |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Windows Enrollment Failure

> [!example] Ticket
> "I bought a new laptop for an employee but it's giving error 0x801c0003 during the setup wizard when signing in with company email."

**L1 Response:** Verify the user has an Intune license assigned and check if they have exceeded the device limit in Entra ID.
**Escalation Trigger:** If licenses are correct and limit is not reached but error persists.
**L2 Resolution:** Check `Mobility (MDM and MAM)` settings in Entra ID to ensure the user is in the **MDM User Scope**. Review Intune Enrollment Restrictions.

### 🎫 Scenario 2: Personal Android Blocking

> [!example] Ticket
> "I am trying to setup Outlook on my personal phone. Company Portal says my device is blocked by organization policy."

**L1 Response:** Ask user for device type and OS version.
**Escalation Trigger:** User is using standard supported Android but still blocked.
**L2 Resolution:** Go to Intune -> `Devices` -> `Enrollment` -> `Enrollment device limit restrictions`. Check Platform Settings and ensure **Android (BYOD/Personal)** is set to **Allow**.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Azure AD Joined and Hybrid Azure AD Joined devices?
> **Answer:** Azure AD Joined is cloud-only, perfect for modern/remote deployments. Hybrid Azure AD Joined means the device is joined to both on-premises AD and Entra ID, used when relying on legacy on-prem servers and GPOs.

==**Exam Tip:** Use Hybrid Join only if strictly necessary for legacy apps; modern deployments should target Entra ID Join.==

> [!question] Q2: How do you secure data on personal smartphones without full device control?
> **Answer:** Deploy a MAM-only (App Protection) policy. Require PIN for Outlook, disable copy-paste to personal apps, and block saving to personal cloud storage. This secures corporate data while keeping personal data private.

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Explains global identity and tenant provisioning.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-02 Device Enrollment|INT-02 Device Enrollment]] — Deep dive into automatic and manual device enrollment.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Details compliance enforcement.
