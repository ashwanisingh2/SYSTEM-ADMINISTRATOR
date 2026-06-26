---
tags: [networking, switching, vlan, intermediate]
aliases: [N-06]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#intermediate` `#ccna`

# N-06: Switching — VLANs and Trunking

> [!abstract] Overview
> This note covers Layer 2 switching concepts, including Virtual Local Area Network (VLAN) design, 802.1Q trunk encapsulation, Voice VLAN implementation, and Inter-VLAN routing (Router-on-a-Stick). Ek support engineer ko yeh aana chahiye taaki woh network traffic isolation aur trunk misconfigurations ko asaani se troubleshoot kar sake.

---
## 🧠 Concept Overview

- **What it is** — A VLAN (Virtual Local Area Network) isolates traffic at Layer 2, splitting a single physical switch into multiple logical switches. Each VLAN is a distinct broadcast domain. A Trunk is a high-speed link that carries traffic for multiple VLANs.
- **Why it matters** — Isolates sensitive data, limits the propagation of broadcast frames, and groups users logically.
- **Where you see this** — Separating Finance from Guest users, or configuring IP phones on a dedicated Voice VLAN.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check if user is in correct VLAN, verify port link status |
| **L2** | Configure VLANs, assign access/trunk ports, fix native VLAN mismatches |
| **L3** | Network architecture, inter-VLAN routing, enterprise Spanning Tree design |

> [!tip] Seedha Simple Mein
> *VLAN physical switch ko chote-chote logical switches mein divide karta hai taaki alag-alag departments ka traffic isolated rahe. Trunk port ek aisi link hai jo multiple VLANs ke data ko ek hi cable se do switches ke beech transport karti hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A physical switch** is like **a co-working space** because...
> 
> - By default, everyone is in one open hall (one broadcast domain).
> - **VLANs** are like invisible, soundproof glass partitions. Finance gets blue badges (VLAN 10) and guests get yellow badges (VLAN 20).
> - A **Trunk** is like a high-speed elevator between floors that carries badges of all colors, attaching color-coded tags as they enter.
> - A **Router** is the security guard desk in the lobby to let blue badges talk to yellow badges.

---
## 🔬 Technical Deep Dive

### 1. VLAN Types

> [!info] Key Concept
> - **Data VLAN (Default VLAN 1):** Carries user-generated traffic (e.g., web browsing).
> - **Voice VLAN:** Higher QoS priority for VoIP calling.
> - **Management VLAN:** Dedicated subnet for SSH/HTTPS administrative access.
> - **Native VLAN:** The single VLAN whose frames are sent untagged across an 802.1Q trunk.

### 2. Access Ports vs. Trunk Ports

- **Access Port:** Belongs to a single VLAN. Used to connect end devices (PCs, printers). Frames leaving the port are untagged.
- **Trunk Port:** Carries traffic for multiple VLANs simultaneously using 802.1Q tagging (inserts a 4-byte tag with a 12-bit VLAN ID).

> [!danger] Common Mistake
> **Configuring IP addresses on physical Router-on-a-Stick interface ports:** Assigning an IP directly to the physical interface `Gi0/0` prevents routing updates and breaks sub-interface encapsulation processing. Only run `no shutdown` to enable the port, then assign IPs to the sub-interfaces (e.g., `Gi0/0.10`).

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Cisco Packet Tracer with 1x 2960 Switch, 1x 1941 Router, and 2x PCs.
> - Connect PC0 to `Fa0/5`, PC1 to `Fa0/10`, and Switch `Gi0/1` to Router `Gi0/0`.

### Step 1: Configure VLANs on the Switch

```bash
# Enter global config and create VLANs
en
conf t
vlan 10
name Sales
vlan 20
name HR
```

### Step 2: Assign ports to VLANs

```bash
# Assign to VLAN 10 and 20
interface fa0/5
switchport mode access
switchport access vlan 10
interface fa0/10
switchport mode access
switchport access vlan 20
```

### Step 3: Configure ROAS on the Router

```bash
# Create sub-interfaces for VLAN routing
en
conf t
interface gi0/0
no shut
interface gi0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```

> [!success] Quick Win
> Ping from PC0 to PC1 should succeed, and `show mac address-table` on the switch will verify correct port allocations.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `switchport mode access` | Sets port for single VLAN (end devices) | `switchport mode access` |
| `switchport access vlan [id]` | Assigns port to specific VLAN | `switchport access vlan 10` |
| `switchport mode trunk` | Enables multiple VLANs on a port | `switchport mode trunk` |
| `switchport trunk native vlan [id]` | Changes Native VLAN | `switchport trunk native vlan 99` |
| `encapsulation dot1Q [id]` | Binds router sub-interface to VLAN | `encapsulation dot1Q 10` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| PCs in same VLAN on diff switches cannot ping | Trunk link mismatch or VLAN not allowed on trunk | Check `show interfaces trunk`. Run `switchport trunk allowed vlan add 10` |
| `%CDP-4-NATIVE_VLAN_MISMATCH` syslog error | Native VLAN mismatch between switches causing frame leakage | Align native VLANs: `interface gi0/1` -> `switchport trunk native vlan 99` |
| IP Phones disconnecting intermittently | Voice and Data traffic in same VLAN saturating interface | Configure Voice VLAN on port: `switchport voice vlan [ID]` |
| 802.1X EAP Authentication Failed | RADIUS certificate or identity store issue | Verify certificate settings on NPS server |
| Wi-Fi Roaming failure (sticky client) | Coverage overlap too high or RSSI threshold missing | Adjust Minimum RSSI settings on APs or ensure 15-20% overlap |
| RF Interference on Wireless | Saturated 2.4GHz bands | Move APs to 5GHz/6GHz using 20/40MHz channels |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Inter-VLAN Communication Failure

> [!example] Ticket
> "I am connected to the new switch on Port 5, but I cannot access the HR server in VLAN 20."

**L1 Response:** Verify user's IP, subnet, and physical switch port status. Confirm they are in the correct VLAN.
**Escalation Trigger:** Port is in the correct VLAN, link is up, but inter-VLAN ping is failing.
**L2 Resolution:** Check the trunk link to the router. Verify Router-on-a-Stick sub-interface encapsulation (`encapsulation dot1Q 10` and `20`) and confirm the gateway IP matches the PCs.

---
## 🎤 Interview Questions

> [!question] Q1: What is a Native VLAN mismatch, and what are its operational consequences?
> **Answer:** It occurs when trunk ports on two connected switches have different Native VLAN IDs. Untagged frames are incorrectly assigned, causing traffic to leak between VLANs without passing through a router, creating severe security vulnerabilities, STP instability, and packet loss.

> [!question] Q2: How does VLAN Hopping work, and how do you configure a switch to prevent it?
> **Answer:** It is an attack where a host transmits traffic outside its access scope using switch spoofing or double tagging. Prevent it by configuring explicit access ports (`switchport mode access`), disabling DTP (`switchport nonegotiate`), and changing the native VLAN of trunks to an unused, isolated VLAN.

==**Exam Tip:** For security compliance, always change the Native VLAN from the default VLAN 1 to an unused dummy VLAN ID (e.g., VLAN 999), and configure the trunk to prune (disallow) VLAN 999.==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-02 Network Devices Deep Dive|N-02 Network Devices Deep Dive]] — Base switch operations and CAM table learning
- [[01-Foundations/02-Networking/N-04 IPv4 Addressing Complete Guide|N-04 IPv4 Addressing Complete Guide]] — VLAN IP subnetting layouts
- [[01-Foundations/02-Networking/N-07 Routing — Static and Dynamic|N-07 Routing — Static and Dynamic]] — Routing protocol configurations between VLAN networks
