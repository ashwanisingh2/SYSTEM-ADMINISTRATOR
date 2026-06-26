---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-13-bash-scripting-for-sysadmins, l-13]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

# L-13: Bash Scripting for Sysadmins

> [!abstract] Overview
> This note covers Bash shell scripting fundamentals, including variables, control structures (conditionals, loops), file operators, functions, and arrays. It contains five production-ready administrative scripts with deployment step-guides.

---

---
## Concept Overview
Think of writing a bash script like training an automated kitchen assistant:
- The **Shebang (`#!/bin/bash`)** is the instruction booklet telling the operating system: "Use the BASH interpreter to read this recipe."
- **Variables** are labels on storage containers (e.g., set variable `TEMP_DIR="/tmp"`).
- **Conditionals (`if/then`)** are decision gates: "If the temperature is too hot (CPU > 85), turn on the cooling fan."
- **Loops (`for/while`)** are repetitive labor orders: "Check every server in this list and report if they are awake."
- **Functions** are pre-configured recipes: instead of writing the same 20 lines of code every time you back up a folder, you package them under a single order named `run_backup`.


---

---
## Technical Deep Dive
### 1. Script Structure & Execution
- **Shebang:** The very first line of the script. Specifies the path to the interpreter:
  `#!/bin/bash`
- **Execution permissions:** Scripts must have execute permissions to run directly:
  `chmod +x script.sh`
  Run using `./script.sh` or run inside a shell: `bash script.sh`.

### 2. Variables & User Input
- **Declaration (no spaces around `=`):**
  `BACKUP_DIR="/mnt/backup"`
- **Referencing:** Use `$` symbol:
  `echo "Target directory is $BACKUP_DIR"` or `${BACKUP_DIR}_old`.
- **Interactive Input:** Use `read`:
  `read -p "Enter username: " NEW_USER`

### 3. Arithmetic Operations
Bash only supports integer arithmetic out-of-the-box.
- **Preferred Method:** `$(( expression ))`
  - `TOTAL=$(( 10 + 20 ))`
  - `COUNT=$(( COUNT + 1 ))`

### 4. Conditional Statements & Operators

#### Comparison Operators (Numeric vs. String)
- **Numeric:** `-eq` (equal), `-ne` (not equal), `-gt` (greater than), `-lt` (less than), `-ge` (greater-equal), `-le` (less-equal).
- **String:** `=` (equal), `!=` (not equal), `-z` (string is empty), `-n` (string is not empty).

#### File Test Operators
- `-f /path/file` — True if target exists and is a regular file.
- `-d /path/dir` — True if target is a directory.
- `-e /path` — True if target exists.
- `-r /path` — True if read permission is active.
- `-w /path` — True if write permission is active.
- `-x /path` — True if execute permission is active.

```bash
if [ -f "/etc/nginx/nginx.conf" ]; then
    echo "NGINX config exists."
else
    echo "NGINX config missing."
fi
```

### 5. Loops: For, While, Until
- **For Loop (iterate over list):**
  ```bash
  for SERVER in srv01 srv02 srv03; do
      ping -c 1 $SERVER >/dev/null && echo "$SERVER online"
  done
  ```
- **While Loop (runs as long as condition matches):**
  ```bash
  while [ $COUNT -lt 10 ]; do
      echo "Count: $COUNT"
      COUNT=$((COUNT+1))
  done
  ```

### 6. Functions & Arrays
- **Function Syntax:**
  ```bash
  log_message() {
      echo "[$(date '+%Y-%m-%d %H:%M:%S')] : $1" >> /var/log/custom.log
  }
  # Invoking:
  log_message "Backup started"
  ```
- **Arrays:**
  - Declaration: `SERVERS=("srv01" "srv02" "srv03")`
  - Referencing: `${SERVERS[0]}`
  - Iterate all: `for S in "${SERVERS[@]}"; do ...`

---

## Five Practical Sysadmin Scripts

### Script 1: Backup Script with Date
Compresses a target directory and appends a date stamp to the archive name.
```bash
#!/bin/bash
# Description: Custom Directory Backup Script
# Target: /data/source_files to /backup

SOURCE="/data/source_files"
BACKUP_DIR="/backup"
DATE=$(date +%Y-%m-%d_%H%M%S)
ARCHIVE_NAME="backup_$DATE.tar.gz"

# Create backup directory if missing
if [ ! -d "$BACKUP_DIR" ]; then
    mkdir -p "$BACKUP_DIR"
fi

# Execute compression
if [ -d "$SOURCE" ]; then
    tar -czf "$BACKUP_DIR/$ARCHIVE_NAME" "$SOURCE" 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "SUCCESS: Backup completed. Saved as $BACKUP_DIR/$ARCHIVE_NAME"
    else
        echo "ERROR: Backup compression failed." >&2
        exit 1
    fi
else
    echo "ERROR: Source directory $SOURCE does not exist." >&2
    exit 1
fi
```

---

### Script 2: User Creation Script
Reads a CSV layout or lists, checking if accounts exist before creating them.
```bash
#!/bin/bash
# Description: Automated user provisioning script
# Input: Username as command line argument

if [ -z "$1" ]; then
    echo "Usage: $0 [username]" >&2
    exit 1
fi

NEW_USER="$1"

# Check if user already exists
id "$NEW_USER" &>/dev/null
if [ $? -eq 0 ]; then
    echo "WARNING: User $NEW_USER already exists. Skipping."
    exit 0
fi

# Create user with standard home directory
useradd -m "$NEW_USER"
if [ $? -eq 0 ]; then
    # Generate random temporary password
    TEMP_PWD=$(openssl rand -base64 12)
    echo "$NEW_USER:$TEMP_PWD" | chpasswd
    # Force password change on first login
    chage -d 0 "$NEW_USER"
    
    echo "SUCCESS: User $NEW_USER created."
    echo "Temp Password: $TEMP_PWD"
else
    echo "ERROR: Failed to create user $NEW_USER." >&2
    exit 1
fi
```

---

### Script 3: Disk Space Alert Script
Scans disk utilization and prints warning notifications if capacity limits exceed.
```bash
#!/bin/bash
# Description: Server Disk Utilization Monitor
# Threshold: 80% usage limit

THRESHOLD=80
ADMIN_EMAIL="sysadmin@company.com"

# Parse disk usage of root partition
CURRENT_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')

if [ "$CURRENT_USAGE" -gt "$THRESHOLD" ]; then
    MSG="CRITICAL: Root partition '/' disk space usage is at $CURRENT_USAGE% (Threshold is $THRESHOLD%)."
    echo "$MSG"
    # Send email (requires postfix/mailx configured)
    # echo "$MSG" | mail -s "Disk Space Alert - $(hostname)" "$ADMIN_EMAIL"
else
    echo "OK: Disk space is normal at $CURRENT_USAGE%."
fi
```

---

### Script 4: Service Monitor and Restarter Script
Checks if critical processes are active; restarts them if offline.
```bash
#!/bin/bash
# Description: Service Health Monitor
# Target: NGINX / HTTPD

SERVICE="nginx"

# Check if service is active
systemctl is-active --quiet "$SERVICE"

if [ $? -ne 0 ]; then
    echo "WARNING: Service $SERVICE is down! Attempting restart."
    systemctl start "$SERVICE"
    
    # Recheck status
    systemctl is-active --quiet "$SERVICE"
    if [ $? -eq 0 ]; then
        echo "SUCCESS: Service $SERVICE was successfully restarted."
    else
        echo "CRITICAL: Failed to restart $SERVICE. Manual intervention needed!" >&2
        exit 1
    fi
else
    echo "OK: Service $SERVICE is running."
fi
```

---

### Script 5: Log Rotation and Archiving Script
Cleans up logs directory by compressing older files.
```bash
#!/bin/bash
# Description: Log Rotator
# Target: /var/log/custom_app/

LOG_DIR="/var/log/custom_app"
RETENTION_DAYS=7

if [ ! -d "$LOG_DIR" ]; then
    echo "ERROR: Directory $LOG_DIR does not exist." >&2
    exit 1
fi

# Find files ending in .log older than retention days, compress them, then delete originals
find "$LOG_DIR" -type f -name "*.log" -mtime +"$RETENTION_DAYS" | while read -r LOG_FILE; do
    echo "Archiving log: $LOG_FILE"
    gzip "$LOG_FILE"
done

# Purge compressed archives older than 30 days
find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -exec rm -f {} \;
echo "Log rotation cleanup completed."
```

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
> A running Linux VM terminal console with root access, and a directory `/data/source_files` containing dummy text files.

### Step 1: Create Directories and Lab Files
1. Create directories:
   ```bash
   mkdir -p /data/source_files
   mkdir -p /backup
   echo "Dummy log entry 1" > /data/source_files/app.log
   ```

### Step 2: Deploy and Test the Backup Script
1. Create script file:
   ```bash
   vim ~/backup_job.sh
   ```
2. Paste the content of **Script 1** (Backup Script) into the file. Save and close.
3. Grant execution rights:
   ```bash
   chmod 700 ~/backup_job.sh
   ```
4. Execute the script:
   ```bash
   ~/backup_job.sh
   ```
5. **Verify:** Check `/backup` contents. You should see a `.tar.gz` archive containing the `/data/source_files` directory.

### Step 3: Schedule the Script via Cron
1. Open the cron schedule manager:
   ```bash
   crontab -e
   ```
2. Append the schedule to run the backup script every night at 2:00 AM:
   ```text
   0 2 * * * /home/root/backup_job.sh >/dev/null 2>&1
   ```
3. Save and close. Verify the active cron list:
   ```bash
   crontab -l
   ```

---

---
## Cheat Sheet / Quick Reference
| Command / Configuration | Scope | Purpose / Example |
|---|---|---|
| `systemctl status <service>` | Linux | Check status of system service |
| `ip address show` | Linux | Display local interface network details |
| `Get-Service` | PowerShell | Verify service status on Windows hosts |
| `Test-NetConnection` | PowerShell | Check network path connectivity to target ports |

---
## Troubleshooting
| Problem | Cause | Fix | Command |
|---|---|---|---|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall | `ping -c 4 <ip>` / `nc -zv <ip> <port>` |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password | N/A |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects | N/A |

---
## Interview Questions
> [!question] L1 Question
> **Q:** How do you verify if the target service is running?
> **A:** On Linux, I would execute `systemctl status <service-name>`. On Windows, I would run `Get-Service <service-name>` in PowerShell or check Services.msc.

> [!question] L2 Question
> **Q:** Explain how you would troubleshoot a network connectivity issue to a remote server.
> **A:** I would verify local IP configuration, test routing gateway using `ping`, trace hops using `traceroute` or `tracert`, and check port accessibility using `telnet` or `Test-NetConnection` on target port.

---
## Seedha Simple Mein
*Seedha simple mein: Bash scripting multiple Linux commands ko ek file mein daal kar automation rules build karne ka process hai. Hum variables, loops aur condition matching ka use karke auto backup, user creation, aur disk space monitoring scripts design karte hain.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Shell navigation and basics.
- [[02-Operating-Systems/04-Linux-RHEL/L-04 Text Editors and File Viewing|L-04 Text Editors and File Viewing]] — File redirections and Vim usage.
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Wrapping scripts inside systemd units.
