---
tags: [desktop-support, windows-server, server-roles, L1]
aliases: [server-roles, roles-vs-features]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #az-104
---

# Roles

---

## Concept Overview
- **What it is**: Windows Server Roles and Features are software components that define the capabilities of a server, allowing it to function as a Domain Controller, DNS Server, DHCP Server, Web Server (IIS), or File Server.
- **Why it matters for a support engineer**: A support engineer must know how to install, manage, and audit server roles to maintain infrastructure services.
- **Where you encounter this in real job**: Deploying new domain controllers, setting up file storage nodes, configuring IIS web hosting directories, and auditing active roles on local networks.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Installs basic roles and features via Server Manager, checks service status, and verifies role installations.
  - **L2**: Manages role migrations, configures advanced role options (like IIS directory security), and deploys features using PowerShell.
  - **L3**: Designs server infrastructure topologies, manages containerized deployments, and defines server capacity and provisioning standards.

---

## Technical Deep Dive

### 1. Server Roles vs. Server Features
Windows Server divides software components into two categories:
- **Server Roles**: Primary functions that define the server's purpose in the network (e.g., Active Directory Domain Services, DHCP Server, DNS Server, IIS Web Server). Roles require configuration settings to define their scope.
- **Server Features**: Auxiliary software components that support or enhance roles (e.g., `.NET Framework`, BitLocker Drive Encryption, Failover Clustering, Windows Server Backup). Features run silently in the background.

### 2. Desktop Experience vs. Server Core
Windows Server can be installed in two modes:
- **Server Core (Recommended)**: Minimal installation option. Lacks a graphical user interface (GUI), runs on a low resource footprint, has a smaller update size, and features a smaller attack surface. Managed exclusively via command line, PowerShell, or remote tools (Windows Admin Center).
- **Desktop Experience**: Full installation option. Includes a standard GUI, consuming higher system resources and requiring more updates. Used when local console administration is required.

### 3. Critical Server Roles Reference Table
Key roles used in enterprise networks:

| Role Name | PowerShell Identifier | Core Function | Example Use Case |
|---|---|---|---|
| **AD DS** | `AD-Domain-Services` | Centralized identity directory database | Domain logins, authentication |
| **DNS Server** | `DNS` | Hostname-to-IP name resolution | Active Directory queries |
| **DHCP Server** | `DHCP` | Dynamic IP address allocation | Client network onboarding |
| **Web Server (IIS)**| `Web-Server` | Hosts website pages and APIs | Intranet portal hosting |
| **File Services** | `FS-FileServer` | Manages shared folders and access controls | Department storage folders |

---

## Commands & Syntax

### PowerShell
```powershell
# List all installed roles and features on the local server
Get-WindowsFeature | Where-Object {$_.Installed -eq $true} |
    Select-Object Name, DisplayName, FeatureType

# Install the IIS Web Server role along with management tools
Install-WindowsFeature -Name "Web-Server" -IncludeManagementTools -Confirm:$false
```

### CMD / Run Box
```cmd
REM Launch the Server Manager GUI console directly
servermanager.exe
REM Query the status of the DNS Server service from cmd
sc query DNS
```

### GUI Path
> Server Manager -> Dashboard -> Click **Add roles and features** -> Follow wizard steps.

### Important Registry Paths
```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\Packages\
HKLM\SYSTEM\CurrentControlSet\Services\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 7036 | Service status successfully transitioned to started/stopped | System Log |
| 7000 | Server role service failed to start due to logon failure | System Log |
| 20001 | Server role package installation started | System Log |

---

## Real-World Scenarios

### Scenario 1: IIS Web Server Fails to Start (Logon Failure Event ID 7000)
**User Complaint:** "The internal timesheet portal is down. I get a page cannot be displayed error in my browser. The server shows the IIS service is stopped."
**Your First 3 Checks:**
1. Check the IIS service status in the Services console.
2. Check the Event Viewer System log for Event ID 7000 or 7009.
3. Verify the credentials configured on the World Wide Web Publishing Service's Log On tab.
**Diagnosis Steps:**
1. Open Event Viewer -> System Log -> Filter by Event ID `7000`.
   - Output: `The World Wide Web Publishing Service failed to start due to the following error: The service did not start due to a logon failure.`
2. Check the service properties -> **Log On** tab.
   - The service is configured to run under a custom domain user account: `domain\svc-web`.
3. Check the active status of `svc-web` in Active Directory.
   - The service account password was changed recently, or the account is locked.
**Root Cause:** The service account credentials stored in the Windows Service Database were outdated, causing authentication to fail and preventing the service from launching.
**Fix:** Update the service's Log On properties with the correct password, or unlock the service account in AD. Restart the service.
**Prevention:** Use **Group Managed Service Accounts (gMSA)** for enterprise services to automate password rotation without service restarts.
**Ticket Close Note:** "Updated the password configuration for service account `svc-web` on the Log On tab. Restarted the service successfully. Verified backup agent launched. Closing ticket."

### Scenario 2: Administrator Cannot Install AD DS Role (Missing Dependencies)
**User Complaint:** "I am trying to install the Active Directory Domain Services role on a new server, but the installation fails during the pre-requisite check."
**Your First 3 Checks:**
1. Check the available free disk space on the system drive.
2. Check if the system has a statically configured IP address.
3. Check the installation logs for missing dependencies.
**Diagnosis Steps:**
1. Run `ipconfig /all` to check the IP configuration:
   - The IP address is leased dynamically from DHCP, which is a blocker for AD DS.
2. Review the pre-requisite check log:
   - Error: `Active Directory Domain Services requires a static IP address.`
3. Assign a static IP address to the server interface.
**Root Cause:** The server was configured with a dynamic IP address, which violated the pre-requisite rules for domain controllers.
**Fix:** Configure a static IP address on the server adapter, and rerun the AD DS promotion wizard.
**Prevention:** Always configure static IP addresses on servers before deploying infrastructure roles.
**Ticket Close Note:** "Configured static IP on the server adapter. Completed AD DS promotion successfully. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Deploy multiple critical infrastructure roles (e.g., AD DS, DHCP, and IIS) on a single physical server chassis.
> - If the physical server fails, all services go down simultaneously, creating a single-point-of-failure outage.
> - Separate roles using virtual machines (Hyper-V/VMware) to isolate services.

> [!warning] Common Trap
> - Installing server roles without installing their corresponding management tools.
> - If you uncheck "Include management tools," the role installs, but you will lack the MMC consoles (like `dnsmgmt.msc`) to manage it locally.
> - Always include management tools during installation.

> [!tip] Senior Engineer Tip
> - When deploying roles on Server Core, use **Windows Admin Center** on a management workstation. This provides a web-based GUI to manage Server Core roles remotely without consuming local server resources.

> [!success] Verification Steps
> - Run the command: `Get-WindowsFeature -Name "Web-Server"` to verify status.
> - Confirm the status reads **Installed**.

> [!question] Interview Alert
> - "What is the difference between a server role and a server feature?"
> - Answer: "A server role defines the primary function of a server, such as Active Directory or DNS, which requires configuration to define its scope. A server feature is an auxiliary component that supports or enhances roles, running silently in the background, such as BitLocker or Failover Clustering."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Installing roles on active sessions | Files locked by processes | Ensure the target user is completely logged off before running installation tools. |
| Forgetting to include management tools | Unchecking the default option | Check the **Include management tools** box during role installation. |
| Leaving unauthorized DCs unauthorized | Installing DHCP on DC without AD authorization | Authorize the DHCP server in Active Directory (Event ID 1044). |

---

## Lab Exercise

**Objective:** Install the IIS Web Server role using PowerShell, configure a basic HTML page, and verify web access.
**Time Required:** 20 minutes
**Environment Needed:** A Windows Server 2022 VM with local administrator rights.
**Pre-requisites:** Administrative access.

**Steps:**
1. Open PowerShell as Administrator.
2. Install the Web Server (IIS) role:
   ```powershell
   Install-WindowsFeature -Name "Web-Server" -IncludeManagementTools
   ```
   - Expected output: Progress bar completes, and status displays `Success : True`.
3. Create a test HTML file:
   ```powershell
   Set-Content -Path "C:\inetpub\wwwroot\index.html" -Value "<h1>IIS Web Server Active</h1>"
   ```
4. Verify the World Wide Web Publishing Service is running:
   ```powershell
   Get-Service -Name "W3SVC"
   ```
   - Expected output: Status is `Running`.
5. Open a browser on the server and navigate to `http://localhost`.
   - Expected output: The page loads and displays "IIS Web Server Active".
6. Verification: Run the verification command.
   ```powershell
   Test-NetConnection -ComputerName "localhost" -Port 80
   ```
   - Expected output: `TcpTestSucceeded : True`

**Success Criteria:** The IIS role is successfully installed, the service is running, and the web page is accessible via port 80.
**Common Failures:** Access denied errors if commands are run in a non-elevated command prompt.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is Server Manager?**
A: Server Manager is a built-in management console in Windows Server that allows administrators to discover, manage, and configure local and remote servers, as well as add or remove roles and features.

**Q: What is the benefit of Server Core over Desktop Experience?**
A: Server Core reduces system resource consumption (RAM and CPU), minimizes disk space usage, has fewer updates, and presents a smaller attack surface by removing the graphical user interface (GUI).

### Intermediate (L2 Level)
**Q: How do you install a server feature using PowerShell?**
A: I use the `Install-WindowsFeature` cmdlet followed by the feature's name identifier (e.g., `Install-WindowsFeature -Name "RSAT-AD-PowerShell"` to install the Active Directory PowerShell module).

### Advanced (L3/Senior Level)
**Q: A web application hosting critical APIs requires access to active directories. You need to configure the service account. Explain your strategy.**
A:
- **Situation**: A web application required secure access to Active Directory to authenticate users.
- **Task**: I needed to configure a secure service account context.
- **Action**: I configured a **Group Managed Service Account (gMSA)** for the application pool identity in IIS. I registered the gMSA, granted the server authorization to retrieve the password, and configured the IIS AppPool to run under the gMSA credentials, which automates password management.
- **Result**: The web application successfully integrated with Active Directory without storing static passwords locally.

### HR / Behavioral
**Q: Tell me about a time you automated a role deployment across multiple servers. What was the impact?**
A: We had to deploy the IIS role on 10 backup servers. I wrote a PowerShell script utilizing `Install-WindowsFeature` and deployed it remotely using PowerShell Remoting, completing the configuration on all servers in under 5 minutes.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: Software components that define the capabilities of a server (roles) or support them (features).
> **Why**: Critical for defining server functions (DNS, AD) and managing services.
> **How**: Use Server Manager or PowerShell `Install-WindowsFeature` to deploy roles.
> **Command**: `Install-WindowsFeature -Name "Web-Server"`
> **Interview Answer Starter**: "In my experience, server role administration starts by identifying the server requirements and matching them to Server Core or Desktop Experience..."

**Key Numbers to Remember:**
- Default IIS Port: 80 (HTTP) / 443 (HTTPS)
- Active Directory default replication interval: 15 minutes
- Max role installations per command: Unlimited (via array)

**3 Things Interviewer Wants to Hear:**
1. Difference between roles (core function) and features (supporting tool)
2. Deploying roles on Server Core using Remote Management (WinRM/WAC)
3. Enforcing static IP configurations before role deployments

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
- [[03-Identity-and-Core-Services/05-Windows-Server/DNS-Server|DNS Server]] — Details the DNS role configuration.
- [[03-Identity-and-Core-Services/05-Windows-Server/DHCP-Server|DHCP Server]] — Details the DHCP role configuration.
- [[03-Identity-and-Core-Services/05-Windows-Server/IIS|IIS]] — Details the Web Server role configuration.

---

## Tags
#desktop-support #windows-server #server-roles #L1 #interview-topic #lab-complete #daily-use

