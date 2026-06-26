---
tags: [sysadmin, networking, security, acl]
aliases: [N-09]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#advanced` `#ccna`

# N-09: Access Control Lists (ACL)

> [!abstract] Overview
> Yeh note Layer 3/4 security filtering using Access Control Lists (ACLs) cover karta hai. Yeh standard, extended, aur named ACL configurations, placement rules, wildcard masking, aur verification commands detail karta hai. Ek support engineer ko yeh aana chahiye taaki wo network traffic properly control aur troubleshoot kar sake.

---
## 🧠 Concept Overview

- **What it is** — Access Control Lists (ACLs) are a set of rules defined on a router to filter incoming or outgoing network traffic based on parameters like IP addresses, protocols, and port numbers.
- **Why it matters** — They are the fundamental building blocks of network security, preventing unauthorized access and controlling traffic flow to conserve bandwidth.
- **Where you see this** — Configuring firewalls, securing router interfaces, restricting remote management access (SSH/Telnet), and protecting servers in data centers.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check if ACLs are applied to an interface using show commands. |
| **L2** | Configure basic standard/extended ACLs, fix sequence number issues, edit named ACLs. |
| **L3** | Architecture, design enterprise-wide access policies, automate ACL deployment, complex troubleshooting. |

> [!tip] Seedha Simple Mein
> *ACL router par traffic filtering ke liye rules ki list hoti hai. Standard ACL sirf Source IP check karta hai. Extended ACL Source, Destination, Port, aur Protocol sab check karta hai. Har ACL ke aakhri mein ek hidden 'Deny All' rule hota hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **An ACL** is like **a guest list managed by a nightclub bouncer** because...
>
> - **Standard ACL:** "If you are from Sector A, you are not allowed in." The bouncer doesn't care where you are going or what you are wearing.
> - **Extended ACL:** "If you are from Sector A AND you want to go to the VIP Lounge AND you are wearing casual clothes, you are denied."
> - **Implicit Deny:** If your name is not explicitly written on the list, the bouncer denies entry by default.

---
## 🔬 Technical Deep Dive

### 1. ACL Classifications

> [!info] Key Concept
> Standard ACLs filter solely on source IP. Extended ACLs filter on source, destination, protocol, and port. Named ACLs allow descriptive names and easy editing.

- **Standard ACLs (Numbered 1-99 & 1300-1999):**
  - Inspects **Source IP Address** only.
  - Cannot filter based on destination or port numbers.
- **Extended ACLs (Numbered 100-199 & 2000-2699):**
  - Filters based on: Source IP, Destination IP, Protocol type (IP, TCP, UDP, ICMP), and Port numbers.
- **Named ACLs:**
  - Can be either Standard or Extended.
  - Allows descriptive alphanumeric naming (e.g., `SECURE_WEB_ONLY`) instead of numbers.
  - **Key Advantage:** Individual lines can be inserted or deleted using sequence numbers without destroying and recreating the entire list.

### 2. Wildcard Masks
Wildcard masks are used in OSPF and ACLs to define IP match boundaries.
- **Subnet Mask vs. Wildcard Mask:** A subnet mask uses binary `1`s to represent match bits and `0`s for ignore. A wildcard mask is the exact opposite: binary `0` means **Match** and `1` means **Ignore**.
- **Calculation Formula:** $\text{Wildcard Mask} = 255.255.255.255 - \text{Subnet Mask}$.
  - *Example 1:* For Subnet `255.255.255.0` (/24):
    $255.255.255.255 - 255.255.255.0 = \mathbf{0.0.0.255}$ (Match first three octets, ignore the last).

### 3. ACL Placement Rules
- **Rule for Standard ACLs:** Place **as close to the destination as possible**. Because standard ACLs only check the source IP, placing it close to the source would block the sender from accessing *all* networks connected to the router, rather than just the target network.
- **Rule for Extended ACLs:** Place **as close to the source as possible**. Because extended ACLs have specific destination and port filters, placing it close to the source prevents unwanted traffic from traversing the network, saving link bandwidth.

### 4. Processing Order & The Implicit Deny
- ACLs are processed sequentially from top to bottom.
- When a packet matches a rule, the permit or deny action is taken, and processing stops. No subsequent lines are read.

> [!danger] Common Mistake
> **Applying an ACL in the wrong direction:** Applying an inbound ACL on an interface outbound (or vice-versa). For example, applying `ip access-group 10 in` on an interface where traffic is leaving the router. The filter will never inspect the packets, allowing unauthorized traffic to pass.
> **Correct approach:** Map the traffic flow direction from the router's perspective. Inbound is traffic entering the router; outbound is traffic exiting the router interface.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Cisco Packet Tracer with 1x 1941 Router, 1x Switch, and 3x PCs.
> - Topology Setup: PC0 (`192.168.1.10`) & PC1 (`192.168.1.20`) on Switch (Gi0/1) connected to Router Gi0/0 (`192.168.1.1`). Web Server PC2 (`10.0.0.50`) connected to Router Gi0/1 (`10.0.0.1`).

### Step 1: Configure Extended ACL to Block PC0 but Permit PC1

```bash
# Enter config mode and create Extended ACL
R1(config)# ip access-list extended FILTER_ICMP
# Deny PC0 from pinging Web Server
R1(config-ext-nacl)# 10 deny icmp host 192.168.1.10 host 10.0.0.50
# Permit all other IP traffic (overrides Implicit Deny)
R1(config-ext-nacl)# 20 permit ip any any
R1(config-ext-nacl)# exit
```

### Step 2: Apply the ACL to Interface

```bash
# Apply inbound to the interface closest to the source (Gi0/0)
R1(config)# interface gi0/0
R1(config-if)# ip access-group FILTER_ICMP in
```

> [!success] Expected Output
> ```
> Ping from PC0 (192.168.1.10) to Server (10.0.0.50) returns Destination Host Unreachable.
> Ping from PC1 (192.168.1.20) to Server (10.0.0.50) succeeds.
> R1# show ip access-lists shows incremented matches on the deny statement.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `access-list [num] [permit/deny] [source]` | Creates a numbered standard ACL | `access-list 10 deny host 192.168.1.5` |
| `ip access-list extended [NAME]` | Creates a named extended ACL | `ip access-list extended SECURE_ACCESS` |
| `ip access-group [num/name] [in/out]` | Applies an ACL to an interface | `ip access-group 10 out` |
| `show ip access-lists` | View all configured ACLs and match counters | `show ip access-lists` |
| `show ip interface [int]` | Check which access-list is active on the interface | `show ip interface gi0/0` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Users lost all connection after new ACL | The ACL lacks a permit statement, triggering the default invisible `deny any` at the bottom. | Append a `permit ip any any` statement to the bottom of the list. |
| Specific permit rule not working, traffic dropped | Incorrect order of rules. A broad deny rule was placed *above* the specific permit rule. | Delete the offending deny rule, insert it below the permit rule using sequence numbers. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Network Access Blocked

> [!example] Ticket
> "I can't access the internet or local fileservers after the security team updated the firewall rules."

**L1 Response:** Verify connection, run `show ip interface` to identify applied ACLs, and use `show ip access-lists` to check drop counters.
**Escalation Trigger:** The ACL is configured but there is no explicit permit statement allowing regular user traffic, and it's hitting the implicit deny.
**L2 Resolution:** Review the ACL requirements, and add the necessary permit statement (e.g., `permit ip 192.168.1.0 0.0.0.255 any`) before the implicit deny.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between a Standard ACL and an Extended ACL?
> **Answer:** A **Standard ACL** only inspects the source IP address of packets. It is configured using numbers 1-99. An **Extended ACL** inspects the source IP, destination IP, protocol (IP, TCP, UDP, ICMP), and source/destination port numbers. It is configured using numbers 100-199 and allows fine-grained security filtering.

==**Exam Tip:** Standard ACLs are placed as close to the destination as possible, whereas Extended ACLs are placed as close to the source as possible.==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Subnets and wildcard calculations.
- [[01-Foundations/02-Networking/N-07 Routing — Static and Dynamic|N-07 Routing — Static and Dynamic]] — Routing configuration filters.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Interface security helpers.
