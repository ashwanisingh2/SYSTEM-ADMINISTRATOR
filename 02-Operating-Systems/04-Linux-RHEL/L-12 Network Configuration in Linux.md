---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-12-network-configuration-in-linux, l-12]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

# L-12: Network Configuration in Linux

> [!abstract] Overview
> This note covers Linux enterprise networking using NetworkManager (`nmcli`/`nmtui`), static IP configuration, bonding, FirewallD rule management, and SELinux policy configurations and audits.

---

---
## Concept Overview
Think of network configuration in Linux like securing and directing traffic in a high-security embassy building:
- **NetworkManager (`nmcli`)** is the civil engineer who sets up physical connections, configures gateways, and assigns office phone extensions (Static IPs).
- **`/etc/resolv.conf`** is the speed dial phone directory pointing to the local directory service (DNS).
- **FirewallD** is the security guards standing at the gates (Zones). They check the type of vehicle (Service/Port) and block unapproved incoming visitors.
- **SELinux (Security-Enhanced Linux)** is the internal security clearance protocol. Even if the firewall let you in, and the filesystem allows you to open a file, if your security clearance badge (SELinux Context Label) does not match the file's label, the protocol blocks you (SELinux denial).


---

---
## Technical Deep Dive
### 1. NetworkManager Command-Line (nmcli)
NetworkManager manages interfaces (Devices) and configuration files (Connections).
- **`nmcli device status`** — List physical network interfaces.
- **`nmcli connection show`** — List active network configuration profiles.
- **Configuring a Static IP:**
  ```bash
  nmcli connection add type ethernet con-name static-eth0 ifname eth0 ip4 192.168.10.20/24 gw4 192.168.10.1
  nmcli connection modify static-eth0 ipv4.dns "8.8.8.8 8.8.4.4"
  nmcli connection modify static-eth0 ipv4.method manual
  nmcli connection up static-eth0
  ```

### 2. Low-Level IP Diagnostics (ip tool)
Replaces the deprecated `ifconfig` and `route` commands:
- **`ip addr show`** — View active IP configurations.
- **`ip link set eth0 up`** — Enable interface port link.
- **`ip route show`** — View the routing table (shows gateway paths).

### 3. DNS Configuration files
- **`/etc/hosts`:** Static local hostname-to-IP lookup file. Checked before DNS servers.
- **`/etc/resolv.conf`:** Holds the nameserver IP addresses used for DNS resolution:
  `nameserver 8.8.8.8`
  - *Warning:* In modern systems, this file is managed by NetworkManager. Manual edits will be overwritten unless configured as write-locked.

### 4. Network Bonding / Teaming
Combines multiple physical NICs into a single logical channel interface for redundancy and load balancing.
- **Modes:** Active-Backup (failover) or LACP/802.3ad (link aggregation).
- **nmcli configuration:**
  ```bash
  nmcli connection add type bond con-name bond0 ifname bond0 mode active-backup
  nmcli connection add type ethernet con-name bond0-port1 ifname eth1 master bond0
  nmcli connection add type ethernet con-name bond0-port2 ifname eth2 master bond0
  ```

### 5. FirewallD Management (firewall-cmd)
FirewallD works with **Zones** (trust levels). The default zone is **public**.
- **Commands Reference:**
  ```bash
  firewall-cmd --get-default-zone               # View active default zone
  firewall-cmd --list-all                      # List all allowed services and ports in active zone
  firewall-cmd --permanent --add-service=http  # Enable HTTP service permanently (port 80)
  firewall-cmd --permanent --add-port=8080/tcp # Enable custom TCP port 8080
  firewall-cmd --reload                        # Reload rules without dropping connections
  ```

### 6. SELinux (Security-Enhanced Linux)
SELinux enforces MAC (Mandatory Access Control) using security labels.
- **States:**
  - **Enforcing (Default):** SELinux blocks unauthorized actions and logs denials.
  - **Permissive:** SELinux does not block actions, but prints denial warnings (used for troubleshooting).
  - **Disabled:** SELinux is completely inactive. Requires reboot to change.
- **Commands:**
  - `getenforce` — Check active state.
  - `setenforce 0` — Set to Permissive. `setenforce 1` — Set to Enforcing.
  - `sestatus` — View detailed SELinux status and policy files.
- **SELinux Context Labels:**
  Format: `user:role:type:sensitivity` (e.g., `system_u:object_r:httpd_sys_content_t:s0`). The **Type** suffix is most critical for sysadmins.
- **Diagnostic Audit Log:**
  Located in `/var/log/audit/audit.log` or `/var/log/messages`. Denial events contain keyword `AVC` (Access Vector Cache).
- **audit2allow Tool:**
  Translates audit denials into custom policy modules:
  ```bash
  grep nginx /var/log/audit/audit.log | audit2allow -M my_nginx_policy
  semodule -i my_nginx_policy.pp
  ```

---

---

### Enterprise RHEL Service & Network Configurations

#### 1. Custom Systemd Service Creation
Create a custom systemd service configuration file `/etc/systemd/system/myapp.service`:
```ini
[Unit]
Description=My Custom Enterprise Application
After=network.target

[Service]
Type=simple
User=sysadmin
ExecStart=/usr/bin/python3 /opt/myapp/server.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
```

#### 2. Log Rotation & Rsyslog Configuration
Configure log rotation rule in `/etc/logrotate.d/myapp` for automatic log cleaning:
```text
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0660 sysadmin sysadmin
}
```
Define Rsyslog rule in `/etc/rsyslog.d/50-myapp.conf` to redirect application logs to a dedicated file:
```text
if $programname == 'myapp' then /var/log/myapp/syslog.log
& stop
```
Restart Rsyslog service:
```bash
sudo systemctl restart rsyslog
```

#### 3. Network Bonding & Teaming (LACP Link Aggregation)
Create a network team interface config file `/etc/sysconfig/network-scripts/ifcfg-team0` (RHEL standard):
```text
DEVICE=team0
DEVICETYPE=Team
BOOTPROTO=none
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
ONBOOT=yes
TEAM_CONFIG='{"runner": {"name": "lacp"}}'
```
Bind slave physical interfaces (e.g., `eth1`) to the team interface:
```text
# /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
ONBOOT=yes
TEAM_MASTER=team0
DEVICETYPE=TeamPort
```

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> A running Rocky Linux VM with one ethernet interface (`eth0`) and root privileges.

### Step 1: Configure a Static IP Address
1. Log in as root. Identify your active connection profile:
   ```bash
   nmcli connection show
   ```
   Assume the connection name is `System eth0`.
2. Configure static parameters:
   ```bash
   nmcli connection modify "System eth0" ipv4.addresses 192.168.10.50/24
   nmcli connection modify "System eth0" ipv4.gateway 192.168.10.1
   nmcli connection modify "System eth0" ipv4.dns "8.8.8.8"
   nmcli connection modify "System eth0" ipv4.method manual
   ```
3. Apply changes:
   ```bash
   nmcli connection up "System eth0"
   ```
4. Verify using the ip tool:
   ```bash
   ip addr show eth0
   ```

### Step 2: Configure FirewallD Rules
1. Allow secure web traffic (HTTPS) and port 8080 permanently:
   ```bash
   firewall-cmd --permanent --add-service=https
   firewall-cmd --permanent --add-port=8080/tcp
   ```
2. Reload firewall configuration:
   ```bash
   firewall-cmd --reload
   ```
3. Verify applied rules:
   ```bash
   firewall-cmd --list-all
   ```
   **Verify:** Confirm `https` is listed under services, and `8080/tcp` is listed under ports.

### Step 3: Troubleshoot a Custom SELinux Port Denial
1. If you configure NGINX to listen on port 8080, it will crash on start because SELinux blocks it.
2. Query the approved ports for HTTP daemon in SELinux:
   ```bash
   semanage port -l | grep http_port_t
   ```
   Note that port 8080 is listed, but if you want to use port 8088, it is missing.
3. Add port 8088 to the HTTP server approved policy:
   ```bash
   semanage port -a -t http_port_t -p tcp 8088
   ```
4. Start NGINX bound to port 8088; it will now start successfully without denials.

---

---
## Cheat Sheet / Quick Reference
| Command / Configuration | Scope | Purpose / Example |
|---|---|---|
| `systemctl status <service>` | Linux | Check status of system service |
| `ip address show` | Linux | Display local interface network details |
| `Get-Service` | PowerShell | Verify service status on Windows hosts |
| `Test-NetConnection` | PowerShell | Check network path connectivity to target ports |

---
## Troubleshooting
| Problem | Cause | Fix | Command |
|---|---|---|---|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall | `ping -c 4 <ip>` / `nc -zv <ip> <port>` |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password | N/A |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects | N/A |

---
## Interview Questions
> [!question] L1 Question
> **Q:** How do you verify if the target service is running?
> **A:** On Linux, I would execute `systemctl status <service-name>`. On Windows, I would run `Get-Service <service-name>` in PowerShell or check Services.msc.

> [!question] L2 Question
> **Q:** Explain how you would troubleshoot a network connectivity issue to a remote server.
> **A:** I would verify local IP configuration, test routing gateway using `ping`, trace hops using `traceroute` or `tracert`, and check port accessibility using `telnet` or `Test-NetConnection` on target port.

---
## Seedha Simple Mein
*Seedha simple mein: Linux mein network manage karne ke liye hum `nmcli` (command line) aur `nmtui` (text GUI) use karte hain. Firewalld traffic ports control karta hai. SELinux kernel-level security module hai jo process contexts ke basis par access control karta hai.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Base diagnostic testing commands.
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Inspecting audit logs using journalctl.
- [[02-Operating-Systems/04-Linux-RHEL/L-09 SSH Configuration and Security|L-09 SSH Configuration and Security]] — Opening custom SSH ports.
