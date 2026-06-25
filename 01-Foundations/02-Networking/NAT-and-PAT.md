---
tags: [desktop-support, networking, nat, pat, L2]
aliases: [network-address-translation, nat-overload]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

# NAT and PAT

---

## Concept Overview
- **What it is**: Network Address Translation (NAT) and Port Address Translation (PAT) are networking protocols used to translate private IP addresses (RFC 1918) within a local network to a public IP address (and vice versa) for routing over the public internet.
- **Why it matters for a support engineer**: A support engineer must understand address translation to troubleshoot connection routing failures, configure remote access to internal servers, and diagnose port forward blocks.
- **Where you encounter this in real job**: Diagnosing "Double NAT" errors on remote user connections, configuring port forwarding for local servers, troubleshooting internal web application access, and setting up network routing tables.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Identifies basic public vs. private IP configurations, checks internet gateway reachability, and identifies local connection drops.
  - **L2**: Configures basic port forwarding (Static NAT/PAT), troubleshoots double NAT, and verifies network connection state tables.
  - **L3**: Designs public IP addressing pools, manages enterprise firewall translation pools, and designs routing boundaries.

---

## Technical Deep Dive

### 1. The IP Address Shortage & Translation Need
IPv4 uses a 32-bit address space, offering approximately 4.3 billion addresses. Because of the rapid growth of network devices, IPv4 addresses were exhausted. NAT resolves this by allowing thousands of local devices to share a single public IP address.

### 2. Types of Translation
- **Static NAT (One-to-One)**: Maps a single private IP address to a single public IP address permanently. Used when a local server (e.g., a web server at `192.168.10.25`) must be accessible from the internet at static public IP `203.0.113.25`.
- **Dynamic NAT (Many-to-Many)**: Maps private IP addresses to any available public IP from a pre-configured pool of public IP addresses.
- **PAT (Port Address Translation / NAT Overload)**: Maps multiple private IP addresses to a single public IP address by assigning a unique TCP/UDP source port number to each connection. This is the default configuration on home and corporate routers.

```
Internal LAN (Private)                      Gateway Router                     External WAN (Public)
`192.168.10.5:1234`   ---[ PAT Translation ]--->  `203.0.113.1:50001`  --->  `142.250.190.46:443`
`192.168.10.6:1234`   ---[ PAT Translation ]--->  `203.0.113.1:50002`  --->  `142.250.190.46:443`
```

### 3. The PAT Translation Table
The router maintains a NAT translation table in memory to track connections:

| Internal Source IP | Internal Source Port | Translated Public IP | Translated Port | Destination IP | Destination Port |
|---|---|---|---|---|---|
| `192.168.10.5` | 1024 | `203.0.113.1` | **50001** | `142.250.190.46` | 443 |
| `192.168.10.6` | 1024 | `203.0.113.1` | **50002** | `142.250.190.46` | 443 |

When returning packets arrive from the destination, the router checks the translated port number in the destination port field, matches it to the table, updates the IP header to the original private IP, and forwards the packet to the correct client.

---

## Commands & Syntax

### PowerShell
```powershell
# Query current local IP routing configurations and adapter metrics
Get-NetRoute | Where-Object {$_.DestinationPrefix -eq "0.0.0.0/0"} |
    Select-Object DestinationPrefix, NextHop, RouteMetric
```

### CMD / Run Box
```cmd
REM Query public IP address from client CLI using public API
curl ifconfig.me
REM Output current route path to external target to check gateway address
tracert 8.8.8.8
```

### GUI Path
> Open browser -> Go to `https://www.whatsmyip.org` to check your translated public IP.
> Or: Access your local gateway router Web GUI -> **NAT/Port Forwarding** settings.

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\
HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 5156 | Windows Filtering Platform allowed a connection | Security Log |
| 5157 | Windows Filtering Platform blocked a connection | Security Log |
| 20227 | Routing and Remote Access Service connection failure | Application Log |

---

## Real-World Scenarios

### Scenario 1: User Experiencing VoIP Dropouts (Double NAT Conflict)
**User Complaint:** "I am working from home, and my softphone application drops calls constantly. Sometimes my calls have no audio, or disconnect after 10 seconds."
**Your First 3 Checks:**
1. Check the client's local IP address and default gateway IP.
2. Check the public WAN IP of the user's home network router.
3. Check the path route using `tracert`.
**Diagnosis Steps:**
1. Run `ipconfig` on the client:
   - IP address: `192.168.10.15` -> Gateway: `192.168.10.1` (User's personal router).
2. Open the user's router status page. The WAN IP of the router is set to `192.168.1.105` (indicating it is connected to a primary modem/router provided by their ISP).
3. Run `tracert 8.8.8.8`:
   - Hop 1: `192.168.10.1` (Private IP range)
   - Hop 2: `192.168.1.1` (Private IP range)
   - This confirms a **Double NAT** configuration.
**Root Cause:** The user has cascaded two routers running active NAT services. Packets are translated twice, breaking peer-to-peer VoIP connection handshakes and port mappings.
**Fix:** Configure the ISP-provided modem/router to **Bridge Mode** (disabling its internal routing and NAT), or configure the user's secondary router to function as an Access Point (AP) only.
**Prevention:** Educate remote workers during onboarding on proper home network routing topologies.
**Ticket Close Note:** "Diagnosed Double NAT configuration. Configured the primary ISP modem to Bridge Mode. Verified call connection stability. Closing ticket."

### Scenario 2: Web Server on Private Subnet Unreachable (Static NAT Fail)
**User Complaint:** "I configured an internal staging web server on our LAN, but outside developers cannot access it via the public IP we assigned."
**Your First 3 Checks:**
1. Check if the web server is listening on port 80/443 locally.
2. Check if the firewall permits inbound port 80/443 traffic.
3. Check the Static NAT mapping on the corporate edge firewall.
**Diagnosis Steps:**
1. Run a local port check on the web server to confirm it is listening.
2. Check the edge firewall NAT policies:
   - Static NAT rule: Public IP `203.0.113.25` maps to Private IP `192.168.10.25`.
3. Check the corresponding Security Policy (Access Control List):
   - The security policy lacks an inbound rule permitting TCP port 80/443 traffic to the public destination IP.
**Root Cause:** The Static NAT translation rule was active, but the firewall security policy blocked inbound traffic to the port.
**Fix:** Add an inbound security rule to the firewall allowing TCP port 80/443 traffic from any source to the web server's private IP.
**Prevention:** Always pair NAT mapping rules with corresponding inbound security policy rules on the edge firewall.
**Ticket Close Note:** "Added firewall inbound security rule allowing TCP 80/443 traffic to the web server's private IP (192.168.10.25). Verified access. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Configure port forwarding (Static NAT) to expose internal administrative services (such as RDP port 3389 or Telnet port 23) directly to the public internet.
> - This exposes these services to port scanners and brute-force attacks, risking network intrusion.
> - Enforce VPN access policies for administrative connections.

> [!warning] Common Trap
> - Assuming that dynamic NAT or PAT alone acts as a security firewall.
> - While translation prevents direct incoming connections to internal IPs, it does not inspect outgoing traffic or protect against malicious downloads.
> - Always configure active firewall security policies alongside translation rules.

> [!tip] Senior Engineer Tip
> - If you suspect a PAT gateway is experiencing connection dropouts under heavy loads, check for **PAT Port Exhaustion**. If a single public IP exhausts its 65,000 available ports, configure the gateway to use a pool of public IPs (NAT Overload with address pools).

> [!success] Verification Steps
> - Run: `curl ifconfig.me` from a client on the LAN.
> - Verify that the returned IP address matches the corporate firewall's public WAN IP.

> [!question] Interview Alert
> - "Explain the difference between NAT and PAT."
> - Answer: "NAT translates IP addresses. Static NAT maps one private IP to one public IP, while Dynamic NAT maps private IPs to a pool of public IPs. PAT (Port Address Translation) maps multiple private IPs to a single public IP by assigning unique source port numbers to each connection, allowing thousands of devices to share one public IP."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Double NAT configurations | Connecting router WAN port to modem LAN | Configure the primary modem to Bridge Mode, or set the secondary router to Access Point mode. |
| Outdated NAT translation tables | High TCP connection timeouts | Configure lower idle timeout values on the gateway to reclaim ports from inactive connections. |
| Asymmetric routing loops | Incorrect gateway configurations | Ensure return traffic passes through the same gateway router that performed the translation. |

---

## Lab Exercise

**Objective:** Configure a port forwarding rule (Static PAT) on a gateway router to allow remote access to a local web server, and verify translation.
**Time Required:** 30 minutes
**Environment Needed:** A local gateway router or firewall, a web server VM, and a client VM.
**Pre-requisites:** Administrative access to the gateway.

**Steps:**
1. Log into your gateway router's administration interface.
2. Verify the Web Server IP is set statically to `192.168.10.25`. Confirm the web service is active on port `80`.
3. Navigate to **NAT/Routing** -> **Port Forwarding** (or Virtual Server settings).
4. Create a new forwarding rule:
   - Application Name: `Staging Web Server`
   - External Port: `8080` (use alternate port for security)
   - Internal Port: `80`
   - Protocol: `TCP`
   - Device IP Address: `192.168.10.25`
5. Save the configuration and restart the router services (if required).
6. Connect the client VM to the external WAN network.
7. Open a browser on the client and enter: `http://[Router-WAN-IP]:8080`
   - Expected output: The staging web page loads successfully.
8. Verification: Run the verification command from the external client.
   ```powershell
   Test-NetConnection -ComputerName "[Router-WAN-IP]" -Port 8080
   ```
   - Expected output: `TcpTestSucceeded : True`

**Success Criteria:** The external client successfully accesses the internal web server via port 8080 on the WAN interface.
**Common Failures:** Connection failure if the gateway firewall blocks incoming traffic on the external port.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the main purpose of NAT?**
A: The main purpose of NAT is to conserve the limited IPv4 address space. It allows multiple devices on a private local network to share one or more public IP addresses to access the public internet.

**Q: What range of IP addresses are reserved for private networks?**
A: RFC 1918 reserves three ranges for private local networks: `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`. These IP addresses are not routable on the public internet.

### Intermediate (L2 Level)
**Q: Explain what Double NAT is, and how it impacts applications.**
A: Double NAT occurs when you connect a private router to another router that is also performing NAT, creating two translation layers (e.g., Hop 1: `192.168.10.1` -> Hop 2: `192.168.1.1`). This breaks peer-to-peer protocols, online gaming, and VPN connections because return packets get lost or blocked at the secondary translation layer.

### Advanced (L3/Senior Level)
**Q: An organization's remote access gateway experiences connection timeouts during peak hours. You suspect PAT Port Exhaustion. Explain your diagnostic and mitigation strategy.**
A:
- **Situation**: A remote access gateway was dropping connection handshakes during peak hours.
- **Task**: I needed to verify if the gateway had exhausted its available translation ports and implement a fix.
- **Action**: I logged into the gateway firewall command line and checked the active translation table size. I observed that the active translation count had reached 64,000, confirming PAT Port Exhaustion. To resolve this, I configured a NAT address pool containing three additional public IP addresses and updated the translation policy to use this pool for dynamic translation.
- **Result**: The translation limit expanded to over 250,000 concurrent connections, resolving the timeout issues.

### HR / Behavioral
**Q: Tell me about a time you had to troubleshoot a network access issue where the source of the problem was on the user's home network.**
A: A remote user could not connect to our corporate database. I ran a route trace on their system, identified a Double NAT configuration on their home network, and guided them to set their ISP modem to Bridge Mode, resolving the connection issue.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: Technology translating private IP addresses to public IP addresses for routing over the public internet.
> **Why**: Critical for conserving IPv4 addresses and managing access to internal network services.
> **How**: Configure Static NAT for servers, and implement PAT (NAT Overload) for client subnets.
> **Command**: `curl ifconfig.me`
> **Interview Answer Starter**: "In my experience, address translation issues are commonly caused by double-NAT configurations on remote networks or misconfigured port forwarding rules..."

**Key Numbers to Remember:**
- Public IP availability limit: ~4.3 billion (IPv4)
- PAT single IP connection limit: ~65,000 ports
- RFC 1918 Private Ranges: 10/8, 172.16/12, 192.168/16

**3 Things Interviewer Wants to Hear:**
1. Difference between Static NAT (1:1) and PAT (Overload)
2. How the router tracks connections using the NAT translation table
3. Troubleshooting Double NAT configurations on remote connections

---

## Related Notes
- [[01-Foundations/02-Networking/OSI-Model|OSI Model]] — Details the Network layer context.
- [[01-Foundations/02-Networking/TCP-IP-and-Ports|TCP/IP and Ports]] — Details transport port structures.
- [[01-Foundations/02-Networking/Subnetting|Subnetting]] — Details IP subnet design.

---

## Tags
#desktop-support #networking #nat #pat #L2 #interview-topic #lab-complete #daily-use
