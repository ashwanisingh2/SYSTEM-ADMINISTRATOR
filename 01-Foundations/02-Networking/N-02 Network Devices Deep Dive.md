---
tags: [sysadmin, networking, hardware, devices]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# N-02: Network Devices Deep Dive

> [!abstract] Overview
> This note analyzes the operation, packet-forwarding logic, and configuration roles of standard network hardware. It covers Hubs, Bridges, Switches, Routers, Firewalls, WAPs, and NICs, providing a reference for network design and diagnostics.

---
## Concept
Think of network devices as the staff in a large sorting office:
- A **Hub** is a blind worker who receives a letter, makes copies of it, and runs around screaming the message to everyone in the building.
- A **Switch** is a smart clerk who checks the name on the envelope, looks up their desk location in a registry (MAC Table), and walks the letter directly to that specific desk.
- A **Router** is the dispatcher who looks at the city and zip code (IP Address) and decides which delivery truck or airplane path is the fastest route to get the letter to a completely different office building.
- A **Firewall** is the security guard standing at the main entrance, checking credentials and only allowing approved visitors or mail packages into the building.

*Seedha simple mein: Network devices hardware components hain jo data ko sahi direction mein bhejte hain. Hub sabko data bhejta hai, Switch specific computer ko bhejta hai, Router pure network ke beech traffic route karta hai, aur Firewall security check lagata hai.*

---
## Technical Deep Dive

### 1. Hub (Layer 1 - Physical)
- **Operation:** A passive multi-port repeater. It has no intelligence. When a frame enters one port, it electrical-copies the signal and floods it out of all other ports.
- **Why deprecated:** 
  - **Shared Bandwidth:** All ports share the same bandwidth.
  - **Single Collision Domain:** Devices must use CSMA/CD to detect electrical collisions. Only one device can transmit at a time.
  - **Security Risk:** Every device connected to the hub can inspect all traffic (using promiscuous mode).

### 2. Bridge (Layer 2 - Data Link)
- **Operation:** A software-based device used to split a large collision domain into two. It keeps a record of MAC addresses on each side of the link.
- **Significance:** It solved the problem of excessive collisions in early coax-cable bus networks by filtering traffic: if Host A sends a frame to Host B on the same side of the bridge, the bridge blocks the frame from crossing to the other side.

### 3. Switch (Layer 2 - Data Link)
- **Operation:** A hardware-based multi-port bridge. It uses ASICs (Application-Specific Integrated Circuits) to forward frames in microsecond speeds.
- **CAM (Content Addressable Memory) Table:** The switch learns the source MAC address of incoming frames and maps them to the physical port number.
- **Forwarding Methods:**
  - **Store-and-Forward:** Receives the entire frame, calculates the CRC checksum to verify integrity, then forwards. Safe but slowest.
  - **Cut-Through:** Reads only the destination MAC address (first 14 bytes) and immediately starts forwarding. Lowest latency but forwards corrupted frames.
  - **Fragment-Free:** Reads the first 64 bytes (the collision window) to ensure no collision has occurred, then forwards.

### 4. Router (Layer 3 - Network)
- **Operation:** Connects different networks (broadcast domains). Reads Layer 3 IP headers.
- **Routing Table:** Matches destination IP networks to outbound interfaces or next-hop IP addresses.
- **ASIC vs. CPU:** Route processing (determining the path) is handled in software/control plane; packet forwarding is offloaded to specialized hardware ASICs (data plane).

### 5. Repeater & Access Point
- **Repeater (L1):** Receives weak signals, cleans up electrical noise, amplifies, and retransmits.
- **Wireless Access Point (WAP) (L2):** Acts as a bridge between wireless networks (802.11) and wired networks (802.3). Operates in modes like Root (standard WAP), Bridge (connecting two LANs), or Repeater (extending wireless range).
  - **SSID (Service Set Identifier):** The logical name of the wireless network.
  - **Channels:** Frequency segments (e.g., Channels 1, 6, 11 in 2.4GHz) selected to prevent overlapping signal interference.

### 6. Firewall (Layer 3 - 7)
- **Stateless Firewall (Packet Filtering):** Inspects individual packets in isolation based on static rules (Source IP, Destination IP, Port, Protocol). Does not track connection state.
- **Stateful Firewall:** Tracks the state of active TCP connections (using a state table). If an outbound connection is established (e.g., client requests a web page), return traffic is allowed automatically.
- **Next-Generation Firewall (NGFW):** Inspects the actual application layer payload (deep packet inspection). Detects malware, prevents intrusions (IPS), and filters traffic based on user identities rather than just IP ports.

### 7. Network Interface Card (NIC)
- **Operation:** The physical controller connecting a device to network media. 
- **MAC Address:** Burned into the ROM of the NIC.
- **Duplex Modes:**
  - **Half-Duplex:** Device can transmit or receive, but not both at the same time (like a walkie-talkie). Common in hubs.
  - **Full-Duplex:** Device can transmit and receive simultaneously (like a telephone). Common in modern switches.
  - **Auto-Negotiation:** NIC and switch port communicate to automatically select the highest matching speed (e.g., 1 Gbps) and duplex mode (Full-Duplex).

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Cisco Packet Tracer software installed on a workstation.

### Step 1: Set up the Topology
1. Open Cisco Packet Tracer.
2. Add the following devices to the canvas:
   - 1x Generic Hub
   - 1x Catalyst 2960 Switch
   - 1x 1941 Router
   - 3x PCs
3. Connect PC0, PC1, and PC2 to ports 1, 2, and 3 on the **Hub**. Labeled this subnet `192.168.1.0/24`.
4. Run another link from the Hub to the **Switch**.
5. Connect the Switch to the Gig0/0 port on the **Router**.

### Step 2: Configure IP Addresses
1. Click on **PC0** -> Desktop -> IP Configuration. Assign IP `192.168.1.10`, Mask `255.255.255.0`, Gateway `192.168.1.1`.
2. Click on **PC1** -> Desktop -> IP Configuration. Assign IP `192.168.1.11`, Mask `255.255.255.0`, Gateway `192.168.1.1`.

### Step 3: Observe Packet Flow in Simulation Mode
1. Switch to **Simulation Mode** (bottom right stopwatch icon).
2. Click **Add Simple PDU** (the envelope icon). Click **PC0** (source) then click **PC1** (destination).
3. Click the **Play** button.
4. **Observe the Hub:** Watch the packet arrive at the Hub. The Hub copies the packet and broadcasts it to both PC2 and the Switch. PC2 discards the packet (since the IP doesn't match), and the Switch receives it.
5. **Observe the Switch:** Now, send a packet from PC1 to PC0. The Switch checks its MAC table, realizes PC0 is located on the port leading back to the Hub, and forwards the packet *only* down that port, avoiding any other ports connected directly to the Switch.

---
## Commands Reference
Operating system and switch command sequences for network device inspection:

```bash
# Cisco IOS (Switch CLI)
show mac address-table            # Display the switch's learned MAC-to-Port mappings
clear mac address-table dynamic   # Flush the dynamic MAC table to force relearning
show interfaces status            # Check port speeds, duplex configurations, and VLANs

# Windows
# Query details about the physical network interface cards (NICs)
Get-NetAdapter | Select-Object Name, InterfaceDescription, MacAddress, LinkSpeed, Status
netsh interface show interface    # Show state of local interfaces
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Workstation users connected to a network segment complain of extremely slow speeds and frequent disconnections. Ping packet loss is high ($15\%-30\%$).
- **Root Cause:** A duplex mismatch. The switch port is manually configured for Full-Duplex, while the workstation NIC is configured for Half-Duplex (or vice versa), causing collision errors.
- **Fix:**
  1. Check the switch port statistics. Look for high counts of **CRC errors**, **runts**, and **collisions**.
  2. Access the switch CLI or workstation NIC settings.
  3. Change the settings on both ends to **Auto-Negotiation**, or force both to **1000Mbps / Full-Duplex**.
  4. Verify that interface collision errors stop increasing.

**Scenario 2:**
- **Problem:** A server cannot communicate with a database server located on a different VLAN/Subnet. Pinging the database returns "Request timed out".
- **Root Cause:** Layer 3 routing configuration failure or firewall blocking the traffic.
- **Fix:**
  1. Run `tracert [Database_IP]` from the source server. Note where the path stops.
  2. If the trace stops at the local default gateway, the router lacks a route to the destination network, or the firewall at the gateway is blocking the ports.
  3. Log into the router. Run `show ip route` to check if a route exists.
  4. If a route exists, check the stateful firewall logs between the zones to verify if traffic on the database port (e.g., TCP 1433 for SQL) is blocked by security policy rules.

---
## Common Mistakes
> [!warning] Avoid These
> **Assuming a switch routes traffic between subnets:** Connecting two networks with different subnets (e.g., `192.168.1.0/24` and `10.0.0.0/24`) to a Layer 2 switch and wondering why they cannot communicate. L2 switches only forward traffic based on MAC addresses within the same subnet.
> **Correct approach:** Use a Router or a Layer 3 Switch to perform routing between different IP subnets.

---
## Pro Tips
> [!tip] Field Experience
> When configuring ports on critical servers, always disable auto-negotiation and manually lock both the server NIC and the switch ports to their maximum supported speed and Full-Duplex. This prevents rare negotiation failures during server reboots that could drop a gigabit link down to 10Mbps half-duplex.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Hub | Passive Layer 1 repeater; floods traffic to all ports, creating a single collision domain. |
| 2 | Switch | Layer 2 device; uses a MAC address table to forward frames directly to target ports. |
| 3 | Router | Layer 3 device; forwards packets between distinct networks based on IP routing tables. |
| 4 | Stateful Firewall | Security device that tracks the state of TCP sessions, allowing reply traffic automatically. |
| 5 | CAM Table | The switch's internal map linking physical ports to the MAC addresses of connected devices. |

---
## Interview Q&A

**Q1: What is the difference between a Layer 2 Switch and a Layer 3 Switch?**
A: A **Layer 2 Switch** forwards traffic based purely on physical MAC addresses (Data Link Layer). It cannot read IP headers or route traffic between different subnets. A **Layer 3 Switch** (also known as a Multilayer Switch) combines the speed of switch hardware (ASICs) with the routing capabilities of a router. It can read Layer 3 IP headers, configure routing protocols (like OSPF or static routes), and perform Inter-VLAN routing at wire speed using hardware routing tables.

**Q2: A company's Wi-Fi network suffers from constant dropouts in a crowded high-rise office. You suspect co-channel interference. How do you resolve this?**
A: 
- **Situation:** The company's Wi-Fi is unstable in a high-density office area.
- **Task:** Identify the frequency overlap and reconfigure channels to minimize interference.
- **Action:** I will use a Wi-Fi analyzer tool to map the surrounding SSIDs and active channels. In the 2.4 GHz spectrum, only channels 1, 6, and 11 do not overlap. If nearby offices are using channels 2, 3, or 4, they create co-channel and adjacent-channel interference. I will configure the WAPs to use non-overlapping channels (1, 6, 11) or migrate users to the 5 GHz band, which has many more non-overlapping channels.
- **Result:** Transitioning devices to 5 GHz and locking 2.4 GHz WAPs to clean, non-overlapping channels resolves the dropouts.

**Q3: Explain how a switch builds its MAC address table (CAM table).**
A: A switch builds its CAM table dynamically by inspecting the **Source MAC Address** of incoming frames. When Host A (MAC: `00aa.bbcc.ddee`) plugs into Port 1 and transmits its first frame, the switch reads the source field and registers `Port 1 -> 00aa.bbcc.ddee` in its table. If the Destination MAC in that frame is unknown (not yet in the table), the switch performs "unknown unicast flooding," copying the frame to all active ports except Port 1. When the destination host responds, the switch records its MAC address and port, ending the need to flood frames for that path.

---
## Related Notes
- [[01-Foundations/02-Networking/N-01 Networking Fundamentals|N-01 Networking Fundamentals]] — Overview of network topologies and the OSI model.
- [[01-Foundations/02-Networking/N-03 Ethernet and MAC Address|N-03 Ethernet and MAC Address]] — Details on frames, MAC formats, and ARP.
- [[01-Foundations/02-Networking/N-06 Switching — VLANs and Trunking|N-06 Switching — VLANs and Trunking]] — Dynamic switch configurations and VLAN design.
