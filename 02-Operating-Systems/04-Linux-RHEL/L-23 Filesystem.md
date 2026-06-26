---
tags: [linux, rhel, filesystem, storage, L1]
aliases: [filesystem]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX / RHEL**

`#complete` `#intermediate` `#rhcsa`

# L-23: Linux Filesystem Structure and Management

> [!abstract] Overview
> Linux Filesystem is a hierarchical structure starting from the root directory (`/`) that governs how data is stored, read, and managed. Ek support engineer ke liye yeh jaanna bahut zaroori hai taaki woh disk usage check kar sake, drives mount kar sake, aur storage full hone pe application crashes ko fix kar sake.

---
## 🧠 Concept Overview

- **What it is** — The hierarchical structure starting from the root directory (`/`) that governs data storage on physical and logical drives (following FHS in RHEL).
- **Why it matters** — Linux hosts corporate databases and web servers. Knowing how to inspect disk space and expand logical volumes prevents critical application crashes.
- **Where you see this** — Troubleshooting full `/var` partitions causing database crashes, mounting NFS, and managing virtual disks.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Checks disk space availability (`df -h`), locates large log files (`du`), and clears temporary storage. |
| **L2** | Partitions and formats new disks, manages LVM physical volumes and volume groups, and configures `/etc/fstab`. |
| **L3** | Designs high-availability distributed storage clusters, debugs kernel-level I/O issues, configures SAN. |

> [!tip] Seedha Simple Mein
> *Filesystem ke bare mein seekhna bahut zaroori hai kyunki yeh linux infrastructure aur system settings ko properly implement karne aur support tickets ko runbooks ke help se standard templates me clear karne me help karta hai. Jaise koi dost file dhoondne mein madad kar raha ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A well-organized corporate office** is like the **Linux Filesystem** because...
>
> - **The Reception (Root `/`)** is where everyone enters and is directed to their specific departments.
> - **The Utilities Closet (`/bin` & `/sbin`)** contains essential tools for employees and building management.
> - **The Records Room (`/var`)** stores logs, databases, and mail that constantly grow and need management.
> - **The Employee Cubicles (`/home`)** are personal spaces for individual workers.

---
## 🔬 Technical Deep Dive

### 1. Filesystem Hierarchy Standard (FHS)

> [!info] Key Concept
> Unlike Windows (which uses drive letters like `C:`), Linux represents everything as files under a single root directory (`/`).

```text
                                  [/] (Root)
       ___________________________|___________________________
      |       |       |       |       |       |       |       |
   [/bin]  [/boot] [/etc]  [/home] [/root] [/usr]  [/var]  [/dev]
```

- **`/bin` / `/sbin`**: Essential user and system administration binaries (`systemctl`, `ip`).
- **`/boot`**: Static files to boot the system (Kernel, Grub, initramfs).
- **`/etc`**: System-wide configuration files (like the Windows Registry).
- **`/home`**: Personal user directories.
- **`/root`**: Home directory for the superuser.
- **`/dev`**: Device files representing hardware (e.g., `/dev/sda`).
- **`/usr`**: User programs, libraries, and documentation.
- **`/var`**: Variable files that grow (logs in `/var/log`, databases in `/var/lib`).

> [!danger] Common Mistake
> Running `rm` on active log files in `/var` to free up space. The space is not released until the process is restarted!

### 2. Logical Volume Manager (LVM)

> [!info] Key Concept
> LVM abstracts physical storage into dynamic logical volumes that can be resized on-the-fly.

1. **Physical Volume (PV)**: Raw hard disks initialized for LVM (e.g., `pvcreate /dev/sdb`).
2. **Volume Group (VG)**: Aggregated storage pool combining PVs.
3. **Logical Volume (LV)**: A slice of the VG formatted with a filesystem and mounted to a directory. LVs can dynamically expand.

### 3. Persistent Mounting and `/etc/fstab`

> [!info] Key Concept
> To make a mounted disk persistent across reboots, it must be added to `/etc/fstab`.

Format: `[Device/UUID]  [Mount Point]  [Filesystem Type]  [Options]  [Dump]  [Pass]`

> [!danger] Common Mistake
> Never add entries to `/etc/fstab` without verifying the mount using `mount -a`. If it contains errors, **the Linux system will crash on the next reboot**.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Root/sudo privileges.
> - A RHEL/Rocky Linux VM with an unformatted secondary virtual disk (`/dev/sdb`).

### Step 1: Initialize LVM PV and VG

```bash
# Verify disk is visible
lsblk
# Create Physical Volume
pvcreate /dev/sdb
# Create Volume Group
vgcreate vg_lab /dev/sdb
```

> [!success] Expected Output
> ```text
> Physical volume "/dev/sdb" successfully created.
> Volume group "vg_lab" successfully created.
> ```

### Step 2: Create Logical Volume and Format

```bash
# Create Logical Volume using 100% free space
lvcreate -n lv_data -l +100%FREE vg_lab
# Format with XFS filesystem
mkfs.xfs /dev/mapper/vg_lab-lv_data
```

### Step 3: Mount Persistently

```bash
# Create mount point and mount temporarily
mkdir -p /mnt/labdata
mount /dev/mapper/vg_lab-lv_data /mnt/labdata
# Find UUID
blkid | grep vg_lab-lv_data
# Add to fstab
echo "UUID=YOUR-UUID-HERE /mnt/labdata xfs defaults 0 0" >> /etc/fstab
```

> [!success] Verification Steps
> ```bash
> umount /mnt/labdata
> mount -a
> df -h | grep labdata
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `df -h` | Check disk space usage on all mounted filesystems | `df -h` |
| `du -ah` | Estimate file space usage to find large files | `du -ah /var \| sort -hr \| head -n 10` |
| `lsblk` | List block devices (disks, partitions, LVMs) | `lsblk` |
| `pvcreate` | Create a Physical Volume on a raw disk | `pvcreate /dev/sdb` |
| `vgcreate` | Create a Volume Group | `vgcreate vg_prod /dev/sdb` |
| `lvcreate` | Create a Logical Volume | `lvcreate -n lv_data -L 20G vg_prod` |
| `mount -a` | Test `/etc/fstab` configuration and mount filesystems | `mount -a` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| System crashes on reboot | Misspelled drive label or typo in `/etc/fstab` | Boot into emergency mode, fix `/etc/fstab`. Always run `mount -a` after editing. |
| Disk full but `du` doesn't show it | Running `rm` on active log files | Use `> filename` or `cat /dev/null > filename` to truncate active files, or run `lsof +L1` to find deleted open files. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Web Server Fails Due to Disk Space

> [!example] Ticket
> "The production Apache web server crashed and won't start. System log shows 'No space left on device'."

**L1 Response:** Check overall disk space using `df -h`, find which partition is 100% full, and locate large files using `du -ah`.
**Escalation Trigger:** If it's a critical database file that cannot be simply cleared, or if LVM needs extension.
**L2 Resolution:** Found `access_log` is 45GB. Clear the active log without breaking file descriptors: `cat /dev/null > /var/log/httpd/access_log`. Start Apache and configure weekly logrotate.

### 🎫 Scenario 2: Mounting a New Virtual Disk Dynamically

> [!example] Ticket
> "We added a new 100GB virtual disk. Please assign it to '/var/lib/mysql', as it's running out of space."

**L1 Response:** Check if the new disk is visible using `lsblk`. Verify if `/var/lib/mysql` is on an LVM.
**Escalation Trigger:** Pass to L2 to perform the LVM extension.
**L2 Resolution:** Run `pvcreate /dev/sdb`, extend VG `vgextend rhel /dev/sdb`, extend LV `lvextend -l +100%FREE /dev/rhel/lv_mysql`, and resize filesystem `xfs_growfs /var/lib/mysql`.

---
## 🎤 Interview Questions

> [!question] Q1: Explain the difference between `df` and `du` commands.
> **Answer:** `df` (disk free) displays the total disk space available and used across all mounted filesystems by reading the superblock. `du` (disk usage) estimates the space used by specific files or directories by recursively scanning, helping locate large files.

==**Exam Tip:** Always mention `df` reads the superblock, while `du` scans individual files!==

> [!question] Q2: Why should you use Logical Volume Manager (LVM) instead of standard partitioning like fdisk?
> **Answer:** LVM abstracts physical disks into a flexible storage pool. LVM logical volumes can be resized dynamically (extended or shrunk online) across multiple physical drives without unmounting directories or losing data.

> [!question] Q3: What does the command `mount -a` do, and why is it important to run after editing `/etc/fstab`?
> **Answer:** It reads `/etc/fstab` and mounts everything listed. It validates the syntax. If there's an error, it returns one immediately so you can fix it before the system boots and crashes.

---
## 🔗 Related Notes

- [[01-Foundations/01-Hardware/Storage-Deep-Dive|Storage Deep Dive]] — Outlines the hardware structures (RAID, SSDs) behind Linux volumes.
- [[02-Operating-Systems/04-Linux-RHEL/Permissions|Linux Permissions]] — Focuses on accessing directories and files inside the structure.
- [[02-Operating-Systems/04-Linux-RHEL/Systemctl|Systemctl]] — Explains system service managers that write to logs.
