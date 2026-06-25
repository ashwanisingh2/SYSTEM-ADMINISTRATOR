---
tags: [desktop-support, networking, osi-model, L1]
aliases: [osi-layers, network-reference]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #ccna
---

# OSI Model

---

## Concept Overview
- **What it is**: The Open Systems Interconnect (OSI) model is a conceptual 7-layer framework used to describe and standardize how network protocols transmit data across a physical network.
- **Why it matters for a support engineer**: Troubleshooting a network issue without a structured model leads to guesswork. The OSI model allows a support engineer to isolate connection failures systematically, from physical cabling issues to application-layer protocol bugs.
- **Where you encounter this in real job**: Diagnosing network dropouts (Layer 1 vs. Layer 3), resolving IP configuration errors (Layer 3), debugging web proxy blocks (Layer 7), and capturing network packets with Wireshark.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Troubleshoots Layer 1 (cabling, link lights) and Layer 2 (switch ports, MAC addresses, ARP).
  - **L2**: Troubleshoots Layer 3 (IP routing, subnetting, ICMP) and Layer 4 (TCP/UDP ports, firewalls).
  - **L3**: Configures Layer 4-7 services (VPN tunnels, web load balancers, DNS zones, and WAN security boundaries).

---

## Technical Deep Dive

### 1. The 7 Layers of the OSI Model
The OSI model divides network functions into seven logical layers, with each layer serving the layer above it:

| Layer Number | Layer Name | PDU (Data Unit) | Core Function | Examples / Protocols |
|---|---|---|---|---|
| **7** | **Application** | Data | User interaction & app integration | HTTP, HTTPS, DNS, DHCP, SMTP, FTP |
| **6** | **Presentation** | Data | Data formatting, encryption, & compression | SSL, TLS, JPEG, ASCII |
| **5** | **Session** | Data | Managing communication sessions | NetBIOS, RPC, Sockets |
| **4** | **Transport** | Segment / Datagram | End-to-end connection & flow control | TCP, UDP |
| **3** | **Network** | Packet | Logical addressing & path routing | IP (IPv4/IPv6), ICMP, IPSec, Routers |
| **2** | **Data Link** | Frame | Physical addressing & MAC table access | Ethernet, Wi-Fi, MAC addresses, Switches |
| **1** | **Physical** | Bits | Bit-level electrical & optical transmission | RJ45 cables, fiber optics, hubs, repeaters |

### 2. Encapsulation & Decapsulation Flow
Data travels down the OSI stack on the sender's side (Encapsulation) and up the stack on the receiver's side (Decapsulation):
1. **Application Data** is generated (e.g., an HTTP GET request).
2. **Layer 4** adds a TCP header, defining source and destination ports (**Segment**).
3. **Layer 3** adds an IP header, defining source and destination IP addresses (**Packet**).
4. **Layer 2** adds a MAC header and a Trailer (FCS) for error checking (**Frame**).
5. **Layer 1** converts the frame into electrical signals or light pulses for transmission (**Bits**).

```
[Sender]                                                   [Receiver]
Application (Data)    ----[ Encapsulation Down ]---->    Application (Data)
Transport (Segment)                                      Transport (Segment)
Network (Packet)                                         Network (Packet)
Data Link (Frame)     <---[ Layer 1 Transmission ]--->   Data Link (Frame)
```

---

## Commands & Syntax

### PowerShell
```powershell
# Verify Layer 3 connection and resolve Layer 7 DNS names
Test-NetConnection -ComputerName "google.com" -Port 80

# Retrieve Layer 2 MAC address and configuration details of local adapters
Get-NetAdapter | Select-Object Name, InterfaceDescription, MacAddress, Status
```

### CMD / Run Box
```cmd
REM Display current Layer 2 ARP cache (IP-to-MAC resolutions)
arp -a
REM Test network connection path and trace hops at Layer 3
tracert 8.8.8.8
```

### GUI Path
> Control Panel -> Network and Internet -> Network and Sharing Center -> Change Adapter Settings -> Double-click Adapter -> **Details**

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\
HKLM\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 10000 | Network connection disconnected (WLAN/Ethernet) | System Log |
| 10001 | Network connection established | System Log |
| 1014 | DNS name resolution timeout | System Log |

---

## Real-World Scenarios

### Scenario 1: User Reports "No Internet" (Physical Layer 1 Outage)
**User Complaint:** "I cannot open any websites or access company files. The browser shows a 'No Internet' error."
**Your First 3 Checks:**
1. Check the physical ethernet link light on the workstation's network interface card (NIC).
2. Check the system tray network status icon.
3. Verify if other devices in the same office area are experiencing the same issue.
**Diagnosis Steps:**
1. Check the NIC back panel. The green link status LED is completely unlit.
2. Unplug the ethernet cable from the wall jack and insert it into a known-working port.
3. Test using a known-working ethernet cable.
   - Once the cable is swapped, the link LED lights up green and flashes amber.
**Root Cause:** A broken plastic clip on the RJ45 connector caused the ethernet cable to slip out of the wall port, breaking the Layer 1 Physical connection.
**Fix:** Replace the damaged patch cable and verify the connection clip clicks locked.
**Prevention:** Avoid pulling cables from wall jacks by the cord; press the retention tab to release.
**Ticket Close Note:** "Replaced damaged Category 6 patch cable. Verified Layer 1 link LED active. Network connection restored. Closing ticket."

### Scenario 2: Active IP Configuration Errors (Layer 3 Address Failure)
**User Complaint:** "My internet shows connected, but I cannot access local servers, and I am getting a 'Self-Assigned IP Address' warning."
**Your First 3 Checks:**
1. Open command prompt and run `ipconfig`.
2. Check if the IP starts with `169.254.x.x` (APIPA).
3. Ping the loopback address (`127.0.0.1`) to check the TCP/IP stack status.
**Diagnosis Steps:**
1. Execute `ipconfig /all`.
   - The IP address is `169.254.12.98`, indicating an Automatic Private IP Addressing (APIPA) configuration.
2. Run a loopback check: `ping 127.0.0.1` -> Successful, confirming the local network stack is active.
3. Run: `ipconfig /renew` to query the DHCP server.
   - The command times out, indicating a Layer 3 configuration handshake failure.
**Root Cause:** The DHCP scope for the user's subnet was exhausted of available IP addresses, preventing the workstation from obtaining a lease and causing it to fall back to an APIPA address.
**Fix:** Add additional IP addresses to the DHCP scope or configure a shorter lease duration.
**Prevention:** Monitor DHCP scope utilization alerts and set alerts to trigger when scope reaches 85% capacity.
**Ticket Close Note:** "Diagnosed APIPA IP address configuration (`169.254.12.98`) caused by DHCP scope exhaustion. Expanded the DHCP pool range on the target subnet router. Renewed lease on client. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Configure firewall exceptions or routing rules at Layer 3/4 before verifying the physical link (Layer 1) and local switch port (Layer 2) connections.
> - This results in troubleshooting network configurations when the physical connection is broken, delaying resolution.
> - Always perform connection checks starting from Layer 1.

> [!warning] Common Trap
> - Confusing Layer 2 (MAC address) diagnostics with Layer 3 (IP address) routing.
> - Ping uses ICMP, which runs at Layer 3. If a ping fails, it does not mean the physical connection is broken; a local firewall or proxy at Layer 4 or Layer 7 may be blocking ICMP traffic.
> - Check Layer 2 status using `arp -a` to verify if the client can see the default gateway MAC address.

> [!tip] Senior Engineer Tip
> - When troubleshooting network issues, remember: "Layer 8 is the user." Many network issues are caused by incorrect credentials, misconfigured proxy profiles in browsers, or VPN clients not being toggled to connect.

> [!success] Verification Steps
> - Run: `Test-NetConnection -ComputerName "google.com" -Port 80` to verify connection across all layers.
> - Check that the output displays **TcpTestSucceeded : True**.

> [!question] Interview Alert
> - "Walk me through how you troubleshoot a network connection issue using the OSI model."
> - Answer: "I start at Layer 1 by verifying the link lights and cabling. Next, I check Layer 2 by checking the network adapter status and looking at the local ARP table. I then troubleshoot Layer 3 using ipconfig and pinging the default gateway. If routing is functional, I test Layer 4 ports using telnet or Test-NetConnection, and check Layer 7 by running DNS queries with nslookup."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Ping troubleshooting with firewall block | ICMP protocol blocked by target system | Use port checkers (like `tnc` or `telnet`) to verify port connectivity at Layer 4. |
| Neglecting DNS checks during website downtime | Assuming internet is down | Attempt to ping a public IP address (e.g., `8.8.8.8`) directly to isolate DNS resolution issues from network outages. |
| Blindly replacing NIC drivers | Ignoring basic hardware checks | Verify link lights and physical connections before modifying software configurations. |

---

## Lab Exercise

**Objective:** Isolate a network outage by tracing connections from Layer 1 through Layer 7.
**Time Required:** 20 minutes
**Environment Needed:** A Windows client machine and local gateway router.
**Pre-requisites:** Workstation connected to network.

**Steps:**
1. Layer 1 Check: Verify the network cable is connected and the link light on the port is active.
2. Layer 2 Check: Open PowerShell and run the command to verify the local switch port connection:
   ```powershell
   Get-NetNeighbor -IPAddress 192.168.10.1
   ```
   - Expected output: The IP address maps to a valid MAC address (e.g., `00-11-22-33-44-55`), confirming Layer 2 connectivity.
3. Layer 3 Check: Ping the default gateway to verify routing connection:
   ```cmd
   ping 192.168.10.1
   ```
   - Expected output: Ping succeeds with 0% packet loss.
4. Layer 4 Check: Test port connectivity to confirm TCP connection:
   ```powershell
   Test-NetConnection -ComputerName "192.168.10.1" -Port 80
   ```
   - Expected output: `TcpTestSucceeded : True`
5. Layer 7 Check: Query the DNS server for resolution confirmation:
   ```cmd
   nslookup company.com
   ```
   - Expected output: DNS resolves the correct IP record.

**Success Criteria:** The system successfully passes checks at all layers and resolves domain name records.
**Common Failures:** Failed DNS resolution indicates a DNS configuration error.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the difference between a MAC address and an IP address?**
A: A MAC (Media Access Control) address is a permanent, physical hardware address assigned to a NIC by the manufacturer, operating at Layer 2. An IP address is a logical network address assigned to a system by a DHCP server or administrator, operating at Layer 3 to route packets across subnets.

**Q: Which OSI layer does a network switch operate on, and what does it read?**
A: A standard switch operates at Layer 2 (Data Link layer). It reads the destination MAC address of incoming Ethernet frames to forward them directly to the target port.

### Intermediate (L2 Level)
**Q: What is the difference between a Layer 2 switch and a Layer 3 switch?**
A: A Layer 2 switch forwards frames based on physical MAC addresses and cannot route traffic between different IP subnets. A Layer 3 switch has routing capabilities, reading IP packet headers to route traffic between different VLANs and subnets directly in hardware.

### Advanced (L3/Senior Level)
**Q: A user cannot access a secure corporate web application. A ping to the server IP succeeds, but connection attempts in the browser time out. Explain your diagnostic workflow using the OSI model.**
A:
- **Situation**: A user was unable to load a secure web application, though basic IP routing (ICMP) was functional.
- **Task**: I needed to identify the failure point at the upper OSI layers (Layer 4 to Layer 7).
- **Action**: Since ping (Layer 3) was successful, I tested Layer 4 port connectivity using `Test-NetConnection -Port 443`. This failed, indicating the port was blocked or the service was offline. I checked the firewall configuration and observed that an active security policy was blocking HTTPS traffic from the user's subnet. I added an exception rule permitting HTTPS port 443 traffic to the server.
- **Result**: The connection test succeeded, and the user was able to access the web application.

### HR / Behavioral
**Q: Tell me about a time you had to deal with a complex network issue that impacted multiple departments.**
A: A core switch port failure caused several departments to lose connection to the local database. I worked systematically down the OSI model. I checked connection links, found MAC table address registration drops, and identified the failed switch module. I migrated the connections to a backup switch, restoring service within 30 minutes.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: A 7-layer conceptual framework standardizing network communication protocols.
> **Why**: Critical for isolating network connection failures systematically, from hardware to software.
> **How**: Diagnose connection issues step-by-step from Layer 1 through Layer 7.
> **Command**: `Test-NetConnection -ComputerName "target" -Port 80`
> **Interview Answer Starter**: "In my experience, network troubleshooting starts by checking the physical layer links and systematically tracing protocols up the OSI stack..."

**Key Numbers to Remember:**
- Layer 2 PDU: Frame / Layer 3 PDU: Packet / Layer 4 PDU: Segment
- Loopback IP: 127.0.0.1
- APIPA IP range: 169.254.0.0/16

**3 Things Interviewer Wants to Hear:**
1. Systematic bottom-up troubleshooting methodology
2. Understanding of protocol layer mappings (e.g., DNS, ICMP)
3. Using port tools to verify Layer 4 connectivity

---

## Related Notes
- [[01-Foundations/02-Networking/TCP-IP-and-Ports|TCP/IP and Ports]] — Details Layer 4 transport protocols.
- [[01-Foundations/02-Networking/DNS|DNS]] — Details Layer 7 name resolution.
- [[01-Foundations/02-Networking/DHCP|DHCP]] — Details Layer 7 automatic address configuration.

---

## Tags
#desktop-support #networking #osi-model #L1 #interview-topic #lab-complete #daily-use
