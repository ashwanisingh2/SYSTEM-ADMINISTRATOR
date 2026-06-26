---
tags: [intune, mdm, mam, architecture, enrollment, m365, autopilot]
aliases: [Intune Basics, MDM Architecture, Intune Enrollment]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#intermediate` `#md-102`

# INT-01: Intune Architecture and Enrollment

> [!abstract] Overview
> *Intune ek cloud-based Mobile Device Management (MDM) aur Mobile Application Management (MAM) solution hai jo devices aur apps ko secure aur manage karne mein help karta hai. L1 aur L2 support engineers ke liye yeh samajhna zaroori hai kyunki aajkal sab kuch cloud aur remote work par shift ho chuka hai. Yeh note aapko Intune ke foundation aur device enrollment process ke baare mein guide karega, jisse aap device management aur troubleshooting effectively kar sako.*

---
## 🧠 Concept Overview

- **What it is** — Microsoft Intune is a cloud-based endpoint management solution. It manages user access and simplifies app and device management across your many devices, including mobile devices, desktop computers, and virtual endpoints. *Asaan shabdon mein, yeh ek centralized dashboard hai jahan se company apne saare laptops aur mobiles ko control aur secure karti hai.*
- **Why it matters** — With remote work, securing company data on corporate and personal devices (BYOD) is critical. Intune ensures compliance and data protection. *Agar koi employee ka laptop chori ho jaye, ya employee company chhod de, toh Intune ke through hum us laptop ka corporate data wipe kar sakte hain, takki company ki sensitive information leak na ho.*
- **Where you see this** — Onboarding new employees with Autopilot, pushing compliance policies, deploying software like Office 365, or configuring VPN/Wi-Fi profiles. *Jab naya laptop company se milta hai aur woh automatically setup hone lagta hai, saare company apps apne aap install ho jaate hain, woh sab Intune ki wajah se hota hai.*

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check device sync status, verify enrollment failure errors, reset passwords, guide users on installing Company Portal app. *User ka device Intune portal mein dikh raha hai ya nahi, aur sync work kar raha hai ya nahi, yeh check karna.* |
| **L2** | Configure, fix, escalate — Deploy applications, configure basic compliance policies, troubleshoot Autopilot failures, remote wipe devices, collect diagnostic logs. *Apps deploy karna aur enrollment issues ko deeply investigate karna event viewer logs ke through.* |
| **L3** | Architecture, design — Design Intune architecture, create custom RBAC roles, integrate Intune with Defender for Endpoint, create conditional access policies, manage Apple Business Manager tokens. *Company ka poora MDM architecture design karna aur complex integrations setup karna.* |

> [!tip] Seedha Simple Mein
> *Intune ek digital security guard aur manager hai. Yeh check karta hai ki aapka device company ke rules follow kar raha hai ya nahi (e.g. antivirus on hai, password laga hai). Agar rules (compliance) follow ho rahe hain, toh hi aapko company emails aur data ka access milega.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Apartment Complex Security System** is like **Intune Architecture** because...
>
> - **Entra ID (Azure AD)** is the resident registry book. *Yeh record rakhta hai ki building mein kaun rehta hai (identities).*
> - **Intune** is the security guard making the rules. *Yeh decide karta hai ki aap kis condition mein building ke andar aa sakte hain (mask pehna hai ya nahi, ID card hai ya nahi).*
> - **Conditional Access** is the turnstile gate. *Yeh gate tabhi khulta hai jab Intune bolta hai ki "Haan, yeh device secure hai (compliant) aur isko entry de do".*
> - **Enrollment** is the process of registering your car at the society office. *Jab tak aapki gaadi (device) society office (Intune) mein register nahi hoti, aapko society ki parking (corporate data) nahi milti.*

---
## 🔬 Technical Deep Dive

### 1. MDM vs MAM

> [!info] Key Concept
> **MDM (Mobile Device Management)** controls the entire device. **MAM (Mobile Application Management)** controls only the specific applications on the device.

- **MDM (Mobile Device Management):** Used for corporate-owned devices. You can wipe the entire device, enforce PINs, install apps, and control OS settings. *Company ka diya hua laptop ya phone, jismein admin ka poora control hota hai.*
- **MAM (Mobile Application Management):** Used for Bring Your Own Device (BYOD) scenarios. It protects corporate data within specific apps (like Outlook, Teams) without controlling the user's personal phone. *User ka apna personal phone, jismein company sirf apne emails aur chat data par control rakhti hai. Personal photos, WhatsApp aur data bilkul safe aur private rehte hain.*

> [!danger] Common Mistake
> Enrolling a user's personal BYOD phone as a fully managed MDM device instead of MAM. *Kabhi bhi user ke personal phone ko MDM mein enroll mat karo bina clear policy ke, warna user privacy compromise ho sakti hai aur company user ke personal data ko wipe kar sakti hai.*

### 2. Enrollment Methods Overview

**Windows Devices:**
- **Windows Autopilot:** Zero-touch provisioning. Device boots up, user signs in, and Intune automatically configures it. *Naya device sidha factory se user ko ship hota hai, aur email login karte hi saari settings apne aap aa jaati hain.*
- **GPO Auto-Enrollment:** For existing on-premise Active Directory joined devices to automatically enroll in Intune (Hybrid Azure AD Join). *Jo purane computers local domain se jude hain, unko automatically background mein Intune mein laana.*
- **Manual Enrollment:** Using "Access work or school" from Windows Settings. *Settings mein jaakar manually office ka email id daal kar device register karna.*

**iOS / Android Devices:**
- **Company Portal App:** The user downloads the app from the public store and signs in. *Play Store ya App Store se Company Portal app download karke manually login karna.*
- **Apple Business Manager (ADE/DEP):** Corporate iOS devices are automatically supervised and enrolled out of the box. *Apple ke corporate devices (iPhones/Macs) ke liye auto-enrollment jo factory se link hota hai.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Microsoft 365 E3/E5, EMS E3/E5, or Microsoft Intune Plan 1 license assigned to the test user.
> - Intune MDM Authority must be set to "Microsoft Intune".
> - A test Windows 10 or Windows 11 Virtual Machine.

### Step 1: Verify MDM Scope in Entra ID

1. Go to `entra.microsoft.com` (Microsoft Entra admin center).
2. Navigate to **Settings** > **Mobility (MDM and MAM)**.
3. Click on **Microsoft Intune**.
4. Set **MDM user scope** to **All** (or 'Some' and select a specific test group). *Yahan hum define karte hain ki kaun se users apne devices ko Intune mein enroll kar sakte hain. Agar yeh 'None' hai, toh enrollment fail hoga.*
5. Save the configuration.

### Step 2: Enroll a Windows Device Manually (BYOD scenario)

1. Open the test Windows 10/11 VM.
2. Click on the Start menu and go to **Settings** > **Accounts** > **Access work or school**.
3. Click the **Connect** button.
4. Enter the test user's Microsoft 365 email address.
5. Enter the password and approve the MFA prompt if required.
6. Wait for the "Setting up your device" screen to complete. *Yeh step device ko Entra ID mein register karega aur policies pull karke Intune mein enroll karega.*

> [!success] Expected Output
> ```text
> You're all set! 
> We've added your account successfully. You now have access to your organization's apps and services.
> ```

### Step 3: Verify Enrollment in Intune Portal

1. Go to `intune.microsoft.com` (Microsoft Intune admin center).
2. Navigate to **Devices** > **All devices**.
3. Search for the computer name of your test VM. *Device dashboard mein dikhna shuru ho jayega. Uski 'Managed by' value 'Intune' hogi aur 'Ownership' value 'Personal' hogi.*
4. Click on the device name to view its hardware details, compliance status, and available actions (like Sync, Restart, Wipe).

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `dsregcmd /status` | Checks Entra ID (Azure AD) join status and PRT token. *Device Entra se connected hai ya nahi, yeh check karta hai.* | `dsregcmd /status` |
| `Get-AutopilotDiagnostics` | Pulls logs for Autopilot failures. *Autopilot fail hone par PowerShell mein detailed logs nikalta hai.* | `Get-AutopilotDiagnostics -online` |
| `Sync-Device` (via Portal) | Forces the device to check in with Intune from the admin side. *Portal se device ko naye policies pull karne ke liye force karna.* | N/A (Portal Action) |
| `eventvwr.msc` | Open Event Viewer to check enrollment logs. *Intune aur enrollment errors ke deep event logs dekhna.* | `eventvwr` |
| `ms-settings:workplace` | Opens the "Access work or school" settings page directly. *Windows settings menu sidha open karne ka shortcut.* | `ms-settings:workplace` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Device not enrolling (Error 80180014)** | MDM User scope is not configured or user is not in the allowed group. *User ko tenant level par enrollment permission nahi mili hai.* | Go to Entra ID > Mobility (MDM and MAM) and set scope to 'All' or add user to the correct group. |
| **Autopilot profile not downloading** | Device hardware hash is not uploaded, or no profile is assigned to the device. *Intune ko pata nahi hai ki yeh device company ka hai.* | Upload the hardware hash CSV to Intune and ensure an Autopilot profile is assigned to the device group. |
| **Sync fails continuously** | Device has lost trust, certificates are expired, or Intune Management Extension is stuck. *Device ka connection Intune server se toot gaya hai.* | Restart "Microsoft Intune Management Extension" service. If still failing, check network firewall blocking Intune endpoints. |
| **"License not found" error during enrollment** | User does not have an Intune license assigned in the M365 admin center. *User ke account mein Intune ka license missing hai.* | Assign an EMS E3/E5 or M365 Business Premium/E3/E5 license to the user. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Cannot Enroll Personal Mobile Device

> [!example] Ticket
> "Hi IT, I am trying to add my work email to my personal iPhone using the Company Portal app, but it says my device is blocked from enrollment. Error says device type is not supported."

**L1 Response:** Check the user's license to ensure they have an Intune license. Ask for a screenshot of the exact error. *Pehle check karo license hai ya nahi aur error code verify karo.*
**Escalation Trigger:** If the user has a license but it still fails with a restriction error, escalate to L2.
**L2 Resolution:** Check the **Enrollment Device Platform Restrictions** in the Intune admin center. Often, companies intentionally block "Personally owned" iOS and Android devices from full MDM enrollment to prevent liability. The fix is to advise the user to use MAM (App Protection Policies) by just signing into the Outlook app directly, without using the Company Portal app for MDM enrollment. 

### 🎫 Scenario 2: Windows Device Not Syncing New Policies

> [!example] Ticket
> "A user's laptop has not received the new VPN client application we deployed via Intune. It shows 'Not evaluated' in the Intune portal."

**L1 Response:** Ask the user to go to Settings > Accounts > Access work or school > click on their work account > Info, and click the "Sync" button. *User ko manual sync karne bolo takki device force check-in kare.*
**Escalation Trigger:** If manual sync fails or throws an error code, escalate to L2.
**L2 Resolution:** Run `dsregcmd /status` on the user's machine to verify Azure AD PRT (Primary Refresh Token) is healthy. Check the Event Viewer `DeviceManagement-Enterprise-Diagnostics-Provider` logs. This issue is often caused by the Intune Management Extension service being stopped. Restart the "Microsoft Intune Management Extension" service via `services.msc` and initiate another sync.

### 🎫 Scenario 3: Device Shows as "Non-Compliant" and Blocks Access

> [!example] Ticket
> "User is getting a message 'You cannot access this resource because your device is not compliant' when trying to open Microsoft Teams."

**L1 Response:** Look up the device in the Intune Portal under "Devices". Click on the device and go to the "Device compliance" tab. Find out WHICH specific policy failed (e.g., BitLocker not enabled, or OS version too low). *Portal mein check karo kis exact wajah se device compliant nahi hai.*
**Escalation Trigger:** If the policy says it requires BitLocker, but the user confirms BitLocker is actually turned on.
**L2 Resolution:** Ask the user to force a manual sync. If it's a false negative (Intune thinks BitLocker is off but it's on locally), a system reboot and a manual Intune sync usually resolves the status. Otherwise, investigate the TPM chip status locally using `tpm.msc` to ensure it is ready and functioning.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Azure AD Join and Azure AD Registered?
> **Answer:** 
> - **Azure AD Join (Entra Joined):** The device is owned by the organization. The primary Windows login is a corporate account (e.g., user@company.com). *Yeh company ka laptop hota hai.*
> - **Azure AD Registered (Entra Registered):** The device is personally owned (BYOD). The primary Windows login is a local or personal Microsoft account, and the work account is added just for accessing work apps. *Yeh personal laptop hai jismein office ka kaam karte hain.*
> ==**Exam Tip:** Intune auto-enrollment works seamlessly with Azure AD Join, while Azure AD Registered often relies on MAM or manual MDM enrollment.==

> [!question] Q2: Explain what Microsoft Autopilot is and what is its main benefit.
> **Answer:** Autopilot is a collection of technologies used to set up and pre-configure new devices. It allows IT to ship a factory-new laptop directly to the end user. When the user connects to Wi-Fi and enters their corporate email, Autopilot automatically joins the device to Entra ID and pulls down the Intune configuration, apps, and policies. *Yeh ek zero-touch deployment solution hai jisme IT ko physically device touch karne ki zaroorat nahi padti.*
> ==**Exam Tip:** Autopilot requires the hardware hash of the device to be uploaded to the tenant beforehand to recognize the device.==

> [!question] Q3: How do you enforce that only Intune-managed devices can access Office 365 data?
> **Answer:** By using **Entra ID Conditional Access Policies**. You create a policy targeting Office 365 applications, and under the "Grant" controls section, you select "Require device to be marked as compliant". Since only Intune can mark a device as compliant, this effectively forces all devices to be enrolled in Intune to access emails. *Conditional access gatekeeper ka kaam karta hai.*
> ==**Exam Tip:** Conditional Access is the critical bridge between Entra ID authentication and Intune device compliance.==

> [!question] Q4: What happens if you issue a "Wipe" command versus a "Retire" command in the Intune portal?
> **Answer:** 
> - **Wipe:** Factory resets the device, removing ALL data, apps, and settings (both personal and corporate). Used primarily for lost or stolen corporate devices. *Yeh poora device format kar deta hai.*
> - **Retire:** Removes ONLY corporate data, managed apps, email profiles, and the Intune MDM profile itself. Personal data (photos, personal apps) remains completely intact. Used when an employee leaves but keeps their personal BYOD phone. *Yeh sirf office ka data delete karta hai.*
> ==**Exam Tip:** Be extremely careful with 'Wipe' on BYOD devices; always use 'Retire' for BYOD offboarding to avoid deleting personal data.==

---
## 🔗 Related Notes

- [[ENT-01 Entra ID Basics|Entra ID Basics]] — *Identity backend for Intune.*
- [[INT-02 Intune Compliance Policies|Intune Compliance Policies]] — *How rules are enforced after enrollment.*
- [[SEC-01 Conditional Access|Conditional Access]] — *How compliance ties into security gates.*
- [[WIN-03 Windows Autopilot Troubleshooting|Windows Autopilot Troubleshooting]] — *Deep dive into Autopilot scenarios.*
