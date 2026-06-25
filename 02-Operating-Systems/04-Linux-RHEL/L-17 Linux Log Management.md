---
tags: [linux, rhel, logging, rsyslog, journalctl]
aliases: [linux-logs]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# L-17 Linux Log Management

> [!abstract] Overview
> Linux systems generate various logs that capture kernel events, system service activities, user authentication, and application behavior. This note covers how to view, filter, configure, rotate, and forward logs using modern systemd tools (`journalctl`), traditional syslog daemons (`rsyslog`), and management utilities (`logrotate`), preparing you for both day-to-day administration and SIEM integration.

---
## Concept Overview
- **What it is** — Log management is the process of generating, collecting, storing, analyzing, rotating, and retiring system and application logs to ensure system health, security compliance, and troubleshooting capability.
- **Why it matters** — Without proper logs, system administrators are blind. Logs are the primary forensic evidence during security breaches, system crashes, or application errors. Mismanaged logs can also fill up the root partition (`/`), causing server-wide outages.
- **Where you encounter this** — Diagnosing a failed service, audit investigations for unauthorized SSH login attempts, configuring automatic rotation of custom application logs, and sending logs to a Security Information and Event Management (SIEM) dashboard.
- **L1 vs L2 vs L3:**
  - **L1**: Reading system logs, checking service status with `journalctl`, and searching `/var/log/secure` or `/var/log/messages` for basic error messages.
  - **L2**: Configuring `rsyslog` rules, resolving "disk full" issues caused by runaway logs, setting up custom `logrotate` policies, and persisting `journald` logs.
  - **L3**: Setting up centralized log forwarding to a SIEM/Syslog server (Splunk, Elastic, Sentinel), troubleshooting corrupted binary journals, and debugging complex log filtering pipelines.

*Seedha simple mein: Linux logs system system ki har choti-badi activity ko record karta hai. `rsyslog` system files mein text log likhta hai, `journalctl` memory-based binary logs check karne ke kaam aata hai, aur `logrotate` un logs ko compress aur delete karke disk space bachata hai.*

---
## Real-World Analogy
Imagine a busy corporate office building.
- **The Security Guard's Register** is like `/var/log/secure` — every visitor (user login, sudo attempt) must sign in and show ID.
- **The CCTV System** is like `journalctl` — it records everything in a high-tech binary format, and you can search for a specific timestamp, gate, or event level instantly.
- **The Archives Room Clerk** is like `logrotate` — once a week, they take the old security registers, put them in cardboard boxes (compression), store them for a few months, and eventually shred them (deletion) so the building doesn't run out of room.
- **The Head Office Security Feed** is like **SIEM forwarding** — a copy of every critical event is immediately transmitted to the headquarters' central control room for security analysis.

---
## Technical Deep Dive

### 1. Systemd Journal and `journalctl`
Modern systemd-based Linux systems use a binary logging daemon called `systemd-journald`. It captures logs from kernel messages, system services, and standard output/error streams.
- **Storage Location**: By default, logs are stored in a volatile format under `/run/log/journal/` (lost on reboot) unless configured to write to `/var/log/journal/` (persistent).
- **Format**: Unlike traditional text logs, journal logs are stored in structured binary files, allowing faster search, indexing, and rich metadata querying.
- **Key journalctl flags**:
  - `-u <service>`: Filters messages for a specific systemd unit (e.g., `journalctl -u sshd`).
  - `-f`: Follows the logs in real-time, matching tail behavior (e.g., `tail -f`).
  - `-n <lines>`: Limits the output to a specific number of lines (e.g., `journalctl -n 20`).
  - `--since` / `--until`: Time-based filtering (e.g., `journalctl --since "2026-06-25 12:00:00" --until "1 hour ago"`).
  - `-p <priority>`: Filters by log level. Priorities range from `emerg` (0) to `debug` (7).
  - `-k`: Displays only kernel messages (similar to `dmesg`).
  - `-xe`: Shows the end of the journal with additional explanation pages (highly useful for debug).

---
### 2. Traditional Syslog (`rsyslog`)
RHEL uses `rsyslogd` (Reliable Syslog Daemon) to manage traditional text logs. It reads system messages from systemd-journald and routes them to text files based on rules in `/etc/rsyslog.conf` and `/etc/rsyslog.d/`.

#### Common Facility and Severity Matrix
Syslog messages are categorized by **Facility** (who sent the message) and **Severity** (how critical it is).

```
Facility: auth, authpriv, cron, daemon, kern, mail, syslog, user, local0-7
Severity: emerg (0), alert (1), crit (2), err (3), warning (4), notice (5), info (6), debug (7)
```

#### Rule Syntax in `/etc/rsyslog.conf`
Rules specify `facility.severity /path/to/log/file`.
```properties
# Save authpriv (security/authentication) logs to secure file
authpriv.*                                              /var/log/secure

# Save cron logs
cron.*                                                  /var/log/cron

# Save everything except mail, authpriv, and cron logs to messages
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
```

#### Important Log Files in `/var/log/`
- `/var/log/messages`: The main global log file for general system activity (RHEL/CentOS/Rocky).
- `/var/log/secure`: Stores authentication, security, and sudo attempts (Debian/Ubuntu use `/var/log/auth.log`).
- `/var/log/boot.log`: Captures messages printed during system startup.
- `/var/log/cron`: Logs job execution status and errors for scheduled cron jobs.
- `/var/log/maillog`: Contains mail server transactions and errors.

---
### 3. Logrotate: Managing Log Sizes
As logs accumulate, they consume disk space. The `logrotate` utility runs as a daily cron job (`/etc/cron.daily/logrotate`) to rotate, compress, and prune old files based on configuration profiles inside `/etc/logrotate.conf` and `/etc/logrotate.d/`.

#### Example Custom Config File (`/etc/logrotate.d/myapp`)
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

---
### 4. Log Forwarding to SIEM
Centralizing logs on a secure server prevents attackers from covering their tracks by altering local logs. `rsyslog` can easily forward logs to remote SIEM platforms (like Splunk, Azure Sentinel, ELK Stack) or central Syslog servers.

- **UDP Forwarding** (Fast, unreliable, no confirmation):
  ```properties
  *.* @192.168.10.50:514
  ```
- **TCP Forwarding** (Reliable, connection-oriented):
  ```properties
  *.* @@192.168.10.50:514
  ```

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A running RHEL/Rocky Linux VM.
> - Root or sudo privileges (`sudo -i`).

### Step 1: Make journalctl Logs Persistent
By default, Rocky Linux/RHEL 9 may store journal logs in `/run/` (volatile memory). Let's configure persistent logging.
```bash
# Check if persistent journal directory exists
ls -ld /var/log/journal
```
If it is not present, edit `/etc/systemd/journald.conf` to set `Storage=persistent`.
```bash
# Update configuration file
sudo sed -i 's/#Storage=auto/Storage=persistent/' /etc/systemd/journald.conf

# Restart the journald service to apply settings
sudo systemctl restart systemd-journald

# Check if directory was created
ls -ld /var/log/journal
```
**Expected Output:** `drwxr-sr-x. 3 root systemd-journal 20 Jun 25 17:40 /var/log/journal`

---

### Step 2: Use journalctl for Advanced Filtering
Generate logs by attempting a failed login or restarting a service, and then query the logs.
```bash
# Restart SSH daemon
sudo systemctl restart sshd

# Query sshd logs for the last 5 minutes
journalctl -u sshd --since "5 minutes ago"
```
**Expected Output:**
```
-- Boot 5a1098de76c449dbba8f070fb67332d7 --
Jun 25 17:42:01 server1 systemd[1]: Stopping OpenSSH server daemon...
Jun 25 17:42:01 server1 sshd[1245]: Received signal 15; terminating.
Jun 25 17:42:01 server1 systemd[1]: Stopped OpenSSH server daemon.
Jun 25 17:42:01 server1 systemd[1]: Starting OpenSSH server daemon...
Jun 25 17:42:01 server1 sshd[3845]: Server listening on 0.0.0.0 port 22.
Jun 25 17:42:01 server1 systemd[1]: Started OpenSSH server daemon.
```

Filter specifically for Error-level messages or higher:
```bash
journalctl -p err -n 5
```

---

### Step 3: Write Custom Log Messages with `logger`
You can inject messages into `/var/log/messages` or `/var/log/secure` for scripting and testing purposes using the `logger` utility.
```bash
# Send custom log message
logger -p local0.info "SysAdmin Lab Test: This is a manual entry"

# Locate the message in logs
tail -n 5 /var/log/messages
```
**Expected Output:** `Jun 25 17:45:10 Rocky-VM user: SysAdmin Lab Test: This is a manual entry`

---

### Step 4: Configure and Test Custom Log Rotation
Let's create a mock log file and write a logrotate policy for it.
```bash
# Create target app log directory
sudo mkdir -p /var/log/mockapp

# Populate mock logs
echo "Initial mock log data" | sudo tee /var/log/mockapp/app.log

# Create a custom rotation rule
cat <<EOF | sudo tee /etc/logrotate.d/mockapp
/var/log/mockapp/app.log {
    rotate 3
    size 10
    compress
    create 0600 root root
}
EOF

# Force logrotate execution manually
sudo logrotate -f /etc/logrotate.d/mockapp

# Check directory contents for rotated file
ls -la /var/log/mockapp/
```
**Expected Output:**
```
total 4
drwxr-xr-x. 2 root root  38 Jun 25 17:48 .
drwxr-xr-x. 9 root root 220 Jun 25 17:48 ..
-rw-------. 1 root root   0 Jun 25 17:48 app.log
-rw-r--r--. 1 root root  42 Jun 25 17:48 app.log.1.gz
```

---

### Step 5: Configure rsyslog to Forward Logs
Let's set up Rocky VM to forward all authentication logs (`authpriv`) to a central logging server (IP: `192.168.1.100`) via TCP.
```bash
# Create custom forwarding rules file
cat <<EOF | sudo tee /etc/rsyslog.d/forward.conf
authpriv.* @@192.168.1.100:514
EOF

# Verify syntax config
sudo rsyslogd -N1

# Restart rsyslog to apply rules
sudo systemctl restart rsyslog
```
**Expected Output:** `rsyslogd: version 8.2102.0-xxx, config validation run (no errors found),...`

---
## Common Commands

| Command | Description | Example |
|---------|-------------|---------|
| `journalctl` | Display all system logs starting from oldest records. | `journalctl` |
| `journalctl -xe` | Show latest journal entries and detailed debug explanations. | `journalctl -xe` |
| `journalctl -f` | Output logs in real-time as they are written (follow stream). | `journalctl -f` |
| `journalctl -u sshd` | View logs generated by a specific systemd service unit. | `journalctl -u sshd` |
| `journalctl -p 3` | View logs filtered by severity (3 is Error, 4 is Warning). | `journalctl -p err` |
| `journalctl --since "1h ago"` | View logs generated only in the last 1 hour. | `journalctl --since "1h ago"` |
| `journalctl --disk-usage` | Check total disk space consumed by binary journal files. | `journalctl --disk-usage` |
| `journalctl --vacuum-size=500M` | Delete oldest journals until size is below 500 Megabytes. | `journalctl --vacuum-size=500M` |
| `logger -p local0.err "Msg"` | Manually write a syslog entry with custom facility and level. | `logger -p auth.crit "Alert"` |
| `logrotate -f /etc/logrotate.d/sys` | Manually force execution of a logrotate profile. | `logrotate -f /etc/logrotate.d/nginx` |

---
## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `/var/log/` or root partition (`/`) is 100% full. | Unrotated application logs or log levels set to debug for noisy apps. | Run `du -sh /var/log/*` to find the culprit. Empty the log (e.g., `cat /dev/null > /var/log/culprit.log`) and create/adjust `/etc/logrotate.d/` rule. |
| `journalctl` logs disappear after reboot. | Journal storage is configured as `volatile` or `/var/log/journal` missing. | Create directory `sudo mkdir -p /var/log/journal`, run `sudo systemctl restart systemd-journald`, or set `Storage=persistent` in `/etc/systemd/journald.conf`. |
| `journalctl` query is extremely slow. | Journal files have accumulated over months, consuming multiple Gigabytes. | Vacuum older records: `sudo journalctl --vacuum-time=7d` or limit max files by editing `/etc/systemd/journald.conf` and setting `SystemMaxUse=2G`. |
| `rsyslog` is running but logs aren't updating. | Rsyslog service is stuck or logging directory has incorrect permissions. | Check status `systemctl status rsyslog`. Verify that `/var/log/messages` has owner/group set to `root` or `adm` with permissions `0600` or `0640`. |

---
## Real-World Ticket Scenarios

### Scenario 1: Disk Full due to `/var/log`
**Ticket:** "Production Server Rocky-VM-01 is unreachable on port 80. Disk monitoring alert: Root partition is at 100%."
**L1 Response:** SSH into the box, run `df -h` to confirm full disk. Run `du -sh /var/log/*` or `ncdu /var/log` to find the largest log file. If it is a systemd journal, run `journalctl --vacuum-size=1G` to free space immediately.
**Escalation Trigger:** If the runaway file belongs to a third-party custom app and simple deletion could break the app daemon, or if disk usage climbs again within minutes.
**L2 Resolution:** Analyze the custom application config to disable debug logging, write a robust logrotate policy in `/etc/logrotate.d/custom-app` that uses size-based triggers (e.g., `size 100M`), and verify `logrotate` works.

---

### Scenario 2: Remote Log Forwarding Failed
**Ticket:** "Splunk Security team reports Rocky-VM-01 has stopped sending audit logs (SSH logs) to the SIEM receiver starting yesterday."
**L1 Response:** Verify `rsyslog` service status: `systemctl status rsyslog`. Check network connectivity to Splunk port 514: `nc -zvw3 192.168.1.100 514`.
**Escalation Trigger:** Network is reachable, but `rsyslog` errors out with socket connection warnings or fails syntax verification.
**L2 Resolution:** Check configuration file `/etc/rsyslog.d/forward.conf`. Ensure it has double-at `@@` for TCP transmission if the firewall blocks UDP. Inspect local SELinux denials using `sealert -a /var/log/audit/audit.log` to see if SELinux is blocking outbound port 514 syslog connections, and allow it via `setsebool -P nis_enabled 1` or configuring standard syslog ports.

---
## Interview Questions

**Q1: What is the main difference between rsyslogd and systemd-journald?**
> **A:** `systemd-journald` is the modern systemd logging daemon that captures structured binary logs (stored in memory/disk under `/run/log` or `/var/log/journal`). It includes rich metadata (PIDs, service names, boots). `rsyslogd` is the traditional syslog daemon that reads messages from systemd-journald and routes them into plain-text files (like `/var/log/messages`) using facility/severity selectors, and supports advanced text filtering/remote forwarding.

**Q2: How do you view logs of a specific service from a previous boot session using journalctl?**
> **A:** First, ensure persistent logging is enabled so journals survive boots. You can list boots using `journalctl --list-boots`. To view logs for a service (e.g., `nginx`) from the previous boot, use the `-b` flag with offset `-1`: `journalctl -b -1 -u nginx`.

**Q3: How does logrotate handle log files without disrupting running processes that are actively writing to them?**
> **A:** Logrotate has two main methods:
> 1. **Reloading the service (Standard)**: It rotates the log file and runs a `postrotate` script block (e.g., `systemctl reload service`) which signals the daemon to reopen its log file handles.
> 2. **copytruncate**: It copies the active log file to a backup and truncates (empties) the active file in-place. This is used for legacy applications that cannot reload log handles gracefully.

**Q4: How do you configure rsyslog to send security auth logs over a secure TCP port instead of UDP?**
> **A:** In `/etc/rsyslog.conf` or a dedicated `.conf` file inside `/etc/rsyslog.d/`, use the double-at `@@` prefix to specify TCP (single `@` is UDP). For example: `authpriv.* @@siem-server-ip:514`.

---
## Related Notes
- [[L-08 Services and Systemd]]
- [[L-14 Linux Security Hardening]]
- [[L-16 Cron Jobs and Task Scheduling]]
