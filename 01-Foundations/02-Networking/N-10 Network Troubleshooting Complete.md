---
tags: [sysadmin, networking, troubleshooting, diagnostics]
aliases: [N-10 Network Troubleshooting]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#advanced` `#ccna`

# N-10: Network Troubleshooting Complete

> [!abstract] Overview
> This note outlines advanced diagnostic workflows for isolating network connectivity, routing, and resolution failures. It details protocol stack validation, command utility parameters, Wireshark traffic analysis, and troubleshooting flowcharts. Ek support engineer ko yeh aana chahiye kyunki network issue isolate karna sabse critical L1/L2 task hai.

---
## 🧠 Concept Overview

- **What it is** — Troubleshooting a network is like diagnosing a clean-water delivery system using a layered model (Physical, Network, Transport, Application).
- **Why it matters** — Isolate network outages correctly to resolve issues quickly instead of guessing.
- **Where you see this** — When users report "internet is down" or "cannot reach server".

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check physical cables, ping tests, basic DNS checks |
| **L2** | Configure, fix, escalate — Run tracert, netstat, check port availability |
| **L3** | Architecture, design — Wireshark packet capture analysis, SD-WAN routing, protocol stack deep dives |

> [!tip] Seedha Simple Mein
> *Network troubleshoot karne ke liye hum step-by-step diagnostic tools use karte hain. Ping connectivity check karta hai, Tracert path ke hops batata hai, Nslookup DNS check karta hai, aur Netstat active ports/connections dikhata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **City Water System** is like **Network Layers** because...
>
> - **Physical/Link (L1/L2):** Is the pipe physically broken or disconnected? Are the connectors leaking?
> - **Network (L3):** Can water travel from the treatment plant to your street block? (Routing/Ping)
> - **Transport (L4):** Is the correct intake valve open at the house? (Ports/Netstat)
> - **Application (L7):** Is the tap giving clean, filtered water, or is the boiler failing? (DNS/HTTP logs)

---
## 🔬 Technical Deep Dive

### 1. Systematic Troubleshooting Framework

> [!info] Key Concept
> Always isolate network outages using a layered model (bottom-up is most effective for physical links).

```
[Application Layer]     <-- Check browser error, DNS name (nslookup)
        |
[Transport Layer]       <-- Check port availability (netstat / Test-NetConnection)
        |
[Network Layer]         <-- Check IP path and routing (ping / tracert)
        |
[Data Link/Physical]    <-- Check link lights, speed duplex settings, swap cable
```

### 2. Wireshark Traffic Analysis Basics

Wireshark captures raw frames from the physical NIC using PCAP libraries.
- **Color Codes:** Black usually indicates packet TCP resets, retransmissions, or checksum errors.

> [!danger] Common Mistake
> **Running traceroute without knowing target network policies:** Assuming a remote host is offline because `tracert` stops responding at hop 5. Many firewalls block ICMP time-exceeded messages, making the host appear offline even if HTTP/SSH ports are open. Always supplement ping/traceroute checks with port-specific scanners.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Basic understanding of CMD or PowerShell
> - Administrator privileges for some commands

### Step 1: Wireshark Packet Capture Filtering

```bash
# Filter for HTTP POST Request Error isolation
http.request.method == "POST" && http.response.code >= 400

# Isolate TCP Retransmissions (Packet Loss)
tcp.analysis.retransmission || tcp.analysis.duplicate_ack

# DNS Server response failures check
dns.flags.response == 1 && dns.flags.rcode != 0
```

> [!success] Expected Output
> ```
> Wireshark displays only the matching packets, reducing noise and focusing on the issue.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `ping` | Checks ICMP connectivity and TTL. | `ping -t 8.8.8.8` |
| `tracert` | Diagnoses path routing using TTL markers. | `tracert 8.8.8.8` |
| `pathping` | Combines ping and tracert for path statistics. | `pathping 8.8.8.8` |
| `netstat -an` | Shows active connections and listening ports. | `netstat -an` |
| `route print` | Shows local routing database (IPv4). | `route print` |
| `Test-NetConnection` | Test if target TCP port is open (PowerShell). | `Test-NetConnection -ComputerName 192.168.1.50 -Port 443` |
| `Resolve-DnsName` | Advanced DNS lookup details. | `Resolve-DnsName www.google.com` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Cannot connect to legacy web server | Web service stopped or wrong port | Run `Test-NetConnection` to verify port 80. RDP in, check `netstat -ano`, start service. |
| Slow file copy to NAS, dropped pings | MTU mismatch causing packet fragmentation | Test MTU with `ping -f -l 8000`. Lower packet size or disable Jumbo Frames (set MTU to 1500). |
| 802.1X EAP Auth Failed (Wi-Fi) | Cert validation or client identity issue | Verify certificate on RADIUS server (NPS) and check client identity store. |
| Roaming failure (sticky client) | AP coverage overlap or RSSI issue | Adjust Minimum RSSI on APs or ensure identical SSIDs with 15-20% overlap. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Legacy Web Server Down

> [!example] Ticket
> "Cannot connect to legacy internal web server. Browser returns 'Connection Refused'."

**L1 Response:** Verify basic connectivity. Ping the server IP. If ping succeeds, confirm the issue is application/port specific.
**Escalation Trigger:** Pass to L2 if service is running but port remains inaccessible from network.
**L2 Resolution:** Open PowerShell, run `Test-NetConnection [IP] -Port 80`. RDP into server, run `netstat -ano | findstr LISTENING`. Start Web Server service (IIS/Apache) if stopped.

---
## 🎤 Interview Questions

> [!question] Q1: A user reports they can access internal resources, but cannot open external websites. Walk through your troubleshooting methodology.
> **Answer:** First, verify IP details (`ipconfig /all`) for gateway and DNS. Ping gateway to check local L3. Ping external IP (`8.8.8.8`). If external IP pings but `google.com` fails, it's a DNS issue.

==**Exam Tip:** The order of operations in troubleshooting is critical: Physical -> Local IP -> Gateway -> External IP -> DNS.==

> [!question] Q2: What is a packet fragment storm, and how does it impact router CPU performance?
> **Answer:** Occurs when packets exceed MTU, forcing router CPU to fragment them in software (control plane) instead of hardware, slowing down all traffic.

> [!question] Q3: How do you identify a TCP SYN Flood DDoS attack using Netstat?
> **Answer:** Run `netstat -an`. Hundreds of connections stuck in `SYN_RECEIVED` or `SYN_RECV` state from spoofed IPs confirm the attack.

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-01 Networking Fundamentals|N-01 Networking Fundamentals]] — The 7 layers of the OSI model.
- [[01-Foundations/02-Networking/N-03 Ethernet and MAC Address|N-03 Ethernet and MAC Address]] — ARP caches and frame details.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Resolution DNS and routing NAT helpers.
