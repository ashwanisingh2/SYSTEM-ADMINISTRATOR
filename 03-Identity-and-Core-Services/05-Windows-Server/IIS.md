---
tags: [desktop-support, windows-server, iis, web-server, L2]
aliases: [internet-information-services, windows-web-hosting]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# IIS

---

## Concept Overview
- **What it is**: Internet Information Services (IIS) is Microsoft's proprietary, extensible web server software role designed for Windows Server to host websites, web applications, and APIs using standard protocols (HTTP, HTTPS, FTP, SMTP).
- **Why it matters for a support engineer**: Support engineers must manage internal company intranets, publish web interfaces for diagnostic tools, configure secure SSL/TLS certificates, and troubleshoot application pool crashes.
- **Where you encounter this in real job**: Managing local helpdesk ticketing web portals, installing SSL certificates on web nodes, troubleshooting "503 Service Unavailable" errors, and setting up access directories.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Restarts IIS services, recycles application pools, checks basic web folder permissions, and reads IIS logs.
  - **L2**: Configures bindings (ports, host headers), imports SSL certificates, manages application pool credentials, and configures directory security.
  - **L3**: Designs load-balanced IIS web farms, configures web config files, optimizes caching, and sets up advanced URL rewriting policies.

---

## Technical Deep Dive

### 1. Application Pools & Worker Processes
IIS separates web applications using Application Pools (App Pools):
- **Application Pool**: An isolation boundary for web applications. If an app pool crashes, other apps running in different app pools on the same server continue to run.
- **Worker Process (`w3wp.exe`)**: The active Windows process that runs the web application code. Each App Pool runs its own instance of `w3wp.exe` under a designated security identity (e.g., `ApplicationPoolIdentity` or a domain service account).

### 2. Website Bindings
Bindings tell IIS which incoming web traffic to route to which website:
- **IP Address**: The network interface IP to listen on (or set to "All Unassigned").
- **Port**: The network port (default is `80` for HTTP, `443` for HTTPS).
- **Host Name (Host Header)**: The domain name (e.g., `timesheet.company.com`). Allows hosting multiple different websites on the same IP address and port.

### 3. HTTP Status Codes Reference
- **200 OK**: The request was successful, and the page loaded.
- **301 Moved Permanently / 302 Found**: Redirection codes.
- **401 Unauthorized**: Authentication is required or has failed.
- **403 Forbidden**: The server understood the request but refuses to authorize it (often due to directory browsing disabled or bad NTFS folder permissions).
- **404 Not Found**: The requested file or directory does not exist on the server.
- **500 Internal Server Error**: A generic error indicating the web application code crashed.
- **503 Service Unavailable**: The server cannot handle the request (usually because the Application Pool is stopped or crashed).

### 4. IIS Log Files
- **Default Path**: `C:\inetpub\logs\LogFiles\W3SVC[Site-ID]\`
- Logs are written in W3C Extended Log File Format, recording client IP, request time, URI target, HTTP status code, and bytes sent.

---

## Commands & Syntax

### PowerShell
```powershell
# Recycle an active IIS Application Pool to clear memory
Restart-WebAppPool -Name "DefaultAppPool"

# Create a new IIS website with specific binding settings
New-IISSite -Name "HelpdeskPortal" -PhysicalPath "C:\inetpub\helpdesk" -BindingInformation "*:8080:"
```

### CMD / Run Box
```cmd
REM Launch the IIS Manager GUI console directly
inetmgr
REM Restart all local IIS web services from the command prompt
iisreset
```

### GUI Path
> Server Manager -> Tools -> **Internet Information Services (IIS) Manager**

### Important Registry Paths
```
HKLM\System\CurrentControlSet\Services\W3SVC\Parameters\
HKLM\SOFTWARE\Microsoft\InetStruc\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 5074 | Application Pool crash (worker process exited unexpectedly) | System Log |
| 5059 | Application Pool disabled due to rapid-fail protection | System Log |
| 7036 | World Wide Web Publishing Service started/stopped | System Log |

---

## Real-World Scenarios

### Scenario 1: Web Application Fails with "503 Service Unavailable" (App Pool Crash)
**User Complaint:** "When I try to access the corporate ticketing portal, the browser immediately returns a 'HTTP Error 503. The service is unavailable' message."
**Your First 3 Checks:**
1. Check the Application Pool status in the IIS Manager console.
2. Check the Event Viewer System log for Event ID 5074 or 5059.
3. Check the credentials configured on the App Pool identity.
**Diagnosis Steps:**
1. Open IIS Manager (`inetmgr`) -> Navigate to **Application Pools**.
2. Locate the App Pool for the site:
   - Status: `Stopped`
3. Open Event Viewer -> System Log -> Filter by Source `WAS` (Windows Process Activation Service):
   - Event ID 5059: `Application pool 'TicketingAppPool' has been disabled. Rapid-fail protection was triggered due to a series of failures.`
   - Event ID 5074: `A worker process serving application pool 'TicketingAppPool' terminated unexpectedly.`
4. Check the App Pool Identity. It is configured to run as a domain service account: `domain\svc-ticket`.
   - The password of `svc-ticket` was changed, causing the worker process (`w3wp.exe`) to fail to start and triggering rapid-fail protection.
**Root Cause:** Outdated credentials on the Application Pool Identity caused the worker process to crash repeatedly, triggering rapid-fail protection and disabling the App Pool.
**Fix:** Update the Application Pool Identity settings with the correct password, right-click the App Pool, and click **Start**.
**Prevention:** Use Group Managed Service Accounts (gMSA) to manage App Pool credentials automatically.
**Ticket Close Note:** "Updated App Pool Identity credentials. Cleared rapid-fail lockout and started the stopped App Pool. Web portal returned online. Closing ticket."

### Scenario 2: Web Server Returns a "403 - Forbidden" Error
**User Complaint:** "I migrated our documentation folder to the IIS web folder, but when I try to open the page, I get a '403 - Forbidden: Access is denied' error."
**Your First 3 Checks:**
1. Check if the default document (e.g., `index.html` or `default.aspx`) exists in the directory.
2. Verify if directory browsing is enabled in the IIS console.
3. Check the NTFS folder permissions on the physical directory.
**Diagnosis Steps:**
1. Open the physical directory `C:\inetpub\wwwroot\docs`.
   - The folder contains files, but no default document name is present.
2. Check the IIS console for the site -> Double-click **Directory Browsing**.
   - Status: `Disabled` (default for security). Since there is no index file and directory browsing is blocked, IIS returns 403.
3. Check NTFS permissions:
   - The local group `IIS_IUSRS` (IIS Worker Processes) does not have read access to the physical folder.
**Root Cause:** Missing default document, directory browsing disabled, and incorrect NTFS folder permissions for the IIS worker process.
**Fix:** Add `IIS_IUSRS` with Read permissions to the folder, and create an `index.html` file or enable directory browsing in the IIS console.
**Prevention:** Always verify that the local `IIS_IUSRS` group is granted read access to new web application physical folders.
**Ticket Close Note:** "Granted Read permissions to IIS_IUSRS on the physical folder. Created an index.html file. The 403 error is resolved. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Grant the `Everyone` group Full Control NTFS permissions on physical web directories to bypass "Access Denied" errors.
> - This represents a security vulnerability; anyone on the network can write or execute malicious files in the web folder, compromising the server.
> - Only grant Read/Execute permissions to the local `IIS_IUSRS` group or the specific App Pool identity.

> [!warning] Common Trap
> - Leaving the default web page active on public-facing IIS installations.
> - The default page displays the IIS version and operating system details, giving attackers information about the server.
> - Remove or replace the default page immediately after installation.

> [!tip] Senior Engineer Tip
> - If an Application Pool crashes randomly and leaves no logs, configure **Orphan Action** in the App Pool Advanced Settings. This allows you to automatically run a script to capture a memory dump of `w3wp.exe` before the process terminates.

> [!success] Verification Steps
> - Run: `Test-NetConnection -ComputerName "localhost" -Port 80` to verify port access.
> - Confirm the connection check returns `True`.

> [!question] Interview Alert
> - "What does a 503 Service Unavailable error mean in IIS, and how do you resolve it?"
> - Answer: "A 503 error indicates that the Application Pool responsible for the website has stopped or crashed. To resolve it, I open IIS Manager, check the status of the App Pools, restart the stopped App Pool, and inspect the System event logs for Event IDs from the Windows Process Activation Service to identify the root cause."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| App Pool crashing on credentials | Updating AD passwords without updating IIS | Use gMSAs or local system accounts for App Pool identities to avoid credential conflicts. |
| Forgetting to bind host headers | Port conflicts on sharing IP addresses | Always configure host names (host headers) when running multiple sites on port 80/443. |
| Neglecting to configure log rotations | IIS logs fill the system drive | Set up scheduled tasks or scripts to compress or delete IIS log files older than 30 days. |

---

## Lab Exercise

**Objective:** Install the Web Server (IIS) role, configure a custom application pool, bind a site to port 8080, and verify access.
**Time Required:** 20 minutes
**Environment Needed:** A Windows Server 2022 VM with local administrator rights.
**Pre-requisites:** Administrative access.

**Steps:**
1. Open PowerShell as Administrator.
2. Install the Web Server role:
   ```powershell
   Install-WindowsFeature -Name "Web-Server" -IncludeManagementTools
   ```
3. Create a custom Application Pool:
   ```powershell
   New-WebAppPool -Name "LabAppPool"
   ```
4. Create the physical directory and a test page:
   ```powershell
   New-Item -ItemType Directory -Path "C:\inetpub\labsite" -Force
   Set-Content -Path "C:\inetpub\labsite\index.html" -Value "<h1>Lab Web Server Active</h1>"
   ```
5. Create the website and bind it to the new App Pool and port 8080:
   ```powershell
   New-Website -Name "LabSite" -PhysicalPath "C:\inetpub\labsite" -Port 8080 -ApplicationPool "LabAppPool" -Force
   ```
6. Grant permissions to the physical folder:
   ```powershell
   icacls "C:\inetpub\labsite" /grant "IIS_IUSRS:(OI)(CI)(RX)"
   ```
7. Verification: Test connection to the site using a browser or run:
   ```powershell
   Test-NetConnection -ComputerName "localhost" -Port 8080
   ```
   - Expected output: `TcpTestSucceeded : True`

**Success Criteria:** The custom site is created, binds to port 8080, and the test page is accessible.
**Common Failures:** Connection failure if the local Windows Defender Firewall blocks port 8080. Create a firewall exception rule.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is an Application Pool in IIS?**
A: An Application Pool is an isolation boundary used to separate web applications running on the same server. If one application pool crashes due to a code error, other applications running in different app pools continue to function.

**Q: Where are the default IIS log files stored?**
A: By default, IIS log files are stored in the `%SystemDrive%\inetpub\logs\LogFiles\` directory, sorted into subfolders named after the website's unique site ID (e.g., `W3SVC1`).

### Intermediate (L2 Level)
**Q: What is a Host Header in IIS website bindings, and why is it used?**
A: A Host Header is a binding setting that maps a domain name (e.g., `app.company.com`) to a website. It is used to host multiple different websites on a single server sharing the same IP address and port, as IIS reads the host header in the HTTP request to route traffic to the correct site.

### Advanced (L3/Senior Level)
**Q: A web application pool crashes consistently under high user load. Event logs show Event ID 5059. Explain your diagnostic and resolution strategy.**
A:
- **Situation**: An application pool crashed consistently under load, triggering rapid-fail protection.
- **Task**: I needed to identify the root cause of the worker process crashes.
- **Action**: I opened Event Viewer and checked the Application log. I observed several **Event 1000 Application Error** logs pointing to `clr.dll`, indicating a memory leak or crash in the .NET runtime code. I used **DebugDiag** to capture a memory dump of the `w3wp.exe` process during a crash. I analyzed the dump and identified a database connection timeout loop.
- **Result**: I worked with the developers to optimize the database query connection pools, resolved the crash, and restarted the App Pool.

### HR / Behavioral
**Q: Tell me about a time you resolved a major web server outage. How did you identify the issue?**
A: Our company intranet went offline with a 503 error. I logged in, checked the IIS console, found the App Pool had stopped due to an expired service account password, updated the credentials, restarted the pool, and brought the portal online.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The Windows Server web server role used to host websites, APIs, and web applications.
> **Why**: Critical for hosting corporate portals, web services, and configuring secure TLS/SSL endpoints.
> **How**: Manage Application Pools, configure bindings, set NTFS permissions, and analyze logs.
> **Command**: `iisreset`
> **Interview Answer Starter**: "In my experience, IIS administration requires separating web applications into distinct Application Pools to prevent single-application failures from causing server outages..."

**Key Numbers to Remember:**
- Default HTTP Port: 80 / HTTPS Port: 443
- Default IIS log path: `C:\inetpub\logs\LogFiles\`
- App Pool crash threshold for rapid-fail: 5 failures within 5 minutes (default)

**3 Things Interviewer Wants to Hear:**
1. Separating applications into distinct App Pools to isolate failures
2. Restricting folder permissions to the local `IIS_IUSRS` group (least-privilege)
3. Troubleshooting 503 errors (checking stopped App Pools and service credentials)

---


---
## Real-World Scenario: File Server Crash (500+ Users Affected)
Agar company ka main file server (500+ users connected) crash ho jaye, toh as a Senior Sysadmin aapka immediate action plan yeh hoga:
1. **Identify & Isolate**: Check if the server is pingable. Verify hardware status via Dell iDRAC / HPE iLO or check virtual machine status in vCenter/Hyper-V.
2. **Access Check & Lock**: If the storage volume is corrupt or unmapped, stop DFS Namespace referrals to the crashed target to redirect users to secondary replica servers if active.
3. **Trigger Disaster Recovery (DR)**: Start restoring the crashed system drive or volumes from the latest VSS snapshot or Azure Backup RSV restore point.
4. **Communication**: Broadcast status updates to the Incident Commander and helpdesk teams to manage incoming support volume.

### PowerShell Automation Snippet: Verify File Shares and Access Permissions
```powershell
# Get all active file shares on the server
Get-SmbShare | Format-Table -AutoSize

# Test local DFS Namespace server connection health
Test-DFSNamespaceTarget -Path "\\corp.local\DFSRoot\Public"
```

---
## Related Notes
- [[02-Operating-Systems/03-Windows-OS/Windows-Services|Windows Services]] — Details the World Wide Web Publishing Service (`W3SVC`).
- [[03-Identity-and-Core-Services/05-Windows-Server/Roles|Roles]] — Details server role deployment settings.
- [[03-Identity-and-Core-Services/05-Windows-Server/DNS-Server|DNS Server]] — Details host header resolution setups.

---

## Tags
#desktop-support #windows-server #iis #web-server #L2 #interview-topic #lab-complete #daily-use

