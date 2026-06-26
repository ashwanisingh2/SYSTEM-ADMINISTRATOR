---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-15-lvm-logical-volume-manager, l-15]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX / RHEL**

`#complete` `#intermediate` `#rhcsa`

# L-15: LVM — Logical Volume Manager

> [!abstract] Overview
> Logical Volume Manager (LVM) is a storage virtualization technology that provides a flexible way to manage disk space. Unlike traditional static partitioning, LVM allows administrators to aggregate multiple physical drives into virtual storage pools (Volume Groups) and allocate dynamic, resizable partitions (Logical Volumes) on the fly. *Yeh note cover karta hai LVM ke core concepts, resizing filesystems, aur unke commands. Ek support engineer ke liye yeh bohot zaroori hai kyunki production systems par disk space extend karna ek aam task hai.*

---
## 🧠 Concept Overview

- **What it is** — LVM is a layer of abstraction between physical storage devices (hard disks) and the operating system's filesystem, enabling online resizing, snapshots, and disk spanning.
- **Why it matters** — Production servers constantly run out of disk space. LVM enables administrators to add new storage drives and expand active partitions (like `/var` or `/home`) with zero downtime.
- **Where you see this** — Adding a new virtual disk to a VM running database services and expanding the live volume without unmounting the database filesystem.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Checks disk space using `df -h`, inspects basic LVM components via `pvs`, `vgs`, and `lvs`. *Basics check karte hain, storage kitni free hai woh dekhte hain.* |
| **L2** | Initialized new storage drives as PVs, extends Volume Groups, creates/extends Logical Volumes, and resizes XFS or ext4 filesystems. Escalate karta hai agar complex issues aayen. *Storage provisioning aur disk resizing karte hain.* |
| **L3** | Handles LVM thin provisioning, configures automated system snapshots, performs volume migrations across physical disks, and resolves root partition extension failures. *Architecture aur complex migrations handle karte hain.* |

> [!tip] Seedha Simple Mein
> *LVM storage manage karne ka ek flexible tareeka hai. Isme hum alag-alag hard disks ko mila kar ek bada storage pool (Volume Group) bana lete hain aur us pool se apni marzi ke size ke partitions (Logical Volumes) banate hain. In logical partitions ka size hum jab chahein live server par extend kar sakte hain bina server restart kiye.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **LVM** is like **a water tank system** because...
>
> - **Pipes (Physical Volumes)** supply water to the main tank from various sources. *Alag alag pipelines (hard drives) pani supply karti hain.*
> - **The Main Tank (Volume Group)** holds all the available water in one pool. *Ek bada tank jisme sab paani jama hota hai (Storage Pool).*
> - **Taps/Rooms (Logical Volumes)** are where the water goes. If a room needs a bigger tap (more storage), you just open it wider, as long as there is water left in the main tank! *Agar ek room ko zyada paani chahiye, toh bas uska valve khol do (size extend kar do).*

---
## 🔬 Technical Deep Dive

### 1. LVM Architecture & Components

> [!info] Key Concept
> LVM works on a three-tier hierarchy that pools physical storage: *Yeh 3 layers mein kaam karta hai.*

```text
  +---------------------------------------+
  |              Filesystem               |  <- ext4, xfs, etc.
  +---------------------------------------+
  |         Logical Volumes (LV)          |  <- /dev/vg_data/lv_sales
  +---------------------------------------+
  |          Volume Groups (VG)           |  <- vg_data (virtual pool)
  +---------------------------------------+
  |        Physical Volumes (PV)          |  <- /dev/sdb1, /dev/sdc
  +---------------------------------------+
```

1. **Physical Volume (PV)**: The raw physical disk or partition (e.g., `/dev/sdb` or `/dev/sdc1`) initialized for LVM. *Yeh actual hard disk hoti hai jo LVM use karega.*
2. **Volume Group (VG)**: A virtual pool of storage composed of one or more Physical Volumes. *Yeh poora combined storage ka tank hota hai.*
3. **Logical Volume (LV)**: The actual partitions carved out from the Volume Group. Filesystems are formatted directly on these. *Yeh actual drives hoti hain jo system use karta hai, jaise Windows mein C: ya D: drive.*
4. **Physical Extent (PE)**: The smallest unit of allocation (usually 4MB) in LVM. Space is allocated in PEs. *Yeh storage blocks ka size hota hai (default 4MB).*

### 2. Filesystem Resizing Mechanics

> [!info] Key Concept
> Resizing LVM is a two-step process. First, the Logical Volume boundary is expanded. Second, the filesystem residing inside that volume must be resized to fit the new boundary: *LVM ko resize karna do steps ka process hai: pehle partition size badhao, phir andar ke file system ko us naye size tak phailao.*

- **XFS Filesystem**: Resized online using `xfs_growfs`.
- **Ext4 Filesystem**: Resized using `resize2fs`. Ext4 can be grown online, and can also be shrunk offline.

> [!danger] Common Mistake
> Trying to reduce an XFS filesystem. XFS does not support shrinking. You must back up data, destroy LV, recreate with smaller size, and restore data. *XFS file system ko kabhi chhota (shrink) nahi kiya jaa sakta! Aisa karna data corrupt kar dega.*

### 3. Enterprise RHEL Service & Network Configurations (Bonus)

> [!info] Key Concept
> Miscellaneous standard system admin configurations often handled alongside storage tasks. *Baaki important cheezein jo saath me manage karni padti hain.*

#### Custom Systemd Service Creation
Create `/etc/systemd/system/myapp.service`:
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
# Enable and start the service
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
```

#### Log Rotation & Rsyslog Configuration
Configure log rotation rule in `/etc/logrotate.d/myapp`:
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

Define Rsyslog rule in `/etc/rsyslog.d/50-myapp.conf`:
```text
if $programname == 'myapp' then /var/log/myapp/syslog.log
& stop
```

Restart Rsyslog service:
```bash
# Restart the logging daemon
sudo systemctl restart rsyslog
```

#### Network Bonding & Teaming (LACP Link Aggregation)
Create a network team interface config file `/etc/sysconfig/network-scripts/ifcfg-team0`:
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
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Red Hat Enterprise Linux system.
> - An unformatted secondary hard disk attached (e.g., `/dev/sdb`).
> - Sudo/root access.

### Step 1: Create a Physical Volume (PV)

Scan for newly added disks:
```bash
# View available partitions
sudo fdisk -l
```

Create a Physical Volume on `/dev/sdb`:
```bash
# Initialize the disk for LVM
sudo pvcreate /dev/sdb
```

> [!success] Expected Output
> ```
> Physical volume "/dev/sdb" successfully created.
> ```

### Step 2: Create a Volume Group (VG)

Create the Volume Group named `vg_data`:
```bash
# Create a virtual storage pool named vg_data
sudo vgcreate vg_data /dev/sdb
```

Verify the Volume Group configuration:
```bash
# Check volume group details
sudo vgdisplay vg_data
```

> [!success] Expected Output
> ```
> Volume group "vg_data" successfully created (along with metadata displaying total capacity).
> ```

### Step 3: Create a Logical Volume (LV) and Format Filesystem

Create a 10GB Logical Volume:
```bash
# Carve out a 10GB LV named lv_app
sudo lvcreate -L 10G -n lv_app vg_data
```

Format and mount the logical volume:
```bash
# Format with XFS and mount
sudo mkfs.xfs /dev/vg_data/lv_app
sudo mkdir /mnt/app
sudo mount /dev/vg_data/lv_app /mnt/app
```

Verify storage allocation:
```bash
# Check filesystem
df -hT /mnt/app
```

> [!success] Expected Output
> ```
> Displays partition of type xfs mounted at /mnt/app with size 10G.
> ```

### Step 4: Dynamically Extend the Logical Volume and Filesystem

Extend the Logical Volume by 5GB:
```bash
# Expand the LV boundary by 5GB
sudo lvextend -L +5G /dev/vg_data/lv_app
```

Grow the XFS filesystem to fill the extended space:
```bash
# Grow the filesystem online
sudo xfs_growfs /mnt/app
```

Check the new filesystem capacity:
```bash
# Verify new size
df -hT /mnt/app
```

> [!success] Expected Output
> ```
> File system size displays as 15G without unmounting the storage.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `pvcreate` | Initialize disk for LVM. *Disk ko LVM format banata hai.* | `sudo pvcreate /dev/sdc` |
| `pvdisplay` | Display information about PVs. *PV ki details dekhta hai.* | `sudo pvdisplay` |
| `vgcreate` | Create a Volume Group. *Naya Volume Group banata hai.* | `sudo vgcreate vg_name /dev/sdb` |
| `vgextend` | Add a new PV to an existing VG. *Storage tank me nayi disk add karta hai.* | `sudo vgextend vg_name /dev/sdc` |
| `lvcreate` | Create a Logical Volume. *VG me se actual partition nikalta hai.* | `sudo lvcreate -L 20G -n lv_name vg_name` |
| `lvextend` | Extend Logical Volume size. *LV ka size badhata hai.* | `sudo lvextend -r -L +10G /dev/vg/lv` |
| `xfs_growfs` | Resize an XFS filesystem. *XFS partition ko badhata hai.* | `sudo xfs_growfs /mount/point` |
| `resize2fs` | Resize an Ext4 filesystem. *Ext4 partition ko resize karta hai.* | `sudo resize2fs /dev/vg/lv` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **`pvcreate` fails with device busy error** | Disk already has an active partition table or is mounted. *Disk par pehle se kuch mount hai.* | Verify disk utilization with `lsblk`. Run `wipefs -a /dev/sdb` to clean disk signature if sure. |
| **Logical Volume extended but file system size is unchanged** | Extended only the LV boundary, forgot to grow filesystem. *Aapne sirf fs boundary extend ki hai, fs nahi.* | Run `xfs_growfs /mount/point` (for XFS) or `resize2fs /dev/vg/lv` (for Ext4). |
| **Cannot reduce Logical Volume size** | Trying to reduce an XFS filesystem. *XFS kabhi shrink nai hota.* | XFS does not support shrinking. You must back up data, destroy LV, recreate with smaller size, and restore data. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Disk Space Exhausted on Database Server

> [!example] Ticket
> "Urgent: The `/var/lib/mysql` partition is 99% full on PROD-DB-01. Database services might crash soon. Please add space."

**L1 Response:** Check disk usage using `df -h` and verify LVM structures using `vgs` and `lvs`. *Pehle size aur space confirm karte hain.*
**Escalation Trigger:** If volume group has free space, expand. If not, escalate to L2 to provision and attach new physical disk. *Agar VG mein space nahi hai toh L2 ko bhejte hain.*
**L2 Resolution:** Initialize new disk with `pvcreate`, add to VG with `vgextend`, and dynamically expand the partition with `lvextend -r`.

### 🎫 Scenario 2: Web Server Log Partition Getting Full

> [!example] Ticket
> "Alert: Critical disk space on `/var/log` for WEBSVR-02. Current usage is at 95%."

**L1 Response:** Access server and run `df -hT` and `lvs` to check free extents in the VG. Review logs temporarily to clear up space or compress them. *Pehle try karte hain thoda free space laane ka old logs remove karke.*
**Escalation Trigger:** If the VG `vg_os` has space available, proceed with expanding. Otherwise escalate.
**L2 Resolution:** Allocate space by expanding the LV: `lvextend -L +5G /dev/vg_os/lv_log` followed by `xfs_growfs /var/log` to apply the resize online. *XFS hai toh xfs_growfs chalayenge.*

### 🎫 Scenario 3: Request to Reduce Volume Size

> [!example] Ticket
> "Request: Please reduce the size of the `/backup` partition on APP-SVR-01 from 100GB to 50GB as we need space for another partition."

**L1 Response:** Check the filesystem type. If it's XFS, inform the user that shrinking is not supported dynamically. *Agar XFS hai, direct mana karenge bina data backup ke shrink nahi hoga.*
**Escalation Trigger:** Pass to L2 to schedule a downtime for backup and recreation if they approve.
**L2 Resolution:** (Assuming ext4 and allowed to shrink) Unmount the partition, run `e2fsck -f /dev/vg_data/lv_backup`, shrink the filesystem with `resize2fs /dev/vg_data/lv_backup 50G`, and then reduce the LV with `lvreduce -L 50G /dev/vg_data/lv_backup`. Finally remount.

---
## 🎤 Interview Questions

> [!question] Q1: What are the three core layers of LVM storage architecture?
> **Answer:** The three core layers are: 1. Physical Volumes (PV), 2. Volume Groups (VG), 3. Logical Volumes (LV). *In teenon se milke hi storage pool banta hai.*

> [!question] Q2: What is the difference between expanding an XFS filesystem and an Ext4 filesystem?
> **Answer:** Both can be expanded online. However, XFS is expanded using `xfs_growfs` by targeting the mount point directory. Ext4 is expanded using `resize2fs` by targeting the Logical Volume block device path (`/dev/vg/lv`). Additionally, Ext4 can be shrunk offline, whereas XFS can never be shrunk. *XFS shrink nahi ho sakta.*

> [!question] Q3: How do you add a new hard drive to an existing Volume Group named `vg_sales`?
> **Answer:** First, initialize the disk as a physical volume: `sudo pvcreate /dev/sdc`. Second, extend the volume group by adding the new PV: `sudo vgextend vg_sales /dev/sdc`.

> [!question] Q4: What does the `-r` option do when running `lvextend`?
> **Answer:** The `-r` (or `--resizefs`) option tells LVM to automatically resize the underlying filesystem (whether it is Ext4 or XFS) immediately after extending the Logical Volume block boundary. *Is flag se baar-baar alag command nahi chalani padti.*

==**Exam Tip:** Remember that XFS filesystems can be grown but never shrunk! Always verify filesystem type before attempting a resize operation. *Ye bahut important sawaal hai RHCSA me!*==

---
## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]]
- [[02-Operating-Systems/04-Linux-RHEL/L-11 File Systems and Storage in Linux|L-11 File Systems and Storage in Linux]]
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-11 Storage — Disk Management and RAID|WS-11 Storage — Disk Management and RAID]]
