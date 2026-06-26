---
tags: [sysadmin, networking, switching, vlan]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# N-06: Switching — VLANs and Trunking

> [!abstract] Overview
> This note covers Layer 2 switching concepts, including Virtual Local Area Network (VLAN) design, 802.1Q trunk encapsulation, Voice VLAN implementation, and Inter-VLAN routing (Router-on-a-Stick). It details configurations for Cisco IOS environments.

---
## Concept
Think of a physical switch as a single floor in a co-working space. By default, everyone is in one open hall; anyone can talk to or shout at anyone else (one large broadcast domain). 

If you want to separate the Finance department from the guest visitors, you could build physical brick walls and buy separate tables (separate physical switches), which is expensive. 

Instead, you use **VLANs** to set up invisible, soundproof glass partitions. Finance workers get blue badges (VLAN 10) and guests get yellow badges (VLAN 20). People with blue badges can only talk to blue badges. 

If a blue badge wants to talk to a yellow badge, they must exit the floor and pass through the security guard desk in the lobby (Router/Layer 3 device). 

A **Trunk** is like a high-speed elevator between floors that carries badges of all colors, keeping them organized by attaching color-coded tags as they enter.

*Seedha simple mein: VLAN physical switch ko chote-chote logical switches mein divide karta hai taaki alag-alag departments ka traffic isolated rahe. Trunk port ek aisi link hai jo multiple VLANs ke data ko ek hi cable se do switches ke beech transport karti hai.*

---
## Technical Deep Dive

### 1. The Switch MAC Table Learning Process
As reviewed in [[01-Foundations/02-Networking/N-02 Network Devices Deep Dive|N-02 Network Devices Deep Dive]], the switch builds its CAM (Content Addressable Memory) table dynamically. It extracts the source MAC address of incoming frames, maps them to the physical port, and registers the VLAN ID assigned to that port.

### 2. The VLAN Concept
A VLAN (Virtual Local Area Network) isolates traffic at Layer 2, splitting a single physical switch into multiple logical switches. Each VLAN represents a distinct broadcast domain and requires its own unique IP subnet.
- **Why we need VLANs:**
  - **Security:** Isolates sensitive data (e.g., HR, Finance) from guest users.
  - **Performance:** Limits the propagation of broadcast frames, reducing network overhead.
  - **Flexibility:** Groups users by logical function rather than physical location.

### 3. VLAN Types
- **Data VLAN (Default VLAN 1):** Carries user-generated traffic (e.g., web browsing, email).
- **Voice VLAN:** Labeled with higher QoS (Quality of Service) priority to ensure clear VoIP calling. Laptops connect through the phone's built-in switch, sharing a single physical switch port.
- **Management VLAN:** Dedicated subnet reserved for SSH/HTTPS administrative access to switches and routers (e.g., VLAN 99).
- **Native VLAN:** Standard 802.1Q trunks expect all frames to be tagged. The Native VLAN is the single VLAN whose frames are sent across the trunk **untagged**. Both ends of the trunk must configure matching Native VLANs (Default is VLAN 1) to prevent traffic leakage.

### 4. Access Ports vs. Trunk Ports
- **Access Port:** Belongs to a single, specific VLAN. Used to connect end devices (PCs, printers, servers). Frames leaving the port to the end device are untagged.
- **Trunk Port:** Carries traffic for multiple VLANs simultaneously. Used to connect Switch-to-Switch, Switch-to-Router, or Switch-to-Hypervisor.

### 5. 802.1Q Tagging Format
IEEE 802.1Q inserts a 4-byte tag into the standard Ethernet II frame header after the Source MAC field:

```
+------------+------------+-------------------+-----------+---------+-----+
| Dest MAC   | Source MAC | 802.1Q Tag        | Ethertype | Payload | FCS |
| (6 Bytes)  | (6 Bytes)  | (4 Bytes)         | (2 Bytes) |         |     |
+------------+------------+-------------------+-----------+---------+-----+
                            |                 |
                            v                 v
                       [TPID: 0x8100]  [TCI: PRI (3b) + DEI (1b) + VLAN ID (12b)]
```
- **VLAN ID (VID):** A 12-bit field specifying the VLAN number. Supports values from $1$ to $4094$.
  - Normal range: 1 – 1005.
  - Extended range: 1006 – 4094.

### 6. Inter-VLAN Routing (Router-on-a-Stick)
Because VLANs are isolated broadcast domains, they cannot communicate directly. To route traffic between them, a Layer 3 device is required. The **Router-on-a-Stick (ROAS)** method routes traffic between multiple VLANs using a single physical router interface connected to a switch trunk port. The router interface is split into logical **sub-interfaces** (e.g., `Gig0/0.10`), each acting as the default gateway for its respective VLAN.

---
## Cisco IOS Configuration Commands

### VLAN Creation and Access Assignment
```text
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name Sales_Dept
Switch(config-vlan)# exit
Switch(config)# interface fastethernet 0/5
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
```

### Trunk Port Configuration
```text
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# switchport trunk encapsulation dot1q   ! (Required on legacy switches like Catalyst 3560)
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99        ! (Changes Native VLAN to 99)
Switch(config-if)# switchport trunk allowed vlan 10,20,99 ! (Limits allowed VLANs)
```

### Voice VLAN Configuration (IP Phone + PC)
```text
Switch(config)# vlan 150
Switch(config-vlan)# name Voice_Traffic
Switch(config-vlan)# exit
Switch(config)# interface fastethernet 0/10
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10              ! (Data VLAN for PC)
Switch(config-if)# switchport voice vlan 150              ! (Voice VLAN for IP Phone)
```

### Router-on-a-Stick (ROAS) Configuration
```text
Router# configure terminal
Router(config)# interface gigabitethernet 0/0
Router(config-if)# no shutdown                           ! (Enable physical port - do not assign IP)
Router(config-if)# exit
Router(config)# interface gigabitethernet 0/0.10         ! (Create sub-interface for VLAN 10)
Router(config-subif)# encapsulation dot1Q 10              ! (Bind sub-interface to VLAN 10 tagging)
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit
Router(config)# interface gigabitethernet 0/0.20         ! (Create sub-interface for VLAN 20)
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Cisco Packet Tracer with 1x 2960 Switch, 1x 1941 Router, and 2x PCs.

### Step 1: Physical Cabling
1. Connect PC0 to Switch Port `Fa0/5`.
2. Connect PC1 to Switch Port `Fa0/10`.
3. Connect Switch Port `Gi0/1` to Router Port `Gi0/0`.

### Step 2: Configure VLANs on the Switch
1. Double-click Switch -> CLI tab. Enter global config:
   ```text
   en
   conf t
   vlan 10
   name Sales
   vlan 20
   name HR
   ```
2. Assign ports to VLANs:
   ```text
   interface fa0/5
   switchport mode access
   switchport access vlan 10
   interface fa0/10
   switchport mode access
   switchport access vlan 20
   ```
3. Configure the Switch trunk uplink:
   ```text
   interface gi0/1
   switchport mode trunk
   ```

### Step 3: Configure ROAS on the Router
1. Double-click Router -> CLI tab. Configure sub-interfaces:
   ```text
   en
   conf t
   interface gi0/0
   no shut
   interface gi0/0.10
   encapsulation dot1Q 10
   ip address 192.168.10.1 255.255.255.0
   interface gi0/0.20
   encapsulation dot1Q 20
   ip address 192.168.20.1 255.255.255.0
   ```

### Step 4: Verify Connectivity
1. Configure PC0: IP `192.168.10.50/24`, Gateway `192.168.10.1`.
2. Configure PC1: IP `192.168.20.50/24`, Gateway `192.168.20.1`.
3. Ping from PC0 to PC1 (`ping 192.168.20.50`).
4. **Verify:** The ping should succeed. Run `show mac address-table` on the switch to verify port allocations.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** PC0 (VLAN 10) cannot ping PC1 (VLAN 10) connected to a different switch. The physical links are active, but local devices cannot see each other.
- **Root Cause:** A trunk link mismatch. Either the switch-to-switch link is not configured as a trunk, or the allowed VLAN list excludes VLAN 10.
- **Fix:**
  1. Log into both switches. Run `show interfaces trunk`.
  2. Verify that the connecting ports are in trunking mode. If they read `access`, run:
     ```text
     switchport mode trunk
     ```
  3. Verify that the allowed VLAN list on the trunk includes VLAN 10:
     ```text
     switchport trunk allowed vlan add 10
     ```

**Scenario 2:**
- **Problem:** Syslog alerts display error messages: `%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on GigabitEthernet0/1 (99), with Switch2 GigabitEthernet0/1 (1).` The network experiences high packet loss.
- **Root Cause:** Native VLAN mismatch. Switch1 expects untagged frames to belong to VLAN 99, while Switch2 expects them to belong to VLAN 1. This causes frame leakage and routing loops.
- **Fix:**
  1. Log into Switch2.
  2. Locate the connecting trunk port `Gi0/1`.
  3. Align the native VLAN configuration:
     ```text
     interface gi0/1
     switchport trunk native vlan 99
     ```
  4. Confirm that syslog native VLAN alerts stop repeating.

---
## Common Mistakes
> [!warning] Avoid These
> **Configuring IP addresses on physical Router-on-a-Stick interface ports:** Assigning an IP directly to the physical interface `Gi0/0` when planning to use sub-interfaces. This prevents routing updates and breaks sub-interface encapsulation processing.
> **Correct approach:** Leave the physical port `Gi0/0` IP configuration unassigned. Only run `no shutdown` to enable the port, then assign IPs to the sub-interfaces (e.g., `Gi0/0.10`).

---
## Pro Tips
> [!tip] Field Experience
> For security compliance, always change the Native VLAN from the default VLAN 1 to an unused dummy VLAN ID (e.g., VLAN 999), and configure the trunk to prune (disallow) VLAN 999. This mitigates "VLAN Hopping" exploits where attackers inject double-tagged 802.1Q headers to cross network boundaries.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | VLAN | Layer 2 broadcast domain isolation; limits broadcast domains within a single physical switch. |
| 2 | Access Port | Switch port dedicated to a single VLAN; strips all tags when transmitting data to endpoints. |
| 3 | Trunk Port | Multiplexed port carrying traffic for multiple VLANs using 802.1Q tags. |
| 4 | Native VLAN | The single VLAN whose frames traverse a trunk link untagged; must match on both ends. |
| 5 | ROAS | Router-on-a-Stick; routes traffic between VLAN subnets using sub-interfaces on a single physical link. |

---
## Interview Q&A

**Q1: What is a Native VLAN mismatch, and what are its operational consequences?**
A: A Native VLAN mismatch occurs when the trunk ports on two connected switches are configured with different Native VLAN IDs. Under 802.1Q, any frame belonging to the Native VLAN is sent untagged. If Switch A (Native VLAN 10) sends an untagged frame across the trunk, Switch B (Native VLAN 20) receives the untagged frame and automatically assigns it to VLAN 20. This causes traffic to leak between VLANs without passing through a router, creating severe security vulnerabilities, Spanning Tree Protocol (STP) topology instability, and packet loss.

**Q2: A client's IP Phones are disconnecting intermittently. The client has both computers and phones plugged into the same switch ports. Explain your diagnosis process.**
A: 
- **Situation:** IP Phones are unstable when sharing switch ports with client workstations.
- **Task:** Verify VLAN segregation and priority marking configurations on the switch ports.
- **Action:** First, I will review the switch port configuration. If both the phone and PC are in a single flat data VLAN, broadcast storms from PCs can saturate the phone's network interface. Second, I will check if the voice VLAN command (`switchport voice vlan [ID]`) is missing. This command instructs the phone to tag its traffic with 802.1p CoS (Class of Service) values, giving voice packets high-priority queuing.
- **Result:** Configuring distinct Data and Voice VLANs on the ports resolves the phone instability.

**Q3: How does VLAN Hopping work, and how do you configure a switch to prevent it?**
A: VLAN Hopping is an attack where a host transmits traffic to VLANs outside its access scope. It can occur via **switch spoofing** (where the host runs dynamic trunking protocols to trick the switch port into forming a trunk) or **double tagging** (where an attacker sends a frame with two 802.1Q tags; the outer tag matches the Native VLAN of a trunk, causing the first switch to strip the outer tag, and the second switch to forward the inner-tagged packet to the target VLAN). To prevent it, configure all client ports as explicit access ports (`switchport mode access`), disable DTP (`switchport nonegotiate`), and change the native VLAN of trunks to an unused, isolated VLAN.

---
## Related Notes
- [[01-Foundations/02-Networking/N-02 Network Devices Deep Dive|N-02 Network Devices Deep Dive]] — Base switch operations and CAM table learning.
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — VLAN IP subnetting layouts.
- [[01-Foundations/02-Networking/N-07 Routing — Static and Dynamic|N-07 Routing — Static and Dynamic]] — Routing protocol configurations between VLAN networks.


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
