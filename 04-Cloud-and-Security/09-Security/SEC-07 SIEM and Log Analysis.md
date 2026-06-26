---
tags: [security, siem, logs, advanced]
aliases: [siem, log-analysis]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-red]
> 🛡️ **SECURITY**

`#complete` `#advanced` `#none`

# SEC-07: SIEM and Log Analysis

> [!abstract] Overview
> *Is note mein hum SIEM (Security Information and Event Management) aur Log Analysis ke advanced concepts cover karenge. Ek system administrator aur security engineer ke liye logs padhna aur SIEM tools manage karna bohot zaroori hai taaki cyber attacks ko detect aur prevent kiya ja sake.*

---
## 🧠 Concept Overview

- **What it is** — SIEM is a centralized platform that collects, stores, and analyzes log data from various sources (servers, firewalls, applications) to detect security threats in real-time. Log Analysis is the process of examining these logs to identify anomalies.
- **Why it matters** — *Agar system mein koi breach hota hai, toh sabse pehle logs hi batate hain ki hacker ne kya kiya.* It helps in incident response, compliance, and threat hunting.
- **Where you see this** — Working with tools like Splunk, ELK Stack (Elasticsearch, Logstash, Kibana), IBM QRadar, or Microsoft Sentinel.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — *SIEM alerts ko monitor karna, false positives ko filter karna, aur basic logs check karna.* |
| **L2** | Configure, fix, escalate kab karta hai — *Custom alerts/dashboards banana, log sources integrate karna, aur actual incidents ko L3 pe escalate karna.* |
| **L3** | Architecture, design, enterprise-level — *SIEM architecture design karna, threat hunting queries likhna, aur advanced correlation rules set up karna.* |

> [!tip] Seedha Simple Mein
> *SIEM ek CCTV camera room ki tarah hai jahan saare servers aur firewalls ki live recording (logs) aati hai. Jab koi suspicious activity hoti hai, toh yeh alarm bajata hai (alerts).*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A SIEM System** is like **a centralized security control room in a huge mall** because...
>
> - Just like CCTV cameras in a mall send video feeds to the control room, servers and firewalls send logs to the SIEM.
> - The security guard looking at the screens is the SIEM tool analyzing patterns. If someone is trying to open locked doors repeatedly, the guard sounds an alarm (SIEM Alert).
> - If an incident occurs, the video recording (historical logs) is checked to see what exactly happened.
> - *Agar guards (rules) theek se set nahi hain, toh bohot saare false alarms bajenge (False Positives).*

---
## 🔬 Technical Deep Dive

### 1. The Core Components of SIEM

> [!info] Key Concept
> A SIEM solution typically consists of Log Collection, Log Parsing/Normalization, Storage, Correlation, and Alerting.

- **Log Collection (Ingestion):** Agents (like Splunk Universal Forwarder or Filebeat) are installed on endpoints to send logs to the SIEM. Syslog is heavily used for network devices.
- **Log Parsing/Normalization:** *Har system alag format mein logs bhejta hai.* The SIEM parses these different formats and converts them into a standard schema (e.g., extracting IP addresses, usernames, and timestamps).
- **Correlation:** This is the brain of the SIEM. It links multiple events together. For example, 5 failed login attempts followed by 1 successful login from the same IP.
- **Alerting:** When a correlation rule is triggered, an alert is generated for the SOC (Security Operations Center) team.

> [!danger] Common Mistake
> *Saare logs ko bina filter kiye SIEM mein bhej dena.* This wastes storage space and increases licensing costs. Always filter out noisy and useless logs before ingestion.

### 2. Log Analysis Techniques

- **Pattern Matching:** Looking for known malicious patterns like SQL injection payloads in web server logs (`/var/log/nginx/access.log`).
- **Anomaly Detection:** *Agar ek user hamesha India se login karta hai, aur achanak US se login attempt hota hai, toh yeh anomaly hai.*
- **Time-based Correlation:** Detecting attacks like Brute Force by tracking the frequency of events within a short timeframe.
- **Threat Intelligence Integration:** Checking IPs in logs against a global database of known malicious IPs (like VirusTotal or AlienVault OTX).

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux server running Apache or Nginx
> - Access to `/var/log/`
> - Basic understanding of `grep`, `awk`, and `sed`

### Step 1: Investigating Failed SSH Logins

> [!info] Objective
> *Hum check karenge ki humare server pe kaun baar-baar SSH login fail kar raha hai.*

```bash
# Check the auth.log or secure log for failed SSH attempts and count them by IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
```

> [!success] Expected Output
> ```
>    150 192.168.1.105
>     45 10.0.0.50
>      3 172.16.5.10
> ```
> *Yahan dikh raha hai ki 192.168.1.105 ne 150 times password fail kiya hai (Brute Force attack).*

### Step 2: Analyzing Web Server (Apache) Access Logs

```bash
# Find the top 5 IP addresses hitting your web server
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head -n 5
```

> [!success] Expected Output
> ```
>   5023 203.0.113.5
>    400 198.51.100.2
>     50 192.0.2.10
> ```

### Step 3: Searching for Specific HTTP Errors (e.g., 404 or 500)

```bash
# Extract only lines that resulted in a 404 (Not Found) error
awk '($9 ~ /404/)' /var/log/apache2/access.log
```

### Step 4: Live Log Monitoring with Filtering

```bash
# Watch the syslog in real-time, but only show lines containing the word "error"
tail -f /var/log/syslog | grep -i "error"
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `tail -f /var/log/syslog` | Live logs dekhne ke liye (*real-time monitoring*) | `tail -f /var/log/nginx/access.log` |
| `grep -i "error" /var/log/messages` | Case-insensitive search for "error" in logs | `grep -i "failed" /var/log/auth.log` |
| `awk '{print $1}' log.txt` | Print the first column (usually IP address) from a log file | `awk '{print $1}' access.log \| sort \| uniq -c` |
| `journalctl -xe` | Systemd journal logs dekhne ke liye (detailed errors) | `journalctl -u sshd.service` |
| `zgrep "pattern" file.gz` | Compressed (`.gz`) log files ke andar search karne ke liye | `zgrep "panic" /var/log/syslog.2.gz` |
| `cut -d' ' -f1 access.log` | Extract specific fields using a delimiter | `cut -d' ' -f1 /var/log/httpd/access_log` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| SIEM is not receiving logs from a Linux server | Syslog service is stopped or firewall is blocking port 514. | `systemctl status rsyslog` check karein. Firewall pe port 514 (TCP/UDP) allow karein. |
| Log parsing is failing in SIEM | The log format has changed (e.g., application update changed the output). | Update the parser regex/schema in the SIEM to match the new format. |
| Disk is 100% full due to massive log files | Log rotation (`logrotate`) is not configured properly. | Check `/etc/logrotate.conf`. Manually clear old logs: `> /var/log/large_file.log` (Do not use `rm`). |
| SIEM alerts are too noisy (False Positives) | Correlation rules are too broad. | Tune the rules. *Exclude known safe IP addresses (e.g., vulnerability scanners).* |
| Logs showing wrong timestamp in SIEM | Timezone mismatch between the source server and SIEM. | *Server aur SIEM dono pe NTP (Network Time Protocol) configure karein.* |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: High Number of Brute Force Alerts

> [!example] Ticket
> "SIEM is generating hundreds of alerts for 'Multiple Failed Login Attempts' on Prod-DB-Server."

**L1 Response:** *Pehle verify karein ki attack sach mein ho raha hai ya koi automated script fail ho rahi hai.* Check the source IP and the target username.
**Escalation Trigger:** If the brute force attack is successful (followed by a successful login) or coming from an internal IP.
**L2 Resolution:** Isolate the source IP. Block the external IP on the firewall. Force a password reset if an account was compromised. *Fail2Ban install aur configure karein server pe.*

### 🎫 Scenario 2: No Logs Received from Core Switch

> [!example] Ticket
> "The network team reports that the Core Switch has not sent any Syslog data to the SIEM for the last 24 hours."

**L1 Response:** Log into the SIEM dashboard and search for the specific IP of the Core Switch. Confirm no logs are present. Ping the switch from the collector.
**Escalation Trigger:** If the SIEM collector is running fine, but traffic is blocked at the network level.
**L2 Resolution:** *Check karein ki collector pe port 514 open hai ya nahi using `tcpdump`.*
```bash
tcpdump -i eth0 port 514
```
Work with the network team to ensure the switch configuration points to the correct SIEM collector IP.

### 🎫 Scenario 3: Malware Beaconing Detected

> [!example] Ticket
> "Critical Alert: Internal host 10.5.5.50 is making repeated outbound connections to a known malicious C2 (Command & Control) IP every 5 minutes."

**L1 Response:** Search the SIEM for 10.5.5.50. Identify the MAC address, hostname, and the logged-in user. Verify the malicious IP in Threat Intel databases.
**Escalation Trigger:** This is a confirmed malware infection. Escalate to L2/L3 immediately.
**L2 Resolution:** *Is machine ko network se turant disconnect karein.* Collect forensic data. Scan the machine with EDR tools. Investigate how the malware got in (e.g., phishing email logs).

### 🎫 Scenario 4: "Impossible Travel" Alert

> [!example] Ticket
> "Alert: User 'asmith' logged in from New York, USA, and then 10 minutes later from London, UK."

**L1 Response:** Review the VPN logs and Office 365 logs for the user. Check if one of the IPs belongs to a corporate VPN or a known proxy.
**Escalation Trigger:** If both logins are confirmed from distinct geographical locations without a VPN, the account is compromised.
**L2 Resolution:** Reset the user's password. Revoke all active sessions. Enable/Enforce MFA (Multi-Factor Authentication). *User se contact karke recent activity verify karein.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between an Event and a Log?
> **Answer:** A **Log** is a raw record of an action that occurred on a system (e.g., `User Bob logged in at 10:00 AM`). An **Event** is a significant occurrence derived from logs that implies a change in state or a potential issue (e.g., `Multiple Failed Logins from same IP -> Brute Force Event`).

==**Exam Tip:** Every event is backed by logs, but not all logs are events.==

> [!question] Q2: How does a SIEM detect an attack that happens over a long period (e.g., "Low and Slow" attack)?
> **Answer:** A SIEM uses **Historical Correlation**. It stores logs over time and runs rules that look back over weeks or months. For example, 1 failed login per day for 30 days might not trigger a daily threshold, but a long-term correlation rule will catch it.

> [!question] Q3: What is "Log Normalization" and why is it important?
> **Answer:** *Normalization ka matlab hai alag-alag log formats ko ek standard format mein convert karna.* A Windows firewall and a Cisco ASA firewall write logs differently. Normalization extracts common fields (Source IP, Dest IP, Port) so you can write a single SIEM query that searches across all devices.

> [!question] Q4: If you see a lot of failed SSH logins followed by a successful one, what is your immediate action?
> **Answer:** This indicates a **Successful Brute Force Attack**. Immediate action: 
> 1. Disable the compromised account. 
> 2. Disconnect the affected server from the network.
> 3. Block the attacking IP on the edge firewall.
> 4. Check bash history and running processes on the server to see what the attacker did after logging in.

> [!question] Q5: What happens if your log partition (`/var/log`) gets 100% full?
> **Answer:** *System services (jaise syslogd, apache) crash ho sakti hain ya naye logs likhna band kar dengi.* Some critical services might refuse to start. We use `logrotate` to prevent this by compressing and deleting old logs.

> [!question] Q6: What is a False Positive in SIEM?
> **Answer:** A False Positive is when the SIEM generates an alert for a security incident, but it is actually legitimate activity. *Jaise koi authorized vulnerability scanner network scan kar raha ho aur SIEM usko attack samajh le.*

---
## 🔗 Related Notes

- [[SEC-01 Introduction to Cyber Security|Introduction to Cyber Security]] — Basic security concepts.
- [[SEC-03 Firewall Rules and Network Security|Firewall Rules and Security]] — How to block the malicious IPs found in logs.
- [[LIN-12 System Monitoring and Logging|Linux System Logging]] — Deep dive into Linux `syslog` and `journalctl`.
- [[WIN-15 Windows Event Viewer|Windows Event Logs]] — Understanding Windows specific security events.
- [[SEC-08 Threat Hunting Basics|Threat Hunting Basics]] — Next steps after analyzing logs.
