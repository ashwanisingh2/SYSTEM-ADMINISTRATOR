---
tags: [sysadmin, linux, networking, firewalld, selinux]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# L-12: Network Configuration in Linux

> [!abstract] Overview
> This note covers Linux enterprise networking using NetworkManager (`nmcli`/`nmtui`), static IP configuration, bonding, FirewallD rule management, and SELinux policy configurations and audits.

---
## Concept
Think of network configuration in Linux like securing and directing traffic in a high-security embassy building:
- **NetworkManager (`nmcli`)** is the civil engineer who sets up physical connections, configures gateways, and assigns office phone extensions (Static IPs).
- **`/etc/resolv.conf`** is the speed dial phone directory pointing to the local directory service (DNS).
- **FirewallD** is the security guards standing at the gates (Zones). They check the type of vehicle (Service/Port) and block unapproved incoming visitors.
- **SELinux (Security-Enhanced Linux)** is the internal security clearance protocol. Even if the firewall let you in, and the filesystem allows you to open a file, if your security clearance badge (SELinux Context Label) does not match the file's label, the protocol blocks you (SELinux denial).

*Seedha simple mein: Linux mein network manage karne ke liye hum `nmcli` (command line) aur `nmtui` (text GUI) use karte hain. Firewalld traffic ports control karta hai. SELinux kernel-level security module hai jo process contexts ke basis par access control karta hai.*

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
## Lab — Step by Step
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
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Base diagnostic testing commands.
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Inspecting audit logs using journalctl.
- [[02-Operating-Systems/04-Linux-RHEL/L-09 SSH Configuration and Security|L-09 SSH Configuration and Security]] — Opening custom SSH ports.

