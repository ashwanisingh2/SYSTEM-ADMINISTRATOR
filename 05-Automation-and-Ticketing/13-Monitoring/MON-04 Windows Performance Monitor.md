---
tags: [windows-server, monitoring, performance, perfmon, desktop-support, data-collector-sets]
aliases: [mon-04, perfmon, windows-performance-monitor]
created: 2026-06-26
status: "#complete"
difficulty: "#intermediate"
cert-relevant: "#md-102"
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS SERVER**

`#complete` `#intermediate` `#md-102`

# MON-04: Windows Performance Monitor

> [!abstract] Overview
> Windows Performance Monitor (`perfmon.exe`) is the built-in Windows GUI tool for collecting, viewing, and analyzing real-time and historical system performance metrics. It provides detailed counters for CPU, Memory, Disk, and Network — making it the go-to diagnostic tool for server slowness, capacity planning, and establishing performance baselines. Yeh note ek fresher L1 engineer ko sikhata hai ki kaise perfmon kholein, counters add karein, Data Collector Sets create karein, PowerShell se automate karein, aur baselines banayein. Har Desktop Support Engineer L1 se L3 tak perfmon use karta hai "server slow" tickets diagnose karne aur system health validate karne ke liye.

---

## 🧠 Concept Overview

- **What it is** — `perfmon.exe` is a built-in Windows GUI tool that lets you monitor real-time and historical system performance metrics through customizable counters, graphs, alerts, and data collector sets.
- **Why it matters** — Server slow tickets are the #1 complaint in any IT environment. Perfmon is the **definitive diagnostic tool** for identifying CPU spikes, memory leaks, disk bottlenecks, and network saturation with ==hard data — not guesswork==.
- **Where you see this** — Production server monitoring, pre/post-patch validation, capacity planning, troubleshooting application crashes, proving (with data) that a server needs more resources.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Opens perfmon, checks basic CPU/RAM live counters, takes screenshots and sends to L2 |
| **L2** | Creates Data Collector Sets, analyzes counter logs, establishes performance baselines, sets alerts |
| **L3** | Designs enterprise monitoring strategy, integrates perfmon with SCOM/Grafana, long-term capacity planning |

> [!tip] Seedha Simple Mein
> *Perfmon ek doctor ka stethoscope hai server ke liye. Jaise doctor heartbeat check karta hai, waise hi perfmon CPU, RAM, Disk, aur Network ki heartbeat check karta hai. Agar koi reading abnormal hai, toh problem ka pata lag jata hai.*

---

## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Car ka Dashboard** is like **Performance Monitor** because both show you real-time vital signs of a complex machine:
>
> - **Speedometer** = `% Processor Time` (CPU Usage) — *kitni fast engine chal raha hai*
> - **Fuel Gauge** = `Available MBytes` (RAM) — *kitna fuel (memory) bacha hai*
> - **Engine Temperature** = `Current Disk Queue Length` — *engine (disk) kitna garam ho raha hai under load*
> - **Odometer** = `Network Bytes Total/sec` — *kitna distance (data) travel ho raha hai*
>
> Just like a driver watches the dashboard for warning lights before the car breaks down, a sysadmin watches perfmon counters for server health before the server crashes. *Agar koi gauge red zone mein hai — action lo!*

---

## 🔬 Technical Deep Dive

### 1. Performance Monitor vs Other Tools

`perfmon.exe` is one of three built-in Windows monitoring tools. Each serves a different purpose:

| 🔧 Feature | 📊 Task Manager | 📈 Resource Monitor | 🖥️ Performance Monitor |
|---------|-------------|-----------------|-------------------|
| **Access** | `Ctrl+Shift+Esc` | `resmon.exe` | `perfmon.msc` |
| **Detail Level** | Basic | Moderate | Advanced |
| **Real-Time View** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Historical Logging** | ❌ No | ❌ No | ✅ Yes (Data Collector Sets) |
| **Custom Counters** | ❌ No | ❌ No | ✅ Yes (1000+ counters) |
| **Alerting** | ❌ No | ❌ No | ✅ Yes |
| **Baseline Comparison** | ❌ No | ❌ No | ✅ Yes |
| **Best For** | Quick glance, kill process | Per-process CPU/Disk/Network | Deep diagnosis, baselines, logging |
| **When to Use** | L1 first look | L1/L2 drill-down | L2/L3 serious troubleshooting |

> [!info] Key Concept — Kab kya use karna hai?
> - **Task Manager** — *Jab sirf dekhna hai ki kaunsa process zyada CPU/RAM kha raha hai (2-second job)*
> - **Resource Monitor** — *Jab specific process ki disk reads/writes ya network connections dekhni hain*
> - **Performance Monitor** — *Jab proper investigation karni hai, data collect karna hai, ya baseline banana hai*

==**Exam Tip:** Task Manager = Quick Triage, Resource Monitor = Process-Level Detail, Performance Monitor = Historical Data + Investigation. Yeh distinction interview mein zaroor puchte hain.==

---

### 2. Key Performance Counters (The Critical 4)

> [!warning] Caution
> Ye 4 counters har sysadmin ko yaad hone chahiye — interview mein bhi yahi puchte hain. Inhe ratta lagao!

| #️⃣ | ⌨️ Counter Path | 🛠️ Kya Monitor Karta Hai | ✅ Normal Range | ⚠️ Warning Threshold |
|---|-------------|----------------------|-------------|----------------------|
| 1 | `\Processor(_Total)\% Processor Time` | Overall CPU usage | 0–70% | **Above 85% sustained = Problem** |
| 2 | `\Memory\Available MBytes` | Free physical RAM | 1000+ MB | **Below 500 MB on server = Critical** |
| 3 | `\PhysicalDisk(_Total)\Current Disk Queue Length` | Disk I/O bottleneck | 0–1 | **Above 2 sustained = Bottleneck** |
| 4 | `\Network Interface(*)\Bytes Total/sec` | Network throughput | Varies | **Near NIC max capacity = Saturated** |

> [!danger] Common Mistake — "Sustained" ka matlab samjho
> Ek do baar 90% CPU aana normal hai (koi process start hua hoga). Lekin agar **15+ minutes** tak consistently 85%+ hai — tab investigation karo. ==Spikes are okay, **sustained high values** are not.==

**Additional Useful Counters:**

| ⌨️ Counter | 🛠️ Purpose |
|---------|---------|
| `\Memory\Pages/sec` | Page file activity *(high = RAM shortage, excessive paging ho rahi hai)* |
| `\Memory\% Committed Bytes In Use` | Total memory commitment percentage |
| `\PhysicalDisk(_Total)\Avg. Disk sec/Read` | Disk read latency *(above 20ms = slow disk)* |
| `\PhysicalDisk(_Total)\Avg. Disk sec/Write` | Disk write latency *(above 20ms = slow disk)* |
| `\Process(processname)\% Processor Time` | CPU usage by specific process |
| `\Process(processname)\Working Set` | RAM usage by specific process |
| `\System\Processor Queue Length` | Threads waiting for CPU *(above 2× cores = bottleneck)* |
| `\TCPv4\Connections Established` | Active TCP connections count |

==**Exam Tip:** Critical 4 counters yaad rakhna zaroori hai. Interview mein sabse pehle yahi puchte hain — "what counters would you monitor on a slow server?"==

---

### 3. Data Collector Sets (DCS)

> [!info] Key Concept
> Data Collector Sets (DCS) perfmon ka sabse powerful feature hai — ye counters ko automatically collect karke log files mein save karta hai. *Jaise CCTV camera record karta hai, waise ye counters ko record karta hai. Baad mein playback karke dekh sakte ho ki kab kya hua.*

**Types:**

| 📂 Type | 📝 Description |
|------|------------|
| **System** | Pre-built by Windows (System Diagnostics, System Performance) |
| **User-Defined** | Custom sets you create for specific monitoring needs |

**How to Create (GUI Method):**

1. Open `perfmon.msc`
2. Expand **Data Collector Sets** → Right-click **User Defined** → **New** → **Data Collector Set**
3. Name it (e.g., "Server_Baseline") → Select **Create manually (Advanced)**
4. Check **Performance counter** → Click **Next**
5. Add your 4 critical counters → Set sample interval (15 seconds recommended)
6. Choose save location → **Finish**

**Scheduling:**
- Right-click your Data Collector Set → **Properties** → **Schedule** tab
- Set start time, end time, and duration
- Can run daily, weekly, or on-demand
- **Best Practice:** Run a 24-hour baseline covering peak and off-peak hours

> [!tip] Pro Tip
> *DCS ka schedule set karo toh peak hours (9 AM – 6 PM) aur off-peak hours (midnight – 6 AM) dono cover karo. Isse true baseline milta hai.*

---

### 4. Performance Baselines

> [!info] Key Concept — Baseline kya hai?
> Baseline matlab "normal" ka snapshot. *Jab server healthy hai aur sab kuch theek chal raha hai — tab ki performance data baseline kehlati hai.* It's the reference point against which you measure everything.

**When to take a baseline:**
- ✅ After fresh OS install and all software deployed
- ✅ After major changes (RAM upgrade, application update, patch deployment)
- ✅ Before a known high-traffic period (month-end, quarter-end)
- ✅ After migration to new hardware/VM

**Why baselines are critical:**

| ❌ Without Baseline | ✅ With Baseline |
|-----------------|--------------|
| "CPU is at 75%... is that normal?" | "Baseline shows normal is 40%, current 75% — 35% spike to investigate" |
| "Server seems slow" (subjective) | "Response time 3x higher than baseline" (objective proof) |
| Can't plan capacity | "Trend shows RAM usage growing 5% monthly — need upgrade in 6 months" |

> [!tip] Pro Tip — Interview Gold
> *Agar interviewer puche "how would you troubleshoot a slow server" — baseline mention karna ==bonus points== deta hai. It shows you think proactively, not reactively.*

---

### 5. Performance Alerts

Perfmon can trigger actions when a counter crosses a threshold. *Ye proactive monitoring ka foundation hai — problem hone se pehle pata chal jata hai.*

**How to set up alerts:**

1. Open `perfmon.msc` → **Data Collector Sets** → **User Defined**
2. Create new set → Select **Performance Counter Alert**
3. Add counter (e.g., `\Processor(_Total)\% Processor Time`)
4. Set condition: **Alert when → Above → 90**
5. In **Alert Action** tab:
   - Log entry in Event Log
   - Start a Data Collector Set (for detailed capture)
   - Run a program/script (send email, create ticket)

**Alert Workflow Example:**

```text
Counter exceeds threshold
    → Perfmon logs event to Application Event Log (Event ID 2031)
    → Triggers a scheduled task
    → Scheduled task runs PowerShell script
    → Script sends email to sysadmin team
```

> [!danger] Common Mistake
> Alert threshold bahut low mat rakho (e.g., CPU > 50%) — otherwise har chhoti si spike pe alert aayega aur "alert fatigue" ho jayegi. ==Recommended: CPU > 85–90% sustained for 5+ minutes.==

---

### 6. PowerShell Integration

> [!info] Key Concept
> PowerShell se perfmon ka sara kaam command line se ho sakta hai — *automation aur remote monitoring ke liye essential hai. GUI kholne ki zaroorat nahi.*

**Key PowerShell Commands:**

| ⌨️ Command | 🛠️ Purpose |
|---------|---------|
| `Get-Counter` | Read live performance counters |
| `Export-Counter` | Save counter data to file (CSV, BLG, TSV) |
| `Import-Counter` | Read saved counter data files |
| `logman` | Create, start, stop Data Collector Sets |
| `typeperf` | Display counter data in real-time in terminal |

**Get-Counter Examples:**

```powershell
# List all available counter sets
Get-Counter -ListSet * | Select-Object CounterSetName | Sort-Object CounterSetName

# List all counters in a specific set
(Get-Counter -ListSet "Processor").Counter

# Get single counter reading
Get-Counter '\Processor(_Total)\% Processor Time'

# Continuous monitoring (every 2 seconds, 5 samples)
Get-Counter '\Processor(_Total)\% Processor Time' -SampleInterval 2 -MaxSamples 5

# Monitor multiple counters simultaneously
Get-Counter -Counter @(
    '\Processor(_Total)\% Processor Time',
    '\Memory\Available MBytes',
    '\PhysicalDisk(_Total)\Current Disk Queue Length'
) -SampleInterval 5 -MaxSamples 10
```

**Remote Server Monitoring:**

```powershell
# Monitor remote server CPU
Get-Counter -ComputerName "SERVER01" '\Processor(_Total)\% Processor Time'

# Monitor multiple servers simultaneously
Get-Counter -ComputerName "SERVER01","SERVER02","SERVER03" '\Memory\Available MBytes'
```

> [!tip] Pro Tip
> *Remote monitoring ke liye WinRM enabled hona chahiye target server pe. `Enable-PSRemoting -Force` run karo agar nahi hai.*

---

## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Windows Server 2022 or Windows 11 (works on Windows 10 too)
> - Administrator access
> - PowerShell 5.1+ (built-in)
> - At least 500 MB free disk space for log storage

### Step 1: Open Performance Monitor

```powershell
# Method 1: Run dialog
# Win+R > perfmon.msc > Enter

# Method 2: PowerShell direct launch
perfmon.msc

# Method 3: Quick system report (60-second auto diagnostic)
perfmon /report
```

> [!success] Expected Output
> Performance Monitor GUI opens with default System Summary view showing CPU, Disk, Memory, and Network overview.

---

### Step 2: Add Critical Counters to Live Graph

1. In perfmon, click **Performance Monitor** (under Monitoring Tools)
2. Right-click the graph area → **Add Counters** (or press `Ctrl+I`)
3. Add these 4 counters one by one:

| 📊 Counter to Add | 📂 Where to Find |
|---------------|---------------|
| `% Processor Time` | Processor → _Total |
| `Available MBytes` | Memory |
| `Current Disk Queue Length` | PhysicalDisk → _Total |
| `Bytes Total/sec` | Network Interface → (your NIC) |

4. Click **Add >>** for each → Click **OK**

> [!success] Expected Output
> Live graph shows 4 colored lines updating in real-time.

> [!tip] Pro Tip
> Right-click each counter in the bottom list → **Properties** → Change color and line width for better visibility. *CPU ko Red, Memory ko Blue rakhna easy identification ke liye.*

---

### Step 3: Create a User-Defined Data Collector Set

```powershell
# Create a new Data Collector Set using logman
logman create counter "Server_Baseline" `
    -c "\Processor(_Total)\% Processor Time" `
       "\Memory\Available MBytes" `
       "\PhysicalDisk(_Total)\Current Disk Queue Length" `
       "\Network Interface(*)\Bytes Total/sec" `
    -si 15 `
    -o "C:\PerfLogs\Baseline" `
    -f bincirc `
    -max 256
```

**Parameters explained:**

| 🔧 Parameter | 📝 Meaning |
|-----------|---------|
| `-c` | Counters to collect |
| `-si 15` | Sample interval = 15 seconds |
| `-o` | Output file path |
| `-f bincirc` | Format = binary circular (overwrites when full) |
| `-max 256` | Max file size = 256 MB |

> [!success] Expected Output
> ```
> The command completed successfully.
> ```

---

### Step 4: Start/Stop Data Collection

```powershell
# Start the data collector
logman start "Server_Baseline"

# Check status
logman query "Server_Baseline"

# Let it run for desired duration (e.g., 24 hours for a proper baseline)
# ...

# Stop the data collector
logman stop "Server_Baseline"
```

> [!success] Expected Output
> ```
> Name:                 Server_Baseline
> Status:               Running
> Root Path:            C:\PerfLogs\Baseline
> ```

---

### Step 5: View Reports

1. Open `perfmon.msc`
2. Navigate to: **Reports** → **User Defined** → **Server_Baseline**
3. Double-click the latest report to view collected data as graph
4. Right-click graph → **Properties** → Adjust time range for analysis

> [!info] Key Concept — Report Formats
> You can also export data: Right-click report → **Save Data As** → Choose CSV for Excel analysis or HTML for sharing with management.

---

### Step 6: PowerShell Live Monitoring

```powershell
# Get live CPU usage (5 samples, every 2 seconds)
Get-Counter '\Processor(_Total)\% Processor Time' -SampleInterval 2 -MaxSamples 5

# Get available memory
Get-Counter '\Memory\Available MBytes'

# Monitor all 4 critical counters
Get-Counter -Counter @(
    '\Processor(_Total)\% Processor Time',
    '\Memory\Available MBytes',
    '\PhysicalDisk(_Total)\Current Disk Queue Length',
    '\Network Interface(*)\Bytes Total/sec'
) -SampleInterval 5 -MaxSamples 10

# Export to CSV for analysis
Get-Counter -Counter '\Processor(_Total)\% Processor Time','\Memory\Available MBytes' `
    -SampleInterval 5 -MaxSamples 100 |
    Export-Counter -Path C:\PerfLogs\report.csv -FileFormat CSV
```

> [!success] Expected Output
> ```
> Timestamp                 CounterSamples
> ---------                 --------------
> 6/26/2026 1:30:00 PM      \\server\processor(_total)\% processor time : 23.456
> ```

> [!tip] Pro Tip — CSV se kya fayda?
> *CSV file Excel mein open karke graphs bana sakte ho — management ko report deni ho toh ye best format hai. Charts mein trend dikhao — "dekho CPU usage month-over-month badh raha hai, upgrade chahiye."*

---

### Step 7: Create Performance Alert via Task Scheduler

**Objective:** When CPU > 90% for 1 minute, trigger an alert script.

```powershell
# Step 7a: Create alert script
$alertScript = @'
$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
$cpu = (Get-Counter '\Processor(_Total)\% Processor Time').CounterSamples.CookedValue
$msg = "ALERT: CPU at $([math]::Round($cpu,2))% on $env:COMPUTERNAME at $timestamp"
Add-Content -Path "C:\PerfLogs\alerts.log" -Value $msg

# Optional: Send email (requires SMTP relay)
# Send-MailMessage -To "admin@company.com" -From "server@company.com" -Subject $msg -SmtpServer "smtp.company.com"
'@
$alertScript | Out-File -FilePath "C:\PerfLogs\CPU_Alert.ps1" -Encoding UTF8

# Step 7b: Create a perfmon alert Data Collector Set
logman create alert "CPU_Alert" `
    -th "\Processor(_Total)\% Processor Time>90" `
    -si 60 `
    -tn "CPU_High_Alert_Task"

# Step 7c: Create scheduled task to run the script
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-ExecutionPolicy Bypass -File C:\PerfLogs\CPU_Alert.ps1"
$trigger = New-ScheduledTaskTrigger -AtLogon
Register-ScheduledTask -TaskName "CPU_High_Alert_Task" -Action $action -Trigger $trigger `
    -Description "Triggers when CPU exceeds 90%" -RunLevel Highest
```

> [!success] Expected Output
> Alert system logs CPU spikes to `C:\PerfLogs\alerts.log` with timestamp.

==**Exam Tip:** Perfmon alerts ka workflow yaad rakhna — Counter threshold cross → Event Log entry → Scheduled Task trigger → PowerShell script runs. Yeh chain samajhna important hai.==

---

## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|---------|--------------|--------|
| `perfmon.msc` | Opens Performance Monitor GUI | `Win+R > perfmon.msc` |
| `perfmon /report` | Generates 60-second system health report | `perfmon /report` |
| `logman create counter` | Creates a Data Collector Set | `logman create counter "MySet" -c "\Processor(_Total)\% Processor Time" -si 15` |
| `logman start` | Starts data collection | `logman start "MySet"` |
| `logman stop` | Stops data collection | `logman stop "MySet"` |
| `logman query` | Lists all Data Collector Sets | `logman query` |
| `logman delete` | Deletes a Data Collector Set | `logman delete "MySet"` |
| `typeperf` | Displays counter data in terminal | `typeperf "\Processor(_Total)\% Processor Time" -si 2 -sc 10` |
| `typeperf -qx` | Lists all available counters | `typeperf -qx > counters.txt` |
| `Get-Counter` | PowerShell cmdlet for live counters | `Get-Counter '\Memory\Available MBytes'` |
| `Get-Counter -ListSet *` | Lists all counter sets | `Get-Counter -ListSet * \| Select CounterSetName` |
| `Export-Counter` | Exports counter data to file | `Get-Counter ... \| Export-Counter -Path report.csv -FileFormat CSV` |
| `Import-Counter` | Reads saved counter data | `Import-Counter -Path report.blg` |
| `relog` | Converts log file formats | `relog input.blg -o output.csv -f csv` |

---

## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| High CPU but no visible process in Task Manager | Hidden services, WMI provider issues, or system interrupts | `Get-Process \| Sort-Object CPU -Descending \| Select-Object -First 10` — also check **Deferred Procedure Calls** with `xperf` |
| Disk Queue Length consistently > 2 | Storage I/O bottleneck — too many reads/writes for disk to handle | Check IOPS with `winsat disk`, consider SSD upgrade, verify no antivirus full-scan running, check for fragmentation |
| Data Collector Set not starting | Insufficient disk space for logs, or permission issue on output folder | Verify free space on target drive, check folder permissions, run `logman query "SetName"` for error details |
| Counter data shows flat/zero lines | Corrupted Data Collector Set or counter path is wrong | Delete and recreate: `logman delete "SetName"` then `logman create counter "SetName" ...` |
| `perfmon.exe` crashes on open | Corrupted WMI repository | Run `winmgmt /salvagerepository` — if that fails, run `winmgmt /resetrepository` (==last resort — resets all WMI data==) |
| `Get-Counter` returns "No data" error | Counter path typo or counter not available on this OS version | Use `Get-Counter -ListSet "Processor"` to verify exact counter names |
| Performance data not matching Task Manager | Different sampling intervals and calculation methods | Normal behavior — perfmon samples at your configured interval, Task Manager updates every 1-2 seconds |
| Log files (.blg) won't open | File corrupted or incompatible version | Try `relog input.blg -o output.csv -f csv` to convert, or recreate the collector |
| `logman` command returns "Access Denied" | PowerShell not running as Administrator | Right-click PowerShell → **Run as Administrator**, then retry the `logman` command |

> [!bug] Troubleshooting Tip
> *Jab bhi perfmon ya logman mein error aaye — pehle PowerShell as Administrator open karo. 80% errors sirf permission ka issue hoti hain.*

---

## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: "Server bahut slow ho gaya hai"

> [!example] Ticket
> "Production server response time 10x slower since morning. Users unable to complete transactions."

**L1 Response:** Open `perfmon.msc`, add CPU and Disk Queue counters. If CPU > 95%, identify the process with `Get-Process | Sort CPU -Desc | Select -First 5`. Take screenshot of perfmon graph. Check if any scheduled tasks or patches running.

**Escalation Trigger:** If no single process is causing the spike, OR disk queue is consistently > 5, OR issue persists after killing non-critical processes.

**L2 Resolution:** Create a Data Collector Set for detailed 1-hour capture. Compare against last known baseline. Common root causes: antivirus full scan during business hours, Windows Update downloading in background, SQL query gone rogue, or a memory leak causing excessive paging. Fix: Reschedule AV scan, pause WSUS, identify and optimize the SQL query, restart leaking application.

---

### 🎫 Scenario 2: "Application keeps crashing with out-of-memory error"

> [!example] Ticket
> "ERP application crashes every 2-3 hours with 'Insufficient Memory' error. Server has 32 GB RAM."

**L1 Response:** Open perfmon → Check `\Memory\Available MBytes` counter. If below 200 MB, note which process is consuming the most RAM with `Get-Process | Sort WorkingSet64 -Desc | Select -First 5`. Document findings.

**Escalation Trigger:** If memory usage climbs steadily over time without releasing (memory leak pattern) — RAM goes from 8 GB used → 12 GB → 20 GB → 28 GB over hours.

**L2 Resolution:** Use perfmon counters `\Process(erp_app)\Working Set` and `\Process(erp_app)\Private Bytes` to confirm memory leak. Set up 24-hour Data Collector Set tracking the process. Temporary fix: Scheduled task to restart the application service during off-hours. Permanent fix: Report memory leak with perfmon evidence to application vendor.

> [!tip] Pro Tip — Evidence-Based Escalation
> *Perfmon ka data attach karo ticket mein — screenshots ya CSV export. "Server slow hai" se zyada impactful hai "CPU 95% sustained for 45 minutes, caused by sqlservr.exe, attached perfmon report." Ye professional approach hai.*

---

### 🎫 Scenario 3: "Server intermittently hangs during business hours"

> [!example] Ticket
> "File server freezes for 30–60 seconds randomly during 10 AM – 2 PM. Users get 'Network path not found' errors."

**L1 Response:** Open perfmon, add `\PhysicalDisk(_Total)\Current Disk Queue Length` and `\PhysicalDisk(_Total)\Avg. Disk sec/Read` counters. Monitor during reported hang window. Take screenshots of any spikes.

**Escalation Trigger:** Disk Queue Length > 5 sustained during hangs, and/or Avg. Disk sec/Read > 50ms consistently.

**L2 Resolution:** Create a Data Collector Set scheduled for 9 AM – 3 PM to capture the full window. Likely root causes: (1) Backup job running during business hours → reschedule to off-hours, (2) Antivirus scanning large shared folders → exclude data directories, (3) Disk hardware degradation → check `SMART` status and Event Log for disk errors. Replace disk if failing.

---

### 🎫 Scenario 4: "Network file transfers are extremely slow"

> [!example] Ticket
> "Copying files from server to client takes 10x longer than usual. Large CAD files (2–5 GB) timing out."

**L1 Response:** Open perfmon → Add `\Network Interface(*)\Bytes Total/sec` and `\Network Interface(*)\Output Queue Length` counters. Check if network is saturated (near NIC max capacity). Run `Get-NetAdapter` to verify link speed.

**Escalation Trigger:** Network consistently at 90%+ utilization, or Output Queue Length > 2 sustained.

**L2 Resolution:** Use perfmon to identify which process is consuming bandwidth. Common causes: (1) WSUS/Windows Update downloading patches → schedule downloads for off-hours, (2) Backup software using full bandwidth → configure QoS or bandwidth throttling, (3) NIC duplex mismatch → verify with `Get-NetAdapter | Select Name, LinkSpeed, MediaConnectionState`. Fix: Apply QoS policies, upgrade to 10 Gbps NIC, or segment traffic with VLANs.

---

## 🎤 Interview Questions

> [!question] Q1: What are the 4 critical performance counters every Windows sysadmin should monitor? Give their warning thresholds.
> **Answer:**
> 1. `\Processor(_Total)\% Processor Time` — **Above 85% sustained** indicates CPU bottleneck
> 2. `\Memory\Available MBytes` — **Below 500 MB** on a server is critical (system may start heavy paging)
> 3. `\PhysicalDisk(_Total)\Current Disk Queue Length` — **Above 2 sustained** indicates disk I/O bottleneck
> 4. `\Network Interface(*)\Bytes Total/sec` — Near the NIC's maximum capacity indicates network saturation
>
> *Ye 4 counters server ki vital signs hain — jaise doctor BP, heart rate, temperature, aur oxygen check karta hai. Inka threshold cross hona matlab patient (server) critical hai.*

==**Exam Tip:** Ye 4 counters yaad rakhna mandatory hai — almost every Windows admin interview mein puchte hain.==

---

> [!question] Q2: What is a Data Collector Set and how do you create one? Explain CLI and GUI methods.
> **Answer:** A Data Collector Set (DCS) is a group of performance counters that are collected together over a period of time and saved to log files for later analysis.
> - **GUI:** Open perfmon → Data Collector Sets → User Defined → Right-click → New → Data Collector Set → Add counters, set interval, set output path
> - **CLI:** `logman create counter "SetName" -c "\Processor(_Total)\% Processor Time" "\Memory\Available MBytes" -si 15 -o "C:\PerfLogs\output"`
> - **Start/Stop:** `logman start "SetName"` / `logman stop "SetName"`
>
> *DCS ek automatic data recorder hai — jaise CCTV camera record karta hai, waise ye counters ko record karta hai. Baad mein playback karke dekh sakte ho ki kab kya hua.*

---

> [!question] Q3: What is a performance baseline and why is it critical for troubleshooting?
> **Answer:** A performance baseline is a snapshot of system performance metrics taken when the server is operating normally and healthy. It's critical because:
> 1. It provides an **objective reference point** — without it, you can't tell if 75% CPU is normal or abnormal for that server
> 2. It enables **trend analysis** — compare current metrics against baseline to identify degradation
> 3. It supports **capacity planning** — track resource growth over months to predict when upgrades are needed
> 4. It speeds up **troubleshooting** — instead of guessing, compare current vs baseline to pinpoint what changed
>
> Best practice: Take baselines after fresh deployment, after major changes, and refresh quarterly.

---

> [!question] Q4: A user says the server is slow. Walk me through your perfmon-based diagnosis process.
> **Answer:** Step-by-step approach:
> 1. **Open perfmon** and add the 4 critical counters (CPU, RAM, Disk Queue, Network)
> 2. **Observe in real-time** for 5–10 minutes — identify which resource is bottlenecked
> 3. **If CPU is high** → Use `Get-Process | Sort CPU -Desc` to find the process → Is it expected (SQL, AV scan)?
> 4. **If RAM is low** → Check for memory leaks (`Working Set` growing over time for a single process)
> 5. **If Disk Queue is high** → Check what's doing I/O (antivirus? Windows Update? Backup job?)
> 6. **Compare against baseline** — Is this normal for this server or a deviation?
> 7. **Create a Data Collector Set** for a detailed capture if the issue is intermittent
> 8. **Document findings** with perfmon screenshots/CSV exports and attach to the ticket
>
> *Pehle vital signs check karo (4 counters), phir jo counter abnormal hai uska deep dive karo, phir baseline se compare karo, aur evidence ke saath escalate ya resolve karo.*

==**Exam Tip:** Yeh structured troubleshooting approach yaad rakhna — interviewer dekhta hai ki tumhara approach systematic hai ya random guessing.==

---

> [!question] Q5: What's the difference between Performance Monitor, Resource Monitor, and Task Manager? When do you use which?
> **Answer:**
>
> | 🔧 Tool | 📊 Detail Level | 📁 Historical Data | 🎯 Best Use Case |
> |------|-------------|----------------|---------------|
> | **Task Manager** | Basic | No | Quick glance — which process is using CPU/RAM, kill a hung process |
> | **Resource Monitor** | Moderate | No | Drill-down — see per-process disk I/O, network connections, CPU waits |
> | **Performance Monitor** | Advanced | Yes (DCS) | Deep investigation — baselines, logging, alerting, capacity planning |
>
> **Rule of thumb:** Start with Task Manager for quick triage → Resource Monitor for process-level detail → Performance Monitor for historical data and proper investigation.
>
> *Task Manager = thermometer (quick check), Resource Monitor = blood test (detailed check), Performance Monitor = full body scan (complete diagnosis with history).*

---

> [!question] Q6: How would you set up proactive monitoring using perfmon alerts?
> **Answer:**
> 1. Open `perfmon.msc` → Data Collector Sets → User Defined → New → **Performance Counter Alert**
> 2. Add the counter (e.g., `\Processor(_Total)\% Processor Time`)
> 3. Set threshold (e.g., "Alert when Above 90")
> 4. Set sample interval (e.g., 60 seconds to avoid false alarms from brief spikes)
> 5. In **Alert Action** tab — configure: Log to Event Log, Start a DCS for detailed capture, or Run a script
> 6. The script can send an email notification, create an ITSM ticket, or trigger a remediation action
>
> *Proactive monitoring ka matlab hai — problem hone se pehle alert aana chahiye. Jaise car mein low fuel warning light aati hai petrol khatam hone se pehle.*

---

> [!question] Q7: How do you monitor remote servers using PowerShell and perfmon?
> **Answer:**
> ```powershell
> # Single remote server
> Get-Counter -ComputerName "SERVER01" '\Processor(_Total)\% Processor Time'
>
> # Multiple servers at once
> Get-Counter -ComputerName "SRV01","SRV02","SRV03" '\Memory\Available MBytes'
> ```
> **Requirements:** WinRM must be enabled on remote servers (`Enable-PSRemoting -Force`), firewall must allow WinRM (TCP 5985/5986), and you need admin credentials on the remote machines.
>
> *Remote monitoring se ek hi console se 10-20 servers monitor kar sakte ho — har server pe individually login karne ki zaroorat nahi.*

---

> [!question] Q8: What is the `perfmon /report` command and when would you use it?
> **Answer:** `perfmon /report` generates an automatic 60-second system diagnostic report. It collects data on CPU, Disk, Memory, Network, and common configuration issues, then presents a formatted HTML report with warnings and recommendations.
>
> **When to use:**
> - Quick health check of a server before/after maintenance
> - Initial triage when a server slow ticket comes in
> - Pre-migration assessment of server health
>
> *Ye ek "quick scan" hai — 60 seconds mein server ki poori health report mil jaati hai. L1 engineer ke liye best starting point hai.*

==**Exam Tip:** `perfmon /report` yaad rakhna — yeh one-liner command interview mein impress karta hai. Shows you know quick diagnostic methods.==

---

## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-18 Linux Performance Monitoring|L-18 Linux Performance Monitoring]] — Linux side ka monitoring (top, htop, vmstat, iostat)
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-16 WSUS — Windows Server Update Services|WS-16 WSUS]] — WSUS downloads can cause CPU/Network spikes
- [[02-Operating-Systems/03-Windows-OS/WIN-08 Windows Services|WIN-08 Windows Services]] — Services that consume resources
- [[02-Operating-Systems/03-Windows-OS/WIN-03 Event Viewer|WIN-03 Event Viewer]] — Perfmon alerts log to Event Viewer
- [[05-Automation-and-Ticketing/13-Monitoring/MON-05 Azure Monitor and Log Analytics|MON-05 Azure Monitor and Log Analytics]] — Cloud-based monitoring evolution
