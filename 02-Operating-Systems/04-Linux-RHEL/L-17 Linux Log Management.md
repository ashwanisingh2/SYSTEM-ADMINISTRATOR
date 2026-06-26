---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-17-linux-log-management, l-17]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX / RHEL**

`#complete` `#intermediate` `#rhcsa`

# L-17: Linux Log Management

> [!abstract] Overview
> Linux systems generate various logs that capture kernel events, system service activities, user authentication, and application behavior. Yeh note sikhata hai ki logs ko view, filter, configure, rotate, aur forward kaise karein `journalctl`, `rsyslog`, aur `logrotate` se, jo SIEM integration aur day-to-day troubleshooting dono ke liye zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — Log management is the process of generating, collecting, storing, analyzing, rotating, and retiring system and application logs to ensure system health and security.
- **Why it matters** — Without proper logs, system administrators are blind. Logs are the primary forensic evidence during security breaches, system crashes, or application errors. Mismanaged logs can also fill up the root partition (`/`), causing server outages.
- **Where you see this** — Diagnosing a failed service, investigating unauthorized SSH login attempts, configuring automatic rotation of custom app logs, and sending logs to a SIEM dashboard.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Reading system logs, checking service status with `journalctl`, and searching `/var/log/secure` or `/var/log/messages` for basic error messages. |
| **L2** | Configuring `rsyslog` rules, resolving "disk full" issues caused by runaway logs, setting up custom `logrotate` policies, and persisting `journald` logs. |
| **L3** | Setting up centralized log forwarding to a SIEM/Syslog server (Splunk, Elastic, Sentinel), troubleshooting corrupted binary journals, and debugging complex log pipelines. |

> [!tip] Seedha Simple Mein
> *Linux logs system ki har choti-badi activity ko record karta hai. `rsyslog` system files mein text log likhta hai, `journalctl` memory-based binary logs check karne ke kaam aata hai, aur `logrotate` un logs ko compress aur delete karke disk space bachata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A busy corporate office building** is like **Linux Log Management** because...
>
> - **The Security Guard's Register** is like `/var/log/secure` — every visitor (user login, sudo attempt) must sign in and show ID.
> - **The CCTV System** is like `journalctl` — it records everything in a high-tech binary format, and you can search for a specific timestamp or event instantly.
> - **The Archives Room Clerk** is like `logrotate` — once a week, they take old registers, compress them in boxes, and eventually shred them so the building doesn't run out of room.
> - **The Head Office Security Feed** is like **SIEM forwarding** — critical events are transmitted to the central control room for security analysis.

---
## 🔬 Technical Deep Dive

### 1. Systemd Journal and `journalctl`

> [!info] Key Concept
> Modern systemd-based Linux systems use a binary logging daemon called `systemd-journald`. It captures logs from kernel messages, system services, and standard output/error streams.

- **Storage Location**: By default, logs are stored in a volatile format under `/run/log/journal/` (lost on reboot) unless configured to write to `/var/log/journal/` (persistent).
- **Format**: Unlike traditional text logs, journal logs are stored in structured binary files, allowing faster search, indexing, and rich metadata querying.

**Key journalctl flags:**
- `-u <service>`: Filters for a specific systemd unit (e.g., `journalctl -u sshd`).
- `-f`: Follows logs in real-time, matching tail behavior.
- `-n <lines>`: Limits output lines (e.g., `journalctl -n 20`).
- `--since` / `--until`: Time-based filtering.
- `-p <priority>`: Filters by log level (`emerg` 0 to `debug` 7).
- `-k`: Kernel messages only (similar to `dmesg`).
- `-xe`: Shows end of journal with explanation pages.

> [!danger] Common Mistake
> Forgetting to make journald logs persistent. If they are stored in `/run/log/journal`, all logs will be permanently lost after a server reboot, destroying troubleshooting evidence!

### 2. Traditional Syslog (`rsyslog`)

> [!info] Key Concept
> `rsyslogd` manages traditional text logs, reading messages from `systemd-journald` and routing them to files based on rules in `/etc/rsyslog.conf` and `/etc/rsyslog.d/`.

**Important Log Files in `/var/log/`:**
- `/var/log/messages`: Main global log file for general system activity (RHEL/CentOS/Rocky).
- `/var/log/secure`: Stores authentication, security, and sudo attempts.
- `/var/log/boot.log`: Captures system startup messages.
- `/var/log/cron`: Logs job execution status for scheduled cron jobs.
- `/var/log/maillog`: Contains mail server transactions and errors.

### 3. Logrotate: Managing Log Sizes

> [!info] Key Concept
> `logrotate` runs as a daily cron job to rotate, compress, and prune old files based on configuration profiles inside `/etc/logrotate.conf` and `/etc/logrotate.d/`.

**Example Custom Config (`/etc/logrotate.d/myapp`)**
```properties
/var/log/myapp/*.log {
    daily                # Rotate the logs once a day
    rotate 7             # Keep 7 days of archives
    compress             # Compress rotated logs (gzip)
    delaycompress        # Delay compression until the next rotation cycle
    missingok            # Do not throw error if the log file is missing
    notifempty           # Do not rotate the log if it is empty
    create 0640 app root # Create new log file with these permissions
    postrotate
        /usr/bin/systemctl reload myapp.service > /dev/null 2>&1
    endscript            # Run this script to reload application after rotation
}
```

### 4. Log Forwarding to SIEM

> [!info] Key Concept
> Centralizing logs on a secure server prevents attackers from covering their tracks. `rsyslog` can forward logs to remote SIEM platforms (Splunk, Azure Sentinel) or central Syslog servers.

- **UDP Forwarding** (Fast, no confirmation): `*.* @192.168.10.50:514`
- **TCP Forwarding** (Reliable, connection-oriented): `*.* @@192.168.10.50:514`

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A running RHEL/Rocky Linux VM.
> - Root or sudo privileges (`sudo -i`).

### Step 1: Make journalctl Logs Persistent

```bash
# Update configuration file
sudo sed -i 's/#Storage=auto/Storage=persistent/' /etc/systemd/journald.conf

# Restart the journald service to apply settings
sudo systemctl restart systemd-journald

# Check if directory was created
ls -ld /var/log/journal
```

> [!success] Expected Output
> ```
> drwxr-sr-x. 3 root systemd-journal 20 Jun 25 17:40 /var/log/journal
> ```

### Step 2: Use journalctl for Advanced Filtering

```bash
# Restart SSH daemon
sudo systemctl restart sshd

# Query sshd logs for the last 5 minutes
journalctl -u sshd --since "5 minutes ago"
```

> [!success] Expected Output
> ```
> -- Boot 5a1098de76c449dbba8f070fb67332d7 --
> Jun 25 17:42:01 server1 systemd[1]: Stopping OpenSSH server daemon...
> Jun 25 17:42:01 server1 sshd[1245]: Received signal 15; terminating.
> Jun 25 17:42:01 server1 systemd[1]: Stopped OpenSSH server daemon.
> Jun 25 17:42:01 server1 systemd[1]: Starting OpenSSH server daemon...
> Jun 25 17:42:01 server1 sshd[3845]: Server listening on 0.0.0.0 port 22.
> Jun 25 17:42:01 server1 systemd[1]: Started OpenSSH server daemon.
> ```

### Step 3: Write Custom Log Messages with `logger`

```bash
# Send custom log message
logger -p local0.info "SysAdmin Lab Test: This is a manual entry"

# Locate the message in logs
tail -n 5 /var/log/messages
```

> [!success] Expected Output
> ```
> Jun 25 17:45:10 Rocky-VM user: SysAdmin Lab Test: This is a manual entry
> ```

### Step 4: Configure rsyslog to Forward Logs

```bash
# Create custom forwarding rules file
cat <<EOF | sudo tee /etc/rsyslog.d/forward.conf
authpriv.* @@192.168.1.100:514
EOF

# Restart rsyslog to apply rules
sudo systemctl restart rsyslog
```

> [!success] Expected Output
> ```
> rsyslogd: version 8.2102.0-xxx, config validation run (no errors found),...
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `journalctl` | Display all system logs starting from oldest. | `journalctl` |
| `journalctl -xe` | Show latest journal entries with detailed debug explanations. | `journalctl -xe` |
| `journalctl -f` | Output logs in real-time (follow stream). | `journalctl -f` |
| `journalctl -u` | View logs for a specific systemd service unit. | `journalctl -u sshd` |
| `journalctl -p` | Filter logs by severity (3=err, 4=warn). | `journalctl -p err` |
| `journalctl --since`| View logs generated in a specific time frame. | `journalctl --since "1h ago"` |
| `journalctl --disk-usage` | Check total disk space consumed by journals. | `journalctl --disk-usage` |
| `journalctl --vacuum-size`| Delete oldest journals until below specific size. | `journalctl --vacuum-size=500M` |
| `logger -p` | Manually write a syslog entry with custom facility/level. | `logger -p auth.crit "Alert"` |
| `logrotate -f` | Manually force execution of a logrotate profile. | `logrotate -f /etc/logrotate.d/nginx` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `/var/log/` or root partition (`/`) is 100% full. | Unrotated app logs or debug levels set for noisy apps. | Run `du -sh /var/log/*` to find culprit. Empty log (`cat /dev/null > /var/log/app.log`) and create `/etc/logrotate.d/` rule. |
| `journalctl` logs disappear after reboot. | Journal storage is volatile or `/var/log/journal` missing. | Create directory `sudo mkdir -p /var/log/journal`, set `Storage=persistent` in `/etc/systemd/journald.conf`, and restart service. |
| `journalctl` query is extremely slow. | Journal files have accumulated, consuming Gigabytes. | Vacuum older records: `sudo journalctl --vacuum-time=7d` or set `SystemMaxUse=2G` in `/etc/systemd/journald.conf`. |
| `rsyslog` is running but logs aren't updating. | Service stuck or log directory has wrong permissions. | Check `systemctl status rsyslog`. Verify `/var/log/messages` owner is `root` or `adm` with `0600` permissions. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Disk Full due to `/var/log`

> [!example] Ticket
> "Production Server Rocky-VM-01 is unreachable on port 80. Disk monitoring alert: Root partition is at 100%."

**L1 Response:** SSH into the box, run `df -h` to confirm full disk. Run `du -sh /var/log/*` to find the largest log file. If it is a systemd journal, run `journalctl --vacuum-size=1G` to free space immediately.
**Escalation Trigger:** If the runaway file belongs to a third-party custom app and simple deletion could break the app daemon.
**L2 Resolution:** Analyze the custom app config to disable debug logging, write a robust logrotate policy in `/etc/logrotate.d/custom-app` that uses size-based triggers (e.g., `size 100M`), and verify `logrotate` works.

### 🎫 Scenario 2: Remote Log Forwarding Failed

> [!example] Ticket
> "Splunk Security team reports Rocky-VM-01 has stopped sending audit logs (SSH logs) to the SIEM receiver starting yesterday."

**L1 Response:** Verify `rsyslog` service status: `systemctl status rsyslog`. Check network connectivity to Splunk port 514: `nc -zvw3 192.168.1.100 514`.
**Escalation Trigger:** Network is reachable, but `rsyslog` errors out with socket connection warnings.
**L2 Resolution:** Check `/etc/rsyslog.d/forward.conf`. Ensure it has double-at `@@` for TCP transmission if the firewall blocks UDP. Inspect SELinux denials (`sealert -a /var/log/audit/audit.log`) to see if it's blocking port 514, and allow it.

---
## 🎤 Interview Questions

> [!question] Q1: What is the main difference between rsyslogd and systemd-journald?
> **Answer:** `systemd-journald` captures structured binary logs with rich metadata. `rsyslogd` reads these messages and routes them into plain-text files using facility/severity selectors, supporting remote forwarding.

> [!question] Q2: How do you view logs of a specific service from a previous boot session using journalctl?
> **Answer:** First, ensure persistent logging is enabled. List boots using `journalctl --list-boots`. To view logs for a service from the previous boot, use the `-b` flag with offset `-1`: `journalctl -b -1 -u nginx`.

> [!question] Q3: How does logrotate handle log files without disrupting running processes?
> **Answer:** It either uses a `postrotate` script block (e.g., `systemctl reload service`) to signal the daemon to reopen its log file handles, or it uses `copytruncate` to copy the active log file to a backup and empty the active file in-place.

> [!question] Q4: How do you configure rsyslog to send security auth logs over a secure TCP port instead of UDP?
> **Answer:** In `/etc/rsyslog.conf` or a `.conf` file inside `/etc/rsyslog.d/`, use the double-at `@@` prefix to specify TCP. Example: `authpriv.* @@siem-server-ip:514`.

==**Exam Tip:** For RHCSA, ensure you know how to make journalctl logs persistent by editing `/etc/systemd/journald.conf` and changing `Storage=persistent`.==

---
## 🔗 Related Notes

- [[L-08 Services and Systemd|Services and Systemd]] — For managing the services that generate logs.
- [[L-14 Linux Security Hardening|Linux Security Hardening]] — For securing the `/var/log` directory.
- [[L-16 Cron Jobs and Task Scheduling|Cron Jobs and Task Scheduling]] — `logrotate` runs as a daily cron job.
