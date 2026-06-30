---
tags: [intune, troubleshooting, mdm, mam, windows, m365]
aliases: [intune-troubleshooting, int-19]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#advanced` `#md-102`

# INT-19: Intune Troubleshooting

> [!abstract] Overview
> Intune troubleshooting involves diagnosing and fixing issues related to device enrollment, app deployment, compliance policies, and configuration profiles. As a senior engineer, you must know how to quickly isolate whether the problem is on the client side, the network, or the Intune service itself. *Ek L1/L2 engineer ko aana chahiye ki logs kahan se nikale, sync issues kaise fix kare, aur Company Portal errors ko kaise samjhe taaki end-user ki device jaldi secure aur productive ho.*

---
## 🧠 Concept Overview

- **What it is** — The systematic process of identifying and resolving issues within Microsoft Intune using local device logs, the Intune portal status reports, Event Viewer, and diagnostic tools.
- **Why it matters** — *Agar device sync nahi hogi ya policy apply nahi hogi, toh company ka data secure nahi rahega. Troubleshooting skills downtime kam karti hain aur security posture maintain rakhti hain.*
- **Where you see this** — When users complain "Apps are not installing from Company Portal", "My device says it is not compliant so I cannot access email", or "I cannot enroll my new laptop".

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Ask user to sync device manually from Company Portal or Settings, check basic internet connection, verify Azure AD login status. |
| **L2** | Collect diagnostic logs from `ProgramData`, check Event Viewer (`DeviceManagement-Enterprise-Diagnostics-Provider`), analyze policy conflicts in Intune Portal. |
| **L3** | Architecture-level troubleshooting, certificate infrastructure issues, Autopilot deep dive, Network traffic analysis, and Microsoft Support escalation. |

> [!tip] Seedha Simple Mein
> *Intune troubleshooting ka matlab hai pata lagana ki cloud se laptop/mobile tak commands kyun nahi pahunch rahi hain. Logs aur sync status sabse best dost hote hain isme.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Intune Troubleshooting** is like **finding a missing courier delivery** because...
>
> - The Intune server is the *courier hub* sending policies, apps, and scripts (the packages).
> - The user's device is the *customer's house* receiving them.
> - If the package doesn't arrive, you don't just guess. You check the tracking number (Sync Status in Portal), ask the local delivery boy (Event Logs on the device), and ensure the delivery address is correct (Device Enrollment status and Azure AD join state).

---
## 🔬 Technical Deep Dive

### 1. The Synchronization Process (Sync)

> [!info] Key Concept
> Intune Management Extension (IME) is the primary agent responsible for executing PowerShell scripts, Win32 apps, and Proactive Remediations on Windows devices.

When a device syncs, it checks in with the Intune service to receive new policies and report its current state. *Agar device check-in fail hota hai, toh nayi policies push nahi ho payengi.* 

The synchronization happens over standard HTTPS (Port 443). Key URLs must be accessible, such as `enrollment.manage.microsoft.com`. The built-in MDM agent handles basic Configuration Service Provider (CSP) policies, while the IME handles heavier tasks like app installations.

### 2. Crucial Log Locations

> [!info] Key Concept
> Logs provide the absolute truth about why a specific action failed. *Hawa mein teer maarne se acha hai logs dekho, issue jaldi resolve hoga.*

- **Intune Management Extension Logs**: Located at `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs`
  - `IntuneManagementExtension.log`: Core log for Win32 apps. Tracks download progress, unzipping, detection rules, and installation exit codes. *Ye sabse important log file hai Win32 app failures ke liye.*
  - `AgentExecutor.log`: Logs related to PowerShell script execution.
  - `ClientHealth.log`: Tracks the health of the IME service itself.
- **MDM Event Logs**: Located in `Event Viewer -> Applications and Services Logs -> Microsoft -> Windows -> DeviceManagement-Enterprise-Diagnostics-Provider -> Admin`
  - Event ID 814: String policy applied successfully.
  - Event ID 404: MDM configuration failed. *Yahan policy apply na hone ke detailed error codes milte hain.*
- **Autopilot Logs**: Collected using `mdmdiagnosticstool.exe`.

### 3. Understanding Policy Conflicts

> [!danger] Common Mistake
> Creating overlapping and conflicting policies across different groups. *Ek policy bolti hai camera disable karo, doosri bolti hai enable karo. Aise mein conflict ho jayega aur Intune console mein 'Conflict' status aayega.*

If a setting is configured differently in two profiles assigned to the same user or device, Intune resolves the conflict based on a specific hierarchy. For example, Compliance policies generally override Configuration policies. However, it often just reports a conflict, requiring you to manually adjust the assignments or create exclusion groups.

### 4. Company Portal Troubleshooting

> [!info] Key Concept
> The Company Portal is just a UI that talks to Intune. If an app is missing, it's usually an assignment issue or a sync issue.

If the Company Portal app is stuck or not showing apps:
- Reset the Company Portal app from Windows Settings > Apps > Installed Apps > Advanced Options > Reset.
- Verify the user is actually in the Azure AD group that has the app assigned as "Available". *Assign zaroor karna chahiye, bina assign kiye app dikhega nahi.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 10/11 physical or virtual device enrolled in Microsoft Intune.
> - Global Admin, Intune Administrator, or Helpdesk Operator rights in the tenant.
> - Local Admin rights on the target device to view logs in `C:\ProgramData`.

### Step 1: Force a Device Sync from Windows Settings

If you push a new policy and don't want to wait 8 hours for the next scheduled sync, force it manually.

```powershell
# Open the Settings app directly to the correct page using run command
start ms-settings:workplace
```
1. Go to **Settings -> Accounts -> Access work or school**.
2. Click on the connected Azure AD/Intune account.
3. Click **Info**.
4. Scroll down and click the **Sync** button.
5. Wait for the timestamp to update.

### Step 2: Collect Intune Diagnostic Logs via Command Line

When escalating to L3 or Microsoft, you need to provide a complete diagnostic bundle.

```cmd
# Use the built-in Windows tool to generate a diagnostic report CAB file
mdmdiagnosticstool.exe -area Autopilot;TPM -cab C:\Temp\MDMLogs.cab
```

> [!success] Expected Output
> ```
> Collection completed. CAB file generated at C:\Temp\MDMLogs.cab
> ```

### Step 3: Analyze the IME Log for App Installation Failures

1. Navigate to the hidden folder: `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs`.
2. Open `IntuneManagementExtension.log` using the **CMTrace** tool (this tool highlights errors in red and is much better than Notepad).
3. Search for the specific App ID (found in the Intune Portal URL when viewing the app) or search for `[Win32App]`.
4. Trace the execution flow: Downloading -> Unzipping -> Detection -> Installation.
5. *Look for exit codes like 1603 (Fatal error during installation) or 1618 (Another installation is in progress).*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `mdmdiagnosticstool.exe` | Collects MDM, Autopilot, and TPM logs into a file. | `mdmdiagnosticstool.exe -area Autopilot -cab c:\logs.cab` |
| `dsregcmd /status` | Shows detailed Azure AD and domain joining status. | `dsregcmd /status` |
| `Get-ScheduledTask -TaskPath "\Microsoft\Intune\"` | Shows Intune related scheduled tasks triggering syncs. | `Get-ScheduledTask -TaskPath "\Microsoft\Intune\"` |
| `Restart-Service IntuneManagementExtension` | Restarts the IME service to force an immediate check-in for apps. | `Restart-Service IntuneManagementExtension` |
| `gpupdate /force` | Can help if local GPOs are conflicting with Intune MDM policies. | `gpupdate /force` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Device not syncing at all** | Device offline, blocked URLs on firewall, expired device token. | Check internet, force sync via Company Portal, verify conditional access and firewall rules. |
| **Win32 App stuck at "Downloading"** | Delivery Optimization issue, blocked ports, proxy blocking BITS. | Restart Delivery Optimization service (`DoSvc`), check proxy settings, clear DO cache. |
| **Policy shows "Pending" for days** | Device is turned off, network disconnected, or not checking in. | Ask user to turn on device, connect to internet, and click 'Sync' manually in Settings. |
| **Configuration Profile Error -2016281112** | Remediation failed (Common in ADMX backed policies). | Check if the setting is applicable to the specific Windows edition (Pro vs Enterprise). |
| **App installation fails with exit code 1618** | Another MSI installation is currently running in the background. | Reboot the device or manually kill the hanging `msiexec.exe` process in Task Manager. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: App Installation Failure (Exit Code 1603)

> [!example] Ticket
> "User is trying to install Google Chrome from the Company Portal, but it fails repeatedly with 'Installation failed'."

**L1 Response:** Ask the user to restart their PC and try installing again over a stable Wi-Fi connection. *Restart karne se pending installations aur locked files clear ho jati hain.*
**Escalation Trigger:** If it still fails after 2-3 attempts and a full reboot.
**L2 Resolution:** Remote into the machine and pull the `IntuneManagementExtension.log`. Searched for the Chrome App ID. Found exit code `1603` during execution. Realized an old, broken version of Chrome was partially uninstalled and blocking the new install. Ran an automated cleanup script via Proactive Remediations, then forced the reinstall from the portal.

### 🎫 Scenario 2: Device Marked as Non-Compliant Blocking Email

> [!example] Ticket
> "Sales Manager cannot access Outlook email on their laptop. The Conditional Access portal shows the device is not compliant."

**L1 Response:** Check the Intune Portal -> Devices -> Device Compliance. Find out exactly *why* it's non-compliant (e.g., OS update missing, BitLocker not active, or password policy failed).
**Escalation Trigger:** If the device physically meets all criteria but Intune still shows it as non-compliant.
**L2 Resolution:** Found that the device hasn't synced with Intune in 7 days, so the compliance state was stale. Had the user open **Settings -> Accounts -> Access Work or School -> Sync**. The device synced successfully, reported its correct encrypted BitLocker status, and instantly became compliant. *Sync na hone ki wajah se cloud mein stale data tha.*

### 🎫 Scenario 3: Autopilot Enrollment Timeout (ESP Blocked)

> [!example] Ticket
> "A new remote laptop is stuck on the ESP (Enrollment Status Page) at 'Installing apps' for over 2 hours."

**L1 Response:** Ask the user to wait up to 60 minutes as some apps are large. If it fails completely, ask them to take a photo of the exact error code on the screen (e.g., 0x81036502).
**Escalation Trigger:** The device consistently times out on the ESP phase across multiple rebuilds.
**L2 Resolution:** Extracted the Autopilot logs by pressing `Shift+F10` on the ESP screen to open CMD, then ran `mdmdiagnosticstool`. Identified that a required Win32 app (a heavy legacy VPN Client) was failing to install silently, causing a timeout that blocked the entire ESP. Changed the app assignment to "Available" instead of "Required" temporarily to let enrollment finish successfully, then deployed the app post-enrollment. *Heavy ya buggy apps ko Autopilot ESP mein block nahi karna chahiye.*

### 🎫 Scenario 4: App Not Showing in Company Portal

> [!example] Ticket
> "User claims they need Visio, but it is not appearing in their Company Portal app to download."

**L1 Response:** Check the Intune portal to verify if Visio is deployed as "Available" to a group that the user is a member of.
**Escalation Trigger:** The user is in the correct group, but the app still doesn't show.
**L2 Resolution:** Found that the Company Portal app was outdated or corrupted. Asked the user to Reset the Company Portal app from Windows Settings. After the reset, the app re-authenticated, synced, and Visio appeared.

---
## 🎤 Interview Questions

> [!question] Q1: Where exactly do you find the logs for Win32 application deployments in Intune?
> **Answer:** The primary log is the `IntuneManagementExtension.log` file, which is located at `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs` on the client device.

==**Exam Tip:** Remember this exact file path! It is the single most commonly asked Intune troubleshooting question in interviews.==

> [!question] Q2: A user's device is not applying a new configuration profile you just created. How do you force a sync locally on a Windows 11 machine?
> **Answer:** You can force a sync from the Company Portal app by clicking settings and sync, or go to Windows Settings > Accounts > Access work or school > Select the connected Azure AD account > Click Info > Scroll down and click the Sync button.

> [!question] Q3: What is the technical difference between "Conflict" and "Not Applicable" in a policy assignment status?
> **Answer:** "Conflict" means two different Intune policies are assigned to the same user/device and are trying to configure the exact same setting with different values (e.g., one says Block, one says Allow). "Not Applicable" means the policy cannot be applied to that specific device because of OS mismatch (e.g., pushing an iOS policy to a Windows device, or an Enterprise feature to a Windows Pro edition).

> [!question] Q4: How do you troubleshoot if a PowerShell script deployed via Intune is not running correctly?
> **Answer:** Check the `AgentExecutor.log` located in the `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs` directory. Also verify if the script was configured to run as SYSTEM or the logged-on user, and ensure the local PowerShell execution policy is not overriding it (Intune usually bypasses it, but it's worth checking).

> [!question] Q5: What built-in Windows command-line tool do you use to gather all MDM related diagnostic logs into a single CAB file for Microsoft Support?
> **Answer:** You use the `mdmdiagnosticstool.exe` utility, typically with the `-area Autopilot;TPM -cab` parameters.

> [!question] Q6: What does the command `dsregcmd /status` do in the context of device management?
> **Answer:** It provides detailed information about the device's registration state with Azure AD (e.g., AzureAdJoined, DomainJoined), PRT (Primary Refresh Token) status, and whether the device is capable of fetching policies. It is vital for fixing identity and authentication issues in Intune.

---
## 🔗 Related Notes

- [[INT-01 Intune Basics|INT-01: Introduction to Intune]] — *For basic Intune architecture and device lifecycle.*
- [[INT-05 Application Deployment|INT-05: Deploying Win32 Apps]] — *Learn how apps are packaged and deployed before you troubleshoot them.*
- [[WIN-12 Event Viewer Tricks|WIN-12: Event Viewer Advanced Tricks]] — *How to read the MDM event logs efficiently.*
- [[SEC-03 BitLocker Management|SEC-03: BitLocker via Intune]] — *Troubleshooting BitLocker compliance and escrow issues.*
