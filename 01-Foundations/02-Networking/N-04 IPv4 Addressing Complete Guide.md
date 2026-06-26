---
tags: [networking, ip, subnetting]
aliases: [N-04, IPv4 Addressing]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#intermediate` `#ccna`

# N-04: IPv4 Addressing Complete Guide

> [!abstract] Overview
> This note covers the IPv4 addressing scheme, including address classes, RFC 1918 private subnets, CIDR notation, and subnetting formulas. It provides detailed walk-throughs of subnet design, Variable Length Subnet Masking (VLSM), and a practice lab set. Ek support engineer ke liye subnetting aana bohot zaroori hai network design aur troubleshooting ke liye.

---
## 🧠 Concept Overview

- **What it is** — An IPv4 address is a 32-bit address that identifies a system. A subnet mask defines the boundary between the network and the host part.
- **Why it matters** — Proper IP addressing prevents conflicts and allows efficient routing of traffic across different networks.
- **Where you see this** — When configuring new servers, troubleshooting DHCP issues, or assigning static IPs to routers.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Checks IP address and Gateway, identifies if APIPA is assigned. |
| **L2** | Configure, fix, escalate kab karta hai — Designs subnet ranges, sets up DHCP scopes, configures static routing. |
| **L3** | Architecture, design, enterprise-level — BGP, SD-WAN IP planning, Enterprise-wide VLSM architecture. |

> [!tip] Seedha Simple Mein
> *IPv4 address 32-bit ka address hota hai jo system ko identify karta hai. Subnet mask yeh tay karta hai ki address ka kitna hissa network (common area) ka hai aur kitna host (personal system) ka hai. Subnetting ke zariye hum ek bade network ko chote aur secure networks mein baant te hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **An IPv4 Address** is like **a physical mail address (Street Name, House Number)** because...
>
> - The Subnet Mask determines which part is the Street (Network ID) and which part is the House (Host ID).
> - If you send mail to the same street, you walk it over (local L2 switch). If it's a different street, you drop it in the mailbox for the postman (Default Gateway/Router) to deliver.

---
## 🔬 Technical Deep Dive

### 1. IPv4 Structure & Binary Conversion

> [!info] Key Concept
> An IPv4 address is 32 bits long, divided into 4 segments of 8 bits each (called **octets**), separated by dots (dotted-decimal notation).

**Binary Conversion Grid:**
```
Bit Position:  1   2   3   4   5   6   7   8
Bit Value:   128  64  32  16   8   4   2   1
```
- **Example:** Convert decimal `192` to binary: Result is `11000000`.

### 2. IPv4 Classes & Default Masks

| Class | First Octet Range | Default Mask | CIDR | Purpose / Use Case |
|---|---|---|---|---|
| **Class A** | 1 – 126 | 255.0.0.0 | /8 | Very large organizations (16M+ hosts per net) |
| **Class B** | 128 – 191 | 255.255.0.0 | /16 | Medium-to-large organizations (65k hosts) |
| **Class C** | 192 – 223 | 255.255.255.0 | /24 | Small local networks (254 hosts) |
| **Class D** | 224 – 239 | N/A | N/A | Multicast groups (no host assignment) |
| **Class E** | 240 – 254 | N/A | N/A | Experimental / Research purposes |

*Note: The range `127` is reserved for loopback diagnostics (e.g., `127.0.0.1`).*

### 3. Public vs. Private IP Addresses (RFC 1918)

Private IP addresses are non-routable on the public Internet.
- **Class A:** `10.0.0.0` to `10.255.255.255`
- **Class B:** `172.16.0.0` to `172.31.255.255`
- **Class C:** `192.168.0.0` to `192.168.255.255`

### 4. Classless Inter-Domain Routing (CIDR)
CIDR appends a slash (`/`) followed by the number of bits set to `1` in the subnet mask.
- `/8` = `255.0.0.0`
- `/24` = `255.255.255.0`
- `/26` = `255.255.255.192`

### 5. Variable Length Subnet Masking (VLSM)
VLSM allows subnets of different sizes to be carved out of the same parent block, preventing IP address waste.

> [!danger] Common Mistake
> **Using the network or broadcast address for hosts:** Assigning the first address (Network ID, e.g., `192.168.1.0/24`) or the last address (Broadcast ID, e.g., `192.168.1.255/24`) to a physical workstation adapter causes IP conflicts.
> **Correct approach:** Only assign IP addresses within the usable host range.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A workstation with command-line or notepad access to check calculations.

### Step 1: Subnetting Formulas & Example

- **Number of Subnets created:** $2^s$, where $s$ is the number of borrowed host bits.
- **Number of Hosts per subnet:** $2^h - 2$, where $h$ is the number of remaining host bits.
- **Block Size:** $256 - \text{Decimal value of the subnetted octet}$.

```bash
# Example: Solve 192.168.1.0/26
# 1. Target is /26. Bits Borrowed = 2. Host Bits Remaining = 6.
# 2. Subnets created = 4. Usable hosts = 62.
# 3. Block Size = 64.
# Subnet 1: Network: 192.168.1.0 | Usable: .1 to .62 | Broadcast: .63
```

> [!success] Expected Output
> Subnet blocks calculated successfully with valid IP boundaries.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-NetIPConfiguration` | Check local IP interfaces, CIDR lengths, and Gateway (PowerShell) | `Get-NetIPConfiguration` |
| `ip route show` | View routing table to see default gateways and local subnet masks (Linux) | `ip route show` |
| `ipconfig /release` | Releases current DHCP lease (Windows) | `ipconfig /release` |
| `ipconfig /renew` | Renews DHCP lease (Windows) | `ipconfig /renew` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| IP is `169.254.24.150` with mask `255.255.0.0` | **APIPA**: Workstation failed to reach the DHCP server, so Windows auto-assigned a link-local address. | 1. Check cabling. 2. Verify DHCP service. 3. `ipconfig /release` then `/renew`. |
| Server `192.168.10.65/26` cannot ping Gateway `192.168.10.1` | Gateway is outside the host's subnet boundary (`.65` is in second subnet `.64-.127`, gateway `.1` is in `.0-.63`). | Modify gateway IP to match host subnet (e.g., `192.168.10.66`), or use a `/24` mask. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Cannot Connect to Local Servers

> [!example] Ticket
> "User cannot connect to the internet or local file shares. IP shows as 169.254.x.x."

**L1 Response:** Check physical cabling and ask the user to reboot or run `ipconfig /release` and `ipconfig /renew`.
**Escalation Trigger:** If the APIPA address persists and other users on the same switch port / VLAN are also affected.
**L2 Resolution:** Verify that the switch port is configured in the correct client VLAN with access to the DHCP helper address. Check DHCP scope exhaustion on the server.

---
## 🎤 Interview Questions

> [!question] Q1: What is the purpose of the Subnet Mask, and how does a router use it to forward packets?
> **Answer:** The subnet mask defines the boundary between the network ID and the host ID. A router performs a bitwise **AND** operation between the destination IP and the subnet masks in its routing table to determine the network address and forward the packet.

> [!question] Q2: Explain the difference between classful routing and classless routing protocols.
> **Answer:** Classful routing protocols (RIPv1, IGRP) do not include subnet mask info in routing updates. Classless routing protocols (OSPF, EIGRP, BGP) include the subnet mask prefix length, allowing VLSM and CIDR.

==**Exam Tip:** When designing subnets for remote sites, assign uniform CIDR blocks (e.g., /24) to allow for simple Route Summarization on core routers!==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-03 Ethernet and MAC Address|N-03 Ethernet and MAC Address]] — Details on Layer 2 MAC structures and ARP.
- [[01-Foundations/02-Networking/N-05 IPv6 Complete Guide|N-05 IPv6 Complete Guide]] — Next-generation addressing schemes and migrations.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Auto configuration services and address mapping.
