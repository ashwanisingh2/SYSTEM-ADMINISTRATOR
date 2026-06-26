---
tags: [linux, rhel, automation, scheduling]
aliases: [l-16, cron, anacron]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX RHEL**

`#complete` `#intermediate` `#rhcsa`

# L-16: Cron Jobs and Task Scheduling

> [!abstract] Overview
> Task scheduling is a fundamental administrative duty in Linux. This note covers task scheduling using the cron daemon (`crond`), user crontabs, global system crontabs, access control, and asynchronous task scheduling via `anacron` for systems that do not run 24/7. *Yeh ek support engineer ko manual tasks automate karne aur server ko healthy rakhne mein madad karta hai.*

---
## 🧠 Concept Overview

- **What it is** — Cron is a time-based job scheduler daemon in Unix-like operating systems. Users and administrators use it to run commands, scripts, or programs automatically at specified times or intervals.
- **Why it matters** — Manual tasks lead to human error. Automating backups, database cleanup, log rotations, and system updates ensures reliable server maintenance. *Roz ek hi command manually run karna IT mein acchi practice nahi hai.*
- **Where you see this** — Scheduling a nightly backup script at 2:00 AM, rotating Apache logs weekly, or setting up a system health-check script that executes every 5 minutes.

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Checks if the `crond` service is running, views crontab log entries in `/var/log/cron`, and lists user jobs using `crontab -l`. |
| **L2** | Configures user cron jobs (`crontab -e`), resolves environment path problems inside scripts, and manages access controls via `cron.allow`. |
| **L3** | Implements system-wide periodic tasks, manages `anacrontab` rules for desktops/VMs, and designs fault-tolerant cron failovers. |

> [!tip] Seedha Simple Mein
> *Cron jobs Windows ke Task Scheduler ki tarah kaam karti hain. Iski help se hum system ko bata sakte hain ki kaunsa script ya command kab run hona chahiye (jaise har Sunday ko backup lena ya har ghante system cache clear karna).*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Cron** is like **a Factory Alarm Clock** because...
> 
> - Just like a factory siren rings exactly at 8:00 AM, 12:00 PM, and 4:00 PM every single day to signal shift changes, cron executes a script exactly at a specified time.
> - If the factory loses power (server is off), the regular alarm (cron) won't ring when power returns, but the smart manager (anacron) will instantly realize a shift change was missed and trigger the process immediately!

---
## 🔬 Technical Deep Dive

### 1. Crontab Syntax Explained

> [!info] Key Concept
> Cron expressions use 5 time/date fields to define schedule, followed by the command to execute.

```text
 *     *     *     *     *    [command to execute]
 ┬     ┬     ┬     ┬     ┬
 │     │     │     │     │
 │     │     │     │     └───── Day of Week (0-7) (0 and 7 = Sunday)
 │     │     │     └────────── Month (1-12)
 │     │     └─────────────── Day of Month (1-31)
 │     └──────────────────── Hour (0-23)
 └───────────────────────── Minute (0-59)
```

**Special Characters:**
- `*` (Asterisk): Matches all values (*har minute, har ghante*).
- `,` (Comma): Specifies a list of values (`1,5` means "1 and 5").
- `-` (Dash): Specifies a range (`1-5` means "Monday to Friday").
- `/` (Slash): Specifies step values (`*/10` means "every 10 minutes").

### 2. Cron Configuration Files

Linux distinguishes between user-defined cron jobs and system-wide cron jobs:
- **User Crontabs**: Stored in `/var/spool/cron/<username>`. Modified using `crontab -e`.
- **System-Wide Crontabs**: Stored in `/etc/crontab` and directory paths:
  - `/etc/cron.d/` — Modular configuration files.
  - `/etc/cron.hourly/` — Scripts here run once every hour.
  - `/etc/cron.daily/` — Scripts here run once a day.
  - `/etc/cron.weekly/` — Scripts here run once a week.
  - `/etc/cron.monthly/` — Scripts here run once a month.

> [!danger] Common Mistake
> *Kabhi bhi `/var/spool/cron/<username>` files ko seedha `vi` ya `nano` se edit mat karo.* Always use the `crontab -e` command to ensure syntax validation and proper daemon reload.

### 3. anacron vs cron

- **cron**: Assumes the system runs continuously (24/7). If the server is powered off at 2:00 AM when a job was scheduled, that job **will not run** when the system boots up again.
- **anacron**: Designed for systems that are shut down frequently (laptops, VMs). It tracks execution frequency rather than absolute times. If a daily job was missed, anacron executes it immediately after the system boots. Configured in `/etc/anacrontab`.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux server (RHEL 8/9).
> - Active `crond` daemon.
> - A regular user account (`student` or `sysadmin`).

### Step 1: Manage User-Level Crontab

We will schedule a simple job that appends system load details to a log file every minute.

```bash
# Edit current user's crontab
crontab -e

# Add this line at the bottom to run uptime every minute
* * * * * /usr/bin/uptime >> /home/student/system_load.log
```

> [!success] Expected Output
> ```
> crontab: installing new crontab
> ```

### Step 2: Restrict Cron Access

We will block a user named `baduser` from using cron scheduling, while allowing only approved admins.

```bash
# Add username to the deny list
echo "baduser" | sudo tee -a /etc/cron.deny

# Verify the block
su - baduser
crontab -e
```

> [!success] Expected Output
> ```
> You (baduser) are not allowed to use this program (crontab)
> ```

### Step 3: Implement System-Wide Cron Job

Create a script that clears `/tmp` files older than 7 days, and run it daily.

```bash
# Create the script inside cron.daily
sudo vi /etc/cron.daily/clear_tmp.sh

# Add the following lines:
#!/bin/bash
find /tmp -type f -atime +7 -delete

# Make the script executable
sudo chmod +x /etc/cron.daily/clear_tmp.sh
```

> [!success] Expected Output
> *Koi output nahi aayega, par script ab har raat daily cycle mein run hoga.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `crontab -l` | List all cron jobs for the current user | `crontab -l` |
| `crontab -e` | Edit cron jobs for the current user | `crontab -e` |
| `crontab -r` | Remove all cron jobs for the current user | `crontab -r` |
| `crontab -u user -e` | Edit cron jobs for a specific user | `sudo crontab -u student -e` |
| `systemctl status crond` | Check if cron service is running | `systemctl status crond` |
| `tail -f /var/log/cron` | View live cron execution logs | `sudo tail -f /var/log/cron` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|---------|-------|
| Cron job fails, but runs fine in terminal | Environment paths (`PATH`) differ in cron's minimal shell. | Use absolute paths (`/usr/bin/tar`). Set `PATH` variable at the top of the crontab. |
| Script errors not visible | Standard error is not redirected, so it gets lost. | Redirect stdout and stderr: `/script.sh > /var/log/script.log 2>&1` |
| Anacron jobs not executing on a VM | Anacron utility missing or misconfigured. | Ensure `anacron` package is installed. Verify config in `/etc/anacrontab`. |
| "crontab: command not found" | Cron package is not installed on the minimal system. | Run `sudo yum install cronie` and start the service. |
| User cannot edit crontab | User is listed in `cron.deny` or missing from `cron.allow`. | Remove user from `/etc/cron.deny` or add them to `/etc/cron.allow`. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Backup Script Fails at Night
> [!example] Ticket
> "Our daily database backup job scheduled at 3:00 AM via cron did not run successfully last night, but works when I run it manually."

**L1 Response:** Check `/var/log/cron` to see if the job actually triggered at 3:00 AM.
**Escalation Trigger:** The job fired but produced errors, or no output was logged.
**L2 Resolution:** Review the user's `crontab -e`. The script was using `mysqldump` without the full path. Cron's limited `$PATH` didn't know where it was. *Script mein `/usr/bin/mysqldump` update kiya aur issue resolve ho gaya.*

### 🎫 Scenario 2: High CPU Usage Every 5 Minutes
> [!example] Ticket
> "The web server spikes to 100% CPU exactly every 5 minutes. Please investigate."

**L1 Response:** Check `top` or `htop` to identify the process causing the spike.
**Escalation Trigger:** The process disappears too fast, but happens consistently on a schedule.
**L2/L3 Resolution:** Check the system and root crontabs. Found `*/5 * * * * /opt/scripts/heavy_sync.sh` running every 5 minutes. Optimized the script and changed the schedule to hourly (`0 * * * *`).

### 🎫 Scenario 3: Missing Logs due to Overwritten Output
> [!example] Ticket
> "My cron job runs daily, but the log file only ever contains the output from the last run, not the historical runs."

**L1 Response:** Escalate to L2 for script debugging.
**Escalation Trigger:** Requires crontab syntax modification.
**L2 Resolution:** Checked `crontab -l`. The user wrote `* * * * * script.sh > output.log`. The single `>` overwrites the file. *Usko `>>` (append) se replace kiya.* `* * * * * script.sh >> output.log`.

---
## 🎤 Interview Questions

> [!question] Q1: What does `*/15 2 1-10 * * /usr/local/bin/backup.sh` translate to?
> **Answer:** It runs every 15 minutes (`*/15`), in the 2:00 AM hour (`2`), from the 1st to the 10th day of every month (`1-10`), regardless of month or day of week.

==**Exam Tip:** Cron syntax reading (Minute, Hour, Day, Month, Weekday) is a guaranteed question in RHCSA/Linux+ exams. Memorize the 5 fields!==

> [!question] Q2: What is the main difference between cron and anacron?
> **Answer:** **cron** assumes continuous 24/7 uptime and misses jobs if the system is powered off. **anacron** tracks execution frequency (daily, weekly) and runs missed jobs immediately upon boot for systems that power down frequently.

> [!question] Q3: How do you verify if user `alice` has any cron jobs scheduled without switching to her account?
> **Answer:** Run `sudo crontab -u alice -l`. This allows the root/admin user to inspect her scheduled tasks.

> [!question] Q4: Why does a cron job fail when calling a command like `systemctl restart nginx` without absolute paths?
> **Answer:** Cron uses a restricted, minimal `PATH` environment variable (typically `/usr/bin:/bin`). Commands located in `/usr/sbin` won't resolve unless specified with their absolute paths (e.g., `/usr/sbin/systemctl`).

> [!question] Q5: How do you discard all output (both stdout and stderr) from a noisy cron job?
> **Answer:** Append `> /dev/null 2>&1` to the end of the cron job entry.

---
## 🔗 Related Notes

- [[02-Operating-Systems/03-Windows-OS/WIN-06 Task Scheduler|WIN-06 Task Scheduler]] — Counterpart in Windows
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Basic CLI usage
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Managing daemons like crond
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-04 PowerShell Scripting|PS-04 PowerShell Scripting]] — Alternative automation method
