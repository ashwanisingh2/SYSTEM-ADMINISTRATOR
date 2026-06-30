---
tags: [desktop-support, linux-rhel, security, selinux, L2]
aliases: [selinux-apparmor, linux-mac]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

# L-24: SELinux and AppArmor (Mandatory Access Control)

> [!note] Overview
> This note covers Mandatory Access Control (MAC) systems in Linux, focusing on SELinux (RHEL/Rocky) and AppArmor (Ubuntu/Debian). It details security contexts, policy enforcement, booleans configuration, audit log analysis, and custom profile troubleshooting.

---
## Concept Overview
- **What it is** — SELinux and AppArmor are Linux Kernel security modules providing Mandatory Access Control (MAC). They restrict processes (like web servers or databases) to only access resources explicitly allowed by security policies, bypassing Discretionary Access Control (DAC) like `chmod` and `chown`.
- **Why it matters** — In standard Linux, if a web server running as root is hacked, the attacker gains full system access. MAC acts as an inner cage: even if a process is compromised or runs as root, it is blocked from accessing files outside its defined profile, mitigating system-wide exploits.
- **Real job encounter** — Fixing "Permission Denied" errors when moving application directories to non-standard mount paths, configuring network connections for local daemons, and auditing security logs for unauthorized activity.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Check active MAC modes (`getenforce` or `aa-status`), view security labels (`ls -Z` or `ps -Z`), and extract log entries from `/var/log/audit/audit.log` for escalation.
  - **Escalation Trigger**: Escalate if standard file permissions are correct but application access is denied, suggesting a MAC policy block.
  - **L2 Resolution**: Toggle policy modes (Enforcing/Permissive), configure file context rules using `semanage` and `restorecon`, toggle policy switch booleans, and generate policies from logs using `audit2allow`.
  - **L3 Resolution**: Author custom SELinux policy modules (`.te`), write AppArmor profiles in `/etc/apparmor.d/`, and design server hardening standards for cloud container hosts.

---
## Technical Deep Dive

### 1. DAC vs. MAC Security Models
- **Discretionary Access Control (DAC)**: Security based on user ownership. The owner of a file decides who has read/write/execute permissions. If a process runs under user `apache`, it can access *any* file that user `apache` has permissions to, including `/tmp` or shared system areas.
- **Mandatory Access Control (MAC)**: Security governed by system-wide policies. The operating system restricts access based on labels. Even if a file is marked `777` (world readable/writable) under DAC, the MAC policy will block a process from accessing it unless the process label matches the target file label.

### 2. SELinux Architecture (RHEL/Rocky)
SELinux uses labels to determine access. The label format is `user:role:type:level`. The **Type** field (Type Enforcement) is the most critical for system administrators:
- **Process Type (Domain)**: E.g., Apache runs under the `httpd_t` domain.
- **File Type**: E.g., Web pages are labeled `httpd_sys_content_t`.
- **Enforcement Rule**: The system only allows processes labeled `httpd_t` to read files labeled `httpd_sys_content_t`. If Apache tries to read a home folder labeled `user_home_t`, the kernel blocks it.

#### SELinux Modes
- **Enforcing**: Security policies are active and violations are blocked and logged.
- **Permissive**: Policies are audited but not enforced. Violations are logged but access is allowed. (Used for troubleshooting).
- **Disabled**: SELinux is completely turned off. (Requires reboot to change).

### 3. AppArmor Architecture (Ubuntu/Debian)
Unlike SELinux, which is based on security labels, AppArmor is **path-based**. You define profile rules directly using the absolute file paths of binaries and allowed directories.
- **Enforce Mode**: Restricts system actions based on profile definitions.
- **Complain Mode**: Permits actions but logs violations.

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
> - A Rocky Linux 9 VM (`Rocky-Sec01`) running SELinux.
> - Root or sudo access.
> - Apache web server installed: `sudo dnf install -y httpd`.

### Step 1: Audit Current SELinux State
1. Check global SELinux status:
```bash
getenforce
```
**Expected Output:** `Enforcing`

2. Inspect the security contexts of the default web root:
```bash
ls -dZ /var/www/html
```
**Expected Output:** `system_u:object_r:httpd_sys_content_t:s0 /var/www/html`

### Step 2: Trigger an SELinux Block Violation
1. Create a custom web directory outside the default location:
```bash
sudo mkdir /custom_web
sudo echo "<h1>SELinux Test Page</h1>" > /custom_web/index.html
```
2. Configure Apache to use this custom directory:
Edit `/etc/httpd/conf/httpd.conf` and update:
`DocumentRoot "/custom_web"` and `<Directory "/custom_web">`
3. Restart Apache:
```bash
sudo systemctl restart httpd
```
4. Access the site via curl:
```bash
curl http://localhost
```
**Expected Output:** `HTTP/1.1 403 Forbidden` (Even though file permission is correct, SELinux blocks access because `/custom_web` has the default `default_t` context type).

### Step 3: Resolve Context Mismatch using Semanage
1. View the incorrect context on the custom directory:
```bash
ls -dZ /custom_web
```
**Expected Output:** `unconfined_u:object_r:default_t:s0 /custom_web`

2. Apply the correct context rule permanently to the directory and its contents:
```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/custom_web(/.*)?"
```
3. Apply the updated context rules to the filesystem:
```bash
sudo restorecon -Rv /custom_web
```
**Expected Output:** `Relabeled /custom_web from default_t to httpd_sys_content_t...`

4. Test web access again:
```bash
curl http://localhost
```
**Expected Output:** `<h1>SELinux Test Page</h1>`

### Step 4: Manage SELinux Booleans
*Scenario*: Your web app must send outbound network connections, but SELinux blocks it.

1. Check current network connect boolean status:
```bash
getsebool httpd_can_network_connect
```
**Expected Output:** `httpd_can_network_connect --> off`

2. Toggle the boolean to active state permanently:
```bash
sudo setsebool -P httpd_can_network_connect on
```
3. Verify status:
```bash
getsebool httpd_can_network_connect
```
**Expected Output:** `httpd_can_network_connect --> on`

---
## Cheat Sheet / Quick Reference

| Command / Setting | Purpose | Example |
|---|---|---|
| `getenforce` | Displays current SELinux status | `getenforce` |
| `setseforce 0` | Temporarily sets SELinux to Permissive mode | `sudo setenforce 0` |
| `setseforce 1` | Temporarily sets SELinux to Enforcing mode | `sudo setenforce 1` |
| `ls -Z` | Lists files showing active security contexts | `ls -Z /etc/shadow` |
| `semanage fcontext` | Configures permanent file context rule mappings | `semanage fcontext -a -t var_log_t "/my_log(/.*)?"` |
| `restorecon -Rv` | Restores default file contexts based on semanage rules | `sudo restorecon -Rv /path` |
| `getsebool -a` | Lists all active SELinux boolean switches | `getsebool -a \| grep httpd` |
| `setsebool -P` | Sets a boolean state permanently | `sudo setsebool -P httpd_enable_homedirs on` |
| `aa-status` | Displays AppArmor active profiles status (Ubuntu) | `sudo aa-status` |
| `/etc/selinux/config` | File defining boot-level SELinux modes | Config file |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Service fails with "Permission Denied" but `chmod 777` is configured. | SELinux context mismatch blocks access. | Check contexts using `ls -Z`. Change target file type context using `semanage fcontext` and apply with `restorecon`. |
| SELinux blocks a custom app, but no pre-built context type fits. | Custom binary requires unique security rules. | Set SELinux to Permissive: `setenforce 0`. Run application to log operations. Generate custom policy: `grep myapp /var/log/audit/audit.log \| audit2allow -M myapp` and load it: `semodule -i myapp.pp`. Turn Enforcing back on. |
| Cannot configure permanent contexts, error: "semanage command not found." | Policy management tool is missing. | Install requirements: `sudo dnf install -y policycoreutils-python-utils`. |
| AppArmor blocks a service on Ubuntu, console returns: "Access Denied." | Binary profile configuration restricts folder paths. | Set profile to complain mode: `sudo aa-complain /usr/sbin/service_name`. Audit logs in `/var/log/syslog` and update the profile file in `/etc/apparmor.d/`. |
| Re-enabling SELinux from Disabled to Enforcing causes boot hangs. | Unlabeled files on system cause startup blocks. | Force system relabel on boot: `sudo touch /.autorelabel` and reboot system. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** How do you temporarily change the SELinux mode to troubleshoot a permission issue without rebooting?
> **A:** I can use the `setenforce` command. Running `sudo setenforce 0` sets SELinux to **Permissive** mode, which allows all actions to pass but logs violations to the audit log. Running `sudo setenforce 1` restores **Enforcing** mode. I can run `getenforce` to verify the active state.

> [!question] L2 Question
> **Q:** What is the difference between `chcon` and `semanage fcontext` commands when fixing SELinux file contexts?
> **A:** `chcon` modifies the file security context temporarily, but the changes are lost if the filesystem is relabeled or `restorecon` is run. `semanage fcontext` registers the context rule permanently in the system-wide SELinux policy database. The setting persists and is applied automatically during relabels using the `restorecon` command.

> [!question] L3/Scenario Question
> **Q:** A customized legacy web application needs to bind to a non-standard port (TCP 8085), but SELinux is blocking the binding with an access denied log. How do you resolve this permanently while keeping SELinux Enforcing?
> **A:** 
> - **Situation:** SELinux blocks a web service from binding to a non-standard port (TCP 8085).
> - **Task:** Allow port binding permanently without lowering system security.
> - **Action:** 
>   1. Identify target port configuration in policies: `semanage port -l | grep http_port_t`.
>   2. Add the custom port to the HTTP port policy mapping:
>      `sudo semanage port -a -t http_port_t -p tcp 8085`.
>   3. If the port is already owned by another service, I will use the `-m` (modify) flag instead of `-a`.
>   4. Restart the web service to verify binding.
> - **Result:** The web service binds to port 8085 successfully, maintaining SELinux security enforcement.

---
## Seedha Simple Mein
*Seedha simple mein: SELinux aur AppArmor Linux security standards hain. Yeh check karte hain ki koi app (jaise Web Server) kisi aisi directory ko touch na kare jiske liye use specify nahi kiya gaya hai. Agar koi user pure files block rules (chmod 777) bhi kar de, tab bhi SELinux context mismatch hone par process ko block kar deta hai.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-06 File Permissions and Ownership|L-06 File Permissions and Ownership]] — Standard DAC rules (chmod/chown).
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Systemd process management constraints.
- [[02-Operating-Systems/04-Linux-RHEL/L-20 Linux Troubleshooting Masterclass|L-20 Linux Troubleshooting Masterclass]] — Diagnosing permission denials in production.

---
*Tags: #desktop-support #linux-rhel #security #selinux #L2 #cert-rhcsa*
