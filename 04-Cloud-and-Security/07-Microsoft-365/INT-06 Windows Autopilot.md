---
tags: [sysadmin, intune, autopilot]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# INT-06: Windows Autopilot

> [!abstract] Overview
> This note details Windows Autopilot, a suite of technologies used to set up and pre-configure new devices. It covers deployment modes, hardware hash extraction, configuration profiles, the Enrollment Status Page (ESP), and troubleshooting strategies.

---
## Concept
Think of Windows Autopilot as a VIP airport check-in service for new laptops. Instead of the IT department taking every laptop out of its box, plugging in a USB drive to wipe the OS, installing drivers, and repackaging it (traditional imaging), you register the laptop's unique barcode (Hardware Hash) in the cloud. When the user receives the sealed box, opens it, and connects to Wi-Fi, the laptop recognizes its corporate identity, configures its settings, and downloads its applications automatically.
*Seedha simple mein: Autopilot ke zariye naye laptops ko bina manually image kiye, directly user ke ghar bheja ja sakta hai. Jaise hi user laptop on karke Wi-Fi se connect karega, company ke apps aur settings automatic deploy ho jayenge.*

---
## Technical Deep Dive

### 1. Autopilot Deployment Modes
- **User-Driven Mode**: Designed for standard deployments. The user turns on the device, selects their language/keyboard, connects to Wi-Fi, and enters their corporate credentials. The device then enrolls and configures.
- **Pre-Provisioning (White Glove)**: A two-stage deployment. A technician unboxes the device and boots to a diagnostic menu (pressing the Windows key 5 times) to pre-download apps and configurations. The device is then resealed and shipped to the user, who only has to perform a quick credential sign-in.
- **Self-Deploying Mode**: Designed for shared devices, kiosks, or digital signage. Joins the device to Entra ID and enrolls it in Intune automatically without requiring user credentials. Requires a physical TPM 2.0 chip.

### 2. Hardware Hash Registration
Every device enrolled in Autopilot must have its unique hardware signature (hardware hash) registered in the tenant. This is done by uploading a CSV file containing:
- Device Serial Number
- Windows Product ID
- Hardware Hash string

### 3. Autopilot Profiles & OOBE Settings
Autopilot Profiles define the Out-of-Box Experience (OOBE) for users:
- **Join type**: Azure AD Joined or Hybrid Azure AD Joined.
- **Privacy settings**: Hide or show privacy configuration screens.
- **User account type**: Standard (recommended for security) or Administrator.
- **Device naming template**: Auto-generate device names based on templates (e.g., `CN-%SERIAL%`).

### 4. Enrollment Status Page (ESP)
The ESP displays configuration progress during device setup. It can block access to the Windows desktop until specified critical applications and configuration profiles have fully installed, ensuring users do not access unconfigured systems.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows 11 virtual machine, access to the Microsoft Intune Admin Center, and a USB drive or shared folder to transfer files.

### Step 1: Extract the Hardware Hash on Client VM
On a clean Windows 11 virtual machine at the first OOBE setup screen (Press `Shift + F10` to open a command prompt):
1. Open PowerShell:
```cmd
powershell -ExecutionPolicy Bypass
```
2. Run the script to download the Autopilot tool and export the hash to a CSV:
```powershell
Install-Script -Name Get-WindowsAutopilotInfo -Force
Get-WindowsAutopilotInfo.ps1 -OutputFile C:\AutopilotHash.csv
```
3. Copy `AutopilotHash.csv` to a USB drive or location accessible from your admin host.

### Step 2: Upload Hardware Hash to Intune
1. Open the **Microsoft Intune Admin Center** -> `Devices` -> `Enrollment` -> `Windows Autopilot` -> `Devices`.
2. Click **Import** -> Select `AutopilotHash.csv` -> Click **Import**.
3. The import process can take up to 15 minutes. Wait until the device serial number appears in the list.

### Step 3: Create an Autopilot Deployment Profile
1. Under `Devices` -> `Enrollment` -> `Deployment Profiles` -> Click **Create profile** -> **Windows PC**.
2. Name: `Corp Autopilot User-Driven Profile`.
3. Out-of-box experience (OOBE) settings:
   - Deployment mode: **User-driven**.
   - Join to Azure AD as: **Azure AD joined**.
   - Microsoft Software License Terms: **Hide**.
   - Privacy settings: **Hide**.
   - User account type: **Standard**.
   - Apply device name template: **Yes** -> Enter `CORP-%SERIAL%`.
4. Click **Next** -> Assign the profile to a security group containing the Autopilot devices -> Click **Create**.

### Step 4: Configure the Enrollment Status Page (ESP)
1. Under `Devices` -> `Enrollment` -> `Enrollment Status Page`.
2. Click the default profile -> Edit properties.
3. Set **Show app and profile configuration progress** to **Yes**.
4. Set **Block device use until all apps and profiles are installed** to **Yes**.
5. Save the settings.
6. Boot the test virtual machine, connect it to the internet, and verify the customized corporate login screen appears.

---
## Commands Reference
```powershell
# Installs script used to collect and upload device hash in one step
Install-Script -Name Get-WindowsAutopilotInfo -Force
# Run script and upload hash directly to tenant (requires admin credentials)
Get-WindowsAutopilotInfo.ps1 -Online
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- Problem: The test virtual machine boots to the standard Windows OOBE setup screen (asking for personal Microsoft account login) instead of the corporate Contoso login screen during Autopilot testing.
- Root Cause: The device's hardware hash was not imported into Intune, or the Autopilot deployment profile has not finished assigning to the device.
- Fix:
  1. Open the **Intune Admin Center** -> `Devices` -> `Windows Autopilot` -> `Devices`.
  2. Verify the serial number is present and the status column displays **Assigned** with your profile name.
  3. If status is **Not assigned**, ensure the device belongs to the target security group for the profile.
  4. On the client machine, run `systemreset -cleanpc` to wipe the system and force it to re-evaluate the Autopilot check upon reboot.

**Scenario 2:**
- Problem: The Autopilot setup process gets stuck on the Enrollment Status Page (ESP) during the "Account setup" phase, eventually returning error `0x800705b4` (timeout).
- Root Cause: One of the applications targeted as "Required" during the ESP phase is failing to install or is waiting for user interaction silently.
- Fix:
  1. On the client machine at the failure screen, press `Ctrl + Shift + D` to open the **diagnostic tools window**.
  2. Export the logs to a USB drive or check the `IntuneManagementExtension.log` on the machine.
  3. Identify the failing application (search for `Installation failed` or exit codes like `1603`).
  4. Edit the ESP profile in the Intune portal to remove the failing app from the critical application block list until the installer is corrected.

---
## Common Mistakes
> [!warning] Avoid These
> Assigning Autopilot profiles to User groups instead of Device groups. Autopilot runs before the user logs in, so profiles must be targeted to device security groups.
> Setting the user account type in the Autopilot profile to Administrator for standard users. This grants users full local admin rights, bypassing security baselines.
> Testing Autopilot on virtual machines without allocating a virtual TPM 2.0 chip, causing Self-Deploying or Pre-Provisioning deployments to fail.

---
## Pro Tips
> [!tip] Field Experience
> Use dynamic device groups to automate Autopilot profile assignment. Create a group with a dynamic query that captures all imported Autopilot serials: `(device.devicePhysicalIDs -any _ -contains "[ZTDID]")`.
> When performing Pre-Provisioning (White Glove), ensure the network has an active DHCP server and does not require web proxy authentication, as the technician phase runs before user credentials can be input.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Hardware Hash | Unique identifier signature used to register devices in the Autopilot database. |
| 2 | User-Driven Mode | Standard deployment flow requiring users to log in with their corporate credentials. |
| 3 | Pre-Provisioning | Technician flow that pre-downloads apps before the device is shipped to the user. |
| 4 | ESP | Status page blocking access to the desktop until critical apps and settings install. |
| 5 | Dynamic Group | Group query matching `[ZTDID]` to automate Autopilot profile assignment. |

---
## Interview Q&A

**Q1: Explain the difference between User-Driven, Pre-Provisioning, and Self-Deploying modes in Windows Autopilot.**
A: User-Driven mode requires the end-user to select regional settings, connect to Wi-Fi, and log in with their credentials to initiate enrollment. Pre-Provisioning allows a technician to pre-stage applications and policies on the device before shipping it to the user, who then only performs a quick login. Self-Deploying mode is designed for shared or kiosk devices; it uses the device's hardware TPM 2.0 chip to auto-join Entra ID and enroll in Intune without requiring any user credentials.

**Q2: An administrator needs to import 500 new laptops into the Autopilot database. What is the best way to accomplish this?**
A:
- **Situation**: The company purchased 500 laptops that needed to be imported into the Autopilot database before delivery to users.
- **Task**: I needed to register the devices without manually booting each one to export the hash.
- **Action**: I contacted the OEM reseller and requested they register the hardware hashes directly. I provided them with our tenant ID and consented to authorization in our Partner Center portal.
- **Result**: The OEM uploaded the hardware hashes directly to our tenant's Autopilot database. The devices were registered before they arrived at the users' homes, saving significant IT labor.

**Q3: How do you access the diagnostic tools window on a client machine if the Autopilot enrollment fails, and what can you find there?**
A: If Autopilot fails on the OOBE or ESP screen, you can press `Ctrl + Shift + D` (or `Shift + F10` and run `explorer.exe`). This opens the diagnostic screen, allowing you to export MDM log files to a USB drive. These logs contain the registry configurations, event logs, and the `IntuneManagementExtension.log` files, which help identify the failing policy, application, or network connection.

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Sets up the tenant MDM authority and licensing.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-02 Device Enrollment|INT-02 Device Enrollment]] — Details platform enrollment profiles.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-05 Application Management|INT-05 Application Management]] — Details Win32 packaging used in the ESP phase.

