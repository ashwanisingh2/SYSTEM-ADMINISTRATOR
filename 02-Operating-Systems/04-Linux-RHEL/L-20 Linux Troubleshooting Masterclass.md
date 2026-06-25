---
tags: [linux, rhel, troubleshooting, sysadmin, diagnostics]
aliases: [linux-troubleshooting]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

# L-20: Linux Troubleshooting Masterclass

**Verification:**
- [ ] Diagnose boot-level failure alerts and read kernel ring buffer logs
- [ ] Recover deleted filesystem capacity occupied by unlinked running processes
- [ ] Investigate application runtime errors using system call tracing (`strace`)
- [ ] Audit active listening sockets and resolve service port binding conflicts
- [ ] Search logs using `journalctl` with specific boot, time, and service filters

> [!abstract] Overview
> This masterclass note aggregates critical Linux system diagnostics. It provides a structured methodology for identifying, isolating, and resolving runtime issues across CPU, memory, disk, network, and application layers, and features real-world troubleshooting scenarios.

---
## Concept Overview
- **What it is** — Linux troubleshooting is the systematic process of diagnosing, isolating, and resolving system malfunctions, resource bottlenecks, boot issues, and service failures.
- **Why it matters for a support engineer** — Production servers fail. When a business-critical database or web portal goes offline, a sysadmin cannot rely on guesswork. You must have a methodical approach using CLI diagnostic tools to pinpoint the exact failure point under pressure.
- **Where you encounter this in real job** — Web service returning 502 Gateway Error, MySQL refusing to start, user filesystems turning read-only, ssh connections hanging, or high load alerts triggering in the middle of the night.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - **L1**: Collect error messages, run basic system checks (`df`, `free`, `top`, `ping`), view logs using `tail`, and escalate with documented steps.
  - **L2**: Troubleshoot service startup failures (`systemctl status`), identify locked file descriptors, resolve network port conflicts, configure basic kernel logs, and extend full partitions.
  - **L3**: Recover systems from grub/boot failures, run system call tracing (`strace`/`ltrace`), debug kernel panic core dumps, resolve advanced storage corruption, and perform packet analysis (`tcpdump`).

*Seedha simple mein: Linux troubleshooting ka matlab hai server par aane wali problems ko step-by-step logic aur commands ke zariye pehchanna aur theek karna. Command line tools ka use karke hum pata karte hain ki problem storage, permissions, networking, ya code execution mein hai.*

---
## Real-World Analogy
Think of a Linux server as a **high-security hospital**:
- If a patient is sick, you don't perform random surgery immediately. First, you check basic vitals (Temperature, Pulse) -> this is like checking `df -h` and `free -m`.
- If they are coughing, you check the patient charts -> this is reviewing `/var/log/messages` or `journalctl`.
- If you need a deep look inside the organs, you run an X-Ray or MRI -> this is like running `strace` or `tcpdump` to watch the system calls and packets in real time.

---
## Technical Deep Dive

### 1. The SysAdmin Diagnostic Pipeline
To troubleshoot efficiently, always isolate the problem domain by checking layers in order:
```
+-----------------------------------------------------------+
| 1. Physical/Virtual Layer (Is VM running? NIC connected?)  |
+-----------------------------------------------------------+
                             |
+-----------------------------------------------------------+
| 2. Storage & Filesystem (Is disk full? Read-only? inodes?)  |
+-----------------------------------------------------------+
                             |
+-----------------------------------------------------------+
| 3. Permission & Ownership (SELinux blocking? chmod/chown?) |
+-----------------------------------------------------------+
                             |
+-----------------------------------------------------------+
| 4. Network & Routing (DNS working? Firewall blocking port?)|
+-----------------------------------------------------------+
                             |
+-----------------------------------------------------------+
| 5. Process & Memory (OOM Killer hit? Port conflict?)      |
+-----------------------------------------------------------+
```

### 2. Core Diagnostics Toolbox
- **Logging Subsystems**:
  - `journalctl`: Systemd's logging tool. Reads binary systemd journal logs.
  - `dmesg`: Displays messages from the kernel ring buffer (vital for hardware/driver failures).
  - `/var/log/messages` (or `/var/log/syslog` on Debian/Ubuntu): Central repository for general operating system events.
- **Process Call Tracing**:
  - `strace`: Traces system calls and signals received by a process. It intercepts calls to the Linux kernel (like opening files or binding network ports) to find where a process gets stuck.

---
## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - A Rocky Linux / RHEL system.
> - Root or sudo access.
> - `sysstat` and `strace` utilities installed:
>   ```bash
>   sudo dnf install -y sysstat strace
>   ```

### Step 1: Troubleshooting Space Reclamation (Deleted but Open Files)
*Scenario:* Your `df -h` shows disk space is 100% full. You run `du -sh` on all directories, but the numbers don't add up (it shows only 10GB used on a 100GB disk).
1. Generate a large log file:
```bash
dd if=/dev/zero of=/var/log/ghost.log bs=1M count=1000
```
2. Start a process that keeps this file open:
```bash
tail -f /var/log/ghost.log > /dev/null &
```
3. Delete the file using standard `rm`:
```bash
rm -f /var/log/ghost.log
```
4. Verify disk space using `df -h`. You will notice the space is **not** freed up because the background `tail` process still holds an active file descriptor.
5. Identify the "deleted but open" file and the owning Process ID (PID):
```bash
lsof +L1
# Or: lsof | grep deleted
```
**Expected Output:**
```text
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NLINK NODE NAME
tail    12345 root    3r   REG  253,0 104857600     0 9942 /var/log/ghost.log (deleted)
```
6. Safely free up the disk space by killing the process or truncating the file descriptor:
```bash
kill -9 12345
```
7. Verify with `df -h` that the disk space has returned.

### Step 2: Debugging Application Failures using `strace`
*Scenario:* An application crashes during execution, returning a generic "Failed to initialize" error.
1. Run a trace on a basic command (e.g., trying to read a non-existent file) to watch system calls:
```bash
strace cat /etc/shadow_backup
```
**Expected Output:**
```text
execve("/usr/bin/cat", ["cat", "/etc/shadow_backup"], 0x7ffd...) = 0
...
openat(AT_FDCWD, "/etc/shadow_backup", O_RDONLY) = -1 ENOENT (No such file or directory)
write(2, "cat: /etc/shadow_backup: No such"..., 40) = 40
```
*Note:* The output shows that `openat` returned `-1 ENOENT` (No such file or directory), which explains the exact system call failure.

### Step 3: Troubleshooting Service Port Conflicts
*Scenario:* You try to start NGINX but it fails to start.
1. Force a conflict by binding port 80 to a python web server:
```bash
sudo python3 -m http.server 80 &
```
2. Attempt to start NGINX service:
```bash
sudo systemctl start nginx
```
**Expected Output:** `Job for nginx.service failed...`
3. Inspect system logs to find the exact reason:
```bash
sudo journalctl -u nginx -n 20 --no-pager
```
**Expected Output:** `[emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)`
4. Identify which PID is blocking the port:
```bash
sudo ss -tulpn | grep :80
```
**Expected Output:**
```text
tcp   LISTEN 0      5          0.0.0.0:80        0.0.0.0:*    users:(("python3",pid=15432,fd=3))
```
5. Terminate the conflicting process:
```bash
sudo kill -9 15432
```
6. Verify NGINX now starts successfully:
```bash
sudo systemctl start nginx && sudo systemctl status nginx
```

---
## Cheat Sheet

| Command / Setting | Purpose | Example |
|---|---|---|
| `journalctl -xe` | Opens systemd journal logs at the end, displaying system errors | `journalctl -xe --no-pager` |
| `journalctl -u nginx --since "1 hour ago"` | Filters logs for a specific service within a timeframe | `journalctl -u nginx --since "2 hours ago"` |
| `dmesg -T \| tail -n 50` | Prints kernel log messages with human-readable timestamps | `dmesg -T \| grep -i "oom"` |
| `lsof -i :22` | Lists all active processes holding port 22 open | `lsof -i :22` |
| `ss -tulpn` | Displays all listening TCP/UDP sockets with process PIDs | `ss -tulpn` |
| `strace -p <PID>` | Attaches to a running process to trace its active system calls | `strace -p 4321` |
| `du -sh /* 2>/dev/null` | Summarizes disk usage per folder in root, hiding error alerts | `du -sh /var/*` |
| `fuser -v /var/log/nginx` | Lists users/processes accessing a specific file or directory | `fuser -k -9 /var/log/nginx` |

---
## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Server disk shows 100% full, but `du -sh` shows minimal file usage. | Deleted files are still held open in memory by active processes. | Run `lsof \| grep deleted`. Identify the PID and restart the service or run `kill -9 <PID>` to release the lock. |
| SSH login is very slow (takes 10-30 seconds to show password prompt). | SSH daemon is trying to perform a reverse DNS lookup of the client IP, which fails or times out. | Edit `/etc/ssh/sshd_config`. Set `UseDNS no`. Save and run `sudo systemctl restart sshd`. |
| "Read-only filesystem" error when trying to write files. | Filesystem has encountered block level corruption, prompting the kernel to lock it for protection. | Run `fsck -y /dev/sdX` (where sdX is the corrupt partition). Reboot the host. *Note: Never run fsck on a mounted filesystem.* |
| Service fails with "Permission Denied" even though files have permissions `777`. | SELinux policies are blocking the service from accessing the target directory path. | Run `getenforce` to verify. Fix directory context: `semanage fcontext -a -t httpd_sys_content_t "/custom(/.*)?"` and `restorecon -Rv /custom`. |
| High Load Average but CPU utilization shows 5% (Idle is 95%). | CPU cores are idle but processes are blocked waiting for slow storage or network disks (High I/O Wait). | Run `vmstat 1 5` and inspect the `wa` column. Run `iostat -xz 1 5` to locate the saturated disk device. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** How do you check if a specific port (like port 443) is open and listening on a Linux server?
> **A:** I can use the socket statistics command `ss` or the older `netstat` tool. The command `sudo ss -tulpn | grep :443` will display listening sockets (`-l`), filtering for TCP (`-t`) and UDP (`-u`), while showing numerical port values (`-n`) and the associated process name/PID (`-p`). If nothing is returned, the port is closed or the service is stopped.

> [!question] L2 Question
> **Q:** If you delete a large database file but notice that the free disk space does not increase, what is the cause, and how do you resolve it?
> **A:** This occurs because the database process still holds an active file descriptor lock on that file. Although the directory link is removed, the kernel will not release the file's disk sectors until all file descriptors referencing it are closed. To resolve this without stopping the system, I would:
> 1. Run `lsof | grep deleted` to find the process ID (PID) holding the file open.
> 2. Safely restart the database service (or kill the specific PID if safe).
> 3. Alternatively, I can truncate the file descriptor by writing empty content to it under `/proc/<PID>/fd/<FD_NUM>`.

> [!question] L3/Scenario Question
> **Q:** An Apache/httpd web service is crashing on startup with a generic error code. The standard logs do not explain the failure. How would you debug this using system call tracing?
> **A:** 
> - **Situation:** Apache fails to start with zero diagnostic messages in standard log files.
> - **Task:** Track the low-level system execution calls of the HTTPD binary on startup to locate the failure.
> - **Action:** I will bypass systemctl and run the daemon binary directly inside `strace`: `strace -f -o /tmp/httpd_trace.txt /usr/sbin/httpd -X`. The `-f` flag traces spawned threads, and `-o` writes the trace to a file. I will search the text file `/tmp/httpd_trace.txt` from the bottom up, looking for failed system calls returning errors like `EACCES` (Permission Denied) or `ENOENT` (No such file).
> - **Result:** This trace might reveal that Apache was crashing because it could not access a specific SSL key file due to a permissions mismatch, allowing me to correct the path ownership and successfully start the service.

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Explains unit configurations and debugging with systemctl.
- [[02-Operating-Systems/04-Linux-RHEL/L-14 Linux Security Hardening|L-14 Linux Security Hardening]] — SELinux troubleshooting and audits.
- [[02-Operating-Systems/04-Linux-RHEL/L-18 Linux Performance Monitoring|L-18 Linux Performance Monitoring]] — Identifying CPU, RAM, and Disk bottlenecks.

---
*Tags: #linux #rhel #troubleshooting #sysadmin #diagnostics #cert-rhcsa*
