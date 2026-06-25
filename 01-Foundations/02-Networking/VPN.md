---
tags: [desktop-support, networking, vpn, L2]
aliases: [virtual-private-network, remote-access]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

# VPN

---

## Concept Overview
- **What it is**: A Virtual Private Network (VPN) is a technology that establishes an encrypted communication tunnel over an untrusted public network (like the internet) to securely connect remote users or branch offices to private corporate networks.
- **Why it matters for a support engineer**: With remote and hybrid work models, VPN issues are one of the most common user tickets. A support engineer must understand tunnel protocols, routing configurations, and authentication flows to troubleshoot remote access failures.
- **Where you encounter this in real job**: Troubleshooting users unable to connect to the corporate network from home, configuring split-tunneling policies, setting up Cisco/Fortinet/GlobalProtect clients, and managing Always On VPN profiles.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Installs VPN client software, walks users through MFA authentication steps, verifies local home internet connections, and configures basic client settings.
  - **L2**: Troubleshoots connection routing errors, resolves certificate validation failures, and edits client configuration files.
  - **L3**: Configures VPN gateway concentrators, designs site-to-site tunnels between clouds and on-premises sites, and defines network access control (NAC) policies.

---

## Technical Deep Dive

### 1. VPN Tunneling Architectures
- **Site-to-Site VPN**: Connects entire networks together (e.g., connecting a branch office network to the main data center). Runs on hardware security appliances (routers/firewalls) using IPSec tunnels. Does not require client software on endpoints.
- **Remote Access (Client-to-Site)**: Connects individual endpoints to the corporate network. Requires client software (or built-in OS clients) to negotiate connection with a corporate VPN gateway.

### 2. Tunneling Protocols Comparison
| Protocol | Port / Transport | Encryption | Security Level | Pros/Cons |
|---|---|---|---|---|
| **PPTP** | TCP 1723 / GRE 47 | MPPE (128-bit) | Obsolete | Fast but highly vulnerable. Blocked by most ISPs. |
| **L2TP / IPSec**| UDP 500 / UDP 4500 | AES (256-bit) | High | Built-in OS support. Can be blocked by NAT firewalls. |
| **SSTP** | TCP 443 | SSL/TLS | High | Traverses almost all firewalls (looks like HTTPS). Windows-only. |
| **IKEv2** | UDP 500 / UDP 4500 | AES (256-bit) | High | Fast reconnects during network switching (good for mobile). |
| **OpenVPN** | TCP/UDP 1194 | OpenSSL | High | Open source, highly configurable, requires third-party client. |

### 3. Split-Tunneling vs. Full-Tunneling
- **Full-Tunneling**: Routes *all* internet and corporate traffic from the client device through the VPN tunnel to the corporate gateway. Secure (all traffic is filtered by corporate firewalls), but consumes high gateway bandwidth and increases latency for public services (e.g., Teams, YouTube).
- **Split-Tunneling**: Routes only traffic destined for the corporate subnets (e.g., `192.168.10.0/24`) through the VPN tunnel. Public internet traffic (e.g., Office 365) routes directly through the user's local ISP interface. Optimizes bandwidth but bypasses corporate edge security filters.

```
Full-Tunneling:  [Client] ===(All Traffic Enrypted)===> [Corporate Gateway] ---> [Internet]
Split-Tunneling: [Client] ===(Corp Traffic Enrypted)==> [Corporate Gateway]
                 [Client] ------------(Public Traffic Directly)------------> [Internet]
```

### 4. Always On VPN
Microsoft's replacement for DirectAccess. It establishes two automatic tunnels:
- **Device Tunnel**: Connects before user login using computer certificates. Enables device management, GPO updates, and domain logins.
- **User Tunnel**: Connects after user login, allowing access to user-level network resources (shares, intranet).

---

## Commands & Syntax

### PowerShell
```powershell
# Retrieve all configured VPN connection profiles on the system
Get-VpnConnection | Select-Object Name, ServerAddress, ConnectionStatus, SplitTunneling

# Create a new L2TP VPN connection profile with split tunneling enabled
New-VpnConnection -Name "CorpVPN" -ServerAddress "vpn.company.com" -TunnelType L2tp -L2tpPsk "SecretKey123" -SplitTunneling $true
```

### CMD / Run Box
```cmd
REM View routing table to verify if corporate subnets route through the VPN interface
route print
REM Check routing path to corporate host to verify tunnel usage
tracert 192.168.10.10
```

### GUI Path
> Start -> Settings -> Network & Internet -> **VPN** -> Add VPN / Configure active profiles
> Or: Control Panel -> Network and Sharing Center -> **Set up a new connection or network**

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\RasMan\Parameters\
HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 20227 | VPN client connection failure (includes error code) | Application Log |
| 20226 | VPN client connected successfully | Application Log |
| 987 | Certificate verification failure during VPN handshake | System Log |

---

## Real-World Scenarios

### Scenario 1: User Cannot Connect to VPN (MFA Loop / Timeout Failure)
**User Complaint:** "I open the corporate VPN client, type in my password, and click connect. It prompts me for MFA on my phone, but after I approve it, the client spins and returns an authentication timeout error."
**Your First 3 Checks:**
1. Check the local client network connection latency and packet loss.
2. Verify if the user's account is locked or disabled in Active Directory.
3. Check the Network Policy Server (NPS) logs on the authentication server.
**Diagnosis Steps:**
1. Run a ping check to a public IP to verify basic home connection stability.
2. Review the user's AD account. The account is active and unlocked.
3. Check the NPS authentication server event log (Event ID 6273 - Access Rejected/Timeout):
   - The log shows the authentication succeeded, but the client failed to respond within the 30-second window.
   - The user took 45 seconds to unlock their phone and approve the authenticator prompt.
**Root Cause:** The authentication request timed out because the user delayed approving the MFA prompt, exceeding the VPN gateway's default handshake timeout configuration.
**Fix:** Instruct the user to keep their phone ready before clicking connect. Alternatively, increase the connection timeout setting on the VPN concentrator gateway.
**Prevention:** Train users during onboarding to have the authenticator app open before initiating the VPN login sequence.
**Ticket Close Note:** "User authentication timeout resolved. Instructed user to approve MFA prompts immediately. Verified successful connection. Closing ticket."

### Scenario 2: User Cannot Access Local Printers While Connected to VPN
**User Complaint:** "When I am connected to the VPN to work from home, I cannot print to my local home Wi-Fi printer. If I disconnect the VPN, printing works immediately."
**Your First 3 Checks:**
1. Check the VPN profile split-tunneling status in PowerShell.
2. View the local client routing table using `route print`.
3. Check the IP subnet of the user's home network.
**Diagnosis Steps:**
1. Run: `Get-VpnConnection` on the client.
   - Output shows: `SplitTunneling : False` (indicating a full-tunnel configuration).
2. Because split-tunneling is disabled, all traffic (including local print requests) is routed into the VPN tunnel to the corporate gateway, which drops the traffic destined for the home subnet.
3. Run `route print`. The default route (`0.0.0.0/0`) points directly to the virtual VPN interface.
**Root Cause:** The VPN client was configured for full-tunneling, routing local subnet print traffic away from the local LAN interface.
**Fix:** Enable split-tunneling on the client's VPN connection properties or deploy a policy updating the profile.
```powershell
Set-VpnConnection -Name "CorpVPN" -SplitTunneling $true
```
**Prevention:** Standardize split-tunneling configurations for all BYOD and corporate laptop VPN profiles.
**Ticket Close Note:** "Enabled split tunneling on the user's VPN connection profile. Verified user can access internal servers and print to their local LAN printer simultaneously. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Deploy remote access VPN profiles using legacy unencrypted protocols like PPTP or using weak pre-shared keys (PSKs) distributed in cleartext files.
> - This allows attackers to perform man-in-the-middle attacks to intercept VPN credentials and access the internal network.
> - Enforce L2TP/IPSec, SSTP, or IKEv2 with certificate-based authentication.

> [!warning] Common Trap
> - Enabling split-tunneling without configuring DNS routing policies.
> - If split-tunneling is enabled but the client continues to route all DNS queries through the local home ISP, users will fail to resolve internal servers (e.g., `dc01.lab.local`).
> - Ensure the VPN profile distributes corporate DNS suffixes and registers them in the client's NRPT (Name Resolution Policy Table).

> [!tip] Senior Engineer Tip
> - If L2TP/IPSec VPN connections fail behind home consumer routers, it is often caused by NAT-T (Network Address Translation Traversal) support. Enable the following registry key on the Windows client to permit IPSec connections behind NAT devices:
>   - Path: `HKLM\SYSTEM\CurrentControlSet\Services\PolicyAgent`
>   - DWORD: `AssumeUDPEncapsulationContextOnSendRule` = `2`

> [!success] Verification Steps
> - Verify the client status displays **Connected** in the system tray.
> - Run the command: `Get-VpnConnection -Name "CorpVPN"` to confirm that the connection status is active.

> [!question] Interview Alert
> - "Explain the difference between split-tunneling and full-tunneling, and the trade-offs of each."
> - Answer: "Full-tunneling routes all traffic through the secure VPN gateway, protecting all data but consuming high gateway bandwidth. Split-tunneling routes only corporate traffic through the tunnel and public internet traffic directly through the local ISP, saving bandwidth but bypassing corporate web security filters."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| MTU size mismatch on WAN | Large packets get dropped | Adjust MTU size settings (typically 1350 to 1400) on the VPN interface to prevent packet fragmentation. |
| Expired client certificates | Failure to audit certificate lifecycles | Configure auto-enrollment policies via GPO to renew user/device certificates before expiration. |
| Duplicate home and corporate subnets | Statically configuring `192.168.1.0/24` | Avoid using standard home IP ranges (like `192.168.1.x`) for corporate subnets to prevent routing conflicts. |

---

## Lab Exercise

**Objective:** Configure a client VPN profile with split-tunneling, verify routing configurations, and test secure connectivity to a target subnet.
**Time Required:** 20 minutes
**Environment Needed:** A Windows 11 VM and a remote gateway endpoint.
**Pre-requisites:** Administrative rights on the workstation VM.

**Steps:**
1. Open PowerShell as Administrator.
2. Create L2TP VPN Connection:
   ```powershell
   New-VpnConnection -Name "TestLabVPN" -ServerAddress "198.51.100.25" -TunnelType L2tp -L2tpPsk "LabKeySecret2026" -PassThru
   ```
3. Enable Split Tunneling:
   ```powershell
   Set-VpnConnection -Name "TestLabVPN" -SplitTunneling $true
   ```
4. Trigger Connection: Connect to the VPN via the network system tray or run:
   ```powershell
   Connect-VpnConnection -Name "TestLabVPN"
   ```
5. Verify Routing: Open a command prompt and print the routing table:
   ```cmd
   route print
   ```
   - Expected output: Verify that only the remote gateway network range (e.g., `192.168.20.0`) routes through the interface, while the default gateway (`0.0.0.0`) remains pointing to the local network router.
6. Verification: Test routing path to target IP.
   ```cmd
   tracert 192.168.20.10
   ```

**Success Criteria:** The VPN connects successfully, the default internet route stays local, and the route to `192.168.20.10` passes through the virtual VPN interface.
**Common Failures:** Connection failure if the L2TP PSK key or server IP is input incorrectly.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a VPN client and what does it do?**
A: A VPN client is a software application installed on an endpoint device that negotiates connection settings with a corporate VPN gateway, encrypts outgoing network packets, and decrypts incoming tunnel packets to establish a secure link.

**Q: What port does SSTP VPN use, and why is it useful?**
A: SSTP (Secure Socket Tunneling Protocol) uses TCP port 443 (standard HTTPS). It is useful because almost all network firewalls and public hotspots open port 443, allowing SSTP connections to bypass blocks that affect other protocols.

### Intermediate (L2 Level)
**Q: What is a split-tunnel VPN, and how does it affect local network printing?**
A: A split-tunnel VPN routes only corporate subnet traffic through the encrypted tunnel, leaving local network traffic (like home printers or LAN devices) routed through the local ISP interface. This allows users to print to local home printers while working on secure corporate files.

### Advanced (L3/Senior Level)
**Q: A hybrid worker reports they cannot log in to their Windows laptop on their first day working from home. Their password was reset by the helpdesk. Explain your resolution strategy.**
A:
- **Situation**: A remote user could not log in to their machine because their locally cached password did not match their updated domain password.
- **Task**: I needed to establish domain connectivity before login to synchronize the credentials.
- **Action**: I instructed the user to connect their laptop to their home router using an ethernet cable. On the login screen, I directed them to click the **Network Sign-In** icon (Always On VPN Device Tunnel). Once the device tunnel established connection to the domain, I instructed the user to enter their new password.
- **Result**: The local cache synchronized with the domain controller, and the user successfully logged in.

### HR / Behavioral
**Q: Tell me about a time you had to deal with a mass service failure affecting remote users. How did you coordinate the response?**
A: A VPN server certificate expiration caused all remote workers to disconnect. I set up a status bridge, compiled manual configuration profiles using alternative protocols (SSTP), and coordinated helpdesk staff to distribute instructions. We renewed the certificate and brought the primary VPN online in under an hour.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: Technology establishing encrypted network tunnels over public networks to access private resources.
> **Why**: Essential for securing remote workforces and linking branch networks.
> **How**: Configure tunneling protocols, manage routing tables, and troubleshoot client authentication.
> **Command**: `Get-VpnConnection`
> **Interview Answer Starter**: "In my experience, VPN troubleshooting starts by isolating home ISP latency from VPN gateway authentication events..."

**Key Numbers to Remember:**
- L2TP/IPSec Ports: UDP 500 / UDP 4500
- SSTP Port: TCP 443 (TLS)
- PPTP Port: TCP 1723

**3 Things Interviewer Wants to Hear:**
1. Difference between split-tunneling and full-tunneling
2. Troubleshooting protocol selection (e.g., choosing SSTP to traverse firewalls)
3. Using Device Tunnels in Always On VPN to update cached credentials

---

## Related Notes
- [[01-Foundations/02-Networking/OSI-Model|OSI Model]] — Details security tunneling layers.
- [[01-Foundations/02-Networking/TCP-IP-and-Ports|TCP/IP and Ports]] — Details transport port mappings (443, 500, 4500).
- [[04-Cloud-and-Security/09-Security/MFA-and-Identity-Protection|MFA and Identity Protection]] — Details integration with authentication.

---

## Tags
#desktop-support #networking #vpn #L2 #interview-topic #lab-complete #daily-use
