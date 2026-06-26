---
tags: [security, network-security, firewalls, advanced]
aliases: [SEC-06, Network Security and Firewalls]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-red]
> 🛡️ **SECURITY**

`#complete` `#advanced` `#none`

# SEC-06: Network Security and Firewalls

> [!abstract] Overview
> Yeh note Network Security aur Firewalls ke core concepts cover karta hai, jisme packet filtering, stateful inspection, aur Next-Generation Firewalls (NGFW) shamil hain. Ek support engineer ko yeh jaanna zaroori hai taaki woh network traffic blocks aur routing issues ko effectively troubleshoot kar sake.

---
## 🧠 Concept Overview

- **What it is** — Network Security is the process of protecting the underlying networking infrastructure from unauthorized access, misuse, or theft. Firewalls act as the primary defense mechanism by monitoring and filtering incoming and outgoing network traffic based on an organization's previously established security policies.
- **Why it matters** — Real job mein, jab bhi koi server internet se ya dusre zone se connect nahi ho pata, pehla shaq firewall par hi jaata hai. *Traffic block ho raha hai ya allow, yeh samajhna critical hai.*
- **Where you see this** — Jab users complain karte hain "Database server is unreachable" ya "Web application timeout ho rahi hai", wahan mostly firewall rules ya network ACLs (Access Control Lists) ka issue hota hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic connectivity check karta hai (ping, telnet, tracert/traceroute). Check if the port is open or closed from the client side. |
| **L2** | Firewall logs check karta hai, packet capture (tcpdump/Wireshark) run karta hai, aur basic rule additions/modifications (jaise iptables, firewalld, ya cloud NSG) karta hai. |
| **L3** | Enterprise firewall architecture (Palo Alto, Fortinet, Check Point), VPN routing, IPS/IDS integration, aur complex policy design karta hai. |

> [!tip] Seedha Simple Mein
> *Firewall aapke ghar ka security guard hai. Jo list (rules) uske paas hai, sirf unhi logon (packets) ko andar aane ya baahar jaane dega. Baaki sabko block kar dega.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Firewall** is like **a bouncer at a VIP club** because...
>
> - **Packet Filtering:** Bouncer sabki ID check karta hai. Agar ID valid nahi hai, toh entry denied. (Checking Source/Destination IP & Port).
> - **Stateful Inspection:** Agar aap club ke andar gaye hain (established connection), toh bouncer aapko yaad rakhta hai aur waapas aane deta hai bina dobara strict checking ke.
> - **Next-Generation Firewall (NGFW):** Yeh bouncer sirf ID nahi check karta, balki bag bhi scan karta hai aur dekhta hai ki koi dangerous item toh nahi hai (Deep Packet Inspection / Application Layer Filtering).

---
## 🔬 Technical Deep Dive

### 1. Types of Firewalls

> [!info] Key Concept
> Firewalls operate at different layers of the OSI model and offer varying levels of security.

- **Stateless Packet Filters:** Operates at Layer 3 (Network) and Layer 4 (Transport). Inspects each packet individually without memory of previous packets. *Bahut basic hota hai, sirf IP aur Port dekhta hai.*
- **Stateful Firewalls:** Maintains a state table of active connections. It knows if a packet is part of a new or existing connection. *Yeh thoda smart hai, connection ka state yaad rakhta hai.*
- **Proxy Firewalls / Application Firewalls:** Operates at Layer 7 (Application). Acts as an intermediary between the client and the server, inspecting the actual payload.
- **Next-Generation Firewalls (NGFW):** Combines stateful inspection with deep packet inspection (DPI), Intrusion Prevention Systems (IPS), and application awareness.

> [!danger] Common Mistake
> Confusing Network Security Groups (NSGs) in the Cloud with OS-level firewalls (like iptables). *Dono alag level pe kaam karte hain. Cloud NSG pehle traffic block kar dega before it even reaches your Linux/Windows VM.* Always check both!

### 2. Firewall Rule Order (Top-Down Approach)

> [!info] Key Concept
> Firewall rules are processed sequentially from top to bottom. The first rule that matches the traffic is applied, and the rest are ignored.

- **Implicit Deny:** The last rule in any firewall is usually an invisible "Deny All" rule. *Agar traffic kisi bhi upar wale rule se match nahi kiya, toh woh by default block ho jayega.*
- Always place specific rules at the top and general rules at the bottom.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux server (RHEL/CentOS/Ubuntu).
> - Sudo or root access.
> - Basic understanding of TCP/UDP ports.

### Step 1: Checking Firewall Status (Linux)

```bash
# Check if UFW (Ubuntu) is running
sudo ufw status verbose

# Check if firewalld (RHEL/CentOS) is running
sudo firewall-cmd --state

# Check raw iptables rules (sabme chalega)
sudo iptables -L -n -v
```

> [!success] Expected Output
> ```
> running
> ```
> *Yaani firewall active hai aur traffic monitor kar raha hai.*

### Step 2: Allowing a Specific Port (e.g., HTTP / Port 80)

```bash
# UFW mein allow karna
sudo ufw allow 80/tcp

# Firewalld mein allow karna
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```

> [!success] Expected Output
> ```
> success
> ```
> *Ab web server ka traffic allow ho gaya hai.*

### Step 3: Blocking a Specific IP Address

```bash
# UFW se block karna (e.g., hacker IP 192.168.1.100)
sudo ufw deny from 192.168.1.100

# Iptables se sidha block karna (Drop the packet silently)
sudo iptables -I INPUT -s 192.168.1.100 -j DROP
```

> [!success] Expected Output
> *Koi output nahi aayega success pe, par rules list karne pe drop rule dikhega.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `telnet <IP> <PORT>` | Checks if a specific port is open and listening remotely. | `telnet 10.0.0.5 443` |
| `nc -zv <IP> <PORT>` | Netcat command to test TCP/UDP connectivity. *Telnet ka accha alternative hai.* | `nc -zv 192.168.1.50 22` |
| `nmap -p <PORT> <IP>` | Port scanning tool to check if a port is open, closed, or filtered by a firewall. | `nmap -p 80,443 10.1.1.10` |
| `traceroute <IP>` / `tracert` | Shows the path packets take to reach the destination. Identifies where traffic is dropping. | `traceroute 8.8.8.8` |
| `tcpdump -i any port <PORT>` | Packet capture tool to see raw traffic hitting the server interface. | `tcpdump -i eth0 port 80 -n` |
| `iptables -F` | **CAUTION:** Flushes (deletes) all iptables rules. *Galti se production mein mat chalana!* | `iptables -F` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Cannot connect to Web Server (Port 80/443), connection times out. | Firewall is silently dropping the packets (DROP rule) or Cloud NSG is blocking it. | Use `telnet` or `nc` to verify. Add an ALLOW rule in the OS firewall and Cloud NSG. |
| Connection refused error when testing a port. | Firewall is allowing traffic, but the application service (e.g., Apache/Nginx) is not running or listening. | Start the service. Check with `netstat -tulpn` or `ss -tulwn`. |
| Internal servers can ping each other, but cannot reach the internet. | NAT (Network Address Translation) is not configured correctly, or outbound firewall rules are restricting traffic. | Check outbound rules. Verify NAT/Masquerading configuration on the edge router/firewall. |
| Traffic is matching the wrong firewall rule. | Rules are processed top-down. A general rule above is matching the traffic before the specific rule below it. | Reorder the firewall rules. Move specific rules to the top. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Developer Cannot Access the New Database Server

> [!example] Ticket
> "Hi Team, I deployed a new PostgreSQL database on Server B (10.0.1.20), but my application on Server A (10.0.1.10) is getting connection timeouts. Please fix."

**L1 Response:** [Pehle kya kare]
L1 engineer pehle Server A pe login karke `telnet 10.0.1.20 5432` run karega. Agar timeout aati hai, toh woh Server B pe login karke check karega ki service running hai (`ss -tulpn | grep 5432`) aur OS firewall rules check karega.

**Escalation Trigger:** [Kab L2 ko pass kare]
Agar Server B pe service running hai, OS firewall (firewalld/iptables) bhi allow kar raha hai, aur Cloud NSG rules bhi sahi hain, par fir bhi traffic drop ho raha hai.

**L2 Resolution:** [L2 steps]
L2 engineer donon servers ke beech ka routing check karega aur `tcpdump` run karega Server B pe.
```bash
sudo tcpdump -i eth0 src 10.0.1.10 and port 5432 -n
```
Agar packets server B tak aa hi nahi rahe hain, toh iska matlab beech mein koi network firewall (Palo Alto / Fortigate) traffic drop kar raha hai. L2 network team ke saath milkar intermediate firewall ke logs (Traffic Logs) check karega aur wahan policy add karwayega.

### 🎫 Scenario 2: External Vendors Reporting Intermittent Connectivity to SFTP

> [!example] Ticket
> "Our external partner company is trying to push files to our SFTP server, but they say the connection drops randomly after 5 minutes."

**L1 Response:** [Pehle kya kare]
L1 verify karega ki SFTP service stable hai aur disk space full toh nahi hai. Logs (`/var/log/secure` or `/var/log/auth.log`) check karega kisi authentication issue ke liye.

**Escalation Trigger:** [Kab L2 ko pass kare]
Agar server side se sab normal dikh raha hai aur connectivity "randomly" drop ho rahi hai, na ki consistently block ho rahi hai.

**L2 Resolution:** [L2 steps]
L2 engineer ko pata hoga ki stateful firewalls "idle connections" ko ek specific time limit ke baad drop kar dete hain (Connection Timeout / Session Timeout). Agar external vendor 5 minute tak koi data send nahi karta, toh firewall session table se entry nikal deta hai.
Fix: L2 network team se bolkar us specific policy ke liye "TCP Timeout" value badhayega (e.g., from 300 seconds to 3600 seconds) ya vendor ko "Keepalive" packets bhejne ko bolega.

### 🎫 Scenario 3: Web Server Compromised / DDoS Attack Mitigation

> [!example] Ticket
> "CRITICAL: Monitoring shows 100% CPU on Web Server 01, and we are seeing millions of requests from a single unknown IP subnet trying to hit the login page."

**L1 Response:** [Pehle kya kare]
L1 jaldi se server pe login karke `top` dekhega aur Apache/Nginx ke access logs (`/var/log/nginx/access.log`) `tail` karega source IPs identify karne ke liye.

**Escalation Trigger:** [Kab L2 ko pass kare]
Immediate escalation as this is a high-severity security incident (potential DDoS or Brute Force). L1 cannot make wide-reaching block rules without approval.

**L2 Resolution:** [L2 steps]
L2 turant us attacking IP subnet ko block karne ke liye firewall rule push karega:
```bash
sudo iptables -I INPUT -s 203.0.113.0/24 -j DROP
```
Agar attack scale bada hai, toh traffic ko upstream WAF (Web Application Firewall) ya Cloudflare pe block karne ke liye Security Team / Network Team ko involve karega. *Local OS firewall ek huge DDoS ko handle nahi kar sakta kyunki network pipe pehle hi choke ho jayegi.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Stateful and Stateless Firewalls?
> **Answer:**
> - **Stateless:** Har packet ko individually check karta hai bina purane packets ki history jaane. Sirf IP aur Port dekhta hai. Fast hota hai par less secure.
> - **Stateful:** Connections ka state (NEW, ESTABLISHED, RELATED) maintain karta hai ek state table mein. Agar TCP handshake complete ho chuka hai, toh return traffic automatically allow ho jata hai. More secure.

==**Exam Tip:** Modern firewalls mostly stateful hi hote hain. Iptables mein `conntrack` module stateful inspection handle karta hai.==

> [!question] Q2: Why does an implicit deny rule exist at the end of a firewall policy?
> **Answer:**
> Security me hum "Zero Trust" model follow karte hain. Iska matlab hai ki sirf wahi traffic allow hona chahiye jisko explicitly allow kiya gaya ho. Implicit deny ensure karta hai ki koi bhi un-matched traffic automatically block ho jaye. *Agar explicit allow nahi hai, toh by default block!*

==**Exam Tip:** Cloud environments (like AWS Security Groups) implicitly deny all inbound traffic by default.==

> [!question] Q3: How do you troubleshoot a firewall issue if `ping` works but `telnet` fails?
> **Answer:**
> `ping` uses ICMP, which is a different protocol than TCP/UDP used by most applications. Agar `ping` chal raha hai matlab Layer 3 routing sahi hai aur server zinda hai. Agar `telnet <IP> <PORT>` fail ho raha hai, toh iska matlab Layer 4 pe (TCP/UDP level) port block hai ya toh local OS firewall pe, cloud NSG pe, ya beech ke kisi network firewall pe. Ya fir service hi run nahi kar rahi.

==**Exam Tip:** Ping success does NOT mean ports are open. Always test the specific application port.==

> [!question] Q4: A user complains they can't access a website. You check the firewall and there is an explicit ALLOW rule for that website. What could be the issue?
> **Answer:**
> Rule order matter karta hai! Firewall top-down approach follow karta hai. Ho sakta hai us ALLOW rule ke upar ek aur rule ho jo us traffic ko pehle hi DENY kar raha ho. Ek baar packet deny wale rule se match ho gaya, toh firewall neeche ke allow rule tak jayega hi nahi.

==**Exam Tip:** Always check the sequence number of rules and move specific rules to the top.==

---
## 🔗 Related Notes

- [[Networking Basics|🌐 Networking Fundamentals]]
- [[Linux System Administration|🐧 Linux Basics]]
- [[Cloud Computing Security|☁️ Cloud NSG and Security Groups]]
