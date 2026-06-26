---
tags: [desktop-support, windows-os, sfc, dism, L1]
aliases: [system-repair, file-integrity]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS**

`#complete` `#beginner` `#md-102`

# WIN-05: SFC and DISM

> [!abstract] Overview
> Yeh note SFC aur DISM commands ke baare mein hai jo corrupt system files aur component store ko repair karne mein use hote hain. Har support engineer ko inka sahi use aana chahiye taaki OS stability issues aur blue screens fix kiye ja sakein.

---
## 🧠 Concept Overview

- **What it is** — System File Checker (SFC) and Deployment Image Servicing and Management (DISM) are built-in Windows command-line administrative utilities used to verify, repair, and restore the integrity of operating system files and the local Windows Component Store.
- **Why it matters** — Operating system files can become corrupted due to power outages, malware, or software conflicts, causing system instability and update failures. A support engineer must know when and how to run SFC and DISM to repair system files.
- **Where you see this** — Resolving Windows Update errors, fixing system file corruption warnings, repairing broken OS features, and diagnosing Blue Screen of Death (BSOD) stability issues.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Runs basic SFC scans (`sfc /scannow`), checks basic logs, and restarts systems. |
| **L2** | Resolves SFC repair failures using DISM online and offline commands, and extracts files from local install images. |
| **L3** | Configures deployment images, manages Windows Component Store cleanup policies, and writes automated deployment verification scripts. |

> [!tip] Seedha Simple Mein
> *SFC aur DISM Windows ke in-built doctor aur surgeon hain. Agar computer ajeeb behave kare ya files corrupt ho jaayein, toh DISM component store (database) repair karta hai aur SFC corrupt OS files replace karta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Windows System Files** are like a **library of books** because...
> 
> - **SFC (System File Checker)** is the librarian walking the aisles. If they find a torn book (corrupt file), they go to the basement storage (`WinSxS` Component Store) to grab a fresh copy.
> - **DISM** is the restoration crew. If the basement storage itself catches fire or has torn books, DISM contacts the publishing house (Microsoft Update Servers) to download fresh copies for the basement.
> - That’s why you always call DISM *before* SFC when there’s a major problem.

---
## 🔬 Technical Deep Dive

### 1. SFC vs. DISM: The Logical Order of Repair

> [!info] Key Concept
> Always run **DISM first** to repair the source database, and then run **SFC second** to repair the active operating system files.

SFC and DISM target different areas of the Windows operating system:
- **SFC (System File Checker)**: Compares active operating system files against a cache of healthy backup files stored in the **Windows Component Store** (`C:\Windows\WinSxS`). If it detects a mismatch, it replaces the active file with the healthy version.
- **DISM (Deployment Image Servicing and Management)**: Verifies and repairs the integrity of the **Windows Component Store** (`C:\Windows\WinSxS`) itself. If the component store is corrupt, SFC has no healthy files to use for repairs.

```
[ Microsoft Update Servers ]
            |
            v  (DISM /RestoreHealth)
[ Windows Component Store ]  <-- C:\Windows\WinSxS
            |
            v  (SFC /scannow)
[ Active System Files ]      <-- C:\Windows\System32
```

> [!danger] Common Mistake
> Running SFC first, letting it fail because the component store is corrupt, and giving up without running DISM.

### 2. The Windows Component Store (`WinSxS`)
The `WinSxS` (Windows Side-by-Side) folder hosts multiple versions of system files, enabling compatibility and update rollbacks. It is the database that Windows uses to restore missing or corrupt files.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 11 VM or physical machine with local administrator rights.

### Step 1: Repair Component Store and Run SFC

```cmd
REM Repair the local component store using Windows Update servers as the source
dism /online /cleanup-image /restorehealth

REM Scan and repair system file integrity
sfc /scannow
```

> [!success] Expected Output
> ```
> The restore operation completed successfully.
> Windows Resource Protection found corrupt files and successfully repaired them.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `sfc /scannow` | Scan and repair system file integrity | `sfc /scannow` |
| `dism /online /cleanup-image /restorehealth` | Repair the local component store via Windows Update | `dism /online /cleanup-image /restorehealth` |
| `Repair-WindowsImage -Online -ScanHealth` | Run a silent DISM scan using PowerShell | `Repair-WindowsImage -Online -ScanHealth` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| SFC Fails: "WRP found corrupt files but was unable to fix some of them" | Local Windows Component Store is corrupt | Run `dism /online /cleanup-image /restorehealth` to repair WinSxS, then rerun `sfc /scannow`. |
| DISM Fails with Error 0x800f081f (Source Files Not Found) | Offline workstation cannot reach Windows Update | Mount a Windows ISO and run DISM pointing to the offline `install.wim` source: `dism /online /cleanup-image /restorehealth /source:wim:D:\sources\install.wim:1 /limitaccess`. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: SFC Fails to Repair Files

> [!example] Ticket
> "My computer is freezing randomly. I ran sfc /scannow as instructed, but it returned a warning saying it found corrupt files but was unable to fix them."

**L1 Response:** Run basic disk checks and restart the PC.
**Escalation Trigger:** SFC continues to fail.
**L2 Resolution:** Checked `CBS.log` which showed "Source file in store is also corrupted." Ran DISM `/RestoreHealth` to repair the Component Store using Microsoft Update, followed by `sfc /scannow` which then completed successfully.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between SFC and DISM, and in what order should you run them?
> **Answer:** SFC compares active operating system files against the local Windows Component Store and replaces corrupt files. DISM repairs the Component Store itself by downloading healthy files from Microsoft Update. I always run DISM first to ensure the component store is healthy, and then run SFC second to repair the active OS files.

==**Exam Tip:** Error code 0x800f081f means the source files could not be found. This occurs when DISM cannot reach Windows Update servers (e.g., system is offline).==

---
## 🔗 Related Notes

- [[02-Operating-Systems/03-Windows-OS/Windows-Update|Windows Update]] — Utilizes the component store for updates.
- [[02-Operating-Systems/03-Windows-OS/Event-Viewer|Event Viewer]] — Audits component store errors.
- [[06-Career-Growth/13-Lab-Projects/Home-lab|Home lab]] — Setting up VM test environments.
