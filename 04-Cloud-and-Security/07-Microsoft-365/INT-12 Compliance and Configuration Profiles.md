---
tags: [microsoft-365, intune, compliance-profiles, configuration-profiles, mdm, security]
aliases: [Intune Profiles, Compliance vs Configuration, MDM Profiles]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#intermediate` `#md-102`

# INT-12: Compliance and Configuration Profiles

> [!abstract] Overview
> Intune allows administrators to manage devices and secure organizational data by pushing policies from the cloud. In this note, we comprehensively cover the two core pillars of Microsoft Intune device management: **Compliance Policies** (Checking if a device is safe and meets baseline standards) and **Configuration Profiles** (Pushing settings, configurations, and features to a device). *Ek IT admin ko aana hi chahiye ki device ko secure aur standard kaise banaye, bina physical touch kiye.*

---
## 🧠 Concept Overview

- **What it is** — 
  - **Compliance Policies**: Set of rules that define the minimum requirements a device must meet to be considered safe (e.g., minimum OS version, BitLocker enabled, Antivirus active, no jailbreak). If a device fails, it's marked "Non-compliant" and can be blocked via Conditional Access.
  - **Configuration Profiles**: Directives and settings applied to devices to configure features and control user behavior (e.g., Wi-Fi profiles, VPN setups, restricting USB access, setting corporate wallpaper, disabling camera).
- **Why it matters** — *Device secure hona chahiye aur usme company ki policies automatically lag jani chahiye. Isse manual kaam bach jata hai aur overall enterprise security posture strong ho jata hai.*
- **Where you see this** — Onboarding new laptops/mobiles, enforcing security baselines across a global workforce, hiding start menu options, pushing corporate Wi-Fi certificates silently.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check device compliance status in Intune Portal, guide user to fix basic issues like updating OS, turning on firewall, or performing a manual device sync. |
| **L2** | Create basic configuration profiles, assign them to correct Azure AD groups, troubleshoot why a profile is failing, pending, or in a conflict state. |
| **L3** | Design complex Conditional Access policies linked with Compliance, create custom OMA-URI profiles for undocumented settings, manage enterprise-wide rollout deployment rings. |

> [!tip] Seedha Simple Mein
> *Compliance Policy ek **Security Guard** hai jo gate par check karta hai ki aap rules follow kar rahe ho ya nahi (e.g., ID card hai kya?). Configuration Policy ek **Office Boy / Interior Designer** hai jo aapki desk ko set up karta hai (e.g., Wi-Fi ka password save karna, default apps set karna).*

---
## 💡 Real-World Analogy

> [!info] Think of it like a School Uniform and a Backpack...
> **Compliance Policy** is like a **School Uniform Inspection** because...
> - If you don't wear the right uniform (like proper OS version or BitLocker enabled), you can't enter the classroom (Access to company email is blocked).
> - It doesn't put the uniform on you, it just checks if you are wearing it.
>
> **Configuration Profile** is like the **School giving you a pre-packed Backpack** because...
> - They already packed your books (Wi-Fi), geometry box (VPN), and timetable (shortcuts). 
> - You don't have to arrange it yourself; it is forced upon your backpack for standardization.

---
## 🔬 Technical Deep Dive

### 1. Compliance Policies

> [!info] Key Concept
> Compliance policies do not configure settings; they only evaluate the current state of the device against predefined rules.

- **Actions for Non-Compliance**: You can set a sequence of actions. For example:
  1. Day 0: Send an email to the user explaining how to fix the issue.
  2. Day 3: Mark device noncompliant immediately.
  3. Day 30: Retire the non-compliant device remotely.
- **Integration with Conditional Access**: *Asli taqat yahan hai.* Azure AD Conditional Access can use device compliance status to dynamically grant or block access to M365 apps. If a device becomes non-compliant, the token is revoked.
- **Grace Period**: You can configure a grace period so a device isn't immediately blocked if it falls out of compliance due to a missed update.

### 2. Configuration Profiles

> [!info] Key Concept
> Configuration profiles actively push settings to the device using Mobile Device Management (MDM) channels and Configuration Service Providers (CSPs).

- **Settings Catalog**: The modern and recommended way to create profiles. It lists thousands of searchable settings (very similar to traditional Group Policy Objects - GPO).
- **Templates**: Pre-defined templates for common administrative scenarios like Endpoint Protection, Wi-Fi, VPN, and Email configurations.
- **Custom Profiles (OMA-URI)**: Used as a fallback when a specific setting is not yet available in the Settings Catalog UI. You manually define the Open Mobile Alliance - Uniform Resource Identifier.

> [!danger] Common Mistake
> Confusing the two concepts! Never try to use a compliance policy to change a setting. *Compliance sirf check karta hai, Configuration change karta hai.* Agar wallpaper change karna hai, toh Configuration Profile banegi.

### 3. Policy Conflicts

If two configuration profiles push different values for the exact same setting to a device, it results in a **Conflict**.
- Intune does not evaluate which policy is "better" or "newer". It simply marks it as a conflict and leaves the setting unchanged on the device.
- *Hamesha groups ko carefully assign karo. Exclusion groups ka use karna seekho taaki overlapping configurations na bane.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Intune Administrator or Global Admin rights.
> - A test Windows 10/11 device enrolled in Intune via Entra ID Join.
> - Basic understanding of Azure AD Groups.

### Step 1: Create a Compliance Policy

1. Go to **Intune Admin Center** -> **Devices** -> **Compliance policies** -> **Create Policy**.
2. Platform: **Windows 10 and later**.
3. Name: `Win10-Compliance-Base`. Description: `Basic security checks for Windows endpoints.`
4. In **Compliance Settings**:
   - Device Health: Require BitLocker -> **Require**.
   - System Security: Require a password to unlock mobile devices -> **Require**.
   - Minimum OS version: `10.0.19045`.
5. Set **Actions for noncompliance**: 
   - Send email to end user -> immediately.
   - Mark device noncompliant -> 1 days.
6. Assign to your pilot testing Azure AD Group (e.g., `Intune-Pilot-Users`).

### Step 2: Create a Configuration Profile

1. Go to **Devices** -> **Configuration profiles** -> **Create profile**.
2. Platform: **Windows 10 and later**. Profile type: **Settings catalog**.
3. Name: `Win10-Config-RestrictUSB`.
4. Click **Add settings**: Search for **Storage** and select **Removable Storage Access** -> check the box for **Removable Disks: Deny write access**.
5. Close the pane. Set the toggle to **Enabled**.
6. Scope tags: Default.
7. Assign to the exact same pilot testing group.

### Step 3: Force Sync from Device

```powershell
# Open command prompt or PowerShell on the test device to trigger MDM sync manually
# Yeh command Intune server se naye policies download karta hai turant
omadmclient.exe

# You can also trigger it via Task Scheduler
schtasks /run /tn "\Microsoft\Windows\EnterpriseMgmt\MDMMaintenenceTask"
```
*Alternative UI method: Settings -> Accounts -> Access work or school -> Info -> Sync*

> [!success] Expected Output
> The device will evaluate compliance (and show Compliant/Non-Compliant in the Intune portal) and subsequently block write access to USB flash drives.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command / Portal Action | 🛠️ Kya karta hai | 📝 Example / Location |
|-----------|-----------------|-----------|
| `dsregcmd /status` | Checks Entra ID (Azure AD) join status and MDM enrollment info | Run in CMD on client |
| `Get-MdmSync` | (Optional script) Initiates MDM Sync via PowerShell | Custom script |
| **Sync** | Forces device to check in with Intune immediately | Company Portal App -> Settings -> Sync |
| **Collect Diagnostics** | Pulls logs from device directly to Intune portal as a zip file | Intune Portal -> Device -> Collect diagnostics |
| **Event Viewer (MDM Logs)** | Check deep MDM client logs locally | `Applications and Services Logs` > `Microsoft` > `Windows` > `DeviceManagement-Enterprise-Diagnostics-Provider` |
| `Sync-Device` | Graph API / PowerShell command to trigger remote sync | `Invoke-IntuneManagedDeviceSyncDevice -managedDeviceId $id` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Profile shows **Pending** for hours | Device is offline, sleeping, or MDM service is stuck on the client. | Ask user to connect to internet and click **Sync** in Company Portal. Restart the device. |
| Profile shows **Conflict** | User or device is in two different groups getting contradictory settings. | Use **Policy Troubleshooting** tool in Intune to find conflicting profiles. Remove user from one group or use Exclusions. |
| Device marked **Non-Compliant** | Device failed a check (e.g., BitLocker is encrypting but not finished 100%). | Click the device in Intune, go to **Device compliance**, click the policy to see exactly which setting failed. *Jo fail hua hai usko fix karo device pe aur dubara sync karo.* |
| Settings Catalog policy **Error** | Setting not supported on that specific OS edition (e.g., trying to push Enterprise setting to Win 11 Home). | Verify the setting documentation. Upgrade OS to Pro/Enterprise if required using edition upgrade policy. |
| **Not Applicable** status | The policy targets a platform the device doesn't run (e.g. iOS policy on Android). | Check assignment groups. Ensure dynamic device groups are filtering properly by OS. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User blocked by Conditional Access

> [!example] Ticket
> "Help, I am getting a message that my device doesn't meet organizational requirements when opening Outlook. I cannot check my emails."

**L1 Response:** Check the user's device in Intune. Navigate to Device Compliance. See what is failing (e.g., Firewall is off, or OS is outdated). *User ko guide karo firewall on karne ke liye aur device sync karne ke liye.*
**Escalation Trigger:** Firewall is on locally but Intune still shows it as off after 24 hours and multiple syncs.
**L2 Resolution:** Force sync manually via CMD. Check local Event Viewer logs under `DeviceManagement-Enterprise-Diagnostics-Provider`. Sometimes the Health Attestation service or WMI has issues. If WMI is corrupt, repair WMI or re-enroll the device.

### 🎫 Scenario 2: USB block policy is applying to the CEO incorrectly

> [!example] Ticket
> "URGENT: The CEO needs to copy sensitive files to an encrypted USB drive for a board meeting, but access is denied by IT policy."

**L1 Response:** Verify that the CEO is indeed getting the policy blocked by checking their device configuration status. Escalate to L2 immediately as this requires policy modification or exclusion.
**Escalation Trigger:** Need to safely modify Intune profile assignments to create an exception.
**L2 Resolution:** Create an Entra ID security group named "SG-Intune-USB-Bypass". Add the CEO to this group. Go to the Configuration Profile for USB Block -> Assignments -> **Excluded groups** -> Add "SG-Intune-USB-Bypass". Ask the CEO to sync the device from Company Portal.

### 🎫 Scenario 3: Wi-Fi profile failing on new laptops

> [!example] Ticket
> "The new batch of laptops in the New York office are not automatically connecting to Corporate Wi-Fi out of the box."

**L1 Response:** Check one affected laptop in the Intune portal. See if the Wi-Fi Configuration profile is assigned and check its status (Pending, Error, Succeeded).
**Escalation Trigger:** Profile shows "Error" for all NY devices.
**L2 Resolution:** Review the Wi-Fi configuration profile settings. Check if the root CA certificate required for EAP-TLS authentication was deployed BEFORE the Wi-Fi profile. *Certificates pehle aane chahiye, Wi-Fi profile baad mein.* Fix the dependency chain, ensure the certificate profile shows success, and retry the Wi-Fi profile deployment.

---
## 🎤 Interview Questions

> [!question] Q1: What is the main difference between a Compliance Policy and a Configuration Profile?
> **Answer:** A compliance policy passively evaluates the device state (safe or not safe) but does not change anything. A configuration profile actively pushes settings to the device to configure features.
> ==**Exam Tip:** Conditional access relies heavily on Compliance Policies, NOT configuration profiles.==

> [!question] Q2: If a user is targeted by two configuration profiles pushing different wallpaper images, what happens?
> **Answer:** It creates a Conflict state. Intune does not apply either of the conflicting settings, and the device retains its previous configuration. Intune does not have a "priority" or "weight" system for configuration profiles like on-premise Group Policy (GPO) does.

> [!question] Q3: What happens to a device that becomes non-compliant?
> **Answer:** It depends on the 'Actions for Non-compliance' defined in the policy. Typically, it gets marked non-compliant. If Entra ID Conditional Access is configured to "Require device to be marked as compliant", the user will lose access to corporate resources (email, Teams, SharePoint) until the device is fixed.

> [!question] Q4: How quickly do Intune policies apply to a device?
> **Answer:** Upon initial enrollment, policies apply immediately. After that, Windows devices typically sync every 8 hours. However, an admin can initiate a remote sync from the Intune portal, or the user can sync manually from the Company Portal app or Windows settings.

> [!question] Q5: Can we use Intune Configuration Profiles instead of Group Policy (GPO)?
> **Answer:** Yes, for modern managed (cloud-native) devices, Intune replaces GPO. Microsoft provides a Group Policy Analytics tool to help administrators migrate existing GPOs to Intune Settings Catalog natively.

> [!question] Q6: What is a Custom OMA-URI profile used for?
> **Answer:** It is used to configure specific MDM settings that are supported by the Windows OS (via CSPs) but do not yet have a built-in user interface option inside the Intune portal. 

> [!question] Q7: How would you troubleshoot a configuration profile that says "Error" in the Intune console?
> **Answer:** First, click into the error to see the specific setting that failed. Often, errors occur if the setting is not supported by the OS edition (e.g., pushing an Enterprise-only feature to Windows Pro), or if there's a prerequisite missing (like deploying a VPN profile before the required root certificate). Check local Event Viewer MDM logs on the client for detailed error codes.

---
## 🔗 Related Notes

- [[INT-01 Introduction to Intune|Introduction to Intune MDM]] — Basics of Device Management architecture.
- [[INT-03 Windows Autopilot|Windows Autopilot]] — How to provision devices out of the box.
- [[SEC-01 Conditional Access Basics|Conditional Access]] — How to block non-compliant devices effectively.
- [[AZ-05 Entra ID Device Joins|Entra ID Join vs Registered]] — Prerequisites for Intune device enrollment.
- [[INT-04 App Deployment|Intune App Deployment]] — Deploying Win32 and Store apps to managed devices.
