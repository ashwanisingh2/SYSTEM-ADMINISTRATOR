---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-14-linux-security-hardening, l-14]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# L-14 Linux Security Hardening

> [!abstract] Overview
> This note covers the fundamentals of securing and hardening a Red Hat Enterprise Linux (RHEL) system. We will explore SELinux states, sudoers privilege management, auditing with auditd, intrusion prevention with fail2ban, SSH configuration hardening, file permissions with umask, and kernel parameter tuning via sysctl.

---

---
## Concept Overview
- **What it is** — Linux hardening is the process of reducing a system's vulnerability surface area by securing its settings, restricting permissions, and enabling advanced access controls.
- **Why it matters for a support engineer** — Enterprise servers are target-rich environments. Misconfigurations (like weak SSH settings or broad sudo permissions) are the easiest entry points for attackers.
- **Where you encounter this in real job** — Setting up new RHEL production servers, configuring restricted access for application developers, auditing server access logs, and closing security vulnerabilities reported by InfoSec scans.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Checks SELinux status, resets user passwords, monitors sshd service status, and verifies basic user login failures.
  - **Escalation Trigger:** Escalate to L2 if user requires custom sudo privileges, SELinux policy blockages occur, or audit rules need to be modified.
  - ****L2 Resolution:**** Configures sudoers files, sets SSH hardening policies, writes basic auditd rules, and configures fail2ban jails.
  - ****L3 Resolution:**** Designs enterprise-wide SELinux policies, configures custom kernel tuning (`sysctl.conf`), and integrates auditd logs with SIEM systems (like Sentinel).


---

---
## Technical Deep Dive
### 1. SELinux (Security-Enhanced Linux)
SELinux enforces Mandatory Access Control (MAC), meaning it overrides user file permissions (DAC). Even if root has access to a file, SELinux can block access if the security contexts do not match.
* **SELinux Modes**:
  * **Enforcing**: Policy is enforced. Unauthorized actions are blocked and logged.
  * **Permissive**: Policy is NOT enforced. Actions are allowed, but warnings/violations are logged.
  * **Disabled**: SELinux is completely turned off. (Requires system reboot to change back).
* **SELinux Context**: Every process, file, and port has a context containing `user:role:type:sensitivity`. Type (e.g., `httpd_sys_content_t`) is the most important for application permissions.

### 2. Sudoers Privilege Escalation
Allowing users to run commands as `root` must be strictly regulated.
* The configuration file is `/etc/sudoers`, but it should **never** be edited directly with a normal editor. Instead, use `visudo`, which checks for syntax errors before saving.
* Individual configs are placed in `/etc/sudoers.d/` for modular management.

### 3. SSH Service Hardening
SSH is the primary entry point for remote admins. Hardening includes:
* Disabling root password logins.
* Enforcement of SSH Key authentication.
* Restricting accessible users (`AllowUsers`).
* Changing the default port (22) to a non-standard port to avoid automated brute-force scripts.

### 4. auditd & fail2ban
* **auditd**: The Linux Audit Daemon tracks system calls, file changes, and access requests. It writes logs to `/var/log/audit/audit.log` based on rules defined in `/etc/audit/rules.d/audit.rules`.
* **fail2ban**: A Python service that parses log files (like `/var/log/secure`) and automatically blocks offending IP addresses in the firewall if they exceed login failure thresholds.

### 5. umask & sysctl
* **umask (User Mask)**: Determines default file creation permissions. A umask of `027` means new files get `640` (rw-r-----) and directories get `750` (rwxr-x---), keeping them hidden from unrelated users.
* **sysctl**: Used to configure kernel parameters at runtime. Modifying `/etc/sysctl.conf` can disable IP forwarding, prevent ICMP redirects, and ignore ping requests (ICMP echo) to shield the server.

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
> [!warning] Pre-requisites
> - A running RHEL 8/9 or CentOS Stream VM.
> - Root or sudo access on the machine.
> - Firewall daemon (`firewalld`) active.

### Step 1: Harden SSH Server Configuration
We will disable root logins, change the default port, and allow only specific users.

1. Open SSH configuration:
```bash
sudo vi /etc/ssh/sshd_config
```
2. Modify or add the following lines:
```text
Port 2222
PermitRootLogin no
PasswordAuthentication no
AllowUsers sysadmin
```
3. Update Firewalld to allow the new SSH port:
```bash
sudo firewall-cmd --add-port=2222/tcp --permanent
sudo firewall-cmd --remove-service=ssh --permanent
sudo firewall-cmd --reload
```
4. Update SELinux to permit SSH on port 2222:
```bash
sudo semanage port -a -t ssh_port_t -p tcp 2222
```
5. Restart sshd service:
```bash
sudo systemctl restart sshd
```
**Expected Output:** Verifying the service status (`systemctl status sshd`) shows it listening on port 2222.

---

### Step 2: Configure Limited Sudo Access via visudo
We will configure a user named `developer` to only reload services, without full root access.

1. Run visudo to create a new config chunk:
```bash
sudo visudo -f /etc/sudoers.d/developer
```
2. Write the following rule:
```text
developer ALL=(root) NOPASSWD: /usr/bin/systemctl restart httpd, /usr/bin/systemctl reload httpd
```
3. Save and exit.
**Expected Output:** Sudo validates syntax. The developer user can now run `sudo systemctl restart httpd` without entering root's password.

---

### Step 3: Install and Configure Fail2ban for SSH Protection
We will deploy fail2ban to automatically block brute force attempts on our hardened port.

1. Install EPEL repository and fail2ban:
```bash
sudo dnf install epel-release -y
sudo dnf install fail2ban -y
```
2. Create local configuration file `/etc/fail2ban/jail.local`:
```bash
sudo vi /etc/fail2ban/jail.local
```
3. Add the configuration configuration blocks:
```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/secure
maxretry = 3
bantime = 1h
```
4. Start and enable fail2ban:
```bash
sudo systemctl enable --now fail2ban
```
**Expected Output:** Checking fail2ban client status: `sudo fail2ban-client status sshd` should output active jails and count blocked IPs.

---

### Step 4: Secure Network Parameters using sysctl
We will protect against network attacks by hardening network parameters.

1. Edit `/etc/sysctl.d/99-security.conf`:
```bash
sudo vi /etc/sysctl.d/99-security.conf
```
2. Write the security parameters:
```ini
# Ignore ICMP echo requests (disable ping response)
net.ipv4.icmp_echo_ignore_all = 1

# Disable IP forwarding (if server is not acting as a router)
net.ipv4.ip_forward = 0

# Enable TCP SYN Cookie protection against SYN floods
net.ipv4.tcp_syncookies = 1
```
3. Load the configuration:
```bash
sudo sysctl --system
```
**Expected Output:** Terminal outputs the newly parsed values showing `net.ipv4.icmp_echo_ignore_all = 1`.

---

---
## Cheat Sheet / Quick Reference
| Command | Description | Example |
|---------|-------------|---------|
| `getenforce` | Check current SELinux status | `getenforce` |
| `setenforce [0\|1]` | Toggle SELinux between Enforcing (1) and Permissive (0) | `sudo setenforce 0` |
| `semanage port -l` | List SELinux allowed ports for services | `sudo semanage port -l \| grep ssh` |
| `visudo` | Edit sudoers configuration with syntax checking | `sudo visudo` |
| `sysctl -a` | List all current kernel parameters | `sysctl -a` |
| `fail2ban-client status` | Show active fail2ban configuration status | `sudo fail2ban-client status` |

---

---
## Troubleshooting
| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **SSH service fails to start after port change** | SELinux blocked the new port configuration. | Add the new port to SELinux policy: `semanage port -a -t ssh_port_t -p tcp [new_port]` |
| **Sudo command throws syntax error error** | Direct edit of `/etc/sudoers` contained typos. | Log in as root, run `visudo` to locate the syntax error, correct the line, and save. |
| **Fail2ban not blocking IPs** | Log path incorrect or jail not enabled. | Check `jail.local` configuration logpath matches `/var/log/secure` and service is enabled. |

---

---
## Interview Questions
**Q1: What is the difference between DAC (Discretionary Access Control) and MAC (Mandatory Access Control) in Linux?**
> A: **DAC** controls access based on user/group permissions (e.g., chmod), where the file owner decides access permissions. **MAC** (enforced by SELinux) controls access based on security policies set by system administrators that processes and users cannot override, even if they are root.

**Q2: How do you temporarily change the SELinux mode without rebooting?**
> A: Use `sudo setenforce 1` to switch to Enforcing mode, or `sudo setenforce 0` to switch to Permissive mode. Changes made via setenforce are lost upon reboot. To make them permanent, edit `/etc/selinux/config`.

**Q3: Why should you always use `visudo` instead of editing `/etc/sudoers` directly?**
> A: `visudo` performs lock protection and syntax validation before writing changes. If there is a typo in your sudo configuration, direct editing can lock everyone out of root/sudo capabilities. `visudo` catches these mistakes and prevents saving invalid configurations.

**Q4: How do you check if a specific port is permitted under SELinux rules?**
> A: Use the command `semanage port -l | grep <port_number>` or check by service name, such as `semanage port -l | grep ssh_port_t`.

---

---
## Seedha Simple Mein
*Seedha simple mein: Hardening ka matlab hai server ko surakshit (secure) banana. Faaltu services ko band karna, SSH logins ko safe banana, permissions tight karna, aur monitoring tools active rakhna taaki koi system ke sath chhed-chhad na kar sake.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-09 SSH Configuration and Security]]
- [[02-Operating-Systems/04-Linux-RHEL/L-06 File Permissions and Ownership]]
- [[04-Cloud-and-Security/09-Security/CIA-Triad-and-Zero-Trust]]
