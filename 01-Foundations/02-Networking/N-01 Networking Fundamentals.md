---
tags: [networking, fundamentals, sysadmin]
aliases: [networking-fundamentals]
created: 2026-06-26
status: #complete
difficulty: #beginner
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#beginner` `#ccna`

# N-01: Networking Fundamentals

> [!abstract] Overview
> Yeh note fundamental networking concepts, topologies, protocols, aur architectural models cover karta hai. Ek L1 support engineer ko yeh aana chahiye taaki vo basic connectivity issues diagnose kar sake aur data flow samajh sake.

---
## 🧠 Concept Overview

- **What it is** — The framework that allows two or more computers to share data and resources, ranging from a local room to the global Internet.
- **Why it matters** — Without networking, every computer is an isolated island. Understanding how devices communicate is the foundation of all IT operations.
- **Where you see this** — Connecting your phone to Wi-Fi, accessing a corporate file share, or browsing a website.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check physical cables, ping gateways, run ipconfig/ip a. |
| **L2** | Configure switches/routers, troubleshoot VLANs, fix STP loops. |
| **L3** | Architecture, design enterprise WAN links (SD-WAN), BGP routing. |

> [!tip] Seedha Simple Mein
> *Jab do ya do se zyada computers aapas mein data share karne ke liye judte hain, toh use network kehte hain. Yeh connectivity small scope (ek room) se lekar worldwide (internet) tak ho sakti hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Courier System** is like **Networking Scopes** because...
>
> - **PAN (Personal):** Passing a pen to the colleague sitting right next to you.
> - **LAN (Local):** A department building where everyone uses internal mailboxes to send letters to other floors.
> - **MAN (Metropolitan):** A delivery truck routing mail between different company offices located within the same city.
> - **WAN (Wide):** Sending an international package through customs and multiple shipping hubs to reach another country.

---
## 🔬 Technical Deep Dive

### 1. Network Scope Classifications

> [!info] Key Concept
> Classifying networks based on their physical span.

- **PAN (Personal Area Network):** Centered around a single person (under 10 meters). E.g., Bluetooth headphones.
- **LAN (Local Area Network):** Localized geographic area like an office or home. E.g., Wi-Fi or Ethernet switches.
- **MAN (Metropolitan Area Network):** Covers a city or campus. E.g., municipal fiber loops.
- **WAN (Wide Area Network):** Spans countries or continents, connecting LANs. E.g., The Internet.

### 2. Network Topologies (Physical Layouts)
Topologies define how devices are connected.

- **Star Topology:** Every device connects to a central hub/switch. Easy to scale but central switch is a Single Point of Failure (SPOF).
- **Mesh Topology:** Every device connects to every other device. Highly redundant but expensive.
- **Bus Topology:** All devices connect to a single central cable. Cheap but high collisions and breaks easily.
- **Ring Topology:** Each device connects to two neighbors. No collisions, but one break takes down the ring.

### 3. OSI & TCP/IP Models

The 7-layer OSI model serves as a diagnostic framework for sysadmins to isolate issues systematically.

| OSI Layer | TCP/IP Layer | Associated Protocols | Troubleshooting Focus |
|---|---|---|---|
| 7. Application | **Application** | HTTP, DNS | App logs, API status |
| 6. Presentation | **Application** | SSL/TLS | Decryption, encryption |
| 5. Session | **Application** | RPC, NetBIOS | Connection sync logs |
| 4. Transport | **Transport** | TCP, UDP | Port blocks, TCP reset |
| 3. Network | **Internet** | IP, ICMP | Routing, ping, ip route |
| 2. Data Link | **Network Access**| Ethernet, ARP, VLANs | Switch MAC table, frame |
| 1. Physical | **Network Access**| Cables, Hubs | Link lights, cabling |

**Data Encapsulation:** As data travels down the stack, headers are prepended. Traveling up, they are stripped (Decapsulation).
`[Application Data] -> [TCP Header][Data] -> [IP Header][TCP][Data] -> [Eth Header][IP][TCP][Data][Trailer]`

### 4. Enterprise Networking & Wireless

- **Wireshark Analysis:** Used to inspect packets at a granular level.
- **Wireless (Enterprise):** Issues involve 802.1X Auth (RADIUS/NPS certs), RF interference (overlapping 2.4GHz/5GHz), or sticky clients.
- **SD-WAN:** Decouples control plane from physical hardware to dynamically route traffic over best links (MPLS/Broadband) based on latency and jitter.

> [!danger] Common Mistake
> **Assuming software issues before checking cables:** Troubleshooting a Windows operating system driver configuration or IP address assignment for an hour without looking at the back of the computer to see that the Ethernet cable link light is completely off. Always start with L1!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A workstation with drawing software (or draw.io / Pen and Paper).

### Step 1: Mapping the Home/Office Network Topology

1. Identify all network devices in your physical space:
   - ISP Entry Point (Modem or Fiber ONT).
   - Core Wireless Router.
   - Smart devices, laptops, phones, servers.
2. Determine connection types (Ethernet vs. Wi-Fi).

### Step 2: Draw the Diagram

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

1. **Classify:** This represents a **Star Topology** branching from the central Wireless Router/Switch.
2. **Boundaries:** Label the WAN boundary (ONT to ISP) and LAN boundary (everything behind the internal interface of the Router).

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `ipconfig /all` | Show complete network adapter configuration (Win) | `ipconfig /all` |
| `ping -t 8.8.8.8` | Send continuous ping to check L3 network latency | `ping -t 8.8.8.8` |
| `tracert -d 1.1.1.1`| Trace hops without resolving DNS (Win) | `tracert -d 1.1.1.1` |
| `ip a` | List all network interfaces and IPs (Linux) | `ip a` |
| `ss -tulpn` | List listening ports and associated processes | `ss -tulpn` |
| `tracepath 8.8.8.8` | Trace path and estimate MTU (Linux) | `tracepath 8.8.8.8` |
| `wireshark filter` | Isolate HTTP POST request errors | `http.request.method == "POST" && http.response.code >= 400` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Server Not Found | L3 DNS lookup failure or L7 web service down | Verify L1/L2. Ping 127.0.0.1, then gateway, then 8.8.8.8. Fix DNS to 8.8.8.8. Test port 443. |
| Half ring nodes fail | Broadcast storm from new switch lacking STP | Disconnect link to break loop, enable RSTP on switch, reconnect. |
| 802.1X EAP Auth Failed | RADIUS certificate or identity store issue | Verify cert validation on NPS server and client identity store. |
| Wi-Fi Roaming failure | Sticky client holding onto far AP | Adjust Minimum RSSI settings on APs, ensure 15-20% coverage overlap. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Website Connectivity Issue

> [!example] Ticket
> "User cannot connect to the internal file server \\fileserver, but internet works."

**L1 Response:** Ping the server's IP directly. If it works, run `nslookup fileserver` to check DNS resolution.
**Escalation Trigger:** If IP pings but DNS fails and checking `ipconfig /all` shows wrong DNS server, L1 can fix it. Escalate if DNS server itself is unresponsive.
**L2 Resolution:** Check Active Directory DNS service status and zone records.

### 🎫 Scenario 2: Network Loop Segment Down

> [!example] Ticket
> "Entire floor lost connection after plugging in an unmanaged mini-switch under a desk."

**L1 Response:** Ask user to unplug the new device immediately. Check if connectivity restores.
**Escalation Trigger:** If network remains down due to saturated uplinks.
**L2 Resolution:** Access core switch, identify port with massive broadcast traffic, manually shut it down, enable port security (BPDU Guard) for future.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between TCP and UDP, and when would you use each?
> **Answer:** **TCP** is connection-oriented, reliable, and ensures ordering (3-way handshake). Used for HTTP, SSH. **UDP** is connectionless, fast, but unreliable. Used for VoIP, video streaming.

==**Exam Tip:** TCP guarantees delivery; UDP guarantees speed.==

> [!question] Q2: Describe the TCP 3-Way Handshake process.
> **Answer:** 
> 1. **SYN:** Client requests connection.
> 2. **SYN-ACK:** Server acknowledges and requests back.
> 3. **ACK:** Client acknowledges. Connection established.

==**Exam Tip:** Always state the flags in order: SYN -> SYN-ACK -> ACK.==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-02 Network Devices Deep Dive|N-02 Network Devices Deep Dive]] — Detailed operation of Hubs, Switches, and Routers.
- [[01-Foundations/02-Networking/N-03 Ethernet and MAC Address|N-03 Ethernet and MAC Address]] — Hardware protocols, Cat cabling, and ARP operations.
- [[01-Foundations/02-Networking/N-10 Network Troubleshooting Complete|N-10 Network Troubleshooting Complete]] — Detailed OSI diagnostic commands and tools.
