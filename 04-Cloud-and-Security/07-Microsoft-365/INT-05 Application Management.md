---
tags: [sysadmin, intune, apps]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# INT-05: Application Management

> [!abstract] Overview
> This note covers application lifecycle management in Microsoft Intune, detailing app packaging, Win32 app deployment, Microsoft 365 apps deployment, and App Protection Policies (MAM).

---
## Concept
Think of Intune Application Management as a digital delivery service.
- **Store Apps & Web Clips** are like ordering items straight from a public store to a user's desk.
- **Win32 Apps (.intunewin)** are like custom machinery. You can't ship them raw; you must place them inside a standardized container box (packaged using the `IntuneWinAppUtil` tool) along with detailed instructions on how to assemble it (install command), check if it's running (detection rules), and return it if needed (uninstall command).
- **App Protection Policies (MAM)** are like corporate safes placed inside personal homes. The user can interact with the app, but they cannot drag files out of the safe and drop them onto their personal tables (e.g., personal OneDrive or Gmail).
*Seedha simple mein: Intune ke zariye aap users ke device par applications auto-install kar sakte hain. Win32 apps ko `.intunewin` package mein convert karke, silent install and detection rules ke sath push kiya jata hai.*

---
## Technical Deep Dive

### 1. Intune App Types
- **Store Apps**: Directly integrated with the Microsoft Store (using the new Windows Package Manager winget interface), Apple App Store, and Google Play Store.
- **Line-of-Business (LOB) Apps**: Direct uploads of `.msi` or `.apk` files. Good for simple installations, but lacks the advanced features (such as dependencies and detection scripts) of Win32 apps.
- **Win32 Apps**: Custom classic Windows apps. Required for any installation involving setup scripts, multiple files, `.exe` installers, or complex parameters.
- **Microsoft 365 Apps**: Built-in template to configure and deploy the Office suite (Word, Excel, Outlook, Teams) to client devices.

### 2. Win32 App Packaging Flow
To deploy a Win32 app, it must be packaged into the `.intunewin` format using the **Microsoft Win32 Content Prep Tool** (`IntuneWinAppUtil.exe`). This tool zips the source files, encrypts them, and calculates verification hashes.
Key requirements during upload:
- **Install command**: Silent installation arguments (e.g., `setup.exe /select /q`).
- **Uninstall command**: Command to remove the app (e.g., `msiexec /x {GUID} /qn`).
- **Detection Rules**: Tell Intune how to verify the installation succeeded. Can check a file registry path, check for a specific file/folder, or run a custom PowerShell detection script.

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
## Lab — Step by Step
> [!info] Lab Setup Needed
> Microsoft Intune Admin Center access. A Windows development machine with the `IntuneWinAppUtil.exe` packaging utility downloaded. A test installer file (e.g., VLC Player `.exe` installer).

### Step 1: Package the Application
1. Create a folder `C:\AppSrc` and place the VLC installer `vlc-setup.exe` inside it.
2. Create a folder `C:\AppOut` to store the output.
3. Open PowerShell as Administrator, navigate to the folder containing `IntuneWinAppUtil.exe`, and run:
```powershell
.\IntuneWinAppUtil.exe -c C:\AppSrc -s vlc-setup.exe -o C:\AppOut
```
4. This generates the package file `vlc-setup.intunewin` inside `C:\AppOut`.

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
## Commands Reference
```powershell
# Get local client Intune Management Extension agent logs
Get-Content -Path "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log" -Tail 50 -Wait
# Clean up cache on a client machine to force app redelivery
Restart-Service -Name "IntuneManagementExtension"
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- Problem: A Win32 app fails to install on client machines, returning error code `0x87D1041C` in the Intune console.
- Root Cause: The application installation completed successfully, but the **Detection Rule** failed to find the file or registry path specified, causing Intune to report the installation as failed.
- Fix:
  1. Log onto the client machine and verify if the application is present and runs.
  2. If the app is installed, verify its exact installation path (e.g., check if it installed to `C:\Program Files (x86)` instead of `C:\Program Files`).
  3. Open the **Intune Admin Center** -> Edit the app configuration -> Under **Detection Rules**, update the path or registry key to match the actual installation location.

**Scenario 2:**
- Problem: An administrator tries to deploy Microsoft 365 Apps, but the deployment fails on machines that have existing retail versions of Office pre-installed.
- Root Cause: The installation engine encounters conflicts with existing MSI-based Office installations.
- Fix:
  1. Open the **Intune Admin Center** -> Navigate to `Apps` -> Select the Microsoft 365 Apps profile -> Edit properties.
  2. Under the configuration settings, toggle **Remove existing MSI versions of Office** to **Yes**.
  3. Save the changes and sync the target clients to trigger the removal and re-installation.

---
## Common Mistakes
> [!warning] Avoid These
> Uploading large installers as LOB (Line-of-Business) `.msi` packages instead of Win32 app `.intunewin` packages. LOB apps fail to handle dependencies or custom exit codes, leading to deployment failures.
> Forgetting the silent switch flag (e.g., `/qn` or `/S`) in the install command. The installation will run in the system context and wait for user input indefinitely, causing the deployment to time out.
> Using detection rules that check for a folder that the app uninstaller fails to delete, which causes Intune to report the app is still installed.

---
## Pro Tips
> [!tip] Field Experience
> When debugging Win32 app installations, use **CMTrace** or **OneTrace** to view the `IntuneManagementExtension.log` on the client machine. This log records the download status, command executions, and detection rule outcomes in real-time.
> Always add **Dependencies** for complex applications. For instance, if an app requires the .NET Framework or a specific runtime, package the runtime as a separate Win32 app and set it as a dependency for the main app.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Win32 App | Standardized package format enabling complex app deployments with detection scripts. |
| 2 | IntuneWinAppUtil | Prep tool that packages installation source folders into encrypted `.intunewin` files. |
| 3 | Detection Rule | Check parameters (file, registry, script) verifying if an app is successfully installed. |
| 4 | MAM (App Protection)| Security policies restricting data sharing and enforcing PIN locks inside corporate apps. |
| 5 | Required Assignment | Forces silent background installation of the app on target devices. |

---
## Interview Q&A

**Q1: What is the purpose of the Microsoft Win32 Content Prep Tool, and why is it required?**
A: The Microsoft Win32 Content Prep Tool (`IntuneWinAppUtil.exe`) packages classic Windows applications (.exe or .msi) into the `.intunewin` format. It compresses the installation files, encrypts the package, and calculates verification hashes. This is required because Intune's Win32 deployment engine requires a standardized package format to securely distribute, decrypt, install, and track applications on client machines.

**Q2: A client reports that their custom accounting application fails to install via Intune. It requires a registry key to be written before installation. How would you handle this?**
A:
- **Situation**: The client needed a legacy app deployed that required registry configuration before launching the installer.
- **Task**: I needed to package the installation steps into a single Win32 app.
- **Action**: I wrote a PowerShell script (`install.ps1`) that writes the required registry keys and then calls the application installer silently: `Start-Process "setup.exe" -ArgumentList "/S" -Wait`. I packaged the script and the installer together into a `.intunewin` file using the prep tool, and set the Intune install command to: `powershell.exe -ExecutionPolicy Bypass -File install.ps1`.
- **Result**: The app deployed successfully. The script configured the registry keys before running the installer, resolving the installation failures.

**Q3: How do App Protection Policies (MAM) protect data on a personal device without enrolling it in MDM?**
A: App Protection Policies manage corporate data *inside* specific applications (like Outlook, Word, and OneDrive). When a user logs into these apps with their corporate account, Intune applies policies that encrypt corporate data, require an app PIN, and block copy-paste actions between corporate apps and personal apps. It also blocks saving corporate files to personal storage. This secures corporate data without requiring full device enrollment.

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes identity joins and MDM authority.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details device and OS configurations.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-06 Windows Autopilot|INT-06 Windows Autopilot]] — Coordinates app deployment during device setup.

