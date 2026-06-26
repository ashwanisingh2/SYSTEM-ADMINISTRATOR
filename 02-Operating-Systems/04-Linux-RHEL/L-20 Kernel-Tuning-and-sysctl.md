---
tags: [desktop-support, linux-rhel, performance, sysctl, L2]
aliases: [kernel-tuning, sysctl-guide]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

# L-20: Kernel Tuning and sysctl

> [!note] Overview
> This note covers performance optimization and resource allocation through Linux kernel tuning. It details the `/proc/sys` virtual filesystem, the `sysctl` configuration utility, parameters for memory/storage/networking optimization, and limits configuration.

---
## Concept Overview
- **What it is** — Kernel tuning is the modification of operating system configuration values (such as memory management thresholds, file handle limits, and TCP stack backlogs) directly within the running Linux kernel. The `sysctl` command serves as the primary administrative tool to manage these parameters.
- **Why it matters** — Standard Linux kernel defaults are balanced for generic desktops or lightweight servers. When hosting high-throughput databases, heavy web apps, or routers, default thresholds will limit capacity, causing connection bottlenecks, slow performance, or system crashes.
- **Real job encounter** — Resolving "Too many open files" errors on web nodes, optimizing swap space usage to prevent disk thrashing, enabling IP forwarding on network gateways, and expanding socket limits.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: View current parameters using `sysctl -a`, check active virtual memory swappiness, and query open file descriptors for troubleshooting logs.
  - **Escalation Trigger**: Escalate if application logs show resource saturation limits or socket drop messages, requiring kernel-level alterations.
  - **L2 Resolution**: Adjust runtime parameters using `sysctl -w`, write persistent settings in `/etc/sysctl.d/`, and increase client resource limits in `/etc/security/limits.conf`.
  - **L3 Resolution**: Configure memory overcommit behaviors, tune TCP receive/transmit buffer windows, adjust dirty memory writeback thresholds, and perform advanced system benchmarking.

---
## Technical Deep Dive

### 1. The virtual `/proc/sys` Filesystem
The Linux kernel exposes its internal configuration parameters through the `/proc/sys/` directory. This is not a physical storage disk; it is a virtual interface residing in memory.
- Writing a value to a file in `/proc/sys/` immediately changes the kernel's runtime behavior.
- The path corresponds directly to the parameter name. E.g., parameter `net.ipv4.ip_forward` maps to file `/proc/sys/net/ipv4/ip_forward`.

### 2. The `sysctl` Configuration Engine
The `sysctl` utility translates dot-notation settings into `/proc/sys/` filesystem paths:
- Dots are converted to slashes.
- E.g., `sysctl vm.swappiness` reads `/proc/sys/vm/swappiness`.
- `sysctl -w` writes settings to running memory instantly. However, these changes are lost upon reboot unless saved in persistent configuration directories.

```
       [ sysctl vm.swappiness = 10 ]
                    |
           Translates Dot-Notation
                    |
                    v
       [ /proc/sys/vm/swappiness ] (Virtual File)
                    |
         Updates Kernel Memory Cache
                    |
                    v
    [ Immediate Behavior Modification ]
```

### 3. Key Kernel Parameters for Performance Tuning

#### Virtual Memory Tuning
- **`vm.swappiness`**: Controls how aggressively the kernel moves active processes to swap space. Value ranges from `0` to `100` (default is `60`).
  - **Setting to `10`**: Minimizes swap use, keeping execution in high-speed RAM as long as possible (ideal for database servers).
- **`vm.overcommit_memory`**: Determines whether the kernel permits applications to request more virtual memory than is physically available.
  - `0`: Heuristic check (default). Allocates reasonable overcommits.
  - `1`: Always allow overcommit. Required for Redis and high-load servers.
  - `2`: Restrict overcommit to physical RAM + Swap. Prevents Out-Of-Memory (OOM) crashes.

#### Storage & File Handles
- **`fs.file-max`**: Defines the system-wide maximum number of concurrently open file handles. High-volume proxy engines (Nginx, HAProxy) require expanding this limit significantly.

#### Network Stack Tuning
- **`net.ipv4.ip_forward`**: Enables IP packet routing between separate network interfaces. Essential for firewalls, routers, VPN hosts, and Docker hypervisors.
- **`net.core.somaxconn`**: Sets the maximum connection backlog size for listening sockets. Prevents connection drops during rapid traffic spikes.

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
> - A Rocky Linux / RHEL 9 VM.
> - Root or sudo access.
> - Execution permissions inside `/etc/sysctl.d/`.

### Step 1: Query and Analyze Current Kernel Settings
1. Check current virtual memory swappiness level:
```bash
sysctl vm.swappiness
```
**Expected Output:** `vm.swappiness = 60`

2. Verify that this matches the virtual filesystem node:
```bash
cat /proc/sys/vm/swappiness
```
**Expected Output:** `60`

### Step 2: Apply Temporary Runtime Modifications
1. Reduce swappiness dynamically:
```bash
sudo sysctl -w vm.swappiness=10
```
**Expected Output:** `vm.swappiness = 10`

2. Confirm the update is active:
```bash
cat /proc/sys/vm/swappiness
```
**Expected Output:** `10` (The change takes effect instantly without server reboot).

### Step 3: Configure Permanent Settings
Temporary writes do not survive reboots. We must create a persistent configuration file.

1. Create a custom tuning file under `/etc/sysctl.d/`:
```bash
sudo vi /etc/sysctl.d/99-performance-tuning.conf
```
2. Insert the following performance rules:
```text
# Custom Server Performance Tuning
vm.swappiness = 10
fs.file-max = 2097152
net.ipv4.ip_forward = 1
net.core.somaxconn = 4096
```
3. Save the file and exit.

### Step 4: Apply Persistent Settings to Kernel Memory
1. Load all configurations from files:
```bash
sudo sysctl --system
```
**Expected Output:** Displays path directories loaded and shows:
`* Applying /etc/sysctl.d/99-performance-tuning.conf ...`
`vm.swappiness = 10`, `fs.file-max = 2097152`, etc.

### Step 5: Configure Application User File Limits
System-wide kernel limits must be matched by user-level limits.

1. Open the system limits file:
```bash
sudo vi /etc/security/limits.conf
```
2. Configure limits for service user `nginx` at the bottom:
```text
# User limits for high-performance Nginx web server
nginx    soft    nofile    65536
nginx    hard    nofile    131072
```
3. Save and close. Users logging in or services starting under the `nginx` group will now inherit these expanded file handle limits.

---
## Cheat Sheet / Quick Reference

| Command / Setting | Purpose | Example |
|---|---|---|
| `sysctl -a` | Lists all active kernel configuration parameters | `sysctl -a` |
| `sysctl <param>` | Queries value of a specific parameter | `sysctl net.ipv4.ip_forward` |
| `sysctl -w <param>=<val>` | Changes parameter value in RAM instantly | `sudo sysctl -w net.ipv4.ip_forward=1` |
| `sysctl -p <file>` | Loads configurations from a specific file path | `sudo sysctl -p /etc/sysctl.conf` |
| `sysctl --system` | Reloads all system configuration directories | `sudo sysctl --system` |
| `ulimit -n` | Queries user-specific maximum open files limit | `ulimit -n` |
| `ulimit -a` | Lists all resource limits applied to the current shell | `ulimit -a` |
| **`/etc/sysctl.conf`** | Legacy primary configuration file | Config File |
| **`/etc/sysctl.d/`** | Modern preferred configuration directory | Settings directory |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Application logs return error: "Fatal error: system out of memory / fork failed." | System hit `vm.max_map_count` limits, or overcommit memory is denied. | Check settings: `sysctl vm.overcommit_memory`. For database/elastic engines, set `vm.max_map_count=262144` and `vm.overcommit_memory=1`. |
| Custom sysctl settings revert back to default values after reboot. | Settings were modified using `sysctl -w` but not written to `/etc/sysctl.conf` or `/etc/sysctl.d/`. | Add configuration entries permanently in `/etc/sysctl.d/99-custom.conf` and load with `sysctl --system`. |
| "Too many open files" errors in Apache logs, but `fs.file-max` is set to 2 million. | System-wide kernel limits are high, but user-level security limits are restricting the Apache service process. | Edit `/etc/security/limits.conf` and set appropriate `soft` and `hard` `nofile` limits for the apache user. |
| Port binding works locally, but network clients cannot access routed subnets. | IP forwarding is disabled, blocking interface packet transit. | Enable routing: `sudo sysctl -w net.ipv4.ip_forward=1` and ensure it is saved permanently. |
| Settings in a custom configuration file under `/etc/sysctl.d/` are not loading. | File name does not end with the required `.conf` extension. | Rename the configuration file to ensure it ends in `.conf` (e.g., `tuning.conf`) and reload. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What does the `vm.swappiness` parameter control, and what is its default value on RHEL systems?
> **A:** `vm.swappiness` controls how aggressively the Linux kernel moves memory pages out of physical RAM and into the Swap space on disk. Its value ranges from `0` to `100`, with a default value of `60` on RHEL. Lowering this value (e.g., to `10`) instructs the kernel to avoid swapping and keep active data in RAM, boosting application speed.

> [!question] L2 Question
> **Q:** Explain the difference between user limits (`limits.conf`) and system-wide kernel parameters (`sysctl.conf`).
> **A:** System-wide kernel parameters (managed via `sysctl.conf`) set the absolute maximum limits for the entire operating system kernel (e.g., `fs.file-max` limits the total open files across all users combined). User limits (configured in `/etc/security/limits.conf` or via `ulimit`) restrict the resources a specific user account or process (like `mysql` or `nginx`) can consume individually. A user limit cannot exceed the global system-wide limit.

> [!question] L3/Scenario Question
> **Q:** You are troubleshooting a high-traffic web server running Nginx. During peak hours, clients experience connections dropping, and log analysis shows "connection socket overflow" events. Which kernel parameters would you adjust to resolve this?
> **A:** 
> - **Situation:** High-traffic web server dropping connections due to socket overflow.
> - **Task:** Increase system connection backlog and buffer limits.
> - **Action:** I will tune key network parameters:
>   1. Increase the maximum backlog queue size: `net.core.somaxconn = 4096`.
>   2. Increase TCP connection queue limits: `net.ipv4.tcp_max_syn_backlog = 4096`.
>   3. Expand TCP buffer sizes to handle larger data windows:
>      `net.ipv4.tcp_rmem = 4096 87380 16777216` (Read) and `net.ipv4.tcp_wmem = 4096 65536 16777216` (Write).
>   4. Write these permanently to `/etc/sysctl.d/99-networking.conf` and apply with `sysctl --system`.
> - **Result:** The kernel accepts larger backlogs, preventing connection drops during traffic spikes.

---
## Seedha Simple Mein
*Seedha simple mein: Kernel Tuning ka matlab hai Linux operating system ke core settings ko change karna taaki performance improve ho sake. `sysctl` command ka use karke hum RAM management (swappiness), networks, aur open file limits ko customize karte hain, jo application crashes hone se bachata hai.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-07 Process Management|L-07 Process Management]] — Resource monitoring and task processes.
- [[02-Operating-Systems/04-Linux-RHEL/L-18 Linux Performance Monitoring|L-18 Linux Performance Monitoring]] — Identifying CPU, RAM, and disk bottlenecks.
- [[02-Operating-Systems/04-Linux-RHEL/L-20 Linux Troubleshooting Masterclass|L-20 Linux Troubleshooting Masterclass]] — Practical process and memory troubleshooting.

---
*Tags: #desktop-support #linux-rhel #performance #sysctl #L2 #cert-rhcsa*
