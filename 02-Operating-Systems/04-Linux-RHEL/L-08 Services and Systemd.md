---
tags: [sysadmin, linux, systemd, services]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# L-08: Services and Systemd

> [!abstract] Overview
> This note covers Linux service management using the `systemd` initialization framework. It details systemctl controls, unit file profiles, target runlevels, journalctl logging, and custom service creation.

---
## Concept
Think of `systemd` as the central automation engine and dispatcher of the Linux operating system. 
In older systems (SysVinit), starting services was like a single line of soldiers marching one by one: service A had to finish loading before service B could start, which was slow. 

`systemd` is a modern coordinator. It starts services in parallel, utilizing sockets (pre-allocating connections so services can start feeding data immediately) and timers. 

- **systemctl** is your console dashboard: you start, stop, or permanently schedule services to start on boot.
- **journalctl** is the centralized flight data recorder, cataloging all errors and outputs from all services into one indexed, searchable database.

*Seedha simple mein: `systemd` modern Linux distributions ka service manager hai jo boot time and services execution ko handle karta hai. `systemctl` command ke zariye hum services manage karte hain aur `journalctl` se logs check karte hain.*

---
## Technical Deep Dive

### 1. Why systemd Replaced SysVinit
- **Parallelization:** Starts services concurrently on boot, speeding up startup times.
- **Dependency Management:** Automatically tracks and resolves requirements (e.g., won't start Web Server until Network Link is UP).
- **Control Groups (cgroups):** Group process trees together, allowing resource throttling and clean, complete terminations (no orphan processes left behind).

### 2. Unit Types in systemd
Every resource managed by systemd is defined in configuration files called **Units**:
- **`.service`:** Manages active system processes/daemons (e.g., `httpd.service`).
- **`.socket`:** Listens on network ports/files. systemd starts the service only when traffic hits the socket.
- **`.target`:** Groups other units together to define system states/runlevels.
- **`.timer`:** Schedules task executions, replacing cron (e.g., executing backup scripts every night).

### 3. systemctl Command Reference

```bash
systemctl start nginx.service     # Start service immediately (non-persistent)
systemctl stop nginx.service      # Stop service immediately
systemctl restart nginx.service   # Stop and restart service
systemctl reload nginx.service    # Reload configuration without dropping connections
systemctl status nginx.service    # View running state, PID, and recent log outputs

systemctl enable nginx.service    # Configure service to start automatically on boot
systemctl disable nginx.service   # Stop service from starting on boot
systemctl is-enabled nginx.service # Verify boot autostart configuration state

systemctl mask nginx.service      # Permanently lock service (symlinks it to /dev/null), preventing manual/auto starts
systemctl unmask nginx.service    # Unlock service for normal operations

systemctl list-units --type=service # List all currently active services in memory
systemctl list-unit-files         # List all installed service files and their boot states
```

### 4. Targets (Runlevels)
systemd uses Targets to configure active operational states:
- **`poweroff.target` (Runlevel 0):** Shuts down system.
- **`rescue.target` (Runlevel 1):** Single-user recovery shell. No networking, root only.
- **`multi-user.target` (Runlevel 3):** Standard multi-user text console with networking. Common on production servers.
- **`graphical.target` (Runlevel 5):** Graphical desktop environment (GNOME/KDE) with networking.
- **`reboot.target` (Runlevel 6):** Reboots system.

### 5. Service Unit File Structure
Unit files are located in `/lib/systemd/system/` (system default) and `/etc/systemd/system/` (admin overrides).
```ini
[Unit]
Description=My Custom Backup Daemon
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/backup_run.sh
Restart=on-failure
User=sysadmin

[Install]
WantedBy=multi-user.target
```
- **`After`:** Specifies dependencies (start after network is online).
- **`ExecStart`:** Absolute path to executable script or program.
- **`WantedBy`:** Defines boot installation target (linking to `multi-user.target` runs it on CLI boot).

### 6. Centralized Logging: journalctl
systemd routes all stdout/stderr from services to the systemd-journald database in binary format.
- `journalctl -u nginx` — View logs only for nginx service.
- `journalctl -f` — Output log changes in real time (follow).
- `journalctl -p err` — Filter by priority: show only Error states or worse.
- `journalctl --since "1 hour ago"` — Time window filter.
- `journalctl -u nginx --since "2026-06-25 08:00:00" --until "2026-06-25 10:00:00"` — Granular search.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A running Linux VM with root access.

### Step 1: Create the Executable Script
1. Log in as root. Create a simple loop script:
   ```bash
   cat << 'EOF' > /usr/local/bin/logger_service.sh
   #!/bin/bash
   while true; do
       echo "Logger Service: Active at $(date)"
       sleep 10
   done
   EOF
   ```
2. Grant execute permissions:
   ```bash
   chmod +x /usr/local/bin/logger_service.sh
   ```

### Step 2: Create the systemd Service File
1. Create a unit file in the admin override directory `/etc/systemd/system/`:
   ```bash
   vim /etc/systemd/system/logger.service
   ```
2. Add configuration content:
   ```ini
   [Unit]
   Description=Custom System Logger Service
   After=network.target

   [Service]
   Type=simple
   ExecStart=/usr/local/bin/logger_service.sh
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```
3. Save and close (`:wq`).

### Step 3: Initialize and Verify Service
1. **CRITICAL:** Reload systemd configuration manager to detect the new file:
   ```bash
   systemctl daemon-reload
   ```
2. Start the service:
   ```bash
   systemctl start logger.service
   ```
3. Enable the service to run on boot:
   ```bash
   systemctl enable logger.service
   ```
4. Verify running status:
   ```bash
   systemctl status logger.service
   ```
5. Check service outputs in log database:
   ```bash
   journalctl -u logger -n 5
   ```
   **Verify:** Confirm that timestamp entries are printed every 10 seconds.

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-04 Text Editors and File Viewing|L-04 Text Editors and File Viewing]] — Writing configurations and reading logs.
- [[02-Operating-Systems/04-Linux-RHEL/L-07 Process Management|L-07 Process Management]] — Monitoring process IDs generated by systemd.
- [[02-Operating-Systems/04-Linux-RHEL/L-13 Bash Scripting for Sysadmins|L-13 Bash Scripting for Sysadmins]] — Writing scripts used in ExecStart.

