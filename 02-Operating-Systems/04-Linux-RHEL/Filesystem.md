---
tags: [desktop-support, linux, filesystem, storage, L1]
aliases: [linux-fhs, lvm-guide, fstab-guide]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #rhcsa
---

# Linux Filesystem Structure and Management

---

## Concept Overview
- **What it is**: The Linux Filesystem is a hierarchical structure starting from the root directory (`/`) that governs how data is stored, read, and managed on physical and logical storage drives. In Red Hat Enterprise Linux (RHEL), this follows the Filesystem Hierarchy Standard (FHS).
- **Why it matters for a support engineer**: Linux hosts corporate databases, web servers, and development containers. A support engineer must know how to inspect disk usage, mount physical or network drives, configure persistent mount points, and expand logical volumes to resolve application crashes caused by out-of-disk-space conditions.
- **Where you encounter this in real job**: Troubleshooting database crashes caused by full `/var` partitions, mounting network file systems (NFS), adding virtual disks to VMs, and auditing disk health.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Checks disk space availability (`df -h`), locates large log files (`du`), and clears temporary storage.
  - **L2**: Partitions and formats new disks, manages Logical Volume Manager (LVM) physical volumes and volume groups, and configures automated mounting in `/etc/fstab`.
  - **L3**: Designs high-availability distributed storage clusters, debugs kernel-level I/O issues, configures storage area networks (SAN), and manages RHEL enterprise backup matrices.

---

## Technical Deep Dive

### 1. Filesystem Hierarchy Standard (FHS)
Unlike Windows, which partitions data under drive letters (`C:`, `D:`), Linux represents everything as files under a single root directory (`/`):

```
                                  [/] (Root)
       ___________________________|___________________________
      |       |       |       |       |       |       |       |
   [/bin]  [/boot] [/etc]  [/home] [/root] [/usr]  [/var]  [/dev]
```

- **`/bin` / `/sbin`**: Essential binaries (user commands) and system administration binaries (e.g., `systemctl`, `ip`).
- **`/boot`**: Holds static files needed to boot the system (Kernel, Grub bootloader, initramfs).
- **`/etc`**: Host-specific system-wide configuration files (analogous to the Windows Registry).
- **`/home`**: Personal user directories (e.g., `/home/john`).
- **`/root`**: The home directory for the superuser/root administrator.
- **`/dev`**: Device files representing hardware components (e.g., `/dev/sda` for a disk, `/dev/urandom`).
- **`/usr`**: User programs, libraries, and documentation.
- **`/var`**: Variable files that grow over time (logs in `/var/log`, databases in `/var/lib`, mail queues).
- **`/tmp`**: Temporary files (wiped automatically on boot).

### 2. Logical Volume Manager (LVM)
LVM is a storage virtualization technology that abstracts physical storage into dynamic logical volumes that can be resized on-the-fly:

```
[Physical Hard Drives] --> /dev/sdb  /dev/sdc
                               |
                        Physical Volumes (PV)
                               |
                         [Volume Group (VG)] (Aggregated Pool)
                               |
                        Logical Volumes (LV) --> /dev/mapper/vg0-lv_data
                               |
                           [Mount Point] (Formatted with xfs/ext4)
```

1. **Physical Volume (PV)**: Raw hard disks or partitions initialized for LVM (e.g., `pvcreate /dev/sdb`).
2. **Volume Group (VG)**: An aggregated storage pool created by combining one or more PVs (e.g., `vgcreate vg_data /dev/sdb /dev/sdc`).
3. **Logical Volume (LV)**: A slice of the VG that is formatted with a filesystem (like XFS or EXT4) and mounted to a directory (e.g., `lvcreate -n lv_web -L 50G vg_data`). LVs can be dynamically expanded as storage needs grow.

### 3. Persistent Mounting and `/etc/fstab`
To make a mounted disk persistent across system reboots, it must be added to the `/etc/fstab` configuration file. The file has six columns:
`[Device/UUID]  [Mount Point]  [Filesystem Type]  [Options]  [Dump]  [Pass]`
- **Options**: `defaults` allows read/write, auto-mount, and execute permissions.
- **Dump**: Set to `0` to disable backup checks.
- **Pass**: Set to `1` for the root filesystem check, `2` for other filesystems, and `0` to skip fsck checks.

---

## Commands & Syntax

### Bash
```bash
# Check disk space usage on all mounted filesystems in human-readable format
df -h

# Estimate file space usage, showing the top 10 largest folders/files in /var
du -ah /var | sort -hr | head -n 10

# List block devices (disks, partitions, and LVM logical volumes)
lsblk

# Scan for newly attached virtual disks without rebooting the system
echo "- - -" > /sys/class/scsi_host/host0/scan

# Create a Physical Volume on a raw disk
pvcreate /dev/sdb

# Create a Volume Group named vg_prod using the physical volume
vgcreate vg_prod /dev/sdb

# Create a Logical Volume named lv_data of size 20GB inside vg_prod
lvcreate -n lv_data -L 20G vg_prod

# Format the logical volume with the XFS filesystem
mkfs.xfs /dev/mapper/vg_prod-lv_data

# Mount the volume to a target directory
mount /dev/mapper/vg_prod-lv_data /mnt/data
```

---

## Real-World Scenarios

### Scenario 1: Web Server Fails to Start Due to Out-of-Disk-Space Condition
**User Complaint:** A developer submits a high-priority ticket: *"The production Apache web server crashed and will not start back up. Users are getting connection timeout errors. System log is throwing 'No space left on device'."*
**Your First 3 Checks:**
1. Check overall disk space availability using `df -h`.
2. Find which partition is at 100% capacity.
3. Locate the specific directories/files consuming the storage using `du`.
**Diagnosis Steps:**
1. SSH into the server as root. Run the disk space query:
   `df -h`
   - Output: `/dev/mapper/rhel-root` is at `100%` capacity, specifically the `/var` directory partition.
2. Navigate to `/var` and search for large files:
   `du -ah /var/log | sort -hr | head -n 5`
   - Top Result: `/var/log/httpd/access_log` is `45GB` in size.
3. *Why did it grow?* The log rotation configuration failed, and Apache has been writing raw HTTP requests to a single log file for a year.
**Root Cause:** A log file grew uncontrolled due to a failure in log rotation configurations, consuming all root filesystem space.
**Fix:**
1. Clear the active log file without locking the process (do not delete the file while Apache is trying to write to it):
   `cat /dev/null > /var/log/httpd/access_log`
   - *This empties the file instantly without breaking file descriptors.*
2. Check disk space again: `df -h`. The utilization drops to `45%`.
3. Start the Apache web service:
   `systemctl start httpd`
4. Configure the log rotation script (`/etc/logrotate.d/httpd`) to rotate logs weekly and compress old files.
**Prevention:** Enforce quotas on log partitions or mount `/var/log` on a separate logical volume.
**Ticket Close Note:** "Cleared HTTP access logs. Configured weekly logrotate rules. Web server started successfully. Closed."

### Scenario 2: Mounting a New Virtual Disk Dynamically using LVM
**User Complaint:** A database administrator requests: *"We added a new 100GB virtual disk to the RHEL VM. Please assign it to our '/var/lib/mysql' database directory, as it is running out of space."*
**Your First 3 Checks:**
1. Check if the newly attached disk is visible in the OS using `lsblk`.
2. Determine if the database folder is currently hosted on an LVM logical volume.
3. Check the Volume Group name to verify available extension space.
**Diagnosis Steps:**
1. Run `lsblk` to identify the new disk.
   - Find: `sdb` is a raw 100GB unpartitioned disk.
2. Check the active logical volume properties:
   `lvdisplay`
   - The database folder `/var/lib/mysql` is mounted to `/dev/mapper/rhel-lv_mysql`.
3. Instead of mounting the new disk as a separate directory (which would break the database paths), we can extend the existing Logical Volume using LVM.
**Root Cause:** Requirement for storage expansion on an active directory database volume.
**Fix:**
1. Initialize the new physical disk for LVM:
   `pvcreate /dev/sdb`
2. Extend the existing Volume Group `rhel` to include the new PV:
   `vgextend rhel /dev/sdb`
3. Extend the Logical Volume to use all newly added free space:
   `lvextend -l +100%FREE /dev/rhel/lv_mysql`
4. Resize the filesystem dynamically while mounted (XFS supports online growth):
   `xfs_growfs /var/lib/mysql`
   - *(If using ext4, use `resize2fs /dev/rhel/lv_mysql` instead).*
5. Run `df -h` to verify `/var/lib/mysql` has grown by 100GB.
**Prevention:** Build monitoring scripts to alert when LVM volume groups fall below 10% free space capacity.
**Ticket Close Note:** "Initialized new physical disk dev sdb. Extended rhel Volume Group and lv_mysql Logical Volume. Grew XFS filesystem online. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never add entries to `/etc/fstab` without verifying the mount first using `mount -a`.
> - If `/etc/fstab` contains a syntax error, a misspelled UUID, or references a missing drive, **the Linux system will crash on the next reboot** and enter emergency maintenance mode. Always run `mount -a` to test the configuration. If it returns zero errors, the file is safe.

> [!warning] Common Trap
> - Deleting a large active log file using `rm` to free up space.
> - If a process (like Apache or MySQL) is actively writing to a file when you run `rm`, the file pointer is deleted but the disk space is not released. The process keeps the file open in memory. You must empty the file (`cat /dev/null > logfile`) or restart the service to free the space.

> [!tip] Senior Engineer Tip
> - When adding entries to `/etc/fstab`, always identify disks using their **UUID** (Universally Unique Identifier) rather than their device name (like `/dev/sdb`). Device names can change when the VM is rebooted or hardware is re-ordered, which can cause mounting failures. Use `blkid` to find the UUID.

> [!success] Verification Steps
> - Run `lsblk` and `df -h` to verify that new disks are mounted to the correct directories.
> - Run `mount -a` to verify `/etc/fstab` integrity.

> [!question] Interview Alert
> - "Explain the difference between `df` and `du` commands."
> - Answer: "`df` (disk free) displays the total disk space available and used across all mounted filesystems by reading the filesystem superblock. `du` (disk usage) estimates the space used by specific files or directories by recursively scanning individual files, making it the tool to locate specific large files consuming space."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Running `rm` on active log files | Impatience and lack of process knowledge | Use `> filename` or `cat /dev/null > filename` to truncate active files. |
| Misspelling drive labels in `/etc/fstab` | Typos in device paths | Always run `blkid` to copy the exact UUID, and verify with `mount -a`. |
| Creating file systems on raw disks without LVM | Quick setup habits | Always deploy storage using LVM to allow for future online disk extensions. |

---

## Lab Exercise

**Objective:** Add a virtual disk to a running Linux VM, initialize it using LVM, format it with XFS, mount it persistently, and verify its configuration.
**Time Required:** 30 minutes
**Environment Needed:** A RHEL or Rocky Linux VM with an unformatted secondary virtual disk (`/dev/sdb` or `/dev/nvme1n1`).
**Pre-requisites:** Root/sudo privileges.

**Steps:**
1. Open a terminal. Scan for new disks and list block devices:
   ```bash
   lsblk
   ```
   - Verify that `/dev/sdb` (or equivalent) is visible and unpartitioned.
2. Initialize LVM PV and VG:
   ```bash
   pvcreate /dev/sdb
   vgcreate vg_lab /dev/sdb
   ```
3. Create a Logical Volume using 100% of the volume group:
   ```bash
   lvcreate -n lv_data -l +100%FREE vg_lab
   ```
4. Format the volume with the XFS filesystem:
   ```bash
   mkfs.xfs /dev/mapper/vg_lab-lv_data
   ```
5. Create a target directory and mount the volume:
   ```bash
   mkdir -p /mnt/labdata
   mount /dev/mapper/vg_lab-lv_data /mnt/labdata
   ```
6. Find the UUID of the new filesystem:
   ```bash
   blkid | grep vg_lab-lv_data
   ```
   - Copy the UUID (e.g., `UUID="1234-abcd-..."`).
7. Edit `/etc/fstab` using `vi` or append via echo:
   ```bash
   echo "UUID=1234-abcd-... /mnt/labdata xfs defaults 0 0" >> /etc/fstab
   ```
8. Verification: Unmount the drive and test persistence:
   ```bash
   umount /mnt/labdata
   mount -a
   df -h | grep labdata
   ```
   - Confirm `/mnt/labdata` is mounted.

**Success Criteria:** The logical volume is formatted, mounted, added to `/etc/fstab` using the UUID, and verified using `mount -a`.
**Common Failures:** The system enters emergency boot mode on restart if the UUID or mount path in `/etc/fstab` has a typo.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the purpose of the `/etc` directory in Linux?**
A: The `/etc` directory contains all system-wide configuration files for the operating system and installed services. It is equivalent to the registry in Windows systems.

**Q: Which command do you run to check if a hard drive partition is almost full?**
A: I run the `df -h` command. This shows a list of all mounted filesystems, their total size, space used, space available, and the percentage utilized in a human-readable format.

### Intermediate (L2 Level)
**Q: Why should you use Logical Volume Manager (LVM) instead of standard partitioning like fdisk?**
A: LVM abstracts physical disks into a flexible storage pool. Unlike standard partitions, which are fixed in size, LVM logical volumes can be resized dynamically (extended or shrunk online) across multiple physical drives without unmounting directories or losing data.

**Q: What does the command `mount -a` do, and why is it important to run after editing `/etc/fstab`?**
A: The `mount -a` command reads `/etc/fstab` and attempts to mount all filesystems listed inside that are not currently mounted. It is important to run after editing `/etc/fstab` because it validates the syntax; if there is an error in the configuration, the command returns an error immediately, alerting you to fix it before the system boots and crashes.

### Advanced (L3/Senior Level)
**Q: A Linux server is reporting a disk space exhaustion alert on the root filesystem, but running `du -sh /*` does not show where the space is being used. How do you troubleshoot this?**
A:
- **Situation**: Disk space is full but du scan fails to locate the files.
- **Task**: Identify hidden disk consumption.
- **Action**: This issue occurs when a large file is deleted while a running process still has it open. The file system superblock marks the blocks as used, but `du` cannot see the file because its directory link is gone. I run the command `lsof +L1` (or `lsof | grep deleted`) to list all open files with a link count of zero. Once I locate the process holding the file open (e.g., a syslog daemon holding a deleted log file), I restart that service.
- **Result**: The process closes its file descriptor, releasing the blocks and restoring disk space immediately.

### HR / Behavioral
**Q: Describe a time you had to solve a technical issue where the initial troubleshooting steps failed. How did you proceed?**
A: Our database server crashed due to disk space issues on `/var`. I ran `du` to locate large files but found nothing abnormal, even though `df -h` showed 100% utilization. I realized standard tools weren't showing the files. I researched and ran `lsof` to check for deleted files held open by active processes. I found a deleted 50GB backup file still held open by a crashed backup script process. I killed the process, which released the disk space, allowing the database to start. I documented this process in our team's wiki to help others with similar issues.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The hierarchical file storage structure of Linux, organized from the root directory (`/`).
> **Why**: Critical for hosting database files, logs, and services, requiring active disk monitoring.
> **How**: Use `df` and `du` to monitor space, partition with LVM for flexibility, and configure persistent mounts in `/etc/fstab`.
> **Command**: `df -h` / `du -sh` / `mount -a`
> **Interview Answer Starter**: "To manage storage in a RHEL environment, I partition drives using LVM, verify disk utilization using df and du, and configure persistent mounts using UUIDs in fstab..."

**Key Numbers to Remember:**
- Default log partition directory: `/var/log`
- Exit status code for a successful `mount -a`: `0`
- Number of fields in an `/etc/fstab` entry: `6`
- Systemd mount target configuration path: `/etc/systemd/system/`

**3 Things Interviewer Wants to Hear:**
- Running `mount -a` before rebooting to test `/etc/fstab` changes
- How to release disk space from a deleted file held open by a process (using `lsof`)
- The benefits of LVM for dynamic online volume resizing

---

## Related Notes
- [[01-Foundations/01-Hardware/Storage-Deep-Dive|Storage Deep Dive]] — Outlines the hardware structures (RAID, SSDs) behind Linux volumes.
- [[02-Operating-Systems/04-Linux-RHEL/Permissions|Linux Permissions]] — Focuses on accessing directories and files inside the structure.
- [[02-Operating-Systems/04-Linux-RHEL/Systemctl|Systemctl]] — Explains system service managers that write to logs.

---

## Tags
#desktop-support #linux #filesystem #storage #L1 #interview-topic #lab-complete #daily-use

