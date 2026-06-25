---
tags: [desktop-support, windows-os, services, sc-exe, L1]
aliases: [windows-services, service-management]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# Windows Services

---

## Concept Overview
- **What it is**: Windows Services are long-running executable background applications that run in their own Windows sessions, operating without a user interface or interactive login.
- **Why it matters for a support engineer**: Crucial operating system features (like print spooling, network connections, and remote access) and third-party software (antivirus, backup agents) run as services. A support engineer must know how to start, stop, configure, and troubleshoot services to restore app functionality.
- **Where you encounter this in real job**: Restarting a hung Print Spooler service, enabling the Remote Registry service for remote diagnostics, setting service recovery behaviors, and killing services stuck in a "Stopping" loop.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Starts, stops, and restarts services via the Services GUI or command line, and changes startup types.
  - **L2**: Troubleshoots service dependencies, manages services using PowerShell and `sc.exe`, and configures recovery actions.
  - **L3**: Scripts automated service monitoring, creates custom service files, and configures security descriptors to restrict service access.

---

## Technical Deep Dive

### 1. Service Security Accounts
Services run in the background under designated security contexts (service accounts):
- **Local SYSTEM (NT AUTHORITY\SYSTEM)**: The most privileged account. Has complete access to local system resources and acts as the computer on the network.
- **Network Service (NT AUTHORITY\NETWORKSERVICE)**: Has limited local rights, but can access network resources using the computer account's credentials.
- **Local Service (NT AUTHORITY\LOCALSERVICE)**: Has limited local rights and acts as an anonymous user on the network.
- **Dedicated User Account**: A specific domain or local account configured to run a service, used to enforce least-privilege security policies (e.g., SQL service accounts).

### 2. Startup Types
- **Automatic**: The service starts during OS boot.
- **Automatic (Delayed Start)**: The service starts shortly after boot (typically 2 minutes delay) after critical services have loaded, reducing boot congestion.
- **Manual**: The service starts only when called by a user or another application.
- **Disabled**: The service cannot be started.

### 3. Critical System Services Reference
| Service Name | Display Name | Core Function | Impact if Stopped |
|---|---|---|---|
| **`Spooler`** | Print Spooler | Manages print queues and print jobs | Users cannot print or detect printers. |
| **`TermService`** | Remote Desktop Services | Provides RDP connection support | Remote administration connections fail. |
| **`WinRM`** | Windows Remote Management | Enables WS-Management protocol | Remote PowerShell administration is blocked. |
| **`wuauserv`** | Windows Update | Manages Windows update delivery | The system cannot scan for or install updates. |
| **`EventLog`** | Windows Event Log | Records system and application events | The system cannot write logs, and dependent services fail. |

---

## Commands & Syntax

### PowerShell
```powershell
# Retrieve all services that are configured to start automatically but are currently stopped
Get-Service | Where-Object {$_.StartType -eq "Automatic" -and $_.Status -eq "Stopped"} |
    Select-Object Name, DisplayName, Status

# Change a service startup type to Automatic (Delayed Start)
Set-Service -Name "RemoteRegistry" -StartupType Automatic -PassThru
```

### CMD / Run Box
```cmd
REM Query service details and configuration parameters using sc.exe
sc qc Spooler
REM Force restart a service from the command prompt
net stop Spooler && net start Spooler
```

### GUI Path
> Start -> Run -> type `services.msc` (Services Management Console)
> Or: Task Manager -> Services Tab -> Click **Open Services** at the bottom.

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\
HKLM\SYSTEM\CurrentControlSet\Services\[Service-Name]\Start
```
*Note: Start Registry Values: 2 = Automatic, 3 = Manual, 4 = Disabled.*

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 7000 | Service failed to start due to logon failure / access denied | System Log |
| 7009 | Service failed to start due to a timeout error (default 30s) | System Log |
| 7036 | Service status successfully transitioned to started/stopped | System Log |
| 7045 | A new service was installed in the system | System Log |

---

## Real-World Scenarios

### Scenario 1: Print Spooler Stuck in "Stopping" State
**User Complaint:** "I tried to print a document and it got stuck. The IT helpdesk told me to restart the Print Spooler, but in the services console, the Spooler is stuck on 'Stopping' and all options are grayed out."
**Your First 3 Checks:**
1. Check the service status in the Services console or command line.
2. Locate the Process ID (PID) running the Print Spooler.
3. Check the print queue directory for corrupt spool files.
**Diagnosis Steps:**
1. Open PowerShell as Administrator and query the spooler service:
   `Get-CimInstance -ClassName Win32_Service -Filter "Name='Spooler'"`
   - Output: `Status: Degraded, State: Stop Pending, ProcessId: 1424`
2. Since the GUI options are grayed out, force-terminate the process using its PID:
   `taskkill /F /PID 1424`
   - Output: `SUCCESS: The process with PID 1424 has been terminated.`
3. Before restarting, clean corrupt print jobs in the spool directory:
   - Navigate to: `C:\Windows\System32\spool\PRINTERS`
   - Delete all files inside (`.SPL` and `.SHD` files).
**Root Cause:** A corrupt print job file caused the Print Spooler service to hang during processing, blocking it from completing a graceful shutdown.
**Fix:** Force-kill the spooler process (PID), clear the corrupt spool files, and restart the service.
```powershell
Start-Service -Name "Spooler"
```
**Prevention:** Educate users to avoid printing corrupt PDF files that exceed format parsing limits.
**Ticket Close Note:** "Force-terminated hung spooler process (PID 1424). Cleared corrupt print jobs from spool directory. Restarted Print Spooler service. Verified queue active. Closing ticket."

### Scenario 2: Third-Party Backup Service Fails to Start (Logon Failure Event ID 7000)
**User Complaint:** "The daily backup report shows that the backup failed because the backup service agent did not launch last night."
**Your First 3 Checks:**
1. Check the current status of the backup service in the Services console.
2. Check the Event Viewer System log for Event ID 7000 or 7009.
3. Verify the credentials configured on the service's **Log On** tab.
**Diagnosis Steps:**
1. Open Event Viewer -> System Log -> Filter by Event ID 7000.
   - Output: `The Backup Agent service failed to start due to the following error: The service did not start due to a logon failure.`
2. Check the service properties -> **Log On** tab.
   - The service is configured to run under a custom domain user account: `domain\svc-backup`.
3. Check the active status of `svc-backup` in Active Directory.
   - The service account password was changed recently, or the account is locked.
**Root Cause:** The service account credentials stored in the Windows Service Database were outdated, causing authentication to fail and preventing the service from launching.
**Fix:** Update the service's Log On properties with the correct password, or unlock the service account in AD. Restart the service.
**Prevention:** Use **Group Managed Service Accounts (gMSA)** for enterprise services to automate password rotation without service restarts.
**Ticket Close Note:** "Updated the password configuration for service account `svc-backup` on the Log On tab. Restarted the service successfully. Verified backup agent launched. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Change the startup type of critical system services like `EventLog`, `RpcSs` (Remote Procedure Call), or `PlugPlay` to **Disabled**.
> - Disabling these core services will cause Windows to crash on boot, resulting in a system repair cycle.
> - Only modify services associated with optional features or verified third-party applications.

> [!warning] Common Trap
> - Attempting to restart a service that has missing or failing **Dependencies**.
> - A service will fail to start if its parent dependency service is stopped or disabled.
> - Always check the **Dependencies** tab in the service properties before attempting to troubleshoot startup failures.

> [!tip] Senior Engineer Tip
> - If a service fails to start within the default 30-second window, Windows logs Event ID 7009. You can increase the service startup timeout threshold by creating a registry entry:
>   - Path: `HKLM\SYSTEM\CurrentControlSet\Control`
>   - DWORD: `ServicesPipeTimeout` = `60000` (sets timeout to 60 seconds).

> [!success] Verification Steps
> - Run the command: `Get-Service -Name "Spooler"` to verify status.
> - Confirm the status reads **Running**.

> [!question] Interview Alert
> - "How do you restart a Windows service that is stuck in a 'Stopping' state?"
> - Answer: "I open a command prompt or PowerShell as Administrator, locate the service's Process ID (PID) using `sc queryex [service-name]` or `Get-CimInstance`, and force-kill the process using `taskkill /F /PID [PID]`. I then start the service normally."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Storing user passwords on services | Accounts get locked on password changes | Use Group Managed Service Accounts (gMSA) or local system accounts for services. |
| Disabling the Windows Update service | Attempting to block updates | Use Group Policy (GPO) to manage update delivery schedules instead of disabling the service. |
| Forgetting to check dependencies | Service fails with depend error | Inspect the Dependencies tab and start parent services first. |

---

## Lab Exercise

**Objective:** Query service parameters, modify startup types, configure service recovery actions, and force-kill a hung service process.
**Time Required:** 20 minutes
**Environment Needed:** A Windows 11 VM or physical machine with local administrator rights.
**Pre-requisites:** Administrative access.

**Steps:**
1. Open PowerShell as Administrator.
2. Query the current configuration of the Remote Registry service:
   ```cmd
   sc qc RemoteRegistry
   ```
3. Change the startup type to Automatic:
   ```powershell
   Set-Service -Name "RemoteRegistry" -StartupType Automatic
   ```
4. Configure Service Recovery: Open the Services console (`services.msc`) -> Locate **Remote Registry** -> Right-click **Properties** -> **Recovery** tab.
   - First failure: Set to **Restart the Service**.
   - Reset fail count after: `1` day. Click **OK**.
5. Simulate service hang and termination: Start the service:
   ```powershell
   Start-Service -Name "RemoteRegistry"
   ```
6. Find the PID:
   ```powershell
   (Get-Process -Name "regsvc").Id
   ```
7. Force-kill the process:
   ```cmd
   taskkill /F /IM regsvc.exe
   ```
8. Verification: Check the service status.
   ```powershell
   Get-Service -Name "RemoteRegistry"
   ```
   - Expected output: Status is `Running` (the recovery policy automatically restarted the service).

**Success Criteria:** The service is configured for auto-restart on failure, and automatically launches after the process is terminated.
**Common Failures:** Access denied errors if commands are run without administrative privileges.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the difference between net start and sc.exe commands?**
A: `net start` is a legacy command used to start active services by name. `sc.exe` (Service Control) is a modern command-line utility used to query service status, edit configurations, manage dependencies, and create or delete service registration entries.

**Q: Where in the Windows GUI can you configure a service to restart automatically if it fails?**
A: I open the Services console (`services.msc`), right-click on the specific service, select **Properties**, navigate to the **Recovery** tab, and configure the actions for first, second, and subsequent failures to "Restart the Service."

### Intermediate (L2 Level)
**Q: Explain what a service dependency is, and what happens if a dependency is disabled.**
A: A service dependency is a requirement where one service depends on the active running state of other parent services to function (e.g., the Print Spooler depends on the HTTP Service and RPC). If a required dependency is disabled, Windows will block the child service from starting, returning a dependency error code in the logs.

### Advanced (L3/Senior Level)
**Q: An administrative script fails to configure a custom service remotely, returning an access denied error. You are using domain admin credentials. Explain your diagnostic process.**
A:
- **Situation**: A service configuration script failed remotely with access denied errors despite using domain admin rights.
- **Task**: I needed to identify if the issue was UAC filtering, WinRM port blocks, or service security descriptors.
- **Action**: I verified that WinRM port 5985 was listening. I checked the service security descriptor permissions using:
   `sc sdshow [service-name]`.
   I observed that the custom service was configured with a security descriptor that explicitly denied write permissions to the Domain Admins group. I updated the security descriptor using `sc sdset` to grant full control permissions to the administrators.
- **Result**: The script successfully configured the service remotely.

### HR / Behavioral
**Q: Tell me about a time you identified a security vulnerability on a client machine and resolved it.**
A: I identified a legacy third-party service running under the local administrator account that had unquoted service path vulnerabilities. I changed the service log on properties to local service and quoted the path, securing the machine.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: Background processes running without user interaction in dedicated Windows sessions.
> **Why**: Critical for managing OS features (printing, RDP) and troubleshooting application launches.
> **How**: Use Services.msc, sc.exe, or PowerShell cmdlets to manage services and configure recovery properties.
> **Command**: `sc queryex [service-name]`
> **Interview Answer Starter**: "In my experience, service failures are resolved by verifying dependencies and checking the service account permissions in the Log On tab..."

**Key Numbers to Remember:**
- Default service start registry value: 2 (Automatic), 3 (Manual), 4 (Disabled)
- Default startup timeout: 30 seconds (Event ID 7009)
- Local System SID: S-1-5-18

**3 Things Interviewer Wants to Hear:**
1. Service isolation methods (force-killing hung processes using PID)
2. Difference between Local System, Network Service, and Local Service accounts
3. Service recovery options and dependencies validation

---

## Related Notes
- [[02-Operating-Systems/03-Windows-OS/Registry|Registry]] — Details service configuration keys under HKLM.
- [[02-Operating-Systems/03-Windows-OS/Event-Viewer|Event Viewer]] — Details Event IDs used to audit service starts.
- [[03-Identity-and-Core-Services/06-Active-Directory/Users-and-Groups|Users and Groups]] — Details service account configurations.

---

## Tags
#desktop-support #windows-os #services #sc-exe #L1 #interview-topic #lab-complete #daily-use

