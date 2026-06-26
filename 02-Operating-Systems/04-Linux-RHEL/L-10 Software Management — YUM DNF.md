---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-10-software-management-yum-dnf, l-10]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# L-10: Software Management — YUM/DNF

> [!abstract] Overview
> This note covers Red Hat package management, detailing RPM database querying, YUM/DNF dependency resolution, repository configuration, local repo creation, and package extraction techniques.

---

---
## Concept Overview
Think of software management in Linux like ordering groceries for a recipe:
- An **RPM File** is a pre-packaged can of ingredients. It lists its contents, but if the recipe demands another can of sauce that you don't have, the manager stops and says, "Error: missing dependency. Go buy sauce yourself."
- **DNF (or YUM)** is an automated grocery delivery app. You say, "I want to cook NGINX." DNF checks the database, finds NGINX, checks the list of needed ingredients (dependencies), automatically adds the sauce, spices, and pots to your shopping cart, downloads them from the warehouse (Repositories), and cooks the meal for you.
- **`/etc/yum.repos.d/`** is your list of approved grocery stores (Repositories).


---

---
## Technical Deep Dive
### 1. RPM Package Format & Database
- **RPM (Red Hat Package Manager):** A low-level package manager. It installs, uninstalls, and queries `.rpm` software files.
- **RPM Database:** Stored in `/var/lib/rpm`. It tracks every file, dependency, and configuration installed on the system via RPM.
- **RPM Query Commands:**
  - `rpm -qa` — Query all installed packages.
  - `rpm -qi nginx` — Display detailed package info (version, release, description).
  - `rpm -ql nginx` — List all files installed by the package.
  - `rpm -qf /etc/nginx/nginx.conf` — Find which package owns a specific file.

### 2. YUM vs. DNF
- **YUM (Yellowdog Updater, Modified):** The default package manager in RHEL 5, 6, and 7. Based on Python 2.
- **DNF (Dandified YUM):** Replaced YUM in RHEL 8 and 9. 
  - **Advantages:** Runs on Python 3 and C, has a faster dependency resolution algorithm, consumes less memory, and supports structured API extensions. In RHEL 8+, `yum` is simply a symbolic link pointing to `dnf`.

### 3. Repository Configuration (`/etc/yum.repos.d/`)
Repositories are defined in files ending in `.repo` inside `/etc/yum.repos.d/`.
- **Repo File Structure:**
  ```ini
  [baseos]
  name=Rocky Linux $releasever - BaseOS
  baseurl=http://dl.rockylinux.org/pub/rocky/$releasever/BaseOS/$basearch/os/
  gpgcheck=1
  enabled=1
  gpgkey=file:///etc/pki/moc-gpg/RPM-GPG-KEY-rockyofficial
  ```
  - **`[baseos]`** — Unique repository ID.
  - **`baseurl`** — Path to the package database (can be HTTP, FTP, or local file system path `file://`).
  - **`gpgcheck`** — 1 enables GPG cryptographic signature checking to verify package authenticity.
  - **`enabled`** — 1 enables the repo; 0 disables it.

### 4. DNF Command Reference

```bash
dnf install nginx          # Install package and dependencies
dnf remove nginx           # Remove package and clean up unused dependencies
dnf update nginx           # Update package to latest version
dnf search webserver       # Search package names and summaries for keyword
dnf info nginx             # Display package details and dependencies

dnf group list             # List available package group profiles (e.g., "Development Tools")
dnf group install "Development Tools" # Install all packages in group

dnf repolist               # List all enabled repositories
dnf clean all              # Clear local metadata cache database
dnf makecache              # Download and build fresh repository metadata cache
```

### 5. Extracting RPM Files Without Installing
If you need to retrieve a single configuration file from an RPM package without installing the entire package, use `rpm2cpio`:
```bash
rpm2cpio package.rpm | cpio -idmv
```
- This extracts the internal archive directories into your current working directory.

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
> A running Rocky Linux VM with internet access and root privileges.

### Step 1: Query Existing Package Database
1. Log in as root. Find if Apache (`httpd`) is installed:
   ```bash
   rpm -q httpd
   ```
2. If it returns "package httpd is not installed," find which package provides the command `/usr/sbin/httpd`:
   ```bash
   dnf provides /usr/sbin/httpd
   ```
   **Verify:** Output should display `httpd-[version]`.

### Step 2: Configure a Custom Local Repository
To simulate an offline datacenter environment:
1. Create a directory to host packages:
   ```bash
   mkdir -p /opt/local_repo
   ```
2. Download a package (e.g., `wget`) to the directory without installing it:
   ```bash
   dnf download --destdir=/opt/local_repo wget
   ```
3. Initialize the directory as a repository database (requires `createrepo_c` package installed):
   ```bash
   createrepo_c /opt/local_repo
   ```
4. Create the repository configuration file:
   ```bash
   vim /etc/yum.repos.d/local.repo
   ```
5. Add configuration:
   ```ini
   [local-backup-repo]
   name=Local Offline Backup Repository
   baseurl=file:///opt/local_repo/
   enabled=1
   gpgcheck=0
   ```
6. Save and close (`:wq`).

### Step 3: Test Local Repo Installation
1. Disable all standard internet repositories temporarily:
   ```bash
   dnf config-manager --disable "*"
   dnf config-manager --enable "local-backup-repo"
   ```
2. Verify active repo list:
   ```bash
   dnf repolist
   ```
   **Verify:** Only `local-backup-repo` should be listed.
3. Install the package from your local repository:
   ```bash
   dnf install wget -y
   ```
4. Re-enable standard repositories when finished:
   ```bash
   dnf config-manager --enable "baseos,appstream"
   ```

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
*Seedha simple mein: RPM low-level package controller hai jo individual files install karta hai lekin automatic dependencies download nahi karta. DNF modern high-level tool hai jo repositories se auto-dependency resolution ke sath packages download aur install karta hai.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Basic execution syntaxes.
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — Navigating `/etc` and `/opt`.
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Initializing services after package installation.
