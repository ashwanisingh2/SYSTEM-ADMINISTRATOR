---
tags: [linux, rhel, log-management]
aliases: [l-17, journalctl, rsyslog]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX RHEL**

`#complete` `#intermediate` `#rhcsa`

# L-17: Linux Log Management

> [!abstract] Overview
> Linux systems generate various logs that capture kernel events, system service activities, user authentication, and application behavior. This note covers how to view, filter, configure, rotate, and forward logs using `journalctl`, `rsyslog`, and `logrotate`. *Yeh SIEM integration aur daily troubleshooting dono ke liye zaroori hai, taaki ek sysadmin hamesha system events ko track kar sake.*

---
## 🧠 Concept Overview

- **What it is** — Log management is the process of generating, collecting, storing, analyzing, rotating, and retiring system and application logs to ensure system health and security.
- **Why it matters** — Without proper logs, system administrators are blind. Logs are the primary forensic evidence during security breaches, system crashes, or application errors. *Mismanaged logs root partition (/) ko full kar sakte hain, jisse server crash ho jayega.*
- **Where you see this** — Diagnosing a failed service, investigating unauthorized SSH login attempts, configuring automatic rotation of custom app logs, and sending logs to a SIEM dashboard.

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Reading system logs, checking service status with `journalctl`, and searching `/var/log/secure` for basic error messages. |
| **L2** | Configuring `rsyslog` rules, resolving "disk full" issues caused by runaway logs, and setting up `logrotate` policies. |
| **L3** | Setting up centralized log forwarding to a SIEM server (Splunk, Elastic), and debugging complex log pipelines. |

> [!tip] Seedha Simple Mein
> *Linux logs system ki har choti-badi activity ko record karta hai. `rsyslog` text log likhta hai, `journalctl` memory-based binary logs check karta hai, aur `logrotate` purane logs ko compress karke disk space bachata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A busy corporate office building** is like **Linux Log Management** because...
>
> - **The Security Guard's Register** is like `/var/log/secure` — every visitor (user login) must sign in and show ID.
> - **The CCTV System** is like `journalctl` — it records everything in a high-tech format, and you can search for a specific timestamp instantly.
> - **The Archives Clerk** is like `logrotate` — once a week, they take old registers, pack them in boxes, and shred them to save physical space.
> - **The Head Office Security Feed** is like **SIEM forwarding** — critical events are transmitted to the central control room for remote monitoring.

---
## 🔬 Technical Deep Dive

### 1. Systemd Journal and `journalctl`

> [!info] Key Concept
> Modern systemd-based Linux systems use a binary logging daemon called `systemd-journald`. It captures logs from kernel messages, system services, and standard streams.

- **Storage Location**: By default, logs are stored in a volatile format under `/run/log/journal/` (lost on reboot) unless configured to write to `/var/log/journal/` (persistent).
- **Format**: Unlike traditional text logs, journal logs are stored in structured binary files, allowing faster search and rich metadata querying.

**Key journalctl flags:**
- `-u <service>`: Filters for a specific systemd unit (e.g., `journalctl -u sshd`).
- `-f`: Follows logs in real-time.
- `-n <lines>`: Limits output lines (e.g., `journalctl -n 20`).
- `--since` / `--until`: Time-based filtering.
- `-p <priority>`: Filters by log level (`emerg` 0 to `debug` 7).

> [!danger] Common Mistake
> Forgetting to make journald logs persistent. *Agar logs `/run/log/journal` mein save ho rahe hain, toh server reboot ke baad sab logs permanently delete ho jayenge.*

### 2. Traditional Syslog (`rsyslog`)

> [!info] Key Concept
> `rsyslogd` manages traditional text logs, reading messages from `systemd-journald` and routing them to files based on rules in `/etc/rsyslog.conf`.

**Important Log Files in `/var/log/`:**
- `/var/log/messages`: Global log file for general system activity.
- `/var/log/secure`: Stores authentication, security, and sudo attempts.
- `/var/log/boot.log`: Captures system startup messages.
- `/var/log/cron`: Logs job execution status for scheduled cron jobs.
- `/var/log/maillog`: Contains mail server transactions and errors.

### 3. Logrotate: Managing Log Sizes

> [!info] Key Concept
> `logrotate` runs as a daily cron job to rotate, compress, and prune old files based on profiles inside `/etc/logrotate.d/`.

**Example Custom Config (`/etc/logrotate.d/myapp`)**
```properties
/var/log/myapp/*.log {
    daily                # Rotate the logs once a day
    rotate 7             # Keep 7 days of archives
    compress             # Compress rotated logs
    delaycompress        # Delay compression until the next cycle
    missingok            # Do not throw error if the log file is missing
    notifempty           # Do not rotate the log if it is empty
}
```

### 4. Log Forwarding to SIEM

> [!info] Key Concept
> Centralizing logs prevents attackers from covering their tracks. `rsyslog` can forward logs to remote SIEM platforms.

- **UDP Forwarding**: `*.* @192.168.10.50:514` (Fast, no confirmation)
- **TCP Forwarding**: `*.* @@192.168.10.50:514` (Reliable, connection-oriented)

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A running RHEL/Rocky Linux VM.
> - Root or sudo privileges (`sudo -i`).

### Step 1: Make journalctl Logs Persistent

```bash
# Update the journald config file
sudo sed -i 's/#Storage=auto/Storage=persistent/' /etc/systemd/journald.conf

# Restart the journald service
sudo systemctl restart systemd-journald

# Check if directory was created
ls -ld /var/log/journal
```

> [!success] Expected Output
> ```
> drwxr-sr-x. 3 root systemd-journal 20 Jun 26 17:40 /var/log/journal
> ```

### Step 2: Use journalctl for Advanced Filtering

```bash
# Restart SSH daemon to generate a log event
sudo systemctl restart sshd

# Query sshd logs for the last 5 minutes
journalctl -u sshd --since "5 minutes ago"
```

> [!success] Expected Output
> ```
> Jun 26 17:42:01 server1 systemd[1]: Starting OpenSSH server daemon...
> Jun 26 17:42:01 server1 sshd[3845]: Server listening on 0.0.0.0 port 22.
> ```

### Step 3: Configure rsyslog Forwarding

```bash
# Create custom forwarding rules file
cat <<EOF | sudo tee /etc/rsyslog.d/forward.conf
authpriv.* @@192.168.1.100:514
EOF

# Restart rsyslog to apply rules
sudo systemctl restart rsyslog
```

> [!success] Expected Output
> *Koi console output nahi aayega, but logs ab remote server par TCP port 514 par jayenge.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `journalctl` | Display all system logs starting from oldest. | `journalctl` |
| `journalctl -f` | Output logs in real-time (follow stream). | `journalctl -f` |
| `journalctl -u` | View logs for a specific systemd service unit. | `journalctl -u sshd` |
| `journalctl -p` | Filter logs by severity (3=err, 4=warn). | `journalctl -p err` |
| `journalctl --since`| View logs generated in a specific time frame. | `journalctl --since "1h ago"` |
| `journalctl --vacuum-size`| Delete oldest journals until below specific size. | `journalctl --vacuum-size=500M` |
| `logger -p` | Manually write a syslog entry with custom facility/level. | `logger -p auth.crit "Alert"` |
| `logrotate -f` | Manually force execution of a logrotate profile. | `logrotate -f /etc/logrotate.d/nginx` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `/var/log/` is 100% full. | Unrotated app logs or debug levels set too high. | Run `du -sh /var/log/*` to find culprit. Empty log (`> /var/log/app.log`) and create `logrotate` rule. |
| `journalctl` logs disappear after reboot. | Journal storage is volatile or directory missing. | Set `Storage=persistent` in `/etc/systemd/journald.conf`, and restart service. |
| `journalctl` query is extremely slow. | Journal files have accumulated, consuming Gigabytes. | Vacuum older records: `sudo journalctl --vacuum-time=7d`. |
| `rsyslog` is running but logs aren't updating. | Service stuck or log directory has wrong permissions. | Verify `/var/log/messages` owner is `root` or `adm` with `0600` permissions. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Disk Full due to Logs
> [!example] Ticket
> "Production Server Rocky-VM-01 is unreachable on port 80. Disk monitoring alert: Root partition is at 100%."

**L1 Response:** SSH into the box, run `df -h` to confirm full disk. Run `du -sh /var/log/*` to find the largest log file. If it is a systemd journal, run `journalctl --vacuum-size=1G` to free space immediately.
**Escalation Trigger:** The runaway file belongs to a custom app and simple deletion could break it.
**L2 Resolution:** Analyzed the custom app config to disable debug logging. Wrote a robust `logrotate` policy in `/etc/logrotate.d/custom-app` that uses size-based triggers (e.g., `size 100M`).

### 🎫 Scenario 2: Remote Log Forwarding Failed
> [!example] Ticket
> "Splunk Security team reports Rocky-VM-01 has stopped sending SSH audit logs to the SIEM receiver."

**L1 Response:** Verify `rsyslog` service status. Check network connectivity to Splunk: `nc -zvw3 192.168.1.100 514`.
**Escalation Trigger:** Network is reachable, but `rsyslog` shows socket connection warnings.
**L2/L3 Resolution:** Checked `/etc/rsyslog.d/forward.conf`. Ensure it has double-at `@@` for TCP transmission if the firewall blocks UDP. *SELinux deny list check kiya aur port 514 ko allow kiya.*

### 🎫 Scenario 3: Investigating Sudden Reboots
> [!example] Ticket
> "The database server rebooted unexpectedly at 3:00 AM. We need the root cause."

**L1 Response:** Run `last reboot` to see the reboot timestamp.
**Escalation Trigger:** Requires inspecting kernel panics or out-of-memory (OOM) killer events.
**L2 Resolution:** Used `journalctl -b -1` to view logs from the previous boot. Searched for `OOM` or `panic`. *Found that the kernel OOM killer terminated the DB process due to lack of RAM, causing a system halt.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the main difference between rsyslogd and systemd-journald?
> **Answer:** `systemd-journald` captures structured binary logs with rich metadata. `rsyslogd` reads these messages and routes them into plain-text files using facility/severity selectors, supporting remote forwarding.

> [!question] Q2: How do you view logs of a specific service from a previous boot session using journalctl?
> **Answer:** First, ensure persistent logging is enabled. List boots using `journalctl --list-boots`. To view logs for a service from the previous boot, use the `-b` flag with offset `-1`: `journalctl -b -1 -u sshd`.

> [!question] Q3: How does logrotate handle log files without disrupting running processes?
> **Answer:** It either uses a `postrotate` script block (e.g., `systemctl reload service`) to signal the daemon to reopen its log file handles, or it uses `copytruncate` to copy the active log file to a backup and empty the active file in-place.

> [!question] Q4: How do you configure rsyslog to send security auth logs over a secure TCP port instead of UDP?
> **Answer:** In `/etc/rsyslog.conf` or a `.conf` file inside `/etc/rsyslog.d/`, use the double-at `@@` prefix to specify TCP. Example: `authpriv.* @@siem-server-ip:514`.

==**Exam Tip:** For RHCSA, ensure you know how to make journalctl logs persistent by editing `/etc/systemd/journald.conf` and changing `Storage=persistent`.==

---
## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — For managing the services that generate logs.
- [[02-Operating-Systems/04-Linux-RHEL/L-14 Linux Security Hardening|L-14 Linux Security Hardening]] — For securing the `/var/log` directory.
- [[02-Operating-Systems/04-Linux-RHEL/L-16 Cron Jobs and Task Scheduling|L-16 Cron Jobs and Task Scheduling]] — `logrotate` runs as a daily cron job.
- [[04-Cloud-and-Security/10-Monitoring-and-Backup/MON-02 Azure Monitor and Log Analytics|MON-02 Azure Monitor and Log Analytics]] — Cloud equivalent of logging.
