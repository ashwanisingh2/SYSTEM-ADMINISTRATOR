---
tags: [sysadmin, networking, ip, subnetting]
difficulty: Intermediate
lab-required: Yes
read-time: 18 mins
---

# N-04: IPv4 Addressing Complete Guide

> [!abstract] Overview
> This note covers the IPv4 addressing scheme, including address classes, RFC 1918 private subnets, CIDR notation, and subnetting formulas. It provides detailed walk-throughs of subnet design, Variable Length Subnet Masking (VLSM), and a practice lab set.

---
## Concept
Think of an IPv4 address like a physical mail address: `Street Name, House Number`. 
A subnet mask acts as a boundary marker. If your address is `192.168.1.50` with a subnet mask of `255.255.255.0`, the mask says: "The first three segments (`192.168.1`) represent your Street Name (Network ID), and the last segment (`50`) is your specific House Number (Host ID)." 

If you want to send mail to someone on your own street (`192.168.1.75`), you walk it over yourself (local L2 switch delivery). If you want to send mail to a different street (`10.0.0.50`), you must drop it in the mailbox for the postman (Default Gateway/Router) to deliver.

*Seedha simple mein: IPv4 address 32-bit ka address hota hai jo system ko identify karta hai. Subnet mask yeh tay karta hai ki address ka kitna hissa network (common area) ka hai aur kitna host (personal system) ka hai. Subnetting ke zariye hum ek bade network ko chote aur secure networks mein baant te hain.*

---
## Technical Deep Dive

### 1. IPv4 Structure & Binary Conversion
An IPv4 address is 32 bits long, divided into 4 segments of 8 bits each (called **octets**), separated by dots (dotted-decimal notation).
- **Binary Conversion Grid:**
  ```
  Bit Position:  1   2   3   4   5   6   7   8
  Bit Value:   128  64  32  16   8   4   2   1
  ```
- **Example:** Convert decimal `192` to binary.
  - $192 \ge 128 \implies \text{Yes (1)}$. Remainder $192-128 = 64$.
  - $64 \ge 64 \implies \text{Yes (1)}$. Remainder $0$.
  - All remaining bits are $0$.
  - Result: `11000000`.

### 2. IPv4 Classes & Default Masks
Classful routing divided the address space into five classes based on the leading bits of the first octet.

| Class | First Octet Range | Default Mask | CIDR | Purpose / Use Case |
|---|---|---|---|---|
| **Class A** | 1 – 126 | 255.0.0.0 | /8 | Very large organizations (16M+ hosts per net) |
| **Class B** | 128 – 191 | 255.255.0.0 | /16 | Medium-to-large organizations (65k hosts) |
| **Class C** | 192 – 223 | 255.255.255.0 | /24 | Small local networks (254 hosts) |
| **Class D** | 224 – 239 | N/A | N/A | Multicast groups (no host assignment) |
| **Class E** | 240 – 254 | N/A | N/A | Experimental / Research purposes |

*Note: The range `127` is reserved for loopback diagnostics (e.g., `127.0.0.1`).*

### 3. Public vs. Private IP Addresses (RFC 1918)
Private IP addresses are non-routable on the public Internet. They are used internally within private LANs.

- **Class A Private Range:** `10.0.0.0` to `10.255.255.255` (1 Class A network)
- **Class B Private Range:** `172.16.0.0` to `172.31.255.255` (16 contiguous Class B networks)
- **Class C Private Range:** `192.168.0.0` to `192.168.255.255` (256 contiguous Class C networks)

### 4. Classless Inter-Domain Routing (CIDR)
CIDR replaced classful addressing. It appends a slash (`/`) followed by the number of bits set to `1` in the subnet mask.
- `/8` = `11111111.00000000.00000000.00000000` = `255.0.0.0`
- `/16` = `255.255.0.0`
- `/24` = `255.255.255.0`
- `/26` = `11111111.11111111.11111111.11000000` = `255.255.255.192`

---
## Subnetting Step by Step

### Subnetting Formulas
- **Number of Subnets created:** $2^s$, where $s$ is the number of borrowed host bits.
- **Number of Hosts per subnet:** $2^h - 2$, where $h$ is the number of remaining host bits. 
  - *Note: We subtract 2 because the first address is the Network ID, and the last is the Broadcast ID.*
- **Block Size (Magic Number):** $256 - \text{Decimal value of the subnetted octet}$.

---

### Example 1: Solve 192.168.1.0/26
1. **Identify Parent Mask:** Standard Class C is `/24`. The target is `/26`.
2. **Bits Borrowed:** $26 - 24 = 2$ bits.
3. **Host Bits Remaining:** $32 - 26 = 6$ bits.
4. **Calculations:**
   - Subnets created: $2^2 = 4$ subnets.
   - Hosts per subnet: $2^6 - 2 = 64 - 2 = 62$ usable hosts.
   - Block Size: $256 - 192 = 64$. (Note: `/26` mask is `255.255.255.192`).
5. **Subnet Blocks:**
   - **Subnet 1:** Network: `192.168.1.0` | Usable: `192.168.1.1` to `.62` | Broadcast: `192.168.1.63`
   - **Subnet 2:** Network: `192.168.1.64` | Usable: `192.168.1.65` to `.126` | Broadcast: `192.168.1.127`
   - **Subnet 3:** Network: `192.168.1.128` | Usable: `192.168.1.129` to `.190` | Broadcast: `192.168.1.191`
   - **Subnet 4:** Network: `192.168.1.192` | Usable: `192.168.1.193` to `.254` | Broadcast: `192.168.1.255`

---

### Example 2: How many hosts in 172.16.0.0/20?
1. **Calculate Host Bits:** $32 - 20 = 12$ host bits.
2. **Apply Formula:** Usable Hosts = $2^{12} - 2 = 4096 - 2 = 4094$ usable hosts.
3. **Mask Representation:** `255.255.240.0`.
4. **Block Size in 3rd Octet:** $256 - 240 = 16$. Networks increment by 16 in the 3rd octet (e.g., `172.16.0.0`, `172.16.16.0`, `172.16.32.0`).

---

### Example 3: Create 5 Subnets from 192.168.10.0/24
1. **Determine bits to borrow ($s$):** We need 5 subnets.
   - If $s=2 \implies 2^2 = 4$ subnets (Too few).
   - If $s=3 \implies 2^3 = 8$ subnets (Sufficient).
2. **New Subnet Mask:** `/24` $+ 3$ bits $= /27$ (Mask: `255.255.255.224`).
3. **Calculations:**
   - Remaining host bits: $32 - 27 = 5$ bits.
   - Usable hosts per subnet: $2^5 - 2 = 30$ hosts.
   - Block Size: $256 - 224 = 32$.
4. **First 5 Subnets:**
   - Subnet 0: `192.168.10.0/27`
   - Subnet 1: `192.168.10.32/27`
   - Subnet 2: `192.168.10.64/27`
   - Subnet 3: `192.168.10.96/27`
   - Subnet 4: `192.168.10.128/27`

---

### Variable Length Subnet Masking (VLSM)
VLSM allows subnets of different sizes to be carved out of the same parent block, preventing IP address waste.
- **Golden Rule:** Always design subnets starting with the **largest host requirement** first down to the smallest.
- **Example Scenario:** Parent block `192.168.1.0/24`. Allocate for:
  - Sales: 100 hosts
  - IT: 50 hosts
  - HR: 20 hosts
  - WAN links (Routers): 2 hosts
- **Solution Process:**
  1. **Sales (100 hosts):** Needs next power of 2 ($2^7 = 128 \ge 100 + 2$). Borrow 1 bit from `/24` $\implies /25$.
     - Block: `192.168.1.0/25` (IPs: `.0` to `.127`).
  2. **IT (50 hosts):** Needs next power of 2 ($2^6 = 64 \ge 50 + 2$). Start next block at `.128`. Borrow 2 bits $\implies /26$.
     - Block: `192.168.1.128/26` (IPs: `.128` to `.191`).
  3. **HR (20 hosts):** Needs next power of 2 ($2^5 = 32 \ge 20 + 2$). Start next block at `.192`. Borrow 3 bits $\implies /27$.
     - Block: `192.168.1.192/27` (IPs: `.192` to `.223`).
  4. **WAN Links (2 hosts):** Needs next power of 2 ($2^2 = 4 \ge 2 + 2$). Start next block at `.224`. Borrow 6 bits $\implies /30$.
     - Block: `192.168.1.224/30` (IPs: `.224` to `.227`).

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A workstation with command-line or notepad access to check calculations.

### Practice Problems (Solve before checking answers below)
1. What is the network ID for `192.168.1.155/27`?
2. How many usable hosts in a `/29` network?
3. What is the subnet mask of `/22`?
4. Find the broadcast address of `10.5.16.0/20`.
5. What is the CIDR prefix for mask `255.255.255.240`?
6. Is `172.16.35.40` a private IP under RFC 1918?
7. What is the usable IP range of `192.168.50.96/28`?
8. Split `192.168.0.0/24` into four equal subnets. What is the mask and block size?
9. Can a host with IP `192.168.1.62/26` ping `192.168.1.65/26` directly without a router?
10. What is the binary representation of `172`?

---

### Answer Sheet & Verification
1. **Network ID:** `192.168.1.128` (Blocks of 32: 0, 32, 64, 96, 128, 160. 155 falls in the 128 block).
2. **Hosts:** $2^3 - 2 = 6$ usable hosts ($32-29 = 3$ host bits).
3. **Subnet Mask:** `255.255.252.0` ($32-22 = 10$ host bits. $256-4 = 252$ in 3rd octet).
4. **Broadcast ID:** `10.5.31.255` (Block size in 3rd octet is 16. Range: `10.5.16.0` to `10.5.31.255`).
5. **CIDR:** `/28` ($24$ bits $+ 4$ bits for $240$).
6. **Private status:** Yes. Class B private range covers `172.16.0.0` to `172.31.255.255`.
7. **Usable range:** `192.168.50.97` to `192.168.50.110` (Network: `.96`, Broadcast: `.111`).
8. **Equal splitting:** `/26` (borrowed 2 bits). Block size: 64.
9. **Direct Ping:** No. `192.168.1.62` is in Subnet 0 (`.0` to `.63`). `192.168.1.65` is in Subnet 1 (`.64` to `.127`). They are in different broadcast domains and require a router.
10. **Binary 172:** `10101100` ($128 + 32 + 8 + 4 = 172$).

---
## Commands Reference
Advanced content only — basics in [[Basic networking commands]]

View routing tables and interface scopes to analyze network boundaries:

```bash
# Windows (PowerShell)
# Check local IP interfaces, CIDR lengths, and Gateway assignments
Get-NetIPConfiguration | Select-Object InterfaceAlias, IPv4Address, IPv4DefaultGateway

# Linux (Bash)
# View routing table to see default gateways and local subnet masks
ip route show
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** A workstation user cannot connect to local servers or the Internet. Running `ipconfig` shows the IP address is `169.254.24.150` with subnet mask `255.255.0.0`.
- **Root Cause:** **APIPA (Automatic Private IP Addressing)**. The workstation failed to reach the DHCP server, and Windows auto-assigned a temporary non-routable link-local address.
- **Fix:**
  1. Check physical cabling (L1 check).
  2. Verify that the DHCP Server service is running on the local subnet.
  3. Run `ipconfig /release` followed by `ipconfig /renew`.
  4. If the APIPA address persists, verify that the switch port is configured in the correct client VLAN with access to the DHCP helper address.

**Scenario 2:**
- **Problem:** You configure a new server with IP `192.168.10.65`, Subnet Mask `255.255.255.192`, Gateway `192.168.10.1`. The server cannot ping the gateway or communicate with any device.
- **Root Cause:** Gateway is outside the host's subnet boundary. Mask `255.255.255.192` (/26) splits the network at `.64`. The host (`.65`) belongs to the second subnet block (`.64` to `.127`), but the gateway (`.1`) belongs to the first subnet block (`.0` to `.63`).
- **Fix:**
  1. Modify the gateway IP configuration on the host to target a gateway located in its own subnet (e.g., `192.168.10.66`).
  2. Alternatively, change the host's subnet mask to `255.255.255.0` (/24) if all devices belong to a single flat subnet.

---
## Common Mistakes
> [!warning] Avoid These
> **Using the network or broadcast address for hosts:** Assigning the first address (Network ID, e.g., `192.168.1.0/24`) or the last address (Broadcast ID, e.g., `192.168.1.255/24`) to a physical workstation adapter. This causes IP validation errors in the OS or IP conflicts on the network.
> **Correct approach:** Only assign IP addresses within the usable host range (e.g., `.1` to `.254` for a `/24`).

---
## Pro Tips
> [!tip] Field Experience
> When designing subnets for remote branch sites, always assign uniform CIDR blocks (e.g., `/24` per site). This allows you to configure simple **Route Summarization** on core routers, reducing routing table sizes and optimizing processor usage.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Usable Hosts | Calculated as $2^h - 2$, removing the network identifier and the broadcast identifier. |
| 2 | RFC 1918 | The standard defining private IP ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`). |
| 3 | APIPA | Link-local range `169.254.0.0/16` auto-assigned by the OS when DHCP server contact fails. |
| 4 | CIDR Notation | Replaced classes; uses a slash prefix (e.g., `/24`) to define the mask bit boundary. |
| 5 | VLSM | Carving subnets of different sizes from one parent pool to prevent host IP wastage. |

---
## Interview Q&A

**Q1: What is the purpose of the Subnet Mask, and how does a router use it to forward packets?**
A: The subnet mask is a 32-bit sequence of consecutive 1s followed by consecutive 0s that defines the boundary between the network ID and the host ID of an IP address. When a router receives a packet, it extracts the destination IP address and performs a bitwise **AND** operation between the destination IP and the subnet masks in its routing table. If the resulting network address matches a route entry, the router knows which interface or next-hop IP to forward the packet.

**Q2: A client demands a network design that supports 5 distinct physical sites. Each site requires at least 25 usable host addresses. You are allocated the block 192.168.5.0/24. Explain your design solution.**
A: 
- **Situation:** Allocate at least 5 subnets with 25 hosts each from a `/24` space.
- **Task:** Determine the correct CIDR prefix and design the subnet layouts.
- **Action:** To support 25 hosts, the next power of 2 is 32 ($2^5$). This requires 5 host bits, leaving 27 bits for the network ($32-5=27$). A `/27` subnet mask supports $2^5 - 2 = 30$ hosts, which meets the 25 hosts requirement. The number of subnets created from a `/24` parent using a `/27` mask is $2^{27-24} = 2^3 = 8$ subnets. This satisfies the 5 sites requirement.
- **Result:** I will assign `/27` subnets incrementing in blocks of 32 (e.g., Site 1: `.0/27`, Site 2: `.32/27`, Site 3: `.64/27`, Site 4: `.96/27`, Site 5: `.128/27`).

**Q3: Explain the difference between classful routing and classless routing protocols.**
A: **Classful routing protocols** (e.g., RIPv1, IGRP) do not include subnet mask information in their routing updates. They assume that all devices use the default subnet mask for Class A, B, or C addresses. They do not support VLSM or CIDR. **Classless routing protocols** (e.g., OSPF, EIGRP, BGP, RIPv2) include the subnet mask prefix length alongside the network address in all routing updates. This permits the use of subnets of varying sizes (VLSM), route summarization, and CIDR.

---
## Related Notes
- [[01-Foundations/02-Networking/N-03 Ethernet and MAC Address|N-03 Ethernet and MAC Address]] — Details on Layer 2 MAC structures and ARP.
- [[01-Foundations/02-Networking/N-05 IPv6 Complete Guide|N-05 IPv6 Complete Guide]] — Next-generation addressing schemes and migrations.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Auto configuration services and address mapping.


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
