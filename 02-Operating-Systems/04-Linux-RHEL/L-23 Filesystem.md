---
tags: [linux, rhel, filesystem, storage]
aliases: [L-23, Linux Filesystem Hierarchy, Filesystem]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX / RHEL**

`#complete` `#intermediate` `#rhcsa`

# L-23: Linux Filesystem Structure & Management

> [!abstract] Overview
> Yeh note Linux Filesystem Hierarchy Standard (FHS) aur partitions/mounts ke baare mein hai. Ek system administrator ko filesystem samajhna bahut zaroori hai taaki disk space manage kar sake, logs dhundh sake, aur troubleshooting kar sake jab disk full ho jaye.

---
## 🧠 Concept Overview

- **What it is** — The Linux filesystem is the structured way in which files and directories are organized on the hard drive. Unlike Windows which uses drive letters (C:, D:), Linux uses a single unified directory tree starting from the root (`/`).
- **Why it matters** — *System chalane ke liye files kahan save hoti hain, yeh pata hona chahiye.* Knowing where logs, configurations, and user data are stored helps you resolve issues quickly and allocate storage effectively.
- **Where you see this** — Almost every day, when extending disk space, mounting new storage LUNs, checking disk usage (`df -h`), or searching for application configuration files.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check disk space, clear temp files, check basic mount points, and monitor inode usage. |
| **L2** | Add new disks, create partitions (`fdisk`, `parted`), format them (`mkfs`), and mount them in `/etc/fstab`. Resolve deleted file holds. |
| **L3** | Design LVM architecture, troubleshoot corrupted filesystems, implement storage quotas, and manage SAN/NAS integrations and multipathing. |

> [!tip] Seedha Simple Mein
> *Windows mein jaise alag alag room (C:, D:) hote hain, Linux mein ek hi bada ghar hota hai (`/`) jismein alag alag rooms (folders) bane hote hain. Har folder ka ek specific kaam hota hai aur system ko pata hota hai kahan kya dhoondna hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Linux Filesystem (`/`)** is like a **Corporate Office Building** because...
>
> - **`/` (Root)** is the main entrance or reception area. Everything starts from here.
> - **`/bin` & `/sbin`** are the tool rooms. Regular employees use `/bin` (standard tools), but only building managers (root) use `/sbin` (administrative tools).
> - **`/etc`** is the HR and management office where all the rules and configuration files are kept. No binaries, only text configs.
> - **`/home`** is the cubicle area where each employee (user) has their own desk to keep personal stuff.
> - **`/var`** is the warehouse or logbook room where daily activity logs and variable data are continuously recorded.
> - **`/tmp`** is the trash bin or temporary locker room that gets cleared out regularly.
> - **`/dev`** is the hardware interface layer. If you add a new AC unit or generator, its controls appear here.

---
## 🔬 Technical Deep Dive

### 1. Filesystem Hierarchy Standard (FHS) in Detail

> [!info] Key Concept
> FHS defines the directory structure and directory contents in Unix and Unix-like operating systems. It ensures consistency across different Linux distributions, so administrators don't have to relearn file locations on every system.

**Detailed Directory Breakdown:**
- `/` - The Root Directory. The absolute peak of the filesystem.
- `/boot` - Contains the Linux kernel (`vmlinuz`), initial RAM disk image (`initramfs`), and bootloader (GRUB) configurations. *Boot hone ke liye sab kuch yahan hai.*
- `/etc` - System-wide configuration files (`/etc/passwd`, `/etc/fstab`, `/etc/ssh/sshd_config`). Never put executable programs here.
- `/home` - User personal directories (e.g., `/home/ashwani`). Usually mounted on a separate partition to protect the root filesystem from users filling it up.
- `/root` - The home directory for the root user. Note that this is different from `/` (the root directory).
- `/var` - Variable data. Crucial for logs (`/var/log`), mail queues, databases (`/var/lib/mysql`), and print queues. This directory grows constantly!
- `/usr` - User binaries, read-only user data, and applications (`/usr/bin`, `/usr/share`).
- `/dev` - Device files (e.g., `/dev/sda` for disk, `/dev/null` for blackhole). In Linux, everything is a file, including hardware.
- `/tmp` - Temporary files created by users and the system. Often cleared on reboot.
- `/opt` - Optional add-on software packages. Used primarily by proprietary software (like Oracle, Splunk).
- `/mnt` & `/media` - Directories for temporarily mounting filesystems (USB drives, network shares).
- `/proc` & `/sys` - Virtual filesystems created in RAM. Provide a window into the running kernel and system hardware (`/proc/cpuinfo`, `/proc/meminfo`).

> [!danger] Common Mistake
> Never delete files directly from `/var/log` using `rm` to free up space. This can break the logging service or file handle. Always empty them using `> /var/log/messages` or use `logrotate`.

### 2. Filesystem Types & Architecture

Linux supports multiple filesystem formats. The most common ones are:
- **EXT4:** The traditional, stable filesystem for Linux. Good performance, supports large files, and offers journaling to prevent data loss.
- **XFS:** The default filesystem in RHEL 7, 8, and 9. Excellent for handling large files and high performance. *Note: XFS cannot be shrunk, it can only be extended.*
- **BTRFS:** Advanced filesystem supporting snapshots, subvolumes, and pooling. Modern alternative in some distros.

### 3. The Concept of Inodes

Every file in Linux consists of two parts:
1. **The Data Blocks:** The actual content of the file.
2. **The Inode:** The metadata about the file (owner, permissions, timestamps, size, and pointers to the data blocks).

*Interviewers love asking about Inodes. Remember that an inode does not contain the file's name. The file name is stored in the directory block, which points to the inode.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux VM (RHEL/CentOS/Ubuntu)
> - Root or sudo privileges
> - An unpartitioned disk attached (e.g., `/dev/sdb`)

### Step 1: Identify the New Disk

First, we need to find the newly attached disk in the system.

```bash
# Yeh command system ke saare block devices dikhata hai
lsblk
```

> [!success] Expected Output
> ```
> NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
> sda      8:0    0   20G  0 disk 
> ├─sda1   8:1    0    1G  0 part /boot
> └─sda2   8:2    0   19G  0 part /
> sdb      8:16   0   10G  0 disk 
> ```
> *Notice `/dev/sdb` is present but has no partitions or mount points.*

### Step 2: Create a Partition

Use `fdisk` to create a new primary partition on `/dev/sdb`.

```bash
# fdisk tool se disk pe partition banayenge
sudo fdisk /dev/sdb

# Inside fdisk interactive prompt:
# Type 'n' for new partition
# Type 'p' for primary
# Press Enter for default partition number (1)
# Press Enter for default first sector
# Press Enter for default last sector (uses full disk)
# Type 'w' to write and save changes
```

### Step 3: Format the Partition with XFS

Now format the partition with the XFS filesystem.

```bash
# XFS format mein filesystem banayenge
sudo mkfs.xfs /dev/sdb1
```

### Step 4: Mount the Filesystem

Create a mount point and mount the newly formatted partition.

```bash
# Ek naya folder banayenge kahan disk attach karna hai
sudo mkdir /data

# Temporary mount command
sudo mount /dev/sdb1 /data

# Verify mount
df -h /data
```

### Step 5: Make Mount Persistent (fstab)

If you reboot, the temporary mount will be lost. We must add it to `/etc/fstab`.

```bash
# Disk ka UUID nikalenge taaki safely mount ho
blkid /dev/sdb1

# fstab mein entry add karenge
sudo nano /etc/fstab
# Add line: UUID=<your-uuid> /data xfs defaults 0 0

# Check fstab syntax to avoid boot issues
sudo mount -a
```

> [!bug] fstab Warning
> *Agar `fstab` mein entry galat hui, toh server reboot hone ke baad emergency mode mein chala jayega. Hamesha `mount -a` se check karo reboot se pehle!*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `df -h` | Disk space usage in human-readable format | `df -h /` |
| `df -i` | Check inode usage | `df -i /` |
| `du -sh *` | Folder ke andar ka size check karta hai | `du -sh /var/*` |
| `lsblk` | List all block devices (disks & partitions) | `lsblk` |
| `fdisk -l` | List partition tables | `sudo fdisk -l` |
| `mkfs.ext4` | Format partition to EXT4 | `sudo mkfs.ext4 /dev/sdb1` |
| `mkfs.xfs` | Format partition to XFS | `sudo mkfs.xfs /dev/sdb1` |
| `mount` | Mount a filesystem manually | `mount /dev/sdb1 /mnt` |
| `umount` | Unmount a filesystem | `umount /mnt` |
| `blkid` | Print UUID and filesystem type of disks | `blkid /dev/sda1` |
| `findmnt` | Find mounted filesystems easily | `findmnt` |
| `lsof \| grep deleted` | Check which deleted files are holding space | `lsof \| grep deleted` |
| `tune2fs` | Adjust tunable filesystem parameters on ext filesystems | `tune2fs -m 1 /dev/sda1` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `No space left on device` but `df -h` shows space | Inodes exhaust ho gaye hain (Too many small files). | Run `df -i`. Find folder with many files: `find / -xdev -type f \| cut -d "/" -f 2 \| sort \| uniq -c \| sort -n` |
| Disk is 100% full, deleted log file but space not freed | Koi process abhi bhi deleted file ko hold karke baitha hai. | Run `lsof \| grep deleted`. Find the PID and restart that service. |
| Server boots into Emergency Mode | `/etc/fstab` mein galat entry hai ya disk corrupt hai. | Give root password, open `/etc/fstab`, comment out the bad entry (`#`), save, and reboot. |
| Read-only file system | Disk hardware error, filesystem corruption, or VMware snapshot issue. | Check logs: `dmesg \| grep -i ext4` or `dmesg \| grep -i xfs`. Run `fsck` if required (unmount first!). |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Disk Space Full on /var

> [!example] Ticket
> "Critical Alert: Root filesystem /var is 98% full on app-server-01."

**L1 Response:** Check disk usage using `df -h` and find the culprit using `du -sh /var/*`. Typically, it's `/var/log` or `/var/cache`.
**Escalation Trigger:** If it's a database file or application data that cannot be deleted or if we need to expand the disk size.
**L2 Resolution:** 
1. Use `lsof | grep deleted` to clear zombie file holds.
2. If space is genuinely needed, expand the disk at the VMware/AWS level.
3. Rescan SCSI bus: `echo 1 > /sys/class/scsi_device/device/rescan`
4. Use `lvextend` and `xfs_growfs` to increase the filesystem size safely without downtime.

### 🎫 Scenario 2: Emergency Mode After OS Patching

> [!example] Ticket
> "Server rebooted after OS patching but didn't come up. Console shows 'Welcome to Emergency Mode'."

**L1 Response:** Check server console in vCenter/iLO. Notice the prompt asking for root password. This indicates a boot failure.
**Escalation Trigger:** Always pass to L2 for filesystem corruption/boot issues.
**L2 Resolution:**
1. Enter root password to access the shell.
2. View the system journal: `journalctl -xb | grep -i mount` to find which disk failed to mount.
3. Open `/etc/fstab` and comment out the failing non-critical mount (like an old NFS share, a missing iSCSI LUN, or a deleted data disk).
4. Run `systemctl reboot`.

### 🎫 Scenario 3: Accidental Deletion of Critical Logs

> [!example] Ticket
> "I deleted the /var/log/secure file to free up space, but now the file is not recreating and SSH logins aren't being logged."

**L1 Response:** Explain that `rm` should not be used on active log files because the rsyslog daemon still holds the file descriptor open to the deleted inode.
**Escalation Trigger:** Pass to L2 if service needs to be restored safely across production servers.
**L2 Resolution:**
1. Create the file manually: `touch /var/log/secure`.
2. Set correct permissions: `chmod 600 /var/log/secure`.
3. Restart the rsyslog service: `systemctl restart rsyslog`.
*Next time, train the user to use `> /var/log/secure` to truncate the file while preserving the inode.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between soft link and hard link?
> **Answer:** 
> - **Soft Link (`ln -s`):** Like a Windows shortcut. It points to the file name. If the original file is deleted, the soft link becomes broken (dangling). Can link across different filesystems and directories.
> - **Hard Link (`ln`):** Points to the exact inode on the disk. Both files have the same inode number. If the original file is deleted, data still exists through the hard link. Cannot link across different filesystems or to directories.
> ==**Exam Tip:** Hard links share the same inode number. Check with `ls -i`.==

> [!question] Q2: Your disk is showing 100% full, but when you check with `du`, you only see 50% usage. What could be the issue?
> **Answer:** A running process is still holding a file that was deleted. The system won't free the blocks until the process releases the file handle. You can find this using `lsof | grep deleted` and fix it by restarting the respective service (like `systemctl restart httpd`).

> [!question] Q3: What is an Inode in Linux?
> **Answer:** An inode (Index Node) is a data structure on a filesystem that stores metadata about a file (permissions, owner, size, timestamps, and where the data blocks are physically located on the disk). It does NOT store the file's name or its actual data.
> ==**Exam Tip:** You can run out of inodes (`df -i`) before you run out of disk space if you have millions of tiny 1KB files.==

> [!question] Q4: How do you mount a filesystem automatically on boot?
> **Answer:** By adding an entry into the `/etc/fstab` file. The entry requires 6 fields: Device (or UUID), Mount Point, Filesystem Type, Options (usually `defaults`), Dump (0), and Fsck Pass (0 or 2). Always test with `mount -a` before rebooting to prevent emergency mode.
> ==**Exam Tip:** Always use UUID (`blkid`) instead of device names like `/dev/sdb1` in `/etc/fstab` because device names can change upon reboot if new drives are added.==

> [!question] Q5: You accidentally unmounted `/tmp`. What is the impact and how to fix it?
> **Answer:** Applications relying on `/tmp` for temporary lock files, socket files (like MySQL), or temporary caching might crash or fail to start. You can remount it using `mount -a` if the entry exists in `/etc/fstab`, or run `systemctl daemon-reload` and restart affected services to recreate their temporary sockets.

---
## 🔗 Related Notes

- [[L-24 LVM Management|L-24 Logical Volume Management (LVM)]] — Next step in advanced storage administration.
- [[L-15 Linux Boot Process|L-15 Linux Boot Process]] — Understand how fstab is read during boot and initramfs execution.
- [[L-08 File Permissions|L-08 File Permissions (chmod, chown)]] — Important for securing directories like /tmp and /var.
