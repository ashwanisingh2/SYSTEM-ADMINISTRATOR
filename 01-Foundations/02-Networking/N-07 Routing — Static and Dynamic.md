---
tags: [sysadmin, networking, routing, ospf]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# N-07: Routing — Static and Dynamic

> [!abstract] Overview
> This note covers routing engine operation, Administrative Distance (AD), static/default route setups, and Open Shortest Path First (OSPF) dynamic routing configuration. It details the OSPF SPF algorithm, neighborhood states, and Cisco IOS commands.

---
## Concept
Think of routing like a global road trip GPS system:
- **Routing Table** is the GPS map database containing active roads.
- **Administrative Distance** is your trust rating for different map apps: if Apple Maps says turn left (AD 120) but your personal experience says go straight (AD 1), you trust your personal experience first.
- **Static Route** is a manual road closure detour you draw on the map yourself because you know a specific path.
- **Default Route** is the general signpost that says "To All Other Cities, Head Toward the Highway."
- **OSPF Dynamic Routing** is like having all delivery drivers on the road constantly radioing each other about traffic, crashes, and speed traps (LSAs) so that every driver can construct a complete 3D model of the city (Link State Database) and independently calculate the fastest possible route (SPF calculation).

*Seedha simple mein: Router packages ko ek network se dusre network bhejta hai. Static routing mein admin rasta manually batata hai (0.0.0.0/0 default route internet ke liye). Dynamic routing mein protocols jaise OSPF automatic aur fast routes calculate karte hain updates share karke.*

---
## Technical Deep Dive

### 1. The Router Decision-Making Process
When a packet arrives at a router interface, the router inspects the destination L3 IP address and parses the routing table using three rules in strict order:
1. **Longest Prefix Match:** The router prefers the route entry with the most specific subnet mask (highest CIDR prefix). E.g., a packet for `192.168.1.50` matches both `192.168.1.0/24` and `192.168.1.48/28`. The router chooses the `/28` route because it is more specific.
2. **Administrative Distance (AD):** If the same network is learned via multiple routing sources, the router installs the route with the lowest AD.
3. **Metric:** If the same network is learned from the same protocol with different paths, the router uses the metric value (e.g., bandwidth, cost, hop count) to choose the best route.

### 2. Administrative Distance (AD) Reference Table
AD measures the trustworthiness of a routing source (Scale: $0$ to $255$. $0$ is most trusted; $255$ will not be installed).

| Route Source | Default AD | Metric Description |
|---|---|---|
| **Connected Interface** | 0 | Direct physical link |
| **Static Route** | 1 | Manually entered by administrator |
| **BGP (External)** | 20 | Inter-autonomous system routing |
| **EIGRP (Internal)** | 90 | Proprietary Cisco hybrid protocol |
| **OSPF** | 110 | Open standard Link-State protocol |
| **IS-IS** | 115 | Link-State protocol |
| **RIP** | 120 | Legacy Distance-Vector protocol (hop count) |
| **Unusable / Unknown** | 255 | Route is ignored |

### 3. Static and Backup (Floating) Routes
- **Static Route:** A manual route defining the outbound interface or next-hop IP:
  - *Cisco command:* `ip route [Network] [Mask] [Next-Hop_IP]`
- **Default Route:** A static route matching all destinations not explicitly present in the routing table:
  - *Format:* `0.0.0.0 0.0.0.0` (also called the Gateway of Last Resort).
- **Floating Static Route:** A backup static route configured with a manually inflated AD (e.g., AD 150). It remains hidden in the configuration and is only installed in the routing table if the primary routing link (e.g., OSPF at AD 110) goes offline.

### 4. Dynamic Routing: OSPF Architecture
**OSPF (Open Shortest Path First)** is a Link-State routing protocol.
- **Dijkstra's SPF Algorithm:** Every router runs the Shortest Path First (SPF) algorithm to calculate a loop-free tree of shortest paths using **Link Cost** ($\text{Cost} = \frac{10^8}{\text{Bandwidth in bps}}$).
- **LSA (Link State Advertisement):** Routers share LSAs describing local link states. LSAs are compiled into the **LSDB (Link State Database)**. All routers in an area share an identical LSDB.
- **Areas:** OSPF uses hierarchical areas to limit LSA propagation. **Area 0** is the core backbone area. All other areas must connect directly to Area 0.

### 5. OSPF Neighbor States
OSPF routers form adjacencies by passing through seven progressive states:
1. **Down:** No OSPF packets received.
2. **Init:** Hello packet received, but the sender's own Router ID is not listed in the neighbor list.
3. **Two-Way:** Bidirectional hello communication established. **DR (Designated Router)** and **BDR (Backup DR)** elections occur on multi-access networks (like Ethernet).
4. **ExStart:** Routers prepare to exchange database summaries. Determine Master/Slave roles.
5. **Exchange:** Routers send **DBD (Database Description)** packets listing LSA headers.
6. **Loading:** Routers request detailed LSAs using Link State Requests (LSRs) and respond with Link State Updates (LSUs).
7. **Full:** Adjacency is complete. LSDBs are synchronized. Routers are fully adjacent.

---
## Cisco IOS Configuration Commands

### Static and Default Routes
```text
Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2      ! (Standard Static Route)
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1              ! (Default Route / Gateway of Last Resort)
Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.6 150  ! (Floating Static Route with AD 150)
```

### OSPF v2 (IPv4) Configuration
```text
Router(config)# router ospf 1                                    ! (Starts OSPF process 1)
Router(config-router)# router-id 1.1.1.1                         ! (Manually sets unique Router ID)
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0     ! (Enable OSPF on network using Wildcard Mask)
Router(config-router)# network 10.0.0.0 0.0.0.3 area 0
Router(config-router)# passive-interface gigabitethernet 0/1     ! (Stop OSPF Hello broadcasts on LAN ports)
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Cisco Packet Tracer with 2x 1941 Routers (R1, R2) and 2x PCs.

### Step 1: Cable the Topology
1. Connect R1 Serial 0/0/0 port to R2 Serial 0/0/0 port.
2. Connect R1 Gi0/0 to PC1. Labeled LAN: `192.168.1.0/24`.
3. Connect R2 Gi0/0 to PC2. Labeled LAN: `192.168.2.0/24`.
4. WAN link Serial subnet: `10.0.0.0/30`. (R1 Serial: `10.0.0.1`, R2 Serial: `10.0.0.2`).

### Step 2: Configure IP Addresses
1. Log into R1. Configure interfaces:
   ```text
   interface gi0/0
   ip address 192.168.1.1 255.255.255.0
   no shut
   interface s0/0/0
   ip address 10.0.0.1 255.255.255.252
   no shut
   ```
2. Log into R2. Configure interfaces:
   ```text
   interface gi0/0
   ip address 192.168.2.1 255.255.255.0
   no shut
   interface s0/0/0
   ip address 10.0.0.2 255.255.255.252
   no shut
   ```

### Step 3: Configure Static Routing
Before routing is configured, PC1 (`192.168.1.10`) cannot ping PC2 (`192.168.2.10`).
1. Configure static route on R1 targeting PC2's LAN via R2:
   ```text
   R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
   ```
2. Configure return static route on R2 targeting PC1's LAN via R1:
   ```text
   R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
   ```
3. **Verify:** Ping from PC1 to PC2. The ping should succeed. Check the routing table using `show ip route` to see the installed `S` (Static) routes.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** OSPF neighbor adjacency is not forming between two routers. The command `show ip ospf neighbor` returns an empty table.
- **Root Cause:** A mismatch in Hello/Dead timers, subnet masks, area IDs, or MTU configurations on the connecting link.
- **Fix:**
  1. Run `show ip ospf interface [name]` on both routers.
  2. Verify:
     - **Hello Timers:** Must match (Default is 10s broadcast, 30s non-broadcast).
     - **Area ID:** Both interfaces must belong to the same Area (e.g., Area 0).
     - **Subnet/Mask:** Both interfaces must reside on the same IP subnet.
  3. If timers or areas mismatch, correct the configurations:
     ```text
     interface gi0/0
     ip ospf hello-interval 10
     ```

**Scenario 2:**
- **Problem:** A backup floating static route fails to take over traffic when the primary OSPF link is physically disconnected.
- **Root Cause:** The floating static route AD was configured incorrectly, or the primary link is still logically active (e.g., a media converter is keeping the switch port status UP even though the backend fiber is severed).
- **Fix:**
  1. Inspect the configuration: `show running-config | include ip route`.
  2. Verify that the backup static route has a metric higher than 110 (e.g., `ip route 0.0.0.0 0.0.0.0 192.168.1.2 120`).
  3. If the physical interface state stays UP, configure **IP SLA (Service Level Agreement)** tracking to ping the destination next-hop. Bind the static route to the SLA tracker:
     ```text
     R1(config)# ip sla 1
     R1(config-ip-sla)# icmp-echo 10.0.0.2
     R1(config-ip-sla-echo)# frequency 5
     R1(config)# track 1 ip sla 1 reachability
     R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2 track 1
     ```

---
## Common Mistakes
> [!warning] Avoid These
> **Configuring conflicting dynamic and static routes without checking AD:** Expecting dynamic OSPF paths to load-balance with static route configurations. The router will ignore the OSPF route because static routes have an AD of 1, while OSPF has an AD of 110.
> **Correct approach:** Use routing protocols consistently. If mixing static and dynamic routes, ensure static backup routes use an AD larger than 110.

---
## Pro Tips
> [!tip] Field Experience
> Always set a manual `router-id` when configuring OSPF. If left to default, the router selects the highest loopback IP, or the highest active physical IP. If that interface bounces (disconnects/reconnects), the Router ID changes, causing OSPF process restarts and network-wide routing recalculations.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Longest Match | The first rule of routing; router selects path with the longest matching prefix (CIDR). |
| 2 | Admin Distance | Trust rating; connected is 0, static is 1, OSPF is 110, RIP is 120. |
| 3 | Default Route | `0.0.0.0/0`; forwards all unknown traffic toward the WAN provider gateway. |
| 4 | OSPF FULL State | The final neighborhood state; indicates databases are fully synchronized. |
| 5 | Passive Interface| OSPF command that stops OSPF hellos on LAN ports while still advertising the subnet. |

---
## Interview Q&A

**Q1: Explain the difference between Link-State and Distance-Vector routing protocols.**
A: **Distance-Vector protocols** (e.g., RIP) advertise their routing tables to directly connected neighbors. The router does not have a complete map of the network; it only knows the direction (vector) and distance (hops) to a network. Updates are sent periodically. **Link-State protocols** (e.g., OSPF) advertise the state of their local links using LSAs. These LSAs are flooded throughout the network, allowing every router to compile an identical, complete map of the network (LSDB) and run Dijkstra's algorithm locally to calculate paths. Updates are sent only on changes.

**Q2: A router has learned the route to 10.5.0.0/16 via OSPF (Cost 20) and via EIGRP (Metric 51200). Which route is installed in the routing table, and how does it change if the primary link breaks?**
A: 
- **Situation:** A route is learned through both OSPF and EIGRP.
- **Task:** Determine route installation based on Administrative Distance (AD).
- **Action:** I will compare the default AD of the protocols. EIGRP has a default AD of 90, and OSPF has a default AD of 110. EIGRP is more trusted because its AD is lower.
- **Result:** The router installs the EIGRP route in the routing table. The OSPF route is stored in the database but not active. If the EIGRP link breaks, the EIGRP route is removed, and the OSPF route is automatically installed.

**Q3: What are Designated Routers (DR) and Backup Designated Routers (BDR) in OSPF, and why are they elected?**
A: DR and BDR are elected on multi-access networks (like Ethernet LAN segments) to prevent excessive OSPF routing traffic. If $N$ routers are connected to a switch, a full mesh of OSPF adjacencies would require $\frac{N(N-1)}{2}$ links, causing massive Hello/LSA flooding. By electing a DR and BDR, all other routers (DROTHERs) only form full adjacencies with the DR and BDR. When a link state changes, the DROTHER sends an update to the DR/BDR via multicast address `224.0.0.6`. The DR then floods the update to all other routers on the segment using `224.0.0.5`.

---
## Related Notes
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — Wildcard masks and IP structures.
- [[01-Foundations/02-Networking/N-06 Switching — VLANs and Trunking|N-06 Switching — VLANs and Trunking]] — Inter-VLAN routing interfaces (ROAS).
- [[01-Foundations/02-Networking/N-09 Access Control Lists|N-09 Access Control Lists]] — Filtering IP routing flows.
