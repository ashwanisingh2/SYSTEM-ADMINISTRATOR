---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-18-linux-performance-monitoring, l-18]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# L-18 Linux Performance Monitoring

> [!abstract] Overview
> When a Linux server runs slow or crashes, system administrators must diagnose which subsystem is bottlenecked: CPU, Memory, Disk I/O, or Network. This note covers key performance monitoring utilities (`top`, `htop`, `vmstat`, `iostat`, `sar`, `free`, `df`, `lsof`, `ss`/`netstat`, `iotop`), and explains how to diagnose resource bottlenecks in production environments.

---

---
## Concept Overview
- **What it is** — Linux performance monitoring is the continuous or on-demand collection and analysis of system resource utilization metrics (CPU cycles, RAM footprint, Disk read/write rates, and Network packet flow).
- **Why it matters for a support engineer** — In production, servers often run out of capacity due to high web traffic, database queries, memory leaks, or heavy disk activities. A support engineer must identify the root cause immediately to restore services and prevent downtime.
- **Where you encounter this** — Web app is sluggish, databases are dropping connections, batch backups are causing high disk wait times, or system memory is full and the Linux kernel starts killing processes (OOM Killer).
- **L1 vs L2 vs L3:**
  - **L1**: Running basic checks like `top`, `free -h`, and `df -h` to see which service or partition is full/stuck, and noting high-level resource usage.
  - **L2**: Inspecting Disk I/O with `iostat`/`iotop`, tracking open network connections with `ss`/`netstat`, identifying files locked by processes using `lsof`, and inspecting historical trends using `sar`.
  - **L3**: Tuning kernel performance parameters via `sysctl`, fixing memory leaks, optimizing storage configurations (RAID/LVM), and setting up automated alerts for performance baselines.


---

---
## Technical Deep Dive
## Real-World Analogy
Think of a Linux server as a busy **restaurant kitchen**:
- **CPU** is the speed of the Chefs. If there are too many orders (processes), the chefs are overloaded (High Load Average).
- **Memory (RAM)** is the Counter Space where chefs prepare dishes. If the counter space is full, they must move plates to a slow storage room downstairs (Swap Space on Disk), slowing down everything.
- **Disk I/O** is the Dishwasher. If dishes aren't washed fast enough (I/O Wait), chefs must wait for clean plates before they can serve food, stalling the kitchen line.
- **Network** is the Delivery Drivers. If the road outside is blocked (Network Bottleneck), food cannot reach customers, even though the kitchen is working perfectly.

---

### 1. Diagnosing CPU Bottlenecks
CPU bottlenecks occur when processes compete for execution time on the processor cores.
- **Load Average**: Shown in `top` (three values representing 1, 5, and 15-minute averages). A load average higher than the number of logical CPU cores indicates process queuing.
- **`top` and `htop`**:
  - `top` is the native interactive task manager.
  - `htop` is a modern, color-coded interactive task manager that visualizes core utilization and process trees.
- **`sar` (System Activity Reporter)**: Collects and reports historical system activity. Use `sar -u` to view CPU history.
- **Key Metrics**:
  - `%us` (User): CPU time spent running non-kernel processes (e.g., databases, web servers).
  - `%sy` (System): CPU time spent running kernel space processes (e.g., driver actions, system calls).
  - `%wa` (I/O Wait): CPU time spent waiting for outstanding disk or network I/O operations to complete. High `%wa` points to disk bottlenecks, not CPU speed.
  - `%id` (Idle): Unused CPU capacity.

---
### 2. Diagnosing Memory (RAM) Bottlenecks
When physical RAM is completely exhausted, the operating system uses virtual memory (Swap) located on physical disks.
- **`free -h`**: Displays total, used, free, and available physical memory and swap. Always look at the **available** column, not "free" (Linux uses unused RAM for cache/buffers to speed up systems).
- **`vmstat` (Virtual Memory Statistics)**: Reports CPU, virtual memory, paging, block I/O, and trap activity.
  - Run `vmstat 1 5` to get updates every 1 second, 5 times.
  - Look at the `si` (swap in) and `so` (swap out) columns. If these values are constantly above zero, the system is actively swapping, causing severe slowdowns (Thrashing).

---
### 3. Diagnosing Disk I/O Bottlenecks
Slow storage response times choke the entire OS since the CPU has to wait for disk operations to complete.
- **`df -h`**: Checks partition space utilization. A 100% full partition will crash applications writing to it.
- **`iostat`**: Reports storage device statistics. Run `iostat -xz 1 5` for extended statistics.
  - `%util`: The percentage of CPU time during which I/O requests were issued to the device. Over 90-100% means the disk is saturated.
  - `await`: The average time (in milliseconds) for I/O requests to be served. Higher than 10-15ms indicates disk latency.
- **`iotop`**: An interactive tool (like `top`) showing which specific processes are performing disk reads/writes in real time.

---
### 4. Diagnosing Network and Sockets
Network congestion, closed ports, or excessive open connections can make services appear down.
- **`ss` (Socket Statistics)**: Replaces the older `netstat`. Used to dump socket statistics.
  - `ss -tulpn`: Shows all listening TCP/UDP ports, processes associated, and socket status.
- **`lsof` (List Open Files)**: Everything in Linux is a file. Sockets, hardware, and files are tracked here.
  - `lsof -i :80`: Finds which process is using port 80.
  - `lsof -u apache`: Lists all files opened by the user apache.

---

## Real-World Ticket Scenarios

### Scenario 1: CPU Saturation due to Runaway Process
**Ticket:** "Customer complaints: Website loading is extremely slow. Server alerts indicate Rocky-Web-01 CPU utilization is at 99%."
**L1 Response:** Log in via SSH. Run `top` or `htop`. Press `P` in `top` to sort processes by CPU usage. Identify the runaway process name and PID.
**Escalation Trigger:** The service causing high CPU is the core database daemon (`mysqld` or `postgres`) or a critical java process that L1 cannot safely restart without impact.
**L2 Resolution:** Analyze application queries. If it's a web process, restart the web service container (`systemctl restart httpd` or `nginx`). If it is a rogue user script, kill it using `kill -15 PID`. Check `sar -u` history to identify when the spikes started.

---

### Scenario 2: Memory Leak causing Out-Of-Memory (OOM) Crash
**Ticket:** "The custom reporting API daemon keeps crashing unexpectedly twice a day. No daemon logs are created."
**L1 Response:** Log in to the server. Run `free -h` to check available memory. Run `grep -i oom /var/log/messages` or check kernel logs: `journalctl -k -g oom`.
**Escalation Trigger:** Confirming OOM Killer killed the daemon, but memory optimization requires software stack changes or system parameters tuning.
**L2 Resolution:** Set up monitoring script or alert rules on memory consumption. Run `vmstat 1` to watch memory usage climb. Temporarily configure the daemon with a systemd memory limit (`MemoryMax=2G` in the unit file) so it restarts gracefully before bringing down the server, or increase the swap file size.

---

---

### Enterprise RHEL Service & Network Configurations

#### 1. Custom Systemd Service Creation
Create a custom systemd service configuration file `/etc/systemd/system/myapp.service`:
```ini
[Unit]
Description=My Custom Enterprise Application
After=network.target

[Service]
Type=simple
User=sysadmin
ExecStart=/usr/bin/python3 /opt/myapp/server.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
```

#### 2. Log Rotation & Rsyslog Configuration
Configure log rotation rule in `/etc/logrotate.d/myapp` for automatic log cleaning:
```text
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0660 sysadmin sysadmin
}
```
Define Rsyslog rule in `/etc/rsyslog.d/50-myapp.conf` to redirect application logs to a dedicated file:
```text
if $programname == 'myapp' then /var/log/myapp/syslog.log
& stop
```
Restart Rsyslog service:
```bash
sudo systemctl restart rsyslog
```

#### 3. Network Bonding & Teaming (LACP Link Aggregation)
Create a network team interface config file `/etc/sysconfig/network-scripts/ifcfg-team0` (RHEL standard):
```text
DEVICE=team0
DEVICETYPE=Team
BOOTPROTO=none
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
ONBOOT=yes
TEAM_CONFIG='{"runner": {"name": "lacp"}}'
```
Bind slave physical interfaces (e.g., `eth1`) to the team interface:
```text
# /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
ONBOOT=yes
TEAM_MASTER=team0
DEVICETYPE=TeamPort
```

---
## Step-by-Step Lab
> [!warning] Pre-requisites
> - A running RHEL/Rocky Linux VM.
> - Root or sudo privileges (`sudo -i`).
> - EPEL repository installed for tools like `htop` and `iotop`.
>   ```bash
>   sudo dnf install -y epel-release && sudo dnf install -y htop iotop sysstat
>   ```

### Step 1: Analyze Memory Allocation and Swap Activity
Check the current memory state and verify whether the system is thrashing (swapping).
```bash
# Check memory in human-readable format
free -h

# Monitor system memory and paging every 2 seconds
vmstat 2 5
```
**Expected Output:**
```
              total        used        free      shared  buff/cache   available
Mem:           3.7Gi       1.2Gi       1.1Gi        12Mi       1.4Gi       2.2Gi
Swap:          2.0Gi       120Mi       1.9Gi

procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0 122880 1153432   2048 1468000   0    0    12    45  345  612  4  2 94  0  0
```
*Note: `si` (swap-in) and `so` (swap-out) are `0`, meaning no active paging is dragging performance down.*

---

### Step 2: Identify High Disk Write Consumers
We will simulate a write load and locate the process responsible.
```bash
# Simulate a write load in the background
dd if=/dev/zero of=/tmp/testfile bs=1M count=2000 oflag=direct &

# Immediately run iotop to see disk activity
sudo iotop -o -b -n 3
```
**Expected Output:**
```
Total DISK READ :       0.00 B/s | Total DISK WRITE :     125.40 M/s
Actual DISK READ:       0.00 B/s | Actual DISK WRITE:     125.40 M/s
  PID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
 4567 be/4  root        0.00 B/s  125.40 M/s  0.00 %  94.50 % dd if=/dev/zero of=/tmp/testfile bs=1M count=2000 oflag=direct
```
*Clean up the generated test file:*
```bash
rm -f /tmp/testfile
```

---

### Step 3: Identify Ports and Open Files
Check listening services and identify which process blocks a specific port.
```bash
# List all active listening TCP and UDP ports with process IDs
sudo ss -tulpn

# Find the specific PID holding port 22 (SSH) open
sudo lsof -i :22
```
**Expected Output:**
```
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    1245 root    3u  IPv4  24501      0t0  TCP *:ssh (LISTEN)
sshd    1245 root    4u  IPv6  24509      0t0  TCP *:ssh (LISTEN)
```

---

### Step 4: Examine CPU Load History using `sar`
Ensure `sysstat` collector service is active to collect system statistics.
```bash
# Enable and start sysstat
sudo systemctl enable --now sysstat

# View CPU statistics history for today
sar -u 2 5
```
**Expected Output:**
```
Linux 5.14.0 (Rocky-VM)     06/25/2026      _x86_64_    (2 CPU)

05:46:12 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
05:46:14 PM     all      1.50      0.00      0.50      0.00      0.00     98.00
05:46:16 PM     all      2.00      0.00      1.00      0.10      0.00     96.90
Average:        all      1.75      0.00      0.75      0.05      0.00     97.45
```

---

---
## Cheat Sheet / Quick Reference
| Command | Description | Example |
|---------|-------------|---------|
| `top` | Classic real-time system process monitor. | `top` |
| `htop` | Interactive colorized task manager. | `htop` |
| `free -h` | Show RAM allocation and swap usage. | `free -h` |
| `df -h` | Check disk space usage on all mounted filesystems. | `df -h` |
| `vmstat 1 5` | Report memory and CPU statistics every 1s, 5 times. | `vmstat 1 5` |
| `iostat -xz 1 5` | Detailed disk I/O metrics and wait queues. | `iostat -xz 1 5` |
| `iotop -o` | Shows only processes actively executing Disk I/O. | `iotop -o` |
| `ss -tulpn` | Show listening TCP/UDP sockets and process names. | `ss -tulpn` |
| `lsof -i :80` | List processes holding network port 80. | `lsof -i :80` |
| `sar -r` | View memory utilization history for the current day. | `sar -r` |

---

---
## Troubleshooting
| Problem | Cause | Fix |
|---------|-------|-----|
| System is extremely slow, and `%wa` (I/O Wait) is high. | A database or application is performing heavy unindexed writes or disk is failing. | Run `iotop -o` to find the process doing high writes. Check hardware logs or storage array performance. |
| Server randomly terminates critical applications (e.g., MySQL/Java). | Out Of Memory (OOM) Killer triggered because system ran completely out of RAM. | Check system logs: `dmesg \| grep -i oom` or `grep -i "killed process" /var/log/messages`. Add Swap space or optimize application memory limit. |
| High CPU usage, but `top` shows CPU idle time is high. | The load is caused by zombie or uninterruptible sleep state processes (`D` state). | Find `D` state processes in `top`. They are waiting for disk I/O to complete and cannot be killed. Fix underlying storage. |
| Port already in use error when starting a service (e.g., Apache/Nginx). | Another application daemon is already bound to that port. | Run `ss -tulpn \| grep :80` or `lsof -i :80` to find the conflicting process. Stop or reconfigure it. |

---

---
## Interview Questions
**Q1: What does a high value in the %wa (I/O Wait) column of top mean? How do you investigate it?**
> **A:** High `%wa` (I/O Wait) indicates that the CPU is idle because all runnable tasks are waiting on outstanding disk or network I/O operations to complete. It means you have a disk/network bottleneck, not a CPU speed bottleneck. You investigate it by running `iostat -xz 1 5` to identify which disk device has high queue sizes (`aqu-sz`) and high latency (`await`), and then run `iotop -o` to pinpoint the specific process running the heavy I/O.

**Q2: What is the difference between "Free" and "Available" memory in the free -h command?**
> **A:** **Free** memory is RAM that is completely untouched and contains no data whatsoever. **Available** memory is the amount of memory that can be allocated to applications without causing swapping. Linux uses unused memory for disk buffers and caches (page cache) to optimize performance, which is technically "used" but will be immediately released if an application requests it. Hence, "Available" is the true metric of system health.

**Q3: How do you identify which process is listening on Port 80 and what files it has open?**
> **A:** You can use `ss -tulpn | grep :80` or `lsof -i :80` to find the process name and PID listening on port 80. Once you have the PID (e.g., `4567`), you can run `lsof -p 4567` to list all files, network sockets, and libraries that this specific process currently has open.

**Q4: What is the Linux OOM Killer, and how does it select which process to terminate?**
> **A:** The Out Of Memory (OOM) Killer is a kernel mechanism that activates when the system runs completely out of memory and swap. To prevent a kernel panic and keep the system alive, it scores processes using an `oom_score` (calculated based on memory consumption, process priority, and runtime). The process with the highest score (usually a memory-hogging database or app server) is terminated.

---

---
## Seedha Simple Mein
*Seedha simple mein: Server slow hone par check karna padta hai ki problem CPU, RAM, Disk, ya Network mein se kiski wajah se hai. top aur free se resource usage dikhti hai, jabki iostat aur ss deep-level disk aur network status batate hain.*

---
## Related Notes
- [[L-07 Process Management]]
- [[L-12 Network Configuration in Linux]]
- [[L-17 Linux Log Management]]
