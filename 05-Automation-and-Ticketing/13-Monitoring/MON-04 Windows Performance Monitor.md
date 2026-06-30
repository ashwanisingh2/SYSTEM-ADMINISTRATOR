---
tags: [windows, monitoring, performance, sysadmin]
aliases: [mon-01, perfmon]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS SERVER / MONITORING**

`#complete` `#intermediate` `#none`

# MON-01: Windows Performance Monitor

> [!abstract] Overview
> Windows Performance Monitor (`perfmon.exe`) is the built-in Microsoft tool for analyzing system performance. It provides real-time and historical data on CPU, Memory, Disk, and Network usage. Sysadmins use it to establish performance baselines, identify bottlenecks, configure Data Collector Sets for long-term logging, and set up alerts for when resource utilization exceeds healthy thresholds.

---
## 🧠 Concept Overview

- **What it is** — PerfMon is an MMC snap-in that tracks system performance using "Counters". Each counter monitors a specific hardware or software component (like the CPU's processor time or a specific disk's queue length).
- **Why it matters** — When users complain "the server is slow", Task Manager is often not enough. PerfMon allows you to dig deep, log data over days or weeks, and pinpoint exactly *which* resource (CPU, RAM, Disk I/O, or Network) is causing the bottleneck.
- **Where you see this** — Troubleshooting a slow SQL database server, analyzing why an IIS web server crashed during peak hours, or proving to management that a VM needs more RAM.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Opens Task Manager/Resource Monitor to check basic spikes, knows how to launch PerfMon and view real-time graphs |
| **L2** | Adds specific counters in PerfMon, creates Data Collector Sets for background logging, analyzes `.blg` files, configures basic alerts |
| **L3** | Establishes enterprise-wide performance baselines, integrates PerfMon data with central monitoring (like SCOM or Azure Monitor), correlates complex application issues with OS bottlenecks |

> [!tip] Seedha Simple Mein
> *PerfMon server ka ECG machine hai. Task Manager sirf current heart rate batata hai, par PerfMon pichle 24 ghante ka pura graph record karke batata hai ki server ko kab aur kyu attack aaya. Isse tum exact bimari (bottleneck) pakad sakte ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Running a Restaurant Kitchen** is like **Windows Performance Monitor** because...
>
> - **CPU (% Processor Time)** = The Head Chef. If he is 100% busy, orders get delayed.
> - **Memory (Available MBytes)** = The Kitchen Counter Space. If there is no space left, you can't prep new dishes.
> - **Disk (Disk Queue Length)** = The Waiter picking up food. If the queue is too long, cooked food sits waiting.
> - **Network (Bytes Total/sec)** = The Delivery Door. How fast food leaves and supplies come in.
> - **Data Collector Set** = The Kitchen Manager recording how the shift went over the whole weekend to find out why Friday night was a disaster.

---
## 🔬 Technical Deep Dive

### 1. The Core Four Counters

To troubleshoot any Windows server, you must check the "Core Four" bottlenecks.

> [!info] Key Concept
> A **Counter** is a specific metric. The format is `\Object(Instance)\Counter`. For example: `\Processor(_Total)\% Processor Time`.

| Resource | Critical Counter | Healthy Baseline | Warning Zone |
|----------|-----------------|------------------|--------------|
| **CPU** | `\Processor(_Total)\% Processor Time` | < 70% | > 85% sustained |
| **Memory** | `\Memory\Available MBytes` | > 10% of total RAM | < 5% of total RAM |
| **Disk** | `\PhysicalDisk(_Total)\Avg. Disk Queue Length` | < 2 per spindle | > 2 sustained |
| **Network**| `\Network Interface(*)\Bytes Total/sec` | < 70% of bandwidth | > 85% of bandwidth |

> [!danger] Common Mistake
> Looking at CPU spikes for 5 seconds and panicking. ==Spikes are normal. Sustained high usage (e.g., CPU > 90% for 10+ minutes) indicates a real bottleneck.==

### 2. Data Collector Sets (DCS)

Real-time monitoring is useless if the server crashes at 2 AM. **Data Collector Sets** run in the background, logging performance data to a file (usually `.blg` format).

- **System Performance**: Built-in DCS that generates a comprehensive diagnostic report.
- **Custom DCS**: You define exactly which counters to log, how often to sample (e.g., every 15 seconds), and when to stop logging.

### 3. Performance Alerts

You can configure PerfMon to trigger an action when a threshold is breached.
- For example: If `Available MBytes` drops below 1024, execute a script or write an Event to the Application Event Log.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server (or Windows 10/11 Pro/Enterprise) machine.
> - Administrator privileges to create Data Collector Sets.

### Step 1: Launch PerfMon and Add Counters

```cmd
# Run box or command prompt se PerfMon open karo
perfmon.exe
```

1. Expand **Monitoring Tools** -> **Performance Monitor**.
2. Click the **green plus (+)** icon at the top.
3. Select **Processor**, click **% Processor Time**, select `_Total`, and click **Add**.
4. Select **Memory**, click **Available MBytes**, and click **Add**.
5. Click **OK**.

> [!success] Expected Output
> You will see a live line graph tracking your CPU usage and Available RAM in real-time.

### Step 2: Create a Custom Data Collector Set

1. Expand **Data Collector Sets** -> **User Defined**.
2. Right-click **User Defined** -> **New** -> **Data Collector Set**.
3. Name it "SQL-Server-Baseline" and select **Create manually (Advanced)**.
4. Select **Create data logs** and check **Performance counter**.
5. Click **Add** and select the "Core Four" counters mentioned in the Deep Dive section.
6. Set Sample interval to **15 Seconds**.
7. Save to the default directory (`C:\PerfLogs\Admin\...`).
8. Select **Start this data collector set now** and click **Finish**.

> [!success] Expected Output
> The DCS starts running. A green play icon appears next to it. It is now silently logging data to `C:\PerfLogs`.

### Step 3: Analyze the Log File

1. Let the DCS run for 5 minutes, then right-click it and select **Stop**.
2. Go to **Performance Monitor**, click the **View Log Data** icon (cylinder icon or `Ctrl+L`).
3. Under the **Source** tab, select **Log files**, click **Add**, and browse to your `.blg` file in `C:\PerfLogs`.
4. Click **OK** to load the historical data into the graph.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `perfmon.exe` | Launches Performance Monitor GUI | `perfmon` |
| `perfmon /report` | Generates a 60-second system diagnostic report automatically | `perfmon /report` |
| `typeperf` | Command-line tool to write perf data to terminal or file | `typeperf "\Processor(_Total)\% Processor Time"` |
| `logman` | CLI to manage Data Collector Sets | `logman start "MyDCS"` |
| `resmon.exe` | Opens Resource Monitor (easier than PerfMon for quick checks) | `resmon` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|---------|-------|
| Cannot create Data Collector Set | User lacks permissions | Run `perfmon` as Administrator. User must be in the "Performance Log Users" group. |
| Log file (.blg) is massive and fills C: drive | Sample interval is too low (e.g., 1 second) over a long period | Increase sample interval to 15 or 60 seconds. Set a maximum size limit in DCS properties. |
| Specific counter is missing from the list | Windows performance counters are corrupted | Run `lodctr /r` in an elevated command prompt to rebuild the counter registry. |
| Disk Queue Length is high, but Disk % Time is low | SAN or Storage Array latency | The issue is on the storage network or physical SAN, not the local Windows OS. Escalate to Storage Team. |
| Alert action script doesn't run | Task Scheduler permissions | The alert uses Task Scheduler in the background; ensure the task runs as `SYSTEM` or an Admin service account. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Intermittent Server Slowness

> [!example] Ticket
> "The ERP application running on SERVER-01 gets extremely slow every day around 2:00 PM, but when IT checks at 3:00 PM, everything looks normal."

**L1 Response:** Check Event Viewer for errors around 2:00 PM. Check Task Manager current state.
**Escalation Trigger:** No obvious errors, but user complains daily. L1 cannot monitor 24/7.
**L2 Resolution:**
1. Configure a Data Collector Set to run from 1:00 PM to 4:00 PM logging CPU, Memory, and Disk IO.
2. Analyze the `.blg` file the next day.
3. Discover that a scheduled antivirus scan kicks off at 2:00 PM, causing the Disk Queue Length to spike to 50+.
4. Reschedule the AV scan to 2:00 AM. Ticket closed.

### 🎫 Scenario 2: Memory Leak Detection

> [!example] Ticket
> "Our custom Java application crashes every weekend. A server reboot fixes it temporarily."

**L1 Response:** Reboot the server to restore service. Verify application is running.
**Escalation Trigger:** The server requires a reboot every week, indicating a systemic issue.
**L2 Resolution:**
1. Open PerfMon and monitor `\Memory\Available MBytes` and `\Process(java)\Private Bytes`.
2. Observe that `Private Bytes` for the java process increases by 500MB every day and never releases it.
3. Confirm a memory leak. Send the PerfMon graphs to the application development team as proof.
4. Set up an automated scheduled task to restart the Java service on Friday nights as a temporary workaround until the devs patch the code.

### 🎫 Scenario 3: CPU Bottleneck Validation

> [!example] Ticket
> "Please add 4 more vCPUs to the Web Server, it's running slow." - Request from App Owner.

**L1 Response:** Forward the resource upgrade request to the virtualization team.
**Escalation Trigger:** Virtualization team rejects the request without proof of utilization.
**L2 Resolution:**
1. Run a DCS for 48 hours capturing `\Processor(_Total)\% Processor Time`.
2. Find that the average CPU usage is actually 15%, and it rarely spikes above 40%.
3. Add `\Network Interface\Bytes Total/sec` to the monitor. Find that the network adapter is maxing out at 1Gbps.
4. Inform the App Owner: The CPU is fine; the bottleneck is network bandwidth. No vCPUs added.

---
## 🎤 Interview Questions

> [!question] Q: What is the difference between Task Manager and Performance Monitor?
> **Answer:** Task Manager is for real-time, surface-level troubleshooting (what is happening *right now*). Performance Monitor is for granular, historical data collection and baseline creation. PerfMon can log data over days to a file and alert on specific thresholds, which Task Manager cannot do.

> [!question] Q: What is a good indicator of a Disk Bottleneck in PerfMon?
> **Answer:** The `Avg. Disk Queue Length` counter. If this number is consistently higher than 2 (per physical disk spindle), it means read/write requests are queuing up because the disk cannot process them fast enough.

> [!question] Q: How would you capture performance data if the issue only happens randomly at night?
> **Answer:** I would create a **Data Collector Set (DCS)** in Performance Monitor. I'd configure it to log the core counters (CPU, RAM, Disk, Network) with a 15-second interval and schedule it to run automatically overnight. In the morning, I can analyze the generated `.blg` log file.

> [!question] Q: A user complains their application is slow, but CPU and Memory are both under 30%. What do you check next?
> **Answer:** I would use PerfMon to check Disk I/O (`Disk Queue Length` and `Disk sec/Read`) and Network throughput (`Bytes Total/sec`). If those are also fine, the bottleneck might be the application code itself, database locking, or an external dependency (like a slow API).

==**Exam Tip:** Know how to rebuild performance counters using `lodctr /r`. It's a common exam and real-world fix for corrupted WMI/PerfMon issues.==

---
## 🔗 Related Notes

- [[02-Operating-Systems/03-Windows-OS/WIN-02 Device Manager|WIN-02 Device Manager]] — Checking physical hardware status.
- [[02-Operating-Systems/03-Windows-OS/WIN-08 Windows Services|WIN-08 Windows Services]] — Managing background processes.
- [[05-Automation-and-Ticketing/13-Monitoring/MON-04 Windows Performance Monitor|MON-04 Windows Performance Monitor]] — Alternate monitoring guide (legacy location).
- [[05-Automation-and-Ticketing/13-Monitoring/MON-05 Azure Monitor and Log Analytics|MON-05 Azure Monitor and Log Analytics]] — Cloud equivalent of PerfMon.
