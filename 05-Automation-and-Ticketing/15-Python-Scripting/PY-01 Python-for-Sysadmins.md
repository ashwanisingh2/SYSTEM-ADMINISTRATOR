---
tags: [desktop-support, automation, scripting, python, L2]
aliases: [python-for-sysadmins, py-01]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-yellow]
> 📜 **PYTHON SCRIPTING**

`#complete` `#intermediate` `#none`

# PY-01: Python for Sysadmins

> [!abstract] Overview
> This note covers Python scripting basics for system administration. It details core system interaction libraries (`os`, `sys`, `shutil`, `subprocess`), REST API interaction using the `requests` library, filesystem manipulation, and automation of repetitive tasks.

---
## 🧠 Concept Overview

- **What it is** — Python is a high-level, interpreted scripting language widely used in system administration, DevOps, and cloud engineering due to its readability, extensive standard library, and cross-platform support.
- **Why it matters** — While bash and PowerShell are standard, Python excels at complex text processing, interfacing with SaaS APIs, handling structured data formats (JSON, CSV, XML), and writing platform-agnostic automation tools.
- **Where you see this** — Automating file cleanups, query and parse API metrics from monitoring endpoints, triggering webhooks to Slack/Teams on alerts, and backing up configurations to cloud buckets.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Run existing scripts, pass parameters via CLI, verify Python install, install required libraries (`pip`) |
| **L2** | Write scripts for local file manipulation, make basic REST API calls, configure virtual environments |
| **L3** | Develop multi-threaded daemon scripts, robust API integrations with error handling, package as executables |

> [!tip] Seedha Simple Mein
> *Python ek powerful aur easy-to-read programming language hai. Sysadmins iska use files copy/delete karne, API connectivity test karne, aur repetitive tasks ko automate karne ke liye karte hain jab bash ya PowerShell complex ho jate hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Swiss Army Knife** is like **Python** because...
>
> - Just as a Swiss Army Knife has multiple specialized tools (screwdriver, knife, scissors) for different jobs.
> - Python has a massive standard library (`os`, `sys`, `json`, `subprocess`) allowing you to fix servers, talk to APIs, or parse text all from one single language!

---
## 🔬 Technical Deep Dive

### 1. Key Standard Libraries for Systems

> [!info] Key Concept
> Python has built-in libraries that allow it to interact natively with any Operating System.

- **`os`**: Provides functions to interact with the operating system (e.g., creating folders, environment variables).
- **`sys`**: Interacts with the Python runtime (e.g., command-line arguments via `sys.argv`, exit codes).
- **`shutil`**: High-level file operations (copying, moving, archiving files and directory trees).
- **`subprocess`**: Spawns external processes and executes OS-level terminal commands.

> [!danger] Common Mistake
> Using `os.system()` to run commands. ==Always use `subprocess.run()` instead of `os.system()` because it handles standard output/error and is much safer against injection attacks.==

### 2. Working with JSON and REST APIs

- Standard APIs return JSON formatted data. Python's built-in `json` library converts JSON strings to Python dictionaries seamlessly.
- The `requests` library (installed via `pip`) simplifies sending HTTP requests (`GET`, `POST`, `PUT`, `DELETE`).

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Python 3.x installed.
> - Outbound internet access to query a public API.
> - `pip` installed to download external packages.

### Step 1: Initialize Virtual Environment

```bash
# Create directory
mkdir ~/python-sysadmin && cd ~/python-sysadmin

# Create venv
python3 -m venv venv

# Activate venv (Linux/Mac)
source venv/bin/activate
# On Windows PowerShell: .\venv\Scripts\Activate.ps1

# Install requests
pip install requests
```

> [!success] Expected Output
> Successfully installs `requests` and its dependencies.

### Step 2: Build a File Archiver Script

Create `archiver.py`:
```python
import os
import shutil
import sys

def archive_large_files(src_dir, dest_dir, max_size_mb):
    max_size_bytes = max_size_mb * 1024 * 1024
    if not os.path.exists(dest_dir):
        os.makedirs(dest_dir)
        
    for item in os.listdir(src_dir):
        item_path = os.path.join(src_dir, item)
        if os.path.isfile(item_path):
            file_size = os.path.getsize(item_path)
            if file_size > max_size_bytes:
                print(f"Archiving {item}...")
                shutil.move(item_path, os.path.join(dest_dir, item))

if __name__ == "__main__":
    archive_large_files("./src", "./archive", 1)
```

### Step 3: Write an API Query Script

Create `api_check.py`:
```python
import requests
import json
import sys

def get_public_ip():
    url = "https://api.ipify.org?format=json"
    try:
        response = requests.get(url, timeout=5)
        response.raise_for_status()
        data = response.json()
        print(f"My Public IP Address is: {data['ip']}")
    except requests.exceptions.RequestException as e:
        print(f"API Error occurred: {e}")
        sys.exit(1)

if __name__ == "__main__":
    get_public_ip()
```

Run: `python api_check.py`

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command / Function | 🛠️ Kya karta hai | 📝 Example |
|---------|-------------|---------|
| `os.path.exists(path)` | Check if path exists | `if os.path.exists('/etc/passwd'):` |
| `os.environ.get('VAR')` | Get environment variable | `home = os.environ.get('HOME')` |
| `shutil.copy(src, dest)` | Copy file | `shutil.copy('file.txt', '/tmp/')` |
| `subprocess.run(args)` | Run OS command | `subprocess.run(['ping', '-c', '4', '8.8.8.8'])` |
| `requests.get(url)` | HTTP GET request | `r = requests.get('https://api.github.com')` |
| `json.loads(string)` | Parse JSON | `data = json.loads(json_string)` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|---------|-------------|-----|
| `ModuleNotFoundError: No module named 'requests'` | Library not installed in environment | Run `pip install requests` after activating venv |
| `PermissionError` | User lacks write permissions | Run script with `sudo` or change folder owner |
| Subprocess output missing | Command passed as string without `shell=True` | Pass args as a list (e.g. `['ls', '-l']`) |
| `SyntaxError` on target host | Host running old Python 2.x | Run using `python3 script.py` explicitly |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Python Script Fails Intermittently

> [!example] Ticket
> "Our monitoring script queries the vendor API every 5 minutes but randomly crashes and the cronjob hangs."

**L1 Response:** Check script logs for the error message. Ensure the API endpoint is reachable.
**Escalation Trigger:** If it's a code issue or unhandled exception.
**L2 Resolution:**
- The script is likely missing a `timeout` parameter in `requests.get()`. Add `timeout=10`.
- Wrap the API call in a `try...except` block to catch `requests.exceptions.Timeout` and exit gracefully.

### 🎫 Scenario 2: Module Errors After OS Update

> [!example] Ticket
> "I just upgraded Ubuntu and now my Python script throws `ModuleNotFoundError` for everything!"

**L1 Response:** Confirm which version of Python is executing (`python3 --version`).
**Escalation Trigger:** If the packages are globally installed but missing.
**L2 Resolution:** 
- The OS upgrade probably updated the system Python version (e.g., 3.8 to 3.10), leaving old pip packages behind.
- Create a virtual environment (`python3 -m venv venv`), activate it, and reinstall dependencies from `requirements.txt`.

---
## 🎤 Interview Questions

> [!question] Q1: What is a Python virtual environment (`venv`) and why should you use it?
> **Answer:** A **virtual environment** is an isolated workspace containing its own copy of the Python executable and third-party packages installed via `pip`. It prevents package version conflicts between different scripts on the same server, ensuring that project-specific dependencies do not interfere with system-wide Python libraries.

> [!question] Q2: What is the difference between `subprocess.run()` and the old `os.system()`?
> **Answer:** `os.system()` simply passes the command to the subshell and only returns the exit code, making it difficult to capture output. `subprocess.run()` is the modern standard; it allows capturing standard output (`stdout`) and standard error (`stderr`), timeout management, and secure array-based parameter passing to avoid command injection vulnerabilities.

> [!question] Q3: How do you prevent a Python script scheduled via cron from overlapping if the previous run hasn't finished?
> **Answer:** Use a PID file or the `fcntl` module (on Linux) to ensure a single instance. If the script detects the lock file is in use, it exits immediately.

==**Exam Tip:** Understanding the `sys.argv` array is crucial for scripting. `sys.argv[0]` is always the script name, and arguments start at `sys.argv[1]`.==

---
## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-13 Bash Scripting|L-13 Bash Scripting]] — Scripting comparisons on Linux systems
- [[05-Automation-and-Ticketing/15-Python-Scripting/PY-02 Python-AD-Automation|PY-02 Python-AD-Automation]] — Managing Active Directory using Python
- [[05-Automation-and-Ticketing/15-Python-Scripting/PY-03 Python-Log-Analysis|PY-03 Python-Log-Analysis]] — Parsing system logs using Python regex
