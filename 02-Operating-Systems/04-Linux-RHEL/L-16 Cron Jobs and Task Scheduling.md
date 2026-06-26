---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-16-cron-jobs-and-task-scheduling, l-16]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX / RHEL**

`#complete` `#beginner` `#rhcsa`

# L-16: Cron Jobs and Task Scheduling

> [!abstract] Overview
> Task scheduling is a fundamental administrative task in Linux. This note covers task scheduling using the cron daemon (`crond`), user crontabs, global system crontabs, cron access control, and asynchronous task scheduling via `anacron` for systems that do not run 24/7. Yeh ek support engineer ko reliable server maintenance aur automation sikhne mein madad karta hai.

---
## 🧠 Concept Overview

- **What it is** — Plain English mein definition: Cron is a time-based job scheduler daemon in Unix-like operating systems. Users and administrators use it to run commands, scripts, or programs automatically at specified times or intervals.
- **Why it matters** — Real job mein kyun zaroori hai: Manual tasks lead to human error. Automating backups, database cleanup, log rotations, and system updates ensures reliable server maintenance.
- **Where you see this** — Actual scenarios jahan yeh aata hai: Scheduling a nightly backup script at 2:00 AM, rotating Apache logs weekly, or setting up a system health-check script that executes every 5 minutes.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Checks if the `crond` service is running, views crontab log entries in `/var/log/cron`, and lists a user's scheduled jobs using `crontab -l`. Escalate to L2 if a cron job fails. |
| **L2** | Configures user cron jobs (`crontab -e`), resolves environment path problems inside scripts, and manages access controls via `/etc/cron.allow` and `/etc/cron.deny`. |
| **L3** | Implements system-wide periodic tasks in `/etc/cron.d/`, manages `/etc/anacrontab` rules for desktops/non-continuous servers, and designs fault-tolerant cron failovers. |

> [!tip] Seedha Simple Mein
> *Cron jobs Windows ke Task Scheduler ki tarah kaam karti hain. Iski help se hum system ko bata sakte hain ki kaunsa script ya command kab run hona chahiye (jaise har Sunday ko backup lena ya har ghante system cache clear karna).*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Cron** is like **an Alarm Clock with specific rules** because...
>
> - Just like an alarm clock wakes you up exactly at 6:00 AM every Monday, cron executes a script exactly at a specified time.
> - If the power goes out (server is off), the regular alarm (cron) won't ring when power returns, but a smart "catch-up" app (anacron) will remind you immediately!

---
## 🔬 Technical Deep Dive

### 1. Crontab Syntax

> [!info] Key Concept
> Cron expressions use 5 time/date fields to define schedule, followed by the command to execute.

```text
 *     *     *     *     *    [command to execute]
 ┬     ┬     ┬     ┬     ┬
 │     │     │     │     │
 │     │     │     │     └───── Day of Week (0-6) (0 = Sunday)
 │     │     │     └────────── Month (1-12)
 │     │     └─────────────── Day of Month (1-31)
 │     └──────────────────── Hour (0-23)
 └───────────────────────── Minute (0-59)
```

**Special Characters:**
- **Bold**: Important terms, commands, file paths
- `*` (Asterisk): Matches all values (e.g., `*` in the hour field means "every hour").
- `,` (Comma): Specifies a list of values (e.g., `1,5` in the hour field means "1:00 and 5:00").
- `-` (Dash): Specifies a range of values (e.g., `1-5` in the day of week field means "Monday to Friday").
- `/` (Slash): Specifies step values/intervals (e.g., `*/10` in the minute field means "every 10 minutes").

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
> Never edit `/var/spool/cron/<username>` files directly with `vi` or `nano`. Always use the `crontab -e` command to ensure syntax validation and proper daemon reload.

### 3. anacron vs cron

- **cron**: Assumes the system runs continuously (24/7). If the server is powered off at 2:00 AM when a job was scheduled, that job **will not run** when the system boots up again.
- **anacron**: Designed for systems that are shut down frequently (laptops, workstations). It tracks execution frequency rather than absolute times. If a daily job was missed because the server was off, anacron executes it immediately after the system boots back up. Configured in `/etc/anacrontab`.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux server (RHEL 8/9).
> - Active `crond` daemon.
> - A regular user account (`student` or `sysadmin`).

### Step 1: Manage User-Level Crontab

We will schedule a simple job that appends system load details to a log file every minute.

```bash
# Open the crontab editor
crontab -e

# Add the following line at the end of the file
* * * * * /usr/bin/uptime >> /home/student/system_load.log
```

> [!success] Expected Output
> ```
> crontab: installing new crontab
> ```

Check output after 2 minutes:
```bash
cat /home/student/system_load.log
```

### Step 2: Restrict Cron Access

We will block a user named `baduser` from using cron scheduling, while allowing only approved admins.

```bash
# Edit cron deny file
sudo vi /etc/cron.deny

# Add the username:
baduser
```

Test by logging in as `baduser` and running crontab:
```bash
su - baduser
crontab -e
```

> [!success] Expected Output
> ```
> You (baduser) are not allowed to use this program (crontab)
> ```

### Step 3: Implement System-Wide Cron Job

```bash
# Create script file
sudo vi /etc/cron.daily/clear_tmp.sh

# Script content
#!/bin/bash
find /tmp -type f -atime +7 -delete

# Make it executable
sudo chmod +x /etc/cron.daily/clear_tmp.sh
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `crontab -l` | List all cron jobs for the current user | `crontab -l` |
| `crontab -e` | Edit cron jobs for the current user | `crontab -e` |
| `crontab -r` | Remove all cron jobs for the current user | `crontab -r` |
| `crontab -u [user] -e` | Edit cron jobs for a specific user (requires sudo) | `sudo crontab -u student -e` |
| `tail -f /var/log/cron` | Monitor active cron job execution logs | `sudo tail -f /var/log/cron` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Cron job fails, but runs fine in terminal** | Environment paths (`PATH`) differ. Cron runs in a minimal shell. | Use absolute paths (`/usr/bin/tar`). Set `PATH` variable at the top of the crontab. |
| **Script redirects error, but cron mail shows errors** | Output redirecting missed stderr stream. | Redirect standard output and error: `/path/to/script.sh > /dev/null 2>&1` |
| **Anacron jobs not executing on a VM** | Anacron utility missing or vm running 24/7 (handled by cron instead). | Ensure `anacron` package is installed. Verify config in `/etc/anacrontab`. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Backup Script Fails at Night

> [!example] Ticket
> "Our daily database backup job scheduled at 3:00 AM via cron did not run successfully last night."

**L1 Response:** Check if `crond` is running. Check `/var/log/cron` to see if the job fired at 3:00 AM.
**Escalation Trigger:** The job fired but produced errors, or user permissions are preventing execution.
**L2 Resolution:** Review the user's `crontab -l`. Notice they didn't use absolute paths for the `mysqldump` command. Update the script to use absolute paths.

---
## 🎤 Interview Questions

> [!question] Q1: What does `*/15 2 1-10 * * /usr/local/bin/backup.sh` translate to?
> **Answer:** It runs every 15 minutes (`*/15`), in the 2:00 AM hour (`2`), from the 1st to the 10th day of every month (`1-10`), regardless of month or day of week.

==**Exam Tip:** Cron syntax reading (Minute, Hour, Day, Month, Weekday) is a guaranteed question in RHCSA/Linux+ exams.==

> [!question] Q2: What is the main difference between cron and anacron?
> **Answer:** **cron** assumes continuous 24/7 uptime and misses jobs if the system is off. **anacron** tracks execution frequency and runs missed jobs immediately upon boot for systems that power down frequently.

> [!question] Q3: How do you verify if user `alice` has any cron jobs scheduled without switching to her account?
> **Answer:** Run `sudo crontab -u alice -l`.

> [!question] Q4: Why does a cron job fail when calling a command like `service httpd restart` without absolute paths?
> **Answer:** Cron uses a restricted minimal `PATH` (typically `/usr/bin:/bin`). Commands in `/usr/sbin` won't resolve unless specified with absolute paths (e.g., `/usr/sbin/service`).

---
## 🔗 Related Notes

- [[02-Operating-Systems/03-Windows-OS/Task-Scheduler|Windows Task Scheduler]] — Counterpart in Windows
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Basic CLI usage
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Automation-and-Scripting|PowerShell Automation]] — Alternative automation method
