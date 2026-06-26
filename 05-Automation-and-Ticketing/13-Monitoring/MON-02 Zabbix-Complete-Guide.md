---
tags: [desktop-support, automation, monitoring, zabbix, L2]
aliases: [zabbix-guide, zabbix-monitoring]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-red]
> 🤖 **AUTOMATION & MONITORING**

`#complete` `#advanced` `#none`

# MON-02: Zabbix Complete Guide (Enterprise Monitoring)

> [!abstract] Overview
> This note covers the deployment, configuration, and administration of Zabbix for enterprise-class distributed infrastructure monitoring. It details database-backed schemas, auto-discovery parameters, agent configurations, items/triggers mapping, and proxy scaling topologies. Ek support engineer ko yeh jaanna zaroori hai taaki wo servers aur network components ko proactively monitor kar sake aur issues ko outages banne se pehle fix kar sake.

---
## 🧠 Concept Overview

- **What it is** — Zabbix is an enterprise-grade distributed monitoring solution. Unlike file-based legacy tools, Zabbix utilizes a central relational database (MySQL/PostgreSQL) to store data, offers a native PHP web dashboard, supports automatic network host discovery, and uses template inheritance to manage configurations at scale.
- **Why it matters** — In corporate networks hosting thousands of endpoints, manually writing config files for every new VM is impossible. Zabbix automatically discovers newly booted servers, installs agents, applies monitoring configurations based on templates, and graphs performance trends automatically.
- **Where you see this** — Setting up automatic scanning rules to discover new servers, deploying Zabbix agents to Windows endpoints via Active Directory GPO, configuring low disk space triggers, and tracking long-term capacity utilization.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Monitor the Zabbix dashboard, read trigger alerts, acknowledge active problems, ping target hosts to check connectivity, and check local agent service health. |
| **L2** | Install and configure Zabbix Agents on target nodes, apply pre-configured templates to new hosts, configure user alert media (emails, SMS), and troubleshoot agent-server communication. |
| **L3** | Install and scale Zabbix servers and databases (SQL clustering), deploy Zabbix Proxies to manage remote locations, write custom UserParameters for custom metrics, and configure API integrations. |

> [!tip] Seedha Simple Mein
> *Zabbix ek advance, enterprise-level monitoring software hai jo company ke saare systems aur devices ko monitor karta hai. Nagios ke opposite, isme data ek central SQL database me store hota hai aur isme autodiscovery features aur templates hote hain jo bade networks ko maintain karna aasan banate hain. Server port 10051 aur agent port 10050 use karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Zabbix** is like a **High-Tech Hospital Security & Vitals System** because...
>
> - **Zabbix Server (The Doctor's Console):** Central hub processing all patient data and triggering alarms.
> - **Zabbix Database (Medical Records):** Stores all historical patient vitals and health history.
> - **Zabbix Agent (Heart Rate Monitor):** Attached to the patient (Server) to constantly send vital stats to the central hub.
> - **Zabbix Proxy (Regional Clinic):** Gathers vitals from remote patients and sends them in bulk to the main hospital, saving bandwidth.

---
## 🔬 Technical Deep Dive

### 1. Zabbix Core Architecture

> [!info] Key Concept
> Zabbix is composed of five distinct components that work together to gather, process, and display metrics.

- **Zabbix Server**: The central engine that processes data, evaluates triggers, runs alerts, and coordinates monitoring schedules.
- **Database Storage**: The backend repository (usually PostgreSQL or MySQL) that stores all configuration settings, templates, historical metric logs, and calculated trends.
- **Web Frontend**: A PHP-based web console used to view dashboards, configure monitoring objects, map networks, and generate reports.
- **Zabbix Agent**: A lightweight client utility running on target hosts (Windows, Linux) that collects system metrics and sends them to the server.
- **Zabbix Proxy**: A remote collector agent. It gathers data from regional hosts on behalf of the central server, caches it locally during WAN link drops, and routes it centrally.

### 2. Items, Triggers, and Actions

Zabbix logic uses a three-stage workflow:
1. **Item**: A specific metric collected from a host (e.g., `system.cpu.load[percpu,avg1]`).
2. **Trigger**: A logical expression evaluating item data to identify when a system enters an abnormal state. Triggers assign severity classifications.
3. **Action**: The automated execution rule defined to run when a trigger transitions into a `PROBLEM` status. Actions can send notification emails/Slack alerts, or execute an operation.

> [!danger] Common Mistake
> Forgetting to update the `Hostname=` parameter in the client's `zabbix_agentd.conf` to exactly match the host name configured in the Zabbix Web UI. It is case-sensitive!

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Rocky Linux 9 server (`SVR-ZAB01` with IP `192.168.10.50`).
> - MariaDB/MySQL database installed and running on the server.
> - Root or sudo privileges on `SVR-ZAB01`.

### Step 1: Install Zabbix Repository

```bash
# Add the official Zabbix 6.0 repository
sudo rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-6.0-4.el9.noarch.rpm
sudo dnf clean all
```

### Step 2: Install Server & Agent Packages

```bash
# Install Zabbix server, frontend, database scripts, and agent
sudo dnf install -y zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
```

### Step 3: Configure Database

```sql
# Run these in MySQL shell to create database and user
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'ZabbixPassword123!';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
EXIT;
```

```bash
# Import default schema to the zabbix database
zcat /usr/share/collab/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -u zabbix -p'ZabbixPassword123!' zabbix
```

### Step 4: Configure Zabbix Server Settings

```bash
# Edit server configuration file
sudo vi /etc/zabbix/zabbix_server.conf
```

Find and update the password:
```text
DBPassword=ZabbixPassword123!
```

### Step 5: Start Services

```bash
# Enable and start services
sudo systemctl restart zabbix-server zabbix-agent httpd php-fpm
sudo systemctl enable zabbix-server zabbix-agent httpd php-fpm
```

> [!success] Expected Output
> Web dashboard should now be accessible at `http://192.168.10.50/zabbix` (Default Login: `Admin` / `zabbix`).

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `zabbix_get` | Diagnostics tool - queries metrics from agent | `zabbix_get -s 192.168.10.30 -k system.cpu.load` |
| `zabbix_sender` | Push agent utility - sends custom data to server | `zabbix_sender -z 192.168.10.50 -s "Host" -k key -o val` |
| `systemctl restart zabbix-agent` | Agent service ko restart karta hai config change ke baad | `sudo systemctl restart zabbix-agent` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Zabbix server service fails to start** | Database credentials are wrong, or the DB server is offline. | Verify the `DBPassword` in `/etc/zabbix/zabbix_server.conf`. Run `systemctl status mariadb`. |
| **"Get value from agent failed: Connection refused"** | Zabbix agent is stopped on client, or firewall blocks TCP 10050. | Start Zabbix Agent service on client. Add inbound port TCP 10050 to the client's firewall. |
| **"Connection refused: host not allowed"** | The agent config lacks the server's IP address. | Edit `/etc/zabbix/zabbix_agentd.conf` and ensure `Server` parameter includes exact Zabbix Server IP. |
| **Web dashboard warning: "Server is not running"** | Port 10051 is blocked or SELinux is blocking web connection. | `setsebool -P httpd_can_network_connect on` and verify port 10051 on local firewall. |
| **Hostname does not match** | Client config `Hostname` != Web UI hostname. | Ensure both names match perfectly (case-sensitive). Restart agent. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Disk Space Exhaustion Alert

> [!example] Ticket
> "Trigger: Free disk space is less than 20% on volume / (SVR-WEB01)"

**L1 Response:** Log into the server via SSH, run `df -h` to verify the actual free space. Run `du -sh /*` to identify large directories consuming space.
**Escalation Trigger:** If log files or non-critical data cannot be safely deleted or identified.
**L2 Resolution:** Clear old rotated log files in `/var/log` or extend the logical volume (LVM) by expanding the disk. Acknowledge the alert in the Zabbix dashboard once resolved.

---
## 🎤 Interview Questions

> [!question] Q1: What are the default listening ports used by Zabbix Server and Zabbix Agents?
> **Answer:** Zabbix Agents use **TCP Port 10050** to listen for incoming data requests (passive checks). Zabbix Server uses **TCP Port 10051** to listen for incoming metric packets pushed by Agents and Proxies (active checks).

> [!question] Q2: Explain the difference between Active and Passive agent modes.
> **Answer:** In **Passive Mode** (default), the server initiates the connection to the agent and asks for metrics. In **Active Mode**, the agent contacts the server, gets a list of what to monitor, and actively pushes data to the server, reducing server polling load.

> [!question] Q3: How do you monitor a remote branch office with unstable WAN links?
> **Answer:** Deploy a **Zabbix Proxy**. It collects metrics locally from the branch servers, buffers them in a local SQLite database, and sends them in bulk over a single connection. If the WAN drops, it caches data locally and syncs it when the link recovers.

==**Exam Tip:** Remember the port difference: 10050 is for the Agent (Client endpoint), 10051 is for the Server/Proxy (Central hub).==

---
## 🔗 Related Notes

- [[05-Automation-and-Ticketing/13-Monitoring/MON-01 Nagios-Setup|MON-01 Nagios-Setup]] — Basic open-source file-based monitoring.
- [[02-Operating-Systems/04-Linux-RHEL/L-18 Linux Performance Monitoring|L-18 Linux Performance Monitoring]] — Local system commands.
- [[05-Automation-and-Ticketing/12-Ansible/ANS-02 Ansible for Linux Admins|ANS-02 Ansible for Linux Admins]] — Automating Zabbix Agent configuration deployments.
