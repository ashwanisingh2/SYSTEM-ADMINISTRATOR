---
tags: [desktop-support, windows-os, task-scheduler, automation, L1]
aliases: [scheduled-tasks, task-automation]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS**

`#complete` `#beginner` `#md-102`

# WIN-06: Task Scheduler

> [!abstract] Overview
> Yeh note Windows Task Scheduler ko cover karta hai, jo ek built-in service hai jisse scripts aur apps ko automatically run kiya ja sakta hai. Ek support engineer ke liye iska aana zaroori hai taaki tasks automate kar sake, maintenance scripts run kar sake, aur failed tasks troubleshoot kar sake.

---
## 🧠 Concept Overview

- **What it is** — Windows Task Scheduler is a built-in operating system service that allows administrators to automate the execution of scripts, applications, and commands based on defined triggers (time, events, startup) and conditions.
- **Why it matters** — Task Scheduler is used to automate maintenance scripts (backups, log cleanup), push software installations, and run diagnostic scripts. A support engineer must know how to create, monitor, and troubleshoot scheduled tasks to maintain system automation.
- **Where you see this** — Scheduling daily antivirus scans, automating log file rotations, configuring user logon scripts, pushing remote update scripts, and troubleshooting failed tasks in the task library.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Creates simple scheduled tasks, monitors execution history, and manually runs or stops tasks. |
| **L2** | Configures tasks to run under specific service accounts, manages tasks using PowerShell and `schtasks.exe`, and troubleshots access control failures. |
| **L3** | Deploys scheduled tasks across the domain using Group Policy Preferences (GPP), designs failover automation tasks, and monitors security logs. |

> [!tip] Seedha Simple Mein
> *Task Scheduler Windows ka alarm clock hai. Aap isme set kar sakte ho ki kis time ya kis action par kaunsa script ya program automatically chalna chahiye, taaki daily repetitive kaam khud ho jayein.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Task Scheduler** is like a **Smart Home Hub** because...
> 
> - **Trigger**: "When it is 7:00 PM" or "When the front door opens".
> - **Action**: "Turn on the living room lights" or "Play music".
> - **Condition**: "Only if the sun has already set".

---
## 🔬 Technical Deep Dive

### 1. Task Components

> [!info] Key Concept
> A scheduled task consists of three primary configuration blocks: Triggers, Actions, and Conditions.

A scheduled task consists of three primary configuration blocks:
- **Triggers**: Define *when* the task runs (e.g., at a specific time, on system startup, when a user logs in, or on a specific Event ID).
- **Actions**: Define *what* the task does (e.g., launching an executable, running a PowerShell script, or sending an email alert).
- **Conditions & Settings**: Define additional rules (e.g., run only if the computer is on AC power, stop the task if it runs longer than 3 days, or restart the task if it fails).

> [!danger] Common Mistake
> Running PowerShell scripts directly instead of calling `PowerShell.exe` with the script as an argument.

### 2. Security Contexts & Execution Options
- **Run only when user is logged on**: Runs within the user's interactive session. Accesses network resources using the user's active credentials.
- **Run whether user is logged on or not**: Runs in a background session. Requires the user's password to be stored in the Windows Credential Vault (unless running as a service account).
- **NT AUTHORITY\SYSTEM**: Runs with highest local system privileges, bypassing user account limitations. Cannot access network shares directly without computer account validation.
- **Run with highest privileges**: Forces the task to run with administrative elevation, bypassing User Account Control (UAC) prompts.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 11 VM or physical machine with local administrator rights.

### Step 1: Create a Scheduled Task via PowerShell

```powershell
# Create a trigger for daily execution at 3:00 AM
$Trigger = New-ScheduledTaskTrigger -Daily -At 3am

# Define the action to run a custom cleanup PowerShell script
$Action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-NoProfile -WindowStyle Hidden -File C:\Scripts\cleanup.ps1"

# Register the new scheduled task under the local SYSTEM account
Register-ScheduledTask -TaskName "DailyCleanup" -Trigger $Trigger -Action $Action -User "NT AUTHORITY\SYSTEM" -Force
```

> [!success] Expected Output
> ```
> The task is successfully registered and visible in the Task Scheduler console under the specified name.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `taskschd.msc` | Open the Task Scheduler Console | `taskschd.msc` |
| `schtasks /query` | Query the status of all configured scheduled tasks | `schtasks /query /fo LIST /v` |
| `Register-ScheduledTask` | PowerShell cmdlet to create tasks | `Register-ScheduledTask ...` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Automated backup script fails with error code 0x1 or 0x80070002 | Script was executed directly without PowerShell interpreter, or 'Start in' directory was blank. | Edit action: Program/script `PowerShell.exe`, Arguments `-ExecutionPolicy Bypass -File C:\Scripts\backup.ps1`, Start in `C:\Scripts\`. |
| Task fails to run after user password reset (Event 110/4625) | Outdated credentials in Windows Credential Vault | Re-enter updated password in task properties or use a service account (gMSA). |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Automated Backup Script Fails

> [!example] Ticket
> "The daily local backup task is showing as completed in Task Scheduler, but the 'Last Run Result' column shows error code 0x1, and no backup file was created."

**L1 Response:** Check the Last Run Result code and script path.
**Escalation Trigger:** Code continues to fail despite script running fine manually.
**L2 Resolution:** Found that the task attempted to run `.ps1` directly instead of calling `PowerShell.exe`. Corrected the Action to use PowerShell with the script path in arguments and specified the "Start in" working directory. Issue resolved.

---
## 🎤 Interview Questions

> [!question] Q1: How do you verify if a scheduled task completed successfully?
> **Answer:** I open the Task Scheduler console, select the task, and check the 'Last Run Result' column. A code of 0 or 0x0 indicates the task completed successfully. Any other code (e.g., 0x1 or 0x80070002) indicates a failure.

==**Exam Tip:** Exit code 267011 (or 0x41301) indicates the task was terminated or stopped by the user or an administrator (e.g. exceeded execution time limit).==

---
## 🔗 Related Notes

- [[02-Operating-Systems/03-Windows-OS/Registry|Registry]] — Details task configurations database keys.
- [[02-Operating-Systems/03-Windows-OS/Event-Viewer|Event Viewer]] — Details task event auditing.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/Basics|PowerShell Basics]] — Details scripting automation foundations.
