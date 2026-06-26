---
tags: [linux, rhel, firewalld, security, networking]
aliases: [firewalld-config, linux-firewall]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX / RHEL**

`#complete` `#intermediate` `#rhcsa`

# L-22: Firewalld

> [!abstract] Overview
> Firewalld is the default firewall management tool in RHEL-based systems. It acts as a frontend for nftables/iptables, providing dynamic firewall management with support for network zones. *Yeh note network traffic control, zones aur port management cover karta hai.* Ek system admin ko firewalld zaroor aana chahiye kyunki server security aur access control isike through manage hota hai.

---
## 🧠 Concept Overview

- **What it is** — Firewalld is a dynamic firewall manager for Linux operating systems. It uses "zones" and "services" rather than complex iptables chains and rules.
- **Why it matters** — Server pe kaunsa traffic allowed hai aur kaunsa blocked, yeh firewalld decide karta hai. Bina iske, aapka server open to internet (unsecured) ho sakta hai.
- **Where you see this** — Web server (port 80/443) ya SSH (port 22) access allow karte waqt, database access restrict karte waqt.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Status check karna, basic ports allow/block karna, rules list karna. |
| **L2** | Custom zones create karna, rich rules likhna, port forwarding setup karna. |
| **L3** | Enterprise firewall architecture design, complex NAT rules, integration with IDPS. |

> [!tip] Seedha Simple Mein
> *Firewalld ek security guard ki tarah hai jo server (building) ke alag-alag gates (ports) par khada hai. Woh decide karta hai ki kaun andar aayega (allow) aur kaun bahar jayega (block).*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Firewalld Zones** is like **Airport Security Zones** because...
>
> - **Public Zone:** Airport ka entry gate jahan har koi aa sakta hai (only specific services allowed like HTTP).
> - **Drop Zone:** Security check jahan unauthorized log bina soche bahar nikal diye jaate hain (all incoming traffic dropped).
> - **Trusted Zone:** VIP lounge jahan sab allow hai bina checking ke (all traffic from trusted IPs allowed).
> - **Internal Zone:** Staff-only area jahan strictly defined rules hote hain.

---
## 🔬 Technical Deep Dive

### 1. Firewalld vs Iptables

> [!info] Key Concept
> Firewalld is dynamic. It applies changes immediately without restarting the firewall service or dropping active connections, unlike old `iptables` scripts.

Firewalld uses two configuration sets:
1. **Runtime Configuration:** Jo currently memory mein active hai. Reboot ya service restart pe flush ho jaati hai.
2. **Permanent Configuration:** Jo disk pe save hoti hai. Next reboot ya reload par apply hoti hai.

*Hamesha yaad rakho ki agar aap permanent rule add kar rahe ho, toh usko active karne ke liye `firewall-cmd --reload` chalana padta hai.*

> [!danger] Common Mistake
> Admin rule add kar dete hain but `--permanent` flag bhool jaate hain. Agle din server reboot hota hai aur rules gayab! Ya fir `--permanent` lagate hain par `--reload` bhool jaate hain, jisse rule turant apply nahi hota.

### 2. Zones in Firewalld

Zones define the trust level of network connections or interfaces.

- **drop:** Any incoming network packets are dropped, there is no reply. Only outgoing network connections are possible.
- **block:** Incoming network connections are rejected with an icmp-host-prohibited message.
- **public:** Represents public, untrusted networks. You don't trust other computers but may allow selected incoming connections on a case-by-case basis. (Default zone)
- **trusted:** All network connections are accepted.

### 3. Rich Rules

Rich rules allow you to create more complex firewall rules in a very expressive way. You can log, audit, accept, drop, or reject traffic based on various conditions.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - RHEL/CentOS 8/9 system
> - Root privileges (`sudo`)
> - Network interface (e.g., `eth0` or `ens160`)

### Step 1: Install and Enable Firewalld

```bash
# Verify if installed, install if not
sudo dnf install firewalld -y

# Start the service
sudo systemctl start firewalld

# Enable it to start on boot
sudo systemctl enable firewalld
```

> [!success] Expected Output
> ```
> Created symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service → /usr/lib/systemd/system/firewalld.service.
> Created symlink /etc/systemd/system/multi-user.target.wants/firewalld.service → /usr/lib/systemd/system/firewalld.service.
> ```

### Step 2: Check Firewall Status and Zones

```bash
# Check if running
sudo firewall-cmd --state

# Get the default zone
sudo firewall-cmd --get-default-zone

# List everything in the default zone
sudo firewall-cmd --list-all
```

### Step 3: Allowing a Service (e.g., HTTP)

```bash
# Add HTTP service permanently
sudo firewall-cmd --zone=public --add-service=http --permanent

# Reload to apply the permanent configuration
sudo firewall-cmd --reload

# Verify
sudo firewall-cmd --zone=public --list-services
```

### Step 4: Port Forwarding (NAT)

*Hum port 8080 ka traffic port 80 pe redirect karenge.*

```bash
# Enable masquerading (required for port forwarding)
sudo firewall-cmd --zone=public --add-masquerade --permanent

# Add forward port rule
sudo firewall-cmd --zone=public --add-forward-port=port=8080:proto=tcp:toport=80 --permanent

sudo firewall-cmd --reload
```

### Step 5: Adding a Rich Rule

*Sirf ek specific IP (192.168.1.100) ko SSH (port 22) allow karna hai, baaki sabko block karna hai.*

```bash
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port port="22" protocol="tcp" accept' --permanent
sudo firewall-cmd --reload
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `firewall-cmd --state` | Checks if firewalld is running | `firewall-cmd --state` |
| `firewall-cmd --reload` | Reloads rules without dropping connections | `firewall-cmd --reload` |
| `firewall-cmd --list-all` | Shows all active rules in current zone | `firewall-cmd --list-all` |
| `firewall-cmd --get-zones` | Lists all available zones | `firewall-cmd --get-zones` |
| `firewall-cmd --add-port` | Opens a specific port | `firewall-cmd --add-port=8080/tcp --permanent` |
| `firewall-cmd --add-service` | Opens default ports for a service | `firewall-cmd --add-service=https --permanent` |
| `firewall-cmd --remove-port` | Closes a specific port | `firewall-cmd --remove-port=8080/tcp --permanent` |
| `firewall-cmd --add-source` | Adds IP to a specific zone | `firewall-cmd --zone=trusted --add-source=10.0.0.5/32 --permanent` |
| `firewall-cmd --panic-on` | Drops all network packets immediately | `firewall-cmd --panic-on` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Added a rule but it's not working | Rule was added permanently but not reloaded | Run `sudo firewall-cmd --reload` |
| Rebooted and all rules vanished | Rules were added without `--permanent` flag | Re-add rules with `--permanent` flag and reload |
| Cannot SSH to the server | SSH service or port 22 is missing in the active zone | Add ssh service using `sudo firewall-cmd --add-service=ssh --permanent` then reload |
| `firewall-cmd` returns "FirewallD is not running" | The firewalld service is stopped or crashed | Run `systemctl start firewalld` and check `systemctl status firewalld` |
| Traffic from a specific IP is blocked | The IP is in the drop zone or a rich rule is blocking it | Check rich rules with `firewall-cmd --list-rich-rules` and remove if necessary |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Web Server Unreachable

> [!example] Ticket
> "Hi IT Support, I just deployed a new Nginx web server on our RHEL 8 VM, but the website is not loading from the outside network. Local curl works fine."

**L1 Response:** *Check karenge ki port 80/443 open hai ya nahi using `firewall-cmd --list-ports` aur `firewall-cmd --list-services`.*
**Escalation Trigger:** Agar port open hai fir bhi connect nahi ho raha (routing issue).
**L2 Resolution:** L2 admin will ensure that `firewalld` has the HTTP/HTTPS services allowed in the correct zone (`public` or custom).
```bash
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-service=https --permanent
firewall-cmd --reload
```

### 🎫 Scenario 2: Restrict Database Access

> [!example] Ticket
> "Security Team requirement: Our MySQL database server (port 3306) should only be accessible from the App Server IP (10.1.2.50). Block all other incoming connections to 3306."

**L1 Response:** Ticket ko acknowledge karenge aur L2 Security/System Admin ko assign karenge kyunki yeh restrictive access ka requirement hai.
**Escalation Trigger:** Immediate L2 task.
**L2 Resolution:** L2 engineer rich rule apply karega.
```bash
# Add rich rule for specific IP
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="10.1.2.50" port port="3306" protocol="tcp" accept' --permanent
# Ensure global mysql service is not allowed
firewall-cmd --zone=public --remove-service=mysql --permanent
firewall-cmd --reload
```

### 🎫 Scenario 3: Temporary Access for Vendor

> [!example] Ticket
> "A vendor needs SSH access to server X for exactly 2 hours to perform maintenance."

**L1 Response:** Vendor ki IP address aur server access details verify karenge.
**Escalation Trigger:** Standard L2 request.
**L2 Resolution:** L2 engineer timeout option use karega firewalld mein.
```bash
# Allows SSH access from vendor IP for 7200 seconds (2 hours)
firewall-cmd --add-rich-rule='rule family="ipv4" source address="203.0.113.45" service name="ssh" accept' --timeout=7200
# Note: Timeout rules are runtime only and do not require --permanent or --reload.
```

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between `--permanent` and runtime configurations in firewalld?
> **Answer:** Runtime rules active memory mein hote hain aur immediately apply hote hain but reboot ke baad erase ho jaate hain. `--permanent` rules disk pe save hote hain, but unko active karne ke liye `firewall-cmd --reload` chalana padta hai.

> [!question] Q2: How do you completely lock down a system using firewalld in an emergency?
> **Answer:** We can use the panic mode. Command: `firewall-cmd --panic-on`. This drops all network packets immediately, acting as a network kill switch.

> [!question] Q3: How do you port forward traffic from port 80 to port 8080 using firewalld?
> **Answer:** First, enable masquerading using `firewall-cmd --add-masquerade --permanent`. Then add the forward port rule using `firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080 --permanent`, followed by a reload.

> [!question] Q4: What is a firewalld "zone"?
> **Answer:** A zone defines the trust level for network connections. Based on the zone an interface is assigned to, firewalld determines which traffic to allow or deny. Examples include `public`, `drop`, `trusted`.

==**Exam Tip:** RHCSA exam mein firewall configuration zaroor aati hai. Yaad rakho ki har `--permanent` change ke baad `firewall-cmd --reload` zaroor chalana hai warna test script fail ho jayegi!==

---
## 🔗 Related Notes

- [[L-21 Networking|Networking in Linux]] — Network interface concepts
- [[L-23 SELinux|SELinux Basics]] — For additional system security layers
- [[L-15 SSH Configuration|SSH Security]] — Securing SSH access
