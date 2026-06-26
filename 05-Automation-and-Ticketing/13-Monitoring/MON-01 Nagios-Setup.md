---
tags: [desktop-support, automation, monitoring, nagios, L2]
aliases: [nagios-setup, nagios-guide]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-red]
> 🤖 **AUTOMATION & MONITORING**

`#complete` `#intermediate` `#none`

# MON-01: Nagios Setup (Open-Source Monitoring)

> [!abstract] Overview
> Yeh note Nagios Core ki setup aur administration cover karta hai. Ek support engineer ko yeh jaanna zaroori hai taaki server health (CPU, Disk) aur services monitor karke downtime se bacha ja sake, aur auto alerts configure ho sakein.

---
## 🧠 Concept Overview

- **What it is** — Nagios Core ek open-source infrastructure monitoring application hai jo servers aur network devices ki health check karti hai.
- **Why it matters** — Manual server check karna possible nahi hai. Nagios automation se disk full ya high CPU ke pehle alert deta hai.
- **Where you see this** — Branch routers ka ping check karna, web servers ka HTTP check, aur database monitoring me.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Nagios web dashboard monitor karna, active alerts acknowledge karna, aur basic ping replies check karna. |
| **L2** | Nagios remote agents (NRPE) install karna, host/service configuration likhna, aur alerts setup karna. |
| **L3** | Nagios Core servers install aur secure karna, custom check plugins likhna, aur escalation paths configure karna. |

> [!tip] Seedha Simple Mein
> *Nagios ek watchman ki tarah hai jo 24/7 aapke servers aur network devices pe nazar rakhta hai. Agar koi service band hoti hai ya disk space bhar jaata hai, toh yeh turant email/SMS alert bhej deta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Nagios** is like an **ICU Heart Monitor** because...
>
> - Just as a heart monitor tracks a patient's pulse and alerts nurses if it drops.
> - Nagios tracks server health (CPU, ping) and alerts admins before the system completely crashes.

---
## 🔬 Technical Deep Dive

### 1. Active vs. Passive Checks

> [!info] Key Concept
> Active checks are initiated by the Nagios server, while Passive checks are run locally by remote hosts and sent to Nagios.

- **Active Checks**: Nagios engine executes a plugin (`check_ping`) to query the device on a schedule.
- **Passive Checks**: Useful for strict firewalls. The remote server runs checks and sends results via NSCA daemon.

> [!danger] Common Mistake
> Relying solely on active checks for servers behind strict NAT firewalls without proper port forwarding.

### 2. Host and Service States

- **Host States**: `UP`, `DOWN`, `UNREACHABLE` (parent switch down).
- **Service States**:
  - `OK` (Exit Code 0): Normal.
  - `WARNING` (Exit Code 1): Threshold crossed.
  - `CRITICAL` (Exit Code 2): Service down/saturated.
  - `UNKNOWN` (Exit Code 3): Plugin error.

**Soft vs. Hard States:**
- **Soft State**: Initial failure, Nagios retries.
- **Hard State**: Retries exhausted, alerts are triggered.

### 3. Nagios Remote Plugin Executor (NRPE)

To monitor internal stats (disk usage) that can't be checked externally:
- Client installs the **NRPE** agent.
- Nagios queries over **TCP 5666**.
- NRPE executes local plugin and returns output.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Rocky Linux 9 server (`SVR-MON01` - `192.168.10.45`) for Nagios Core.
> - Target server (`SVR-WEB01` - `192.168.10.30`) to monitor.
> - Root / Sudo access.

### Step 1: Install Prerequisites

```bash
# Compile dependencies and Apache/PHP
sudo dnf install -y gcc glibc glibc-common gd gd-devel make net-snmp openssl-devel wget httpd php php-cli

# Create users and groups
sudo useradd nagios
sudo groupadd nagcmd
sudo usermod -a -G nagcmd nagios
sudo usermod -a -G nagcmd apache
```

### Step 2: Download and Compile Nagios

```bash
cd /tmp
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.9.tar.gz
tar -xzf nagios-4.4.9.tar.gz
cd nagios-4.4.9

# Configure and compile
./configure --with-command-group=nagcmd --with-httpd-conf=/etc/httpd/conf.d
make all

# Install components
sudo make install
sudo make install-init
sudo make install-commandmode
sudo make install-config
sudo make install-webconf
```

### Step 3: Configure Web Auth & Start

```bash
# Set admin password (e.g. SecurePassword123)
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

# Start services
sudo systemctl enable --now httpd nagios
```

> [!success] Expected Output
> ```
> Web dashboard accessible at http://192.168.10.45/nagios using nagiosadmin.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command / Path | 🛠️ Kya karta hai | 📝 Example |
|------------------|----------------|-----------|
| `/usr/local/nagios/etc/nagios.cfg` | Main configuration file | Main Settings |
| `/usr/local/nagios/etc/objects/` | Directory for host/service definitions | Configs |
| `nagios -v` | Verifies syntax of config files | `nagios -v /usr/local/nagios/etc/nagios.cfg` |
| `check_ping` | Checks connection status via ICMP | `check_ping -H 192.168.10.30 -w 100.0,20% -c 500.0,60%` |
| `check_disk` | Checks storage space | `check_disk -w 10% -c 5% -p /` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Web UI shows "Internal Server Error" | PHP missing | `sudo dnf install -y php php-cli` & restart Apache |
| Nagios fails to start (Config error) | Syntax error in host config | Run `nagios -v` to locate the missing bracket/error |
| Connection refused on TCP 5666 | NRPE stopped or Firewall blocked | Start NRPE, open TCP 5666 in `firewalld` |
| Host `DOWN` but ping works locally | Firewall blocks ICMP or bad threshold | Check Nagios server firewall & `check_ping` params |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Critical Disk Space Alert

> [!example] Ticket
> "Nagios Alert: SVR-WEB01 / disk space is CRITICAL (95% full)"

**L1 Response:** Dashboard verify karega aur check karega ki kaunsa partition full hai.
**Escalation Trigger:** Agar unnecessary logs delete karne ke baad bhi space clear nahi hoti.
**L2 Resolution:** `/var/log` me large files clear karega, ya LVM use karke disk space expand karega.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between a Hard State and a Soft State?
> **Answer:** A **Soft State** is an initial check failure where Nagios will retry. A **Hard State** means all retries failed, and Nagios will now trigger alert notifications.

==**Exam Tip:** NRPE uses port TCP 5666 to communicate. Always remember to allow this in your firewall!==

> [!question] Q2: How does Nagios monitor internal statistics like disk space on a remote Linux server?
> **Answer:** It uses the **NRPE (Nagios Remote Plugin Executor)** agent. The Nagios server queries the NRPE daemon (TCP 5666) on the client, which runs the local check (e.g., `check_disk`) and returns the result.

---
## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Background daemons configuration.
- [[05-Automation-and-Ticketing/13-Monitoring/MON-02 Zabbix-Complete-Guide|MON-02 Zabbix-Complete-Guide]] — Enterprise monitoring alternative.
- [[02-Operating-Systems/04-Linux-RHEL/L-18 Linux Performance Monitoring|L-18 Linux Performance Monitoring]] — Performance check scripts.
