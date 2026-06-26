---
tags: [sysadmin, networking, fundamentals]
difficulty: Beginner
lab-required: Yes
read-time: 12 mins
---

# N-01: Networking Fundamentals

> [!abstract] Overview
> This note introduces fundamental networking concepts, topology designs, protocols, and architectural models. It details how data flows across physical media and provides a framework for troubleshooting using logical stack analysis.

---
## Concept
Think of a network like an office courier system. 
- A **PAN** is passing a pen to the colleague sitting right next to you.
- A **LAN** is a department building where everyone uses the internal mailboxes to send letters to other floors.
- A **MAN** is a delivery truck routing mail between different company offices located within the same city.
- A **WAN** is sending an international package through customs and multiple shipping hubs (airports, sorting centers) to reach another country.

*Seedha simple mein: Jab do ya do se zyada computers aapas mein data share karne ke liye judte hain, toh use network kehte hain. Yeh connectivity small scope (ek room) se lekar worldwide (internet) tak ho sakti hai.*

---
## Technical Deep Dive

### 1. Network Scope Classifications
- **PAN (Personal Area Network):** Centered around a single person. Range is typically under 10 meters (e.g., Bluetooth connection between phone and headphones).
- **LAN (Local Area Network):** Connects hosts within a single localized geographic area, such as a home, school, or office building. Owned and managed by a single organization (e.g., Wi-Fi or Ethernet switches).
- **MAN (Metropolitan Area Network):** Covers a larger area than a LAN but smaller than a WAN, such as a city or university campus. Often owned by a consortium or a service provider (e.g., municipal fiber loops).
- **WAN (Wide Area Network):** Spans large geographic regions, countries, or continents. Connects multiple LANs. Uses public transit systems, telecom lines, and satellites (e.g., the Internet).

### 2. Network Topologies (Physical Layouts)
Topologies define how devices are connected.

#### Star Topology
Every device connects to a central hub/switch via dedicated cables.
```
      [Host A]
         |
[Host B]- -[Switch]- -[Host C]
         |
      [Host D]
```
- **Pros:** Easy to install and scale. If one host cable breaks, only that host goes offline.
- **Cons:** Central switch is a Single Point of Failure (SPOF). If the switch fails, the entire network goes down.

#### Mesh Topology (Full Mesh)
Every device is connected directly to every other device.
```
  [Host A]-------[Host B]
     |   \       /   |
     |    \     /    |
     |     \   /     |
     |      \ /      |
  [Host D]-------[Host C]
```
- **Formula for links:** $\text{Links} = \frac{N(N-1)}{2}$ where $N$ is the number of nodes.
- **Pros:** Highly redundant and fault tolerant.
- **Cons:** Extremely expensive and complex to cable physically.

#### Bus Topology
All devices connect to a single central cable (the bus) with terminators at each end.
```
---[Terminator]=======[Bus Cable]=======[Terminator]---
                |           |          |
            [Host A]    [Host B]   [Host C]
```
- **Pros:** Cheap, uses minimal cabling.
- **Cons:** If the main bus cable breaks, the entire network splits and goes offline due to signal reflection. High collisions.

#### Ring Topology
Each device connects to two neighbors, forming a closed loop.
```
   [Host A]-----[Host B]
      |            |
   [Host D]-----[Host C]
```
- **Pros:** No collisions; data flows in one direction (using tokens).
- **Cons:** If one node or link breaks, the entire ring fails unless a dual-ring (FDDI) is used.

### 3. OSI Model (Open Systems Interconnection)
Advanced content only — basics in [[OSI model intro]]

The 7-layer OSI model serves as a diagnostic framework for sysadmins to isolate issues systematically.

```
+------------------------------------+--------------------------+
| OSI Layer                          | Troubleshooting Focus    |
+------------------------------------+--------------------------+
| L7 - Application (HTTP, DNS)       | App logs, API status     |
| L6 - Presentation (SSL/TLS, JSON)  | Decryption, encryption   |
| L5 - Session (RPC, NetBIOS)        | Connection sync logs     |
| L4 - Transport (TCP, UDP)          | Port blocks, TCP reset   |
| L3 - Network (IP, ICMP)            | Routing, ping, ip routing|
| L2 - Data Link (MAC, ARP, VLANs)   | Switch MAC table, frame  |
| L1 - Physical (Cables, Hubs)       | Link lights, cabling     |
+------------------------------------+--------------------------+
```

#### Troubleshooting via OSI: Bottom-Up vs. Top-Down
- **Bottom-Up:** Start at L1. Check link lights, swap cables, verify physical port locks. Proceed to L2 (ARP table, VLAN matching), L3 (Ping IP), and upward. Best for physical link outages.
- **Top-Down:** Start at L7. Check if application opens, query DNS, review application logs. Best for software errors on an otherwise functional connection.

### 4. TCP/IP Model vs. OSI Mapping
The TCP/IP model is the practical implementation used on the Internet.

| OSI Layer | TCP/IP Layer | Associated Protocols | Data Unit (PDU) |
|---|---|---|---|
| 7. Application | | | |
| 6. Presentation | **Application** | HTTP, HTTPS, DNS, SMTP, DHCP | Data / Payload |
| 5. Session | | | |
| 4. Transport | **Transport** | TCP, UDP | Segment (TCP) / Datagram (UDP) |
| 3. Network | **Internet** | IP, ICMP, ARP, OSPF | Packet |
| 2. Data Link | **Network Access** | Ethernet, Wi-Fi (802.11), PPP | Frame |
| 1. Physical | | | Bits |

### 5. Data Encapsulation and Decapsulation
As data travels down the stack, headers are prepended (Encapsulation). As it travels up the stack on the receiving end, headers are stripped (Decapsulation).

```
[Application Data]                                           (L5-7: Data)
      |
      v
[TCP Header] [Application Data]                              (L4: Segment)
      |
      v
[IP Header] [TCP Header] [Application Data]                  (L3: Packet)
      |
      v
[Ethernet Header] [IP Header] [TCP Header] [App Data] [Trailer] (L2: Frame)
      |
      v
01001001 01010100 01001111 ...                               (L1: Bits)
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A workstation with drawing software (or draw.io / Pen and Paper).

### Step 1: Mapping the Home/Office Network Topology
1. Identify all network devices in your physical space:
   - ISP Entry Point (Modem or Fiber ONT).
   - Core Wireless Router.
   - Smart devices, laptops, phones, servers.
2. Determine connection types (Ethernet vs. Wi-Fi).

### Step 2: Draw the Diagram
Create an ASCII or visual block representation of the network topology:

```
[ISP Fiber Line]
       |
  [Fiber ONT]
       | (Ethernet)
[Wireless Router] <-----------------------+
   | (LAN Port 1)                         | (2.4/5GHz Wi-Fi)
[Unmanaged Switch]                        +--- [Smart TV]
   |          |                           +--- [Mobile Phone]
[My Workstation] [Home NAS Server]        +--- [User Laptop]
```

### Step 3: Identify Topologies
1. Classify the diagram: This represents a **Star Topology** branching from the central Wireless Router/Switch.
2. Label the WAN boundary (connection between ONT and ISP) and LAN boundary (everything behind the internal interface of the Wireless Router).

---
## Commands Reference
Advanced content only — basics in [[Basic networking commands]]

Verify local adapter configurations and diagnostic paths:

```bash
# Windows
ipconfig /all                     # Show complete network adapter configuration (IP, MAC, DNS)
ping -t 8.8.8.8                   # Send continuous ping to check L3 network latency
tracert -d 1.1.1.1                # Trace hops to destination without resolving DNS

# Linux
ip a                              # List all network interfaces and IP addresses
ss -tulpn                         # List listening ports and associated processes (L4)
tracepath 8.8.8.8                 # Trace network path and estimate MTU values
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** A user cannot connect to a web server. They get a "Server Not Found" error in the browser.
- **Root Cause:** A Layer 3 (DNS lookup failure) or Layer 7 (web service down) issue.
- **Fix:** Use the OSI bottom-up diagnostic protocol.
  1. **L1/L2:** Run `ping 127.0.0.1` (loopback) to verify the TCP/IP stack is working, then verify the physical link lights.
  2. **L3:** Ping the local gateway (e.g., `192.168.1.1`) to check local path connectivity.
  3. **L3/DNS:** Ping a public IP (e.g., `8.8.8.8`). If it succeeds, ping `google.com`. If it fails, the DNS resolver configuration is incorrect. Fix by setting DNS to `8.8.8.8`.
  4. **L4/L7:** If `google.com` pings but the web page does not load, test connection to port 443 using `Test-NetConnection google.com -Port 443`.

**Scenario 2:**
- **Problem:** A switch replacement causes a segment of a Ring topology network to fail, partitioning communication for half the nodes.
- **Root Cause:** The new switch does not support STP (Spanning Tree Protocol) or it was disabled, creating a network loop (broadcast storm) that saturates the physical link (L1/L2 overload).
- **Fix:**
  1. Disconnect one redundant link of the ring immediately to break the physical loop.
  2. Access the new switch configuration interface.
  3. Enable **RSTP** (Rapid Spanning Tree Protocol) on all switches in the loop.
  4. Reconnect the link and verify that the switch correctly blocks one port to maintain a loop-free logical tree while preserving physical redundancy.

---
## Common Mistakes
> [!warning] Avoid These
> **Assuming software issues before checking cables:** Troubleshooting a Windows operating system driver configuration or IP address assignment for an hour without looking at the back of the computer to see that the Ethernet cable link light is completely off.
> **Correct approach:** Always start with Layer 1 (Physical check) first when diagnosing connection failures.

---
## Pro Tips
> [!tip] Field Experience
> When mapping network topologies, always designate logical networks (VLANs, subnets) on the diagram alongside the physical cable paths. A physical star topology can behave as multiple distinct networks logically, and troubleshooting routing issues requires knowing the difference between the physical wire and the logical path.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | LAN vs WAN | LAN is a private network under single local control; WAN connects LANs across large geographical zones. |
| 2 | Star Topology | Standard layout where all hosts connect to a central hub; failure of one cable only drops that host. |
| 3 | OSI L2 vs L3 | L2 operates on MAC addresses (Switches/Frames); L3 operates on IP addresses (Routers/Packets). |
| 4 | Encapsulation | The process of adding headers (TCP, IP, MAC) to payload data as it moves down the network stack. |
| 5 | Network Loop | A loop in physical pathing that causes broadcast storms unless prevented by Spanning Tree Protocol. |

---
## Interview Q&A

**Q1: What is the difference between TCP and UDP, and when would you use each?**
A: **TCP (Transmission Control Protocol)** is a connection-oriented, reliable protocol operating at Layer 4. It establishes a session using a 3-way handshake, ensures data ordering, performs flow control, and retransmits lost segments. It is used when data integrity is critical (e.g., HTTP, SSH, FTP). **UDP (User Datagram Protocol)** is a connectionless, unreliable protocol. It sends packets ("datagrams") without establishing a connection or verifying delivery, resulting in low overhead and high speed. It is used when speed is prioritized over reliability (e.g., VoIP, DNS queries, live video streaming).

**Q2: A client workstation can ping the IP address of a server (192.168.1.50) but cannot access the shared folder using the server's DNS name (\\fileserver). Walk through how you would resolve this.**
A: 
- **Situation:** L3 IP connectivity is verified, but L7 host name resolution fails.
- **Task:** Resolve the DNS resolution failure to restore access via server name.
- **Action:** I will first execute `nslookup fileserver` to check if the DNS server can resolve the hostname to the correct IP. If it returns an incorrect IP or timeouts, I will check the workstation's DNS server configuration (`ipconfig /all`). If it uses an external DNS (like `8.8.8.8`) instead of the domain controller DNS, internal name resolution will fail.
- **Result:** Updating the workstation's DNS settings to target the Active Directory DNS server restores access to the shared folder.

**Q3: Describe the TCP 3-Way Handshake process.**
A: The 3-way handshake establishes a reliable TCP connection. 
1. **SYN (Synchronize):** The client sends a TCP segment to the server with a random sequence number ($x$) and the SYN flag set.
2. **SYN-ACK:** The server receives the segment, records sequence number $x$, and responds with a segment where both SYN and ACK flags are set. Its acknowledgment number is set to $x+1$, and it generates its own random sequence number ($y$).
3. **ACK:** The client receives the response and sends a final segment with the ACK flag set. Its sequence number is set to $x+1$, and the acknowledgment number is set to $y+1$. The connection state changes to **Established**.

---
## Related Notes
- [[01-Foundations/02-Networking/N-02 Network Devices Deep Dive|N-02 Network Devices Deep Dive]] — Detailed operation of Hubs, Switches, and Routers.
- [[01-Foundations/02-Networking/N-03 Ethernet and MAC Address|N-03 Ethernet and MAC Address]] — Hardware protocols, Cat cabling, and ARP operations.
- [[01-Foundations/02-Networking/N-10 Network Troubleshooting Complete|N-10 Network Troubleshooting Complete]] — Detailed OSI diagnostic commands and tools.


---

### Enterprise Networking & Wireless Analysis

#### 1. Wireshark Packet Capture Filtering
Use these filters during packet capture analysis to isolate issues quickly:
- **HTTP POST Request Error isolation**: `http.request.method == "POST" && http.response.code >= 400`
- **Isolate TCP Retransmissions (Packet Loss)**: `tcp.analysis.retransmission || tcp.analysis.duplicate_ack`
- **DNS Server response failures check**: `dns.flags.response == 1 && dns.flags.rcode != 0`
- **Isolate host traffic (excluding noise)**: `ip.addr == 192.168.1.50 && !arp && !dns`

#### 2. Wireless Troubleshooting (Enterprise Wi-Fi)
- **802.1X EAP Authentication Failed**: Verify certificate validation settings on RADIUS server (NPS); check client identity store configuration.
- **Roaming failure (sticky client)**: Adjust Minimum RSSI settings on wireless access points (APs) or check that both APs broadcast identical SSIDs with overlapping coverage ranges (15-20% overlap).
- **RF Interference**: Run a channel survey; move corporate APs from saturated 2.4GHz bands to clean 5GHz/6GHz channels using 20MHz or 40MHz channel widths.

#### 3. Software-Defined WAN (SD-WAN) Basics
- **What it is**: SD-WAN decouples the network control plane from the physical hardware forwarding plane. It manages WAN links (MPLS, Broadband, LTE) dynamically.
- **Why it matters**: It automatically routes business-critical traffic (like VoIP) over the lowest latency links while routing general web traffic over cheap broadband, using real-time link quality metrics (jitter, packet loss, latency).
