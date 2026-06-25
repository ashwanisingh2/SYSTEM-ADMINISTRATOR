---
tags: [desktop-support, powershell, scripting, automation, L3]
aliases: [advanced-powershell, try-catch-guide, scheduled-tasks-powershell]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

# PowerShell Automation and Scripting

---

## Concept Overview
- **What it is**: PowerShell Automation and Scripting involves combining cmdlets into structured, logical script files (`.ps1`) to perform complex administrative tasks. It utilizes advanced functions, looping constructs, error handling (`Try-Catch` blocks), and headless automation engines (Windows Task Scheduler).
- **Why it matters for a support engineer**: A support engineer grows into a system administrator by moving from manual commands to automated solutions. Writing resilient scripts that handle errors gracefully, log output values, and run on schedules prevents outages and eliminates manual labor.
- **Where you encounter this in real job**: Scheduling daily disk space checks, writing custom cleanup scripts, handling database service failures automatically, and logging script execution telemetry.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Reviews scheduled task runs, checks script execution logs, and runs predefined scripts.
  - **L2**: Modifies existing scripts, writes standard loops to process user lists, and registers basic scheduled tasks.
  - **L3**: Builds advanced tool modules, writes robust scripts with complete error handling, configures secure credential storage, and manages enterprise automation servers.

---

## Technical Deep Dive

### 1. Looping Constructs
Loops allow you to execute a block of code repeatedly for each item in a collection (such as a list of users or computer names):
- **`foreach`**: The most common loop. Processes items one-by-one.
  - *Example*: `foreach ($User in $UserList) { Reset-Password $User }`
- **`for`**: Counter-based loop. Used when you need to loop a specific number of times.
- **`while`**: Condition-based loop. Executes as long as a specified test returns `$true` (e.g., waiting for a VM status to change from stopping to stopped).

### 2. Error Handling (Try-Catch Blocks)
By default, when a PowerShell cmdlet encounters an error (like file not found), it prints a red warning on screen but **continues executing the rest of the script**. This is a non-terminating error.
- **Terminating Errors**: To catch and handle an error, you must force it to become a terminating error by setting **`-ErrorAction Stop`**.
- **`Try-Catch`**: 
  - **`Try`**: The code block you want to test.
  - **`Catch`**: The code block that runs *only* if an error occurs inside the Try block. The error message is stored in the special variable **`$_`**.

```
[Start Script]
      |
      v
    [Try] ---> (No Errors) ---> [Rest of Script]
      |
      +---> (Error Occurs with -ErrorAction Stop)
      |
      v
   [Catch] ---> [Logs Error via $_] ---> [Clean Exit]
```

### 3. Advanced Functions & CmdletBinding
An advanced function behaves like a compiled native cmdlet:
- **`[CmdletBinding()]`**: Enables standard cmdlet behaviors, such as support for the `-Verbose` and `-ErrorAction` parameters.
- **Parameter Validation**: You can enforce rules on parameters (e.g., setting `Mandatory=$true` or validating input formats using `ValidatePattern`).

---

## Commands & Syntax

### Code Structures (Loops & Try-Catch)
```powershell
# 1. LOOPING CONSTRUCT (Wait for a service to stop)
$Service = Get-Service -Name "Spooler"
while ($Service.Status -ne "Stopped") {
    Write-Output "Waiting for spooler to stop..."
    Start-Sleep -Seconds 5
    $Service = Get-Service -Name "Spooler"
}

# 2. TRY-CATCH ERROR HANDLING
try {
    # -ErrorAction Stop is mandatory to force catching non-terminating errors
    Remove-Item -Path "C:\Windows\Temp\LockedFile.txt" -ErrorAction Stop
    Write-Output "File deleted successfully."
} catch {
    # $_ holds the active error object
    Write-Warning "Failed to delete file. Error details: $_"
}
```

### Advanced Function Template
```powershell
function Get-DiskSpaceReport {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)]
        [string]$ComputerName,

        [Parameter(Mandatory=$false)]
        [int]$ThresholdGB = 10
    )

    process {
        Write-Verbose "Querying disk space on $ComputerName..."
        try {
            $Disks = Get-CimInstance -ComputerName $ComputerName -ClassName Win32_LogicalDisk -Filter "DriveType=3" -ErrorAction Stop
            foreach ($Disk in $Disks) {
                $FreeGB = [math]::round($Disk.FreeSpace / 1GB, 2)
                if ($FreeGB -lt $ThresholdGB) {
                    [PSCustomObject]@{
                        Computer     = $ComputerName
                        Drive        = $Disk.DeviceID
                        FreeSpace_GB = $FreeGB
                        Alert        = "Low Disk Space!"
                    }
                }
            }
        } catch {
            Write-Error "Failed to query $ComputerName. Error: $_"
        }
    }
}
```

### Scheduling Tasks via PowerShell
```powershell
# Define the action (what script to run)
$Action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-ExecutionPolicy Bypass -File C:\Scripts\DiskAudit.ps1"

# Define the trigger (when to run - e.g., daily at 2:00 AM)
$Trigger = New-ScheduledTaskTrigger -Daily -At "2:00 AM"

# Define execution settings (Run as SYSTEM for full privileges)
$Principal = New-ScheduledTaskPrincipal -UserId "NT AUTHORITY\SYSTEM" -LogonType ServiceAccount -RunLevel Highest

# Register the Scheduled Task in Windows
Register-ScheduledTask -TaskName "DailyDiskAudit" -Action $Action -Trigger $Trigger -Principal $Principal -Description "Audits disk space usage daily."
```

---

## Real-World Scenarios

### Scenario 1: File-Cleanup Script Fails Silently and Fills Up Disk Space
**User Complaint:** A disk alert triggers: *"The C: drive on our application server 'VM-AppServer' is at 99% capacity. A scheduled cleanup script was supposed to delete old temporary files, but it has not been running correctly."*
**Your First 3 Checks:**
1. Check the Task Scheduler history for the cleanup task.
2. Verify if the script crashes on locked files, halting execution.
3. Check if error logs are generated by the script.
**Diagnosis Steps:**
1. Log in to `VM-AppServer`. Open the cleanup script: `C:\Scripts\cleanup.ps1`.
   `Get-ChildItem "C:\App\Temp" | Remove-Item`
2. Run the command manually in PowerShell.
   - Output: `Remove-Item : Cannot remove item C:\App\Temp\active.log: Access Denied.`
3. *Why did it fail?* The cleanup script encountered a locked log file. Because there was no error handling, the script threw a non-terminating error and stopped processing subsequent temporary files, leaving gigabytes of old files untouched.
**Root Cause:** The script lacked error handling, causing it to fail silently when encountering locked files.
**Fix:**
1. Modify the script to use a `Try-Catch` block with `-ErrorAction Stop`, letting it skip locked files and continue processing others:
```powershell
$Files = Get-ChildItem -Path "C:\App\Temp" -File

foreach ($File in $Files) {
    try {
        # Force Stop on error to trigger Catch block
        Remove-Item -Path $File.FullName -Force -ErrorAction Stop
    } catch {
        # Log the locked file details and continue the loop
        Write-Verbose "Skipped locked file $($File.Name). Reason: $_"
    }
}
```
2. Run the modified script. It skips `active.log` and successfully deletes the other 20GB of stale files.
**Prevention:** Always implement Try-Catch loops when deleting or modifying files in shared folders.
**Ticket Close Note:** "Added Try-Catch logic to the cleanup script. Skips locked files and purges stale cache data. Resolved disk space alert. Closed."

### Scenario 2: Automatically Restarting the Print Spooler on Services Failure
**User Complaint:** Helpdesk reports: *"Users are complaining they cannot print. The Print Spooler service on the print server 'SRV-PRINT-01' crashes repeatedly. We have to log in and start it manually every time."*
**Your First 3 Checks:**
1. Check the System event log on the server to find why the service crashes.
2. Verify the Spooler service recovery options properties.
3. Create a PowerShell script to monitor the service and register it as a Task.
**Diagnosis Steps:**
1. Open Event Viewer on `SRV-PRINT-01`. Go to the System log.
   - Find **Event ID 7031**: *"The Print Spooler service terminated unexpectedly."*
2. Construct a monitoring script `WatchSpooler.ps1`:
   - Checks if Spooler is stopped.
   - If stopped, restarts it and logs the event.
3. Register the script to run under Task Scheduler, triggered whenever Event ID 7031 is logged.
**Root Cause:** Print spooler crash caused by a corrupted print driver thread.
**Fix:**
1. Create the watchdog script:
```powershell
$Service = Get-Service -Name "Spooler"
if ($Service.Status -eq "Stopped") {
    try {
        Start-Service -Name "Spooler" -ErrorAction Stop
        # Log the recovery action to the Application log
        Write-EventLog -LogName "Application" -Source "MsiInstaller" -EventId 9999 -EntryType Information -Message "Watchdog: Restarts Print Spooler service successfully."
    } catch {
        Write-EventLog -LogName "Application" -Source "MsiInstaller" -EventId 9999 -EntryType Error -Message "Watchdog failed: $_"
    }
}
```
2. Create a Scheduled Task triggered by a **System Event**.
   - Log: `System`. Source: `Service Control Manager`. Event ID: `7031`.
   - Action: Run `WatchSpooler.ps1`.
3. Test: Manually stop the Spooler service. Within 5 seconds, the watchdog task triggers and restarts the service automatically.
**Prevention:** Audit print drivers and remove legacy v3 drivers in favor of modern v4 drivers.
**Ticket Close Note:** "Deployed monitoring watchdog task triggered by Event 7031 to automatically restart the Print Spooler. Verified auto-recovery. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never run a scheduled task under a domain administrator's personal user account credentials.
> - When the administrator changes their password (typically every 30-90 days), **the scheduled task will fail to authenticate and stop running**, causing silent outages. Always use **NT AUTHORITY\SYSTEM** for local tasks, or dedicated service accounts with non-expiring passwords.

> [!warning] Common Trap
> - Forgetting to set `-ErrorAction Stop` when using `Try-Catch` blocks on standard cmdlets.
> - Most cmdlets throw *non-terminating* errors. If you do not specify `-ErrorAction Stop`, the `Catch` block is ignored, and the script will continue running past the failure blindly.

> [!tip] Senior Engineer Tip
> - When testing automated scripts via Task Scheduler, configure the action argument to log output to a log file:
>   `-Argument "-ExecutionPolicy Bypass -File C:\Script.ps1 > C:\Temp\runlog.txt 2>&1"`
>   This captures both standard output (stdout) and error codes (stderr) in a text file, allowing you to troubleshoot why a script fails when run headlessly as SYSTEM.

> [!success] Verification Steps
> - Run `Get-ScheduledTask -TaskName "Task-Name"` to confirm the task is registered and its state is `Ready`.
> - Check that script logs contain expected run timestamps and zero error alerts.

> [!question] Interview Alert
> - "How do you handle errors in a PowerShell script to prevent silent failures?"
> - Answer: "I use `Try-Catch` blocks. Because standard cmdlets generate non-terminating errors, I append the `-ErrorAction Stop` parameter to the cmdlets inside the `Try` block. This forces any failure to become terminating, redirecting the execution flow to the `Catch` block, where I log the error message using the `$_` variable."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Using personal credentials for automated tasks | Laziness during task setup | Use `SYSTEM` or dedicated service accounts with non-expiring passwords. |
| Catch blocks failing to trigger | Leaving ErrorAction at default | Always specify `-ErrorAction Stop` on cmdlets inside the Try block. |
| Printing UI output in automated backend scripts | Script hangs waiting for user input | Avoid interactive prompts (`Read-Host`, `Confirm`) in headless scripts; use parameters instead. |

---

## Lab Exercise

**Objective:** Write an advanced function with parameter validation, run a disk space query within a Try-Catch block, write results to a local log file, and register it as a Windows Scheduled Task.
**Time Required:** 30 minutes
**Environment Needed:** A Windows 11 client computer.
**Pre-requisites:** PowerShell run as Administrator.

**Steps:**
1. Open PowerShell. Create a folder for scripts:
   ```powershell
   New-Item -Path "C:\LabScripts" -ItemType Directory -ErrorAction SilentlyContinue
   ```
2. Create the script file `C:\LabScripts\DiskMonitor.ps1` containing the monitoring code:
   ```powershell
   cat << 'EOF' > C:\LabScripts\DiskMonitor.ps1
   try {
       $Disk = Get-CimInstance -ClassName Win32_LogicalDisk -Filter "DeviceID='C:'" -ErrorAction Stop
       $FreeGB = [math]::round($Disk.FreeSpace / 1GB, 2)
       $LogMsg = "$(Get-Date): C: drive has $FreeGB GB free space."
       Add-Content -Path "C:\LabScripts\run_log.txt" -Value $LogMsg
   } catch {
       $LogMsg = "$(Get-Date): ERROR - $_"
       Add-Content -Path "C:\LabScripts\run_log.txt" -Value $LogMsg
   }
   EOF
   ```
3. Test running the script:
   ```powershell
   & C:\LabScripts\DiskMonitor.ps1
   Get-Content -Path "C:\LabScripts\run_log.txt"
   ```
   - Verify that it outputs the correct free space log.
4. Register the script as a Windows Scheduled Task running daily:
   ```powershell
   $Action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-ExecutionPolicy Bypass -File C:\LabScripts\DiskMonitor.ps1"
   $Trigger = New-ScheduledTaskTrigger -Daily -At "11:00 PM"
   $Principal = New-ScheduledTaskPrincipal -UserId "NT AUTHORITY\SYSTEM" -LogonType ServiceAccount -RunLevel Highest
   Register-ScheduledTask -TaskName "LabDiskMonitor" -Action $Action -Trigger $Trigger -Principal $Principal -Force
   ```
5. Verification: Trigger the task manually and check the log:
   ```powershell
   Start-ScheduledTask -TaskName "LabDiskMonitor"
   Start-Sleep -Seconds 3
   Get-Content -Path "C:\LabScripts\run_log.txt"
   ```
   - Confirm a second log entry is written by the SYSTEM task run.

**Success Criteria:** The script is written, executes, creates log entries, registers as a Task, runs under the SYSTEM account, and updates logs successfully.
**Common Failures:** The task registration fails if user account control (UAC) blocks admin tasks (requires PowerShell run as Admin).

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a Scheduled Task in Windows and how do you run a PowerShell script using it?**
A: A Scheduled Task is a Windows feature that automatically runs a program or script on a set schedule or event trigger. To run a PowerShell script, I set the action to execute `powershell.exe` and pass the arguments `-ExecutionPolicy Bypass -File C:\Path\To\Script.ps1`.

**Q: What does the variable '$_' represent inside a PowerShell Catch block?**
A: The `$_` variable represents the current object in the pipeline. Inside a `Catch` block, it specifically holds the error message and exception details generated by the failed command in the `Try` block, allowing me to log the error.

### Intermediate (L2 Level)
**Q: Why do you need to append '-ErrorAction Stop' to commands inside a Try-Catch block?**
A: Most PowerShell cmdlets throw non-terminating errors on failure, meaning they print the error on screen but continue running. `Try-Catch` blocks only catch terminating errors. Appending `-ErrorAction Stop` forces the cmdlet to treat the failure as terminating, immediately stopping execution and redirecting the flow to the `Catch` block.

**Q: What is the benefit of running a Scheduled Task under the 'SYSTEM' account?**
A: Running a task under the `NT AUTHORITY\SYSTEM` account grants the script full administrative privileges on the local machine without requiring a user password. This prevents the task from failing when an administrator changes their domain password, which would break a task configured under personal credentials.

### Advanced (L3/Senior Level)
**Q: You are deploying an automated system patch script that reboot servers. How do you implement logging and recovery logic in case the script fails mid-way?**
A:
- **Situation**: Designing a reboot automation script with failure recovery.
- **Task**: Enforce progress tracking and clean recovery paths.
- **Action**: I implement a configuration file on disk (state file, e.g. `state.json`) that records the script's progress. Before performing any action, the script checks the state file. If a step (like patching) finishes, the script writes `Status: Patched` to the state file and triggers a reboot. Upon reboot, the script resumes, reads the state file, skips the patching step, and runs post-reboot validation checks. I wrap every step in `Try-Catch` blocks to catch errors, logging failures to both the local event log and a centralized file share.
- **Result**: The script can survive reboots and resume execution automatically, and logs errors clearly if a step fails.

### HR / Behavioral
**Q: Describe a time you automated a task that went wrong. What did you learn?**
A: I wrote a cleanup script to delete user profile folders on terminal servers inactive for 60 days. I did not test it thoroughly in a sandbox. The script failed to exclude the `Default` user profile folder and deleted it, which blocked new users from logging in. I immediately restored the default profile folder from a backup and unblocked the server. I learned that automated scripts must contain strict safeguards and exclusions, and that I should always test scripts in a sandbox environment before running them in production.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The methodology of using loops, advanced functions, and error handling to automate IT workflows.
> **Why**: Standardizes administration, prevents silent crashes, and automates daily tasks securely.
> **How**: Use `foreach` loops for lists, wrap cmdlets in `Try-Catch` with `-ErrorAction Stop`, and configure Scheduled Tasks running as SYSTEM.
> **Command**: `Register-ScheduledTask` / `$Error[0]` / `Try-Catch`
> **Interview Answer Starter**: "To design robust automation scripts, I implement Try-Catch error blocks and utilize NT AUTHORITY\SYSTEM for scheduled tasks to avoid credential dependency issues..."

**Key Numbers to Remember:**
- Default error array variable: `$Error` (latest error is `$Error[0]`)
- Task Scheduler SYSTEM User ID: `NT AUTHORITY\SYSTEM`
- Sleep command: `Start-Sleep -Seconds X`
- Cmdlet to load advanced features: `[CmdletBinding()]`

**3 Things Interviewer Wants to Hear:**
- Using `-ErrorAction Stop` is mandatory for Try-Catch blocks to trigger
- Running scheduled tasks as SYSTEM prevents password-expiration failures
- Validating parameters (`[Parameter(Mandatory=$true)]`) protects functions from bad inputs

---

## Related Notes
- [[02-Operating-Systems/03-Windows-OS/Task-Scheduler|Task Scheduler]] — Details the GUI configuration of automated tasks.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Fundamentals|PowerShell Fundamentals]] — Covers the base cmdlet pipelines.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-for-Active-Directory|PowerShell for Active Directory]] — Focuses on directory objects modified by automation.

---

## Tags
#desktop-support #powershell #scripting #automation #L3 #interview-topic #lab-complete #daily-use
