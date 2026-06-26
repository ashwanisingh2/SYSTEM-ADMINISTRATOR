---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-05-application-management, int-05]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#md-102`

# INT-05: Application Management

> [!abstract] Overview
> This note covers application lifecycle management in Microsoft Intune, detailing app packaging, Win32 app deployment, Microsoft 365 apps deployment, and App Protection Policies (MAM).

---
## 🧠 Concept Overview

- **What it is** — Intune Application Management is a digital delivery service for corporate applications and policies.
- **Why it matters** — Allows IT to securely distribute, track, and manage applications on corporate and personal devices.
- **Where you see this** — When pushing new software to employees, managing app updates, or securing corporate data on personal phones.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check app installation status, review client logs |
| **L2** | Package and deploy Win32 apps, configure App Protection Policies |
| **L3** | Architect company-wide application deployment strategies and compliance |

> [!tip] Seedha Simple Mein
> *Intune ke zariye aap users ke device par applications auto-install kar sakte hain. Win32 apps ko `.intunewin` package mein convert karke, silent install and detection rules ke sath push kiya jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Intune Application Management** is like **a digital delivery service** because...
>
> - **Store Apps & Web Clips** are like ordering items straight from a public store to a user's desk.
> - **Win32 Apps (.intunewin)** are like custom machinery. You can't ship them raw; you must place them inside a standardized container box (packaged using the `IntuneWinAppUtil` tool) along with detailed instructions on how to assemble it (install command), check if it's running (detection rules), and return it if needed (uninstall command).
> - **App Protection Policies (MAM)** are like corporate safes placed inside personal homes. The user can interact with the app, but they cannot drag files out of the safe and drop them onto their personal tables.

---
## 🔬 Technical Deep Dive

### 1. Intune App Types

> [!info] Key Concept
> Intune supports various application types depending on the deployment scenario and platform.

- **Store Apps**: Directly integrated with the Microsoft Store (using the new Windows Package Manager winget interface), Apple App Store, and Google Play Store.
- **Line-of-Business (LOB) Apps**: Direct uploads of `.msi` or `.apk` files. Good for simple installations, but lacks the advanced features (such as dependencies and detection scripts) of Win32 apps.
- **Win32 Apps**: Custom classic Windows apps. Required for any installation involving setup scripts, multiple files, `.exe` installers, or complex parameters.
- **Microsoft 365 Apps**: Built-in template to configure and deploy the Office suite (Word, Excel, Outlook, Teams) to client devices.

> [!danger] Common Mistake
> Uploading large installers as LOB (Line-of-Business) `.msi` packages instead of Win32 app `.intunewin` packages. LOB apps fail to handle dependencies or custom exit codes, leading to deployment failures.

### 2. Win32 App Packaging Flow

To deploy a Win32 app, it must be packaged into the `.intunewin` format using the **Microsoft Win32 Content Prep Tool** (`IntuneWinAppUtil.exe`). This tool zips the source files, encrypts them, and calculates verification hashes.

Key requirements during upload:
- **Install command**: Silent installation arguments (e.g., `setup.exe /select /q`).
- **Uninstall command**: Command to remove the app (e.g., `msiexec /x {GUID} /qn`).
- **Detection Rules**: Tell Intune how to verify the installation succeeded. Can check a file registry path, check for a specific file/folder, or run a custom PowerShell detection script.

> [!danger] Common Mistake
> Forgetting the silent switch flag (e.g., `/qn` or `/S`) in the install command. The installation will run in the system context and wait for user input indefinitely, causing the deployment to time out.
> Using detection rules that check for a folder that the app uninstaller fails to delete, which causes Intune to report the app is still installed.

> [!tip] Pro Tip
> Always add **Dependencies** for complex applications. For instance, if an app requires the .NET Framework or a specific runtime, package the runtime as a separate Win32 app and set it as a dependency for the main app.

### 3. Assignments & Targets

- **Required**: The app installs automatically in the background.
- **Available**: The app appears in the Company Portal, allowing users to install it manually.
- **Uninstall**: Automatically removes the app if it is detected on the device.

### 4. App Protection Policies (MAM)

Protects corporate data at the application level:
- Prevents users from copying data from corporate-managed apps (like Outlook) and pasting it into personal apps (like WhatsApp).
- Restricts saving file attachments to local personal storage, forcing files onto OneDrive for Business.
- Enforces an app-specific access PIN, independent of the device passcode.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Microsoft Intune Admin Center access.
> - A Windows development machine with the `IntuneWinAppUtil.exe` packaging utility downloaded.
> - A test installer file (e.g., VLC Player `.exe` installer).

### Step 1: Package the Application

1. Create a folder `C:\AppSrc` and place the VLC installer `vlc-setup.exe` inside it.
2. Create a folder `C:\AppOut` to store the output.
3. Open PowerShell as Administrator, navigate to the folder containing `IntuneWinAppUtil.exe`, and run:

```powershell
# Generate the .intunewin package
.\IntuneWinAppUtil.exe -c C:\AppSrc -s vlc-setup.exe -o C:\AppOut
```

> [!success] Expected Output
> This generates the package file `vlc-setup.intunewin` inside `C:\AppOut`.

### Step 2: Upload the Win32 App to Intune

1. Open the **Microsoft Intune Admin Center** -> `Apps` -> `All apps` -> Click `Add`.
2. App type: Select **Windows app (Win32)** -> Click **Select**.
3. Select the app package file: Browse and upload `vlc-setup.intunewin`.
4. Name the app `VLC Media Player` and enter the publisher as `VideoLAN`. Click **Next**.

### Step 3: Configure Install Commands & Detection Rules

1. Set the following parameters:
   - Install command: `vlc-setup.exe /S` (case sensitive silent switch for VLC).
   - Uninstall command: `C:\Program Files\VideoLAN\VLC\uninstall.exe /S`.
   - Install behavior: **System**.
2. Click **Next** -> Under **Requirements**:
   - Operating system architecture: Select **64-bit**.
   - Minimum operating system: Select **Windows 10 21H2**.
3. Click **Next** -> Under **Detection Rules**:
   - Rules format: **Manually configure detection rules**.
   - Click **Add** -> Rule type: **File**.
   - Path: `C:\Program Files\VideoLAN\VLC`.
   - File or folder: `vlc.exe`.
   - Detection method: **File or folder exists**.
4. Click **OK** -> Click **Next**.

### Step 4: Assign and Verify Installation

1. Under **Assignments**, go to the **Required** section.
2. Click **Add group** and select your test device security group.
3. Click **Next** -> Click **Create** to upload the package.
4. Verify deployment progress under `Apps` -> `All apps` -> Select `VLC Media Player` -> Monitor the **Installation Status** chart.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-Content` | Get local client Intune Management Extension agent logs | `Get-Content -Path "C:\...\IntuneManagementExtension.log" -Tail 50 -Wait` |
| `Restart-Service` | Clean up cache on a client machine to force app redelivery | `Restart-Service -Name "IntuneManagementExtension"` |

---
## 🚑 Troubleshooting Guide

> [!bug] Troubleshooting Note
> When debugging Win32 app installations, use **CMTrace** or **OneTrace** to view the `IntuneManagementExtension.log` on the client machine. This log records the download status, command executions, and detection rule outcomes in real-time.

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Win32 app fails to install, returning error code `0x87D1041C` | App installed successfully, but **Detection Rule** failed to find the file/registry path specified | 1. Verify if app is installed and runs. <br>2. Verify exact installation path. <br>3. Edit app config in Intune to update Detection Rule path. |
| Microsoft 365 Apps deployment fails on machines with retail Office | Installation engine encounters conflicts with existing MSI-based Office installations | Edit M365 Apps profile in Intune -> toggle **Remove existing MSI versions of Office** to **Yes**. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Application Deployment Failure

> [!example] Ticket
> "User is missing the new Accounting app. Intune shows deployment failed with error code 0x87D1041C."

**L1 Response:** Log onto the client machine and verify if the application actually installed but was not detected, or if it failed to install entirely.
**Escalation Trigger:** Pass to L2 if the app is not installed at all and requires package modification.
**L2 Resolution:** Review `IntuneManagementExtension.log` to determine the point of failure. Adjust the Detection Rule in the Intune Admin Center if the path is incorrect.

---
## 🎤 Interview Questions

> [!question] Q1: What is the purpose of the Microsoft Win32 Content Prep Tool, and why is it required?
> **Answer:** The Microsoft Win32 Content Prep Tool (`IntuneWinAppUtil.exe`) packages classic Windows applications (.exe or .msi) into the `.intunewin` format. It compresses the installation files, encrypts the package, and calculates verification hashes. This is required because Intune's Win32 deployment engine requires a standardized package format to securely distribute, decrypt, install, and track applications on client machines.

> [!question] Q2: A client reports that their custom accounting application fails to install via Intune. It requires a registry key to be written before installation. How would you handle this?
> **Answer:** Write a PowerShell script (`install.ps1`) that writes the required registry keys and then calls the application installer silently: `Start-Process "setup.exe" -ArgumentList "/S" -Wait`. Package the script and the installer together into a `.intunewin` file using the prep tool, and set the Intune install command to: `powershell.exe -ExecutionPolicy Bypass -File install.ps1`.

> [!question] Q3: How do App Protection Policies (MAM) protect data on a personal device without enrolling it in MDM?
> **Answer:** App Protection Policies manage corporate data *inside* specific applications (like Outlook, Word, and OneDrive). When a user logs into these apps with their corporate account, Intune applies policies that encrypt corporate data, require an app PIN, and block copy-paste actions between corporate apps and personal apps. It also blocks saving corporate files to personal storage. This secures corporate data without requiring full device enrollment.

==**Exam Tip:** Win32 Apps require `.intunewin` packaging, silent install switches, and accurate detection rules to deploy successfully.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes identity joins and MDM authority.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details device and OS configurations.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-06 Windows Autopilot|INT-06 Windows Autopilot]] — Coordinates app deployment during device setup.
