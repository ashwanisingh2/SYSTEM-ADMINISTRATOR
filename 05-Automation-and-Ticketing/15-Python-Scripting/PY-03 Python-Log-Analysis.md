---
tags: [desktop-support, automation, logging, python, L2]
aliases: [python-log-analysis, py-03]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# PY-03: Python Log Analysis

> [!note] Overview
> This note covers parsing, filtering, and summarizing raw application and system log files (such as IIS, Apache/Nginx, Windows Event Logs, and Linux syslog) using Python. It focuses on using regular expressions (`re`), memory-efficient large file parsing, and data exportation.

---
## Concept Overview
- **What it is** — Log analysis in Python is the process of using scripts to search through text-based logs, identify patterns (like errors or security anomalies), extract specific data points, and compile them into reports or alerts.
- **Why it matters** — Enterprise servers generate gigabytes of log entries daily. Manually searching logs using text editors is slow and fails for large datasets. Python scripts automate this process, allowing engineers to isolate issues in seconds.
- **Real job encounter** — Parsing web server access logs to identify IPs executing brute force attacks, scanning syslogs for disk controller failure errors, and generating daily error rate reports.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Run existing log scanner scripts against specified log directories and review the generated reports for obvious error codes.
  - **Escalation Trigger**: Escalate if log parsing scripts run out of memory, crash on unknown character encodings, or when regex patterns fail to match modified log schemas.
  - **L2 Resolution**: Write custom regex filters, modify parser scripts to scan different logs, count occurrences of error events, and export anomalies to CSV.
  - **L3 Resolution**: Build automated real-time log ingestion processors, configure multi-file monitoring (similar to `tail -f`), parse nested JSON/XML logs, and integrate scripts with SIEM engines.

*Seedha simple mein: Server par hazaron lines ke logs generate hote hain jinhe manually dekhna impossible hai. Python script likh kar hum specific error codes (jaise 404, 500, ya Failed Logins) ko search kar sakte hain aur unhe list karke readable format me export kar sakte hain.*

---
## Technical Deep Dive

### 1. File Reading and Memory Management
When dealing with large production log files (several gigabytes), loading the entire file into memory using `file.read()` or `file.readlines()` will crash the system.
- **Memory-Efficient Method**: Read the file line-by-line using a `for` loop. This keeps only one line in memory at any given time.
```python
with open("huge_system.log", "r") as file:
    for line in file:
        process_line(line) # Consumes minimal RAM
```

### 2. Regular Expressions (`re` Module)
Python's `re` module uses pattern matching syntax to identify complex strings:
- **`\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}`**: Matches IPv4 addresses.
- **`\[(.*?)\]`**: Captures dates/timestamps inside square brackets.
- **`" (GET|POST) (.*?) HTTP/`**: Extracts HTTP methods and requested URLs.
- **`re.match()`** vs **`re.search()`**: `match` checks the start of the string, while `search` scans the entire line for a match.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - Python 3 installed.
> - A text editor to create scripts.

### Step 1: Create a Mock Log File
We will create a dummy web server log containing both success and error entries.

1. Create a script called `make_logs.py` and execute it to generate `web_server.log`:
```python
log_data = """192.168.1.50 - - [25/Jun/2026:10:00:01] "GET /index.html HTTP/1.1" 200 4500
10.0.0.12 - - [25/Jun/2026:10:00:05] "POST /login.php HTTP/1.1" 401 230
192.168.1.80 - - [25/Jun/2026:10:01:12] "GET /images/logo.png HTTP/1.1" 200 12000
172.16.5.99 - - [25/Jun/2026:10:02:18] "GET /api/v1/data HTTP/1.1" 500 1050
10.0.0.12 - - [25/Jun/2026:10:02:22] "POST /login.php HTTP/1.1" 401 230
192.168.1.50 - - [25/Jun/2026:10:03:00] "GET /dashboard.html HTTP/1.1" 200 8900
10.0.0.12 - - [25/Jun/2026:10:03:05] "POST /login.php HTTP/1.1" 401 230
192.168.1.99 - - [25/Jun/2026:10:04:15] "GET /admin HTTP/1.1" 403 0"""

with open("web_server.log", "w") as f:
    f.write(log_data.strip())
print("Mock log file created.")
```
2. Run command:
```bash
python make_logs.py
```

### Step 2: Write the Log Parser Script
We will write a script to search for client IP addresses, count how many times they accessed the server, and flag all status codes >= 400.

1. Create `log_parser.py`:
```python
import re
from collections import Counter
import csv

# Regular Expression Pattern for Common Log Format (CLF)
# Matches: IP - - [Date] "Method URL Version" StatusCode Bytes
LOG_PATTERN = r'^(\S+) \S+ \S+ \[(.*?)\] "(\S+) (\S+) \S+" (\d{3}) (\d+|-)'

def analyze_logs(log_file_path):
    ip_counter = Counter()
    failed_requests = []

    with open(log_file_path, 'r') as file:
        for line_num, line in enumerate(file, 1):
            match = re.match(LOG_PATTERN, line.strip())
            if match:
                ip_address = match.group(1)
                timestamp = match.group(2)
                method = match.group(3)
                url = match.group(4)
                status_code = int(match.group(5))
                
                # Track page hits per IP
                ip_counter[ip_address] += 1
                
                # If status code is an error (4xx or 5xx)
                if status_code >= 400:
                    failed_requests.append({
                        'line': line_num,
                        'ip': ip_address,
                        'time': timestamp,
                        'request': f"{method} {url}",
                        'status': status_code
                    })
                    
    # Print results
    print("\n--- Top Accessing IP Addresses ---")
    for ip, count in ip_counter.most_common(5):
        print(f"IP: {ip:<15} Hits: {count}")
        
    print("\n--- Failed HTTP Requests (4xx/5xx) ---")
    for req in failed_requests:
        print(f"Line {req['line']}: IP {req['ip']} -> {req['request']} returned status {req['status']}")
        
    # Export failed requests to CSV for further security audit
    with open('failed_logins_report.csv', 'w', newline='') as csvfile:
        fieldnames = ['line', 'ip', 'time', 'request', 'status']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(failed_requests)
    print("\nReport exported to failed_logins_report.csv")

if __name__ == "__main__":
    analyze_logs("web_server.log")
```
2. Execute the parser:
```bash
python log_parser.py
```
**Expected Output:** Displays top IP address stats, prints failed status codes, and exports the data to a CSV.

---
## Cheat Sheet / Quick Reference

| Operation | Regex Pattern | Purpose |
|---|---|---|
| IP Matching | `\b\d{1,3}(?:\.\d{1,3}){3}\b` | Extract IPv4 Addresses |
| Status Codes | `\b[45]\d{2}\b` | Search for HTTP 4xx and 5xx errors |
| Date Pattern | `\b\d{2}/[A-Za-z]{3}/\d{4}:\d{2}:\d{2}:\d{2}\b` | Parse standard Apache/IIS dates |
| Word Match | `\bERROR\b` or `\bFAIL\b` | Simple case-sensitive error match |
| Memory Read | `for line in open('log.txt')` | Stream lines sequentially |
| Output Export | `csv.DictWriter()` | Write structured logs directly to spreadsheets |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| Script crashes with: `UnicodeDecodeError`. | Log files contain non-UTF-8 characters (e.g. IIS logs are often UTF-16 or ANSI). | Specify correct encoding when opening the file. | `open(file, 'r', encoding='latin-1')` |
| Regex matches return empty list / fails. | The log format changed or the regex pattern has typos (e.g., extra spacing). | Print raw log line to console and test regex step-by-step using online tools like regex101. | N/A |
| Processing is extremely slow. | Recompiling the regex pattern inside the line-by-line parsing loop. | Compile the regex pattern once outside the loop using `re.compile()`. | `pat = re.compile(pattern)` |
| Execution fails with `MemoryError`. | Loading large files using `file.readlines()` or `file.read()`. | Iterate over the file object directly; avoid storing all lines in lists. | N/A |

---
## Interview Questions

> [!question] L1 Question
> **Q:** Why shouldn't you open a 50GB log file inside a text editor like Notepad?
> **A:** Text editors like Notepad try to load the entire file into the computer's Random Access Memory (RAM) all at once. For a 50GB file, this will exhaust system RAM, causing the editor to freeze, crash, and potentially bring down the host operating system. System admins should use command-line streaming tools like `tail`, `grep`, or Python scripts that process lines sequentially.

> [!question] L2 Question
> **Q:** What is the difference between `re.search()` and `re.findall()` in Python?
> **A:** `re.search()` scans the string to find the first location where the pattern matches, returning a single Match object (or `None`). `re.findall()` scans the entire string and returns all non-overlapping matches as a list of strings (or tuples of groups) in one execution.

> [!question] L3/Scenario Question
> **Q:** You have a web server cluster generating 100GB of access logs daily. You need a Python script to scan for sudden spikes in HTTP 5xx errors and send a Slack notification if errors exceed 100 in 1 minute. How would you design this?
> **A:** 
> - **Situation:** Real-time log monitoring and alerting under high volumes.
> - **Task:** Build an alert system with sliding time windows.
> - **Action:** 
>   1. **Tail File Implementation**: Write a python script using a generator that tracks file size and reads new lines as they are appended (similar to `tail -f`).
>   2. **Timestamp Aggregation**: Parse the timestamp of each HTTP 500 entry and store them in a deque (double-ended queue) from the `collections` module.
>   3. **Alert Evaluation**: For every new error, push its timestamp to the deque and pop all timestamps older than 60 seconds. If `len(deque) > 100`, trigger the alert.
>   4. **Webhook Integration**: Send a POST request containing the alert JSON payload to the Slack Incoming Webhook endpoint using the `requests` library.
> - **Result:** Real-time alerts are triggered without overloading system resources, reducing MTTR (Mean Time to Resolution).

---
## Seedha Simple Mein
*Seedha simple mein: Python scripts se massive size ke logs ko bina RAM crash kiye process kiya ja sakta hai. `for line in file` loop ke help se single single line read hoti hai. `re` library aur Regular Expressions ka use karke custom IP addresses ya error status codes (jaise 404 ya 500) dhundna bahut easy ho jata hai.*

---
## Related Notes
- [[02-Operating-Systems/03-Windows-OS/Event-Viewer|Event-Viewer]] — Windows built-in event logs interface.
- [[02-Operating-Systems/04-Linux-RHEL/L-07 Systemd and Journalctl|L-07 Systemd and Journalctl]] — System daemon logging command.
- [[05-Automation-and-Ticketing/15-Python-Scripting/PY-01 Python-for-Sysadmins|PY-01 Python-for-Sysadmins]] — Core Python automation library list.

---
*Tags: #desktop-support #automation #logging #python #L2*
