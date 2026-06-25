---
tags: [linux, automation, cron, rhcsa]
aliases: [cron-jobs, scheduling]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #rhcsa
---

# L-16 Cron Jobs and Task Scheduling

> [!abstract] Overview
> Task scheduling is a fundamental administrative task in Linux. This note covers task scheduling using the cron daemon (`crond`), user crontabs, global system crontabs, cron access control, and asynchronous task scheduling via `anacron` for systems that do not run 24/7.

---

## Concept Overview
- **What it is** — Cron is a time-based job scheduler daemon in Unix-like operating systems. Users and administrators use it to run commands, scripts, or programs automatically at specified times or intervals.
- **Why it matters for a support engineer** — Manual tasks lead to human error. Automating backups, database cleanup, log rotations, and system updates ensures reliable server maintenance.
- **Where you encounter this in real job** — Scheduling a nightly backup script at 2:00 AM, rotating Apache logs weekly, or setting up a system health-check script that executes every 5 minutes.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Checks if the `crond` service is running, views crontab log entries in `/var/log/cron`, and lists a user's scheduled jobs using `crontab -l`.
  - **Escalation Trigger:** Escalate to L2 if a cron job fails to execute as expected, or if a user needs cron access permission configured.
  - ****L2 Resolution:**** Configures user cron jobs (`crontab -e`), resolves environment path problems inside scripts, and manages access controls via `/etc/cron.allow` and `/etc/cron.deny`.
  - ****L3 Resolution:**** Implements system-wide periodic tasks in `/etc/cron.d/`, manages `/etc/anacrontab` rules for desktop environments/non-continuous servers, and designs fault-tolerant cron failovers.

*Seedha simple mein: Cron jobs Windows ke Task Scheduler ki tarah kaam karti hain. Iski help se hum system ko bata sakte hain ki kaunsa script ya command kab run hona chahiye (jaise har Sunday ko backup lena ya har ghante system cache clear karna).*

---

## Technical Deep Dive

### 1. Crontab Syntax
Cron expressions are defined using 5 time/date fields followed by the command to execute:

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

#### Special Characters:
*   `*` (Asterisk): Matches all values (e.g., `*` in the hour field means "every hour").
*   `,` (Comma): Specifies a list of values (e.g., `1,5` in the hour field means "1:00 and 5:00").
*   `-` (Dash): Specifies a range of values (e.g., `1-5` in the day of week field means "Monday to Friday").
*   `/` (Slash): Specifies step values/intervals (e.g., `*/10` in the minute field means "every 10 minutes").

### 2. Cron Configuration Files
Linux distinguishes between user-defined cron jobs and system-wide cron jobs:
*   **User Crontabs**: Stored in `/var/spool/cron/<username>`. Modified using `crontab -e`.
*   **System-Wide Crontabs**: Stored in `/etc/crontab` and directory paths:
    *   `/etc/cron.d/` — Modular configuration files.
    *   `/etc/cron.hourly/` — Scripts here run once every hour.
    *   `/etc/cron.daily/` — Scripts here run once a day.
    *   `/etc/cron.weekly/` — Scripts here run once a week.
    *   `/etc/cron.monthly/` — Scripts here run once a month.

### 3. anacron vs cron
*   **cron**: Assumes the system runs continuously (24/7). If the server is powered off at 2:00 AM when a job was scheduled, that job **will not run** when the system boots up again.
*   **anacron**: Designed for systems that are shut down frequently (laptops, workstations). It tracks execution frequency rather than absolute times. If a daily job was missed because the server was off, anacron executes it immediately after the system boots back up. Configured in `/etc/anacrontab`.

---

## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - A Linux server (RHEL 8/9).
> - Active `crond` daemon.
> - A regular user account (`student` or `sysadmin`).

### Step 1: Manage User-Level Crontab
We will schedule a simple job that appends system load details to a log file every minute.

1. Log in as a regular user and open the crontab editor:
```bash
crontab -e
```
2. Add the following line at the end of the file:
```text
* * * * * /usr/bin/uptime >> /home/student/system_load.log
```
3. Save and close the editor (if using Vim, press `ESC`, type `:wq`, and press `Enter`).
**Expected Output:** `crontab: installing new crontab`.

4. Wait 2 minutes and check the output log:
```bash
cat /home/student/system_load.log
```
**Expected Output:** Logs containing uptime outputs showing execution at 1-minute intervals.

---

### Step 2: Restrict Cron Access
We will block a user named `baduser` from using cron scheduling, while allowing only approved admins.

1. View the cron deny file:
```bash
sudo vi /etc/cron.deny
```
2. Add the username to restrict:
```text
baduser
```
3. Test by logging in as `baduser` and running crontab commands:
```bash
su - baduser
crontab -e
```
**Expected Output:** `You (baduser) are not allowed to use this program (crontab)`.

---

### Step 3: Implement System-Wide Cron Job
We will deploy a daily cache clearing script to the system-wide `/etc/cron.daily/` directory.

1. Create a script file in `/etc/cron.daily/`:
```bash
sudo vi /etc/cron.daily/clear_tmp.sh
```
2. Write the script block:
```bash
#!/bin/bash
# Remove user temp files older than 7 days
find /tmp -type f -atime +7 -delete
```
3. Make the script executable:
```bash
sudo chmod +x /etc/cron.daily/clear_tmp.sh
```
**Expected Output:** The script is registered. The system cron daemon runs this script daily.

---

## Common Commands / Cheat Sheet

| Command | Description | Example |
|---------|-------------|---------|
| `crontab -l` | List all cron jobs for the current user | `crontab -l` |
| `crontab -e` | Edit cron jobs for the current user | `crontab -e` |
| `crontab -r` | Remove all cron jobs for the current user | `crontab -r` |
| `crontab -u [user] -e` | Edit cron jobs for a specific user (requires sudo) | `sudo crontab -u student -e` |
| `tail -f /var/log/cron` | Monitor active cron job execution logs | `sudo tail -f /var/log/cron` |

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **Cron job command fails, but runs fine in terminal** | Environment paths (`PATH`) differ. Cron runs in a minimal shell. | Use absolute paths for all commands (e.g., `/usr/bin/tar` instead of `tar`). Set variables at the top of the crontab. |
| **Script redirects error, but cron mail shows errors** | Output redirecting missed stderr stream. | Redirect standard output and error: `/path/to/script.sh > /dev/null 2>&1` or output to log file. |
| **Anacron jobs not executing on a virtual machine** | Anacron utility missing or vm running 24/7 (handled by cron instead). | Ensure `anacron` package is installed. Verify config in `/etc/anacrontab`. |

---

## Interview Questions

**Q1: What does `*/15 2 1-10 * * /usr/local/bin/backup.sh` cron expression translate to?**
> A: This job will run:
> - Every 15 minutes (`*/15`)
> - In the 2:00 AM hour (`2`)
> - From the 1st to the 10th day of every month (`1-10`)
> - Regardless of the month or day of the week.

**Q2: What is the main difference between cron and anacron?**
> A: **cron** runs continuously and executes jobs at precise absolute times; if the server is off when a job is scheduled, the job is missed. **anacron** is designed for desktops/servers that are shut down; it runs periodically and ensures missed jobs are executed as soon as the system boots up.

**Q3: How do you verify if user `alice` has any cron jobs scheduled without switching to her account?**
> A: Use the command `sudo crontab -u alice -l`.

**Q4: Why does a cron job fail when calling a command like `service httpd restart` without absolute paths?**
> A: Cron executes commands under a very restricted shell environment with a minimal `PATH` variable (typically only `/usr/bin:/bin`). Commands stored in `/usr/sbin` or custom binary paths won't be resolved unless specified with absolute paths (e.g., `/usr/sbin/service`) or path variables are explicitly declared inside the script/crontab.

---

## Related Notes
- [[02-Operating-Systems/03-Windows-OS/Task-Scheduler]]
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics]]
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Automation-and-Scripting]]
