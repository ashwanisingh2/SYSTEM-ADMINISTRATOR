---
tags: [sysadmin, networking, routing, ospf]
aliases: [N-07]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#advanced` `#ccna`

# N-07: Routing — Static and Dynamic

> [!abstract] Overview
> Yeh note routing engine ke operations, Administrative Distance (AD), static aur default routes, aur OSPF dynamic routing ko cover karta hai. Ek support engineer ke liye yeh jaanna bahut zaroori hai kyunki network connectivity issues aur packet drops ko resolve karne ke liye routing concepts ki samajh hona critical hai.

---
## 🧠 Concept Overview

- **What it is** — Routing ek process hai jisse router packets ko ek network se doosre network tak best path ke through bhejta hai. Static routing manually configure hoti hai, jabki dynamic (OSPF) automatically routes calculate karta hai.
- **Why it matters** — Real job mein agar routing sahi nahi hai, toh branch offices aapas mein ya internet se connect nahi kar payenge (Network down).
- **Where you see this** — Jab aap branch-to-HQ connectivity, internet access issues, ya "Destination Host Unreachable" errors troubleshoot kar rahe hote hain.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic connectivity check karta hai (ping, tracert) aur dekhta hai ki default route present hai ya nahi. |
| **L2** | Static routes configure karta hai, OSPF neighbors check karta hai, aur routing issues fix/escalate karta hai. |
| **L3** | Complex OSPF design, BGP integrations, aur enterprise-level routing architecture design karta hai. |

> [!tip] Seedha Simple Mein
> *Router packages ko ek network se dusre network bhejta hai. Static routing mein admin rasta manually batata hai (0.0.0.0/0 default route internet ke liye). Dynamic routing mein protocols jaise OSPF automatic aur fast routes calculate karte hain updates share karke.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Routing** is like a **global road trip GPS system** because...
>
> - **Routing Table** is the GPS map database containing active roads.
> - **Administrative Distance** is your trust rating for different map apps: if Apple Maps says turn left (AD 120) but your personal experience says go straight (AD 1), you trust your personal experience first.
> - **Static Route** is a manual road closure detour you draw on the map yourself.
> - **OSPF Dynamic Routing** is like having all delivery drivers constantly radioing each other about traffic so everyone knows the fastest possible route.

---
## 🔬 Technical Deep Dive

### 1. The Router Decision-Making Process

> [!info] Key Concept
> Router packet forward karne se pehle 3 strict rules follow karta hai: Longest Prefix Match, Administrative Distance, aur Metric.

1. **Longest Prefix Match:** The router prefers the route entry with the most specific subnet mask (highest CIDR prefix).
2. **Administrative Distance (AD):** If the same network is learned via multiple routing sources, the router installs the route with the lowest AD.
3. **Metric:** If the same network is learned from the same protocol with different paths, the router uses the metric value (e.g., bandwidth, cost, hop count).

**Administrative Distance Reference:**
Connected=0, Static=1, eBGP=20, EIGRP=90, OSPF=110, RIP=120.

### 2. OSPF Architecture

> [!info] Key Concept
> OSPF (Open Shortest Path First) is a Link-State routing protocol.

- **Dijkstra's SPF Algorithm:** Every router runs the SPF algorithm to calculate a loop-free tree of shortest paths using Link Cost.
- **LSA & LSDB:** Routers share Link State Advertisements (LSAs) which are compiled into a shared Link State Database (LSDB).
- **Neighbor States:** OSPF passes through 7 states to form adjacencies: Down -> Init -> Two-Way -> ExStart -> Exchange -> Loading -> Full.

> [!danger] Common Mistake
> **Configuring conflicting dynamic and static routes without checking AD:** Expecting dynamic OSPF paths to load-balance with static route configurations. The router will ignore the OSPF route because static routes have an AD of 1, while OSPF has an AD of 110. Ensure static backup routes use an AD larger than 110.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Cisco Packet Tracer with 2x 1941 Routers (R1, R2) and 2x PCs.
> - R1 and R2 connected via Serial; PCs connected via GigabitEthernet.

### Step 1: Configure Interfaces & Static Routing

```bash
# R1 Configuration
interface gi0/0
ip address 192.168.1.1 255.255.255.0
no shut

interface s0/0/0
ip address 10.0.0.1 255.255.255.252
no shut

# Configure static route targeting R2's LAN
ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

> [!success] Expected Output
> PC1 (192.168.1.10) should successfully ping PC2 (192.168.2.10) after return route is added on R2.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `ip route` | Standard static route configure karta hai | `ip route 192.168.2.0 255.255.255.0 10.0.0.2` |
| `ip route 0.0.0.0 0.0.0.0` | Default route (Gateway of Last Resort) | `ip route 0.0.0.0 0.0.0.0 203.0.113.1` |
| `router ospf` | OSPF process start karta hai | `router ospf 1` |
| `network` | OSPF enabled interface match karta hai wildcard mask se | `network 192.168.10.0 0.0.0.255 area 0` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| OSPF neighbor adjacency not forming | Hello/Dead timers, subnet masks, area IDs, or MTU mismatch | Run `show ip ospf interface`, match Hello timers and Area ID on both sides. |
| Floating static backup route fails | AD configured incorrectly or primary physical link remains UP logically | Configure IP SLA tracking with `track 1 ip sla 1 reachability` to monitor next-hop. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: OSPF Adjacency Down

> [!example] Ticket
> "Branch A router is no longer communicating with the HQ Core router via OSPF after a switch reboot."

**L1 Response:** Log into the router, run `show ip route` aur check karein routing table mein OSPF routes hain ya nahi. Ping test karein next hop par.
**Escalation Trigger:** Agar ping hop clear hai but `show ip ospf neighbor` list empty hai.
**L2 Resolution:** Check `show ip ospf interface`. Identify ki Hello timer mismatch ho gaya hai ya MTU size incorrect ho gaya hai reboot ke baad. Fix the interface parameters so OSPF reaches FULL state.

### 🎫 Scenario 2: SD-WAN / Wireless Troubleshooting
*(Integrated Enterprise Analysis)*

> [!example] Ticket
> "Enterprise Wi-Fi clients are failing 802.1X Auth or experiencing SD-WAN voice lag."

**L1 Response:** Check Wireshark packet capture: isolate HTTP/TCP issues (`tcp.analysis.retransmission`). Verify basic connectivity.
**Escalation Trigger:** Agar SD-WAN latency high dikhe ya RADIUS server rejection error ho.
**L2 Resolution:** For Wi-Fi, verify certificate validation on RADIUS/NPS. For SD-WAN, check real-time link quality metrics (jitter, packet loss) and verify control plane decoupling paths.

---
## 🎤 Interview Questions

> [!question] Q1: Explain the difference between Link-State and Distance-Vector routing protocols.
> **Answer:** Distance-Vector (RIP) advertises routing tables to direct neighbors (hop count focus). Link-State (OSPF) uses LSAs to build a complete network map (LSDB) and calculates the shortest path via Dijkstra's algorithm.

==**Exam Tip:** OSPF default Administrative Distance is 110, EIGRP is 90. EIGRP routes will be preferred over OSPF.==

> [!question] Q2: What are Designated Routers (DR) and Backup Designated Routers (BDR) in OSPF?
> **Answer:** DR/BDR are elected on multi-access networks (like Ethernet) to prevent excessive OSPF routing traffic (LSA flooding). Other routers (DROTHERs) form full adjacencies only with the DR/BDR.

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Wildcard masks and IP structures.
- [[01-Foundations/02-Networking/N-06 Switching — VLANs and Trunking|N-06 Switching — VLANs and Trunking]] — Inter-VLAN routing interfaces (ROAS).
- [[01-Foundations/02-Networking/N-09 Access Control Lists|N-09 Access Control Lists]] — Filtering IP routing flows.
