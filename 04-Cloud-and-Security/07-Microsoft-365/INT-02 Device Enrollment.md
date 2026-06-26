---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-02-device-enrollment, int-02]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#md-102`

# INT-02: Device Enrollment

> [!abstract] Overview
> This note covers the enrollment protocols for bringing Windows, Android, iOS/iPadOS, and macOS devices into Microsoft Intune. It explains platform-specific flows, enrollment restrictions, and troubleshooting procedures. Ek support engineer ko yeh aana chahiye kyunki device onboarding first step hota hai.

---
## 🧠 Concept Overview

- **What it is** — Device Enrollment woh step hai jis ke zariye target hardware ko Intune key credentials aur server configurations ke sath bind kiya jata hai.
- **Why it matters** — Iske bina corporate policies, security restrictions aur applications deploy nahi ho sakte.
- **Where you see this** — Jab naya employee join karta hai aur naya device setup hota hai, ya jab personal device ko company mail/apps access karne ke liye register kiya jata hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check Entra ID join status, guide user to connect account |
| **L2** | Configure enrollment restrictions, troubleshoot MDM sync errors |
| **L3** | Architecture of Autopilot profiles, setup Apple Business Manager |

> [!tip] Seedha Simple Mein
> *Device Enrollment woh step hai jis ke zariye target hardware (Windows, iOS, Android, macOS) ko Intune key credentials aur server configurations ke sath bind kiya jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Device Enrollment** is like **onboarding a new employee at a high-security office** because...
>
> - **Windows Autopilot** is like ordering a custom-tagged briefcase directly from the factory. The employee receives it, opens it, connects to Wi-Fi, and the lock combination and layouts automatically apply.
> - **Android Enterprise Work Profile** is like dividing a personal backpack into two compartments: one side has a padlock for company documents, and the other side is open for personal snacks and keys.
> - **Apple Business Manager (ADE)** is like registering a company-owned phone directly with Apple so that the moment it turns on, it immediately checks in with your company's servers.

---
## 🔬 Technical Deep Dive

### 1. Windows Enrollment Methods

> [!info] Key Concept
> Windows enrollment integrates with Entra ID to automatically apply MDM policies to devices.

- **Automatic MDM Enrollment**: Triggered when a device joins Microsoft Entra ID. The user enters their credentials during Windows Out-of-Box Experience (OOBE), and the device auto-registers in Intune.
- **Windows Autopilot**: A cloud-based service that customizes the OOBE. It allows IT departments to ship pre-configured devices directly to end-users without imaging.
- **GPO-based Enrollment**: Used for domain-joined machines. A Group Policy Object initiates background enrollment into Intune using the device's system credentials.

### 2. Android Enrollment Profiles

> [!info] Key Concept
> Android separates devices based on ownership (corporate vs personal) and use case.

- **Android Enterprise Work Profile (BYOD)**: Splits the device into personal and work profiles. Data cannot cross the profile boundaries.
- **Fully Managed (COBO)**: The entire device is managed. Users cannot install personal apps.
- **Dedicated Devices (COSU)**: Kiosk mode (e.g., ticket printing or inventory scanners).
- **Corporate-Owned with Work Profile (COPE)**: Corporate owned, but permits personal use in a split space.

### 3. iOS/iPadOS & macOS Enrollment

> [!info] Key Concept
> Apple devices use APNs and can force-enroll via Apple Business Manager.

- **Automated Device Enrollment (ADE)**: Integrates Intune with Apple Business Manager (ABM). When devices boot for the first time, they force-enroll in Intune. This prevents users from removing the MDM profile.
- **User Enrollment (BYOD)**: Protects corporate data using a managed Apple ID, keeping personal and work data separate.
- **Direct Enrollment**: Admin uses Apple Configurator to manually push the enrollment profile.

### 4. Enrollment Restrictions

> [!info] Key Concept
> Allows blocking specific platforms (e.g., block personal macOS devices), enforcing OS version minimums, and setting maximum device counts per user.

> [!danger] Common Mistake
> - Joining a Windows client via "Add a work or school account" in Settings instead of "Join this device to Microsoft Entra ID".
> - Deploying iOS ADE profiles without setting up the Apple Push Notification service (APNs) certificate first.
> - Setting up Android enrollment without linking an Android Enterprise Google account.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 11 virtual machine or physical client
> - An Intune tenant
> - A test user licensed for Microsoft Intune

### Step 1: Configure Windows MDM Automatic Enrollment in Intune Portal

```bash
# Navigate to the portal
Microsoft Intune Admin Center -> Devices -> Enrollment -> Automatic Enrollment
```

1. Ensure **MDM User Scope** is set to **All** (or a specific security group).
2. Ensure **MAM User Scope** is set to **None**.

### Step 2: Configure Enrollment Restrictions to Block Personal Windows Devices

1. Go to `Devices` -> `Enrollment` -> `Enrollment device platform restrictions`.
2. Click the **Windows restrictions** tab -> Click **Create restriction**.
3. Under **Platform settings**, toggle **Windows (MDM)** to **Allow**, but toggle **Personally owned devices** to **Block**.
4. Assign this restriction to your target user group.

### Step 3: Trigger Client-Side Windows Enrollment (User-Driven)

On the test Windows 11 machine:
1. Go to **Settings** -> **Accounts** -> **Access work or school**.
2. Click **Connect**.
3. Under "Alternate actions", click **Join this device to Microsoft Entra ID**.
4. Enter the test user credentials (`testuser@yourtenant.onmicrosoft.com`).
5. Confirm the organization details and click **Join**.
6. Click **Done** when complete.

> [!success] Expected Output
> Device appears in Intune as Corporate Windows device and is Compliant.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `dsregcmd /status` | Confirms Entra ID Join status (AzureAdJoined: YES) | `dsregcmd /status` |
| `Get-ScheduledTask ...` | Lists MDM synchronization schedules | `Get-ScheduledTask -TaskPath "\Microsoft\Windows\EnterpriseMgmt\"` |
| `deviceenroller.exe /c ...` | Trigger manual sync from the client | `Start-Process -FilePath "C:\Windows\System32\deviceenroller.exe" -ArgumentList "/c /mobiledeviceenrollmenttoday"` |

> [!tip] Pro Tip
> Renew your Apple APNs certificate on time every year. Set calendar reminders 30 days before expiration! Use ESP for Autopilot to block desktop access until security profiles install.

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| User deletes MDM profile on ADE device | ADE "Locked enrollment" set to "No" | Edit Apple Token Profile in Intune -> set **Locked enrollment** to **Yes**. Factory wipe device. |
| Error `0x80180018` during enrollment | MDM scope misconfigured or missing license | Check Entra ID -> Users -> Licenses. Check MDM User Scope in Mobility settings. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Accidental Profile Deletion

> [!example] Ticket
> "I accidentally removed the company management profile from my iPad and now I can't access email."

**L1 Response:** Verify the user's device serial number in Intune and confirm it was enrolled via ADE.
**Escalation Trigger:** Pass to L2 if the profile was originally unlocked and needs an architecture change.
**L2 Resolution:** Change ADE profile to "Locked enrollment: Yes", wipe device, and re-enroll automatically during setup.

---
## 🎤 Interview Questions

> [!question] Q1: What are the requirements for Windows MDM automatic enrollment?
> **Answer:** Requires Microsoft Entra ID integration, an active Intune license, and Windows 10/11. Configure in Entra ID Portal under Mobility (MDM and MAM) -> Microsoft Intune -> set MDM User Scope to "All" or a group.

> [!question] Q2: How configure shared Android tablets for inventory tracking?
> **Answer:** Use Android Enterprise Dedicated Device profile. Set up a Device Configuration profile in Single-App Kiosk Mode.

> [!question] Q3: Why must the APNs certificate be renewed using the same Apple ID?
> **Answer:** Using a different Apple ID generates a new certificate, which breaks the trust chain for all enrolled iOS/macOS devices and requires them to be factory reset and re-enrolled.

==**Exam Tip:** Joining via "Add a work or school account" instead of "Join this device to Microsoft Entra ID" registers the device as personal, which fails if personal devices are blocked.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes basic license structures and MDM authority.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Governs device health states post-enrollment.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-06 Windows Autopilot|INT-06 Windows Autopilot]] — Deep dive into zero-touch Windows deployments.
