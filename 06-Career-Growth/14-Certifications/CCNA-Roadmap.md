---
tags: [desktop-support, career-roadmap, certification, cisco, ccna, networking, L1, L2, L3]
aliases: [ccna-prep, cisco-roadmap]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #ccna-200-301
---

# CCNA (200-301) Certification Study Roadmap

---

## Concept Overview
- **What it is**: A comprehensive study guide, network lab outline, and exam strategy for the Cisco Certified Network Associate (CCNA 200-301) certification.
- **Why it matters**: The CCNA is the gold standard for networking certifications. It teaches fundamental routing, switching, IP services, and network automation, transforming an L1 desktop engineer into an L2/L3 systems administrator.
- **Where you encounter this in real job**: Configuring office switches, setting up VLANs, troubleshooting connection failures across routers, and managing IP address allocations.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Identifies bad network cables, checks interface link lights, resolves basic IP address lease issues, and logs connectivity tickets.
  - **L2**: Configures switch access ports, maps VLAN ranges, sets up trunk links, and configures static routing tables.
  - **L3**: Architectures routing protocols (OSPF), implements Access Control Lists (ACLs) for security, configures wireless networks (WLC), and writes network automation scripts.

---

## Technical Deep Dive

### CCNA Exam Domain Breakdown
The CCNA 200-301 exam evaluates candidates across six key network domains:
```
  [1. Network Fundamentals (20%)]  --->  [2. Network Access (20%)]    --->  [3. IP Connectivity (25%)]
             |                                       |                                   |
             v                                       v                                   v
  [4. IP Services (10%)]           --->  [5. Security Fundamentals (15%)] --->  [6. Automation & Programmability (10%)]
```

### Key Cisco CLI Configuration Concepts
Cisco IOS switches and routers use a state-based configuration model divided into modes:
- **User EXEC Mode (`Router>`)**: Read-only access. Used for basic system monitoring.
- **Privileged EXEC Mode (`Router#`)**: Detailed status monitoring, debugging, and system restarts. Entered via `enable`.
- **Global Configuration Mode (`Router(config)#`)**: System-wide configuration. Entered via `configure terminal`.
- **Interface Configuration Mode (`Router(config-if)#`)**: Specific configuration for a port (e.g. Gig0/1).

---

## Commands & Syntax

### Cisco IOS Configuration (CLI)
Common Cisco command sets for configuring VLANs and Trunking:
```ios
# 1. Enter global configuration mode
enable
configure terminal

# 2. Configure a VLAN and assign to an access interface
vlan 10
 name Marketing-Dept
exit
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
exit

# 3. Configure a trunk interface to carry all VLANs
interface GigabitEthernet0/24
 description Core-Switch-Link
 switchport mode trunk
 switchport trunk allowed vlan all
exit

# 4. Save the configuration to non-volatile memory
end
copy running-config startup-config
```

### Windows Shell: Network Verification
```cmd
REM Display local routing table to check default gateway configuration
route print

REM Query active network paths and gateway hops
pathping 8.8.8.8
```

### GUI Path
> **Cisco Packet Tracer**: Add Switch -> Click device -> CLI tab (for command simulation)
> **Workstation**: Device Manager -> Ports (COM & LPT) -> Select Serial Console -> Open PuTTY

---

## Real-World Scenarios

### Scenario 1: VLAN Mismatch causing Network Isolation
**User Complaint**: A new computer was plugged into port 12 on the department switch. The device cannot obtain an IP address from DHCP and displays a "No Internet Access" warning.
**Your First 3 Checks**:
1. Check the physical link lights on the switch port.
2. Check the VLAN configuration assigned to switch port 12.
3. Review the DHCP scope leases.
**Diagnosis Steps**:
1. Connect a console cable to the switch and run: `show interface GigabitEthernet0/12 status`. Port is `connected/up`.
2. Run: `show mac address-table interface GigabitEthernet0/12`. The switch is learning the PC's MAC address.
3. Run: `show run interface GigabitEthernet0/12`. The port is configured with `switchport access vlan 99`.
**Root Cause**: Port 12 is assigned to VLAN 99, which is a legacy VLAN with no active DHCP helper address configured on the gateway router. The PC should be on VLAN 10 (Marketing).
**Fix**:
1. Enter switch configuration mode:
   ```ios
   configure terminal
   interface GigabitEthernet0/12
   switchport access vlan 10
   end
   ```
2. Verify the configuration change.
3. On the client PC, run `ipconfig /renew`. The PC receives a lease from the VLAN 10 DHCP scope.
**Prevention**: Standardize switch port assignments by writing description tags to unused ports and shutting them down until provisioned.

### Scenario 2: Native VLAN Mismatch on Trunk Link
**User Complaint**: Network traffic between Switch-A and Switch-B is running slow, and log files are flooded with error messages.
**Your First 3 Checks**:
1. Check the trunk configuration on the connecting interfaces of both switches.
2. Review the switch logs for STP (Spanning Tree) or CDP (Cisco Discovery Protocol) alerts.
3. Check the native VLAN configurations on both sides of the trunk.
**Diagnosis Steps**:
1. Log into Switch-A and run `show interface trunk`. The native VLAN on GigabitEthernet0/24 is set to `VLAN 1`.
2. Log into Switch-B and run `show interface trunk`. The native VLAN on GigabitEthernet0/24 is set to `VLAN 99`.
3. Check Switch-A console logs: `%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on GigabitEthernet0/24 (1), with Switch-B GigabitEthernet0/24 (99).`
**Root Cause**: A native VLAN mismatch exists on the trunk link. Untagged traffic is being routed to different subnets on each switch, causing Spanning Tree loops and packet loss.
**Fix**:
1. Align the native VLAN configuration. Change Switch-A's native VLAN to match Switch-B:
   ```ios
   configure terminal
   interface GigabitEthernet0/24
   switchport trunk native vlan 99
   end
   write memory
   ```
2. Confirm the Native VLAN Mismatch log alerts stop.
**Prevention**: Enforce a template standard where the native VLAN is always explicitly configured on all trunk ports across the enterprise network.

---

## Critical Points

> [!danger] Never Do This
> Do not connect redundant network switches together without verifying that **Spanning Tree Protocol (STP)** is enabled. Disabling STP on redundant links will cause a broadcast storm, crashing the entire local switch network within seconds.

> [!warning] Common Trap
> Configuring configurations on a Cisco device and failing to run `copy running-config startup-config`. If the device loses power or is rebooted, all unsaved changes will be lost, reverting the switch to its previous state.

> [!tip] Senior Engineer Tip
> Set the native VLAN to an unused VLAN ID (e.g., VLAN 999) on all trunk links, and disable dynamic trunk negotiation (`switchport nonegotiate`) on access ports to prevent VLAN hopping security attacks.

> [!success] Verification Steps
> To verify a trunk link carries target traffic:
> 1. Run `show interface trunk` on the switch console.
> 2. Confirm the connecting port shows "trunking" status.
> 3. Verify target VLAN IDs are listed in the "VLANs allowed and active in management domain" list.

> [!question] Interview Alert
> "What is the difference between TCP and UDP?"
> - **Answer**: TCP (Transmission Control Protocol) is a connection-oriented protocol that guarantees delivery via handshakes, sequencing, and error checks, used for data like web browsing (HTTP) or email (SMTP). UDP (User Datagram Protocol) is a connectionless, best-effort protocol that prioritizes speed over reliability, used for streaming media, DNS lookup, and VoIP.

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Subnetting calculation errors | Rushing calculations mentally | Master the finger-subnetting method or binary subtraction templates for the exam. |
| Forgetting to enable ports | Interfaces are shut down by default | Always run the `no shutdown` command when configuring a Cisco interface. |
| Missing default gateway | Router not routing external traffic | Configure the default gateway routing using `ip route 0.0.0.0 0.0.0.0 [Next-Hop-IP]`. |

---

## Lab Exercise

**Objective**: Configure a Cisco Switch with Basic Security, VLANs, and an IP Address for Remote Management using Cisco Packet Tracer.
**Time Required**: 30 minutes
**Environment Needed**: Cisco Packet Tracer.

**Steps**:
1. Open Packet Tracer, add a `2960 Switch` and a host `PC-A` connected via a straight-through cable to port `Fa0/1`.
2. Open the Switch CLI and configure basic host settings:
   ```ios
   enable
   configure terminal
   hostname SW-Core-01
   enable secret class
   ```
3. Configure the Switch Virtual Interface (SVI) for remote management:
   ```ios
   interface vlan 1
    ip address 192.168.1.10 255.255.255.0
    no shutdown
   exit
   ```
4. Set up the VTY (Telnet/SSH) access lines:
   ```ios
   line vty 0 4
    password cisco
    login
   exit
   ```
5. On `PC-A`, configure IP static to `192.168.1.5`, subnet `255.255.255.0`.
6. Open the Command Prompt on `PC-A` and test connectivity:
   ```cmd
   ping 192.168.1.10
   telnet 192.168.1.10
   ```
   - Enter password `cisco` to access the switch.

**Pass Criteria**: PC-A successfully pings the switch SVI and connects via Telnet.

---

## Interview Questions & Answers

### L1 Level

#### Q1: What is the purpose of a Subnet Mask?
**A**: A subnet mask is a 32-bit number used to divide an IP address into network and host portions. It helps devices determine whether a target IP address is on the local network segment (meaning it can be reached directly) or on an external network (meaning the packet must be sent to the default gateway).

#### Q2: What is the default port for DNS, DHCP, and SSH?
**A**: DNS uses port **53** (UDP for queries, TCP for zone transfers). DHCP uses ports **67** (servers) and **68** (clients). SSH uses secure port **22**.

---

### L2 Level

#### Q3: Explain how Spanning Tree Protocol (STP) prevents network loops.
**A**: STP builds a loop-free logical topology by selecting a single Root Bridge. All other switches calculate the shortest path to the Root Bridge. STP then places redundant paths into a blocking state (preventing data loop transmission), while keeping primary paths forwarding. If a primary link fails, STP recalculates and transitions the blocking port to forwarding.

#### Q4: What is the difference between routing protocol administrative distance (AD) and metric?
**A**:
- **Administrative Distance (AD)**: Measures the trustworthiness of a routing source (lower is preferred). For example, OSPF has an AD of 110, while Static Routes have an AD of 1.
- **Metric**: Calculates the cost to reach a destination within a specific routing protocol (e.g., OSPF uses bandwidth-based cost; RIP uses hop count).

---

### L3 Level

#### Q5: Explain how you would troubleshoot an OSPF neighbor adjacency failure between two routers.
**A**:
- **Situation**: A newly installed branch router was unable to establish an OSPF adjacency with the core office router, blocking local access.
- **Task**: Identify the configuration mismatch preventing neighbors from reaching the FULL state.
- **Action**: I ran `show ip ospf neighbor` on both routers. No neighbors were listed. I then ran `show ip ospf interface` and compared parameters. I checked the area IDs, subnet masks, hello/dead intervals, and OSPF MTU sizes. I discovered a hello-interval mismatch (10 seconds vs 30 seconds).
- **Result**: I corrected the hello timer values on the branch router interface. The OSPF adjacency transitioned from INIT to FULL immediately, restoring routing tables.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **OSI Model**: Layer 1 (Physical), Layer 2 (Data Link - MAC/Switch), Layer 3 (Network - IP/Router), Layer 4 (Transport - TCP/UDP).
> **Switching**: access mode for endpoint PCs; trunk mode to carry multiple VLANs across switches.
> **Default Cisco ADs**: Direct (0), Static (1), EIGRP (90), OSPF (110), RIP (120).
> **Command Save**: `copy run start` or `write mem`.

---

## Related Notes
- [[01-Foundations/02-Networking/OSI-Model|OSI Model]]
- [[01-Foundations/02-Networking/Subnetting|Subnetting]]
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Fundamentals|PowerShell Fundamentals]]

---

## Tags
#desktop-support #networking #cisco #ccna #routing-switching #vlan-configuration #subnetting #certification-roadmap #L2 #L3
