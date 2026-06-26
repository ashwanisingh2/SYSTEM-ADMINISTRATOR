---
tags: [networking, ethernet, arp, ccna]
aliases: [N-03, Ethernet, MAC]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#intermediate` `#ccna`

# N-03: Ethernet and MAC Address

> [!abstract] Overview
> Yeh note physical and logical layer standards for Ethernet networks ko cover karta hai. Ismein copper cabling profiles, MAC addressing architecture, Address Resolution Protocol (ARP), frame structures, aur broadcast domains ki details hain. Ek support engineer ke liye troubleshooting aur network connectivity concepts samajhne ke liye yeh foundation janna zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — MAC address is a 48-bit physical identifier burned into the network interface card (NIC).
- **Why it matters** — It allows devices to find each other and communicate on a local network segment using Layer 2 frames.
- **Where you see this** — Troubleshooting IP conflicts, checking ARP tables, understanding switch port forwarding.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check physical cabling, verify MAC address using `getmac` or `ipconfig /all`. |
| **L2** | Configure switch ports, troubleshoot ARP cache issues, resolve IP conflicts manually. |
| **L3** | Design VLAN architecture, manage complex spanning tree and enterprise broadcast domains. |

> [!tip] Seedha Simple Mein
> *MAC address hardware ka unique footprint hota hai. Jab ek local network mein computers ko aapas mein baat karni hoti hai, toh woh ARP protocol ka use karke IP address se MAC address ka pata lagate hain aur switch ke zariye frame send karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Physical national ID card** is like **MAC address** because...
>
> - It is unique to you, printed on plastic, and never changes no matter where you move.
> - When you want to mail a letter in your office, you shout "Who has ID 12345?" (ARP request broadcast), and they reply "I'm at Desk 4" (ARP reply unicast).
> - The mail clerk (Switch) then delivers it directly.

---
## 🔬 Technical Deep Dive

### 1. Ethernet Standards & Speeds

> [!info] Key Concept
> Ethernet is defined by the IEEE **802.3** working group.

- **10BASE-T:** Legacy Ethernet running at 10 Mbps over copper twisted pair. Max length 100m.
- **100BASE-TX (Fast Ethernet):** Runs at 100 Mbps over Cat5 or better copper.
- **1000BASE-T (Gigabit Ethernet):** Runs at 1 Gbps over Cat5e or better. Requires all four pairs. Max length 100m.
- **10GBASE-T:** Runs at 10 Gbps over copper (Cat6 up to 55m, Cat6a up to 100m).

### 2. Copper Cabling Categories (UTP/STP)
- **Cat5e:** Up to 1 Gbps. Bandwidth 100 MHz.
- **Cat6:** Up to 10 Gbps (limit 55 meters). Bandwidth 250 MHz.
- **Cat6a (Augmented):** Up to 10 Gbps (full 100 meters). Bandwidth 500 MHz. Fully shielded pairs.
- **Cat7:** Up to 10 Gbps. Bandwidth 600 MHz. Not widely adopted in enterprise compared to Cat6a.

### 3. Pinouts: Straight-Through vs. Crossover Cables
- **Straight-Through Cable:** Connects **dissimilar devices** (e.g., PC to Switch).
- **Crossover Cable:** Connects **similar devices** (e.g., PC to PC, Switch to Switch).

> [!danger] Common Mistake
> **Setting manual duplicate MAC addresses:** Configuring duplicate MAC addresses on a network (usually through virtualization cloning). This confuses switch CAM tables, causing frames to flap erratically between ports.

### 4. MAC Address Format
A 48-bit physical address expressed as 12 hexadecimal characters (e.g., `00:1A:2B:3C:4D:5E`).
- **OUI (Organizationally Unique Identifier):** The first 24 bits. Assigned to the manufacturer.
- **Device Identifier:** The last 24 bits. A unique serial number.

### 5. Address Resolution Protocol (ARP)
Maps L3 IP addresses to L2 MAC addresses.
- **Step 1:** Check local ARP cache.
- **Step 2:** Broadcast ARP Request ("Who has IP `192.168.1.20`?"). Destination MAC: `FF:FF:FF:FF:FF:FF`.
- **Step 3:** Unicast ARP Reply ("I have that IP. My MAC is X").

### 6. Collision Domain vs. Broadcast Domain
- **Collision Domain:** Physical network segment where collisions occur. Switches break these up.
- **Broadcast Domain:** Logical division where broadcast frames travel. Routers break these up.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows client machine connected to a local switch.

### Step 1: Check Local ARP Table

```cmd
# View your current ARP cache mappings
arp -a
```

> [!success] Expected Output
> Mappings of local IP addresses to physical MAC addresses.

### Step 2: Clear the ARP Cache

```powershell
# Windows (PowerShell admin) - clear cache to force a new lookup sequence
Remove-NetNeighbor -AddressFamily IPv4
```

### Step 3: Trigger ARP and Verify

```cmd
# Ping a host on the local network
ping 192.168.1.1
# Verify the entry has re-appeared dynamically
arp -a
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `getmac /v` | Show physical adapters and associated MAC addresses (Windows) | `getmac /v` |
| `arp -a` | Display active ARP table entries | `arp -a` |
| `arp -d *` | Clear all ARP entries (Admin cmd) | `arp -d *` |
| `ip link show` | Display MAC addresses of active interfaces (Linux) | `ip link show` |
| `ip neighbor show` | View Linux ARP table | `ip neighbor show` |
| `sudo ip neigh flush all` | Flush complete ARP table on all interfaces (Linux) | `sudo ip neigh flush all` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| IP address conflicts intermittently | Static IP configured manually conflicts with DHCP dynamically assigned address | Ping conflicting IP, use `arp -a` to find MAC, track down device, and reconfigure to DHCP. |
| Replaced PLC cannot communicate with legacy HMI | Old HMI cache contains a stale static ARP entry pointing to the old PLC's MAC | Log into HMI console, clear ARP cache table or reboot HMI, ping new PLC IP to force new ARP broadcast. |
| Connection drops / Frame flaps | Duplicate MAC addresses on virtual machines | Ensure every virtual network adapter has a unique, randomly generated MAC address during VM replication. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: IP Address Conflict

> [!example] Ticket
> "Workstation user complains they lose connection to local servers intermittently. Event Viewer logs warning: The system detected an IP address conflict for 192.168.1.100."

**L1 Response:** Verify the error, ping the conflicting IP from a third machine to confirm it's online.
**Escalation Trigger:** If the conflicting device is unknown or network-wide issues are occurring.
**L2 Resolution:** Run `arp -a 192.168.1.100` to find the MAC address. Look up the OUI prefix to identify the manufacturer. Track down the rogue device and reconfigure its IP to avoid conflicts.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between an ARP Request and an ARP Reply at Layer 2?
> **Answer:** An **ARP Request** is a Layer 2 broadcast frame (Destination MAC `FF:FF:FF:FF:FF:FF`) sent to query all devices on the local segment. An **ARP Reply** is a Layer 2 unicast frame sent directly back to the requester since the responding host knows the sender's MAC.

==**Exam Tip:** ARP Requests are always Broadcast; ARP Replies are always Unicast.==

> [!question] Q2: A workstation is moved to a new floor. It gets an IP, but the user cannot ping the local gateway. How to troubleshoot?
> **Answer:** Verify MAC learning. Check the local ARP table (`arp -a`) for the gateway's MAC. If missing or `00-00-00-00-00-00`, there's a Layer 2 failure. Check if the switch port is assigned to the incorrect VLAN.

> [!question] Q3: What are runt and giant frames in Ethernet networks?
> **Answer:** **Runt frames** are < 64 bytes (excluding preamble/SFD), usually caused by collisions on half-duplex links. **Giant frames** (jumbo) exceed the standard 1518 bytes, typically caused by MTU mismatch configurations.

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-01 Networking Fundamentals|N-01 Networking Fundamentals]] — Overview of LAN classifications and the OSI model layers.
- [[01-Foundations/02-Networking/N-02 Network Devices Deep Dive|N-02 Network Devices Deep Dive]] — Details on switch MAC tables and packet forwarding.
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Information on IP subnets and gateway routing.
