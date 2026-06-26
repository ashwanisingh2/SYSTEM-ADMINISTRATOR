---
tags: [linux, rhel, performance]
aliases: [l-18, top, htop, vmstat]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX RHEL**

`#complete` `#intermediate` `#rhcsa`

# L-18: Linux Performance Monitoring

> [!abstract] Overview
> Jab ek Linux server slow chalta hai ya crash hota hai, toh ek system administrator ko diagnose karna padta hai ki bottleneck kahan hai: CPU, Memory, Disk I/O, ya Network. Yeh note aapko sikhayega ki `top`, `htop`, `vmstat`, `iostat` aur doosre tools use karke production servers pe resource bottlenecks kaise dhundhte hain aur fix karte hain.

---
## 🧠 Concept Overview

- **What it is** — Linux performance monitoring is the continuous or on-demand collection and analysis of system resource utilization metrics (CPU cycles, RAM footprint, Disk read/write rates, and Network packet flow).
- **Why it matters** — Servers often run out of capacity due to high web traffic, database queries, memory leaks, or heavy disk activities. A support engineer must identify the root cause immediately to restore services and prevent downtime. *Agar root cause pata nahi chalega, toh hum baar-baar server reboot karte rahenge.*
- **Where you see this** — Actual scenarios mein web app sluggish lagti hai, databases connections drop karte hain, batch backups ki wajah se high disk wait times aate hain, ya system memory full ho jati hai aur Linux kernel processes ko kill karna shuru kar deta hai (OOM Killer).

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Running basic checks like `top`, `free -h`, and `df -h` to see which service or partition is full/stuck. |
| **L2** | Inspecting Disk I/O with `iostat`, tracking network connections with `ss`/`netstat`, and identifying files locked by processes using `lsof`. |
| **L3** | Tuning kernel performance parameters via `sysctl`, fixing memory leaks, optimizing storage configurations (RAID/LVM). |

> [!tip] Seedha Simple Mein
> *Server slow hone par check karna padta hai ki problem CPU, RAM, Disk, ya Network mein se kiski wajah se hai. `top` aur `free` se resource usage dikhti hai, jabki `iostat` aur `ss` deep-level disk aur network status batate hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Linux server** is like a **busy restaurant kitchen** because...
>
> - **CPU** is the speed of the Chefs. If there are too many orders (processes), the chefs are overloaded (High Load Average).
> - **Memory (RAM)** is the Counter Space where chefs prepare dishes. If the counter space is full, they must move plates to a slow storage room downstairs (Swap Space on Disk), slowing down everything.
> - **Disk I/O** is the Dishwasher. If dishes aren't washed fast enough (I/O Wait), chefs must wait for clean plates before they can serve food, stalling the kitchen line.
> - **Network** is the Delivery Drivers. If the road outside is blocked (Network Bottleneck), food cannot reach customers, even though the kitchen is working perfectly.

---
## 🔬 Technical Deep Dive

### 1. Diagnosing CPU Bottlenecks

> [!info] Key Concept
> CPU bottlenecks occur when processes compete for execution time on the processor cores.

- **Load Average**: Shown in `top` (1, 5, and 15-minute averages). A load average higher than the number of logical CPU cores indicates process queuing.
- **`top` and `htop`**:
  - `top` is the native interactive task manager.
  - `htop` is a modern, color-coded interactive task manager.
- **Key Metrics**:
  - `%us` (User): CPU time spent running non-kernel processes (e.g., databases, web servers).
  - `%sy` (System): CPU time spent running kernel space processes (e.g., system calls).
  - `%wa` (I/O Wait): CPU time spent waiting for disk or network I/O.
  - `%id` (Idle): Unused CPU capacity.

> [!danger] Common Mistake
> Assuming high `%wa` means the CPU is slow. It actually means the CPU is idle, waiting for slow disk or network I/O to complete.

### 2. Diagnosing Memory (RAM) Bottlenecks

> [!info] Key Concept
> When physical RAM is completely exhausted, the OS uses virtual memory (Swap) located on physical disks.

- **`free -h`**: Always look at the **available** column, not "free" (Linux uses unused RAM for cache/buffers to speed up systems).
- **`vmstat` (Virtual Memory Statistics)**: Reports CPU, virtual memory, paging, block I/O.
  - Run `vmstat 1 5`. Look at `si` (swap in) and `so` (swap out). If these are constantly above zero, the system is actively swapping, causing severe slowdowns (Thrashing).

### 3. Diagnosing Disk I/O Bottlenecks

> [!info] Key Concept
> Slow storage response times choke the entire OS since the CPU has to wait for disk operations to complete.

- **`df -h`**: Checks partition space utilization. A 100% full partition crashes apps.
- **`iostat`**: Reports storage device statistics. Run `iostat -xz 1 5`.
  - `%util`: Over 90-100% means the disk is saturated.
  - `await`: Higher than 10-15ms indicates disk latency.
- **`iotop`**: Shows which specific processes are performing disk reads/writes in real time.

### 4. Diagnosing Network and Sockets

> [!info] Key Concept
> Network congestion, closed ports, or excessive open connections can make services appear down.

- **`ss` (Socket Statistics)**: Replaces `netstat`. `ss -tulpn` shows all listening TCP/UDP ports and associated processes.
- **`lsof` (List Open Files)**: Everything in Linux is a file. Sockets, hardware, and files are tracked here. `lsof -i :80` finds which process is using port 80.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A running RHEL/Rocky Linux VM.
> - EPEL repository installed for tools like `htop` and `iotop`.
>   ```bash
>   sudo dnf install -y epel-release && sudo dnf install -y htop iotop sysstat
>   ```

### Step 1: Analyze Memory Allocation

```bash
# Check memory in human-readable format
free -h

# Monitor system memory and paging every 2 seconds, 5 times
vmstat 2 5
```

> [!success] Expected Output
> ```
>               total        used        free      shared  buff/cache   available
> Mem:           3.7Gi       1.2Gi       1.1Gi        12Mi       1.4Gi       2.2Gi
> Swap:          2.0Gi       120Mi       1.9Gi
> ```

### Step 2: Identify High Disk Write Consumers

```bash
# Simulate a heavy write load in the background
dd if=/dev/zero of=/tmp/testfile bs=1M count=2000 oflag=direct &

# Immediately run iotop to see disk activity
sudo iotop -o -b -n 3
```

> [!success] Expected Output
> ```
> Total DISK READ :       0.00 B/s | Total DISK WRITE :     125.40 M/s
>   PID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
>  4567 be/4  root        0.00 B/s  125.40 M/s  0.00 %  94.50 % dd if=/dev/zero of=/tmp/testfile bs=1M count=2000 oflag=direct
> ```

```bash
# Clean up the generated test file
rm -f /tmp/testfile
```

### Step 3: Identify Open Network Ports

```bash
# List all active listening TCP and UDP ports
sudo ss -tulpn

# Find the specific PID holding port 22 (SSH) open
sudo lsof -i :22
```

> [!success] Expected Output
> ```
> COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
> sshd    1245 root    3u  IPv4  24501      0t0  TCP *:ssh (LISTEN)
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `top` | Classic real-time system process monitor. | `top` |
| `htop` | Interactive colorized task manager. | `htop` |
| `free -h` | Show RAM allocation and swap usage. | `free -h` |
| `df -h` | Check disk space usage on all mounted filesystems. | `df -h` |
| `vmstat 1 5` | Report memory and CPU statistics every 1s, 5 times. | `vmstat 1 5` |
| `iostat -xz 1 5` | Detailed disk I/O metrics and wait queues. | `iostat -xz 1 5` |
| `iotop -o` | Shows only processes actively executing Disk I/O. | `iotop -o` |
| `ss -tulpn` | Show listening TCP/UDP sockets and process names. | `ss -tulpn` |
| `lsof -i :80` | List processes holding network port 80. | `lsof -i :80` |
| `sar -u` | View CPU utilization history for the current day. | `sar -u` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| System is extremely slow, `%wa` is high. | A database is performing heavy unindexed writes or disk is failing. | Run `iotop -o` to find the process doing high writes. |
| Server terminates critical applications (e.g., MySQL/Java). | OOM Killer triggered because system ran completely out of RAM. | Check system logs: `dmesg \| grep -i oom`. Add Swap space or optimize application memory limit. |
| High CPU usage, but CPU idle time is high in `top`. | The load is caused by zombie or uninterruptible sleep (`D`) state processes. | Find `D` state processes in `top`. They are waiting for disk I/O to complete. Fix underlying storage. |
| Port already in use error when starting Apache/Nginx. | Another application daemon is already bound to that port. | Run `ss -tulpn \| grep :80` or `lsof -i :80` to find the conflicting process. Stop or reconfigure it. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: CPU Saturation due to Runaway Process
> [!example] Ticket
> "Customer complaints: Website loading is extremely slow. Server alerts indicate Rocky-Web-01 CPU utilization is at 99%."

**L1 Response:** Log in via SSH. Run `top` or `htop`. Press `P` in `top` to sort processes by CPU usage. Identify the runaway process name and PID.
**Escalation Trigger:** The service causing high CPU is the core database daemon (`mysqld`) that L1 cannot safely restart without impact.
**L2 Resolution:** Analyzed application queries. *Pata chala ek rouge Python script 100% CPU kha raha tha.* Killed the rogue user script using `kill -15 PID`. Checked `sar -u` history to identify when the spikes started to prevent recurrence.

### 🎫 Scenario 2: Memory Leak causing OOM Crash
> [!example] Ticket
> "The custom reporting API daemon keeps crashing unexpectedly twice a day. No daemon logs are created."

**L1 Response:** Log in to the server. Run `free -h` to check available memory. Checked kernel logs using `journalctl -k -g oom`.
**Escalation Trigger:** Confirmed OOM Killer killed the daemon, but memory optimization requires software stack changes.
**L2 Resolution:** Set up monitoring script. Ran `vmstat 1` to watch memory usage climb. Temporarily configured the daemon with a systemd memory limit (`MemoryMax=2G` in the unit file) so it restarts gracefully before bringing down the server.

### 🎫 Scenario 3: Disk I/O Choking the Server
> [!example] Ticket
> "The ERP application is extremely sluggish every night between 2:00 AM and 4:00 AM."

**L1 Response:** Check crontab using `crontab -l` to see if there is a scheduled job at 2:00 AM.
**Escalation Trigger:** Scheduled job is a necessary backup script.
**L2 Resolution:** Reviewed `sar` reports. Found `%wa` spiking to 95% at 2:00 AM. *Disk I/O itna high tha ki baaki processes wait kar rahe the.* Used `ionice` inside the cron job script to lower the I/O priority of the backup process: `ionice -c2 -n7 backup.sh`.

---
## 🎤 Interview Questions

> [!question] Q1: What does a high value in the %wa (I/O Wait) column of top mean? How do you investigate it?
> **Answer:** High `%wa` (I/O Wait) indicates that the CPU is idle because all runnable tasks are waiting on outstanding disk or network I/O operations to complete. It means you have a disk/network bottleneck, not a CPU speed bottleneck. You investigate it by running `iostat -xz 1 5` to identify which disk device has high queue sizes (`aqu-sz`) and high latency (`await`), and then run `iotop -o` to pinpoint the specific process.

==**Exam Tip:** High %wa means I/O bottleneck, not CPU bottleneck!==

> [!question] Q2: What is the difference between "Free" and "Available" memory in the free -h command?
> **Answer:** **Free** memory is RAM that is completely untouched. **Available** memory is the amount of memory that can be allocated to applications without causing swapping. Linux uses unused memory for disk buffers and caches (page cache) to optimize performance, which is technically "used" but will be immediately released if an application requests it. Hence, "Available" is the true metric of system health.

> [!question] Q3: How do you identify which process is listening on Port 80 and what files it has open?
> **Answer:** You can use `ss -tulpn | grep :80` or `lsof -i :80` to find the process name and PID listening on port 80. Once you have the PID (e.g., `4567`), you can run `lsof -p 4567` to list all files, network sockets, and libraries that this specific process currently has open.

> [!question] Q4: What is the Linux OOM Killer, and how does it select which process to terminate?
> **Answer:** The Out Of Memory (OOM) Killer is a kernel mechanism that activates when the system runs completely out of memory and swap. To prevent a kernel panic and keep the system alive, it scores processes using an `oom_score` (calculated based on memory consumption, process priority, and runtime). The process with the highest score (usually a memory-hogging database or app server) is terminated.

---
## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-07 Process Management|L-07 Process Management]] — Connected to CPU and Memory process handling.
- [[02-Operating-Systems/04-Linux-RHEL/L-12 Network Configuration in Linux|L-12 Network Configuration in Linux]] — Connected to network troubleshooting.
- [[02-Operating-Systems/04-Linux-RHEL/L-17 Linux Log Management|L-17 Linux Log Management]] — Connected to investigating system logs for performance issues.
