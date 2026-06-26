---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-15-lvm-logical-volume-manager, l-15]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# L-15 LVM — Logical Volume Manager

> [!abstract] Overview
> Logical Volume Manager (LVM) is a storage virtualization technology that provides a flexible way to manage disk space. Unlike traditional static partitioning, LVM allows administrators to aggregate multiple physical drives into virtual storage pools (Volume Groups) and allocate dynamic, resizable partitions (Logical Volumes) on the fly.

---

---
## Concept Overview
- **What it is** — LVM is a layer of abstraction between physical storage devices (hard disks) and the operating system's filesystem, enabling online resizing, snapshots, and disk spanning.
- **Why it matters for a support engineer** — Production servers constantly run out of disk space. LVM enables administrators to add new storage drives and expand active partitions (like `/var` or `/home`) with zero downtime.
- **Where you encounter this in real job** — Adding a new virtual disk to an VM running database services and expanding the live volume without unmounting the database filesystem.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Checks disk space using `df -h`, inspects basic LVM components via `pvs`, `vgs`, and `lvs`.
  - **Escalation Trigger:** Escalate to L2 if a logical volume needs to be created, extended, or if disk space is completely exhausted on a critical partition.
  - ****L2 Resolution:**** Initialized new storage drives as PVs, extends Volume Groups, creates/extends Logical Volumes, and resizes XFS or ext4 filesystems.
  - ****L3 Resolution:**** Handles LVM thin provisioning, configures automated system snapshots, performs volume migrations across physical disks, and resolves root partition extension failures.


---

---
## Technical Deep Dive
### 1. LVM Architecture & Components
LVM works on a three-tier hierarchy that pools physical storage:

```
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

1. **Physical Volume (PV)**: The raw physical disk or partition (e.g., `/dev/sdb` or `/dev/sdc1`) initialized for LVM.
2. **Volume Group (VG)**: A virtual pool of storage composed of one or more Physical Volumes.
3. **Logical Volume (LV)**: The actual partitions carved out from the Volume Group. Filesystems are formatted directly on these.
4. **Physical Extent (PE)**: The smallest unit of allocation (usually 4MB) in LVM. Space is allocated in PEs.

### 2. Filesystem Resizing Mechanics
Resizing LVM is a two-step process. First, the Logical Volume boundary is expanded. Second, the filesystem residing inside that volume must be resized to fit the new boundary:
* **XFS Filesystem**: Resized online using `xfs_growfs`. XFS *cannot* be shrunk.
* **Ext4 Filesystem**: Resized using `resize2fs`. Ext4 can be grown online, and can also be shrunk offline.

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
> [!warning] Pre-requisites
> - A Red Hat Enterprise Linux system.
> - An unformatted secondary hard disk attached (e.g., `/dev/sdb`).
> - Sudo/root access.

### Step 1: Create a Physical Volume (PV)
We will initialize the newly attached raw disk for LVM usage.

1. Scan for newly added disks:
```bash
sudo fdisk -l
```
2. Create a Physical Volume on `/dev/sdb`:
```bash
sudo pvcreate /dev/sdb
```
**Expected Output:** `Physical volume "/dev/sdb" successfully created.`

---

### Step 2: Create a Volume Group (VG)
We will create a virtual storage pool named `vg_data` and add our new PV to it.

1. Create the Volume Group:
```bash
sudo vgcreate vg_data /dev/sdb
```
2. Verify the Volume Group configuration:
```bash
sudo vgdisplay vg_data
```
**Expected Output:** `Volume group "vg_data" successfully created` along with metadata displaying total capacity.

---

### Step 3: Create a Logical Volume (LV) and Format Filesystem
We will carve out a 10GB Logical Volume named `lv_app` from our `vg_data` pool and format it with an XFS filesystem.

1. Create a 10GB Logical Volume:
```bash
sudo lvcreate -L 10G -n lv_app vg_data
```
2. Format the new logical volume with XFS:
```bash
sudo mkfs.xfs /dev/vg_data/lv_app
```
3. Create mount point and mount it:
```bash
sudo mkdir /mnt/app
sudo mount /dev/vg_data/lv_app /mnt/app
```
4. Verify standard storage allocation:
```bash
df -hT /mnt/app
```
**Expected Output:** Displays partition of type `xfs` mounted at `/mnt/app` with size 10G.

---

### Step 4: Dynamically Extend the Logical Volume and Filesystem
Now we will extend `/mnt/app` from 10GB to 15GB by adding 5GB on the fly, demonstrating online expansion.

1. Extend the Logical Volume by 5GB:
```bash
sudo lvextend -L +5G /dev/vg_data/lv_app
```
2. Grow the XFS filesystem to fill the extended space:
```bash
sudo xfs_growfs /mnt/app
```
3. Check the new filesystem capacity:
```bash
df -hT /mnt/app
```
**Expected Output:** File system size displays as `15G` without unmounting the storage.

---

---
## Cheat Sheet / Quick Reference
| Command | Description | Example |
|---------|-------------|---------|
| `pvcreate` | Initialize disk for LVM | `sudo pvcreate /dev/sdc` |
| `pvdisplay` | Display information about PVs | `sudo pvdisplay` |
| `vgcreate` | Create a Volume Group | `sudo vgcreate vg_name /dev/sdb` |
| `vgextend` | Add a new PV to an existing VG | `sudo vgextend vg_name /dev/sdc` |
| `lvcreate` | Create a Logical Volume | `sudo lvcreate -L 20G -n lv_name vg_name` |
| `lvextend` | Extend Logical Volume size | `sudo lvextend -r -L +10G /dev/vg/lv` |
| `xfs_growfs` | Resize an XFS filesystem | `sudo xfs_growfs /mount/point` |
| `resize2fs` | Resize an Ext4 filesystem | `sudo resize2fs /dev/vg/lv` |

---

---
## Troubleshooting
| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **`pvcreate` fails with device busy error** | Disk already has an active partition table or is mounted. | Verify disk utilization with `lsblk`. Run `wipefs -a /dev/sdb` to clean disk signature if sure. |
| **Logical Volume extended but file system size is unchanged** | Extended only the LV boundary, forgot to grow filesystem. | Run `xfs_growfs /mount/point` (for XFS) or `resize2fs /dev/vg/lv` (for Ext4). |
| **Cannot reduce Logical Volume size** | Trying to reduce an XFS filesystem. | XFS does not support shrinking. You must back up data, destroy LV, recreate with smaller size, and restore data. |

---

---
## Interview Questions
**Q1: What are the three core layers of LVM storage architecture?**
> A: The three core layers are:
> 1. **Physical Volumes (PV)**: Physical disks or partitions.
> 2. **Volume Groups (VG)**: The virtual storage pool combining PVs.
> 3. **Logical Volumes (LV)**: The dynamic partitions allocated from VGs.

**Q2: What is the difference between expanding an XFS filesystem and an Ext4 filesystem?**
> A: Both can be expanded online. However, XFS is expanded using `xfs_growfs` by targeting the mount point directory. Ext4 is expanded using `resize2fs` by targeting the Logical Volume block device path (`/dev/vg/lv`). Additionally, Ext4 can be shrunk offline, whereas XFS can never be shrunk.

**Q3: How do you add a new hard drive to an existing Volume Group named `vg_sales`?**
> A: First, initialize the disk as a physical volume: `sudo pvcreate /dev/sdc`. Second, extend the volume group by adding the new PV: `sudo vgextend vg_sales /dev/sdc`.

**Q4: What does the `-r` option do when running `lvextend`?**
> A: The `-r` (or `--resizefs`) option tells LVM to automatically resize the underlying filesystem (whether it is Ext4 or XFS) immediately after extending the Logical Volume block boundary. This combines `lvextend` and the filesystem resize tool into a single command.

---

---
## Seedha Simple Mein
*Seedha simple mein: LVM storage manage karne ka ek flexible tareeka hai. Isme hum alag-alag hard disks ko mila kar ek bada storage pool (Volume Group) bana lete hain aur us pool se apni marzi ke size ke partitions (Logical Volumes) banate hain. In logical partitions ka size hum jab chahein live server par extend kar sakte hain.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management]]
- [[02-Operating-Systems/04-Linux-RHEL/L-11 File Systems and Storage in Linux]]
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-11 Storage — Disk Management and RAID]]
