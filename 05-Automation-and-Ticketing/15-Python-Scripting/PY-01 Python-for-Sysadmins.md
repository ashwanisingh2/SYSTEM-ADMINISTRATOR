---
tags: [desktop-support, automation, scripting, python, L2]
aliases: [python-for-sysadmins, py-01]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# PY-01: Python for Sysadmins

> [!note] Overview
> This note covers Python scripting basics for system administration. It details core system interaction libraries (`os`, `sys`, `shutil`, `subprocess`), REST API interaction using the `requests` library, filesystem manipulation, and automation of repetitive tasks.

---
## Concept Overview
- **What it is** — Python is a high-level, interpreted scripting language widely used in system administration, DevOps, and cloud engineering due to its readability, extensive standard library, and cross-platform support.
- **Why it matters** — While bash and PowerShell are standard, Python excels at complex text processing, interfacing with SaaS APIs, handling structured data formats (JSON, CSV, XML), and writing platform-agnostic automation tools.
- **Real job encounter** — Automating file cleanups, query and parse API metrics from monitoring endpoints, triggering webhooks to Slack/Teams on alerts, and backing up configurations to cloud buckets.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Run existing Python scripts, pass parameters via command line, verify Python is installed, and install required libraries using `pip`.
  - **Escalation Trigger**: Escalate when scripts crash with environment path errors (`ModuleNotFoundError`), API token verification failures, or when modifications to scripts are required.
  - **L2 Resolution**: Write scripts for local file manipulation, parse and structure local log outputs, make basic REST API calls, and configure virtual environments.
  - **L3 Resolution**: Develop advanced multi-threaded daemon scripts, create robust API integrations with error handling/retries, package scripts as executables, and manage securely stored credentials using system secret stores.

*Seedha simple mein: Python ek powerful aur easy-to-read programming language hai. Sysadmins iska use files copy/delete karne, API connectivity test karne, aur repetitive tasks ko automate karne ke liye karte hain jab bash ya PowerShell complex ho jate hain.*

---
## Technical Deep Dive

### 1. Key Standard Libraries for Systems
- **`os`**: Provides functions to interact with the operating system (e.g. creating folders, environment variables).
- **`sys`**: Interacts with the Python runtime (e.g. command-line arguments via `sys.argv`, exit codes).
- **`shutil`**: High-level file operations (copying, moving, archiving files and directory trees).
- **`subprocess`**: Spawns external processes and executes OS-level terminal commands (replaces deprecated `os.system`).

### 2. Working with JSON and REST APIs
- Standard APIs return JSON formatted data. Python's built-in `json` library converts JSON strings to Python dictionaries seamlessly.
- The `requests` library (installed via `pip`) simplifies sending HTTP requests (`GET`, `POST`, `PUT`, `DELETE`) compared to standard Python `urllib`.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - Python 3.x installed.
> - Outbound internet access to query a public API.
> - `pip` installed to download external packages.

### Step 1: Install Dependencies and Setup Environment
We will configure a Python Virtual Environment to keep dependencies isolated.

1. Open your terminal and create a directory for your scripts:
```bash
mkdir ~/python-sysadmin && cd ~/python-sysadmin
```
2. Initialize a virtual environment named `venv`:
```bash
python3 -m venv venv
```
3. Activate the virtual environment:
   - **Linux/macOS:**
     ```bash
     source venv/bin/activate
     ```
   - **Windows (PowerShell):**
     ```powershell
     .\venv\Scripts\Activate.ps1
     ```
4. Install the `requests` library:
```bash
pip install requests
```
**Expected Output:** Successfully installs `requests` and its dependencies.

### Step 2: Build a File Archiver Script
We will write a script to traverse a directory, identify files larger than 1MB, and archive them.

1. Create `archiver.py`:
```python
import os
import shutil
import sys

def archive_large_files(src_dir, dest_dir, max_size_mb):
    # Convert size to bytes
    max_size_bytes = max_size_mb * 1024 * 1024
    
    if not os.path.exists(dest_dir):
        os.makedirs(dest_dir)
        print(f"Created destination directory: {dest_dir}")
        
    for item in os.listdir(src_dir):
        item_path = os.path.join(src_dir, item)
        # Check if it is a file and meets size threshold
        if os.path.isfile(item_path):
            file_size = os.path.getsize(item_path)
            if file_size > max_size_bytes:
                print(f"Archiving {item} ({file_size / (1024*1024):.2f} MB)...")
                shutil.move(item_path, os.path.join(dest_dir, item))

if __name__ == "__main__":
    # Example call
    archive_large_files("./src", "./archive", 1)
```
2. Run the script:
```bash
python archiver.py
```

### Step 3: Write an API Query Script
We will write a script that queries a REST API and prints formatted results.

1. Create `api_check.py`:
```python
import requests
import json

def get_public_ip():
    url = "https://api.ipify.org?format=json"
    try:
        response = requests.get(url, timeout=5)
        # Check HTTP response status code
        response.raise_for_status()
        
        # Parse JSON to Python dictionary
        data = response.json()
        print(f"My Public IP Address is: {data['ip']}")
    except requests.exceptions.RequestException as e:
        print(f"API Error occurred: {e}")
        sys.exit(1)

if __name__ == "__main__":
    get_public_ip()
```
2. Run the script:
```bash
python api_check.py
```
**Expected Output:** `My Public IP Address is: [Your Public IP]`

---
## Cheat Sheet / Quick Reference

| Module / Function | Purpose | Syntax Example |
|---|---|---|
| `os.path.exists(path)` | Check if path exists | `if os.path.exists('/etc/passwd'):` |
| `os.environ.get('VAR')` | Get environment variable | `home = os.environ.get('HOME')` |
| `shutil.copy(src, dest)` | Copy file to destination | `shutil.copy('file.txt', '/tmp/')` |
| `subprocess.run(args)` | Execute system command | `subprocess.run(['ping', '-c', '4', '8.8.8.8'])` |
| `requests.get(url)` | Send HTTP GET request | `r = requests.get('https://api.github.com')` |
| `json.loads(string)` | Parse JSON string to Dict | `data = json.loads(json_string)` |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| Script fails with: `ModuleNotFoundError: No module named 'requests'`. | The required library is not installed in the current Python environment. | Install the library using pip. Ensure virtual environment is activated. | `pip install requests` |
| Script fails with: `PermissionError` when writing to a folder. | The active system user executing the script lacks write permissions. | Run the script with elevated permissions, or change target folder owner. | `sudo python3 script.py` / `chmod +w <dir>` |
| Subprocess calls fail or command outputs are missing. | Commands passed as strings without `shell=True`, or command not present in environment system PATH. | Pass arguments as a list (e.g. `['ls', '-l']`) instead of string, or verify command location. | `which <cmd>` |
| Python script has syntactical errors on target host (e.g., SyntaxError). | Target host has an older Python 2.x runtime running. | Explicitly invoke using `python3` instead of `python`. | `python3 script.py` |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is a Python virtual environment (`venv`) and why should you use it?
> **A:** A **virtual environment** is an isolated workspace containing its own copy of the Python executable and third-party packages installed via `pip`. It prevents package version conflicts between different scripts on the same server, ensuring that project-specific dependencies do not interfere with system-wide Python libraries.

> [!question] L2 Question
> **Q:** What is the difference between `subprocess.run()` and the old `os.system()`?
> **A:** `os.system()` simply passes the command to the host's subshell and only returns the command's exit code, making it difficult to capture output or manage errors. `subprocess.run()` is the modern standard; it allows capturing standard output (`stdout`) and standard error (`stderr`), timeout management, and secure array-based parameter passing to avoid command injection vulnerabilities.

> [!question] L3/Scenario Question
> **Q:** You have a Python monitoring script scheduled via cron that queries a REST API and saves metrics. The script occasionally hangs indefinitely, causing subsequent cron runs to pile up and consume CPU. How do you resolve this?
> **A:** 
> - **Situation:** Unbounded HTTP connections causing scripts to hang.
> - **Task:** Limit process runtime and network connection limits.
> - **Action:** 
>   1. **HTTP Timeouts**: I will add explicit timeout parameters to the network calls: `requests.get(url, timeout=10)` (in seconds) to prevent infinite wait times.
>   2. **Single Instance Locking**: I will use a PID file or use the `fcntl` module (on Linux) to ensure that if a previous instance is still running, the new instance exits immediately:
>      ```python
>      import sys, fcntl
>      f = open('/tmp/monitor.lock', 'w')
>      try:
>          fcntl.lockf(f, fcntl.LOCK_EX | fcntl.LOCK_NB)
>      except IOError:
>          print("Another instance is running. Exiting.")
>          sys.exit(0)
>      ```
> - **Result:** Resource exhaustion is prevented, network queries timeout gracefully, and duplicate runs are blocked.

---
## Seedha Simple Mein
*Seedha simple mein: Python system administration tasks ko automate karne ka ek powerful tarika hai. `os` aur `shutil` se files handle hoti hain, aur `subprocess` se direct operating system commands run kiye ja sakte hain. Dependency conflicts se bachne ke liye virtual environments (`venv`) ka use zaroori hai.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-13 Bash Scripting|L-13 Bash Scripting]] — Scripting comparisons on Linux systems.
- [[05-Automation-and-Ticketing/15-Python-Scripting/PY-02 Python-AD-Automation|PY-02 Python-AD-Automation]] — Managing Active Directory using Python.
- [[05-Automation-and-Ticketing/15-Python-Scripting/PY-03 Python-Log-Analysis|PY-03 Python-Log-Analysis]] — Parsing system logs using Python regex.

---
*Tags: #desktop-support #automation #scripting #python #L2*
