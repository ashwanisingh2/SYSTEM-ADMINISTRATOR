---
tags: [networking, hardware, devices]
aliases: [network-devices]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#intermediate` `#ccna`

# N-02: Network Devices Deep Dive

> [!abstract] Overview
> Yeh note standard network hardware jaise Hubs, Switches, Routers, Firewalls, WAPs, aur NICs ke packet-forwarding logic aur configuration roles ko cover karta hai. Ek support engineer ke liye yeh jaanna zaroori hai kyunki network design aur diagnostics ke bina kisi bhi connectivity issue ko troubleshoot karna namumkin hai.

---
## 🧠 Concept Overview

- **What it is** — Network devices are hardware components that connect computers, printers, and other devices to a network, allowing them to communicate and share data.
- **Why it matters** — Every IT infrastructure relies on these devices to route traffic securely and efficiently. Understanding them is crucial for diagnosing network slowdowns, outages, and connectivity drops.
- **Where you see this** — In corporate offices connecting employee PCs, in data centers routing server traffic, and in enterprise Wi-Fi deployments.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check physical connections, verify link lights, basic ping tests, and confirm IP settings. |
| **L2** | Configure switch ports, VLANs, check MAC address tables, analyze port statistics for errors, and escalate complex routing issues. |
| **L3** | Architecture, design, core routing protocols (OSPF, BGP), firewall policy design, and enterprise SD-WAN implementations. |

> [!tip] Seedha Simple Mein
> *Network devices hardware components hain jo data ko sahi direction mein bhejte hain. Hub sabko data bhejta hai, Switch specific computer ko bhejta hai, Router pure network ke beech traffic route karta hai, aur Firewall security check lagata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Sorting Office** is like **Network Devices** because...
> 
> - A **Hub** is a blind worker who receives a letter, makes copies of it, and runs around screaming the message to everyone in the building.
> - A **Switch** is a smart clerk who checks the name on the envelope, looks up their desk location in a registry (MAC Table), and walks the letter directly to that specific desk.
> - A **Router** is the dispatcher who looks at the city and zip code (IP Address) and decides which delivery truck or airplane path is the fastest route to get the letter to a completely different office building.
> - A **Firewall** is the security guard standing at the main entrance, checking credentials and only allowing approved visitors.

---
## 🔬 Technical Deep Dive

### 1. Hub (Layer 1 - Physical)

> [!info] Key Concept
> A passive multi-port repeater. It has no intelligence. When a frame enters one port, it electrical-copies the signal and floods it out of all other ports.

**Why deprecated:** 
- **Shared Bandwidth:** All ports share the same bandwidth.
- **Single Collision Domain:** Devices must use CSMA/CD to detect electrical collisions. Only one device can transmit at a time.
- **Security Risk:** Every device connected to the hub can inspect all traffic (using promiscuous mode).

### 2. Bridge & Switch (Layer 2 - Data Link)

> [!info] Key Concept
> A **Switch** is a hardware-based multi-port bridge that uses ASICs to forward frames in microsecond speeds based on MAC addresses.

- **CAM (Content Addressable Memory) Table:** The switch learns the source MAC address of incoming frames and maps them to the physical port number.
- **Forwarding Methods:**
  - **Store-and-Forward:** Receives the entire frame, calculates CRC checksum. Safe but slowest.
  - **Cut-Through:** Reads only the destination MAC (first 14 bytes) and forwards. Lowest latency but forwards corrupted frames.
  - **Fragment-Free:** Reads the first 64 bytes to ensure no collision occurred.

> [!danger] Common Mistake
> **Assuming a switch routes traffic between subnets:** Connecting two networks with different subnets to a Layer 2 switch and wondering why they cannot communicate. L2 switches only forward traffic based on MAC addresses within the same subnet. Use a Router or a Layer 3 Switch to perform routing between different IP subnets.

### 3. Router & SD-WAN (Layer 3 - Network)

> [!info] Key Concept
> Connects different networks (broadcast domains) and reads Layer 3 IP headers to make forwarding decisions.

- **Routing Table:** Matches destination IP networks to outbound interfaces or next-hop IP addresses.
- **ASIC vs. CPU:** Route processing (determining path) is handled in software (control plane); packet forwarding is offloaded to ASICs (data plane).
- **SD-WAN Basics:** Decouples network control plane from physical hardware. Automatically routes business-critical traffic over the lowest latency links.

### 4. Firewall (Layer 3 - 7)

> [!info] Key Concept
> A security device that inspects and filters traffic based on predefined rules.

- **Stateless Firewall:** Inspects individual packets based on static rules (IP, Port, Protocol).
- **Stateful Firewall:** Tracks the state of active TCP connections, allowing return traffic automatically.
- **Next-Generation (NGFW):** Inspects actual application layer payload (deep packet inspection). Detects malware and filters based on user identities.

### 5. Wireless & Access Points

> [!info] Key Concept
> A Wireless Access Point (WAP) acts as a bridge between wireless networks (802.11) and wired networks (802.3).

- **SSID:** The logical name of the wireless network.
- **Channels:** Frequency segments selected to prevent overlapping signal interference.
- **Enterprise Wi-Fi Troubleshooting:**
  - **802.1X EAP Failure:** Verify certificate validation on RADIUS server.
  - **Roaming Failure:** Adjust Minimum RSSI settings on APs or ensure 15-20% overlap.
  - **RF Interference:** Run a channel survey; move to clean 5GHz/6GHz channels using 20/40MHz widths.

### 6. Network Interface Card (NIC)

> [!info] Key Concept
> The physical controller connecting a device to network media, with a burned-in MAC address.

- **Duplex Modes:** Half-Duplex (transmit or receive, not both simultaneously), Full-Duplex (transmit and receive simultaneously).
- **Auto-Negotiation:** NIC and switch port automatically select highest matching speed and duplex mode.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Cisco Packet Tracer software installed on a workstation.

### Step 1: Set up the Topology

```bash
# Devices needed in Cisco Packet Tracer:
# 1x Generic Hub, 1x Catalyst 2960 Switch, 1x 1941 Router, 3x PCs
```
1. Connect PC0, PC1, and PC2 to ports 1, 2, and 3 on the **Hub**. Label this subnet `192.168.1.0/24`.
2. Run another link from the Hub to the **Switch**.
3. Connect the Switch to the Gig0/0 port on the **Router**.

### Step 2: Configure IP Addresses

1. Click on **PC0** -> Desktop -> IP Configuration. Assign IP `192.168.1.10`, Mask `255.255.255.0`, Gateway `192.168.1.1`.
2. Click on **PC1** -> Desktop -> IP Configuration. Assign IP `192.168.1.11`, Mask `255.255.255.0`, Gateway `192.168.1.1`.

### Step 3: Observe Packet Flow in Simulation Mode

1. Switch to **Simulation Mode** (stopwatch icon).
2. Click **Add Simple PDU** (envelope icon). Click **PC0** (source) then **PC1** (destination).
3. Click **Play**.
4. **Observe the Hub:** It copies the packet and broadcasts to PC2 and Switch. PC2 discards it.
5. **Observe the Switch:** Send a packet from PC1 to PC0. Switch checks MAC table, forwards *only* down the port to the Hub.

> [!success] Expected Output
> ```
> Packets successfully routed to specific destination via Switch, while Hub broadcasts to all ports.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `show mac address-table` | Display the switch's learned MAC-to-Port mappings | `show mac address-table` |
| `clear mac address-table dynamic` | Flush the dynamic MAC table to force relearning | `clear mac address-table dynamic` |
| `show interfaces status` | Check port speeds, duplex configurations, and VLANs | `show interfaces status` |
| `Get-NetAdapter` | Query details about Windows physical NICs | `Get-NetAdapter \| Select-Object Name` |
| `netsh interface show interface` | Show state of local Windows interfaces | `netsh interface show interface` |
| `http.request.method == "POST" && http.response.code >= 400` | Wireshark: HTTP POST Request Error isolation | `http.request.method == "POST"` |
| `tcp.analysis.retransmission \|\| tcp.analysis.duplicate_ack` | Wireshark: Isolate TCP Retransmissions (Packet Loss) | `tcp.analysis.retransmission` |
| `dns.flags.response == 1 && dns.flags.rcode != 0` | Wireshark: DNS Server response failures check | `dns.flags.response == 1` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Extremely slow speeds, frequent disconnects, high ping packet loss. | Duplex mismatch (one end Full-Duplex, other Half-Duplex). | Check switch port for CRC errors/collisions. Set both ends to Auto-Negotiation or force 1000Mbps/Full-Duplex. |
| Server cannot communicate with a DB on different VLAN. Ping returns "Request timed out". | Layer 3 routing configuration failure or firewall blocking traffic. | Run `tracert`. Check router with `show ip route`. Verify stateful firewall logs to ensure port isn't blocked. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Inter-VLAN Routing Failure

> [!example] Ticket
> "Database team reports that web servers in VLAN 10 cannot reach the SQL server in VLAN 20."

**L1 Response:** Ping the SQL server from the web server. Check physical links and verify basic IP configuration on both servers.
**Escalation Trigger:** Pass to L2 if ping fails and IP configuration is correct.
**L2 Resolution:** Check the router or Layer 3 switch for correct inter-VLAN routing setup. Verify `show ip route` for both subnets. Check firewall logs to ensure port 1433 isn't blocked between zones.

### 🎫 Scenario 2: Wi-Fi Dropouts

> [!example] Ticket
> "Users in the high-rise office complain of unstable Wi-Fi connections and constant dropouts."

**L1 Response:** Verify the users' laptop Wi-Fi settings and check if the issue is isolated to one area or affecting everyone.
**Escalation Trigger:** Pass to L2 if multiple users in the same area report issues.
**L2 Resolution:** Use a Wi-Fi analyzer tool. Identify co-channel interference on 2.4 GHz. Reconfigure WAPs to use non-overlapping channels (1, 6, 11) or migrate users to the 5 GHz band.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between a Layer 2 Switch and a Layer 3 Switch?
> **Answer:** A **Layer 2 Switch** forwards traffic based purely on physical MAC addresses. A **Layer 3 Switch** combines the speed of switch hardware (ASICs) with the routing capabilities of a router, performing Inter-VLAN routing at wire speed.

==**Exam Tip:** Layer 3 switches use ASICs for fast routing, whereas traditional routers use software/CPU.==

> [!question] Q2: Explain how a switch builds its MAC address table (CAM table).
> **Answer:** It builds its CAM table dynamically by inspecting the **Source MAC Address** of incoming frames. If the Destination MAC is unknown, it performs "unknown unicast flooding." Once the destination responds, the switch records it.

==**Exam Tip:** Switches learn from the SOURCE MAC address, not the destination!==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-01 Networking Fundamentals|N-01 Networking Fundamentals]] — Overview of network topologies and the OSI model.
- [[01-Foundations/02-Networking/N-03 Ethernet and MAC Address|N-03 Ethernet and MAC Address]] — Details on frames, MAC formats, and ARP.
- [[01-Foundations/02-Networking/N-06 Switching — VLANs and Trunking|N-06 Switching — VLANs and Trunking]] — Dynamic switch configurations and VLAN design.
