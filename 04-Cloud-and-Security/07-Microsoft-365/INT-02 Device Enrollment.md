---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-02-device-enrollment, int-02]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# INT-02: Device Enrollment

> [!abstract] Overview
> This note covers the enrollment protocols for bringing Windows, Android, iOS/iPadOS, and macOS devices into Microsoft Intune. It explains platform-specific flows, enrollment restrictions, and troubleshooting procedures.

---

---
## Concept Overview
Think of device enrollment as onboarding a new employee at a high-security office.
- **Windows Autopilot** is like ordering a custom-tagged briefcase directly from the factory. The employee receives it, opens it, connects to Wi-Fi, and the lock combination and layouts automatically apply.
- **Android Enterprise Work Profile** is like dividing a personal backpack into two compartments: one side has a padlock for company documents, and the other side is open for personal snacks and keys.
- **Apple Business Manager (ADE)** is like registering a company-owned phone directly with Apple so that the moment it turns on, it immediately checks in with your company's servers and configures itself.

---

---
## Technical Deep Dive
### 1. Windows Enrollment Methods
- **Automatic MDM Enrollment**: Triggered when a device joins Microsoft Entra ID. The user enters their credentials during Windows Out-of-Box Experience (OOBE), and the device auto-registers in Intune.
- **Windows Autopilot**: A cloud-based service that customizes the OOBE. It allows IT departments to ship pre-configured devices directly to end-users without imaging.
- **GPO-based Enrollment**: Used for domain-joined machines. A Group Policy Object initiates background enrollment into Intune using the device's system credentials.

### 2. Android Enrollment Profiles
- **Android Enterprise Work Profile (BYOD)**: Splits the device into personal and work profiles. Data cannot cross the profile boundaries.
- **Fully Managed (COBO - Corporate Owned, Business Only)**: The entire device is managed. Users cannot install personal apps.
- **Dedicated Devices (COSU - Corporate Owned, Single Use)**: Kiosk mode (e.g., ticket printing or inventory scanners).
- **Corporate-Owned with Work Profile (COPE)**: Corporate owned, but permits personal use in a split space.

### 3. iOS/iPadOS & macOS Enrollment
- **Automated Device Enrollment (ADE)**: Integrates Intune with Apple Business Manager (ABM). When devices boot for the first time, they force-enroll in Intune. This prevents users from removing the MDM profile.
- **User Enrollment (BYOD)**: Protects corporate data using a managed Apple ID, keeping personal and work data separate.
- **Direct Enrollment**: Admin uses Apple Configurator to manually push the enrollment profile.

### 4. Enrollment Restrictions
Allows blocking specific platforms (e.g., block personal macOS devices), enforcing OS version minimums, and setting maximum device counts per user.

---

## Common Mistakes
> [!warning] Avoid These
> Joining a Windows client via "Add a work or school account" in Settings instead of "Join this device to Microsoft Entra ID". The former registers the device as a personal (BYOD) device, which blocks corporate policies if personal device enrollment is disabled.
> Deploying iOS ADE profiles without setting up the Apple Push Notification service (APNs) certificate first. Intune cannot communicate with Apple devices without an active APNs certificate.
> Setting up Android enrollment without linking an Android Enterprise Google account, preventing Android profiles from deploying.

---

## Pro Tips
> [!tip] Field Experience
> Renew your Apple APNs certificate on time every year. If you let it expire, you must re-enroll all your organization's iOS and macOS devices from scratch, as the token keys will break. Set calendar reminders 30 days before expiration.
> Use the **Enrollment Status Page (ESP)** for Autopilot. This blocks the Windows desktop access until the critical apps and security profiles have fully installed, preventing users from logging into unconfigured systems.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> A Windows 11 virtual machine or physical client, an Intune tenant, and a test user licensed for Microsoft Intune.

### Step 1: Configure Windows MDM Automatic Enrollment in Intune Portal
1. Open the **Microsoft Intune Admin Center** -> `Devices` -> `Enrollment` -> `Automatic Enrollment`.
2. Ensure **MDM User Scope** is set to **All** (or a specific security group).
3. Ensure **MAM User Scope** is set to **None**.

### Step 2: Configure Enrollment Restrictions to Block Personal Windows Devices
Prevent users from enrolling personal Windows machines:
1. Go to `Devices` -> `Enrollment` -> `Enrollment device platform restrictions`.
2. Click the **Windows restrictions** tab -> Click **Create restriction**.
3. Under **Platform settings**, toggle **Windows (MDM)** to **Allow**, but toggle **Personally owned devices** to **Block**.
4. Assign this restriction to your target user group.

### Step 3: Trigger Client-Side Windows Enrollment (User-Driven)
On the test Windows 11 machine:
1. Go to **Settings** -> **Accounts** -> **Access work or school**.
2. Click **Connect**.
3. Under "Alternate actions", click **Join this device to Microsoft Entra ID**. (Do not enter email in the main box, as that results in a personal register).
4. Enter the test user credentials (`testuser@yourtenant.onmicrosoft.com`).
5. Confirm the organization details and click **Join**.
6. Click **Done** when complete.

### Step 4: Verify Device Enrollment in Intune
1. Back in the **Intune Admin Center**, navigate to `Devices` -> `All devices`.
2. Search for the computer name.
3. Verify that the **OS** reads `Windows`, **Ownership** reads `Corporate`, and **Compliance** status displays `Compliant`.

---

---
## Cheat Sheet / Quick Reference
```powershell
dsregcmd /status                                # Confirms Entra ID Join status (AzureAdJoined: YES)
Get-ScheduledTask -TaskPath "\Microsoft\Windows\EnterpriseMgmt\" # Lists MDM synchronization schedules
# Trigger manual sync from the client
Start-Process -FilePath "C:\Windows\System32\deviceenroller.exe" -ArgumentList "/c /mobiledeviceenrollmenttoday"
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Autopilot | Automated cloud-driven Windows provisioning direct to end-users. |
| 2 | Work Profile | Android container separating corporate apps and personal data on a single phone. |
| 3 | ABM (ADE) | Apple portal linking hardware purchases directly to Intune for forced enrollment. |
| 4 | APNs Certificate | Apple encryption token required for Intune to communicate with iOS/macOS. |
| 5 | Device Restrictions | Policy defining which platforms and ownership types can enroll in the tenant. |

---

---
## Troubleshooting
**Scenario 1:**
- Problem: An iOS device is enrolled using Apple Business Manager, but the user is able to delete the MDM profile from the settings app.
- Root Cause: The enrollment profile assigned in Apple Business Manager had the "Locked enrollment" configuration set to "No".
- Fix:
  1. Open the **Intune Admin Center** -> `Devices` -> `iOS/iPadOS` -> `iOS enrollment` -> `Enrollment program tokens`.
  2. Select your Apple token -> Click `Profiles` -> Select the active profile.
  3. Edit properties and toggle **Locked enrollment** to **Yes**.
  4. Save the profile. Note: The iOS device must be factory reset (wiped) for this new locked setting to apply.

**Scenario 2:**
- Problem: Windows device enrollment fails during "Access work or school" connection, returning error `0x80180018`.
- Root Cause: The MDM user scope is misconfigured in Entra ID, or the user's Intune license is missing or expired.
- Fix:
  1. Open the **Microsoft Entra portal** -> `Identity` -> `Users` -> Select the target user -> `Licenses`.
  2. Verify that **Microsoft Intune** or **M365 Business Premium** is checked and active.
  3. Navigate to `Mobility (MDM and MAM)` -> `Microsoft Intune` and verify that the user belongs to the group allowed for auto-enrollment.

---

---
## Interview Questions
**Q1: What are the requirements for Windows MDM automatic enrollment, and how would you configure it?**
A: Windows MDM auto-enrollment requires Microsoft Entra ID integration, an active Intune license for the enrolling user, and the client machine running Windows 10/11 (Pro, Enterprise, or Education). Configuration is done by logging into the Entra ID Portal under Mobility (MDM and MAM) -> Microsoft Intune, and setting the MDM User Scope to "All" or a selected group, while setting the MAM User Scope to "None".

**Q2: A client reports that their shipping agents use shared Android tablets that must only run one inventory tracking app. How would you configure these?**
A:
- **Situation**: The client needed 50 warehouse tablets configured as dedicated single-purpose devices.
- **Task**: I needed to set up an Android Enterprise Kiosk enrollment profile in Intune.
- **Action**: I linked the tenant to a managed Google Play account, then created an **Android Enterprise Dedicated Device** enrollment profile. Next, I set up a Device Configuration profile configured in **Single-App Kiosk Mode**, targeting the inventory application.
- **Result**: The tablets auto-configured upon scanning a QR code, locking the screen to the inventory app and disabling access to the home screen, settings, and other apps.

**Q3: Why must the APNs certificate be renewed using the same Apple ID every year, and what happens if you use a different ID?**
A: The APNs certificate creates a secure trust link between Intune and Apple's push servers. It must be renewed using the same Apple ID used to create it. If a different Apple ID is used, a new certificate is generated instead of renewing the old one. This breaks the trust chain for all enrolled iOS/macOS devices, requiring them to be factory reset and re-enrolled to restore connection to Intune.

---

---
## Seedha Simple Mein
*Seedha simple mein: Device Enrollment woh step hai jis ke zariye target hardware (Windows, iOS, Android, macOS) ko Intune key credentials aur server configurations ke sath bind kiya jata hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes basic license structures and MDM authority.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Governs device health states post-enrollment.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-06 Windows Autopilot|INT-06 Windows Autopilot]] — Deep dive into zero-touch Windows deployments.
