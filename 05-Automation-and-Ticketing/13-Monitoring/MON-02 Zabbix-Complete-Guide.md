---
tags: [desktop-support, automation, monitoring, zabbix, L2]
aliases: [zabbix-guide, zabbix-monitoring]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# MON-02: Zabbix Complete Guide (Enterprise Monitoring)

> [!note] Overview
> This note covers the deployment, configuration, and administration of Zabbix for enterprise-class distributed infrastructure monitoring. It details database-backed schemas, auto-discovery parameters, agent configurations, items/triggers mapping, and proxy scaling topologies.

---
## Concept Overview
- **What it is** — Zabbix is an enterprise-grade distributed monitoring solution. Unlike file-based legacy tools, Zabbix utilizes a central relational database (MySQL/PostgreSQL) to store data, offers a native PHP web dashboard, supports automatic network host discovery, and uses template inheritance to manage configurations at scale.
- **Why it matters** — In corporate networks hosting thousands of endpoints, manually writing config files for every new VM is impossible. Zabbix automatically discovers newly booted servers, installs agents, applies monitoring configurations based on templates, and graphs performance trends automatically.
- **Real job encounter** — Setting up automatic scanning rules to discover new servers, deploying Zabbix agents to Windows endpoints via Active Directory GPO, configuring low disk space triggers, and tracking long-term capacity utilization.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Monitor the Zabbix dashboard, read trigger alerts, acknowledge active problems, ping target hosts to check connectivity, and check local agent service health.
  - **Escalation Trigger**: Escalate if the Zabbix server database runs out of disk capacity, or if major network sectors report host-unreachable statuses simultaneously.
  - **L2 Resolution**: Install and configure Zabbix Agents on target nodes, apply pre-configured templates to new hosts, configure user alert media (emails, SMS), and troubleshoot agent-server communication.
  - **L3 Resolution**: Install and scale Zabbix servers and databases (SQL clustering/performance tuning), deploy Zabbix Proxies to manage remote locations, write custom UserParameters for custom metrics, and configure API integrations.

---
## Technical Deep Dive

### 1. Zabbix Core Architecture
Zabbix is composed of five distinct components:
- **Zabbix Server**: The central engine that processes data, evaluates triggers, runs alerts, and coordinates monitoring schedules.
- **Database Storage**: The backend repository (usually PostgreSQL or MySQL) that stores all configuration settings, templates, historical metric logs (History), and calculated trends.
- **Web Frontend**: A PHP-based web console used to view dashboards, configure monitoring objects, map networks, and generate reports.
- **Zabbix Agent**: A lightweight client utility running on target hosts (Windows, Linux) that collects system metrics (CPU load, disk usage, RAM footprint) and sends them to the server.
- **Zabbix Proxy**: A remote collector agent. It gathers data from regional hosts on behalf of the central server, caches it locally during WAN link drops, and routes it centrally. Essential for securing remote branch firewalls.

```
       [ Client Web Browser ]
                 |
                 v
        [ Zabbix Web Frontend ]
                 |
                 v
        [ Zabbix Server Core ] <---> [ SQL Database ]
                 |
         +-------+-------+
         |               |
         v               v
   [ Local Agent ] [ Zabbix Proxy ] (WAN)
   (Passive/Active)      |
                         v
                   [ Remote Agent ]
```

### 2. Items, Triggers, and Actions
Zabbix logic uses a three-stage workflow:
1. **Item**: A specific metric collected from a host (e.g., `system.cpu.load[percpu,avg1]` or `vfs.fs.size[/,pfree]`).
2. **Trigger**: A logical expression evaluating item data to identify when a system enters an abnormal state (e.g., `last(/TargetHost/vfs.fs.size[/,pfree]) < 10` meaning free root disk space is below 10%). Triggers assign severity classifications (Warning, Average, High, Disaster).
3. **Action**: The automated execution rule defined to run when a trigger transitions into a `PROBLEM` status. Actions can send notification emails/Slack alerts, or execute an **Operation** command to run a script locally (e.g., restart service).

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Rocky Linux 9 server (`SVR-ZAB01` with IP `192.168.10.50`).
> - MariaDB/MySQL database installed and running on the server.
> - Root or sudo privileges on `SVR-ZAB01`.

### Step 1: Install Zabbix Repository
1. Log in to `SVR-ZAB01` and install the official Zabbix repository:
```bash
sudo rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-6.0-4.el9.noarch.rpm
sudo dnf clean all
```

### Step 2: Install Zabbix Server, Web Frontend, and Agent
1. Install package bundles:
```bash
sudo dnf install -y zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
```

### Step 3: Configure MariaDB Database for Zabbix
We must build a database and import the default database tables.

1. Log in to MariaDB as root:
```bash
mysql -u root -p
```
2. Run database creation queries:
```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'ZabbixPassword123!';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
EXIT;
```
3. Import the default database schema:
```bash
zcat /usr/share/collab/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -u zabbix -p'ZabbixPassword123!' zabbix
```
4. Reset global database parameters:
```bash
mysql -u root -p -e "SET GLOBAL log_bin_trust_function_creators = 0;"
```

### Step 4: Configure Zabbix Server Settings
1. Open the main server configuration file:
```bash
sudo vi /etc/zabbix/zabbix_server.conf
```
2. Find and edit the database password entry:
```text
DBPassword=ZabbixPassword123!
```
3. Save the file and exit.

### Step 5: Start Services and Complete Web Setup
1. Enable and restart all required services:
```bash
sudo systemctl restart zabbix-server zabbix-agent httpd php-fpm
sudo systemctl enable zabbix-server zabbix-agent httpd php-fpm
```
2. Open a browser and navigate to `http://192.168.10.50/zabbix`.
3. Complete the Zabbix setup wizard:
   - Database connection: Input Database Password `ZabbixPassword123!`.
   - Zabbix server name: `SVR-ZAB01`.
4. Log in using default credentials:
   - Username: **`Admin`**
   - Password: **`zabbix`**
5. Go to Administration -> Users and immediately change the default password.

### Step 6: Configure Remote Linux Agent (On Target Host)
1. On the target Linux client server (`SVR-WEB01`), install the Zabbix agent:
```bash
sudo rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-6.0-4.el9.noarch.rpm
sudo dnf install -y zabbix-agent
```
2. Edit `/etc/zabbix/zabbix_agentd.conf`:
```text
Server=192.168.10.50       # Zabbix Server IP (For Passive checks)
ServerActive=192.168.10.50 # Zabbix Server IP (For Active checks)
Hostname=SVR-WEB01         # Must match host name in Web UI exactly
```
3. Restart agent service:
```bash
sudo systemctl enable --now zabbix-agent
```

---
## Cheat Sheet / Quick Reference

| Configuration File / Tool | Location / Port | Purpose / Example |
|---|---|---|
| `/etc/zabbix/zabbix_server.conf` | Zabbix Server | Main server settings config file |
| `/etc/zabbix/zabbix_agentd.conf` | Client Host | Zabbix agent daemon configuration file |
| `zabbix_get` | Diagnostics tool | Queries metrics from agent: `zabbix_get -s 192.168.10.30 -k system.cpu.load` |
| `zabbix_sender` | Push agent utility | Sends custom data to server: `zabbix_sender -z 192.168.10.50 -s "Host" -k key -o val` |
| **Port TCP 10050** | Agent Port | Listening port on agent hosts (Server queries this port for passive checks) |
| **Port TCP 10051** | Server Port | Listening port on Zabbix Server (Agents push data to this port for active checks) |
| **`/var/log/zabbix/`** | Log path | Directory containing server and agent log files |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Zabbix server service fails to start, logs show: "Connection refused to database." | Database credentials are wrong, or the database server is offline. | Verify the `DBPassword` value matches the SQL database grant credentials. Run `systemctl status mariadb` to check service state. |
| Host status icon (ZBX) is red: "Get value from agent failed: Connection refused." | Zabbix agent is stopped on client, or firewalls block TCP port 10050. | Start Zabbix Agent service on client. Add inbound port TCP 10050 to the client's local firewall config. |
| Zabbix Agent is active, but server log shows: "Connection refused: host not allowed." | The Zabbix agent configuration lacks the server's IP address. | Edit the client's `/etc/zabbix/zabbix_agentd.conf` and ensure the `Server` parameter includes the exact IP of the Zabbix Server. |
| Zabbix Web dashboard displays warning: "Zabbix Server is not running." | Zabbix server port 10051 is blocked on the server, or SELinux is blocking the web page from connecting. | Configure SELinux: `setsebool -P httpd_can_network_connect on` and verify port 10051 is allowed on local firewall. |
| Host status remains offline: "Hostname does not match." | The `Hostname` parameter in the client configuration file does not match the host name typed in the Web console. | Ensure both names match exactly (including case-sensitivity). Restart the Zabbix agent on the client. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What are the default listening ports used by Zabbix Server and Zabbix Agents?
> **A:** Zabbix Agents use **TCP Port 10050** to listen for incoming data requests from the Zabbix Server (passive checks). Zabbix Server uses **TCP Port 10051** to listen for incoming metric packets pushed by Zabbix Agents (active checks) and Zabbix Proxies.

> [!question] L2 Question
> **Q:** Explain the difference between Active and Passive agent modes in Zabbix.
> **A:** In **Passive Mode** (default), the Zabbix Server initiates connection requests to the Zabbix Agent on port 10050, requests specific metrics, and the agent replies. In **Active Mode**, the Zabbix Agent contacts the Zabbix Server on port 10051, requests the list of items it needs to monitor, collects the metrics locally, and actively pushes the data packets to the server on a schedule, reducing server-side polling load.

> [!question] L3/Scenario Question
> **Q:** You are designing a Zabbix monitoring architecture for a global corporation with three branch offices connected via low-bandwidth VPN tunnels. Pinging hosts over the WAN causes alerts, and WAN link drops interrupt monitoring. How do you design a scalable solution?
> **A:** 
> - **Situation:** Distributed enterprise networks with low-bandwidth, unstable WAN links.
> - **Task:** Design a highly available, low-overhead monitoring architecture.
> - **Action:** I will implement **Zabbix Proxies** in each branch office:
>   1. Deploy a lightweight virtual machine running the Zabbix Proxy service with a local SQLite database in each branch office.
>   2. Configure the local branch agents to check in and send metrics directly to their regional Zabbix Proxy IP.
>   3. The Zabbix Proxy will collect and buffer the metrics locally. It will compress the data and stream it back to the primary Zabbix Server over a single TCP connection.
>   4. Enable database cache on the proxy. If the WAN VPN drops, the proxy will store up to 24 hours of data locally and sync it automatically once the WAN link recovers.
> - **Result:** WAN bandwidth is optimized, firewall rule complexity is minimized, and monitoring remains active during WAN outages.

---
## Seedha Simple Mein
*Seedha simple mein: Zabbix ek advance, enterprise-level monitoring software hai jo company ke saare systems aur devices ko monitor karta hai. Nagios ke opposite, isme data ek central SQL database me store hota hai aur isme autodiscovery features aur templates hote hain jo badhe networks ko maintain karna asan banate hain. Server port 10051 aur agent port 100050 use karte hain.*

---
## Related Notes
- [[05-Automation-and-Ticketing/13-Monitoring/MON-01 Nagios-Setup|MON-01 Nagios-Setup]] — Basic open-source file-based monitoring.
- [[02-Operating-Systems/04-Linux-RHEL/L-18 Linux Performance Monitoring|L-18 Linux Performance Monitoring]] — Local system commands.
- [[05-Automation-and-Ticketing/12-Ansible/ANS-02 Ansible for Linux Admins|ANS-02 Ansible for Linux Admins]] — Automating Zabbix Agent configuration deployments.

---
*Tags: #desktop-support #automation #monitoring #zabbix #L2*
