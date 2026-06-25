---
tags: [desktop-support, networking, tcp-ip, ports, L1]
aliases: [ports-list, tcp-handshake]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #ccna
---

# TCP/IP and Ports

---

## Concept Overview
- **What it is**: The Transmission Control Protocol/Internet Protocol (TCP/IP) suite is the standard communication framework of the internet. Ports are logical sub-addresses (numbers 0 to 65535) used to direct network traffic to specific services running on a host.
- **Why it matters for a support engineer**: A support engineer must understand transport protocols and port mappings to configure firewall rules, troubleshoot service reachability, and diagnose network security policy blocks.
- **Where you encounter this in real job**: Verifying if RDP (port 3389) or SMB (port 445) is active, identifying port conflicts on local web servers, troubleshooting email connection failures, and analyzing network connections.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Identifies connection issues, runs simple ping/telnet tests, and verifies local listening ports using netstat.
  - **L2**: Configures firewall rules, troubleshoots connection resets, and analyzes TCP handshakes using packet captures.
  - **L3**: Designs corporate routing architectures, manages WAN firewall security policies, and designs port forwarding rules.

---

## Technical Deep Dive

### 1. TCP vs. UDP
The Transport Layer (OSI Layer 4) uses two main protocols:
- **TCP (Transmission Control Protocol)**: Connection-oriented. Establishes a connection before sending data, guarantees packet delivery, performs error checking, and manages flow control. Used for critical data (HTTP, SSH, SMB).
- **UDP (User Datagram Protocol)**: Connectionless. Sends data without verification (best-effort delivery), has lower overhead, and does not perform flow control. Used for real-time applications (VoIP, DNS, video streaming).

### 2. The TCP 3-Way Handshake & 4-Way Close
To establish a secure TCP connection, the sender and receiver run a 3-way handshake:
1. **SYN**: Client sends a SYN (Synchronize) packet containing a random sequence number (Seq = X) to the server.
2. **SYN-ACK**: Server replies with a SYN-ACK packet. It acknowledges the client's packet (Ack = X + 1) and sends its own sequence number (Seq = Y).
3. **ACK**: Client replies with an ACK (Acknowledge) packet (Ack = Y + 1), establishing the connection.

To terminate a connection, a 4-way handshake is run using **FIN** (Finish) and **ACK** packets from both sides to close the connection socket.

```
Client                                     Server
  | ------------[ SYN (Seq=X) ]----------->  |
  | <-------[ SYN-ACK (Seq=Y, Ack=X+1) ]---  |
  | ------------[ ACK (Ack=Y+1) ]--------->  |
```

### 3. Common Enterprise Ports Table
| Port Number | Protocol | Default Service | Core Function |
|-------------|----------|-----------------|---------------|
| **21** | TCP | FTP | File Transfer Protocol control |
| **22** | TCP | SSH / SFTP | Secure Shell remote login |
| **23** | TCP | Telnet | Legacy unencrypted remote command line |
| **25** | TCP | SMTP | Simple Mail Transfer Protocol (sending mail) |
| **53** | UDP/TCP | DNS | Domain Name System resolution |
| **80** | TCP | HTTP | Unencrypted web page traffic |
| **110** | TCP | POP3 | Post Office Protocol (retrieving mail) |
| **135** | TCP | RPC | Remote Procedure Call endpoint mapper |
| **137-139** | UDP/TCP | NetBIOS | Legacy Windows name resolution & service binding |
| **143** | TCP | IMAP | Internet Message Access Protocol (syncing mail) |
| **443** | TCP | HTTPS | Secure encrypted web page traffic |
| **445** | TCP | SMB | Server Message Block (Windows file sharing) |
| **3389** | TCP | RDP | Remote Desktop Protocol |

---

## Commands & Syntax

### PowerShell
```powershell
# Verify if RDP port 3389 is active and reachable on a remote server
Test-NetConnection -ComputerName "192.168.10.10" -Port 3389

# Get all active network connections and the processes running them
Get-NetTCPConnection | Where-Object {$_.State -eq "Listen"} |
    Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State
```

### CMD / Run Box
```cmd
REM Display all active TCP connections and listening ports with PIDs
netstat -ano
REM Find which process ID (PID) is listening on port 445
netstat -ano | findstr :445
```

### GUI Path
> Start -> Run -> type `resmon` -> Network Tab -> Expand **Listening Ports** to view active ports and processes.

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\PortNumber
HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 7036 | Service status changed (e.g., Firewall service started/stopped) | System Log |
| 5156 | Windows Filtering Platform allowed a connection | Security Log |
| 5157 | Windows Filtering Platform blocked a connection | Security Log |

---

## Real-World Scenarios

### Scenario 1: Administrator Cannot RDP to Remote Workstation (Port Configuration Issue)
**User Complaint:** "I am trying to connect to a user's workstation using Remote Desktop to install software, but the connection times out."
**Your First 3 Checks:**
1. Check if the workstation is powered on and connected to the network.
2. Verify if the Remote Desktop service is enabled in the workstation settings.
3. Test if RDP port 3389 is open using `Test-NetConnection`.
**Diagnosis Steps:**
1. Ping the workstation IP. Ping succeeds, confirming network reachability.
2. Test port 3389: `Test-NetConnection -ComputerName "192.168.10.55" -Port 3389`
   - Output: `TcpTestSucceeded : False`
3. Log into the workstation locally or via remote registry. Navigate to the Terminal Server registry path:
   `HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp`
   - Observe the `PortNumber` value. It is set to `3390` (a custom security port configuration).
**Root Cause:** The default RDP port was changed to 3390, causing connection attempts on default port 3389 to time out.
**Fix:** Connect to RDP appending the custom port number (`192.168.10.55:3390`) or restore the registry value to the default `3389`.
**Prevention:** Maintain a centralized inventory of custom port mappings for corporate endpoints.
**Ticket Close Note:** "Diagnosed custom RDP port configuration (3390). Connected using custom port. Restored default 3389 port mapping. Closing ticket."

### Scenario 2: Web Server Fails to Start Due to Port Conflict
**User Complaint:** "I am trying to launch our local web server application for testing, but it fails to start and logs an error stating address is already in use."
**Your First 3 Checks:**
1. Identify the default port used by the web application (typically 80 or 443).
2. Check which active process is currently holding the target port.
3. Check the application error logs for socket errors.
**Diagnosis Steps:**
1. Open command prompt as Administrator.
2. Run the command to locate the process holding port 80:
   `netstat -ano | findstr :80`
   - Output: `TCP 0.0.0.0:80 0.0.0.0:0 LISTENING 4122`
3. Match the PID (4122) to the process name using Task Manager or PowerShell:
   `Get-Process -Id 4122`
   - Output: `Name: Skype`
**Root Cause:** Another application (Skype) was configured to bind to port 80 for incoming connections, creating a port conflict that blocked the web server application.
**Fix:** Stop the conflicting process using `Stop-Process` or reconfigure Skype to use alternative ports.
**Prevention:** Change the web server configuration to bind to alternative test ports (e.g., port 8080) to avoid conflicts.
**Ticket Close Note:** "Identified port conflict on port 80 caused by Skype (PID 4122). Terminated the process. Configured the web application to use port 8080. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Expose legacy unencrypted ports (such as Telnet port 23, FTP port 21, or HTTP port 80) to the public internet.
> - Data sent through these ports is unencrypted, allowing attackers to sniff passwords and traffic.
> - Always enforce secure alternatives (SSH, SFTP, HTTPS).

> [!warning] Common Trap
> - Assuming that a failed port ping test (`ping host`) means the port is closed.
> - Ping uses the ICMP protocol, which does not use TCP or UDP ports. A host can block ping packets while its web server ports remain open.
> - Always use port verification tools (like `Test-NetConnection` or `telnet`) to verify port status.

> [!tip] Senior Engineer Tip
> - If you need to quickly verify if a remote port is listening without using PowerShell, open a command prompt and run:
>   `telnet host port`
>   - If the screen goes blank, the port is open and listening. (Enable Telnet Client in Windows Features first).

> [!success] Verification Steps
> - Run: `Test-NetConnection -ComputerName "localhost" -Port 8080` to verify port status.
> - Confirm the connection test succeeds.

> [!question] Interview Alert
> - "Explain the difference between TCP and UDP, and give an example protocol of each."
> - Answer: "TCP is a connection-oriented, reliable protocol that guarantees packet delivery using a 3-way handshake and error checking, used by HTTP. UDP is a connectionless, fast protocol that sends packets without verification, used by DNS and VoIP services."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Unsecured administrative ports | Exposing RDP port 3389 publicly | Use VPN tunnels or Azure Bastion to access remote administration ports securely. |
| Neglecting port capacity limits | Leaving dynamic port ranges default | Configure custom dynamic port ranges if local services exhaust port capacity. |
| Troubleshooting connection drops as port issues | Failing to check route paths | Run traceroute diagnostics to check routing paths before checking port configurations. |

---

## Lab Exercise

**Objective:** Verify RDP service status, change the default RDP port mapping in the registry, and confirm the custom port is listening.
**Time Required:** 20 minutes
**Environment Needed:** A Windows 11 VM or physical machine with local administrator rights.
**Pre-requisites:** Backup the registry path before editing.

**Steps:**
1. Open the Registry Editor: Press `Win + R` -> type `regedit`.
2. Navigate to the RDP port configuration registry path:
   `HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp`
3. Locate the `PortNumber` DWORD. Right-click -> **Modify** -> Change Base to **Decimal** -> Enter value `3390`. Click **OK**.
4. Restart the Remote Desktop service to apply changes:
   ```powershell
   Restart-Service -Name "TermService" -Force
   ```
5. Open command prompt as Administrator and verify the new port is listening:
   ```cmd
   netstat -ano | findstr :3390
   ```
   - Expected output: `TCP 0.0.0.0:3390 0.0.0.0:0 LISTENING [PID]`
6. Verification: Test local connection to the custom port.
   ```powershell
   Test-NetConnection -ComputerName "127.0.0.1" -Port 3390
   ```
   - Expected output: `TcpTestSucceeded : True`

**Success Criteria:** The RDP service binds to port 3390, and `Test-NetConnection` returns `True`.
**Common Failures:** Failed connection if Windows Defender Firewall blocks the custom port. Add a firewall exception rule for port 3390.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a port conflict and how do you resolve it?**
A: A port conflict occurs when two applications attempt to bind to the same network port on a system simultaneously. To resolve it, I run `netstat -ano` to find the conflicting process ID (PID), terminate it using Task Manager or `Stop-Process`, or reconfigure one of the applications to use an alternative port.

**Q: Which ports are used for sending and receiving email?**
A: SMTP (Simple Mail Transfer Protocol) uses port 25 (or secure port 587) to send emails. To receive and sync emails, POP3 uses port 110 (or secure port 995), and IMAP uses port 143 (or secure port 993).

### Intermediate (L2 Level)
**Q: What are the three flags used in a TCP 3-way handshake?**
A: The three steps use the **SYN** (Synchronize) flag sent by the client, the **SYN-ACK** (Synchronize-Acknowledgment) flag returned by the server, and the **ACK** (Acknowledgment) flag sent by the client to establish the connection.

### Advanced (L3/Senior Level)
**Q: A user reports they cannot connect to a critical file share on a remote subnet. Ping succeeds, but connection attempts to `\\server` fail. Explain your diagnostic process.**
A:
- **Situation**: A user was unable to access a file share, though basic Layer 3 routing was functional.
- **Task**: I needed to identify if the target SMB port was open and reachable.
- **Action**: I tested SMB port 445 connectivity using `Test-NetConnection -Port 445`. The connection failed. I verified that the Server service was active on the target file server. I inspected the network firewall configuration and observed that traffic on port 445 was blocked between the subnets for security compliance. I requested a firewall exception permitting SMB port 445 traffic between the subnets.
- **Result**: The connection test succeeded, and the file share became accessible.

### HR / Behavioral
**Q: Tell me about a time you had to resolve a service outage caused by a firewall configuration change.**
A: After a firewall update, users lost access to our local ticketing system. I ran port checks, found that port 80/443 traffic was blocked on the segment, and notified the security team to restore the rule, returning the system online.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: Logical address channels (0-65535) used to direct network traffic to specific services on a host.
> **Why**: Critical for configuring firewall policies, troubleshooting service reachability, and resolving port conflicts.
> **How**: Use netstat to list listening ports, and verify remote ports using Test-NetConnection.
> **Command**: `netstat -ano | findstr :3389`
> **Interview Answer Starter**: "In my experience, troubleshooting port issues requires verifying if the target service is listening locally using netstat, and testing remote access using Test-NetConnection..."

**Key Numbers to Remember:**
- RDP Port: 3389 (TCP)
- SMB Port: 445 (TCP)
- Dynamic / Ephemeral port range: 49152 - 65535

**3 Things Interviewer Wants to Hear:**
1. Difference between TCP (reliable) and UDP (fast) protocols
2. TCP 3-way handshake sequence (SYN -> SYN-ACK -> ACK)
3. Using port checks (telnet/powershell) to isolate firewall blocks

---

## Related Notes
- [[01-Foundations/02-Networking/OSI-Model|OSI Model]] — Details the transport layer context.
- [[01-Foundations/02-Networking/DNS|DNS]] — Operating protocol on UDP port 53.
- [[01-Foundations/02-Networking/DHCP|DHCP]] — Operating protocol on UDP ports 67/68.

---

## Tags
#desktop-support #networking #tcp-ip #ports #L1 #interview-topic #lab-complete #daily-use
