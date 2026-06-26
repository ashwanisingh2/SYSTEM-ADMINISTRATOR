---
tags: [sysadmin, networking, ipv6, protocols]
aliases: [N-05, IPv6]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#intermediate` `#ccna`

# N-05: IPv6 Complete Guide

> [!abstract] Overview
> Yeh note IPv6 ki architecture, address formats, aur routing protocols cover karta hai. Ek support engineer ke liye yeh jaanna zaroori hai kyunki modern networks aur cloud environments ab IPv4 se IPv6 par migrate ho rahe hain, aur troubleshooting (jaise NDP aur MTU issues) aana chahiye.

---
## 🧠 Concept Overview

- **What it is** — IPv6 128-bit ka ek naya IP addressing system hai jo hexadecimal format use karta hai aur trillions of unique addresses provide karta hai.
- **Why it matters** — IPv4 addresses exhaust ho gaye the. IPv6 se har device ko direct, globally unique address milta hai bina NAT ke.
- **Where you see this** — Modern enterprise networks, cloud computing (Azure/AWS), aur dual-stack internet connections mein.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check karna ki client ke paas valid IPv6 aur Link-Local address (fe80::) hai ya nahi. |
| **L2** | Configure, fix, escalate kab karta hai — Dual-stack issues, PMTUD failures resolve karna, aur IPv6 pings test karna. |
| **L3** | Architecture, design, enterprise-level — OSPFv3/BGP routing setup karna, IPv6 migration strategy (Tunneling/NAT64) design karna. |

> [!tip] Seedha Simple Mein
> *IPv4 addresses khatam ho rahe the, isliye IPv6 ko laya gaya. IPv6 128-bit ka hexadecimal address hota hai jo trillions of unique addresses provide karta hai. Isme broadcast frames nahi hote aur security by default built-in hoti hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **IPv6** is like **a new 15-digit global phone number system** because...
>
> - Purane 10-digit system (IPv4) mein numbers khatam ho gaye the, toh humein extensions (NAT) use karne padte the.
> - Naye system mein itne numbers hain ki dharti ki har mitti ke kan ko apna unique number mil sakta hai, aur har koi seedhe direct dial kar sakta hai.

---
## 🔬 Technical Deep Dive

### 1. IPv6 Address Format

> [!info] Key Concept
> An IPv6 address is 128 bits long (4 times longer than IPv4), written in hexadecimal, divided into 8 groups of 16 bits (hextets) separated by colons. Example: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`

To make IPv6 addresses easier to write, apply two rules:
- **Rule 1: Drop Leading Zeros:** Within any 16-bit hextet, leading zeros can be omitted. `0db8` becomes `db8`.
- **Rule 2: Compress Consecutive Zeros:** Double colons (`::`) can replace a single, contiguous run of one or more all-zero hextets.

> [!danger] Common Mistake
> You can only use the double colon (`::`) **once** in any IPv6 address to avoid ambiguity when parsing address length. Writing `2001::db8::1` is invalid.

### 2. Important Address Scopes
- **Global Unicast Address (GUA):** Equivalent to public IPv4. Prefix `2000::/3`.
- **Link-Local Address (LLA):** Non-routable on internet, auto-generated. Prefix `fe80::/10`.
- **Unique Local Address (ULA):** Equivalent to private IPv4. Prefix `fc00::/7` (starts with `fd`).
- **Loopback:** `::1`.
- **Unspecified:** `::`.

### 3. IPv4 vs IPv6
IPv6 has 128 bits vs IPv4's 32 bits. IPv6 uses a fixed 40-byte header, doesn't require NAT, and replaces ARP broadcasts with ICMPv6 Neighbor Discovery Protocol (NDP).

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows client workstation and command prompt access.

### Step 1: Check Local IPv6 Configurations

```cmd
# Query your adapter configuration
ipconfig
```

> [!success] Expected Output
> ```
> Link-local IPv6 Address . . . . . : fe80::a1b2:c3d4:e5f6:g7h8%12
> IPv6 Address. . . . . . . . . . . : 2001:db8::1
> ```

### Step 2: Perform an IPv6 Ping Test

```cmd
# Ping your local loopback address
ping ::1
```

> [!success] Expected Output
> ```
> Reply from ::1: time<1ms
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `ipconfig` | Shows IPv6 and IPv4 addresses | `ipconfig /all` |
| `ping ::1` | IPv6 loopback test | `ping ::1` |
| `ip -6 addr show` | View active IPv6 configurations (Linux) | `ip -6 addr show` |
| `ping6` | Ping using IPv6 (Linux) | `ping6 -c 4 google.com` |
| `Get-NetIPAddress -AddressFamily IPv6` | View active IPv6 (PowerShell) | `Get-NetIPAddress -AddressFamily IPv6` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Website resolves but won't load | PMTUD failure. Firewall blocks ICMPv6 "Packet Too Big". | Allow ICMPv6 in firewall, or lower interface MTU manually using `netsh`. |
| 5-second delay before loading web pages | Dual-stack preference issue. Broken IPv6 route. | Check `tracert -6`. Fix IPv6 route or temporarily prefer IPv4 via prefix policy. |
| Can't ping local link-local gateway | Link-local ping requires interface zone index. | Use `ping fe80::1%12` (Windows) or `ping6 -I eth0 fe80::1` (Linux). |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Website Loading Delay on Dual Stack

> [!example] Ticket
> "Users are experiencing a 5-6 second delay when opening external websites like Google or Office 365. After the delay, pages load normally."

**L1 Response:** Check basic connectivity. Try pinging google.com and notice it resolves to an IPv6 address but times out before falling back to IPv4.
**Escalation Trigger:** Agar client PC pe direct IPv6 gateway reach nahi ho raha aur enterprise-wide issue lag raha hai.
**L2 Resolution:** Run `tracert -6 google.com` to confirm broken IPv6 route. Temporarily adjust prefix policy on clients using `netsh interface ipv6 set prefixpolicy ::ffff:0:0/96 46 4` to prefer IPv4 until network team fixes the IPv6 gateway.

---
## 🎤 Interview Questions

> [!question] Q1: How does SLAAC (Stateless Address Autoconfiguration) work in IPv6?
> **Answer:** SLAAC lets a client configure its own IP. The client sends a Router Solicitation (RS) multicast. The router replies with a Router Advertisement (RA) containing the prefix. The client appends its interface ID (EUI-64 or random) and runs Duplicate Address Detection (DAD).

==**Exam Tip:** SLAAC relies on ICMPv6 NDP (RS/RA messages), not DHCP broadcasts.==

> [!question] Q2: What is the difference between DHCPv6 Stateful and DHCPv6 Stateless modes?
> **Answer:** Stateful DHCPv6 acts like IPv4 DHCP (assigns IPs and tracks leases). Stateless DHCPv6 lets clients use SLAAC for IP assignment but provides additional options like DNS and NTP without tracking client state.

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-01 Networking Fundamentals|N-01 Networking Fundamentals]] — Base IP mapping and comparison to OSI.
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Comparison with legacy subnets and class concepts.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Server configurations for DNS and DHCP.
