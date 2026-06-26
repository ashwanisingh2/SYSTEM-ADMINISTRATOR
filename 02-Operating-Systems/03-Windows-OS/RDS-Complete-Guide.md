---
tags: [desktop-support, windows-os, rds, virtualization, L2]
aliases: [rds-complete-guide, remote-desktop-services]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# Remote Desktop Services (RDS) Complete Guide

> [!note] Overview
> This note covers Microsoft Remote Desktop Services (RDS) architecture, roles configuration, session host pools, RemoteApp publishing, and licensing administration. It details secure deployment configurations and session troubleshooting workflows.

---
## Concept Overview
- **What it is** — Remote Desktop Services (RDS) is a server-based virtualization platform in Windows Server. It allows users to access remote desktop environments, session-based multi-user environments, and individual published applications (RemoteApps) hosted on central servers.
- **Why it matters** — Shipping physical corporate laptops to remote contractors or temporary workers is expensive, slow, and raises data security issues. RDS solves this by keeping all data inside the secure corporate datacenter, allowing remote users to stream their workspaces over encrypted connections.
- **Real job encounter** — Supporting remote employees connecting to corporate environments, publishing line-of-business applications via web browsers, monitoring system performance on host servers, and resolving printer redirection issues.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Terminate hung user processes, disconnect frozen user sessions, assist users in mounting redirected local drives, and check host ping connectivity.
  - **Escalation Trigger**: Escalate if users receive RDS licensing errors ("No CALs available"), if the web portal displays SSL certificate warnings, or if connection broker load balancing fails.
  - **L2 Resolution**: Build and deploy RDS Session Collections, publish and restrict RemoteApps to security groups, install SSL certificates on Gateway servers, and configure session timeout limits.
  - **L3 Resolution**: Architect highly available RDS Connection Broker clusters using SQL databases, implement Secure Remote Desktop Gateways with MFA/RADIUS, and manage user profile disks (UPD) or FSLogix storage environments.

---
## Technical Deep Dive

### 1. RDS Core Architecture Roles
An enterprise RDS environment is built from five distinct roles working in tandem:
- **Remote Desktop Session Host (RDSH)**: The workhorse. This server hosts the actual desktop interfaces and applications that users run inside their sessions.
- **Remote Desktop Connection Broker (RDCB)**: The traffic cop. It handles incoming connection requests, load-balances users across available RDSH servers, and reconnects disconnected users to their existing sessions.
- **Remote Desktop Web Access (RDWA)**: The storefront. Provides users with a web portal (URL) to view and launch their allowed RemoteApps and virtual desktops.
- **Remote Desktop Gateway (RDGW)**: The security guard. Encapsulates standard RDP traffic inside an HTTPS (SSL/TLS port 443) wrapper. This allows remote workers to connect securely from public networks without using a standard VPN.
- **RD Licensing Server**: Manages Client Access Licenses (CALs) required to connect to the environment.

```
                  +--------------------------------+
                  |  External Client (Internet)    |
                  +--------------------------------+
                                  |
                           RDP over HTTPS (443)
                                  |
                                  v
                  +--------------------------------+
                  |    RD Gateway (RDGW)           |
                  +--------------------------------+
                                  |
                           Internal RDP (3389)
                                  |
                                  v
                  +--------------------------------+
                  |  RD Connection Broker (RDCB)   |<---> [ RD Licensing ]
                  +--------------------------------+
                                  |
                   Load Balances User Connection
                                  |
                                  v
         +------------------------+------------------------+
         |                                                 |
         v                                                 v
+-------------------------------+                 +-------------------------------+
|  RD Session Host 1 (RDSH-01)  |                 |  RD Session Host 2 (RDSH-02)  |
+-------------------------------+                 +-------------------------------+
```

### 2. User Connection Lifecycle
When a user launches an app via the RD Web Access portal:
1. The client browser downloads an `.rdp` file and launches the remote desktop client.
2. Connection request is sent to the **RD Gateway** (port 443).
3. The Gateway validates connection authorization policies (CAP) and resource authorization policies (RAP), decrypts HTTPS, and forwards RDP (port 3389) to the **Connection Broker**.
4. The Connection Broker checks the session database. If the user has a disconnected session, they are redirected to that host. Otherwise, it targets the host with the lowest active session load.
5. The **RDSH** requests a CAL license from the **Licensing Server**.
6. The session is established, loading the user profile via FSLogix or UPD.

### 3. Licensing Models: Device vs. User CALs
To connect to an RDS environment, you must install CAL licenses:
- **Per-User CAL**: Assigned to a specific user account. A user can connect from an unlimited number of devices. Best for environments where users have multiple machines.
- **Per-Device CAL**: Assigned to a physical device. Unlimited users can log in from that device. Best for shift-work offices or call centers sharing machines.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server 2022 domain controller (`DC-01.corp.local`).
> - A member server (`SVR-RDS01.corp.local`) assigned a static IP (`192.168.10.20`).
> - Local administrator privileges.

### Step 1: Install RDS Roles
For simple deployments, we will install Connection Broker, Web Access, and Session Host on a single server.

1. Log in to `SVR-RDS01`. Open PowerShell as Administrator and run:
```powershell
# Install core RDS host and broker roles
Install-WindowsFeature -Name Remote-Desktop-Services, RDS-RD-Server, RDS-Connection-Broker, RDS-Web-Access -IncludeManagementTools
```
**Expected Output:**
```text
Success Restart Needed Exit Code Feature Result
------- -------------- --------- --------------
True    Yes            Success   {Remote Desktop Services, Remote ...}
```
2. Restart the server: `Restart-Computer -Force`.

### Step 2: Create a Session Collection
A collection groups session host servers and defines which resources are presented to users.

1. Open PowerShell and define a new collection:
```powershell
New-RDSessionCollection -CollectionName "Finance-Apps" -SessionHost "SVR-RDS01.corp.local" -ConnectionBroker "SVR-RDS01.corp.local" -CollectionDescription "Hosted apps for Finance team"
```
**Expected Output:** Collection created successfully.

### Step 3: Publish RemoteApp Programs
We will publish standard Windows Calculator and Notepad as accessible RemoteApps.

1. Publish Calculator:
```powershell
New-RDRemoteApp -CollectionName "Finance-Apps" -Alias "Calculator" -DisplayName "Corp Calc" -FilePath "C:\Windows\System32\calc.exe" -ConnectionBroker "SVR-RDS01.corp.local"
```
2. Publish Notepad:
```powershell
New-RDRemoteApp -CollectionName "Finance-Apps" -Alias "Notepad" -DisplayName "Corp Notepad" -FilePath "C:\Windows\System32\notepad.exe" -ConnectionBroker "SVR-RDS01.corp.local"
```
**Expected Output:** Both applications appear listed under the Published RemoteApps list.

### Step 4: Configure Session Timeout Policies
Configure automated cleanup rules to prevent idle users from exhausting CPU and memory resources.

1. Set Session Limits using PowerShell:
```powershell
Set-RDSessionCollectionConfiguration -CollectionName "Finance-Apps" -ConnectionBroker "SVR-RDS01.corp.local" -ConnectionActiveSessionLimitMin 480 -ConnectionIdleSessionLimitMin 60 -ConnectionDisconnectedSessionLimitMin 15 -AutomaticReconnectionEnabled $true
```
*Note: This logs off disconnected sessions after 15 minutes and idle sessions after 60 minutes.*

### Step 5: Verify Client Connection
1. From a client workstation, open a web browser and navigate to: `https://SVR-RDS01.corp.local/RDWeb`.
2. Authenticate using domain credentials.
3. Verify that the "Corp Calc" and "Corp Notepad" icons are displayed.
4. Click an icon to run the RemoteApp. Ensure it loads seamlessly in its own window.

---
## Cheat Sheet / Quick Reference

| Command / Setting | Purpose | Example |
|---|---|---|
| `Get-RDUserSession` | Lists active and disconnected user sessions on host | `Get-RDUserSession -ConnectionBroker "SVR-RDS01.corp.local"` |
| `Invoke-RDUserSignOut` | Forces logoff of a target user session | `Invoke-RDUserSignOut -HostServer "RDSH-01" -UnifiedSessionId 3` |
| `Get-RDSessionHost` | Queries status of hosts in a collection | `Get-RDSessionHost -CollectionName "Finance-Apps"` |
| `Set-RDSessionHost` | Changes host state (e.g. enabling drain mode) | `Set-RDSessionHost -NewConnectionAllowed No` |
| `mstsc /v:<IP>` | Launches RDP client connection from command line | `mstsc /v:192.168.10.20` |
| `mstsc /admin` | Bypasses broker redirect to log in directly to host console | `mstsc /v:192.168.10.20 /admin` |
| **Port 3389 (TCP/UDP)** | Default port used for Remote Desktop protocol | Firewall configuration entry |
| **Port 443 (TCP)** | Port used for RD Gateway external HTTPS tunnels | Firewall configuration entry |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| User gets error: "The remote session was disconnected because there are no Remote Desktop License Servers available." | The grace period (120 days) has expired, or the licensing server is not configured in group policy. | Set the RDS licensing mode and licensing server address using Local Group Policy (`gpedit.msc` -> Administrative Templates > Windows Components > Remote Desktop Services > Remote Desktop Session Host > Licensing). |
| Users cannot connect, receiving error: "The host server is currently not accepting new connections." | The target Session Host is configured in Drain Mode (`NewConnectionAllowed` set to `No`). | Change host connection settings: `Set-RDSessionHost -NewConnectionAllowed Yes` or set drain mode to Disabled in the server collection GUI. |
| RemoteApp launch returns certificate warning: "The identity of the remote computer cannot be verified." | The RDS deployment is using default self-signed SSL certificates. | Install a publicly trusted SSL certificate (issued by GoDaddy, DigiCert, or internal AD CS CA) and bind it to Gateway, Broker, and Web Access roles. |
| User session hangs on a black screen when connecting. | Display driver mismatch or bitmap caching issues on client. | Disable "Persistent bitmap caching" in the RDP client options. On the host, update the display driver or restart the RDS Session Host agent services. |
| Printer redirection fails (user cannot print to their local printer from session). | The required print driver is not installed on the Session Host server. | Install the matching printer driver on the RDSH server, or enable the "Easy Print driver" policy in GPO to allow generic redirection. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** How do you log off a hung user session from an RDS host using the command line?
> **A:** First, I will run `query user` or `Get-RDUserSession` to locate the user's Session ID. Once identified, I will run `logoff <Session_ID>` (or `Invoke-RDUserSignOut` in PowerShell) to force the hung user session to log off and free up server resources.

> [!question] L2 Question
> **Q:** What is "Drain Mode" on an RDS Session Host, and when would you use it?
> **A:** Drain Mode prevents new user sessions from connecting to a specific RD Session Host server while allowing users who are currently logged in to continue their work. This is used during maintenance windows, allowing administrators to patch or reboot hosts without abruptly terminating active user sessions. It is enabled using `Set-RDSessionHost -NewConnectionAllowed No`.

> [!question] L3/Scenario Question
> **Q:** Explain the role of the RD Gateway in an enterprise network. How does it protect the network compared to direct port forwarding?
> **A:** 
> - **Situation:** Securing remote desktop access for external users.
> - **Task:** Compare RD Gateway architecture against port forwarding.
> - **Action:** Direct port forwarding exposes UDP/TCP port 3389 directly to the public internet, exposing hosts to brute force attacks and exploitation. The **RD Gateway** secures this by acting as a reverse proxy at the network boundary:
>   1. It encapsulates RDP packets inside an SSL/TLS tunnel using **TCP Port 443 (HTTPS)**.
>   2. It terminates the SSL connection at the DMZ, authenticates the user, checks MFA, and validates access permissions against Connection Authorization Policies (CAP) and Resource Authorization Policies (RAP).
>   3. Only authorized RDP traffic is decrypted and forwarded internally to specific hosts.
> - **Result:** The internal network topology is hidden, port 3389 remains closed to the internet, and RDP access is encrypted and authenticated at the perimeter.

---
## Seedha Simple Mein
*Seedha simple mein: Remote Desktop Services (RDS) se multiple users ek hi server (Session Host) par ek sath login karke applications aur desktops use kar sakte hain. RD Gateway ke zariye RDP traffic port 443 (HTTPS) par securely wrap hokar internet par chalta hai, jisse systems ko direct network exposes se bachaya ja sake.*

---
## Related Notes
- [[02-Operating-Systems/03-Windows-OS/Windows-Services|Windows-Services]] — Managing system services like `TermService`.
- [[02-Operating-Systems/03-Windows-OS/FSLogix|FSLogix]] — Redirection profile management for RDS environments.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-14 VPN and Remote Access (RRAS)|WS-14 VPN and Remote Access (RRAS)]] — Alternative remote access options.

---
*Tags: #desktop-support #windows-os #rds #virtualization #L2*
