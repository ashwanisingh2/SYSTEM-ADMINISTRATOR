---
tags: [sysadmin, networking, ethernet, arp]
difficulty: Intermediate
lab-required: Yes
read-time: 12 mins
---

# N-03: Ethernet and MAC Address

> [!abstract] Overview
> This note covers physical and logical layer standards for Ethernet networks. It details copper cabling profiles, MAC addressing architecture, the Address Resolution Protocol (ARP), frame structures, and broadcast domains.

---
## Concept
Think of a MAC address as your physical government-issued national ID card number. It is unique to you, printed on plastic (burned into the NIC chip), and never changes no matter where you move. 

When you want to mail a letter to someone in your office, you can't just shout their name. You must find their physical location. You shout out, "Who has ID number 12345?" (ARP request broadcast). They respond, "That's me! I'm sitting at Desk 4" (ARP reply unicast). Now, you can wrap your letter in a protective folder (Ethernet frame) labeled "From: My ID, To: 12345" and hand it to the mail clerk (Switch), who delivers it directly to Desk 4.

*Seedha simple mein: MAC address hardware ka unique footprint hota hai. Jab ek local network mein computers ko aapas mein baat karni hoti hai, toh woh ARP protocol ka use karke IP address se MAC address ka pata lagate hain aur switch ke zariye frame send karte hain.*

---
## Technical Deep Dive

### 1. Ethernet Standards & Speeds
Ethernet is defined by the IEEE **802.3** working group.

- **10BASE-T:** Legacy Ethernet running at 10 Mbps over copper twisted pair. Max length 100m.
- **100BASE-TX (Fast Ethernet):** Runs at 100 Mbps over Cat5 or better copper. Uses two pairs of wires.
- **1000BASE-T (Gigabit Ethernet):** Runs at 1 Gbps over Cat5e or better. Requires all four pairs of wires in the cable. Max length 100m.
- **10GBASE-T:** Runs at 10 Gbps over copper (Cat6 up to 55m, Cat6a up to 100m).

### 2. Copper Cabling Categories (UTP/STP)
Twisted-pair cables use copper wires twisted together to reduce **crosstalk** (electromagnetic interference between wire pairs).
- **Cat5e:** Speed up to 1 Gbps. Bandwidth 100 MHz. Shielded or unshielded.
- **Cat6:** Speed up to 10 Gbps (limit 55 meters). Bandwidth 250 MHz. Features a plastic spline splitter inside to separate wire pairs.
- **Cat6a (Augmented):** Speed up to 10 Gbps (full 100 meters). Bandwidth 500 MHz. Fully shielded pairs.
- **Cat7:** Speed up to 10 Gbps. Bandwidth 600 MHz. Requires individual shielding on each pair and the outer jacket (GG45/TERA connectors). Not widely adopted in enterprise compared to Cat6a.

### 3. Pinouts: Straight-Through vs. Crossover Cables
Modern networking cables are wired using two standards: **T568A** and **T568B**.

```
T568B Pinout Order:
Pin 1: Orange/White  | Pin 2: Orange      | Pin 3: Green/White | Pin 4: Blue
Pin 5: Blue/White    | Pin 6: Green       | Pin 7: Brown/White | Pin 8: Brown
```

- **Straight-Through Cable:** Both ends are wired using the same standard (usually T568B on both ends). Used to connect **dissimilar devices** (e.g., PC to Switch, Switch to Router).
- **Crossover Cable:** One end is T568A, the other is T568B. Pins 1 & 2 swap with 3 & 6. Used to connect **similar devices** (e.g., PC to PC, Switch to Switch).
- *Note:* Modern devices use **Auto-MDIX** (Medium Dependent Interface Crossover) to detect the connection type automatically and swap transmit/receive paths in silicon, making physical crossover cables mostly obsolete.

### 4. MAC Address Format
A Media Access Control (MAC) address is a 48-bit physical address expressed as 12 hexadecimal characters (e.g., `00:1A:2B:3C:4D:5E` or `001a.2b3c.4d5e`).
- **OUI (Organizationally Unique Identifier):** The first 24 bits (first 6 hex characters). Assigned by the IEEE to the NIC manufacturer (e.g., `00:00:0C` is Cisco).
- **Device Identifier:** The last 24 bits (last 6 hex characters). A unique serial number assigned by the manufacturer.

### 5. Address Resolution Protocol (ARP)
ARP maps Layer 3 IP addresses to Layer 2 MAC addresses within a local network.
- **Step 1: Check Local Cache:** Host A wants to send data to `192.168.1.20`. It checks its local ARP cache.
- **Step 2: The Broadcast Request:** If not found, Host A broadcasts an ARP Request frame: "Who has IP `192.168.1.20`? Tell `192.168.1.10`." 
  - Destination MAC: `FF:FF:FF:FF:FF:FF` (Broadcast).
- **Step 3: The Unicast Reply:** All hosts on the LAN segment receive the broadcast. Only the host with IP `192.168.1.20` responds with a unicast ARP Reply: "I have that IP. My MAC is `00:aa:bb:11:22:33`."
- **Step 4: Cache Update:** Host A saves the MAC-to-IP mapping in its ARP table and transmits the payload frame.

### 6. Transmission Types
- **Unicast:** One-to-one communication. Frame targets a single destination MAC address.
- **Broadcast:** One-to-all communication. Target MAC is `FF:FF:FF:FF:FF:FF`. Every device on the local network must process the frame.
- **Multicast:** One-to-many communication. Target MAC starts with a reserved range (e.g., `01:00:5E` for IPv4 multicast). Only subscribed hosts process it.

### 7. Ethernet II Frame Structure
```
+-------------+-------------+------------------+------------------+-----------+-----------+-------+
| Preamble    | SFD         | Destination MAC  | Source MAC       | Type/     | Payload   | FCS   |
| (7 Bytes)   | (1 Byte)    | (6 Bytes)        | (6 Bytes)        | Length    | (46-1500  | (4 B) |
|             |             |                  |                  | (2 Bytes) | Bytes)    |       |
+-------------+-------------+------------------+------------------+-----------+-----------+-------+
```
- **Preamble & SFD (Start Frame Delimiter):** Synchronizes receiver clock timing.
- **Ethertype:** Labeled as Type/Length. Identifies the encapsulated Layer 3 protocol (e.g., `0x0800` for IPv4, `0x0806` for ARP).
- **Payload:** The Layer 3 packet. Minimum size is 46 bytes; maximum is 1500 bytes (standard MTU).
- **FCS (Frame Check Sequence):** A 32-bit Cyclic Redundancy Check (CRC) value. If the receiving switch calculates a mismatch, the frame is discarded as corrupted.

### 8. Collision Domain vs. Broadcast Domain
- **Collision Domain:** A physical network segment where devices share media and electrical collisions can occur. Switches break up collision domains (every port is a separate collision domain running full-duplex).
- **Broadcast Domain:** A logical division of a computer network where any device can transmit broadcast frames directly to all other devices. Routers break up broadcast domains (each router port is a separate broadcast domain).

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows client machine connected to a local switch.

### Step 1: Check Local ARP Table
1. Open PowerShell or Command Prompt.
2. View your current ARP cache mappings:
   ```cmd
   arp -a
   ```
3. Locate the IP address of your default gateway and write down its physical MAC address.

### Step 2: Clear the ARP Cache
1. Open PowerShell as Administrator.
2. Clear the cache to force a new lookup sequence:
   ```powershell
   # Windows (PowerShell admin)
   Remove-NetNeighbor -AddressFamily IPv4
   
   # Alternative (Command Prompt admin)
   arp -d *
   ```

### Step 3: Trigger ARP and Verify
1. Ping a host on the local network (e.g., your gateway):
   ```cmd
   ping 192.168.1.1
   ```
2. Re-run the ARP command:
   ```cmd
   arp -a
   ```
3. Verify that the entry for `192.168.1.1` has re-appeared in the cache dynamically.

---
## Commands Reference
Advanced content only — basics in [[Basic networking commands]]

Diagnostic tools for viewing local MAC configuration and executing table clearing:

```bash
# Windows
getmac /v                         # Show physical adapters and associated MAC addresses
arp -a                            # Display active ARP table entries
arp -d 192.168.1.50               # Delete specific IP address mapping from ARP cache

# Linux
ip link show                      # Display MAC addresses of active interfaces
ip neighbor show                  # View Linux ARP table (IP-to-MAC mapping)
sudo ip neigh flush all           # Flush the complete ARP table on all interfaces
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** IP address conflicts. A workstation user complains they lose connection to local servers intermittently. Event Viewer logs warning: "The system detected an IP address conflict for 192.168.1.100 with the system having network hardware address 00:11:22:AA:BB:CC."
- **Root Cause:** A static IP address was configured manually on a device that conflicts with a dynamically assigned DHCP address.
- **Fix:**
  1. Ping the conflicting IP `192.168.1.100` from a third machine.
  2. Run `arp -a 192.168.1.100` to find the MAC address of the conflicting host.
  3. Look up the OUI prefix of that MAC to identify the device manufacturer (e.g., `00:11:22` = Apple).
  4. Track down the device or reconfigure the workstation to obtain an IP automatically via DHCP to avoid the conflict.

**Scenario 2:**
- **Problem:** An industrial PLC (Programmable Logic Controller) is replaced, but it cannot communicate with the legacy HMI panel. The configuration, IPs, and cabling are correct.
- **Root Cause:** The old HMI cache contains a stale static ARP entry pointing to the old PLC's MAC address.
- **Fix:**
  1. Log into the HMI controller console.
  2. Clear the ARP cache table or reboot the HMI panel.
  3. Ping the new PLC IP from the HMI.
  4. The HMI broadcasts a new ARP request, maps the new PLC MAC address, and restores communication.

---
## Common Mistakes
> [!warning] Avoid These
> **Setting manual duplicate MAC addresses:** Configuring duplicate MAC addresses on a network (usually through virtualization cloning). This confuses switch CAM tables, causing incoming frames to flap erratically between ports, dropping packets on both host virtual machines.
> **Correct approach:** Ensure every virtual network adapter has a unique, randomly generated MAC address during VM replication.

---
## Pro Tips
> [!tip] Field Experience
> When buying network patches, avoid CCA (Copper Clad Aluminum) cables. They are cheaper but brittle, have higher resistance, fail PoE requirements, and struggle to support gigabit speeds over distances where pure bare-copper Cat6 cables run reliably.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Cat6 vs Cat6a | Cat6 supports 10G up to 55m; Cat6a is fully shielded and supports 10G up to the full 100m limit. |
| 2 | OUI | The first 24 bits of a MAC address that identify the hardware manufacturer. |
| 3 | ARP Request | A Layer 2 broadcast frame (`FF:FF:FF:FF:FF:FF`) sent to query the MAC address of an IP. |
| 4 | Collision Domain | Segment where data collisions occur; isolated by switch ports running full-duplex. |
| 5 | Broadcast Domain| Segment where broadcast frames travel; bounded and isolated by router ports. |

---
## Interview Q&A

**Q1: What is the difference between an ARP Request and an ARP Reply at Layer 2?**
A: An **ARP Request** is a Layer 2 broadcast frame. Since the sender does not know the destination MAC address, it must query all devices on the local segment. The destination MAC in the Ethernet header is set to `FF:FF:FF:FF:FF:FF`. An **ARP Reply** is a Layer 2 unicast frame. Since the responding host knows the sender's MAC address from the incoming request, it replies directly. The destination MAC is set to the specific sender's MAC address.

**Q2: A workstation is moved to a new floor. It connects physically and gets an IP, but the user cannot ping the local gateway. Describe how you would troubleshoot this.**
A: 
- **Situation:** A physical link is established and IP is assigned, but gateway communication fails.
- **Task:** Verify MAC learning and VLAN matching across the connection.
- **Action:** First, I will run `ipconfig` to verify the IP subnet configuration. Second, I will check the local ARP table (`arp -a`) to see if the gateway's IP is mapped to a MAC. If the MAC is missing or reads `00-00-00-00-00-00`, there is a Layer 2 failure. Third, I will check if the switch port has been assigned to the incorrect VLAN.
- **Result:** Reconfiguring the switch port to the correct VLAN matching the workstation's subnet allows ARP resolution and restores gateway connection.

**Q3: What are runt and giant frames in Ethernet networks, and what causes them?**
A: **Runt frames** are Ethernet frames that are smaller than the minimum allowable size of 64 bytes (excluding preamble/SFD) and have an invalid FCS. They are usually caused by collisions on half-duplex links or physical card failures. **Giant frames** (or jumbo frames) are frames that exceed the standard maximum transmission unit (MTU) size of 1518 bytes (or 1522 bytes with VLAN tags). They are caused by mismatch configurations where a host sends jumbo frames (often up to 9000 bytes for storage performance) to a switch port that does not have jumbo frames enabled.

---
## Related Notes
- [[01-Foundations/02-Networking/N-01 Networking Fundamentals|N-01 Networking Fundamentals]] — Overview of LAN classifications and the OSI model layers.
- [[01-Foundations/02-Networking/N-02 Network Devices Deep Dive|N-02 Network Devices Deep Dive]] — Details on switch MAC tables and packet forwarding.
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Information on IP subnets and gateway routing.
