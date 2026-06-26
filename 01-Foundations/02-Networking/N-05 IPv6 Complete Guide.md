---
tags: [sysadmin, networking, ipv6, protocols]
difficulty: Intermediate
lab-required: Yes
read-time: 12 mins
---

# N-05: IPv6 Complete Guide

> [!abstract] Overview
> This note covers the architecture, address formats, shortening rules, and routing protocols of IPv6. It details neighbor discovery (NDP), address types, migration techniques (Dual Stack, Tunneling), and client-side configuration.

---
## Concept
Think of IPv6 as a legacy 10-digit telephone numbering system. As the population grew and every person got multiple devices, we ran out of numbers (IPv4 exhaustion). 

IPv6 is like switching to a new numbering system where every single grain of sand on Earth can have its own billions of phone numbers. Because we have so many numbers, we no longer need to share one number using complex routing tricks (NAT). Every device gets its own direct, globally unique address, and we use a new, silent neighborhood directory (NDP) to find the MAC addresses of our neighbors instead of shouting out loud broadcasts.

*Seedha simple mein: IPv4 addresses khatam ho rahe the, isliye IPv6 ko laya gaya. IPv6 128-bit ka hexadecimal address hota hai jo trillions of unique addresses provide karta hai. Isme broadcast frames nahi hote aur security by default built-in hoti hai.*

---
## Technical Deep Dive

### 1. IPv6 Address Format
An IPv6 address is 128 bits long (4 times longer than IPv4), written in hexadecimal, divided into 8 groups of 16 bits (hextets) separated by colons.
- **Example:** `2001:0db8:85a3:0000:0000:8a2e:0370:7334`

### 2. Compression Rules
To make IPv6 addresses easier to write, apply two rules:
- **Rule 1: Drop Leading Zeros:** Within any 16-bit hextet, leading zeros can be omitted.
  - `0db8` becomes `db8`
  - `0000` becomes `0`
  - *Result of Rule 1:* `2001:db8:85a3:0:0:8a2e:370:7334`
- **Rule 2: Compress Consecutive Zeros:** Double colons (`::`) can replace a single, contiguous run of one or more all-zero hextets.
  - `:0000:0000:` becomes `::`
  - *Result of Rule 2:* `2001:db8:85a3::8a2e:370:7334`
  - **CRITICAL:** You can only use the double colon (`::`) **once** in any IPv6 address to avoid ambiguity when parsing address length.

### 3. IPv6 Address Types
IPv6 does not use broadcast addresses. Instead, it uses three communication patterns:
- **Unicast:** One-to-one communication (delivers to a single interface).
- **Multicast:** One-to-many communication (delivers to all interfaces subscribed to the multicast group).
- **Anycast:** One-to-nearest communication (delivers to the closest interface sharing the same anycast address, determined by routing tables).

### 4. Important Address Scopes

- **Global Unicast Address (GUA):** Equivalent to public IPv4 addresses. Routable on the Internet.
  - Prefix range: **`2000::/3`** (First three bits are `001`, covering addresses starting with `2` or `3`).
- **Link-Local Address (LLA):** Non-routable on the internet, used only for communication within a single physical link segment. Every interface auto-generates an LLA when initialized.
  - Prefix range: **`fe80::/10`**.
- **Unique Local Address (ULA):** Equivalent to private RFC 1918 IPv4 addresses. Routable only within an organization's LANs.
  - Prefix range: **`fc00::/7`** (typically starts with `fd`).
- **Loopback Address:** Equivalent to `127.0.0.1`.
  - Format: **`::1`** (or `::1/128`).
- **Unspecified Address:** Used when a host does not have an IP yet.
  - Format: **`::`** (or `::/128`).

### 5. IPv4 vs. IPv6 Comparison

| Feature | IPv4 | IPv6 |
|---|---|---|
| **Address Length** | 32 Bits (Dotted Decimal) | 128 Bits (Hexadecimal) |
| **Header Size** | Variable (20 to 60 Bytes) | Fixed (40 Bytes - faster router processing) |
| **Address Count** | ~4.3 Billion ($2^{32}$) | ~340 Undecillion ($2^{128}$) |
| **NAT** | Required (PAT/NAT) | Not required (direct end-to-end routing) |
| **Addressing Method** | DHCP / Manual | SLAAC / DHCPv6 / Manual |
| **ARP / Broadcast** | ARP Broadcast (L2) | ICMPv6 NDP Multicast |

### 6. ICMPv6 and Neighbor Discovery Protocol (NDP)
NDP operates at Layer 3 using ICMPv6 messages to replace ARP and ICMP redirect functions.
- **Neighbor Solicitation (NS):** Sent by a host to learn the MAC address of a neighbor (corresponds to ARP Request). Uses a solicited-node multicast address instead of a broadcast.
- **Neighbor Advertisement (NA):** Sent in response to an NS (corresponds to ARP Reply).
- **Router Solicitation (RS):** Sent by a host to locate routers on the link.
- **Router Advertisement (RA):** Sent by routers periodically or in response to an RS. Contains prefix information, MTU, and autoconfiguration flags.

### 7. IPv6 Transition Mechanisms
- **Dual Stack:** Routers and hosts run both IPv4 and IPv6 protocols simultaneously. The application selects the stack based on DNS response (A vs. AAAA record).
- **Tunneling:** Encapsulating an IPv6 packet inside an IPv4 packet to transit across a legacy IPv4-only network (e.g., 6to4, Teredo).
- **Translation (NAT64):** Translates IPv6 packets into IPv4 packets using a translation gateway, allowing IPv6-only hosts to talk to IPv4-only servers.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows client workstation and command prompt access.

### Step 1: Check Local IPv6 Configurations
1. Open Command Prompt.
2. Query your adapter configuration:
   ```cmd
   ipconfig
   ```
3. Locate your primary adapter. Find:
   - The **Link-local IPv6 Address** (starts with `fe80::`).
   - The **IPv6 Address** (if your ISP/router has delegated a GUA prefix starting with `2001::` or `2400::`).

### Step 2: Shorten an IPv6 Address
1. Practice compression on this address: `2001:0000:0000:000a:0000:0000:0000:0001`
   - Apply Rule 1 (drop leading zeros): `2001:0:0:a:0:0:0:1`
   - Apply Rule 2 (compress largest zero run): `2001:0:0:a::1` (Note: the three consecutive zeros on the right were compressed; the two on the left remain as `:0:0:` to ensure `::` is only used once).

### Step 3: Perform an IPv6 Ping Test
1. Ping your local loopback address:
   ```cmd
   ping ::1
   ```
2. Ping your local gateway's link-local address. Note that LLAs require you to specify the interface zone index (e.g., `%12` in Windows) because the address scope is link-local:
   ```cmd
   # Windows (where %12 is your interface index from ipconfig)
   ping fe80::1%12
   
   # Linux (where eth0 is your interface name)
   ping6 -I eth0 fe80::1
   ```
3. Verify that the ping replies successfully.

---
## Commands Reference
Advanced content only — basics in [[Basic networking commands]]

Configure and inspect IPv6 parameters on local interfaces:

```bash
# Windows (PowerShell)
Get-NetIPAddress -AddressFamily IPv4, IPv6 | Format-Table IPAddress, InterfaceAlias, AddressState  # View active IPs
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "2001:db8::10" -PrefixLength 64              # Assign static GUA

# Linux (Bash)
ip -6 addr show                       # View active IPv6 configurations on all interfaces
ip -6 route show                      # View IPv6 routing table
ping6 -c 4 google.com                 # Ping host using IPv6 pathing
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** A workstation can resolve websites (DNS responds) but cannot load any pages. The client has an active IPv6 configuration.
- **Root Cause:** A Path MTU Discovery (PMTUD) failure. IPv6 routers do not fragment packets; they discard packets exceeding MTU and send an ICMPv6 "Packet Too Big" message back. If firewalls block ICMPv6, the client never learns to shrink its packets, leading to connection timeouts.
- **Fix:**
  1. Verify connection status using `ping6 google.com`.
  2. Modify local firewall rules to allow all ICMPv6 traffic (specifically Type 2 Packet Too Big and Type 129/130 Solicitations).
  3. Alternatively, lower the interface MTU manually:
     ```powershell
     netsh interface ipv6 set subinterface "Ethernet" mtu=1400 store=persistent
     ```

**Scenario 2:**
- **Problem:** Dual-stacked clients experience a 5-second delay before opening external web pages.
- **Root Cause:** The DNS server resolves both A (IPv4) and AAAA (IPv6) records. The OS attempts to connect via IPv6 first (standard preference), timeouts due to a broken IPv6 WAN gateway route, and then falls back to IPv4.
- **Fix:**
  1. Run `tracert -6 google.com` to see where the IPv6 path breaks.
  2. If the IPv6 gateway route is offline, repair the router's IPv6 WAN configuration.
  3. If a quick fix is needed, adjust the Windows prefix policy table to prefer IPv4 over IPv6 temporarily:
     ```cmd
     netsh interface ipv6 set prefixpolicy ::ffff:0:0/96 46 4
     ```

---
## Common Mistakes
> [!warning] Avoid These
> **Using double colons (::) twice in an address:** Writing `2001::db8::1`. The parser cannot determine if this represents `2001:0000:db8:0000:0000:0000:0000:0001` or `2001:0000:0000:0000:0000:db8:0000:0001`.
> **Correct approach:** Only use `::` once. Leave other zero blocks as a single `0` (e.g., `2001:0:db8::1`).

---
## Pro Tips
> [!tip] Field Experience
> When configuring SLAAC (Stateless Address Autoconfiguration) on routers, always ensure the prefix length is exactly `/64`. The standard IPv6 autoconfiguration mechanism (which uses EUI-64 or randomized interface IDs) requires `/64` blocks; using any other size (like `/60`) will prevent clients from auto-generating addresses.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | 128 Bits | The length of an IPv6 address, written as 8 hextets of hexadecimal characters. |
| 2 | Double Colon | Used once to compress consecutive groups of zeros to shorten address notation. |
| 3 | Link-Local | Address starting with `fe80::` used for communication inside a single physical link segment. |
| 4 | NDP | Neighbor Discovery Protocol; uses ICMPv6 to replace L2 broadcast-based ARP. |
| 5 | SLAAC | Stateless address autoconfiguration; client configures its own IP using Router Advertisements. |

---
## Interview Q&A

**Q1: How does SLAAC (Stateless Address Autoconfiguration) work in IPv6?**
A: SLAAC allows a client host to configure its own IPv6 address and default gateway without a stateful DHCP server. 
1. The client sends a **Router Solicitation (RS)** multicast message to all-routers.
2. The local router responds with a **Router Advertisement (RA)** containing the local IPv6 prefix (e.g., `2001:db8:1::/64`) and gateway address.
3. The client takes this prefix and appends its own interface identifier (using EUI-64 based on its MAC address, or a randomized interface ID).
4. The client executes **Duplicate Address Detection (DAD)** using NDP to ensure no other device shares the address before finalizing configuration.

**Q2: A client is deploying IPv6. They need to map their MAC address (00:11:22:AA:BB:CC) to an EUI-64 Interface ID. Explain how this is calculated.**
A: 
- **Situation:** A host needs an EUI-64 identifier derived from its 48-bit MAC address.
- **Task:** Split the MAC, insert the helper hex string, and flip the 7th bit.
- **Action:** First, I split the MAC address in the middle: `0011:22` and `AABB:CC`. Second, I insert the hexadecimal string `FFFE` in the middle, creating a 64-bit value: `0011:22FF:FEAA:BBCC`. Third, I locate the 7th bit of the first byte (the Universal/Local bit) and invert it. The first byte `00` in binary is `00000000`. Inverting the 7th bit makes it `00000010` which is `02` in hex.
- **Result:** The final EUI-64 Interface ID is `0211:22FF:FEAA:BBCC`.

**Q3: What is the difference between DHCPv6 Stateful and DHCPv6 Stateless modes?**
A: In **Stateful DHCPv6**, the DHCP server acts similarly to an IPv4 DHCP server: it assigns IPv6 addresses to clients, manages leases, and records state information tracking which client has which IP. In **Stateless DHCPv6**, the client uses SLAAC to generate its own IPv6 address, but queries the DHCPv6 server "statelessly" to obtain options like DNS server IPs, domain search lists, and NTP servers. The server does not track IP assignments or lease states.

---
## Related Notes
- [[01-Foundations/02-Networking/N-01 Networking Fundamentals|N-01 Networking Fundamentals]] — Base IP mapping and comparison to OSI.
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Comparison with legacy subnets and class concepts.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Server configurations for DNS and DHCP.


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
