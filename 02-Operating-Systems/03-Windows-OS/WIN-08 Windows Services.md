---
tags: [desktop-support, windows-os, services, sc-exe, L1]
aliases: [windows-services, service-management]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS**

`#complete` `#beginner` `#md-102`

# WIN-08: Windows Services

> [!abstract] Overview
> Yeh note Windows Services ko cover karta hai, jo background applications hain aur OS ya third-party apps ke liye zaroori hain. Ek support engineer ko inko start/stop karna, troubleshoot karna aur hung processes ko force-kill karna aana chahiye.

---
## 🧠 Concept Overview

- **What it is** — Windows Services are long-running executable background applications that run in their own Windows sessions, operating without a user interface or interactive login.
- **Why it matters** — Crucial operating system features (like print spooling, network connections, and remote access) and third-party software (antivirus, backup agents) run as services. A support engineer must know how to start, stop, configure, and troubleshoot services to restore app functionality.
- **Where you see this** — Restarting a hung Print Spooler service, enabling the Remote Registry service for remote diagnostics, setting service recovery behaviors, and killing services stuck in a "Stopping" loop.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Starts, stops, and restarts services via the Services GUI or command line, and changes startup types. |
| **L2** | Troubleshoots service dependencies, manages services using PowerShell and `sc.exe`, and configures recovery actions. |
| **L3** | Scripts automated service monitoring, creates custom service files, and configures security descriptors to restrict service access. |

> [!tip] Seedha Simple Mein
> *Windows Services computer ke invisible mazdoor hain. Yeh background mein network, printing aur updates ka kaam karte hain bina kisi interface ke. Agar print nahi ho raha toh ho sakta hai Print Spooler service ruki ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Windows Services** are like the **backstage crew** at a theater because...
> 
> - The audience never sees them.
> - They handle the lights, sound, and curtains.
> - If the backstage crew stops working, the entire show (the operating system) crashes.

---
## 🔬 Technical Deep Dive

### 1. Service Security Accounts

> [!info] Key Concept
> Services run in the background under designated security contexts, like Local System, Network Service, or Local Service.

Services run in the background under designated security contexts (service accounts):
- **Local SYSTEM (NT AUTHORITY\SYSTEM)**: The most privileged account. Has complete access to local system resources and acts as the computer on the network.
- **Network Service (NT AUTHORITY\NETWORKSERVICE)**: Has limited local rights, but can access network resources using the computer account's credentials.
- **Local Service (NT AUTHORITY\LOCALSERVICE)**: Has limited local rights and acts as an anonymous user on the network.
- **Dedicated User Account**: A specific domain or local account configured to run a service, used to enforce least-privilege security policies.

> [!danger] Common Mistake
> Disabling critical services like `EventLog` or `RpcSs` which will cause Windows to crash on boot.

### 2. Startup Types
- **Automatic**: The service starts during OS boot.
- **Automatic (Delayed Start)**: The service starts shortly after boot (typically 2 minutes delay) after critical services have loaded, reducing boot congestion.
- **Manual**: The service starts only when called by a user or another application.
- **Disabled**: The service cannot be started.

### 3. Critical System Services Reference
| Service Name | Display Name | Core Function | Impact if Stopped |
|---|---|---|---|
| **`Spooler`** | Print Spooler | Manages print queues and jobs | Users cannot print or detect printers. |
| **`TermService`** | Remote Desktop Services | Provides RDP connection support | Remote administration connections fail. |
| **`wuauserv`** | Windows Update | Manages update delivery | The system cannot scan for or install updates. |
| **`EventLog`** | Windows Event Log | Records system and app events | The system cannot write logs, dependencies fail. |

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 11 VM or physical machine with local administrator rights.

### Step 1: Force Kill and Restart a Service

```powershell
# Find PID of the hung service
Get-CimInstance -ClassName Win32_Service -Filter "Name='Spooler'"

# Kill process by PID (e.g., 1424)
taskkill /F /PID 1424

# Start the service again
Start-Service -Name "Spooler"
```

> [!success] Expected Output
> ```
> SUCCESS: The process with PID 1424 has been terminated.
> Service Spooler successfully started.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `services.msc` | Open the Services Management Console | `services.msc` |
| `sc qc` | Query service details and configuration via CMD | `sc qc Spooler` |
| `Get-Service` | PowerShell cmdlet to list service statuses | `Get-Service | Where-Object Status -eq "Stopped"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Service stuck in "Stopping" | Corrupt file blocking shutdown | Find Process ID via `Get-CimInstance`, use `taskkill /F /PID [PID]`, clear corrupt files (e.g., spool folder), then start service. |
| Service fails to start (Event ID 7000) | Logon Failure / Outdated Password | Update the service account password in the "Log On" tab of the service properties, or unlock the AD account. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Print Spooler Stuck in "Stopping" State

> [!example] Ticket
> "I tried to print a document and it got stuck. The IT helpdesk told me to restart the Print Spooler, but in the services console, the Spooler is stuck on 'Stopping' and all options are grayed out."

**L1 Response:** Check service status and attempt basic restart if possible.
**Escalation Trigger:** Service UI grayed out and stuck on "Stopping".
**L2 Resolution:** Used `Get-CimInstance` to find Spooler PID. Executed `taskkill /F /PID <PID>` to kill the hung process. Cleared corrupt `.SPL` and `.SHD` files in `C:\Windows\System32\spool\PRINTERS`. Restarted Spooler service.

---
## 🎤 Interview Questions

> [!question] Q1: How do you restart a Windows service that is stuck in a "Stopping" state?
> **Answer:** I open a command prompt or PowerShell as Administrator, locate the service's Process ID (PID) using `Get-CimInstance` or `sc queryex`, and force-kill the process using `taskkill /F /PID [PID]`. I clear any stuck files (like corrupt print jobs), then start the service normally.

==**Exam Tip:** Check the **Dependencies** tab before troubleshooting a service failure. A service won't start if its required parent service is stopped or disabled.==

---
## 🔗 Related Notes

- [[02-Operating-Systems/03-Windows-OS/WIN-04 Registry|WIN-04 Registry]] — Details service configuration keys under HKLM.
- [[02-Operating-Systems/03-Windows-OS/WIN-03 Event Viewer|WIN-03 Event Viewer]] — Details Event IDs used to audit service starts.
- [[03-Identity-and-Core-Services/06-Active-Directory/Users-and-Groups|Users and Groups]] — Details service account configurations.
