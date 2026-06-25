---
tags: [desktop-support, powershell, scripting, automation, L1]
aliases: [powershell-basics, powershell-pipeline, execution-policy]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# PowerShell Fundamentals

---

## Concept Overview
- **What it is**: PowerShell is a cross-platform task automation and configuration management framework consisting of a command-line shell and scripting language. Unlike text-based shells (like Bash), PowerShell is **object-oriented**, passing structured .NET objects through the pipeline.
- **Why it matters for a support engineer**: Command-line automation is the bridge between L1 support and L3 systems engineering. Support engineers use PowerShell to query system states, filter logs, automate software installs, and run bulk administration tasks.
- **Where you encounter this in real job**: Fetching hardware details, exporting user lists to CSV, stopping locked services, and bypass-running scripts blocked by local execution policies.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Executes pre-written scripts, runs basic query cmdlets (`Get-Process`, `Get-Service`), and unblocks local execution policies.
  - **L2**: Writes basic custom functions, constructs advanced pipe chains (using `Where-Object` and `Select-Object`), and exports audited data.
  - **L3**: Builds enterprise-scale automation modules, writes scripts with advanced error handling (`Try-Catch`), manages remote endpoints (WinRM), and deploys Just-Enough-Administration (JEA) endpoints.

---

## Technical Deep Dive

### 1. Object-Oriented Pipeline vs. Text-Based Pipeline
In standard Linux shells (like Bash), commands output plain text. To extract a process ID, you must parse the string using utilities like `grep`, `awk`, or `cut`.
PowerShell uses the **Object Pipeline**:
- Every cmdlet outputs structured **Objects** containing **Properties** (data attributes) and **Methods** (executable actions).
- When you pipe a command (`Get-Process | Stop-Process`), PowerShell passes the actual memory object, allowing the second cmdlet to access its properties directly without text parsing.

```
[Get-Process] ===(passes Process Objects)===> [Where-Object] ===(filters Objects)===> [Stop-Process]
```

### 2. Execution Policies
PowerShell Execution Policies are safety controls that determine what scripts are allowed to run, protecting systems from untrusted executions:
- **Restricted**: Default configuration. No scripts can run; only individual commands are allowed.
- **AllSigned**: Only scripts signed by a trusted digital publisher can run.
- **RemoteSigned**: Local scripts can run without signatures. Scripts downloaded from the internet must be signed by a trusted publisher.
- **Unrestricted**: All scripts can run. Prompts warnings for internet-downloaded scripts.
- **Bypass**: Nothing is blocked, and no warnings are shown. Often used during automated software deployments.
- *Warning*: Execution policies are **not security boundaries**. Users can easily bypass them (e.g., using `powershell.exe -ExecutionPolicy Bypass`). They are speed-bumps to prevent accidental runs.

### 3. Core Help & Discovery Cmdlets
PowerShell includes three "discovery" cmdlets to navigate commands:
1. **`Get-Command`**: Searches for cmdlets, functions, or aliases (e.g., `Get-Command *service*`).
2. **`Get-Help`**: Displays usage instructions, syntax parameters, and real examples (e.g., `Get-Help Get-Service -Examples`).
3. **`Get-Member`**: Inspects an object to list its available properties and methods (e.g., `Get-Service | Get-Member`).

---

## Commands & Syntax

### PowerShell
```powershell
# Update the local help files from Microsoft servers
Update-Help -Force -ErrorAction SilentlyContinue

# Find all cmdlets related to managing network adapters
Get-Command *NetAdapter*

# Inspect a service object to view its properties and methods
Get-Service | Get-Member

# Query running processes, filter for those using over 100MB of RAM, sort descending, and select properties
Get-Process |
    Where-Object {$_.WorkingSet -gt 100MB} |
    Sort-Object WorkingSet -Descending |
    Select-Object Name, Id, WorkingSet |
    Format-Table -AutoSize

# Export a filtered list of stopped Automatic services to a CSV report
Get-Service |
    Where-Object {$_.Status -eq 'Stopped' -and $_.StartType -eq 'Automatic'} |
    Select-Object Name, DisplayName, StartType, Status |
    Export-Csv -Path "C:\Reports\StoppedServices.csv" -NoTypeInformation

# Get the active execution policy for the current user session
Get-ExecutionPolicy -List

# Set the execution policy to RemoteSigned at the local machine scope
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine -Force
```

### CMD / Run Box
```cmd
:: Launch PowerShell bypassing the execution policy for a single script run
powershell.exe -ExecutionPolicy Bypass -File C:\Temp\setup-script.ps1
```

### Important Registry Paths
- Enforced system execution policies:
  ```
  HKLM\SOFTWARE\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell
  (Value "ExecutionPolicy" defines the system baseline)
  ```

---

## Real-World Scenarios

### Scenario 1: Automated Maintenance Script Blocked by Execution Policy
**User Complaint:** A helpdesk technician reports: *"I am trying to run an onboarding script 'new-hire.ps1' on a user's laptop, but the console blocks it, throwing a red error: 'File cannot be loaded because running scripts is disabled on this system.' I need to get this laptop ready immediately."*
**Your First 3 Checks:**
1. Check the active execution policy on the computer using `Get-ExecutionPolicy -List`.
2. Check if the script was downloaded from the internet (which tags it with a hidden "Zone.Identifier" alternate data stream).
3. Determine if you have administrative permissions to modify the policy scope.
**Diagnosis Steps:**
1. Open PowerShell and run:
   `Get-ExecutionPolicy`
   - Output: `Restricted`.
2. *Why is it blocked?* The Restricted policy blocks all script executions.
3. *Alternative check*: Check if the file is flagged as internet-downloaded by looking at its file properties. If it is, and the policy is `RemoteSigned`, it will still block until unblocked.
**Root Cause:** The system's default execution policy is set to 'Restricted', blocking all local script runs.
**Fix:**
1. To run the script once without changing the global system configuration:
   Open CMD or Run dialog, and launch:
   `powershell -ExecutionPolicy Bypass -File C:\Temp\new-hire.ps1`
2. If you want to configure the machine permanently for support work:
   Open PowerShell as Administrator and run:
   `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine -Force`
3. Test running the script normally: `.\new-hire.ps1`. It executes without blocks.
**Prevention:** Configure Active Directory GPOs to enforce `RemoteSigned` as the baseline execution policy for all corporate workstations.
**Ticket Close Note:** "Bypassed execution policy for script run. Set global policy to RemoteSigned on the client PC. Verified script execution. Closed."

### Scenario 2: Service Diagnostic Audit & Report Generation
**User Complaint:** An application manager complains: *"Our app server is failing randomly. We suspect some critical Windows services set to 'Automatic' start are stopping and not restarting. We need a list of these stopped automatic services emailed to us immediately."*
**Your First 3 Checks:**
1. Verify the service query cmdlet (`Get-Service`).
2. Filter for services matching the criteria: Status is `Stopped` and StartType is `Automatic`.
3. Format and export the data to a clean CSV file.
**Diagnosis Steps:**
1. Run a basic query:
   `Get-Service`
   - *This returns all services but does not show the StartType property in the default view.*
2. Pipe the command to `Get-Member` to find the correct property name.
   - Property found: `StartType`.
3. Construct the filter chain:
   `Get-Service | Where-Object {$_.Status -eq 'Stopped' -and $_.StartType -eq 'Automatic'}`
4. Export the results to a CSV report on the desktop:
   `Get-Service | Where-Object {$_.Status -eq 'Stopped' -and $_.StartType -eq 'Automatic'} | Select-Object Name, DisplayName, StartType, Status | Export-Csv -Path "$env:USERPROFILE\Desktop\StoppedServicesReport.csv" -NoTypeInformation`
**Root Cause:** Requirement to audit stopped automatic services for diagnostic analysis.
**Fix:**
1. Run the completed pipeline script.
2. Verify the CSV file generates successfully on the desktop.
3. Send the file to the application manager for review.
**Prevention:** Set up an automated task running a script that monitors these services and sends alerts when an automatic service stops.
**Ticket Close Note:** "Generated stopped automatic services report using PowerShell pipeline. Saved to CSV and sent to application manager. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never run a PowerShell command or script copied from internet forums (like StackOverflow) without reading and understanding every line of the code.
> - Malicious scripts can contain obfuscated code block strings that download payloads, wipe directories, or modify local security registry keys. Test unknown commands in a sandbox environment first.

> [!warning] Common Trap
> - Assuming that `Get-ExecutionPolicy` returning `Unrestricted` means you can run internet-downloaded scripts without prompts.
> - Unrestricted will still display a popup warning before executing scripts tagged with the web zone identifier. To fully unblock a downloaded script without warnings, right-click the file -> Properties -> check **Unblock**, or run `Unblock-File -Path .\script.ps1`.

> [!tip] Senior Engineer Tip
> - When writing script outputs, avoid using `Write-Host` to generate text logs. `Write-Host` prints text directly to the screen console and cannot be redirected to a file or pipeline. Use `Write-Output` or `Write-Verbose` instead, which supports redirection and logging frameworks.

> [!success] Verification Steps
> - Run `Get-ExecutionPolicy -List` to confirm that the scopes are set correctly.
> - Verify that piped commands display target properties in a structured table or CSV.

> [!question] Interview Alert
> - "Why does PowerShell use objects instead of text strings in its pipeline?"
> - Answer: "PowerShell is object-oriented to eliminate the need for complex text-parsing tools (like grep or awk). By passing .NET objects, the receiving cmdlet can access data properties directly by name (e.g., `$Process.Id` or `$Service.Status`), making script logic clean, reliable, and less prone to string formatting errors."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Using `Write-Host` for log data | Lack of output redirection knowledge | Use `Write-Output` or `Write-Verbose` to allow redirection to log files. |
| Forgetting to unblock downloaded scripts | Assuming ExecutionPolicy handles it | Use `Unblock-File` in PowerShell to remove the alternate data stream web tag. |
| Running complex pipelines that consume high CPU | Inefficient filtering order | Filter data early in the pipeline (`Where-Object` first) before sorting or selecting properties. |

---

## Lab Exercise

**Objective:** Query running processes using pipelines, filter by memory usage, sort, export to CSV, write a basic loop, and verify execution policy changes.
**Time Required:** 30 minutes
**Environment Needed:** A Windows 11 client computer.
**Pre-requisites:** PowerShell access.

**Steps:**
1. Open PowerShell.
2. Query active processes using more than 150MB of RAM:
   ```powershell
   Get-Process | Where-Object {$_.WorkingSet -gt 150MB}
   ```
3. Sort the filtered processes by memory usage descending and select Name and RAM:
   ```powershell
   Get-Process | Where-Object {$_.WorkingSet -gt 150MB} | Sort-Object WorkingSet -Descending | Select-Object Name, Id, @{Name="RAM(MB)"; Expression={[math]::round($_.WorkingSet / 1MB, 2)}}
   ```
4. Export this query to a CSV file in your Temp folder:
   ```powershell
   Get-Process | Where-Object {$_.WorkingSet -gt 150MB} | Sort-Object WorkingSet -Descending | Select-Object Name, Id, @{Name="RAM(MB)"; Expression={[math]::round($_.WorkingSet / 1MB, 2)}} | Export-Csv -Path "C:\Windows\Temp\RAM_Usage.csv" -NoTypeInformation
   ```
5. Write a basic loop that reads the CSV and prints the names:
   ```powershell
   $Processes = Import-Csv -Path "C:\Windows\Temp\RAM_Usage.csv"
   foreach ($Proc in $Processes) {
       Write-Output "Heavy Process Detected: $($Proc.Name) (ID: $($Proc.Id))"
   }
   ```
6. Check your active execution policies:
   ```powershell
   Get-ExecutionPolicy -List
   ```

**Success Criteria:** The process query filters memory, exports the data to a CSV file, loops through the CSV to output names, and lists active execution policies.
**Common Failures:** The CSV export fails if the destination folder path (`C:\Windows\Temp`) has local write restrictions (use a personal directory like `$env:TEMP` instead).

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a cmdlet in PowerShell and how does it differ from a standard command?**
A: A cmdlet is a built-in, lightweight command tool native to PowerShell. Cmdlets follow a simple "Verb-Noun" naming structure (e.g., `Get-Service`, `Stop-Process`), making them easy to learn, and they return structured objects instead of raw text.

**Q: Which command do you run to find out what properties and methods are available for a cmdlet's output?**
A: I pipe the cmdlet's output to the `Get-Member` cmdlet (e.g., `Get-Process | Get-Member`). This lists all the data properties and executable actions available for that object.

### Intermediate (L2 Level)
**Q: What is a PowerShell Execution Policy, and does it act as a security firewall?**
A: An Execution Policy is a configuration setting that determines what types of scripts are allowed to execute on the system. It does not act as a security firewall; it is a safety speed-bump designed to prevent users from accidentally running untrusted scripts. It can be easily bypassed by running `powershell.exe -ExecutionPolicy Bypass`.

**Q: How do you unblock a script file that was downloaded from the internet and fails to run?**
A: I can right-click the script file in File Explorer, open Properties, check the "Unblock" box, and click Apply. In PowerShell, I can run the command:
`Unblock-File -Path C:\Path\To\Script.ps1`
This removes the alternate data stream "Zone.Identifier" tag that Windows applies to web downloads.

### Advanced (L3/Senior Level)
**Q: You are writing a script that queries 100 remote servers for active service states. How do you optimize the pipeline execution and handle potential offline servers?**
A:
- **Situation**: Querying service states across 100 remote servers.
- **Task**: Optimize execution speeds and handle connectivity drops.
- **Action**: First, instead of using a standard sequential `foreach` loop (which halts when a server is offline), I utilize `Invoke-Command` with the `-ComputerName` array parameter, which queries the servers in parallel using WinRM. I implement a `Try-Catch` block inside the script to capture connection timeouts. To optimize speed, I run the filter logic on the remote server itself rather than pulling all data back to my local machine:
  `Invoke-Command -ComputerName $Servers -ScriptBlock { Get-Service -Name "AppSvc" -ErrorAction Stop } -ErrorVariable ConnectionFailures`
- **Result**: The script executes in parallel, reducing execution time from minutes to seconds, and logs offline servers without crashing the script.

### HR / Behavioral
**Q: Tell me about a time you had to automate a repetitive task. What was the task and how much time did it save?**
A: Our helpdesk spent 15 minutes manually creating user accounts, mailbox settings, and Intune groups for every new hire, which led to typing mistakes. I wrote a PowerShell provisioning script using the Active Directory and Microsoft Graph modules. Now, the technician simply inputs the user's name and department. The script builds the account, sets the correct UPN suffix, and assigns licensing groups. This reduced provisioning time to under 1 minute per user, eliminating configuration errors and saving our team about 3 hours of manual work per week.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: An object-oriented scripting and shell automation framework.
> **Why**: Critical for querying system states, automating administration, and executing bulk changes.
> **How**: Utilize verb-noun cmdlets, leverage object pipelines, manage execution policies, and inspect objects using `Get-Member`.
> **Command**: `Get-Member` / `Set-ExecutionPolicy` / `Get-Help`
> **Interview Answer Starter**: "PowerShell's primary strength is its object-oriented pipeline, which passes .NET objects directly to cmdlets without text parsing, allowing me to build robust scripts..."

**Key Numbers to Remember:**
- Default execution policy: Restricted
- Number of core discovery cmdlets: `3` (Get-Command, Get-Help, Get-Member)
- RAM limit conversion unit: `1MB` = `1024KB` (used in arithmetic filters)
- Port required for WinRM (HTTP): TCP 5985 / (HTTPS): TCP 5986

**3 Things Interviewer Wants to Hear:**
- Objects, not text, are passed through the pipeline, eliminating string parsing (awk/grep)
- Execution policies are safety features, not absolute security barriers
- Filtering early in the pipeline (`Where-Object` first) optimizes CPU usage

---

## Related Notes
- [[02-Operating-Systems/03-Windows-OS/Windows-Services|Windows Services]] — Focuses on background services managed via cmdlets.
- [[02-Operating-Systems/03-Windows-OS/Registry|Registry]] — Outlines registry key edits automated by PowerShell scripts.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Automation-and-Scripting|PowerShell Automation and Scripting]] — Advanced automation loops and error handling.

---

## Tags
#desktop-support #powershell #scripting #automation #L1 #interview-topic #lab-complete #daily-use
