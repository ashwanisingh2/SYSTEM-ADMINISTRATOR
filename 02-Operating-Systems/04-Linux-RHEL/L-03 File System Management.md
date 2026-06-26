---
tags: [desktop-support, linux, rhel, L1]
aliases: [l-03-file-system-management, l-03]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# L-03: File System Management

> [!abstract] Overview
> This note covers the Linux Filesystem Hierarchy Standard (FHS), file type classifications, metadata analysis using `ls`, link types (Hard vs. Soft), `find` query options, and compression utilities.

---

---
## Concept Overview
Think of the Linux directory structure as a single, massive organic tree growing from a single seed called the **Root (`/`)**. 
Unlike Windows, which has different trees for different drive letters (`C:\`, `D:\`), Linux mounts all storage devices, folders, and virtual systems onto branches of this single tree. 

Each branch has a designated zone:
- `/etc` is the registry vault containing config scripts.
- `/home` is the resident housing district.
- `/dev` is the hardware garage where physical devices are treated as files.
- **Soft Links** are street signs pointing to a house; if you tear down the house, the sign points to nothing.
- **Hard Links** are duplicate entries in the municipal registrar matching the exact same physical concrete foundation; if you delete one entry, the house still exists as long as the second registrar record is active.


---

---
## Technical Deep Dive
### 1. Filesystem Hierarchy Standard (FHS)
Linux organizes filesystems according to the FHS specification:

- **`/` (Root):** The parent directory of the entire system.
- **`/bin`:** Essential user command binaries (e.g., `ls`, `bash`). Replicated as symbolic link to `/usr/bin` in modern distros.
- **`/sbin`:** Essential system binaries used by root for administration (e.g., `fdisk`, `reboot`). Linked to `/usr/sbin`.
- **`/etc`:** Host-specific configuration files (settings, scripts). Replaces the Windows Registry.
- **`/var`:** Variable data files. Contains logs (`/var/log`), mail queues, database files, and web server root files.
- **`/home`:** Personal directories for standard users (e.g., `/home/sysadmin`).
- **`/root`:** Personal home directory for the root superuser.
- **`/usr`:** User binaries, libraries, and documentation. Non-volatile read-only system files.
- **`/opt`:** Add-on application software packages (third-party tools like Google Chrome or database engines).
- **`/tmp`:** Temporary files. Cleared automatically on system reboots.
- **`/dev`:** Device files representing hardware (e.g., `/dev/sda` for primary disk, `/dev/urandom` for random bytes).
- **`/proc` & `/sys`:** Virtual, in-memory filesystems representing active kernel structures, process states, and hardware settings. They consume zero disk space.

### 2. Linux File Types
In Linux, "everything is a file." Run `ls -l`; the very first character of the permissions column defines the file type:
- **`-` (Regular File):** Standard text, binary, or image files.
- **`d` (Directory):** Folder containing directory listings.
- **`l` (Link):** Symbolic shortcut link.
- **`b` (Block Device):** Buffered hardware interfaces like storage disks (`/dev/sda`).
- **`c` (Character Device):** Unbuffered serial hardware inputs like terminal consoles (`/dev/tty1`).
- **`s` (Socket):** Inter-process network communication pipes.
- **`p` (Named Pipe):** Inter-process unidirectional data queues.

### 3. Parsing `ls -l` Metadata Output
```
-rwxr-xr-x.  1 root root  18235 Jun 25 12:18 backup.sh
^    ^    ^  ^  ^    ^      ^        ^         ^
|    |    |  |  |    |      |        |         +-- Filename
|    |    |  |  |    |      |        +------------ Modification Date
|    |    |  |  |    |      +--------------------- File Size (Bytes)
|    |    |  |  |    +---------------------------- Owner Group
|    |    |  |  +--------------------------------- Owner User
|    |    |  +------------------------------------ Link Count
|    |    +--------------------------------------- Others Permissions (read/execute)
|    +-------------------------------------------- Group Permissions (read/execute)
+------------------------------------------------- Owner Permissions (read/write/execute)
```

### 4. Hard Links vs. Soft (Symbolic) Links
Every file is referenced by an **inode** (index node) containing metadata (size, permissions, physical block locations on disk) except the filename.

- **Hard Link:**
  - A directory entry pointing directly to the file's original inode.
  - Sharing the same inode number.
  - Cannot span across different filesystems/disks.
  - If the original file is deleted, the data is still accessible via the hard link.
- **Soft Link (Symlink):**
  - A shortcut file containing the path string pointing to the original file.
  - Gets a unique inode number.
  - Can span across different filesystems/disks.
  - If the original file is deleted, the symlink breaks ("broken link" pointing to nothing).

### 5. Advanced Find Options
`find` searches files dynamically in real-time.
- `find /etc -type f -name "*.conf"` — Search only files (`-type f`) matching name pattern.
- `find /var/log -mtime -7` — Find files modified (`-mtime`) within the last 7 days.
- `find /home -user sysadmin` — Find files owned by a specific user.
- `find /data -size +100M` — Find files larger than 100 Megabytes.
- `find /tmp -type f -name "*.tmp" -exec rm -f {} \;` — Execute command (`rm`) on all matching files.

### 6. Compression & Archiving
- **tar (Tape Archive):** Groups multiple files into one file (archive), but does not compress by default.
  - `tar -cvf archive.tar /data` — Create (`-c`), verbose list (`-v`), file name (`-f`).
- **Compression Gzip/Bzip2/XZ:**
  - **Gzip:** Fast speed, moderate compression. Extension: `.tar.gz`.
    - *Command:* `tar -czvf backup.tar.gz /data` (`-z` filters through gzip).
  - **Bzip2:** Slower speed, higher compression. Extension: `.tar.bz2`.
    - *Command:* `tar -cjvf backup.tar.bz2 /data` (`-j` filters through bzip2).
  - **XZ:** Slowest speed, maximum compression rates. Extension: `.tar.xz`.
    - *Command:* `tar -cJvf backup.tar.xz /data` (`-J` filters through xz).

---

## Common Mistakes
> [!warning] Avoid These
> **Creating recursive hard links to directories:** Attempting to run `ln` on a directory directory to create a hard link. This is strictly blocked by the operating system kernel because it would create infinite directory loops, breaking filesystem navigation and index sweeps.
> **Correct approach:** Only create soft symbolic links (`ln -s`) when linking directory targets.

---

## Pro Tips
> [!tip] Field Experience
> When compressing massive data directories over remote SSH terminals, avoid gzip. Use XZ (`tar -cJvf`) if bandwidth is limited because it achieves significantly higher compression ratios, saving transfer time. If CPU usage is the bottleneck, use pigz (parallel gzip) to utilize all CPU cores instead of standard single-core gzip.

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
> A running Linux terminal session.

### Step 1: Navigating the File System
1. Go to the configuration directory:
   ```bash
   cd /etc
   ```
2. List files, sorted by modification date, showing sizes in human-readable formats:
   ```bash
   ls -lt -h
   ```
3. Locate the primary system log file using long output options:
   ```bash
   ls -la /var/log/messages
   ```

### Step 2: Link Creation and Verification
1. Create a dummy file in your home folder:
   ```bash
   echo "Target text" > ~/target.txt
   ```
2. Create a hard link pointing to the file:
   ```bash
   ln ~/target.txt ~/hard_link.txt
   ```
3. Create a soft link pointing to the file:
   ```bash
   ln -s ~/target.txt ~/soft_link.txt
   ```
4. Verify inode numbers:
   ```bash
   ls -i ~/target.txt ~/hard_link.txt ~/soft_link.txt
   ```
   **Verify:** Note that `target.txt` and `hard_link.txt` share the **exact same inode number**, while `soft_link.txt` has a unique inode.
5. Delete the original file: `rm ~/target.txt`.
6. Read the links:
   - `cat ~/hard_link.txt` should output "Target text".
   - `cat ~/soft_link.txt` will return a "No such file or directory" error because the symlink is broken.

### Step 3: Find and Compress Files
1. Find all configuration files modified in `/etc` within the last day:
   ```bash
   find /etc -type f -mtime -1
   ```
2. Create a compressed backup of your home folder configuration files:
   ```bash
   tar -czf ~/home_configs.tar.gz ~/.*.txt 2>/dev/null
   ```

---

---
## Cheat Sheet / Quick Reference
```bash
# Display inode numbers of files in current directory
ls -i

# Force disk write sync and output filesystem usage details
df -hT

# Check inode utilization of mounted partitions
df -i
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | FHS | Filesystem Standard separating configuration (/etc), binaries (/bin), and logs (/var). |
| 2 | Inode | File metadata index containing file size, blocks location, permissions, and owner. |
| 3 | Soft Link | Symbolic link (shortcut) pointing to a target filepath; breaks if original is deleted. |
| 4 | Hard Link | Duplicate directory pointer sharing the same inode; remains active if original is deleted. |
| 5 | XZ Compression | High-ratio compression utility; slower execution but generates smallest archive footprints. |

---

---
## Troubleshooting
**Scenario 1:**
- **Problem:** A server reports it is out of disk space (`No space left on device` error). However, running `df -h` shows that only $60\%$ of the storage volume capacity is used.
- **Root Cause:** Inode exhaustion. The system has millions of micro-files (e.g., broken session logs or mail queues) which have consumed all available inode addresses in the filesystem metadata table, preventing new file creations even though physical block space remains free.
- **Fix:**
  1. Verify inode utilization:
     ```bash
     df -i
     ```
  2. If `/var` or `/` shows $100\%$ inode usage, locate the directories containing the largest count of files:
     ```bash
     find /var -xdev -type d -exec sh -c 'echo "$(find "$1" -type f | wc -l) $1"' _ {} \; | sort -n
     ```
  3. Clean up the folder containing the massive file counts (e.g., delete stale sessions or cache directories).

**Scenario 2:**
- **Problem:** After updating an application, a critical configuration symlink `app.conf` displays in red when listed with `ls` and application crashes.
- **Root Cause:** A broken symbolic link. The target file path was renamed or deleted during the update process.
- **Fix:**
  1. Read the symlink path target:
     ```bash
     readlink app.conf
     ```
  2. Verify if the target file exists at that path.
  3. If missing, locate the new configuration file path using `find` or recreate the link:
     ```bash
     rm app.conf
     ln -s /etc/app/new_version.conf app.conf
     ```

---

---
## Interview Questions
**Q1: Explain the structural difference between a Hard Link and a Soft Link.**
A: A **Hard Link** is a directory entry that points directly to the same physical inode as the original file. They share the same inode number and metadata. Modifying the content of either modifies the file, and deleting the original file does not delete the data because the hard link still references the inode. Hard links cannot link directories or span across different filesystem partitions. A **Soft Link** (symlink) is a unique file containing a path string that points to the original file. It has its own inode number. If the original file is deleted, the symlink remains but becomes broken, pointing to a non-existent path. Symlinks can target directories and cross partition boundaries.

**Q2: A server's backup script fails with a write error. Running df shows /backup is at $100\%$ utilization. Explain how you would identify the largest files clogging the volume.**
A: 
- **Situation:** The backup partition is full, and large files must be purged.
- **Task:** Locate and list files larger than a specific limit in order of size.
- **Action:** I will run the `find` command targeting the `/backup` mount point. I will filter for files (`-type f`) larger than 500MB (`-size +500M`) and execute a long list sorting command.
  ```bash
  find /backup -type f -size +500M -exec ls -lh {} \; | sort -k 5 -h -r
  ```
- **Result:** The output lists the largest files in descending order, allowing me to safely archive or purge stale logs.

**Q3: Describe the role of virtual directories `/proc` and `/sys` in the Linux filesystem.**
A: `/proc` (process information pseudo-filesystem) and `/sys` (system pseudo-filesystem) are virtual filesystems generated dynamically in memory by the Linux kernel on boot. They contain no real data blocks on disk and consume zero bytes. `/proc` exposes active process states (grouped by PID directories), system memory stats (`/proc/meminfo`), and kernel configurations. `/sys` exposes hardware device structures, driver states, and kernel subsystem tunables. Writing to specific files in these directories (e.g., echoing a value to `/proc/sys/net/ipv4/ip_forward`) dynamically alters kernel behavior without rebooting.

---

---
## Seedha Simple Mein
*Seedha simple mein: Linux mein saari files '/' (root) directory ke andar store hoti hain. Har folder ka FHS ke mutabik ek rules hota hai (jaise /etc settings ke liye aur /var changing logs ke liye). Soft links shortcut ki tarah kaam karte hain aur Hard links exact copy hote hain.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Shell execution syntaxes.
- [[02-Operating-Systems/04-Linux-RHEL/L-04 Text Editors and File Viewing|L-04 Text Editors and File Viewing]] — Accessing configuration files in /etc.
- [[02-Operating-Systems/04-Linux-RHEL/L-06 File Permissions and Ownership|L-06 File Permissions and Ownership]] — Detail on rwx bits listed in ls -l.
