---
tags: [desktop-support, itsm, on-call, runbook, L2]
aliases: [on-call-runbooks, itsm-06]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #itil-v4
---

# ITSM-06: On-Call Runbooks

> [!note] Overview
> This note covers on-call operations and incident response runbooks. It details the incident lifecycle (from alert trigger to post-incident review), on-call communication channels, monitoring bridge lines, and the execution of emergency steps to contain and restore services.

---
## Concept Overview
- **What it is** — An On-Call Runbook is a step-by-step technical guide detailing how to acknowledge, diagnose, contain, and resolve critical service alerts during after-hours support rotations.
- **Why it matters** — System outages do not respect regular business hours. When critical servers fail at 2 AM, the on-call engineer must quickly resolve the issue without waiting for the primary systems architect. A runbook provides standard operating procedures, reducing error rates under pressure.
- **Real job encounter** — Receiving an automated SMS alert at 3 AM from PagerDuty because the company's customer-facing database disk space reached 98%, logging in remotely via VPN, and executing the disk clean-up runbook steps.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Monitor automated monitoring dashboard alerts, verify that on-call engineers acknowledge notifications, and direct incoming emergency user calls to the appropriate bridge lines.
  - **Escalation Trigger**: Trigger escalation when the primary on-call L2 engineer fails to acknowledge an alert within the 15-minute SLA limit, or when an outage escalates to affect multiple sites.
  - **L2 Resolution**: Acknowledge alerts, log in remotely, run diagnostic runbook scripts, execute standard remediation steps (e.g. service restarts, disk space cleanups), and notify stakeholders.
  - **L3 Resolution**: Resolve issues that fail standard runbook resolution steps, rebuild corrupted systems, lead the incident review meetings, and update the runbook documentation based on lessons learned.

*Seedha simple mein: On-Call Runbook ek ready-made guide hai jo systems engineer ko batati hai ki jab office time ke baad (raat ko ya weekend par) koi bada system crash hota hai, toh use step-by-step kaise thik karna hai. Isme system log files verify karne, restart commands chalane, aur emergency groups ko contact karne ke details hote hain.*

---
## Technical Deep Dive

### 1. Incident Response Lifecycle
Standard incident response models include:
- **Identification**: Monitoring tools (Nagios, Zabbix) detect failure and trigger an alert (via PagerDuty, Opsgenie, or SMS).
- **Acknowledge (ACK)**: The active on-call engineer marks the alert as acknowledged to stop escalation loops.
- **Containment**: Quick actions to stop the issue from spreading (e.g., block a compromised IP, route traffic to a secondary node).
- **Eradication & Recovery**: Restoring the main system to a healthy state (e.g., restart services, purge log folders).
- **Post-Incident Review (PIR)**: Analyzing the root cause (RCA) and implementing fixes to prevent recurrence.

### 2. Runbook Structure Best Practices
- **Prerequisites**: Clear system access requirements (VPN logins, required access permissions).
- **Verification Commands**: Commands to confirm the outage is real and check target systems status.
- **Remediation Steps**: Exact commands to execute, with failure/fallback loops.
- **Communications Guide**: Who to alert (management, customers) and templates for updates.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A remote terminal (SSH or PowerShell) connected to a mock target server.
> - Access to a simulated monitoring alert dashboard or system logs.

### Step 1: Receiving and Acknowledging the Alert
1. An automated system (like PagerDuty) triggers an alert:
```text
[ALARM] Host: WEB-SRV-01 | Service: Disk Space | Status: CRITICAL (99% Full) | SLA: 30m
```
2. **Acknowledge the alert**: Log into the alerting interface or reply to the SMS to change status to **Acknowledged**. This informs the team that an engineer is actively investigating.

### Step 2: Accessing the Target System and Verification
1. Log into the corporate VPN.
2. Establish an SSH connection to the target server:
```bash
ssh admin-user@web-srv-01.corp.local
```
3. Run the verification command to check the disk space allocation:
```bash
df -h /
```
**Expected Output:** Displays `/` filesystem usage at 99%.
```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   49G  1.0G  99% /
```

### Step 3: Executing the Remediation Steps
1. Navigate to the log directory where the issue is likely originating:
```bash
cd /var/log
```
2. Find the largest log files using a sorting command:
```bash
sudo du -sh * | sort -h
```
**Expected Output:** Large files are listed, showing `syslog.1` or a custom application log is consuming several gigabytes.

3. According to the runbook, do **not** delete active logs, as it can cause application errors. Instead, truncate the file to free up space instantly:
```bash
# Truncate log file safely
sudo truncate -s 0 /var/log/app_debug.log
```
4. Verify disk space again:
```bash
df -h /
```
**Expected Output:** Usage falls below 70%.

5. Clean up old compressed logs that are no longer needed:
```bash
sudo find /var/log -name "*.gz" -type f -mtime +14 -delete
```

### Step 4: Verification and Communication
1. Verify the application service is running:
```bash
sudo systemctl status apache2
```
2. Log into the monitoring console, manually resolve the alarm if it doesn't auto-resolve.
3. Update the ticket:
```text
Outage resolved. The root filesystem was 99% full due to app_debug.log. Truncated the log file to restore disk space. Apache is running normally. Will follow up with the dev team to disable debug logging in production.
```

---
## Cheat Sheet / Quick Reference

| Linux Command | Purpose | Example |
|---|---|---|
| `df -h` | Display disk space usage | `df -h` |
| `du -sh *` | Display size of files in directory | `du -sh /var/log/*` |
| `truncate -s 0 <file>` | Truncate file content to zero size | `sudo truncate -s 0 debug.log` |
| `find <dir> -mtime +X`| Locate files older than X days | `find /tmp -mtime +7` |
| `systemctl restart <svc>`| Restart application system service | `sudo systemctl restart nginx` |
| `journalctl -u <svc> -n 50`| View latest 50 logs for service | `journalctl -u docker -n 50` |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| SSH connection to server fails during after-hours call. | Corporate VPN is disconnected, or server lacks network access. | Verify VPN client status; try pinging the server gateway IP to locate path breaks. | `ping <server-ip>` |
| Truncating log file did not release disk space. | The application process is still holding a file handle on the deleted/truncated log. | Restart the application process to release the file handle, or check using lsof. | `sudo lsof \| grep deleted` / `systemctl restart <service>` |
| Runbook commands fail with "permission denied." | Current user lacks administrative permissions (sudo setup). | Request access elevation, or run commands with the sudo prefix if authorized. | `sudo <command>` |
| Alert is resolving but immediately re-triggering. | The threshold settings in monitoring tools are too narrow. | Adjust metric thresholds in Zabbix/Nagios to prevent alert loops. | N/A |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the first thing you should do when you receive an on-call alert from PagerDuty?
> **A:** The first action is to **Acknowledge (ACK)** the alert in the app or system dashboard. Acknowledging stops the notification escalation path, indicating to the team and management that a responder is active and investigating. It prevents the alert from waking up secondary on-call staff.

> [!question] L2 Question
> **Q:** If deleting a log file does not recover disk space on a Linux server, how do you fix it without rebooting the server?
> **A:** This happens because a running process still has a file handle open on the deleted log. The space is not reclaimed until the handle is closed. To fix it, I would run `sudo lsof | grep deleted` to identify the process holding the file open. Once identified, restarting that specific service (e.g., `sudo systemctl restart nginx`) will release the handle and immediately reclaim the disk space.

> [!question] L3/Scenario Question
> **Q:** An on-call alert indicates that the production SQL database is unresponsive. You log in and follow the runbook, but restarting the SQL service fails with an "Out of Memory" error. What do you do?
> **A:** 
> - **Situation:** Production SQL database fails to start due to out-of-memory errors, and runbook steps are exhausted.
> - **Task:** Restore service stability while operating outside the standard runbook.
> - **Action:** 
>   1. **Resource Check**: Run `free -m` and `top` to locate processes consuming system RAM.
>   2. **Terminate Leaks**: Identify and terminate non-essential processes (like test containers or backup scripts running in the background) that are hogging memory.
>   3. **Kernel Logs**: Check kernel logs using `dmesg | grep -i oom` to verify if the OS Out-Of-Memory Killer terminated SQL.
>   4. **Escalate**: If resources are clear and SQL still fails to start due to potential DB corruption, immediately initiate functional escalation to the Lead Database Administrator (DBA) and notify the Incident Commander.
>   5. **PIR Documentation**: After resolution, update the runbook to include memory check commands and database integrity verification procedures.
> - **Result:** System memory constraints are cleared, team support is mobilized, and future incident operations are enhanced.

---
## Seedha Simple Mein
*Seedha simple mein: On-call operations me system alerts (jaise disk space full hona ya service down hona) ko handle kiya jata hai. Runbook ek manual book ki tarah hai jise dekh kar engineeer system pe check karta hai ki kya issue hai aur use solve karta hai. Sabse pahle alert ko Acknowledge karna zaroori hai, taaki baki managers ko updates milti rahe.*

---
## Related Notes
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/ITSM-05 Escalation-Matrix|ITSM-05 Escalation-Matrix]] — Routing tickets to secondary tiers.
- [[02-Operating-Systems/04-Linux-RHEL/L-06 Managing Linux Storage|L-06 Managing Linux Storage]] — Managing disk volumes and space.
- [[05-Automation-and-Ticketing/13-Monitoring/MON-02 Zabbix-Complete-Guide|MON-02 Zabbix-Complete-Guide]] — Incident tracking alerts setup.

---
*Tags: #desktop-support #itsm #on-call #runbook #L2*
