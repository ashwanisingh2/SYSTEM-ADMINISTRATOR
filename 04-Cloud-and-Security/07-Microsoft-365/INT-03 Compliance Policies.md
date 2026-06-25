---
tags: [sysadmin, intune, compliance]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# INT-03: Compliance Policies

> [!abstract] Overview
> This note covers Intune Device Compliance Policies, detailing how to evaluate device health parameters (BitLocker, OS updates, passcode strength), set up non-compliance actions, and integrate with Conditional Access to block non-compliant devices.

---
## Concept
Think of Compliance Policies as a health check for cars parking in a secure corporate garage. The security guard has a checklist: the car must have working brakes, an active alarm system (BitLocker), a secure steering wheel lock (passcode), and registration tags that aren't expired (OS updates). If a car fails the check, it is flagged as "Non-compliant." The guard might give them a 3-day warning ticket (Grace Period), but if they don't fix the issues, they are blocked from entering the garage (Conditional Access Block).
*Seedha simple mein: Compliance Policies woh rules hain jinse device ki security health measure ki jaati hai (jaise antivirus enabled hai ya nahi). Agar rules fail hote hain toh device "Not Compliant" mark hota hai aur user ka corporate login block kar diya jata hai.*

---
## Technical Deep Dive

### 1. Device Compliance Framework
- Compliance is a binary status: a device is either **Compliant** or **Not Compliant**.
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

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Microsoft Intune Admin Center with Intune Administrator permissions, and an enrolled Windows 11 device.

### Step 1: Create a Windows 10/11 Compliance Policy
1. Open the **Microsoft Intune Admin Center** -> `Devices` -> `Compliance`.
2. Click **Create policy** -> Platform: **Windows 10 and later** -> Click **Create**.
3. Name: `Corp Windows Compliance Policy`.
4. Under **Settings**:
   - **Device Health**: Set **Require BitLocker** to **Require**. Set **Secure Boot** to **Require**.
   - **System Security**: Set **Firewall** to **Require**. Set **Antivirus** to **Require**.
   - **Device Properties**: Set **Minimum OS version** to `10.0.19045` (Windows 10 22H2).
5. Click **Next** to proceed to actions for non-compliance.

### Step 2: Configure Actions for Non-Compliance & Grace Period
Set up a phased response to non-compliance:
1. Action 1: **Mark device non-compliant** -> Schedule (days after non-compliance): `3`. (This grants a 3-day grace period).
2. Action 2: Click Add -> Choose **Send email to end user** -> Select an email template. Set Schedule to `0` (immediate notification).
3. Click **Next** -> Assign the policy to the `All Devices` group -> Click **Create**.

### Step 3: Integrate with Conditional Access
Block access for non-compliant devices:
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
To verify the policy:
1. Log into your test Windows 11 device.
2. Turn off BitLocker encryption or disable the local Windows Defender Firewall.
3. Open the **Company Portal** app -> Click on your device -> Click **Check Status**.
4. The status will update to **Not compliant**, showing which settings failed the health check.

---
## Commands Reference
```powershell
# Check client local MDM policy diagnostics registry path
Get-ChildItem -Path "HKLM:\SOFTWARE\Microsoft\Provisioning\Diagnostics\Autopilot"
# Force manual sync via command line on client machine
Get-ScheduledTask -TaskPath "\Microsoft\Windows\EnterpriseMgmt\*" | Start-ScheduledTask
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- Problem: An administrator configures a compliance policy requiring BitLocker, but all newly enrolled Windows devices report as "Error" or "Not Applicable" for the BitLocker check, even though the drives are encrypted.
- Root Cause: The policy is evaluating BitLocker status before the encryption process completes, or it is checking for hardware TPM configurations that are disabled in the BIOS.
- Fix:
  1. Go to the Intune portal -> `Devices` -> `Compliance` -> Select the policy -> Click on **Device status** to see the detailed error.
  2. Verify that TPM 2.0 is enabled in the BIOS/UEFI settings on the client machines.
  3. Ensure you have deployed a separate **Configuration Profile** that automates BitLocker encryption activation at enrollment, allowing the drive encryption to complete before compliance checks run.

**Scenario 2:**
- Problem: A user's device is marked as non-compliant due to an outdated OS version. They updated Windows, but the Intune portal still shows them as non-compliant, blocking their login.
- Root Cause: The client device has not performed an MDM check-in (sync) since the updates were installed, meaning Intune is unaware of the change.
- Fix:
  1. Instruct the user to open the **Company Portal** app on their device.
  2. Select the device, click **Settings**, and click **Sync** to force an immediate check-in.
  3. Alternatively, trigger a remote sync from the **Intune Admin Center**: Navigate to `Devices` -> `All devices` -> Select the device -> Click **Sync** on the top toolbar.

---
## Common Mistakes
> [!warning] Avoid These
> Deploying a Conditional Access policy requiring device compliance before deploying any compliance policies in Intune. This marks all devices as non-compliant and blocks all users from signing in.
> Setting the grace period to `0` days for critical updates without warning users. This immediately blocks users from logging in when an update releases, flooding the helpdesk.
> Forgetting to exclude break-glass accounts from Conditional Access policies, risking a lockout if a policy is configured incorrectly.

---
## Pro Tips
> [!tip] Field Experience
> Set the global tenant compliance policy to "Mark devices with no compliance policy assigned as: **Not Compliant**". This ensures that any rogue or unmanaged device that manages to join the network is blocked from accessing data until an administrator applies a policy.
> Provide clear instruction templates in non-compliance notification emails. Include links to self-help guides on how to enable BitLocker or run Windows Updates so users can resolve issues on their own.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Compliance | Binary health status checking settings like BitLocker, TPM, and OS updates. |
| 2 | Grace Period | Specified timeframe giving users room to fix security issues before access is blocked. |
| 3 | Non-Compliance Action | Response actions like sending warning emails, locking screens, or wiping data. |
| 4 | Conditional Access | Entra rules engine that blocks logins if a device is marked non-compliant. |
| 5 | Sync Client | Triggers local health re-evaluation, sending updated status to the Intune portal. |

---
## Interview Q&A

**Q1: How do Intune Compliance Policies work with Entra ID Conditional Access to protect corporate resources?**
A: Intune Compliance Policies act as a health assessment tool. They evaluate device parameters (e.g., BitLocker, Antivirus, OS version). If the device meets all requirements, Intune flags it as "Compliant" and syncs this status to Microsoft Entra ID. Conditional Access acts as the gatekeeper. When a user tries to sign in, the CA policy checks their Entra ID device status. If the policy requires a compliant device and the device is flagged as "Not Compliant", the login is blocked.

**Q2: An executive's laptop is flagged as non-compliant because they are traveling and haven't run Windows Updates for 30 days. They are blocked from accessing Outlook. How do you resolve this?**
A:
- **Situation**: An executive was blocked from working due to an outdated OS version triggering a compliance policy.
- **Task**: I needed to temporarily restore their access while ensuring their laptop was updated.
- **Action**: I added the executive's user account to a pre-configured "Temporary Compliance Bypass" Azure AD group. This group was excluded from the main Conditional Access policy block. I then initiated a remote sync on their device and verified access was restored.
- **Result**: The executive was able to work immediately. I scheduled a call with them that evening to install the pending updates, sync the device, and remove them from the bypass group once the machine was compliant.

**Q3: What are the security risks of leaving the default Intune setting "Mark devices with no compliance policy assigned as" set to "Compliant"?**
A: If set to "Compliant", any newly enrolled device (including personal devices or machines with no security policies applied) is flagged as compliant. This allows them to bypass Conditional Access checks and access corporate data. This represents a significant security risk, as unmanaged devices with no encryption or passcode requirements can download sensitive company data.

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes identity joins and console controls.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-02 Device Enrollment|INT-02 Device Enrollment]] — Controls how devices are onboarded before evaluation.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details configuration policies like BitLocker settings.

