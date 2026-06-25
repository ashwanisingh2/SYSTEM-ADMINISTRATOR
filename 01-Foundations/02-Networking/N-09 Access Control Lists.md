---
tags: [sysadmin, networking, security, acl]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# N-09: Access Control Lists (ACL)

> [!abstract] Overview
> This note covers Layer 3/4 security filtering using Access Control Lists (ACLs). It details standard, extended, and named ACL configurations, placement rules, wildcard masking, and verification commands.

---
## Concept
Think of an Access Control List (ACL) as a guest list managed by a nightclub bouncer (the Router) standing at the door. 
- A **Standard ACL** is a simple rule: "If you are from Sector A (Source IP), you are not allowed in." The bouncer doesn't care where you are going or what you are wearing.
- An **Extended ACL** is a detailed rule: "If you are from Sector A (Source IP) AND you want to go to the VIP Lounge (Destination IP) AND you are wearing casual clothes (Port 80/HTTP), you are denied. But if you want to go to the Dance Floor (VLAN 10) wearing formal clothes (Port 443/HTTPS), you are allowed."
- The **Implicit Deny** is the rule that if your name is not explicitly written on the list, the bouncer denies entry by default.

*Seedha simple mein: ACL router par traffic filtering ke liye rules ki list hoti hai. Standard ACL sirf Source IP check karta hai. Extended ACL Source, Destination, Port, aur Protocol sab check karta hai. Har ACL ke aakhri mein ek hidden 'Deny All' rule hota hai.*

---
## Technical Deep Dive

### 1. ACL Classifications

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
  - *Example 2:* For Subnet `255.255.255.224` (/27):
    $255.255.255.255 - 255.255.255.224 = \mathbf{0.0.0.31}$.

### 3. ACL Placement Rules
- **Rule for Standard ACLs:** Place **as close to the destination as possible**. Because standard ACLs only check the source IP, placing it close to the source would block the sender from accessing *all* networks connected to the router, rather than just the target network.
- **Rule for Extended ACLs:** Place **as close to the source as possible**. Because extended ACLs have specific destination and port filters, placing it close to the source prevents unwanted traffic from traversing the network, saving link bandwidth.

### 4. Processing Order & The Implicit Deny
- ACLs are processed sequentially from top to bottom.
- When a packet matches a rule, the permit or deny action is taken, and processing stops. No subsequent lines are read.
- **THE GOLDEN RULE:** At the end of every ACL, there is an invisible, silent **`deny any`** statement. If a packet does not match any permit rules, it is dropped by default. Therefore, every ACL must contain at least one `permit` statement to function.

---
## Cisco IOS Configuration Commands

### Numbered Standard ACL Configuration
Blocks host `192.168.10.5` from accessing the LAN connected to interface `Gi0/1`.
```text
Router(config)# access-list 10 deny host 192.168.10.5           ! (Block single host)
Router(config)# access-list 10 permit any                       ! (Allow all other traffic - overrides Implicit Deny)
Router(config)# interface gigabitethernet 0/1
Router(config-if)# ip access-group 10 out                       ! (Apply outbound to interface)
```

### Numbered Extended ACL Configuration
Permits VLAN 10 (`192.168.10.0/24`) to access Web Server `10.0.0.50` on port 443, but blocks all other traffic.
```text
Router(config)# access-list 101 permit tcp 192.168.10.0 0.0.0.255 host 10.0.0.50 eq 443
Router(config)# access-list 101 deny ip any any                 ! (Explicitly write deny for logging/readability)
Router(config)# interface gigabitethernet 0/0                   ! (Interface closest to VLAN 10 source)
Router(config-if)# ip access-group 101 in                       ! (Apply inbound to interface)
```

### Named Extended ACL Configuration (Editing Lines)
```text
Router(config)# ip access-list extended SECURE_ACCESS
Router(config-ext-nacl)# 10 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.10 eq 80
Router(config-ext-nacl)# 20 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.10 eq 443
Router(config-ext-nacl)# exit
```
- *To insert a line between 10 and 20:*
  ```text
  Router(config)# ip access-list extended SECURE_ACCESS
  Router(config-ext-nacl)# 15 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.10 eq 22
  ```
- *To delete a single line:*
  ```text
  Router(config-ext-nacl)# no 10
  ```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Cisco Packet Tracer with 1x 1941 Router, 1x Switch, and 3x PCs.

### Step 1: Topology Configuration
1. Connect PC0 and PC1 to Switch. Labeled Subnet: `192.168.1.0/24`.
   - PC0: `192.168.1.10`, Gateway: `192.168.1.1`
   - PC1: `192.168.1.20`, Gateway: `192.168.1.1`
2. Connect Switch Gi0/1 to Router Gi0/0.
3. Connect Router Gi0/1 to PC2 (Simulating Web Server).
   - Server PC2: `10.0.0.50`, Gateway: `10.0.0.1`

### Step 2: Configure Router Interfaces
1. Enter CLI on Router:
   ```text
   en
   conf t
   interface gi0/0
   ip address 192.168.1.1 255.255.255.0
   no shut
   interface gi0/1
   ip address 10.0.0.1 255.255.255.0
   no shut
   ```

### Step 3: Configure Extended ACL to Block PC0 but Permit PC1
Goal: Block PC0 from pinging Web Server PC2, but allow PC1 to ping.
1. Create Extended ACL on Router:
   ```text
   R1(config)# ip access-list extended FILTER_ICMP
   R1(config-ext-nacl)# 10 deny icmp host 192.168.1.10 host 10.0.0.50
   R1(config-ext-nacl)# 20 permit ip any any
   R1(config-ext-nacl)# exit
   ```
2. Apply the ACL inbound to the interface closest to the source (Gi0/0):
   ```text
   R1(config)# interface gi0/0
   R1(config-if)# ip access-group FILTER_ICMP in
   ```

### Step 4: Verify
1. Ping from PC0 (`192.168.1.10`) to Server (`10.0.0.50`).
   - **Verify:** The ping should return **Destination Host Unreachable**.
2. Ping from PC1 (`192.168.1.20`) to Server (`10.0.0.50`).
   - **Verify:** The ping should succeed.
3. Run verification command on Router:
   ```text
   R1# show ip access-lists
   ```
   Confirm that the deny match count has increased.

---
## Commands Reference
Advanced content only — basics in [[Basic networking commands]]

Show access rules and active filters:

```text
# Cisco IOS commands
show ip access-lists              ! View all configured ACLs and match counters
show ip interface gi0/0           ! Check which access-list is active (inbound/outbound) on the interface
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** After configuring a new ACL to block security threats, users report they have completely lost connection to local fileservers and the Internet.
- **Root Cause:** The ACL lacks a permit statement, triggering the default invisible `deny any` at the bottom of the list.
- **Fix:**
  1. Inspect the configuration: `show ip access-lists`.
  2. Verify if the list only contains `deny` statements.
  3. Append a permit statement to the bottom of the list:
     ```text
     Router(config)# ip access-list extended [NAME]
     Router(config-ext-nacl)# permit ip any any
     ```

**Scenario 2:**
- **Problem:** An extended ACL is configured to permit HTTP traffic (`port 80`) to a server, but deny all other protocols. When users try to browse to the site, it fails to load, but pinging the server works.
- **Root Cause:** Incorrect order of rules. A broad `deny ip any any` or `deny host` rule was placed *above* the specific `permit tcp port 80` rule. The router hits the deny rule first and discards the packet.
- **Fix:**
  1. Inspect the sequence numbers: `show ip access-lists`.
  2. Locate the sequence order (e.g., Line 10: Deny, Line 20: Permit).
  3. Delete the offending deny rule, insert it below the permit rule:
     ```text
     Router(config)# ip access-list extended FILTER_TRAFFIC
     Router(config-ext-nacl)# no 10
     Router(config-ext-nacl)# 30 deny ip any host 10.0.0.50
     ```

---
## Common Mistakes
> [!warning] Avoid These
> **Applying an ACL in the wrong direction:** Applying an inbound ACL on an interface outbound (or vice-versa). For example, applying `ip access-group 10 in` on an interface where traffic is leaving the router. The filter will never inspect the packets, allowing unauthorized traffic to pass.
> **Correct approach:** Map the traffic flow direction from the router's perspective. Inbound is traffic entering the router; outbound is traffic exiting the router interface.

---
## Pro Tips
> [!tip] Field Experience
> When configuring ACLs on remote production routers over SSH, always configure a reload timer before applying changes (e.g., `reload in 10`). If you make a mistake and lock yourself out by blocking SSH traffic (port 22), the router will automatically reboot to its saved config after 10 minutes, saving you from a physical trip to the site. If the config works, cancel the reload: `reload cancel`.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Standard ACL | Filters source IP only; should be placed as close to the destination as possible. |
| 2 | Extended ACL | Filters source, destination, port, and protocol; place close to the source. |
| 3 | Wildcard Mask | Inverse subnet mask; binary 0 means match, 1 means ignore/don't care. |
| 4 | Implicit Deny | The silent default rule at the bottom of every ACL that drops all non-matched traffic. |
| 5 | Named ACL | Modern ACL style allowing easy line insertion/deletion via sequence numbers. |

---
## Interview Q&A

**Q1: What is the difference between a Standard ACL and an Extended ACL?**
A: A **Standard ACL** only inspects the source IP address of packets. It cannot check the destination network, transport layer protocol (TCP/UDP), or port numbers. It is configured using numbers 1-99. An **Extended ACL** inspects the source IP, destination IP, protocol (IP, TCP, UDP, ICMP), and source/destination port numbers. It is configured using numbers 100-199 and allows fine-grained security filtering (e.g., blocking only SSH but permitting HTTPS).

**Q2: A technician wants to block all hosts in the subnet 192.168.10.0/28 from accessing a specific server. Calculate the Wildcard Mask and write the Cisco command.**
A: 
- **Situation:** A `/28` subnet needs to be blocked from accessing a host.
- **Task:** Calculate the wildcard mask and write the extended ACL command.
- **Action:** First, I convert the `/28` mask to decimal: `255.255.255.240`. Second, I calculate the wildcard mask: $255.255.255.255 - 255.255.255.240 = \mathbf{0.0.0.15}$. Third, I write the extended ACL statement.
- **Result:** The command is: `access-list 101 deny ip 192.168.10.0 0.0.0.15 host [Server_IP]`.

**Q3: Explain how the router processes ACLs and what happens if a packet matches line 15 of a 50-line list.**
A: The router processes ACLs sequentially from top to bottom. It compares the packet headers against the conditions of each line in order of sequence numbers. The moment a packet matches the criteria of a line (e.g., Line 15), the router immediately executes the defined action (permit or deny) and stops processing. It does not read any of the remaining 35 lines. If no match is found across all 50 lines, the packet is discarded by the implicit deny rule.

---
## Related Notes
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Subnets and wildcard calculations.
- [[01-Foundations/02-Networking/N-07 Routing — Static and Dynamic|N-07 Routing — Static and Dynamic]] — Routing configuration filters.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Interface security helpers.
