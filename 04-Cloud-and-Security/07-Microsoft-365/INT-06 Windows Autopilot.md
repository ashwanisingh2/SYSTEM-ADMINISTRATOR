---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-06-windows-autopilot, int-06]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#advanced` `#md-102`

# INT-06: Windows Autopilot

> [!abstract] Overview
> Yeh note Windows Autopilot ko cover karta hai, jo ek suite of technologies hai naye devices ko set up aur pre-configure karne ke liye. Ek support engineer ko yeh aana chahiye taaki woh bina physical intervention ke zero-touch deployment kar sake aur time bacha sake.

---
## 🧠 Concept Overview

- **What it is** — Cloud-based deployment service jo naye Windows devices ko automatically setup karta hai without traditional imaging.
- **Why it matters** — IT team ka time bachata hai kyunki naye laptops direct vendor se user ke ghar bheje ja sakte hain aur automatically configure ho jate hain.
- **Where you see this** — Jab naya employee join karta hai aur usko naya laptop assign hota hai, ya jab purana laptop reset karke doobara use mein laana ho.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check karna ki Autopilot profile device pe assign hui hai ya nahi, aur hardware hash extract karke dena. |
| **L2** | Hardware hash Intune mein upload karna, ESP (Enrollment Status Page) errors troubleshoot karna aur profile create karna. |
| **L3** | Autopilot deployment architecture design karna, complex app packaging (ESP blocking ke liye) aur hybrid join setup karna. |

> [!tip] Seedha Simple Mein
> *Autopilot ke zariye naye laptops ko bina manually image kiye, directly user ke ghar bheja ja sakta hai. Jaise hi user laptop on karke Wi-Fi se connect karega, company ke apps aur settings automatic deploy ho jayenge.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Windows Autopilot** is like **VIP airport check-in service** because...
>
> - Jaise VIP check-in mein aapka luggage aur boarding pass pehle se sorted hota hai, waise hi device ki profile pehle se cloud mein ready hoti hai.
> - User ko bas apna identity (passport/credentials) dikhana hota hai aur baaki sab apne aap set ho jata hai.

---
## 🔬 Technical Deep Dive

### 1. Autopilot Deployment Modes

> [!info] Key Concept
> Autopilot ke alag-alag modes hote hain jo alag use-cases ke liye banaye gaye hain.

- **User-Driven Mode**: Standard deployments ke liye. User device on karta hai, Wi-Fi connect karta hai aur apne corporate credentials dalta hai.
- **Pre-Provisioning (White Glove)**: Two-stage deployment jisme technician pehle device boot karke apps download kar deta hai, phir user ko deta hai taaki login fast ho.
- **Self-Deploying Mode**: Shared devices/kiosks ke liye. Bina user credentials ke automatically Entra ID join aur Intune enroll ho jata hai (TPM 2.0 required).

> [!danger] Common Mistake
> Testing Autopilot on virtual machines without allocating a virtual TPM 2.0 chip, causing Self-Deploying or Pre-Provisioning deployments to fail.

### 2. Hardware Hash Registration

Hardware hash upload karna zaroori hai. Isme Device Serial Number, Windows Product ID, aur Hardware Hash string hoti hai.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 11 virtual machine
> - Access to the Microsoft Intune Admin Center
> - A USB drive or shared folder to transfer files

### Step 1: Extract the Hardware Hash on Client VM

```powershell
# Open PowerShell (Shift + F10 at OOBE)
powershell -ExecutionPolicy Bypass

# Install the script and export hash
Install-Script -Name Get-WindowsAutopilotInfo -Force
Get-WindowsAutopilotInfo.ps1 -OutputFile C:\AutopilotHash.csv
```

> [!success] Expected Output
> ```
> Hash successfully generated and saved to C:\AutopilotHash.csv
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Install-Script -Name Get-WindowsAutopilotInfo` | Autopilot hash nikalne wala tool install karta hai | `Install-Script -Name Get-WindowsAutopilotInfo -Force` |
| `Get-WindowsAutopilotInfo.ps1 -Online` | Script run karke directly tenant mein hash upload karta hai (admin rights required) | `Get-WindowsAutopilotInfo.ps1 -Online` |
| `systemreset -cleanpc` | System wipe karke Autopilot check dobara force karta hai | `systemreset -cleanpc` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Standard Windows OOBE screen instead of corporate login | Hash import nahi hua ya profile assign nahi hui | Verify serial in Intune -> Devices -> Autopilot. Run `systemreset -cleanpc` to retry. |
| ESP gets stuck on "Account setup", returns timeout 0x800705b4 | Ek required application install hone mein fail ho rahi hai ya silent switch missing hai | Press `Ctrl + Shift + D` to get logs. Check IntuneManagementExtension.log for exit codes. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: New Hire Laptop Setup

> [!example] Ticket
> "New user John Doe is starting on Monday. His laptop is shipped directly to his home, but he doesn't see the company login screen."

**L1 Response:** Check if the device is connected to the internet and verify the serial number with the vendor.
**Escalation Trigger:** Agar serial tenant mein registered nahi milta ya profile 'Not assigned' show karta hai.
**L2 Resolution:** Import the hash manually or verify dynamic group membership `(device.devicePhysicalIDs -any _ -contains "[ZTDID]")`. Instruct user to reboot.

---
## 🎤 Interview Questions

> [!question] Q1: Explain the difference between User-Driven, Pre-Provisioning, and Self-Deploying modes.
> **Answer:** User-Driven needs user credentials. Pre-Provisioning lets a tech pre-load apps for faster end-user setup. Self-Deploying requires no user credentials and uses TPM 2.0 to auto-enroll kiosks.

==**Exam Tip:** Self-Deploying mode always requires a physical TPM 2.0 chip!==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Sets up the tenant MDM authority and licensing.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-02 Device Enrollment|INT-02 Device Enrollment]] — Details platform enrollment profiles.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-05 Application Management|INT-05 Application Management]] — Details Win32 packaging used in the ESP phase.
