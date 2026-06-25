---
tags: [desktop-support, career-roadmap, certification, microsoft, intune, autopilot, md-102, L1, L2, L3]
aliases: [md-102-prep, endpoint-administrator-roadmap]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

# MD-102: Endpoint Administrator Certification Study Roadmap

---

## Concept Overview
- **What it is**: A structured study guide, technical checklist, and practice laboratory framework for passing the Microsoft Endpoint Administrator (MD-102) certification exam.
- **Why it matters**: MD-102 is the ultimate certification for modern desktop administrators. It validates skills in deploying Windows clients via Autopilot, managing compliance policies, building Intune configuration profiles, and packaging enterprise applications.
- **Where you encounter this in real job**: Provisioning new corporate laptops, troubleshooting BitLocker compliance blocks, push deploying software silently to remote users, and managing Windows Update rings.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Runs Autopilot hardware hash collection scripts, adds users to group deployments, triggers remote device syncs, and guides users through initial logins.
  - **L2**: Builds configuration profiles (VPN/Wi-Fi), packages Win32 apps, configures Windows Update Rings, and resolves enrollment failures.
  - **L3**: Architectures tenant-wide compliance baselines, implements conditional access policies, sets up Windows Autopilot deployment profiles, and structures Co-Management with SCCM.

---

## Technical Deep Dive

### MD-102 Exam Domain Weightings
The exam evaluates candidates across four critical modern management domains:
```
  [1. Deploy Windows Client (25-30%)]      --->  [2. Manage Identity & Compliance (15-20%)]
                                                   |
                                                   v
  [3. Manage, Maintain, Protect Devices (40-45%)] --->  [4. Manage Applications (10-15%)]
```

### Windows Autopilot Flow
Autopilot allows the provisioning of new devices without maintaining custom operating system images:
```
[OEM / Vendor] ---> (Upload Hardware Hash to Tenant) ---> [User Unboxes Device]
                                                                  |
                                                                  v (Connects to Wi-Fi)
[Device Enrolls in MDM] <--- (Applies Policies & Profiles) <--- [User Inputs M365 Email]
```
- **Hardware Hash**: A unique 4K hardware identifier collected from the device BIOS and uploaded to the company's Microsoft tenant to register the machine.
- **OOBE (Out-Of-Box Experience)**: The initial setup screen on Windows. Autopilot customizes this, skipping license terms and keyboard options, forcing the user to log in with their corporate email.

---

## Commands & Syntax

### PowerShell: MD-102 Administrative Operations
```powershell
# 1. Collect and export the local machine's Autopilot hardware hash to CSV
Install-Script -Name Get-WindowsAutopilotInfo -Force
Get-WindowsAutopilotInfo.ps1 -OutputFile "C:\temp\AutopilotHash.csv"

# 2. Check the local machine's Microsoft Entra ID join and MDM status
dsregcmd /status

# 3. Query the Microsoft Graph Intune module for device enrollment status
Connect-MgGraph -Scopes "DeviceManagementManagedDevices.Read.All"
Get-MgDeviceManagementManagedDevice | Select-Object DeviceName, OperatingSystem, ComplianceState
```

### CMD / Run Box
```cmd
REM Force an immediate local policy sync with Microsoft Intune
deviceenroller.exe /c /urlelink
```

### GUI Path
> **Intune Portal (endpoint.microsoft.com)**: Devices -> Windows -> Windows enrollment -> Devices (for Autopilot registration)
> **Intune Portal**: Devices -> Compliance policies -> Create Policy (to define security baseline standards)

---

## Real-World Scenarios

### Scenario 1: Autopilot Setup Hangs during Device Preparation
**User Complaint**: A remote user is provisioning their new laptop via Autopilot, but the setup hangs at the "Device Preparation" stage, showing the error code: `0x80070002` (Hardware registration failure).
**Your First 3 Checks**:
1. Check if the device hardware hash was successfully uploaded to the Intune portal.
2. Verify that an Autopilot Deployment Profile is assigned to the device object.
3. Check the device's internet connection security (e.g. SSL inspection blocks).
**Diagnosis Steps**:
1. Open the Intune portal and check Devices -> Windows enrollment -> Devices.
2. The device serial number shows up, but the Profile Status column is marked as: **Not Assigned**.
3. Check the dynamic security group: `DG-Autopilot-Devices` used to target deployment profiles.
**Root Cause**: The dynamic security group took too long to populate the newly uploaded device hash, causing the machine to boot without an assigned Autopilot profile.
**Fix**:
1. In the Intune portal, manually assign the Autopilot Profile to the device or trigger a group member update in Entra ID.
2. Confirm the Profile Status in Intune changes to: **Assigned**.
3. On the client machine, restart the PC and begin the OOBE setup again.
**Prevention**: Upload device hardware hashes at least 2 hours before handing them to users to allow Entra ID dynamic groups to process membership.

### Scenario 2: Laptop Blocked by Conditional Access (Compliance Failure)
**User Complaint**: A user cannot access their corporate email on their company-issued laptop. They receive an error stating: "You cannot access this resource because your device does not comply with your organization's security policies."
**Your First 3 Checks**:
1. Check the device's compliance state in the Microsoft Intune console.
2. Verify which compliance rule is marked as "Not Compliant".
3. Check the BitLocker encryption status on the client's local drive.
**Diagnosis Steps**:
1. Log into `endpoint.microsoft.com` -> Devices -> search the user's device name.
2. The compliance status shows: **Not Compliant**.
3. Click Device compliance -> select active policy -> Policy settings list.
4. The rule **"Require BitLocker encryption"** is marked as: **Failed**.
5. Check client PC: BitLocker is disabled on the C: drive.
**Root Cause**: The user plugged in a personal USB drive that interrupted the auto-encryption task, or the TPM chip failed to initialize, preventing BitLocker from active.
**Fix**:
1. Log in to the client PC as administrator and run: `manage-bde -status C:`.
2. Encrypt the drive manually:
   ```cmd
   manage-bde -on C: -RecoveryPassword
   ```
3. Open Settings -> Accounts -> Access work or school -> Click your account -> Info -> Click **Sync**.
4. Once synced, Intune updates the device state to "Compliant" and access to Outlook is restored.
**Prevention**: Enforce silent BitLocker encryption through configuration profiles that apply automatically without user interaction.

---

## Critical Points

> [!danger] Never Do This
> Do not register a personal device as an autopilot-managed system. Autopilot registration binds the device permanently to the corporate tenant. Releasing the device requires deleting the Autopilot record from the Intune portal first.

> [!warning] Common Trap
> Assuming a configuration profile applies immediately. MDM policy refreshes can take up to 8 hours for active clients. For immediate testing, force a manual sync through the local Settings app or via the Intune Portal remote actions.

> [!tip] Senior Engineer Tip
> Use **Intune Win32 App Packaging tool** (`IntuneWinAppUtil.exe`) instead of raw `.msi` or `.exe` installers. Win32 app wrappers allow you to configure custom detection rules, install requirements, and return codes, ensuring clean software rollouts.

> [!success] Verification Steps
> To verify MDM registration:
> 1. Open Command Prompt on the client PC.
> 2. Run `dsregcmd /status`.
> 3. Verify `AzureAdJoined` is `YES` and `MdmUrl` points to Microsoft Intune endpoints.

> [!question] Interview Alert
> "What is the difference between a Configuration Profile and a Compliance Policy in Intune?"
> - **Answer**: Configuration Profiles are used to configure device settings (e.g. Wi-Fi profiles, VPNs, desktop backgrounds, or registry keys). Compliance Policies are used to evaluate if a device meets specific security criteria (e.g. minimum OS version, BitLocker enabled, or firewall active) and passes this state to Conditional Access to grant or block resource access.

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| App install failures | Incorrect Detection Rules in Win32 package | Use precise File, Registry, or Registry product code detection paths in configuration. |
| Autopilot profile bypass | Device booted without network access | Force network connection on the initial OOBE screen (`Shift + F10` -> `control netconnections`). |
| Windows updates block | Conflicting policies between GPO and Intune | Configure MDM Wins over GPO settings using custom URI policies (`./Device/Vendor/MSFT/Policy/Config/ControlPolicyConflict/MDMWinsOverGP`). |

---

## Lab Exercise

**Objective**: Package and deploy a software application (e.g. 7-Zip) as a Win32 App via Microsoft Intune.
**Time Required**: 30 minutes
**Environment Needed**: Windows 11 Workstation and Intune Admin Portal Access.

**Steps**:
1. Create a working directory: `C:\IntuneTemp` and download the 7-Zip `.msi` file.
2. Download the Microsoft Win32 Content Prep Tool (`IntuneWinAppUtil.exe`).
3. Package the app. Open CMD and run:
   ```cmd
   IntuneWinAppUtil.exe -c C:\IntuneTemp -s 7z2301-x64.msi -o C:\IntuneTemp\output
   ```
   - This creates `7z2301-x64.intunewin` in the output folder.
4. Log into the Intune Admin Center (`endpoint.microsoft.com`).
5. Navigate to Apps -> All apps -> Add -> Select **Windows app (Win32)**.
6. Upload the `.intunewin` file.
7. Configure App Information and Program details (install commands are auto-populated from MSI).
8. Configure Detection Rules: Select Rule type: **MSI** (product code is auto-detected).
9. Assign the app to a test group and verify installation on client machines.

**Pass Criteria**: The app is successfully packaged, uploaded, and installs silently on client machines.

---

## Interview Questions & Answers

### L1 Level

#### Q1: How do you trigger a manual sync on a Windows 10/11 machine managed by Intune?
**A**: I guide the user to go to Settings -> Accounts -> Access work or school. Click on their corporate account, select Info, scroll down to the Device sync status section, and click the Sync button. This pulls the latest policies from Intune immediately.

#### Q2: What is the "Company Portal" app and why do users need it?
**A**: The Company Portal is a self-service application repository provided by Microsoft Intune. It allows users to install approved business applications, check their device's compliance state, and locate contact details for the IT support desk.

---

### L2 Level

#### Q3: Explain how you would deploy a custom Registry key using Intune.
**A**: I can deploy a registry key using two methods:
1. Write a PowerShell script that checks and writes the registry key using `New-ItemProperty` and upload it to Intune under Devices -> Scripts.
2. Create a custom OMA-URI policy under Devices -> Configuration profiles using the Registry CSP path.

#### Q4: A user reports that Windows Update rebooted their machine during working hours, closing unsaved files. How do you prevent this?
**A**: I configure **Windows Update Rings** in Intune. I configure the "Active Hours" window (e.g., 8:00 AM to 5:00 PM) during which reboots are blocked. I also configure a restart grace period (e.g. 2 days) to allow users to schedule reboots at a convenient time.

---

### L3 Level

#### Q5: Explain how you would troubleshoot an application installation failure through Intune.
**A**:
- **Situation**: A critical line-of-business application failed to deploy to 100 remote workstations, returning generic exit codes.
- **Task**: Locate the root cause of the installation failure and deploy a fix.
- **Action**: I logged into a failed client machine and checked the MDM client logs stored in `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log`. I searched for the application ID and found the installer failed because the target directory was locked by another service. I updated the install script to terminate the conflicting service before running the setup.
- **Result**: Updated the Win32 application package in Intune. The app deployed successfully, reducing installation errors to zero.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **Autopilot**: Deploys Windows clients without local imaging. Requires uploading the Hardware Hash.
> **Intune Agent**: The Intune Management Extension (IME) runs as a service to manage Win32 apps and PowerShell scripts.
> **Sync Command**: `deviceenroller.exe /c /urlelink`.
> **Compliance**: Evaluates device settings. Failed compliance blocks access via Conditional Access.

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]]
- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]]
- [[00-MOC/Master-Index|Master Index]]

---

## Tags
#desktop-support #intune-administration #windows-autopilot #endpoint-security #compliance-policy #app-packaging #certification-roadmap #L2 #L3

