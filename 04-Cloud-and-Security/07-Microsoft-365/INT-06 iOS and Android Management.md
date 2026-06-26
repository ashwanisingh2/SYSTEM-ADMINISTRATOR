---
tags: [m365, intune, ios, android, mdm, mam]
aliases: [iOS/Android Intune, Mobile Device Management, MAM]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#intermediate` `#md-102`

# INT-06: iOS and Android Management

> [!abstract] Overview
> Mobile Device Management (MDM) aur Mobile Application Management (MAM) modern workplace ka core hissa hai. Microsoft Intune ke zariye hum iOS aur Android devices ko secure, enroll aur manage karte hain taaki corporate data safe rahe aur users apne personal devices (BYOD) ya company-owned devices par kaam kar sakein. Ek system admin ko MDM/MAM concepts, Apple Push Notification service (APNs), aur Managed Google Play ki aukaat samajhna bohot zaroori hai. Is note mein hum mobile devices ki end-to-end management seekhenge.

---
## 🧠 Concept Overview

- **What it is** — Intune uses MDM and MAM protocols to configure, secure, and monitor mobile devices (iOS/iPadOS and Android). It relies on OS-level APIs provided by Apple and Google.
- **Why it matters** — User log kahin se bhi kaam karte hain. Agar device chori ho jaye ya hack ho jaye, toh corporate data leak na ho isliye Intune inpe policies (like PIN, encryption, remote wipe) lagata hai. Data protection is the ultimate goal.
- **Where you see this** — Jab naya employee apni personal email pe Outlook set karta hai, toh usse Intune Company Portal download karne ko kaha jata hai, ya unhe corporate iPhone diya jata hai jo pehle se managed hota hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | User ko Company Portal install aur enroll karne mein help karna. Basic compliance check karna. Device sync aur basic troubleshooting karna. |
| **L2** | Compliance policies, App protection policies (APP) create aur troubleshoot karna. Device Wipe/Retire actions execute karna. |
| **L3** | Apple Business Manager (ABM), Managed Google Play integration, APNs lifecycle management, aur zero-touch enrollment architecture design karna. |

> [!tip] Seedha Simple Mein
> *Intune ek virtual security guard hai jo aapke phone pe baithta hai. Agar company ka phone hai (MDM), toh wo poore phone ko control karega. Agar aapka personal phone hai (MAM), toh wo sirf company ke apps (jaise Outlook/Teams) ko protect karega aur personal WhatsApp/Photos ko nahi dekhega.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Mobile Device Management (MDM)** is like **Renting a Fully Furnished Company Apartment** because...
>
> - Company owns the building and sets the rules (no pets, no loud music).
> - They have the key to enter anytime and can evict you entirely (Remote Wipe).
> - Every room is monitored for compliance.
>
> On the other hand, **Mobile Application Management (MAM / App Protection)** is like **Bringing your own lunchbox to the office** because...
> - The box is yours (BYOD), but the company decides what happens to the food inside the lunchbox (corporate data). 
> - They can't touch your personal stuff in your bag, just the work lunchbox.

---
## 🔬 Technical Deep Dive

### 1. Enrollment Methods for Android

> [!info] Key Concept
> Android Enterprise is the modern standard for managing Android devices. Device Administrator is legacy and has been deprecated by Google.

- **Android Enterprise Personally-Owned with Work Profile (BYOD):** Creates a separate "Work" container on the user's personal phone. It usually shows a briefcase icon on work apps. *User ka personal data aur apps bilkul safe rehte hain, Admin sirf work profile delete kar sakta hai.*
- **Android Enterprise Corporate-Owned Fully Managed:** Company owns the device. User has no personal profile. Admin has full control over the device settings, apps, and hardware features (like blocking the camera).
- **Android Enterprise Corporate-Owned with Work Profile (COPE):** Company owns it, but allows personal use in a separate profile. Good balance for company-issued phones.
- **Android Enterprise Dedicated (Kiosk):** Devices locked to a single app or a specific set of apps (e.g., a tablet at a restaurant or a barcode scanner in a warehouse).

### 2. Enrollment Methods for iOS/iPadOS

> [!info] Key Concept
> Apple devices strictly require an APNs (Apple Push Notification service) certificate to communicate with Intune. Without this, no iOS management is possible.

- **User Enrollment (BYOD):** Designed for personal devices. Uses Managed Apple IDs. Provides limited admin control, heavily protecting user privacy.
- **Device Enrollment:** Typical enrollment via the Intune Company Portal for user-owned or corporate devices without ABM integration.
- **Automated Device Enrollment (ADE) via Apple Business Manager (ABM):** For corporate devices purchased directly from Apple or authorized resellers. *Device box se nikalte hi, WiFi connect karte hi direct Intune mein enroll ho jata hai (Zero-touch).* Needs a token sync between ABM and Intune.

### 3. App Protection Policies (APP / MAM)

APP allows you to protect corporate data at the application level without needing to manage the entire physical device.
- **Requires:** Supported applications (e.g., Microsoft Outlook, Edge, Teams, Word). Native apps like iOS Mail are not supported.
- **Capabilities:** Require a PIN to open the app, block copy/paste from work to personal apps, prevent saving attachments to local storage or iCloud, block screenshots (on Android).

> [!danger] Common Mistake
> Forgetting to renew the APNs certificate! It expires every 365 days. *Agar APNs expire ho gaya, toh naye iOS devices enroll nahi honge aur purane manage nahi honge. Renew karte waqt EXACTLY SAME Apple ID use karna zaroori hai!*

### 4. App Configuration Policies (ACP)

App configuration policies can help you supply settings to an app before the user even runs it. For example, automatically configuring the server URL for an internal app, or turning off focused inbox in Outlook for everyone.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Intune Administrator or Global Admin rights.
> - A corporate Apple ID (for APNs).
> - A corporate Managed Google Play account.

### Step 1: Set up Apple Push Notification service (APNs) Certificate

```bash
# This is a GUI process in the Intune admin center
1. Go to Endpoint Manager > Devices > iOS/iPadOS > iOS/iPadOS enrollment.
2. Select "Apple MDM Push certificate".
3. Download the Intune CSR (Certificate Signing Request) file.
4. Go to the Apple Push Certificates Portal (identity.apple.com).
5. Login with a corporate Apple ID (Not a personal one!).
6. Create a certificate by uploading the CSR file.
7. Download the resulting .pem certificate from Apple.
8. Upload the .pem file back to Intune and enter the Apple ID used.
```

> [!success] Expected Output
> APNs status will show as "Active" with an expiration date exactly 1 year from today. iOS enrollment is now unlocked in the tenant.

### Step 2: Link Managed Google Play for Android Enterprise

```bash
# GUI Process to unlock Android Management
1. Go to Endpoint Manager > Devices > Android > Android enrollment.
2. Select "Managed Google Play".
3. Click "I agree" and "Launch Google to connect now".
4. Sign in with a corporate Google account (e.g., intuneadmin@company.com).
5. Click "Get Started" > Input company name > Agree to terms.
6. Complete registration and return to the Intune portal.
```

> [!success] Expected Output
> Managed Google Play status shows a green checkmark. All Android Enterprise enrollment profiles (Work Profile, Fully Managed, Dedicated) are now unlocked.

---
## ⌨️ Command Cheat Sheet

*(Note: Mobile management is primarily GUI/Graph API driven, but PowerShell can interact with MS Graph for reporting and bulk actions)*

| ⌨️ Command (PowerShell / Graph) | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Connect-MgGraph` | Connects to Microsoft Graph API. | `Connect-MgGraph -Scopes "DeviceManagementManagedDevices.ReadWrite.All"` |
| `Get-MgDeviceManagementManagedDevice` | Lists all enrolled devices in Intune. | `Get-MgDeviceManagementManagedDevice -All` |
| `Sync-MgDeviceManagementManagedDevice` | Forces a device to check-in with Intune immediately. | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ID>` |
| `Wipe-MgDeviceManagementManagedDevice` | Initiates a factory reset (Wipe) on the device. | `Wipe-MgDeviceManagementManagedDevice -ManagedDeviceId <ID>` |
| `Retire-MgDeviceManagementManagedDevice`| Removes only company data (MAM/MDM payload). | `Retire-MgDeviceManagementManagedDevice -ManagedDeviceId <ID>` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Users cannot enroll iOS devices. | APNs certificate expired or missing. | Renew/Create APNs certificate using the EXACT same Apple ID. |
| Android Work Profile not creating. | Device doesn't support Android Enterprise / Google Play is blocked on network. | Check OS version (needs Android 8.0+ usually). Ensure Google endpoints are unblocked on WiFi. |
| "Device cap reached" error during enrollment. | User has hit the maximum device limit (default 5 or 15 devices per user). | Go to Intune > Devices > Enrollment restrictions and increase limit, or delete stale/old devices for the user. |
| App Protection Policy not applying. | User is not targeted, or using an unsupported app (like native iOS Mail). | Verify user group membership. Tell user to use Microsoft Outlook app, not native Apple/Samsung mail apps. |
| iOS device stuck at "Awaiting final configuration" in ADE. | Intune failed to send the success signal to the device after Setup Assistant. | Usually a transient network issue. Reboot the device. Check ADE profile settings in Intune. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User leaving company, needs data wiped from personal phone

> [!example] Ticket
> "John Doe is resigning today. He uses his personal iPhone for work emails. Please ensure company data is removed."

**L1 Response:** Find the device in Intune under the user's name. Check if it's enrolled as personal (BYOD).
**Escalation Trigger:** If device is not responding, offline for many days, or the user is uncooperative.
**L2 Resolution:** Initiate a "Retire" action (not "Wipe"). *Retire command sirf company ka data (emails, apps, Wi-Fi profiles) remove karta hai, user ki personal photos aur apps safe rehti hain.*

### 🎫 Scenario 2: CEO's iPad was stolen!

> [!example] Ticket
> "URGENT: The CEO left his corporate iPad in a taxi. It contains sensitive financial docs."

**L1 Response:** Immediately search for the device in Intune. Confirm it's a corporate-owned device. Note the last check-in time.
**Escalation Trigger:** Need executive approval to wipe a C-level device.
**L2 Resolution:** Verify approval, then issue the "Wipe" command immediately. If it's cellular/WiFi connected, it will factory reset. Also, block the CEO's account sign-ins temporarily in Azure AD to be doubly safe.

### 🎫 Scenario 3: Outlook blocking copy/paste to WhatsApp

> [!example] Ticket
> "I can't copy text from a work email in Outlook and paste it into my WhatsApp to send to a client."

**L1 Response:** Explain to the user that this is expected behavior due to Data Loss Prevention (DLP) security policies.
**Escalation Trigger:** User insists it's a critical business process and demands a bypass.
**L2 Resolution:** Review the App Protection Policy (APP) targeting the user. Confirm "Restrict cut, copy and paste between other apps" is set to "Policy managed apps". Explain to management that changing this opens massive data leak risks. Do not change without InfoSec team approval.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between "Wipe" and "Retire" in Intune?
> **Answer:** "Wipe" restores the device to factory default settings, deleting all data (useful for lost, stolen, or corporate devices being repurposed). "Retire" only removes managed app data, settings, and email profiles assigned by Intune, leaving the user's personal data intact (used for BYOD scenarios when an employee leaves).

> [!question] Q2: Your company's APNs certificate expires tomorrow. What happens if you renew it using a different Apple ID than the one originally used?
> **Answer:** If you use a different Apple ID, you will generate a completely new APNs certificate. All currently enrolled iOS devices will immediately lose communication with Intune and will need to be factory reset or completely re-enrolled manually. *Yeh ek system admin ke career ki sabse badi mistake ho sakti hai! Hamesha purana Apple ID hi use karein.*

> [!question] Q3: How do you enforce Intune policies on users who refuse to enroll their personal devices?
> **Answer:** By using App Protection Policies (MAM without enrollment) combined with Azure AD Conditional Access. We can configure Conditional Access to require an "Approved Client App" or "App Protection Policy" to access Exchange Online/SharePoint. So they don't need to enroll the full device, just use the protected Outlook app.

> [!question] Q4: What is Apple Business Manager (ABM) and how does it integrate with Intune?
> **Answer:** ABM is Apple's portal for purchasing devices and apps. By linking ABM with Intune via an Automated Device Enrollment (ADE) token, corporate devices purchased from authorized resellers are automatically pushed to Intune for zero-touch provisioning. The user just turns it on, connects to WiFi, and it locks into company management without the admin touching the device.

> [!question] Q5: A user complains their Android device is showing two Google Play stores. Is this a virus?
> **Answer:** No, this is standard behavior for the Android Enterprise Work Profile. The Play Store with the little blue briefcase icon is the "Managed Google Play" store, which only contains apps approved and pushed by the company. The other one is their personal unmanaged store.

==**Exam Tip:** For MD-102 and MS-102 exams, always remember: APNs certificate requires strict yearly renewal with the same ID, and Android Device Administrator is dead (always choose Android Enterprise for any modern scenario).==

---
## 🔗 Related Notes

- [[INT-01 Introduction to Microsoft Intune|Introduction to Intune]] — Basics of Endpoint Manager architecture
- [[INT-04 Windows Autopilot|Windows Autopilot]] — The Windows equivalent of Apple ADE
- [[INT-05 Compliance Policies|Compliance Policies]] — How to check if devices meet security baselines (like Minimum OS or Jailbreak status)
- [[SEC-03 Conditional Access Basics|Conditional Access]] — Tying device compliance to login security and MFA
- [[AD-05 Azure AD vs Active Directory|Azure AD Basics]] — The identity provider behind all Intune enrollments
