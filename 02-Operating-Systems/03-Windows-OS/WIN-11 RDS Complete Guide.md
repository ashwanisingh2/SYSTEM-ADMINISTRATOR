---
tags: [desktop-support, windows-os, rds, virtualization]
aliases: [rds-complete-guide, remote-desktop-services]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS**

`#complete` `#advanced`

# Remote Desktop Services (RDS) Complete Guide

> [!abstract] Overview
> This note covers Microsoft Remote Desktop Services (RDS) architecture, roles configuration, session host pools, RemoteApp publishing, and licensing administration. Ek support engineer ko yeh jaanna zaroori hai kyunki remote work environments mein users directly RDS ke through corporate apps aur desktops access karte hain.

---
## 🧠 Concept Overview

- **What it is** — Remote Desktop Services (RDS) is a server-based virtualization platform in Windows Server. It allows users to access remote desktop environments, session-based multi-user environments, and individual published applications (RemoteApps) hosted on central servers.
- **Why it matters** — Shipping physical corporate laptops to remote contractors or temporary workers is expensive, slow, and raises data security issues. RDS solves this by keeping all data inside the secure corporate datacenter, allowing remote users to stream their workspaces over encrypted connections.
- **Where you see this** — Supporting remote employees connecting to corporate environments, publishing line-of-business applications via web browsers, monitoring system performance on host servers, and resolving printer redirection issues.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Terminate hung user processes, disconnect frozen user sessions, assist users in mounting redirected local drives, and check host ping connectivity. |
| **L2** | Build and deploy RDS Session Collections, publish and restrict RemoteApps to security groups, install SSL certificates on Gateway servers, and configure session timeout limits. |
| **L3** | Architect highly available RDS Connection Broker clusters using SQL databases, implement Secure Remote Desktop Gateways with MFA/RADIUS, and manage user profile disks (UPD) or FSLogix storage environments. |

> [!tip] Seedha Simple Mein
> *Remote Desktop Services (RDS) se multiple users ek hi server (Session Host) par ek sath login karke applications aur desktops use kar sakte hain. RD Gateway ke zariye RDP traffic port 443 (HTTPS) par securely wrap hokar internet par chalta hai, jisse systems ko direct network exposes se bachaya ja sake.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **An RDS Environment** is like **a secure Corporate Office Building** because...
>
> - **RD Web Access** is the reception desk where you see the directory of available offices (apps).
> - **RD Gateway** is the security guard at the front door checking your ID card before letting you inside.
> - **RD Connection Broker** is the receptionist who directs you to a conference room that has empty seats (load balancing) or takes you back to the room you were already in (reconnection).
> - **RD Session Host** is the actual conference room where the work happens.
> - **RD Licensing** is the manager ensuring the company paid for enough chairs for everyone.

---
## 🔬 Technical Deep Dive

### 1. RDS Core Architecture Roles

> [!info] Key Concept
> An enterprise RDS environment is built from five distinct roles working in tandem to securely connect remote users to internal resources.

- **Remote Desktop Session Host (RDSH)**: The workhorse. This server hosts the actual desktop interfaces and applications that users run inside their sessions.
- **Remote Desktop Connection Broker (RDCB)**: The traffic cop. It handles incoming connection requests, load-balances users across available RDSH servers, and reconnects disconnected users to their existing sessions.
- **Remote Desktop Web Access (RDWA)**: The storefront. Provides users with a web portal (URL) to view and launch their allowed RemoteApps and virtual desktops.
- **Remote Desktop Gateway (RDGW)**: The security guard. Encapsulates standard RDP traffic inside an HTTPS (SSL/TLS port 443) wrapper. This allows remote workers to connect securely from public networks without using a standard VPN.
- **RD Licensing Server**: Manages Client Access Licenses (CALs) required to connect to the environment.

### 2. User Connection Lifecycle

1. The client browser downloads an `.rdp` file and launches the remote desktop client.
2. Connection request is sent to the **RD Gateway** (port 443).
3. The Gateway validates connection authorization policies (CAP) and resource authorization policies (RAP), decrypts HTTPS, and forwards RDP (port 3389) to the **Connection Broker**.
4. The Connection Broker checks the session database. If the user has a disconnected session, they are redirected to that host. Otherwise, it targets the host with the lowest active session load.
5. The **RDSH** requests a CAL license from the **Licensing Server**.
6. The session is established, loading the user profile via FSLogix or UPD.

### 3. Licensing Models: Device vs. User CALs

- **Per-User CAL**: Assigned to a specific user account. A user can connect from an unlimited number of devices. Best for environments where users have multiple machines.
- **Per-Device CAL**: Assigned to a physical device. Unlimited users can log in from that device. Best for shift-work offices or call centers sharing machines.

> [!danger] Common Mistake
> Forgetting to install and configure the RD Licensing Server within the 120-day grace period. Once the grace period expires, no users will be able to connect!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server 2022 domain controller (`DC-01.corp.local`).
> - A member server (`SVR-RDS01.corp.local`) assigned a static IP (`192.168.10.20`).
> - Local administrator privileges.

### Step 1: Install RDS Roles

```powershell
# Install core RDS host and broker roles
Install-WindowsFeature -Name Remote-Desktop-Services, RDS-RD-Server, RDS-Connection-Broker, RDS-Web-Access -IncludeManagementTools
```

> [!success] Expected Output
> ```text
> Success Restart Needed Exit Code Feature Result
> ------- -------------- --------- --------------
> True    Yes            Success   {Remote Desktop Services, Remote ...}
> ```

Restart the server: `Restart-Computer -Force`.

### Step 2: Create a Session Collection

```powershell
# Define a new collection grouping session hosts
New-RDSessionCollection -CollectionName "Finance-Apps" -SessionHost "SVR-RDS01.corp.local" -ConnectionBroker "SVR-RDS01.corp.local" -CollectionDescription "Hosted apps for Finance team"
```

> [!success] Expected Output
> Collection created successfully.

### Step 3: Publish RemoteApp Programs

```powershell
# Publish Calculator
New-RDRemoteApp -CollectionName "Finance-Apps" -Alias "Calculator" -DisplayName "Corp Calc" -FilePath "C:\Windows\System32\calc.exe" -ConnectionBroker "SVR-RDS01.corp.local"

# Publish Notepad
New-RDRemoteApp -CollectionName "Finance-Apps" -Alias "Notepad" -DisplayName "Corp Notepad" -FilePath "C:\Windows\System32\notepad.exe" -ConnectionBroker "SVR-RDS01.corp.local"
```

### Step 4: Configure Session Timeout Policies

```powershell
# Set Session Limits
Set-RDSessionCollectionConfiguration -CollectionName "Finance-Apps" -ConnectionBroker "SVR-RDS01.corp.local" -ConnectionActiveSessionLimitMin 480 -ConnectionIdleSessionLimitMin 60 -ConnectionDisconnectedSessionLimitMin 15 -AutomaticReconnectionEnabled $true
```

> [!tip] Pro Tip
> This logs off disconnected sessions after 15 minutes and idle sessions after 60 minutes, freeing up CPU and memory.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-RDUserSession` | Lists active and disconnected user sessions on host | `Get-RDUserSession -ConnectionBroker "SVR-RDS01.corp.local"` |
| `Invoke-RDUserSignOut` | Forces logoff of a target user session | `Invoke-RDUserSignOut -HostServer "RDSH-01" -UnifiedSessionId 3` |
| `Get-RDSessionHost` | Queries status of hosts in a collection | `Get-RDSessionHost -CollectionName "Finance-Apps"` |
| `Set-RDSessionHost` | Changes host state (e.g. enabling drain mode) | `Set-RDSessionHost -NewConnectionAllowed No` |
| `mstsc /v:<IP>` | Launches RDP client connection from command line | `mstsc /v:192.168.10.20` |
| `mstsc /admin` | Bypasses broker redirect to log in directly to host console | `mstsc /v:192.168.10.20 /admin` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **"The remote session was disconnected because there are no Remote Desktop License Servers available."** | The grace period (120 days) has expired, or the licensing server is not configured in group policy. | Set the RDS licensing mode and server address using Local Group Policy (`gpedit.msc`). |
| **"The host server is currently not accepting new connections."** | The target Session Host is configured in Drain Mode. | `Set-RDSessionHost -NewConnectionAllowed Yes` |
| **Certificate warning: "The identity of the remote computer cannot be verified."** | The RDS deployment is using default self-signed SSL certificates. | Install a publicly trusted SSL certificate and bind it to Gateway, Broker, and Web Access roles. |
| **User session hangs on a black screen when connecting.** | Display driver mismatch or bitmap caching issues on client. | Disable "Persistent bitmap caching" in RDP client options. Restart RDS Session Host agent services. |
| **Printer redirection fails.** | The required print driver is not installed on the Session Host server. | Install the matching printer driver on the RDSH server, or enable the "Easy Print driver" policy in GPO. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Hung User Application

> [!example] Ticket
> "My RemoteApp Calculator froze and I can't close it or open a new one."

**L1 Response:** Use Server Manager or PowerShell (`Get-RDUserSession` and `Invoke-RDUserSignOut`) to identify and forcefully log off the user's hung session.
**Escalation Trigger:** The user's session keeps freezing immediately after multiple logoffs.
**L2 Resolution:** Check RDSH server event logs for application crashes or profile corruption (FSLogix/UPD issues). Rebuild user profile if necessary.

---
## 🎤 Interview Questions

> [!question] Q1: How do you log off a hung user session from an RDS host using the command line?
> **Answer:** First, run `query user` or `Get-RDUserSession` to locate the user's Session ID. Once identified, run `logoff <Session_ID>` (or `Invoke-RDUserSignOut` in PowerShell) to force the hung user session to log off and free up server resources.

> [!question] Q2: What is "Drain Mode" on an RDS Session Host, and when would you use it?
> **Answer:** Drain Mode prevents new user sessions from connecting to a specific RD Session Host server while allowing users who are currently logged in to continue their work. This is used during maintenance windows, allowing administrators to patch or reboot hosts without abruptly terminating active user sessions.

> [!question] Q3: Explain the role of the RD Gateway in an enterprise network. How does it protect the network compared to direct port forwarding?
> **Answer:** Direct port forwarding exposes UDP/TCP port 3389 directly to the public internet, exposing hosts to brute force attacks. The **RD Gateway** acts as a reverse proxy at the network boundary, encapsulating RDP packets inside an SSL/TLS tunnel using **TCP Port 443 (HTTPS)**. It authenticates the user, checks MFA, and only forwards authorized traffic internally to port 3389.

==**Exam Tip:** Port 443 (HTTPS) is used externally by RD Gateway, while Port 3389 (RDP) is used internally by RD Connection Broker and RD Session Hosts.==

---
## 🔗 Related Notes

- [[02-Operating-Systems/03-Windows-OS/WIN-08 Windows Services|WIN-08 Windows Services]] — Managing system services like `TermService`.
- [[02-Operating-Systems/03-Windows-OS/WIN-10 FSLogix|WIN-10 FSLogix]] — Redirection profile management for RDS environments.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-14 VPN and Remote Access (RRAS)|WS-14 VPN and Remote Access (RRAS)]] — Alternative remote access options.
