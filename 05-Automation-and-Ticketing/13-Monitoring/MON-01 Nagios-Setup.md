---
tags: [desktop-support, automation, monitoring, nagios, L2]
aliases: [nagios-setup, nagios-guide]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# MON-01: Nagios Setup (Open-Source Monitoring)

> [!note] Overview
> This note covers the setup, configuration, and administration of Nagios Core for infrastructure monitoring. It details host/service definitions, active vs. passive checks, NRPE agent deployment, email alert configurations, and status diagnostics.

---
## Concept Overview
- **What it is** — Nagios Core is an open-source host and service monitoring application. It queries servers, switches, and databases to check their health, presents status dashboards, and triggers automated email/SMS alerts when system states deviate from normal operating parameters.
- **Why it matters** — Expecting support teams to manually log in to every server to check disk space or CPU loads is inefficient. Nagios automates this polling, warning admins about disk fills or high load averages before a service crashes.
- **Real job encounter** — Setting up ping monitoring for branch routers, configuring service alerts for web servers (HTTP port check), monitoring SQL databases, and receiving alert emails.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Monitor the Nagios web dashboard, acknowledge active alerts (stopping notification loops), verify basic ping replies, and log host DOWN states in the ticketing tool.
  - **Escalation Trigger**: Escalate if critical production hosts remain in a CRITICAL state after basic service restarts, suggesting server hardware or deep OS failures.
  - **L2 Resolution**: Install and configure Nagios remote agents (NRPE/NCPA), write host and service configuration blocks, and set up alert contact notifications.
  - **L3 Resolution**: Install and secure Nagios Core parent servers, write custom bash/python check plugins, integrate Nagios with directory services, and configure notification escalation paths.

---
## Technical Deep Dive

### 1. Active vs. Passive Checks
- **Active Checks**: Initiated by the Nagios server on a configured schedule (e.g. every 5 minutes). The Nagios engine executes a plugin (like `check_ping` or `check_http`) to query the target device.
- **Passive Checks**: Initiated and run by the remote host itself. Useful for servers behind strict firewalls or inside private subnets. The remote server runs the checks locally and sends the results to the Nagios server using the NSCA (Nagios Service Check Acceptor) daemon.

### 2. Host and Service States
Nagios separates target assets into Hosts (IP addresses) and Services (daemons or metrics on the host):
- **Host States**: `UP`, `DOWN`, `UNREACHABLE` (parent switch is down).
- **Service States**:
  - `OK` (Exit Code 0): Everything running normally.
  - `WARNING` (Exit Code 1): Resource utilization has crossed the warning threshold.
  - `CRITICAL` (Exit Code 2): Service is down or resource is saturated.
  - `UNKNOWN` (Exit Code 3): Service check failed or plugin returned errors.
- **Soft vs. Hard States**:
  - **Soft State**: A service check fails once. Nagios retries the check a defined number of times (e.g. 3 times) to rule out transient errors.
  - **Hard State**: The check fails all retry attempts. Nagios confirms the state is Hard and triggers the alert notifications (emails/SMS).

### 3. Nagios Remote Plugin Executor (NRPE)
To monitor internal server statistics (like disk usage or CPU load), the Nagios server cannot check from the outside.
- The remote client installs the **NRPE** agent.
- Nagios server queries NRPE over port **TCP 5666**.
- The NRPE daemon executes the local plugin on the client, collects the output, and sends it back to the Nagios server.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Rocky Linux 9 server (`SVR-MON01` with IP `192.168.10.45`) to host Nagios Core.
> - A target Rocky Linux 9 server (`SVR-WEB01` with IP `192.168.10.30`) to monitor.
> - Root or sudo access on both machines.

### Step 1: Install Prerequisites on the Host Server
1. Log in to `SVR-MON01`. Install compile dependencies:
```bash
sudo dnf install -y gcc glibc glibc-common gd gd-devel make net-snmp openssl-devel wget httpd php php-cli
```
2. Create the default nagios user and group:
```bash
sudo useradd nagios
sudo groupadd nagcmd
sudo usermod -a -G nagcmd nagios
sudo usermod -a -G nagcmd apache
```

### Step 2: Download and Compile Nagios Core
1. Navigate to `/tmp`, download Nagios Core source files, and extract:
```bash
cd /tmp
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.9.tar.gz
tar -xzf nagios-4.4.9.tar.gz
cd nagios-4.4.9
```
2. Configure build files and compile:
```bash
./configure --with-command-group=nagcmd --with-httpd-conf=/etc/httpd/conf.d
make all
```
3. Install binaries, services, and default configurations:
```bash
sudo make install
sudo make install-init
sudo make install-commandmode
sudo make install-config
sudo make install-webconf
```

### Step 3: Configure Web Authentication
1. Generate the admin credential file to restrict access to the web dashboard:
```bash
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
# Enter password: SecurePassword123
```
2. Enable and start Apache and Nagios services:
```bash
sudo systemctl enable --now httpd nagios
```

### Step 4: Install Standard Nagios Plugins
Without plugins, Nagios cannot check any services.

1. Download and extract plugins:
```bash
cd /tmp
wget https://nagios-plugins.org/download/nagios-plugins-2.4.6.tar.gz
tar -xzf nagios-plugins-2.4.6.tar.gz
cd nagios-plugins-2.4.6
```
2. Compile and install:
```bash
./configure --with-nagios-user=nagios --with-nagios-group=nagios
make
sudo make install
```
3. Verify access: Open web browser, navigate to `http://192.168.10.45/nagios` and authenticate with `nagiosadmin`.

### Step 5: Configure Host Definition to Monitor SVR-WEB01
1. Open the configuration file directory:
```bash
cd /usr/local/nagios/etc/objects/
sudo vi SVR-WEB01.cfg
```
2. Insert host and service check definitions:
```text
define host {
    use                     linux-server
    host_name               SVR-WEB01
    alias                   Production Web Server
    address                 192.168.10.30
    max_check_attempts      5
    check_period            24x7
    notification_interval   30
    notification_period     24x7
}

define service {
    use                     generic-service
    host_name               SVR-WEB01
    service_description     SSH Service Check
    check_command           check_ssh
    notification_interval   30
}
```
3. Register the new configuration file in `/usr/local/nagios/etc/nagios.cfg`:
```text
# Add this line in nagios.cfg
cfg_file=/usr/local/nagios/etc/objects/SVR-WEB01.cfg
```
4. Verify configuration syntax:
```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```
**Expected Output:** `Total Warnings: 0`, `Total Errors: 0`.

5. Reload Nagios service:
```bash
sudo systemctl restart nagios
```
Open the web console and verify that `SVR-WEB01` appears in the Host Status dashboard list.

---
## Cheat Sheet / Quick Reference

| File Path / Port | Purpose | Configuration Example / Command |
|---|---|---|
| `/usr/local/nagios/etc/nagios.cfg` | Main configuration file for Nagios | Main Settings |
| `/usr/local/nagios/etc/objects/` | Directory storing host and service definitions | Config Directory |
| `/usr/local/nagios/bin/nagios -v` | Verifies syntax of all configuration files | `nagios -v /etc/nagios.cfg` |
| `/usr/local/nagios/libexec/` | Directory storing compiled check executable plugins | Plugin Binaries path |
| `check_ping` | Checks host connection status using ICMP | `check_ping -H <IP> -w 100.0,20% -c 500.0,60%` |
| `check_disk` | Checks storage space capacity on local drive | `check_disk -w 10% -c 5% -p /` |
| **Port TCP 5666** | Default listening port used by NRPE agents | Firewall config rule |
| **Port TCP 80** | Port used to access the Nagios Web interface | URL access |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Nagios Web UI returns: "Internal Server Error" or displays raw PHP code. | PHP execution engine or php-cli package is missing. | Install PHP packages: `sudo dnf install -y php php-cli`, restart Apache. |
| Nagios fails to start, console logs show: "Configuration errors found." | Syntax error or missing bracket inside the host definition files. | Run syntax validation: `/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg`. Locate line error and fix. |
| Service check returns: "Connection refused or timed out" (Code 5666). | Remote server NRPE agent is stopped, or port TCP 5666 is blocked by firewall. | Start NRPE service on client server. Open port TCP 5666 in local `firewalld` configurations. |
| Host status shows `DOWN` but pinging from target server works. | Nagios server firewall blocks return ICMP echo replies, or ping settings are wrong. | Verify Nagios server firewall configuration. Check the `check_ping` threshold parameters in the host config block. |
| Check returns error: "NRPE: Command not defined." | The requested command in Nagios configuration is missing in client's `/etc/nagios/nrpe.cfg` file. | Edit client's `nrpe.cfg`. Add definition rule: `command[check_disk]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /`. Restart NRPE. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between a Hard State and a Soft State in Nagios?
> **A:** A **Soft State** occurs when a host or service check fails for the first time. Nagios does not send alert notifications immediately. Instead, it schedules retries to rule out temporary glitches. A **Hard State** occurs when the check fails continuously through all configured retry attempts. Once a Hard state is reached, Nagios logs the failure and triggers contact notifications (emails/alerts).

> [!question] L2 Question
> **Q:** How does Nagios monitor internal statistics (like disk space) on a remote Linux server where it cannot run network checks?
> **A:** Nagios uses the **NRPE (Nagios Remote Plugin Executor)** agent. The remote server runs the NRPE daemon listening on port TCP 5666. The Nagios server sends check queries to the NRPE daemon. NRPE executes the requested local plugin script (such as `check_disk`) locally on the client, gathers the stdout results, and sends them back to the Nagios server.

> [!question] L3/Scenario Question
> **Q:** You need to monitor 100 remote POS terminal devices connected via cellular networks with high latency. Pings frequently fail momentarily. How do you configure Nagios to prevent false-positive alert floods?
> **A:** 
> - **Situation:** Monitoring high-latency POS systems experiencing transient drops.
> - **Task:** Configure Nagios to prevent false warning alarms.
> - **Action:** I will tune the host check parameters:
>   1. Increase the **`max_check_attempts`** limit from the default 3 to `10` or `15` to allow for latency spikes before confirming a Hard State.
>   2. Set **`retry_interval`** to `2` minutes and **`normal_check_interval`** to `10` minutes.
>   3. Modify the ping check timeout parameters in the plugin call command: `check_ping -t 30` (extending the timeout limit to 30 seconds before marking packet drops).
>   4. Group the hosts and enable notification delay options.
> - **Result:** Soft states allow transient latency spikes to resolve without alerting, eliminating spam notifications.

---
## Seedha Simple Mein
*Seedha simple mein: Nagios ek monitoring tool hai jo servers, databases aur network systems ki health ko track karta hai. Yeh check karta hai ki machines working hain ya nahi (UP/DOWN status) aur server resources (CPU, Disk) limits cross hone par email alerts bhejta hai. Remote check execution ke liye `NRPE` service use hoti hai.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Background daemons configuration.
- [[05-Automation-and-Ticketing/13-Monitoring/MON-02 Zabbix-Complete-Guide|MON-02 Zabbix-Complete-Guide]] — Enterprise monitoring alternative.
- [[02-Operating-Systems/04-Linux-RHEL/L-18 Linux Performance Monitoring|L-18 Linux Performance Monitoring]] — Performance check scripts.

---
*Tags: #desktop-support #automation #monitoring #nagios #L2*
