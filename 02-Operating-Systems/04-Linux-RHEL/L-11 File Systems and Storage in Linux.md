---
tags: [desktop-support, linux, rhel, L2]
aliases: [l-11-file-systems-and-storage-in-linux, l-11]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

# L-11: File Systems and Storage in Linux

> [!abstract] Overview
> This note covers Linux storage engineering. It details filesystem formatting (ext4/xfs), persistent mounting (`/etc/fstab`), Logical Volume Manager (LVM) provisioning/resizing, and NFS mounting.

---

---
## Concept Overview
Think of storage management in Linux like building out commercial real estate:
- A **Raw Hard Drive (`/dev/sdb`)** is raw, undeveloped land.
- **Partitioning (`fdisk`)** is drawing boundary fences on the land to divide it into distinct plots (partitions).
- **Formatting (`mkfs`)** is paving the dirt plot with concrete foundation lines (filesystem tables) so buildings can sit on it.
- **Mounting** is assigning a postal address to a paved plot (e.g., mapping a drive to `/data` so users can enter it).
- **LVM (Logical Volume Manager)** is modular construction: instead of buying fixed plots, you dump all land into a giant common pool (Volume Group), build virtual skyscrapers (Logical Volumes) out of the pool, and if you need more space, you buy another plot of land (Physical Volume), throw it in the pool, and instantly expand your skyscraper floors (extend LV) while people are still working inside.


---

---
## Technical Deep Dive
### 1. Filesystem Type Comparison
- **ext4 (Fourth Extended Filesystem):**
  - Legacy standard Linux filesystem.
  - Supports resizing (shrinking and growing).
  - Uses journaling to prevent data corruption during power loss.
- **XFS:**
  - Modern RHEL standard default filesystem.
  - Highly optimized for parallel I/O and massive files (up to 8 Exabytes).
  - **Limitation:** Can be extended, but **cannot be shrunk/reduced** in size.
- **Btrfs:**
  - Copy-on-Write (CoW) filesystem. Supports subvolumes, snapshotting, and built-in RAID logic.
- **NFS (Network File System):**
  - Protocol used to mount remote directory shares over network links.

### 2. Disk Partitioning: fdisk vs. gdisk
- **`fdisk`:** Partitioning utility for legacy MBR disks (supports up to 4 primary partitions, max 2.2TB).
- **`gdisk`:** Partitioning utility designed for GPT disks (supports up to 128 partitions, sizes up to ZBs).

### 3. Persistent Mounts: Parsing `/etc/fstab`
To mount storage automatically on system boot, add entries to `/etc/fstab`. Each line contains six fields:
```
UUID=3c9b-4a5f-8d9e-12345ef6    /data      xfs     defaults,noatime    0   0
             1                     2        3             4            5   6
```
1. **Device Identifier:** The block device path (e.g., `/dev/sdb1`) or the **UUID (Universally Unique Identifier)**.
   - *Why UUID is preferred:* Device names can change on reboot (e.g., `/dev/sdb` becomes `/dev/sdc` if you plug in a USB drive), breaking mounts. UUID is burned into the filesystem metadata and never changes.
2. **Mount Point:** The target directory path where the storage mounts.
3. **Filesystem Type:** E.g., `xfs`, `ext4`, `nfs`, `swap`.
4. **Mount Options:** Configuration flags (e.g., `defaults`, `ro` for read-only, `noatime` to disable access time updates to save write overhead).
5. **Dump Flag:** `1` enables legacy dump utility backup; `0` disables it (standard).
6. **FSCK Order:** Dictates the order `fsck` runs filesystem checks on boot. `1` is reserved for Root (`/`); `2` for other local disks; `0` disables checks (used for NFS mounts).

### 4. Logical Volume Manager (LVM) Architecture
LVM virtualizes storage using three abstraction layers:
1. **Physical Volumes (PV):** Raw block devices or partitions initialized for LVM (e.g., `/dev/sdb1`).
2. **Volume Groups (VG):** The combined storage pool created by grouping PVs together (e.g., `vg_data`).
3. **Logical Volumes (LV):** The virtual partitions carved out of the VG, formatted with a filesystem and mounted (e.g., `/dev/vg_data/lv_sales`).

```
Physical Disks:    [/dev/sdb1]      [/dev/sdc1]
                       |                |
Physical Volumes:  [  PV 1   ]      [  PV 2   ]
                       \               /
Volume Group:      [      VG: vg_data         ] (Combined Pool)
                       /               \
Logical Volumes:   [ LV: lv_sales ]    [ LV: lv_hr ]
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
### Initializing and Creating LVM
```bash
pvcreate /dev/sdb1 /dev/sdc1                  # Initialize disks as Physical Volumes
vgcreate vg_data /dev/sdb1 /dev/sdc1           # Create Volume Group "vg_data"
lvcreate -L 50G -n lv_sales vg_data           # Create 50GB Logical Volume "lv_sales" from pool
```

### Expanding LVM (Dynamic Resize)
```bash
# Step 1: Add physical space (if VG is full, add a new PV)
pvcreate /dev/sdd1
vgextend vg_data /dev/sdd1

# Step 2: Extend the Logical Volume size by 20GB
lvextend -L +20G /dev/vg_data/lv_sales

# Step 3: Resize the filesystem to utilize the new space online (without unmounting)
resize2fs /dev/vg_data/lv_sales               # Use if formatted with ext4
xfs_growfs /dev/vg_data/lv_sales              # Use if formatted with xfs
```

---
> [!info] Lab Setup Needed
> A VM running Linux with two unallocated virtual disks (`/dev/sdb` and `/dev/sdc`) of 10GB each.

### Step 1: Initialize PVs and VG
1. Log in as root. Identify connected disks:
   ```bash
   lsblk
   ```
2. Create Physical Volumes:
   ```bash
   pvcreate /dev/sdb /dev/sdc
   ```
3. Create Volume Group `vg_store`:
   ```bash
   vgcreate vg_store /dev/sdb /dev/sdc
   ```
4. Verify VG size: `vgs` should show a capacity of $\approx 20\text{ GB}$.

### Step 2: Create, Format, and Persistent-Mount an LV
1. Create a 5GB Logical Volume named `lv_data`:
   ```bash
   lvcreate -L 5G -n lv_data vg_store
   ```
2. Format the LV with the standard RHEL **XFS** filesystem:
   ```bash
   mkfs.xfs /dev/vg_store/lv_data
   ```
3. Create the mount directory:
   ```bash
   mkdir /mnt/data_store
   ```
4. Find the UUID of the new filesystem:
   ```bash
   blkid /dev/vg_store/lv_data
   ```
   Copy the UUID string.
5. Edit the mount configuration file:
   ```bash
   vim /etc/fstab
   ```
6. Append the mounting configuration:
   ```text
   UUID=[Paste_UUID_Here]  /mnt/data_store  xfs  defaults  0  0
   ```
7. Mount all drives immediately based on fstab settings:
   ```bash
   mount -a
   ```
8. Verify mount success: `df -h | grep data_store`.

### Step 3: Extend the Volume Online
1. Extend the logical volume by 3GB:
   ```bash
   lvextend -L +3G /dev/vg_store/lv_data
   ```
2. Grow the XFS filesystem to fill the expanded boundary:
   ```bash
   xfs_growfs /mnt/data_store
   ```
3. Verify new size: `df -h /mnt/data_store` should now show 8GB capacity.

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
*Seedha simple mein: Linux storage format karne ke liye hum xfs ya ext4 filesystem use karte hain. `/etc/fstab` file server boot par disks auto-mount karti hai. LVM ke zariye hum storage space ko dynamically extend aur resize kar sakte hain bina server shutdown kiye.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — Base directory permissions and hierarchy.
- [[02-Operating-Systems/04-Linux-RHEL/L-06 File Permissions and Ownership|L-06 File Permissions and Ownership]] — Configuring read-write rights on mounted shares.
- [[02-Operating-Systems/04-Linux-RHEL/L-12 Network Configuration in Linux|L-12 Network Configuration in Linux]] — NFS firewall network settings.
