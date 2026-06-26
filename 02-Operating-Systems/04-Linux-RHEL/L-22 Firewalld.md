---
tags: [desktop-support, linux, rhel, L2, firewall, network-security]
aliases: [firewalld]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX / RHEL**

`#complete` `#intermediate` `#rhcsa`

# L-22: Firewalld

> [!abstract] Overview
> Firewalld is the standard dynamic firewall manager daemon on Red Hat Enterprise Linux (RHEL). It provides a stateful packet filtering firewall using Zones to classify network interface trust levels, allowing rules to be updated without dropping active connections. Ek support engineer ko yeh jaanna zaroori hai kyunki server ki local firewall sabse primary defense hoti hai ports open ya malicious traffic ko block karne ke liye.

---
## 🧠 Concept Overview

- **What it is** — Firewalld is a dynamic firewall manager daemon standard on RHEL systems that provides stateful packet filtering using Zones.
- **Why it matters** — A server's local firewall is its primary defense. Support engineers open network ports for newly deployed services, restrict administrative access to trusted management IP subnets, and block malicious scanning traffic in real-time.
- **Where you see this** — Opening port `80/443` for web servers, blocking brute-force SSH attacks, checking active firewall rules, and assigning network interfaces to zones.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Checks firewall status (`systemctl status firewalld`), lists active rules (`firewall-cmd --list-all`), and verifies port availability. |
| **L2** | Configures zone bindings, opens permanent TCP/UDP ports, permits pre-configured system services, and reloads firewall daemons. |
| **L3** | Designs complex packet routing schemes, writes rich rules for custom network logging/rate-limiting, configures IP masquerading (NAT), and troubleshoots panic mode lockouts. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Firewalld aapke Linux server ka security guard hai. Yeh dekhta hai ki kaunsa traffic andar aa sakta hai aur kaunsa bahar jaa sakta hai, zones aur ports ke rules ke hisaab se.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Firewalld** is like a **Building Security System** because...
>
> - **Zones** are like different security checkpoints (Public lobby vs. Employee-only areas).
> - **Ports/Services** are like ID badges allowing access to specific rooms (HTTP/80 is the public cafeteria, SSH/22 is the server room).
> - **Rich Rules** are like the VIP or Banned lists (Allowing only the CEO's IP, or dropping a known hacker's IP).

---
## 🔬 Technical Deep Dive

### 1. The Concept of Firewall Zones

> [!info] Key Concept
> Firewalld organizes network traffic filtering using **Zones** based on the level of trust assigned to network interfaces.

- **`public`**: (Default) For untrusted networks. Allows only selected inbound connections (e.g., SSH).
- **`external`**: For external routers. Used when masquerading (NAT) is enabled to protect internal networks.
- **`dmz`**: For isolated demilitarized zone servers with limited access to internal networks.
- **`work` / `home` / `internal`**: For trusted local networks, permitting more standard services.

> [!danger] Common Mistake
> Assigning interfaces to the `trusted` zone. **WARNING**: All network packets are accepted. Use only for internal loopbacks or isolated testing switches. Never use this to bypass firewall issues quickly.

### 2. Runtime vs. Permanent Configuration

> [!info] Key Concept
> Changes in firewalld can be temporary (Runtime) or permanent.

- **Runtime Configuration**: Changes take effect immediately but **are lost** when the firewalld service is reloaded, restarted, or the server reboots. Great for temporary rule testing.
- **Permanent Configuration**: Changes are written to XML configuration files on disk. They **do not apply** immediately, but survive restarts. To activate permanent rules, you must run `firewall-cmd --reload`.

> [!danger] Common Mistake
> Forgetting to run `--reload` after a permanent change. Always run `firewall-cmd --reload` to copy disk configuration changes into active memory.

### 3. Rich Rules

> [!info] Key Concept
> Rich Rules provide fine-grained, advanced access controls.

While standard rules open a port to everyone, Rich Rules allow you to define combinations of source IP addresses, destination ports, logging, and actions.
*Example Syntax*: `rule family="ipv4" source address="192.168.1.100" port port="22" protocol="tcp" accept`

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A RHEL VM
> - Root or sudo access

### Step 1: View active zones and configuration

```bash
# View active zones and configuration
firewall-cmd --get-active-zones
```

> [!success] Expected Output
> ```
> public
>   interfaces: eth0
> ```

### Step 2: Create a new custom zone named `secureapp`

```bash
# Create a new custom zone named secureapp and reload
firewall-cmd --permanent --new-zone=secureapp
firewall-cmd --reload
```

### Step 3: Add the `http` service and TCP port `9000` permanently to the new zone

```bash
# Add the http service and TCP port 9000 permanently to the new zone
firewall-cmd --permanent --zone=secureapp --add-service=http
firewall-cmd --permanent --zone=secureapp --add-port=9000/tcp
```

### Step 4: Add a rich rule to the `secureapp` zone allowing SSH ONLY from a lab IP (`192.168.50.10`) permanently

```bash
# Add a rich rule to the secureapp zone allowing SSH ONLY from a lab IP
firewall-cmd --permanent --zone=secureapp --add-rich-rule='rule family="ipv4" source address="192.168.50.10" service name="ssh" accept'
```

### Step 5: Reload the firewall to apply changes and Verify

```bash
# Reload the firewall to apply changes
firewall-cmd --reload

# Verification: List the configuration of the custom zone
firewall-cmd --zone=secureapp --list-all
```

> [!success] Expected Output
> ```
> secureapp
>   target: default
>   icmp-block-inversion: no
>   interfaces: 
>   sources: 
>   services: http
>   ports: 9000/tcp
>   protocols: 
>   forward: no
>   masquerade: no
>   forward-ports: 
>   source-ports: 
>   icmp-blocks: 
>   rich rules: 
>         rule family="ipv4" source address="192.168.50.10" service name="ssh" accept
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `firewall-cmd --state` | Check if firewalld is active and running | `firewall-cmd --state` |
| `firewall-cmd --list-all` | List all active configurations for the default zone | `firewall-cmd --list-all` |
| `firewall-cmd --set-default-zone` | Change the default zone | `firewall-cmd --set-default-zone=work` |
| `firewall-cmd --add-service` | Add a service temporarily (Runtime only) | `firewall-cmd --add-service=http` |
| `firewall-cmd --permanent --add-port` | Add a port permanently | `firewall-cmd --permanent --add-port=8080/tcp` |
| `firewall-cmd --reload` | Reload the firewall to apply all permanent configurations | `firewall-cmd --reload` |
| `firewall-cmd --permanent --add-rich-rule` | Add a Rich Rule permanently | `firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="198.51.100.0/24" reject'` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Service connection timeout** | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall using `firewall-cmd --add-port` |
| **Changes not applying after server reboot** | Forgetting to add the `--permanent` flag when configuring firewall rules | Always use `--permanent` flag and run `firewall-cmd --reload` to apply |
| **Locked out of a remote server after bad firewall rule** | Active attack or misconfiguration blocking legitimate traffic | Use the Panic Mode switch: `firewall-cmd --panic-on` (cuts off all traffic). Then fix rules and `firewall-cmd --panic-off`. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: New Web Service on Port 8080 is Unreachable Externally

> [!example] Ticket
> "The application runs locally, and I can curl it from the server console. However, when I try to open 'http://server-ip:8080' from my laptop, the connection times out."

**L1 Response:** Verify application port status using `ss -tunlp | grep 8080`. Query active firewalld rules using `firewall-cmd --list-all`. Confirm port 8080 is blocked.
**Escalation Trigger:** Pass to L2 to modify the permanent firewall rules.
**L2 Resolution:** Add the port rule permanently to the default public zone (`firewall-cmd --permanent --add-port=8080/tcp`) and reload the firewall (`firewall-cmd --reload`). Verify connection from developer workstation.

### 🎫 Scenario 2: Blocking a Brute-Force SSH Attack on a Public Linux Server

> [!example] Ticket
> "A Linux server hosted in our cloud environment is experiencing a high volume of failed SSH login attempts from IP address 198.51.100.42, risking account compromises."

**L1 Response:** Audit the secure log (`tail -n 20 /var/log/secure | grep "Failed password"`) to confirm the attack IP.
**Escalation Trigger:** Pass to L2/L3 to immediately block the malicious IP.
**L2 Resolution:** Add a rich rule to drop all traffic from the malicious IP address immediately (Runtime first to cut connection, then permanent): `firewall-cmd --add-rich-rule='rule family="ipv4" source address="198.51.100.42" drop'` and `firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="198.51.100.42" drop'`.

---
## 🎤 Interview Questions

> [!question] Q1: How do you verify if a port rule will persist across a firewalld reload?
> **Answer:** I check if the rule was written with the `--permanent` flag. I run `firewall-cmd --permanent --list-all` to inspect the rules stored on disk. If the port is listed there, it will persist after running `firewall-cmd --reload`.

> [!question] Q2: What is the difference between running a firewall-cmd command with and without the '--permanent' flag?
> **Answer:** Without the `--permanent` flag, the change is applied immediately to the runtime memory but is deleted when the firewall is reloaded or the server reboots. With the `--permanent` flag, the change is written to configuration files on disk; it survives reboots but does not apply until you run `firewall-cmd --reload`.

> [!question] Q3: A database server needs to accept connections on port 3306 (MySQL) but only from our application server (IP: 10.0.1.15). How do you configure this?
> **Answer:** I implement a Rich Rule in the default zone. I run: `firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.1.15" port port="3306" protocol="tcp" accept'`, then `firewall-cmd --reload`.

==**Exam Tip:** Permanent rules do not take effect until `firewall-cmd --reload` is executed!==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/TCP-IP-and-Ports|TCP/IP and Ports]] — Underpins the port protocols filtered by firewalld.
- [[02-Operating-Systems/04-Linux-RHEL/Systemctl|Systemctl]] — The service manager that controls the firewalld daemon.
- [[04-Cloud-and-Security/09-Security/Zero-Trust|Zero Trust]] — The identity security model utilizing local host firewalls.
