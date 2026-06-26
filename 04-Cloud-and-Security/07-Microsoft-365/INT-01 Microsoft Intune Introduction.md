---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-01-microsoft-intune-introduction, int-01]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# INT-01: Microsoft Intune Introduction

> [!abstract] Overview
> This note introduces Microsoft Intune, exploring device and application management (MDM vs. MAM), enrollment modes, identity joins, and co-management setups.

---

---
## Concept Overview
Think of Microsoft Intune as a remote management center for corporate and personal devices.
- **MDM (Mobile Device Management)** is like giving an employee a company-owned laptop. You control the hardware, partition the drives, install the operating system, and configure security baselines. If they leave, you can wipe the entire device clean.
- **MAM (Mobile Application Management)** is like allowing an employee to read company emails on their personal smartphone. You don't control the phone itself, but you wrap corporate apps (like Outlook and Teams) in a secure bubble. If they leave, you only wipe the corporate data inside those apps, leaving their personal photos and messages untouched.

---

---
## Technical Deep Dive
### 1. MDM vs. MAM Comparison
- **MDM (Mobile Device Management)**:
  - Enrolls the entire device under organization authority.
  - Grants control over device settings, hardware (disable camera, USB ports), and OS upgrades.
  - Allows full remote wipe. Best for company-owned devices.
- **MAM (Mobile Application Management)**:
  - Enrolls only the application context, not the device.
  - Prevents data copy/paste between corporate apps (e.g., Outlook) and personal apps (e.g., WhatsApp).
  - Allows selective wipe of organizational data. Best for Bring Your Own Device (BYOD) scenarios.

### 2. Intune Licensing & M365 Integration
Intune is included in several subscription tiers:
- Microsoft 365 Business Premium (designed for SMBs up to 300 users)
- Microsoft 365 E3 / E5 (Enterprise-grade)
- Enterprise Mobility + Security (EMS) E3 / E5
- Microsoft 365 F1 / F3 (Frontline workers)

### 3. Enrollment Methods Comparison
| OS Platform | Enrollment Method | Device Ownership | Description |
|-------------|-------------------|------------------|-------------|
| **Windows** | Windows Autopilot | Corporate | Out-of-box automated configuration directly from the manufacturer. |
| **Windows** | GPO / Co-management| Corporate | Hybrid environments synced from on-premises AD and SCCM. |
| **iOS/Android** | BYOD (User Enrollment)| Personal | Installs Company Portal app, enforces MAM and basic PIN locks. |
| **iOS** | Apple ADE (ABM/ASM) | Corporate | Device Enrollment Program; locks device into MDM management. |
| **Android** | Android Enterprise | Corporate/Personal | Dedicated corporate device or Work Profile partition on personal. |

### 4. Entra ID Joins
- **Azure AD Join (Entra Joined)**: Device is joined only to Microsoft Entra ID. Perfect for cloud-only environments. Users sign in using their corporate email credentials.
- **Hybrid Azure AD Join**: Device is joined to both traditional on-premises Active Directory Domain Services (AD DS) and Microsoft Entra ID. Best for existing AD DS setups transitioning to cloud.
- **Workplace Join (Registered)**: Device registers with Entra ID to access resources, but is not joined to the corporate domain. Typically used for personal mobile devices (BYOD).

### 5. Intune vs. SCCM vs. Co-management
- **SCCM (System Center Configuration Manager / MECM)**: On-premises, agent-based management for local networks. Excellent for massive software distribution and server patching.
- **Intune**: Cloud-native, agentless (uses built-in OS MDM APIs) management for remote/hybrid users.
- **Co-management**: Bridges on-premises SCCM and cloud Intune. You deploy the SCCM client and enroll the device in Intune, then shift sliders (workloads) to determine which platform manages what (e.g., Endpoint Protection via Intune, Software updates via SCCM).

---

## Common Mistakes
> [!warning] Avoid These
> Setting both MDM User Scope and MAM User Scope to "All" for Windows clients. This causes Windows devices to fail auto-enrollment or register incorrectly as MAM-only, blocking GPO and configuration policies.
> Deploying Intune policies without setting up an Entra ID group hierarchy first. Ad-hoc policy targeting leads to configuration conflicts and slow deployments.
> Neglecting to configure default support branding in the Company Portal, which leaves users confused about how to contact the helpdesk during enrollment failures.

---

## Pro Tips
> [!tip] Field Experience
> Always keep at least one "Clean" test computer or VM that has no policies applied to test new configuration profiles.
> Use Entra Dynamic Groups to automate Intune targeting. For instance, target policies to a dynamic group containing all Windows 11 corporate devices: `(device.deviceOSVersion -startsWith "10.0.22") and (device.deviceOwnership -eq "Company")`.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> A Microsoft 365 Tenant with Intune licenses active. Access to the Microsoft Intune Admin Center.

### Step 1: Access the Intune Admin Center
1. Navigate to the **Microsoft Intune Admin Center** (`https://intune.microsoft.com`).
2. Explore the main blade navigation: `Devices`, `Apps`, `Users`, `Groups`, `Tenant administration`.

### Step 2: Configure Tenant-Wide MDM Authority
Verify that Intune is set as your MDM authority:
1. Navigate to `Tenant administration` -> `Tenant status`.
2. Ensure the **MDM authority** is set to **Microsoft Intune**. (If not set, trigger activation).

### Step 3: Configure Company Portal Branding
Customize the Company Portal app that users use to self-enroll:
1. Navigate to `Tenant administration` -> `Customization`.
2. Edit the **Default Policy**.
3. Set the following branding values:
   - Organization Name: `Contoso IT`
   - Support Phone Number: `+1-555-0199`
   - Support Email Address: `helpdesk@contoso.com`
   - Theme Color: Choose a sleek corporate blue hex code (`#0078D4`).
   - Logo: Upload a PNG logo.
4. Click **Save** to publish the branding.

### Step 4: Verify MDM Auto-Enrollment Settings in Entra ID
Enable auto-enrollment for Windows devices:
1. Open the **Microsoft Entra Portal** (`https://entra.microsoft.com`).
2. Go to `Identity` -> `Devices` -> `Registration` -> `Mobility (MDM and MAM)`.
3. Click on **Microsoft Intune**.
4. Set **MDM User scope** to **All** (or select a pilot group).
5. Set **MAM User scope** to **None** (to avoid enrollment conflicts on Windows devices).
6. Click **Save**.

---

---
## Cheat Sheet / Quick Reference
```powershell
dsregcmd /status                   # Runs on Windows client to verify Entra ID Join status (look for AzureAdJoined : YES)
dsregcmd /join                     # Initiates manual register/join process on client
Get-Service -Name "IntuneManagementExtension" # Verifies the Intune agent service is running on Windows client
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | MDM | Full device control, OS settings, hardware restrictions, and complete wipe capability. |
| 2 | MAM | Application data control, preventing copy/paste from corporate to personal space. |
| 3 | Company Portal | The user-facing app store and self-service portal for corporate device registration. |
| 4 | Hybrid Join | Dual join to on-premises AD and Azure AD, facilitating hybrid work scenarios. |
| 5 | Co-management | Dual management of Windows devices via both SCCM and Intune concurrently. |

---

---
## Troubleshooting
**Scenario 1:**
- Problem: An administrator tries to enroll a new Windows 11 device, but the enrollment fails with error code `0x801c0003` during the setup wizard.
- Root Cause: The maximum device limit per user has been reached in Entra ID, or the user is not included in the MDM user scope.
- Fix:
  1. Open the **Microsoft Entra Portal** -> `Identity` -> `Devices` -> `Device Settings`.
  2. Check the value of **Maximum number of devices per user** (defaults to 50, but might be set to a lower number like 5).
  3. Verify the user has an Intune license assigned.
  4. Ensure the user is within the **MDM User Scope** group configured under Mobility (MDM and MAM).

**Scenario 2:**
- Problem: A user installs the Company Portal app on a personal Android device, but it blocks enrollment, stating that the device is blocked by organization policy.
- Root Cause: Intune Enrollment Restrictions are set to block personal Android devices or restrict access to corporate-owned devices only.
- Fix:
  1. Open the **Intune Admin Center** -> `Devices` -> `Enrollment` -> `Enrollment device limit restrictions`.
  2. Edit the priority policy matching the user.
  3. Go to **Platform Settings** and ensure that **Android (BYOD/Personal)** is set to **Allow**.
  4. Save the policy and instruct the user to retry enrollment.

---

---
## Interview Questions
**Q1: What is the difference between Azure AD Joined and Hybrid Azure AD Joined devices, and when would you choose each?**
A: Azure AD Joined devices are cloud-only, signing in with Entra ID credentials and managed by Intune. They are chosen for new deployments, remote workers, or organizations transitioning away from on-premises servers. Hybrid Azure AD Joined devices are joined to both on-premises AD and Entra ID. They are chosen when an organization has a large legacy footprint, relies on on-premises GPOs, and still needs cloud integration.

**Q2: A company is rolling out a BYOD policy for personal smartphones. How would you design the enrollment flow to protect data without invading privacy?**
A:
- **Situation**: The company needed to enable access to email and Teams on personal mobile devices while respecting user privacy and securing data.
- **Task**: I needed to configure a MAM-only (App Protection) deployment.
- **Action**: I created Intune App Protection Policies targeting iOS and Android platforms. I configured the policies to require a 6-digit PIN to open Outlook, disabled copy-paste from Outlook to personal apps, and blocked saving attachments to personal cloud storage (like Google Drive). I did not require full MDM device enrollment.
- **Result**: Users could access work resources securely without the company managing their personal apps or photos, resulting in high employee adoption and secure data boundaries.

**Q3: What role does the MDM Authority play, and how do you verify it in a tenant?**
A: The MDM Authority determines which service manages mobile devices for the tenant (e.g., Intune, Office 365 MDM, or Configuration Manager). You verify this in the Intune Admin Center under Tenant Administration -> Tenant Status. The MDM authority must display "Microsoft Intune" for you to push policies and manage devices.

---

---
## Seedha Simple Mein
*Seedha simple mein: Intune corporate aur personal devices ko securely configure aur control karne ka central platform hai, jise MDM (device-level) aur MAM (app-data level) ke zaraye control kiya jata hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Explains global identity and tenant provisioning.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-02 Device Enrollment|INT-02 Device Enrollment]] — Deep dive into automatic and manual device enrollment.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Details compliance enforcement.
