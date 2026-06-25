---
tags: [desktop-support, networking, troubleshooting, tools, L1]
aliases: [network-tools, network-diagnostics]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #ccna
---

# Troubleshooting Tools

---

## Concept Overview
- **What it is**: The suite of command-line utilities (ping, tracert, pathping, nslookup, netstat, arp) and graphical packet analysis tools (Wireshark) used to diagnose network connectivity, routing, and name resolution issues.
- **Why it matters for a support engineer**: A support engineer must know how to use these tools to isolate connection faults across the network path, proving where traffic is failing.
- **Where you encounter this in real job**: Diagnosing slow internet loading times, troubleshooting server access timeouts, checking if ports are listening, clearing duplicate IP address assignments, and isolating packet drops.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Runs basic ping and traceroute diagnostics, checks ipconfig details, and flushes client DNS caches.
  - **L2**: Performs detailed port checks (Test-NetConnection), reads routing tables (route print), and inspects local ARP tables.
  - **L3**: Runs packet captures (Wireshark) to analyze connection protocols, analyzes routing tables, and troubleshoots network bottlenecks.

---

## Technical Deep Dive

### 1. Command-Line Utilities Reference

#### Ping
Tests ICMP reachability to a target host.
- `-t`: Pings the target continuously until interrupted (`Ctrl + C`).
- `-a`: Resolves the IP address to its hostname.
- `-n <count>`: Sends a specific count of packets.
- `-l <size>`: Sets the packet payload size (default is 32 bytes).
- `-f`: Sets the "Don't Fragment" flag in the IP header (used to test path MTU limits).

#### Tracert / Traceroute
Identifies the route path taken by packets to a destination.
- Works by sending packets with increasing TTL values starting at 1.
- Each hop decrements the TTL. When it hits 0, the router drops the packet and returns an ICMP "Time Exceeded" packet, revealing its IP address.

#### Pathping
Combines features of `ping` and `tracert`.
- Runs a traceroute first, then sends 100 packets to every router along the path over 5 minutes.
- Calculates packet loss percentages per hop, allowing you to locate which specific router is dropping packets or experiencing latency.

#### Netstat
Displays active TCP connections, listening ports, and socket statistics.
- States:
  - `LISTENING`: The port is open and waiting for incoming connections.
  - `ESTABLISHED`: The active TCP handshake is complete, and data is flowing.
  - `TIME_WAIT`: The connection was closed, and the socket is waiting for lingering packets to clear.

#### ARP
Queries and modifies the Address Resolution Protocol cache.
- `arp -a`: Displays the current IP-to-MAC address mapping table.
- `arp -d *`: Clears the ARP cache (requires admin rights), forcing the system to query devices again.

#### Route Print
Displays the local system's active IP routing table, including default gateways and adapter interface metrics.

### 2. Wireshark Packet Analysis
Wireshark captures raw network frames on local adapters for protocol analysis:
- **Core Filters**:
  - `ip.addr == 192.168.10.10` — Show only packets matching this IP.
  - `tcp.port == 443` — Show only HTTPS traffic.
  - `dns` — Show only DNS query/response packets.
  - `http.request` — Show outgoing HTTP GET/POST queries.

---

## Commands & Syntax

### PowerShell
```powershell
# Perform a port scan and trace the path to a destination
Test-NetConnection -ComputerName "google.com" -TraceRoute

# Get active TCP connections matching port 80
Get-NetTCPConnection -LocalPort 80 | Select-Object LocalAddress, LocalPort, State
```

### CMD / Run Box
```cmd
REM Clear the ARP cache to resolve duplicate IP address anomalies
arp -d *
REM Run a pathping diagnostics check to a public target
pathping -n 8.8.8.8
```

### GUI Path
> Start -> search `Resource Monitor` -> Network Tab -> view network connection details.

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\
HKLM\SYSTEM\CurrentControlSet\Services\NlaSvc\Parameters\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 1014 | DNS name resolution timeout | System Log |
| 4199 | Duplicate IP address conflict warning | System Log |
| 10000 | Network interface connection state disconnected | System Log |

---

## Real-World Scenarios

### Scenario 1: Intermittent Database Disconnects (Pathping Analysis)
**User Complaint:** "My local application disconnects from the database server randomly during the day. It reconnects immediately, but I lose my current unsaved work."
**Your First 3 Checks:**
1. Ping the database server to check for general connection drops.
2. Check for Event Viewer System network disconnection logs (Event ID 10000).
3. Run a pathping analysis to identify the hop experiencing packet drops.
**Diagnosis Steps:**
1. Ping the database server continuously: `ping -t 192.168.20.25`
   - Observe the output. Pings are successful with occasional latency spikes, but no packet drops are immediately visible.
2. Run a pathping test to trace the entire path:
   `pathping -n 192.168.20.25`
3. Analyze the output once the calculations complete (approx. 5 minutes):
   - Hop 3 (`10.0.1.254`): `15% packet loss`
   - Hop 4 (`192.168.20.25`): `0% packet loss`
   - This indicates that Hop 3 (an intermediate network switch) is dropping packets under load.
**Root Cause:** A failing interface port on the intermediate switch at Hop 3 was dropping packets under network load, causing database connection drops.
**Fix:** Replace the patch cable on the switch interface or migrate the connection to an alternative port.
**Prevention:** Configure automated threshold alerts in SNMP tools to monitor switch interface error rates.
**Ticket Close Note:** "Diagnosed switch port failure at Hop 3 (`10.0.1.254`) using pathping. Migrated connection to an alternative port. verified 0% packet loss. Closing ticket."

### Scenario 2: User Cannot Connect to Corporate Web Server (Netstat Verification)
**User Complaint:** "I configured our custom web server application on a test machine, but when I try to connect from my browser, it fails to load."
**Your First 3 Checks:**
1. Check if the web service process is active.
2. Check if the port is listening locally using `netstat`.
3. Check if the Windows Defender Firewall blocks the port.
**Diagnosis Steps:**
1. Verify the process is running in Task Manager.
2. Open a command prompt as Administrator and check for port 80/443 activity:
   `netstat -ano | findstr :80`
   - No output is returned, indicating the web application has not bound the port.
3. Check the application configuration file:
   - The config file has the port binding set to `8080` instead of `80`.
**Root Cause:** The application was configured to listen on custom port 8080, causing connection attempts on default port 80 to fail.
**Fix:** Connect to the application using the custom port (`http://server:8080`) or reconfigure the application to use default port 80.
**Prevention:** Standardize port configuration guidelines for all local testing applications.
**Ticket Close Note:** "Identified that the application was listening on port 8080 using netstat. Reconfigured port to default port 80. Verified connection. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Run continuous continuous ping commands (`ping -t`) with massive packet sizes (e.g., `ping -t -l 65500`) against production servers or networking devices.
> - This mimics a Denial of Service (DoS) attack (Ping of Death), consuming device CPU resources and potentially crashing legacy network interfaces.
> - Use standard packet sizes (32 bytes) for diagnostics.

> [!warning] Common Trap
> - Assuming that a successful ping test means the target service port is open and accessible.
> - Ping uses the ICMP protocol, which does not check TCP/UDP ports. A machine can reply to ping checks while its web server service (port 80/443) is stopped.
> - Always run port checks (like `Test-NetConnection`) to verify service status.

> [!tip] Senior Engineer Tip
> - If you suspect a DNS server is returning stale or incorrect records, bypass the local client resolver cache and query the DNS server directly using:
>   `nslookup domain.com [DNS-Server-IP]`

> [!success] Verification Steps
> - Run: `Test-NetConnection -ComputerName "localhost" -Port 80` to verify service reachability.
> - Confirm the connection test returns `True`.

> [!question] Interview Alert
> - "Explain the difference between traceroute and pathping."
> - Answer: "Traceroute identifies the routing hops to a destination by sending packets with increasing TTL values. Pathping combines traceroute with ping, sending 100 packets to every router along the path over a 5-minute period to calculate exact packet loss percentages per hop, allowing you to isolate network bottlenecks."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Assuming network down on ping fail | ICMP blocked by firewall | Use port verify tools to check service reachability. |
| Troubleshooting without clearing cache | Querying stale DNS/ARP memory | Flush DNS cache (`ipconfig /flushdns`) and clear ARP cache (`arp -d *`) before diagnostics. |
| Running Wireshark without capture filters | Packet capture buffer overflow | Configure capture filters (like `host IP`) to record only relevant diagnostic packets. |

---

## Lab Exercise

**Objective:** Diagnose a network connection dropout, verify routing paths using trace tools, and capture network handshake packets using Wireshark.
**Time Required:** 30 minutes
**Environment Needed:** A Windows 11 VM, a remote host, and Wireshark installed.
**Pre-requisites:** Administrative access.

**Steps:**
1. Launch Wireshark. Select your active network interface card (NIC).
2. Configure Capture Filter: Enter `icmp` in the capture filter bar -> Click **Start**.
3. Open a command prompt and trigger network queries:
   ```cmd
   ping -n 4 8.8.8.8
   tracert 8.8.8.8
   ```
4. Observe Wireshark. Stop the capture once the commands complete.
   - Expected output: Wireshark displays the ICMP Echo Request and Echo Reply packets, along with the ICMP Time Exceeded packets returned during the traceroute.
5. Identify the IP address of the first hop from the Wireshark packets.
6. Verify local network statistics and check for active listening ports:
   ```cmd
   netstat -ano | findstr :443
   ```
7. Verification: Run the verification command.
   ```powershell
   Test-NetConnection -ComputerName "8.8.8.8" -Port 53
   ```
   - Expected output: `TcpTestSucceeded : True`

**Success Criteria:** The connection diagnostics complete successfully, Wireshark records the ICMP handshake packets, and the port check returns `True`.
**Common Failures:** Failed Wireshark capture if the selected NIC interface is inactive.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: How does the ping command work?**
A: The ping command uses the ICMP (Internet Control Message Protocol) protocol. It sends an ICMP Echo Request packet to the target IP address. If the target is online and firewall rules permit, it replies with an ICMP Echo Reply packet, confirming connection.

**Q: What is the purpose of the ipconfig command and its flags?**
A: `ipconfig` displays the network configuration details of local adapters. Key flags include `/all` (shows detailed configuration including MAC and DNS), `/release` (drops the current DHCP IP lease), and `/renew` (requests a new IP lease from the DHCP server).

### Intermediate (L2 Level)
**Q: How does the tracert command identify the path route to a destination?**
A: Tracert sends packets with the Time-To-Live (TTL) field starting at 1. The first router decrements the TTL to 0, drops the packet, and returns an ICMP "Time Exceeded" message, revealing its IP. Tracert then increases the TTL by 1 for subsequent packets, mapping each router along the path until it reaches the destination.

### Advanced (L3/Senior Level)
**Q: A critical client application experiences connection timeouts during database transactions. Ping checks show 0% packet loss. Explain your diagnostic workflow using Wireshark.**
A:
- **Situation**: A client application timed out during database transactions despite successful ping checks.
- **Task**: I needed to identify if the timeouts were caused by database response delays, packet drops, or TCP window limits.
- **Action**: I launched Wireshark on the client machine, set a capture filter targeting the database IP (`ip.addr == 192.168.20.25`), and reproduced the timeout error. I analyzed the capture logs. I observed several **TCP Retransmissions** and **Duplicate ACKs** occurring during the transaction, followed by a TCP connection reset (RST) packet sent by the database server.
- **Result**: The packet analysis showed that the database server was dropping connection packets due to network buffer limits under load. Replacing the network cable resolved the packet drops.

### HR / Behavioral
**Q: Tell me about a time you had to resolve an network issue under pressure. How did you utilize troubleshooting tools to isolate the fault?**
A: During a company presentation, the executive's connection dropped. I ran a quick trace, identified a DNS timeout error, reconfigured their DNS server to the backup controller, and restored connection in under 5 minutes.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The command-line utilities and software tools used to diagnose network connectivity, routing, and name resolution.
> **Why**: Critical for systematically isolating network failures and resolving connection drops.
> **How**: Use ping to check reachability, tracert to check routing paths, netstat to check ports, and Wireshark to analyze packets.
> **Command**: `pathping -n 8.8.8.8`
> **Interview Answer Starter**: "In my experience, network diagnostics start by verifying connection parameters using ping, and tracing paths using tracert to isolate failures..."

**Key Numbers to Remember:**
- Default ping payload size: 32 bytes (Windows)
- Max TTL value: 255
- Wireshark DNS query filter: `dns`

**3 Things Interviewer Wants to Hear:**
1. Systematic bottom-up diagnostic methodology (Layer 1 through Layer 7)
2. How traceroute uses TTL to identify routing hops
3. Using port verify tools to identify firewall blocks

---

## Related Notes
- [[01-Foundations/02-Networking/OSI-Model|OSI Model]] — Details the protocol layers under check.
- [[01-Foundations/02-Networking/TCP-IP-and-Ports|TCP/IP and Ports]] — Details transport port mappings under check.
- [[01-Foundations/02-Networking/Subnetting|Subnetting]] — Details IP subnet design check.

---

## Tags
#desktop-support #networking #troubleshooting #tools #L1 #interview-topic #lab-complete #daily-use
