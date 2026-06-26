---
tags: [microsoft-365, intune, app-deployment, mam, mdm]
aliases: [Intune App Deployment, MAM, Mobile Application Management]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#intermediate` `#md-102`

# INT-03: App Deployment and MAM

> [!abstract] Overview
> Yeh note Microsoft Intune ke through applications ko deploy karne aur Mobile Application Management (MAM) policies ko configure karne par focus karta hai. Ek system administrator ke roop mein, aapko pata hona chahiye ki apps ko secure tareeqe se users tak kaise pahunchana hai bina unke personal data ko compromise kiye (BYOD scenarios mein). App deployment modern workplace management ka ek core pillar hai.

---
## 🧠 Concept Overview

- **What it is** — App deployment is the process of distributing software (Win32, Store apps, iOS/Android apps) to corporate devices using Intune. MAM (Mobile Application Management) focuses specifically on protecting corporate data *within* the app, even on personal, unmanaged devices (BYOD).
- **Why it matters** — Security aur compliance maintain karne ke liye. Users har jagah se kaam karte hain (Work from Anywhere), isliye corporate apps ko install karna aur usme se copy-paste restrict karna zaroori hai taaki data leak na ho.
- **Where you see this** — Jab company naya software (like Office 365, VPN clients, or Zoom) sabhi laptops par silently install karna chahti hai, ya jab users apne personal mobile pe Outlook app use karna chahte hain but IT wants to block copy-pasting into personal apps like WhatsApp or personal notes.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check karna ki app Company Portal mein visible hai ya nahi, basic app sync issue resolve karna, log collect karna. |
| **L2** | App packages (Win32) banana, silent switches ke sath deploy karna, aur MAM App Protection Policies (APP) create/troubleshoot karna. |
| **L3** | Complex app dependencies, registry-based detection methods design karna, App Supersedence rules set karna, aur overall mobile security architecture banana. |

> [!tip] Seedha Simple Mein
> *App Deployment matlab software ko users ke system me automatically bhejna bina unko disturb kiye. Aur MAM ka matlab hai ki app ke andar ka company data safe rahe, chahe mobile user ka apna personal hi kyun na ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **MAM (Mobile Application Management)** is like **Ek safe vault (Tijori) inside a normal house** because...
>
> - **[Normal House]** is the user's personal smartphone (BYOD). The company doesn't control the house. They can't see the photos, texts, or personal apps.
> - **[Safe Vault]** is the managed application (like Microsoft Outlook or Teams). The company only controls the vault.
> - You can bring things (emails/files) into the vault from outside, but you cannot take them out of the vault and put them in the rest of the house (restrict copy-paste to personal apps).

---
## 🔬 Technical Deep Dive

### 1. Types of App Deployments in Intune

> [!info] Key Concept
> Intune supports various application formats depending on the target OS and enrollment type.

- **Microsoft Store Apps (New):** Directly linked from the Microsoft Store using the Windows Package Manager (Winget) integration. Easy to deploy and auto-updates natively.
- **Microsoft 365 Apps:** Automated deployment of the Office suite (Word, Excel, PowerPoint, Teams) directly from Intune without needing external install files.
- **Win32 Apps (.intunewin):** Legacy `.exe` or `.msi` applications that need to be packaged using the Intune Win32 App Packaging Tool. These are the most common for enterprise custom software.
- **LOB (Line of Business) Apps:** Simple `.msi`, `.appx`, `.apk`, or `.ipa` files. *Note: Mixing LOB and Win32 apps during Autopilot is not recommended.*

> [!danger] Common Mistake
> *Kabhi bhi Autopilot ESP (Enrollment Status Page) phase mein Win32 aur LOB apps ko ek saath mix mat karna. Isse deployment conflict hota hai aur Autopilot fail ho sakta hai.*

### 2. Win32 App Packaging Architecture

To deploy a standard Win32 application, you must convert it to `.intunewin` format. Yeh format file ko compress aur encrypt karta hai Azure storage ke liye.

1. **Installer:** Need the original `.exe` or `.msi`.
2. **Install Command:** Intune UI me silent switches dene hote hain (e.g., `setup.exe /quiet /norestart`).
3. **Uninstall Command:** Required so Intune can remove the app if the user is unassigned.
4. **Detection Rules:** Intune needs to know if the app is successfully installed (e.g., checking a file path, registry key, or MSI product code).

### 3. Dependencies and Supersedence

- **Dependencies:** Agar ek app (App B) ko install hone se pehle koi dusra framework (App A, like Java or .NET) chahiye, toh aap Intune me dependency define kar sakte hain. Intune pehle App A install karega, fir App B.
- **Supersedence:** Used for upgrading apps. Agar App V2 aa gaya hai, toh aap Supersedence rule laga kar keh sakte ho ki pehle App V1 ko uninstall karo, fir V2 install karo.

### 4. Mobile Application Management (MAM)

> [!info] Key Concept
> MAM works through App Protection Policies (APP) and doesn't require MDM enrollment. Iska matab device ko fully manage kiye bina apps ko manage kiya jaa sakta hai.

MAM policies control:
- **Data Relocation:** Prevent Save As to local storage, block cut/copy/paste between managed (work) and unmanaged (personal) apps.
- **Access Requirements:** Require a PIN or Biometrics (FaceID/Fingerprint) to open the app, even if the phone itself is already unlocked.
- **Conditional Launch:** Block access on jailbroken/rooted devices, or devices with an outdated OS version.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Intune Administrator or Global Admin rights.
> - Microsoft Intune Win32 App Packaging Tool downloaded (from GitHub).
> - A test app installer (e.g., `Notepad++.exe`).

### Step 1: Package a Win32 App

```powershell
# Open PowerShell and run the IntuneWinAppUtil.exe tool
# -c = Source folder, -s = Setup file, -o = Output folder
.\IntuneWinAppUtil.exe -c "C:\AppSource" -s "Notepad++.exe" -o "C:\AppOutput"
```

> [!success] Expected Output
> ```text
> Validating parameters
> Done!!!
> Wrote C:\AppOutput\Notepad++.intunewin
> ```
> *Yeh tool aapke setup file ko compress aur encrypt karke ek .intunewin file bana dega jo directly Intune portal pe upload ki jayegi.*

### Step 2: Upload and Configure in Intune Portal

1. Go to **Intune admin center** -> **Apps** -> **Windows**.
2. Click **Add**, select **Windows app (Win32)**.
3. Upload the `.intunewin` file created in Step 1.
4. Configure App Information (Name, Description, Publisher).
5. Specify **Install command**: `Notepad++.exe /S` (Silent install flag specific to Notepad++).
6. Specify **Uninstall command**: `C:\Program Files\Notepad++\uninstall.exe /S`.
7. Set installation behavior to **System** (installs for all users) or **User** (installs only for the logged-in user).

### Step 3: Set Detection Rules

> [!info] Important Step
> *Detection rule yeh confirm karti hai ki installation successful hui ya nahi. Agar yeh galat hui, toh app bar-bar install hone ki koshish karegi.*

1. **Rules format:** Manually configure detection rules.
2. **Rule type:** File.
3. **Path:** `C:\Program Files\Notepad++`
4. **File or folder:** `notepad++.exe`
5. **Detection method:** File or folder exists.

### Step 4: Create MAM App Protection Policy (iOS/Android)

1. Go to **Apps** -> **App protection policies** -> **Create policy** (Select iOS/iPadOS).
2. Name it `BYOD - Protect Outlook Data`.
3. Target to **Selected apps** -> Add `Microsoft Outlook` and `Microsoft Teams`.
4. Under **Data Protection**, set **Restrict cut, copy, and paste between other apps** to `Managed apps with paste in`.
5. Under **Access requirements**, set **PIN for access** to `Require`.
6. Assign policy to the target user group.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `IntuneWinAppUtil.exe -c <src> -s <setup> -o <out>` | App ko `.intunewin` format mein package karta hai. | `IntuneWinAppUtil.exe -c "C:\src" -s "app.exe" -o "C:\out"` |
| `Get-Service IntuneManagementExtension` | Check karta hai ki Intune Agent service (IME) chal rahi hai ya nahi. | `Get-Service IntuneManagementExtension` |
| `Restart-Service IntuneManagementExtension -Force` | Intune agent ko restart karke policies/apps ko force pull karta hai. | `Restart-Service IntuneManagementExtension -Force` |
| `Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*` | Registry se installed apps ke Uninstall strings aur Display names nikalta hai (useful for detection rules). | `Get-ItemProperty ... | Select-Object DisplayName` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Win32 App failed to install (Error 0x80070002)** | File path in install command incorrect or missing dependencies. System cannot find the file specified. | Check install command arguments. Verify the source folder during packaging had all necessary files. |
| **App installs but shows "Failed" in Intune portal** | Detection rules are wrong. *App install ho gaya par Intune dhoondh nahi paaya.* | Update detection rule registry key or file path to exactly match what the app creates upon successful installation. |
| **MAM policy not applying on mobile device** | App is not MTD (Mobile Threat Defense) compliant or user hasn't logged in with work account. | Ensure user logs into Outlook with company credentials. Check policy assignment groups. |
| **Company Portal app stuck on "Downloading"** | Delivery Optimization issue or network firewall/proxy blocking Microsoft Intune endpoints. | Check network proxy/firewall rules. Restart IntuneManagementExtension service. |
| **User prompted for UAC during app installation** | App installation context is set to 'User' instead of 'System', and user is standard non-admin. | Change App installation behavior to 'System' in Intune so it runs with SYSTEM privileges. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Cannot Copy Text from Outlook to WhatsApp

> [!example] Ticket
> "I am trying to copy an address from a work email on my phone and paste it into my personal notes app, but it says 'Your organization's data cannot be pasted here'."

**L1 Response:** Explain to the user that this is expected behavior due to company security policies (MAM) to prevent data leakage.
**Escalation Trigger:** None required if it's standard policy. Escalate if an exception is explicitly approved by HR/Security.
**L2 Resolution:** No action needed unless a specific app exception needs to be added to the App Protection Policy Data Transfer Exemptions.

### 🎫 Scenario 2: Win32 Application Deployment Failure

> [!example] Ticket
> "The new Adobe Acrobat Reader deployment is failing on 20% of Windows devices."

**L1 Response:** Ask user to sync device via Company Portal. Check if device has enough disk space and is connected to the internet.
**Escalation Trigger:** If manual sync doesn't fix it and the error code points to a detection or installation syntax issue across multiple users.
**L2 Resolution:** Review the `IntuneManagementExtension.log` in `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs`. Fix the detection rule or installation command line (e.g., adding `/qn` for silent install) and update the app package in Intune.

### 🎫 Scenario 3: Missing Required Application

> [!example] Ticket
> "I got my new laptop but the corporate VPN client software is not installed automatically."

**L1 Response:** Open Company Portal, see if the app is available for manual download. Check if the user is in the correct Azure AD (Entra ID) group assigned to the app.
**Escalation Trigger:** If user is in the group but the app never evaluates or shows as "Not Applicable".
**L2 Resolution:** Check the App Requirements (e.g., requires Windows 11, but user is on Windows 10, or architecture mismatch like x86 vs x64). Modify the app requirements or assign the correct version.

---
## 🎤 Interview Questions

> [!question] Q1: What is the exact difference between MDM and MAM?
> **Answer:** MDM (Mobile Device Management) controls the entire device (wiping the device, setting device PIN, Wi-Fi profiles). MAM (Mobile Application Management) controls only the specific applications and the corporate data within them. MAM is ideal for BYOD (Bring Your Own Device) because you secure the data without enrolling the whole personal device.

==**Exam Tip:** Intune can apply MAM policies (App Protection Policies) to devices that are NOT enrolled in Intune MDM. This is a very common exam question.==

> [!question] Q2: Explain the `.intunewin` file format and why it is used.
> **Answer:** It is a proprietary, compressed, and encrypted package format used by Intune to deploy Win32 applications. It wraps the original `.exe` or `.msi` installers and their dependencies into a single package using the Intune Win32 App Packaging tool before uploading to Azure storage.

> [!question] Q3: What happens if your Win32 app installs successfully, but your detection rule is incorrect?
> **Answer:** Intune will run the installer. The app will install correctly. Then Intune will check the detection rule to verify. Because the rule fails (e.g., looking at the wrong path), Intune will report the deployment as "Failed" in the console, and it may repeatedly try to reinstall the app on subsequent syncs.

> [!question] Q4: How do you troubleshoot Win32 app deployments on a Windows 10/11 client? Which log file is critical?
> **Answer:** You must look at the `IntuneManagementExtension.log` located at `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs`. This log contains detailed step-by-step information about app download, extraction, execution, and detection rule evaluation.

> [!question] Q5: What is App Supersedence in Intune?
> **Answer:** App supersedence allows you to update or replace an existing Win32 app with a newer version. You specify that App V2 supersedes App V1, and you can choose whether to automatically uninstall V1 before installing V2.

---
## 🔗 Related Notes

- [[INT-01 Introduction to Intune|Introduction to Intune]] — Basics of Microsoft Endpoint Manager.
- [[INT-02 Device Enrollment|Device Enrollment]] — How devices get into Intune (Autopilot, etc.).
- [[Azure AD Groups|Azure AD Groups]] — Used for assigning applications to users.
