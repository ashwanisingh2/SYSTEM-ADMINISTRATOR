---
tags: [python, automation, scripting, intermediate]
aliases: [python-automation, py-sysadmin]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-green]
> 🐍 **PYTHON SCRIPTING**

`#complete` `#intermediate` `#none`

# PY-02: Python System Automation

> [!abstract] Overview
> Python ek powerful scripting language hai jo system administration tasks ko automate karne ke liye widely use hoti hai. Yeh note cover karta hai ki kaise OS module, subprocess, aur file handling ka use karke daily tasks ko script kiya jaye, aur ek support engineer ko iska knowledge hona kyun zaroori hai. In skills ka use karke aap ghanto ka manual kaam seconds mein complete kar sakte hain.

---
## 🧠 Concept Overview

- **What it is** — Python scripts likhna jo operating system ke saath interact karte hain, files manage karte hain, aur external commands run karte hain. Isme text parsing aur basic logic lagana bhi shamil hai.
- **Why it matters** — Manual repetitive tasks ko script karne se time bachta hai aur human error kam hota hai. Real job mein roz ke log analysis, user creation, server health checks, ya backup validation ko automate karna padta hai.
- **Where you see this** — Server backups, bulk file renaming, log parsing, system resource monitoring, aur cron jobs jo background mein data sync karte hain.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Existing Python scripts ko run karna, unka output check karna, aur errors aane par escalate karna. |
| **L2** | Scripts mein minor changes karna, cron job setup karna, variables modify karna, aur basic parameters pass karna. |
| **L3** | Complex automation pipelines design karna, error handling likhna, APIs integrate karna (jaise AWS boto3 ya Datadog API), aur enterprise scale par deployment karna. |

> [!tip] Seedha Simple Mein
> *Jaise aap GUI mein click karke files move karte ho ya task manager dekhte ho, Python ke through aap code likh kar wahi sab auto-pilot par daal dete ho. Ek baar code likho, hazaar baar chalao.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Python Script** is like a **Smart Factory Robot** because...
>
> - Factory mein worker har item ko manually pack karta hai (yeh manual admin task hai).
> - Robot ko ek baar set up kar do, toh wo hazaaron items bina thake, bina galti kiye pack karta hai (yeh Python automation hai).
> - Agar koi kharab item aaye toh robot alarm bajata hai ya usko alag bin mein daal deta hai (yeh Exception / Error Handling hai).

---
## 🔬 Technical Deep Dive

### 1. The OS and Shutil Modules

> [!info] Key Concept
> `os` module Python ko operating system se baat karne ki taqat deta hai, jaise file paths banana, directories list karna, aur environment variables padhna. `shutil` high-level file operations (copy, move, delete) ke liye use hota hai.

Python mein `os` aur `shutil` sabse common modules hain system admin ke liye.

```python
import os
import shutil

# Get current directory
current_dir = os.getcwd()

# List files
files = os.listdir('/var/log')

# Create a new directory
if not os.path.exists('/backup'):
    os.mkdir('/backup')

# Copy file
shutil.copy('/var/log/syslog', '/backup/syslog.bak')
```

> [!danger] Common Mistake
> Path directly string concatenation se mat banao (e.g., `dir + "/" + file`). Windows aur Linux ke path separators alag hote hain. Hamesha `os.path.join(dir, file)` ya nayi `pathlib` library ka use karo.

### 2. The Subprocess Module

> [!info] Key Concept
> `subprocess` module se aap Python script ke andar system commands (jaise bash commands, PowerShell commands) run kar sakte ho aur unka output capture kar sakte ho.

Yeh `os.system()` se bahut behtar aur safe hai.

```python
import subprocess

# Run a command and capture output
result = subprocess.run(['df', '-h'], capture_output=True, text=True)

if result.returncode == 0:
    print("Command ran successfully!")
    print(result.stdout)
else:
    print("Error occurred!")
    print(result.stderr)
```

> [!danger] Common Mistake
> `shell=True` ka use karna security risk (shell injection) ho sakta hai agar user input pass ho raha ho. Hamesha list of arguments pass karo `['ls', '-l', '/tmp']` without shell=True jab tak explicitly shell string execute karne ki zaroorat na ho.

### 3. File and Log Handling

> [!info] Key Concept
> Logs padhna aur likhna L1/L2 admin ka daily kaam hai. Python mein `open()` function ka use karke files ko efficiently read kiya ja sakta hai bina memory crash kiye.

```python
# Read log file line by line
with open('/var/log/syslog', 'r') as file:
    for line in file:
        if 'error' in line.lower() or 'critical' in line.lower():
            print("Found an issue:", line.strip())
```

> [!danger] Common Mistake
> File ko without `with` statement open karna ek bad practice hai. Agar script crash ho gayi, toh file memory mein open reh jayegi. `with open(...)` block apne aap file ko safely close kar deta hai chahe error aaye ya nahi.

### 4. Working with APIs (The `requests` module)

> [!info] Key Concept
> Aaj kal sysadmins ko APIs se baat karni padti hai (e.g. Jira mein ticket create karna, Slack par alert bhejna). Iske liye `requests` library sabse popular hai.

```python
import requests

response = requests.get('https://api.github.com')
if response.status_code == 200:
    print("API is UP!")
```

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Python 3.x installed hona chahiye (`python3 --version`).
> - Text editor (Vim, Nano, ya VS Code).
> - Basic understanding of variables and loops in Python.

### Step 1: Create a System Health Check Script

Hum ek script banayenge jo disk space check karegi aur alert degi agar 80% se zyada use ho gaya ho. Yeh daily cron job ke liye perfect hai.

```bash
# Script create karo
nano health_check.py
```

### Step 2: Write the Python Code

*Python Code in `health_check.py`:*

```python
import shutil
import sys

def check_disk_usage(disk):
    # Get disk stats
    usage = shutil.disk_usage(disk)
    free = usage.free / (1024 ** 3)  # Convert to GB
    total = usage.total / (1024 ** 3)
    percent_free = (free / total) * 100
    
    # Alert if less than 20% free (80% used)
    if percent_free < 20:
        print(f"CRITICAL: Low disk space! Only {percent_free:.2f}% free on {disk}")
        sys.exit(2) # Return critical code
    else:
        print(f"OK: Disk is healthy. {percent_free:.2f}% free on {disk}")
        sys.exit(0) # Return success code

# Check root partition
check_disk_usage('/')
```

### Step 3: Run the Script and Check Exit Code

```bash
# Script ko execute karo
python3 health_check.py

# Exit code check karo (0 means success)
echo $?
```

> [!success] Expected Output
> ```
> OK: Disk is healthy. 45.20% free on /
> 0
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `os.getcwd()` | Current working directory batata hai | `pwd_path = os.getcwd()` |
| `os.listdir()` | Directory ke contents list karta hai | `files = os.listdir('/tmp')` |
| `os.path.join()` | Paths ko safely combine karta hai | `path = os.path.join('dir', 'file.txt')` |
| `os.remove()` | File delete karta hai | `os.remove('/tmp/old_log.txt')` |
| `subprocess.run()` | External command run karta hai | `subprocess.run(['ls', '-l'])` |
| `shutil.copy()` | File ko ek jagah se doosri jagah copy karta hai | `shutil.copy('src.txt', 'dest.txt')` |
| `shutil.disk_usage()`| Disk space ki information deta hai | `info = shutil.disk_usage('/')` |
| `sys.exit()` | Script ko terminate karke return code set karta hai | `sys.exit(1)` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `ModuleNotFoundError: No module named 'requests'` | Third-party module installed nahi hai. | `pip install requests` run karein. |
| `PermissionError: [Errno 13] Permission denied: '/var/log/syslog'` | Script ko system file read karne ki permission nahi hai. | Script ko `sudo python3 script.py` se run karein, ya user permissions theek karein. |
| `FileNotFoundError: [Errno 2] No such file or directory` | Jo file path script mein diya hai, wo system par exist nahi karta. | Path check karein, absolute path use karein (`/path/to/file` instead of `file`). `os.path.exists()` se path verify karein. |
| Script hangs / doesn't finish | Script kisi infinite loop mein hai ya subprocess return ka wait kar raha hai (jaise koi command prompt open ho gayi). | `timeout` parameter add karein `subprocess.run(timeout=10)` mein, ya manually `Ctrl+C` press karke kill karein. |
| `IndentationError: expected an indented block` | Python code formatting par bhaut strict hai, spaces aur tabs mix ho gaye hain. | Apne editor mein set karein ki Tab button hamesha 4 spaces generate kare. Spaces use karna best practice hai. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Disk Space Filling Up Fast

> [!example] Ticket
> "Server PROD-DB-01 is alerting for low disk space on `/var/log` every night. Needs manual cleanup."

**L1 Response:** L1 engineer server pe login karke manual `rm` command se old logs delete karta hai temporary fix ke liye.
**Escalation Trigger:** Agar issue bar-bar aaye aur manual delete time-consuming ho jaye, toh monitoring team ticket L2 ko bhejti hai.
**L2 Resolution:** L2 admin ek Python script likhta hai jo `os.walk` aur `os.stat` use karke 7 din se purane `.log` files ko find karta hai aur `os.remove` se delete kar deta hai. Isko daily cron job mein set kar diya jata hai: `0 2 * * * python3 /scripts/cleanup.py`.

### 🎫 Scenario 2: Massive User Creation Request

> [!example] Ticket
> "HR has sent an Excel/CSV file with 500 new employees. Please create their Active Directory/Linux accounts by tomorrow."

**L1 Response:** L1 manually 500 account banana shuru karta hai jo 2 din lega aur error prone hoga.
**Escalation Trigger:** Bulk request dekh kar L1 is task ko automate karne ke liye ya help ke liye escalate karta hai.
**L2 Resolution:** L2 Python ka `csv` module use karta hai file read karne ke liye aur `subprocess` use karke loop mein `useradd` (Linux) ya PowerShell script (Windows) call karta hai, jisse yeh bulk task 5 minute mein bina galti ke complete ho jata hai.

### 🎫 Scenario 3: Application Process Crashing Randomly

> [!example] Ticket
> "The billing application randomly stops responding. Restarting the service fixes it."

**L1 Response:** L1 portal pe dekhta hai, jab bhi app down hota hai, command line se service restart kar deta hai.
**Escalation Trigger:** Ye permanent fix nahi hai, aur L1 hamesha monitoring screen nahi dekh sakta. Night shift mein issue badh jata hai.
**L2 Resolution:** L2 ek Python watchdog script banata hai. Script har minute `requests.get()` karke app URL hit karti hai. Agar 200 OK HTTP code nahi aata, toh script `subprocess.run(['systemctl', 'restart', 'billing'])` chala deti hai aur ek email/Slack alert bhej deti hai admin team ko.

---
## 🎤 Interview Questions

> [!question] Q1: Python mein system commands run karne ke liye kaunsa module best hai, aur `os.system` kyun avoid karna chahiye?
> **Answer:** `subprocess` module best hai. `os.system` bahut purana function hai aur output capture nahi kar sakta, sirf exit code deta hai. `subprocess.run()` aapko stdout, stderr capture karne, error handling karne, aur timeouts lagane ki facility deta hai.

==**Exam Tip:** *Interview mein hamesha `subprocess` ka naam lo, `os.system` ko avoid karne ka reason (security/output capture) zaroor batao.*==

> [!question] Q2: Agar tumhe ek bhaut badi 10GB ki log file read karni hai Python mein aisi server par jisme sirf 2GB RAM hai, toh kaise karoge?
> **Answer:** Main file ko pura memory mein ek sath load nahi karunga (`file.read()` ya `file.readlines()` se). Uske badle main `with open()` use karunga aur line-by-line iterate karunga `for line in file:` taaki memory usage sirf ek line ke barabar (kuch bytes) hi rahe. Yeh memory-efficient generator pattern hai.

==**Exam Tip:** *Badi file ko `read()` se load karna sabse common galti hai jisse script crash hoti hai.*==

> [!question] Q3: `os.path.join` ka use kyun karte hain path banane ke liye? Directly string add kyu nahi karte?
> **Answer:** Kyunki alag-alag OS mein folder path separators alag hote hain (Windows mein `\`, Linux mein `/`). `os.path.join('folder', 'file.txt')` OS environment ke hisaab se sahi slash laga kar safe path banata hai jisse script cross-platform compatible ban jati hai.

> [!question] Q4: Ek Python script banani hai jo har roz raat 2 baje chale. Python mein sleep timer lagakar loop likhoge ya koi aur tool use karoge?
> **Answer:** Main Python mein sirf task ka code likhunga. Usko raat 2 baje chalane ka kaam Python mein timer lagakar nahi karunga kyunki agar server reboot ho gaya toh script band ho jayegi. Iske liye OS scheduler ka use karunga (Linux mein `cron` ya systemd timers, aur Windows mein Task Scheduler).

==**Exam Tip:** *Scripting interview mein " separation of concerns" samajhna zaroori hai. Scheduling ka kaam OS ka hai, script ka nahi.*==

---
## 🔗 Related Notes

- [[PY-01 Introduction to Python|PY-01 Introduction to Python]] — Basics of Python syntax and variables
- [[LNX-05 Cron Jobs and Scheduling|LNX-05 Cron Jobs and Scheduling]] — How to schedule these automation scripts in Linux
- [[LNX-10 Bash Scripting Basics|LNX-10 Bash Scripting Basics]] — The native OS alternative to Python for simpler tasks
- [[WIN-12 PowerShell Automation|WIN-12 PowerShell Automation]] — Equivalent automation skills for Windows environments
