---
tags: [desktop-support, windows-os, registry, L2]
aliases: [windows-registry, regedit-guide]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS**

`#complete` `#intermediate` `#md-102`

# WIN-04: Registry

> [!abstract] Overview
> Yeh note Windows Registry ko cover karta hai, jo ek centralized database hai jahan OS aur apps ke configuration settings save hote hain. Ek support engineer ke liye isme kaam karna aana chahiye taaki advanced issues ko resolve kiya ja sake, lekin galti se system crash hone ka risk hota hai.

---
## 🧠 Concept Overview

- **What it is** — The Windows Registry is a centralized, hierarchical database used by the operating system to store configuration settings, hardware parameters, user preferences, and application properties.
- **Why it matters** — Many advanced settings, policy overrides, and system fixes can only be configured by editing the registry. A support engineer must know how to navigate the registry, backup and restore keys, and resolve corruption issues without causing system crashes.
- **Where you see this** — Resolving profile loading loops, configuring custom RDP ports, disabling startup programs, correcting file association errors, and enabling security policies.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Navigates key paths, backs up specific folders (exporting `.reg` files), and modifies basic DWORD values under instruction. |
| **L2** | Resolves profile corruption keys (`ProfileList`), resets app locks, and deploys registry fixes using command-line tools. |
| **L3** | Configures registry deployment templates via Group Policy Preferences (GPP), creates custom administrative templates (ADMX), and restores registry hives from offline backups. |

> [!tip] Seedha Simple Mein
> *Registry computer ka settings database hai. Windows aur apps ki saari configurations yahan store hoti hain. Yahan kuch galat delete karne se computer crash ho sakta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Windows Registry** is like the **DNA of the operating system** because...
> 
> - It dictates how everything looks, behaves, and functions.
> - Small, precise edits can cure an illness (fix an app bug), but reckless edits can cause the whole system to fail.

---
## 🔬 Technical Deep Dive

### 1. The 5 Registry Root Hives (Keys)

> [!info] Key Concept
> The registry is structured into five primary root keys, each starting with `HKEY` which store different parts of system configuration.

The registry is structured into five primary root keys, each starting with `HKEY`:

| Root Key name | Abbreviation | Core Contents |
|---|---|---|
| **`HKEY_CLASSES_ROOT`** | **HKCR** | File association mappings (e.g., `.txt` opens with Notepad) and OLE/COM data. |
| **`HKEY_CURRENT_USER`** | **HKCU** | Configuration settings for the currently logged-in user, loaded from `NTUSER.DAT`. |
| **`HKEY_LOCAL_MACHINE`** | **HKLM** | System-wide configuration settings for hardware, security, software, and drivers. |
| **`HKEY_USERS`** | **HKU** | Configuration settings for all user profiles loaded on the system. |
| **`HKEY_CURRENT_CONFIG`**| **HKCC** | Hardware profile configuration settings initialized during boot. |

> [!danger] Common Mistake
> Modifying values without taking a registry backup first, causing unrecoverable errors.

### 2. Registry Data Types
- **REG_SZ**: A standard Unicode text string value.
- **REG_DWORD (32-bit)**: A 32-bit numerical value, represented in Hexadecimal or Decimal (used for boolean flags, e.g., 0 = disabled, 1 = enabled).
- **REG_QWORD (64-bit)**: A 64-bit numerical value for larger integer storage.
- **REG_BINARY**: Raw binary data, commonly used for hardware configurations or driver states.
- **REG_EXPAND_SZ**: An expandable string containing system variables (e.g., `%SystemRoot%\System32`).

### 3. Registry Database Files (Hives)
The registry database is compiled from multiple binary files called **Hives**:
- System-wide hives are located in: `%systemroot%\System32\config\` (files named `SYSTEM`, `SOFTWARE`, `SAM`, `SECURITY`).
- User-specific hives are stored in: `C:\Users\<username>\NTUSER.DAT`.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 11 VM or physical machine with local administrator rights.

### Step 1: Export a Registry Key Configuration

```cmd
REM Export a registry key configuration to a backup file
reg export "HKLM\SOFTWARE\MyCustomApp" "C:\Backups\customapp.reg"
```

> [!success] Expected Output
> ```
> The operation completed successfully. The backup file customapp.reg is created.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `regedit` | Launch the graphical Registry Editor console | `regedit` |
| `reg export` | Export a registry key to a backup file | `reg export "HKLM\SOFTWARE" "C:\backup.reg"` |
| `Get-ItemProperty` | Query a registry key value using PowerShell | `Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Forgotten local admin password | Cannot log in | Boot to Windows PE, load offline hive `D:\Windows\System32\config\SYSTEM`, change `SetupType` to 2 and `CmdLine` to `cmd.exe`. Reboot, use cmd to `net user Administrator newpass`. Restore keys. |
| Classic Outlook stuck on "Loading Profile" | Profile registry configuration is corrupt | Rename `HKCU\Software\Microsoft\Office\16.0\Outlook\Profiles` to `Profiles.old`. Restart Outlook to build clean profile. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Resetting a Forgotten Supervisor Password (Offline Registry Mounting)

> [!example] Ticket
> "We have a critical local workstation, and the local administrator password has been lost. The machine is not joined to the domain, and we cannot log in."

**L1 Response:** Verify no other local accounts have admin rights.
**Escalation Trigger:** Password requires system bypass.
**L2 Resolution:** Booted from Windows installation USB. Accessed offline system registry. Loaded offline `SYSTEM` hive. Set `CmdLine` to `cmd.exe` and `SetupType` to `2`. Rebooted, reset password via cmd prompt, and restored registry default configurations.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between HKEY_LOCAL_MACHINE and HKEY_CURRENT_USER?
> **Answer:** HKEY_LOCAL_MACHINE (HKLM) stores system-wide configuration settings that apply to all users and hardware components on the machine. HKEY_CURRENT_USER (HKCU) stores configuration settings and preferences specific to the currently logged-in user, loaded dynamically from their NTUSER.DAT profile file.

==**Exam Tip:** Always export (backup) the target registry key to a `.reg` file before making any manual edits to prevent accidental system corruption.==

---
## 🔗 Related Notes

- [[02-Operating-Systems/03-Windows-OS/WIN-07 User Profiles|WIN-07 User Profiles]] — Details registry file mappings (`NTUSER.DAT`).
- [[02-Operating-Systems/03-Windows-OS/WIN-08 Windows Services|WIN-08 Windows Services]] — Details service configuration keys under HKLM.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-05 Group Policy — Complete Guide|WS-05 Group Policy — Complete Guide]] — Details registry policy deployments.
