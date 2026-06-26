---
tags: [desktop-support, linux, rhel, L1]
aliases: [l-07-process-management, l-07]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# L-07: Process Management

> [!abstract] Overview
> This note covers Linux process lifecycle administration, including process states, priority tuning (nice/renice), foreground/background job controls, signal routing, `/proc` memory diagnostics, and zombie resolution.

---

---
## Concept Overview
Think of process management in Linux like managing tasks in a busy kitchen:
- A **Process** is an active recipe being cooked (it has its own kitchen counter space/memory and chefs).
- A **Thread** is multiple chefs working together on the *same* recipe sharing the *same* counter space (like one chopping onions, one boiling water).
- **Process States** dictate what's happening: a chef is actively cooking (Running), waiting for water to boil (Sleeping), ordered to pause (Stopped), or finished their task but waiting for the head chef to check their plate (Zombie).
- **Nice Values** are task priorities: a high-priority emergency order is "less nice" (Nice -20) and cuts the queue, while a low-priority task is "very nice" (Nice +19) and steps aside for other chefs.
- **Signals** are orders shouted by the head chef: "Pause what you are doing" (SIGSTOP), "Wrap up nicely" (SIGTERM), or "Get out of the kitchen immediately" (SIGKILL).


---

---
## Technical Deep Dive
### 1. Process vs. Thread
- **Process:** An executing instance of a program. It is assigned a unique **PID (Process ID)**, its own isolated virtual memory space, file descriptors, and security context. Processes do not share memory directly.
- **Thread:** A lightweight unit of execution inside a process. Multiple threads share the parent process's memory space and resources, allowing fast concurrency but riskier stability: if one thread crashes, the entire parent process dies.

### 2. Process States
Linux processes display their current state in the `S` column of `ps` or `top`:
- **R (Running / Runnable):** The process is either currently executing on a CPU core, or waiting in the run queue to get CPU time.
- **S (Interruptible Sleep):** The process is waiting for an event or resource (like user input or disk read). It wakes up if it receives a signal.
- **D (Uninterruptible Sleep):** The process is waiting for critical hardware I/O (like a direct disk write). It cannot be interrupted or killed by signals until the I/O completes.
- **T (Stopped):** The process has been suspended by a user or signal (e.g., `Ctrl+Z` or SIGSTOP).
- **Z (Zombie):** The process has completed execution (`exit()`), but its parent process has not yet read its exit status code using `wait()`. It consumes no CPU or RAM, but occupies a slot in the system PID table.

### 3. Monitoring Utilities: ps, top, htop

#### Reading `ps aux` Columns
- `a` — Show processes for all users.
- `u` — Display user-oriented format (owner, memory).
- `x` — Include processes without controlling terminals (daemons).
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 173512  9820 ?        Ss   Jun25   0:02 /usr/lib/systemd/systemd
```
- **RSS (Resident Set Size):** The actual physical RAM (in KB) the process is consuming.
- **VSZ (Virtual Size):** The total virtual memory allocated to the process (includes swapped memory and shared libraries).

#### top Command Shortcuts
Interactive real-time process manager:
- `M` — Sort processes by Memory usage.
- `P` — Sort processes by CPU usage.
- `k` — Kill a process (prompts for PID and signal).
- `r` — Renice a process (change priority).
- `q` — Quit top.
- *htop:* A modern, color-coded terminal process viewer. Supports mouse clicks, search filters, and visual CPU/RAM charts.

### 4. Process Priority: Nice and Renice
Linux schedules processes based on priority. The **Nice value** ranges from **`-20`** (highest priority, least nice to others) to **`+19`** (lowest priority, very nice).
- **`nice` (Start with custom priority):**
  - `nice -n -10 tar -czf backup.tar.gz /data` (Runs backup with high priority. Only root can assign negative nice values).
- **`renice` (Modify existing priority):**
  - `renice -n 5 -p 1234` (Increases nice value of PID 1234 to 5, lowering its priority).

### 5. Job Control: Foreground vs. Background
- **Background Execution:** Append `&` to run a command in the background immediately:
  - `dd if=/dev/urandom of=/dev/null &`
- **Suspension (`Ctrl + Z`):** Suspends the active foreground process and sends it to the background in a **Stopped** state.
- **Job Commands:**
  - `jobs` — Lists active background jobs and their job numbers (e.g., `[1]`).
  - `bg %1` — Resumes background job 1 in the background (**Running** state).
  - `fg %1` — Pulls background job 1 to the foreground.

### 6. Process Signals
Signals are asynchronous messages sent to processes to trigger actions.

| Signal Name | Number | Action / Meaning |
|---|---|---|
| **SIGHUP** | 1 | Hangup. Used to instruct daemon processes to reload configuration files without restarting. |
| **SIGINT** | 2 | Interrupt. Sent by pressing **`Ctrl+C`**. Gracefully interrupts foreground processes. |
| **SIGKILL**| 9 | Kill. Instantly terminates the process at the kernel level. The process cannot catch or ignore this signal. |
| **SIGTERM**| 15 | Terminate. The default signal sent by `kill`. Instructs the process to clean up resources and exit gracefully. |
| **SIGSTOP**| 19 | Stop. Suspends the process (puts it in Stopped state). Cannot be ignored. |

### 7. Resolving Zombie Processes
A zombie process has exited but remains in the process table because its parent is dead or hung and failed to call `wait()`.
- **How to resolve:** You cannot kill a zombie process directly using `kill -9` because it is already dead. Instead, you must:
  1. Send **SIGHUP** or **SIGTERM** to the **Parent Process** (PPID) to force it to clean up its children.
  2. If the parent does not respond, kill the parent process. The zombie will be adopted by `systemd` (PID 1), which automatically cleans up orphan zombies.

---

## Common Mistakes
> [!warning] Avoid These
> **Defaulting to kill -9 for all process terminations:** Running `kill -9` on databases or active applications. This bypasses the application cleanup routines, leaving temporary locks active, half-written database logs corrupted, and files unclosed.
> **Correct approach:** Always start with `kill -15` (SIGTERM) to allow the application to save states and close files. Only use `kill -9` as a last resort if the process is completely hung.

---

## Pro Tips
> [!tip] Field Experience
> Use the `/proc` filesystem to check running processes manually when troubleshooting. For example, if a process name is hidden or modified, you can inspect `/proc/[PID]/exe` (which links to the actual binary path) or read `/proc/[PID]/cmdline` to see the exact start arguments used.

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
> [!info] Lab Setup Needed
> A Linux terminal console session.

### Step 1: Execute Job Control
1. Start a dummy background process:
   ```bash
   sleep 1000 &
   ```
2. Start a second process in the foreground:
   ```bash
   sleep 2000
   ```
3. Suspend the active `sleep 2000` process by pressing **`Ctrl + Z`**.
4. View the jobs status list:
   ```bash
   jobs
   ```
   **Verify:** Note that `sleep 1000` is running in the background, and `sleep 2000` is stopped.
5. Resume the stopped job in the background:
   ```bash
   bg %2
   ```
6. Check `jobs` again; both should now read Running.

### Step 2: Priority Tuning (Nice / Renice)
1. Find the PID of your `sleep 1000` job:
   ```bash
   ps aux | grep "sleep 1000" | grep -v grep
   ```
   Write down the PID (e.g., `4523`).
2. Lower its priority by increasing its nice value to `10`:
   ```bash
   renice -n 10 -p 4523
   ```
3. Verify the nice value change:
   ```bash
   ps -o pid,ni,cmd -p 4523
   ```
   Confirm that the `NI` column reads `10`.

### Step 3: Signal Execution
1. Send a graceful terminate signal to the first job:
   ```bash
   kill 4523
   ```
2. Verify if it terminated. If it remains, force kill:
   ```bash
   kill -9 4523
   ```
3. Kill the second job by name:
   ```bash
   killall sleep
   ```

---

---
## Cheat Sheet / Quick Reference
```bash
# Display process details of PID 1234, including parent PID (PPID) and nice value
ps -o pid,ppid,ni,stat,cmd -p 1234

# Trace open file descriptors of a process (requires lsof installed)
lsof -p 1234

# View real-time resource usage of systemd services (requires systemd-cgtop)
systemd-cgtop
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Process | Active program instance assigned a unique PID and isolated virtual memory space. |
| 2 | nice | Priority tuner; values range from -20 (highest priority) to +19 (lowest priority). |
| 3 | SIGTERM (15) | Default terminate signal instructing processes to clean up and exit gracefully. |
| 4 | SIGKILL (9) | Force kill signal executed at kernel level; cannot be caught or ignored. |
| 5 | D State | Uninterruptible sleep; process is waiting for hardware I/O and cannot be killed. |

---

---
## Troubleshooting
**Scenario 1:**
- **Problem:** An administrator runs `kill -9 1234` on a process consuming high resources, but the process remains active, ignoring the signal. Running `ps` shows its state as `D`.
- **Root Cause:** The process is in Uninterruptible Sleep (`D` state) waiting for a physical hardware response (typically a dead NFS mount or failing block storage controller). In this state, the kernel blocks all signal deliveries until the I/O system call returns.
- **Fix:**
  1. Identify the blocked mount point or storage device.
  2. If it is a dead NFS mount, force unmount the connection:
     ```bash
     umount -f -l /mnt/dead_share
     ```
  3. If it is a hardware disk block issue, repair the storage controller or reboot the server.

**Scenario 2:**
- **Problem:** The server CPU load average is extremely high, but running `top` shows CPU usage is low ($5\%-10\%$).
- **Root Cause:** High load average with low active CPU usage indicates that many processes are queued in the **Run queue** or **Uninterruptible Sleep queue** waiting for disk I/O (high I/O Wait).
- **Fix:**
  1. Open `top` or `htop`.
  2. Check the **`wa`** (I/O wait) metric at the top CPU line.
  3. If it is high (above $20\%$), run `iotop` to identify which processes are saturating the disk bandwidth.
  4. Optimize database write queries or upgrade slow HDDs to SSD arrays.

---

---
## Interview Questions
**Q1: What is a zombie process, and how do you resolve it if you cannot kill it directly?**
A: A zombie process (marked `Z` in `ps`) is a process that has completed execution but still occupies a slot in the process table. This occurs because the parent process has failed to read the child's exit status. You cannot kill a zombie directly with `kill -9` because it is already dead. To resolve it, I would first identify the parent process ID (PPID) using `ps -o ppid -p [Zombie_PID]`. I would send a `SIGHUP` (1) or `SIGTERM` (15) to the parent to instruct it to run the cleanup. If the parent remains hung, killing the parent process forces `systemd` (PID 1) to adopt the zombie and clean it up automatically.

**Q2: A background script is running under Job ID [1] with PID 8452. Explain the commands you would run to pull it to the foreground, suspend it, and terminate it.**
A: 
1. To pull the job to the foreground, I run: `fg %1` (or `fg 8452`).
2. Once in the foreground, I suspend (stop) it by pressing **`Ctrl + Z`**.
3. To terminate the process, I can send a SIGTERM signal using its PID: `kill 8452`. If it does not respond, I force kill: `kill -9 8452`.

**Q3: Explain the difference between VSZ (Virtual Memory Size) and RSS (Resident Set Size) in `ps aux` output.**
A: **VSZ** is the total amount of virtual memory allocated to a process, including memory swapped out to disk, memory mapped to shared libraries, and memory allocated but not yet used. **RSS** is the actual physical RAM (in Kilobytes) that the process is currently using on the system memory chips. RSS is the accurate metric to evaluate if a process is consuming too much system memory.

---

---
## Seedha Simple Mein
*Seedha simple mein: Linux mein har running program ek process (PID) hota hai. Process monitoring ke liye hum `ps`, `top` aur `htop` use karte hain. `kill` command ke zariye signals (1, 9, 15) bhej kar processes ko terminate kiya jata hai.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Basic CLI execution commands.
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — Virtual directory mappings like `/proc`.
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Managing background system processes.
