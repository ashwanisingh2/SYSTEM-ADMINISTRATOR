---
tags: [desktop-support, windows-os, task-scheduler, automation, L1]
aliases: [scheduled-tasks, task-automation]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# Task Scheduler

---

## Concept Overview
- **What it is**: Windows Task Scheduler is a built-in operating system service that allows administrators to automate the execution of scripts, applications, and commands based on defined triggers (time, events, startup) and conditions.
- **Why it matters for a support engineer**: Task Scheduler is used to automate maintenance scripts (backups, log cleanup), push software installations, and run diagnostic scripts. A support engineer must know how to create, monitor, and troubleshoot scheduled tasks to maintain system automation.
- **Where you encounter this in real job**: Scheduling daily antivirus scans, automating log file rotations, configuring user logon scripts, pushing remote update scripts, and troubleshooting failed tasks in the task library.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Creates simple scheduled tasks, monitors execution history, and manually runs or stops tasks.
  - **L2**: Configures tasks to run under specific service accounts, manages tasks using PowerShell and `schtasks.exe`, and troubleshoots access control failures.
  - **L3**: Deploys scheduled tasks across the domain using Group Policy Preferences (GPP), designs failover automation tasks, and monitors security logs for unauthorized task creations.

---

## Technical Deep Dive

### 1. Task Components
A scheduled task consists of three primary configuration blocks:
- **Triggers**: Define *when* the task runs (e.g., at a specific time, on system startup, when a user logs in, or on a specific Event ID).
- **Actions**: Define *what* the task does (e.g., launching an executable, running a PowerShell script, or sending an email alert).
- **Conditions & Settings**: Define additional rules (e.g., run only if the computer is on AC power, stop the task if it runs longer than 3 days, or restart the task if it fails).

### 2. Security Contexts & Execution Options
- **Run only when user is logged on**: Runs within the user's interactive session. Accesses network resources using the user's active credentials.
- **Run whether user is logged on or not**: Runs in a background session. Requires the user's password to be stored in the Windows Credential Vault (unless running as a service account).
- **NT AUTHORITY\SYSTEM**: Runs with highest local system privileges, bypassing user account limitations. Cannot access network shares directly without computer account validation.
- **Run with highest privileges**: Forces the task to run with administrative elevation, bypassing User Account Control (UAC) prompts.

---

## Commands & Syntax

### PowerShell
```powershell
# Create a trigger for daily execution at 3:00 AM
$Trigger = New-ScheduledTaskTrigger -Daily -At 3am

# Define the action to run a custom cleanup PowerShell script
$Action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-NoProfile -WindowStyle Hidden -File C:\Scripts\cleanup.ps1"

# Register the new scheduled task under the local SYSTEM account
Register-ScheduledTask -TaskName "DailyCleanup" -Trigger $Trigger -Action $Action -User "NT AUTHORITY\SYSTEM" -Force
```

### CMD / Run Box
```cmd
REM Create a scheduled task to run disk cleanup weekly
schtasks /create /tn "WeeklyCleanup" /tr "cleanmgr.exe /sagerun:1" /sc weekly /d SUN /st 01:00
REM Query the status of all configured scheduled tasks
schtasks /query /fo LIST /v
```

### GUI Path
> Start -> Run -> type `taskschd.msc` (Task Scheduler Console)
> Or: Computer Management -> System Tools -> **Task Scheduler**

### Important Registry Paths
```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks\
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\
```

### Key Event IDs
Task Scheduler audits events in the log path: `Applications and Services Logs\Microsoft\Windows\TaskScheduler\Operational`

| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 100 | Task started by user or trigger | Operational Log |
| 102 | Task completed successfully | Operational Log |
| 110 | Task failed to start | Operational Log |
| 201 | Task execution failed (returns error exit code) | Operational Log |

---

## Real-World Scenarios

### Scenario 1: Automated Backup Script Fails with Error Code `0x1` or `0x80070002`
**User Complaint:** "The daily local backup task is showing as completed in Task Scheduler, but the 'Last Run Result' column shows error code 0x1, and no backup file was created."
**Your First 3 Checks:**
1. Check the **Last Run Result** column in the Task Scheduler console.
2. Verify the path configuration of the script in the **Actions** tab.
3. Check the Task Scheduler Operational logs for Event ID 201 details.
**Diagnosis Steps:**
1. Open Task Scheduler and locate the `BackupTask` entry.
   - Last Run Result: `0x80070002` (File not found) or `0x1` (Function incorrect/Script error).
2. Open the task properties -> **Actions** tab.
   - Action: Start a program -> Program/script: `C:\Scripts\backup.ps1`.
   - Windows cannot run `.ps1` files directly from the script line; it requires the PowerShell interpreter.
3. Open the task settings. The **Start in (optional)** directory field is blank, causing relative file paths in the script to fail.
**Root Cause:** The task attempted to run a PowerShell script directly instead of calling the PowerShell executable, and the working directory was undefined.
**Fix:** Edit the Action settings:
- Program/script: `PowerShell.exe`
- Add arguments: `-ExecutionPolicy Bypass -File C:\Scripts\backup.ps1`
- Start in: `C:\Scripts\`
**Prevention:** Always use full executable paths and execution policies when launching scripts via Task Scheduler.
**Ticket Close Note:** "Corrected the task action parameters to launch the script using PowerShell.exe. Verified successful execution with exit code 0x0. Closing ticket."

### Scenario 2: Maintenance Task Fails to Run After User Password Reset
**User Complaint:** "The server log consolidation task has stopped running. The Task Scheduler history shows Event ID 110: Task failed to start."
**Your First 3 Checks:**
1. Check the System event log for service logon failures.
2. Verify the user account configured to run the task.
3. Check if the task is configured to "Run whether user is logged on or not."
**Diagnosis Steps:**
1. Open Event Viewer -> Security log -> Filter by Event ID `4625` (Logon Failure).
   - Logged: `Logon Type 4` (Batch logon). User: `domain\administrator`.
2. Inspect the Task Scheduler task properties.
   - Security options: Set to run as `domain\administrator` with "Run whether user is logged on or not" enabled.
3. Verify if the account password was changed recently.
   - The administrator password was updated yesterday.
**Root Cause:** The task failed to run because the stored credentials in the Windows Credential Vault were outdated, causing a logon failure (Event ID 4625 / 7000).
**Fix:** Edit the task properties, click **OK**, and enter the updated password when prompted to refresh the stored credentials.
**Prevention:** Run automated maintenance tasks using dedicated service accounts with non-expiring passwords, or use the local `SYSTEM` account if network access is not required.
**Ticket Close Note:** "Updated the stored credentials for the task owner. Verified successful execution. Recommended migrating the task to a service account. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Configure scheduled tasks to run as `NT AUTHORITY\SYSTEM` or local `Administrator` if the task script only needs to read or copy files within a user's directory.
> - If an attacker gains write access to that script file, they can inject malicious code that will execute with full administrative privileges, compromising the system (Privilege Escalation).
> - Enforce least-privilege access when assigning service accounts.

> [!warning] Common Trap
> - Creating a task configured to run a script when the system starts, without configuring a startup delay.
> - If the task runs before essential network services (like DHCP or domain connections) have fully loaded, the script will fail to access network paths.
> - Configure a **Delay task for** (e.g., 1 minute) setting in the trigger properties.

> [!tip] Senior Engineer Tip
> - When troubleshooting tasks that run silently in the background, redirect script outputs and errors to a local log file (e.g., `backup.ps1 > C:\Logs\backup_err.txt 2>&1`) to capture details that Task Scheduler does not log.

> [!success] Verification Steps
> - Run the command: `Get-ScheduledTask -TaskName "DailyCleanup" | Get-ScheduledTaskInfo` to check task execution details.
> - Verify that the **LastTaskResult** displays `0` (indicating success).

> [!question] Interview Alert
> - "What is the meaning of the Task Scheduler exit code 267011?"
> - Answer: "Exit code `267011` (or Hex `0x41301`) indicates that the task was terminated or stopped by the user or an administrator. It commonly occurs when a task exceeds its maximum execution time limit and is killed by the Task Scheduler daemon."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Running scripts without execution policies | PowerShell blocks unsigned scripts | Add `-ExecutionPolicy Bypass` to the task arguments. |
| Forgetting network share permissions for SYSTEM | SYSTEM account has no network rights | Use a dedicated domain service account or computer account validation for network shares. |
| Running tasks on battery power | Default condition blocks execution | Uncheck **Start the task only if the computer is on AC power** in the settings tab. |

---

## Lab Exercise

**Objective:** Create a scheduled task using PowerShell that runs a cleanup script under the SYSTEM account, configure triggers, and verify execution status.
**Time Required:** 20 minutes
**Environment Needed:** A Windows 11 VM or physical machine with local administrator rights.
**Pre-requisites:** Administrative access.

**Steps:**
1. Create a script directory and test file: Open PowerShell as Administrator.
   ```powershell
   New-Item -ItemType Directory -Path "C:\LabScripts" -Force
   Set-Content -Path "C:\LabScripts\cleanup.ps1" -Value "Remove-Item -Path C:\LabScripts\test.tmp -Force"
   New-Item -ItemType File -Path "C:\LabScripts\test.tmp" -Force
   ```
2. Define the Task Action:
   ```powershell
   $Action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File C:\LabScripts\cleanup.ps1"
3. Define the Task Trigger: Configure the task to run daily at 11:00 PM.
   ```powershell
   $Trigger = New-ScheduledTaskTrigger -Daily -At 11pm
   ```
4. Register the Task: Register the task under the local SYSTEM account.
   ```powershell
   Register-ScheduledTask -TaskName "LabCleanupTask" -Action $Action -Trigger $Trigger -User "NT AUTHORITY\SYSTEM" -Force
   ```
5. Run the Task: Start execution manually:
   ```powershell
   Start-ScheduledTask -TaskName "LabCleanupTask"
   ```
6. Verification: Check if the test file was deleted.
   ```powershell
   Test-Path -Path "C:\LabScripts\test.tmp"
   ```
   - Expected output: `False` (confirming the task successfully ran the script and deleted the file).

**Success Criteria:** The task runs successfully, deletes the file, and displays a last task result code of `0` in the console.
**Common Failures:** Failed execution if script path names contain spaces and are not enclosed in quotes.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What are triggers and actions in Task Scheduler?**
A: Triggers are the events or conditions that cause a scheduled task to run (e.g., a time schedule or user login). Actions are the specific commands, scripts, or executables that the task executes when it is triggered.

**Q: How do you verify if a scheduled task completed successfully?**
A: I open the Task Scheduler console, select the task, and check the **Last Run Result** column. A code of `0` or `0x0` indicates the task completed successfully. Any other code (e.g., `0x1` or `0x80070002`) indicates a failure.

### Intermediate (L2 Level)
**Q: Explain the difference between running a task as local SYSTEM versus running it under a user account.**
A: Running a task as local SYSTEM grants it full administrative privileges on the local machine, allowing it to modify system files and services without UAC prompts, but restricts its direct access to external network shares. Running a task under a user account restricts its privileges to that user's rights, but allows it to access network shares using the user's domain credentials.

### Advanced (L3/Senior Level)
**Q: A critical reporting task must run on a database server daily. It requires domain access, but security policies block password storage in the local task cache. Explain your resolution strategy.**
A:
- **Situation**: A reporting task required domain access, but local security policies blocked storing user passwords in Task Scheduler.
- **Task**: I needed to execute the task securely without storing credentials locally.
- **Action**: I configured the task to run under the **Group Managed Service Account (gMSA)** context. I registered the gMSA on the domain controller, installed the AD service account on the database server, and configured the task to run under the gMSA name (e.g., `domain\gmsa-report$`), which allows Windows to manage authentication automatically without local password storage.
- **Result**: The task executed securely without storing passwords locally, complying with the security policy.

### HR / Behavioral
**Q: Tell me about a time you automated a manual repetitive task. What was the impact?**
A: Our team had to manually clear temporary log files every Friday, which was often forgotten. I wrote a PowerShell script to clean the directories, set it to run weekly as a scheduled task under the SYSTEM account, and verified it reclaimed 50 GB of disk space weekly.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The Windows utility used to automate scripts and tasks based on triggers.
> **Why**: Critical for automating administrative tasks, maintenance, and diagnostics.
> **How**: Define triggers, actions, and security contexts, and manage tasks using PowerShell.
> **Command**: `Register-ScheduledTask`
> **Interview Answer Starter**: "In my experience, scheduled task troubleshooting requires checking the Last Run Result code and verifying the security context on the Log On tab..."

**Key Numbers to Remember:**
- Successful exit code: 0x0
- Default timeout Event ID: Event ID 201
- Local System account name: NT AUTHORITY\SYSTEM

**3 Things Interviewer Wants to Hear:**
1. Using service accounts (gMSA) instead of user accounts for tasks
2. Restricting task privileges (least-privilege access)
3. Troubleshooting exit codes (e.g., 0x1, 0x80070002)

---

## Related Notes
- [[02-Operating-Systems/03-Windows-OS/Registry|Registry]] — Details task configurations database keys.
- [[02-Operating-Systems/03-Windows-OS/Event-Viewer|Event Viewer]] — Details task event auditing.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/Basics|PowerShell Basics]] — Details scripting automation foundations.

---

## Tags
#desktop-support #windows-os #task-scheduler #automation #L1 #interview-topic #lab-complete #daily-use
