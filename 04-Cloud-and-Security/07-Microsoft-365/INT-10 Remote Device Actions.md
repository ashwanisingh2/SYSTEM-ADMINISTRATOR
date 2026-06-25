---
tags: [sysadmin, intune, operations]
difficulty: Intermediate
lab-required: Yes
read-time: 12 mins
---

# INT-10: Remote Device Actions

> [!abstract] Overview
> This note covers remote device actions in Microsoft Intune, detailing administrative commands (Sync, Restart, Remote Lock), data wiping states (Fresh Start, Wipe, Retire), diagnostic collection, and remote assistance options.

---
## Concept
Think of remote device actions as a remote management system for a fleet of vehicles.
- **Sync** is like sending a radio signal checking if the vehicle's onboard computer has received new route updates.
- **Retire** is like peeling the corporate magnetic logo off a delivery driver's personal car. The car remains theirs, but they can no longer use it to enter secure corporate loading zones.
- **Wipe** is like activating the vehicle's self-destruct mechanism, erasing all data on the computer and resetting the ignition to factory settings.
- **Fresh Start** is like performing a clean engine swap, removing all aftermarket modifications (bloatware) and leaving only the factory standard setup.
*Seedha simple mein: Remote actions ke zariye aap devices ko cloud console se control kar sakte hain, jaise phone/laptop locks, diagnostic reports retrieve karna, aur device data wipe karna.*

---
## Technical Deep Dive

### 1. Administrative Actions
- **Sync**: Forces the device to check in with Intune immediately and pull down pending policies and apps.
- **Restart**: Initiates an OS reboot. The user receives a notification warning before the system restarts.
- **Remote Lock**: Instantly locks the device screen. The user must input their passcode to unlock it.
- **Reset Passcode**: Generates a new temporary passcode or removes the lock screen passcode entirely (applicable to iOS and Android).

### 2. Device Clean Wipes: Key Differences
| Action | Target Platform | User Data | Company Data | Pre-installed Apps / Bloatware | Description |
|--------|-----------------|-----------|--------------|--------------------------------|-------------|
| **Retire (Selective Wipe)** | All | Preserved | Wiped | Preserved | Removes corporate policies, certificates, and Intune-managed applications. Leaves personal photos and files intact. Best for BYOD. |
| **Wipe (Factory Reset)** | All | Wiped | Wiped | Restored to default | Performs a complete factory reset, erasing all partitions and user data. Best for decommissioning corporate hardware. |
| **Fresh Start** | Windows | Wiped or Preserved | Wiped | Removed (Clean OS) | Installs a clean version of Windows, removing OEM-installed bloatware and trials. Upgrades the device to the latest Windows version. |
| **Autopilot Reset** | Windows | Wiped | Preserved (AD status) | Preserved | Wipes local data, settings, and apps, but retains the Entra ID join status and enrollment configurations, preparing the device for the next user. |

### 3. Diagnostics & Support
- **Collect Diagnostics**: Instructs the client device to package its MDM logs, registry settings, and event logs into a ZIP file and upload it to the Intune console.
- **Remote Assistance**: Integrates Intune with support tools like **Microsoft Remote Help** or **TeamViewer**, allowing helpdesk technicians to view or control the user's screen during support sessions.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Access to Microsoft Intune Admin Center with Intune Administrator permissions. Enrolled test Windows 11 client device.

### Step 1: Trigger a Remote Device Sync
1. Open the **Microsoft Intune Admin Center** -> `Devices` -> `All devices`.
2. Select your test Windows 11 device.
3. Click the **Sync** button on the top action bar.
4. Click **Yes** to confirm.
5. Verify the synchronization status by checking the **Last contact** timestamp under device properties after a few minutes.

### Step 2: Collect Remote Diagnostics from the Client
Retrieve system logs remotely:
1. Under the selected device properties, click the **Collect diagnostics** button.
2. Click **Yes** to confirm.
3. Monitor progress in the **Device diagnostics** status blade.
4. Once completed, click the **Download diagnostics** button to download the ZIP file containing system logs (`C:\ProgramData\Microsoft\IntuneManagementExtension\Logs`, Event Viewer logs, and registry exports) to your admin workstation.

### Step 3: Trigger a Selective Wipe (Retire) on a BYOD Device
Simulate decommissioning a personal device:
1. Select a test device enrolled as Personally Owned.
2. Click the **Retire** button on the top action bar.
3. Click **Yes** to confirm.
4. Verify on the client machine that the corporate account has been removed from **Access work or school**, and corporate apps (like Outlook) prompt for enrollment again.

### Step 4: Perform a Remote Lock
1. Select your test device.
2. Click the **Remote Lock** button.
3. Confirm the action.
4. Observe the client machine lock instantly, requiring the user passcode to unlock.

---
## Commands Reference
```powershell
# Run on Windows client to force manual sync to pull down remote actions
C:\Windows\System32\deviceenroller.exe /c /mobiledeviceenrollmenttoday
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- Problem: An administrator triggers a "Wipe" command on a lost corporate laptop, but the device status remains "Wipe Pending" indefinitely, and the device is not wiped.
- Root Cause: The device is turned off or disconnected from the internet, preventing it from receiving the push notification command from Intune.
- Fix:
  1. The wipe command remains active until the device connects to the internet. Do not delete the device object from Intune or Entra ID, as this destroys the management trust link and prevents the device from wiping if it turns on later.
  2. If the device has been stolen and is unlikely to connect to the internet, locate the device in the **Microsoft Entra portal** and disable the device object to block access to corporate resources.

**Scenario 2:**
- Problem: A technician triggers a "Fresh Start" command on a computer with a corrupt operating system, but the process fails with error `0x80070002` (File not found).
- Root Cause: The local Windows Recovery Environment (WinRE) partition is missing, disabled, or corrupted. Fresh Start relies on WinRE files to rebuild the operating system.
- Fix:
  1. Log onto the client machine and open a command prompt as Administrator.
  2. Run `reagentc /info` to check the status of the recovery partition.
  3. If disabled, run `reagentc /enable` to activate the recovery partition.
  4. If the partition is missing, reinstall Windows manually using bootable media or rebuild the recovery partition before triggering the remote action.

---
## Common Mistakes
> [!warning] Avoid These
> Clicking **Wipe** on a user's personal (BYOD) mobile phone when they leave the organization. This triggers a complete factory reset, erasing their personal photos and files and creating liability issues. Use **Retire** instead.
> Deleting the device object from the Intune console before triggering a Wipe or Retire command. This breaks the management link and prevents the device from executing the command.
> Running Fresh Start on devices containing proprietary legacy hardware drivers. This replaces custom drivers with generic Microsoft drivers, potentially disabling key components (like custom trackpads or graphics cards).

---
## Pro Tips
> [!tip] Field Experience
> Use the **Autopilot Reset** command when recycling laptops within the same department. It resets Windows to a clean state while retaining the Entra ID join status and Autopilot configurations, saving time compared to a full Wipe.
> When decommissioning corporate hardware, trigger the **Wipe** action and select **Clean device, and continue to wipe even if device loses power**. This performs a secure data write across all sectors of the drive, preventing data recovery.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Retire | Selective wipe that removes corporate data while leaving personal files intact. |
| 2 | Wipe | Complete factory reset that erases all partitions and user data on the device. |
| 3 | Fresh Start | Clean OS install that removes OEM bloatware and upgrades to the latest Windows version. |
| 4 | Autopilot Reset | Wipes local user profiles and apps, but retains Entra ID join status for reuse. |
| 5 | Sync | Direct command that forces a client device to check in and pull down pending policies. |

---
## Interview Q&A

**Q1: Explain the difference between the Retire and Wipe remote actions. When would you use each?**
A: **Retire** performs a selective wipe, removing corporate configurations, certificates, emails, and managed applications, while leaving personal files and applications intact. It is used for personal/BYOD devices. **Wipe** performs a full factory reset, erasing all user data, applications, and partitions. It is used when decommissioning or repurposing corporate-owned hardware, or if a device is lost or stolen.

**Q2: A user reports their corporate laptop was stolen. Explain the step-by-step process to secure the corporate data on that device.**
A:
- **Situation**: A user's corporate laptop containing sensitive financial files was stolen.
- **Task**: I needed to secure the data and prevent unauthorized access.
- **Action**: I opened the Intune portal, located the device, and immediately triggered a **Wipe** command, selecting the option to secure-wipe the drive. Next, I went to the Microsoft Entra portal and **disabled** the device object. I then reset the user's Entra ID password and revoked all active refresh tokens to block access from other locations.
- **Result**: If the laptop is turned on and connects to the internet, it will execute the wipe command. If it remains offline, the disabled Entra ID status blocks authentication attempts and access to cloud resources, while BitLocker protects the local data on the drive.

**Q3: What are the requirements for the "Fresh Start" remote action to succeed on a Windows device?**
A: The "Fresh Start" action requires the client machine to run Windows 10/11 (Pro, Enterprise, or Education), have an active Intune management connection, and possess a functional and enabled local Windows Recovery Environment (WinRE) partition. If WinRE is missing or disabled, the Fresh Start action will fail.

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes identity joins and MDM authority.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-02 Device Enrollment|INT-02 Device Enrollment]] — Details device onboarding flows.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-05 Application Management|INT-05 Application Management]] — Details application deployment profiles.

