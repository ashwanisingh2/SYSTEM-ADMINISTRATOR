---
tags: [desktop-support, career-roadmap, certification, linux, rhel, rhcsa, sysadmin, L1, L2, L3]
aliases: [rhcsa-prep, redhat-roadmap]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa-ex200
---

# RHCSA (EX200) Certification Study Roadmap

---

## Concept Overview
- **What it is**: A comprehensive study guide, configuration playbook, and exam execution roadmap for the Red Hat Certified System Administrator (RHCSA EX200) exam.
- **Why it matters**: Linux powers a significant portion of enterprise servers, databases, and container hosts. The RHCSA is a 100% practical, hands-on exam that verifies an engineer's ability to configure local storage, manage security parameters (SELinux/Firewalls), manage system services, and troubleshoot boot failures.
- **Where you encounter this in real job**: Managing file servers, running databases, configuring web hosts, automating shell tasks, and maintaining system security baselines.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Monitors Linux server availability, checks CPU/disk usage (`top`, `df`), restarts stopped services, and resets basic user passwords.
  - **L2**: Manages user accounts and group memberships, configures file permissions and ACLs, configures SSH keys, and manages package repositories.
  - **L3**: Resets broken GRUB bootloaders, configures Logical Volume Manager (LVM) partitions, configures SELinux policy contexts, and writes complex bash automation scripts.

---

## Technical Deep Dive

### RHCSA Exam Domain Objectives
The RHCSA EX200 exam is a 3-hour practical exam assessing the following core domains:
```
  [1. Essential Tools (chmod, chown, tar)] ---> [2. Operating Systems (reboot, boot targets)]
                                                       |
                                                       v
  [3. Configure Storage (partitions, LVM)] ---> [4. Create Filesystems (ext4, xfs, fstab)]
                                                       |
                                                       v
                 [5. Manage Security (SELinux, firewalld, permissions)]
```

### Logical Volume Manager (LVM) Architecture
LVM provides flexible storage allocation, allowing administrators to resize disk partitions online without shutting down services:
```
[Physical Disks: /dev/sdb, /dev/sdc] ---> (Physical Volumes - PV)
                                                |
                                                v
                                  (Volume Groups - VG: vg_data)
                                                |
                                                v
                          [Logical Volumes - LV: lv_store, lv_backup]
                                                |
                                                v (Format & Mount)
                                       [/data, /backup]
```

---

## Commands & Syntax

### RHEL CLI: Essential Admin Commands
```bash
# 1. Create a Physical Volume, Volume Group, and Logical Volume
pvcreate /dev/sdb1
vgcreate vg_data /dev/sdb1
lvcreate -n lv_store -L 5G vg_data

# 2. Format the Logical Volume with XFS filesystem and mount it persistently
mkfs.xfs /dev/vg_data/lv_store
echo "/dev/vg_data/lv_store /data xfs defaults 0 0" >> /etc/fstab
mount -a

# 3. Configure Firewall rules to permit HTTP service persistently
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

### Configuration File Paths
- `/etc/fstab`: Static information about filesystems mounted persistently at boot.
- `/etc/selinux/config`: System-wide SELinux state parameters (Enforcing, Permissive, Disabled).
- `/etc/sysconfig/network-scripts/`: Legacy network configuration files (replaced by NetworkManager `/etc/NetworkManager/system-connections/` in RHEL 9).

### Service Logs
```bash
# Read systemd journal log output filtered for SSH service errors
journalctl -u sshd.service -n 50 --no-pager
```

---

## Real-World Scenarios

### Scenario 1: System Stuck in Emergency Mode due to `/etc/fstab` Typo
**User Complaint**: After adding a new virtual disk and editing storage parameters, a reboot causes the Linux server to hang, booting into "Emergency Mode" and prompting for the root password.
**Your First 3 Checks**:
1. Input the root password to access the emergency shell.
2. Read the system logs using `journalctl -xb` to identify the mounting failure.
3. Check the syntax of the `/etc/fstab` configuration file.
**Diagnosis Steps**:
1. Open `/etc/fstab` using `vi` or `cat`.
2. Locate the line added for the new disk:
   `/dev/sdb1  /backup  xfss  defaults  0  0`
3. Notice the spelling mistake in the filesystem type: `xfss` instead of `xfs`.
**Root Cause**: The system failed to parse the invalid filesystem type `xfss` during the boot mount phase, causing systemd to halt the boot process and drop into Emergency Mode.
**Fix**:
1. Mount the root filesystem in read-write mode (it is read-only by default in emergency mode):
   ```bash
   mount -o remount,rw /
   ```
2. Open `/etc/fstab` using `vi` and correct `xfss` to `xfs`. Save the file.
3. Verify the mount configurations:
   ```bash
   mount -a
   ```
4. Confirm no errors are returned, and type `reboot` to boot the system normally.
**Prevention**: Never reboot a server after editing `/etc/fstab` without running the `mount -a` command first. If `mount -a` fails, fix the errors before rebooting.

### Scenario 2: Root Password Recovery (`rd.break`)
**User Complaint**: A database host has been offline for months. The local administrator has resigned, and the root password is lost. You must recover local console access.
**Your First 3 Checks**:
1. Connect to the host using a physical console or hypervisor terminal window.
2. Prepare to reboot the system to access the GRUB boot menu.
3. Interrupt the kernel boot sequence.
**Diagnosis Steps**:
1. Reboot the system. At the GRUB menu, select the default kernel and press the **`e`** key to edit the boot parameters.
2. Scroll to the line starting with `linux` (or `linux16`).
3. Append **`rd.break`** to the end of this line. This parameter instructs systemd to pause the boot sequence before handing control to the root filesystem.
4. Press **`Ctrl + X`** to boot with these temporary parameters.
**Root Cause**: The root password is forgotten, and access must be regained from the emergency console before the target filesystems mount in normal secure mode.
**Fix**:
1. Remount the `/sysroot` filesystem with write permissions:
   ```bash
   switch_root:/# mount -o remount,rw /sysroot
   ```
2. Enter a chroot environment using `/sysroot` as the root directory:
   ```bash
   switch_root:/# chroot /sysroot
   ```
3. Update the root password:
   ```bash
   sh-5.1# passwd root
   ```
4. Create the required file to trigger an SELinux relabeling task on the next boot (critical when changing passwords outside normal boot):
   ```bash
   sh-5.1# touch /.autorelabel
   ```
5. Exit the chroot, exit the shell, and allow the system to boot.
   ```bash
   sh-5.1# exit
   switch_root:/# exit
   ```
**Prevention**: Store server local administrator credentials in a secure, central password manager with delegated access permissions.

---

## Critical Points

> [!danger] Never Do This
> Do not disable SELinux (`SELINUX=disabled` in `/etc/selinux/config`) on a production server. Disabling SELinux requires a system reboot and removes mandatory access controls, exposing the server to privilege escalation exploits. If troubleshooting permission issues, change the mode to `Permissive` instead.

> [!warning] Common Trap
> Editing configuration files in `/etc` without making a backup copy first (e.g. `cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak`). If a syntax error is introduced, restoring from a backup copy is much faster than diagnosing broken configurations.

> [!tip] Senior Engineer Tip
> Use the `df -h` command to check disk space usage, and `du -sh /*` to identify which directories contain the largest files. This helps isolate rogue log files filling up system directories.

> [!success] Verification Steps
> To verify a persistent mount:
> 1. Run `umount /target_directory`.
> 2. Run `mount -a`.
> 3. Run `df -h` and ensure the filesystem mounts back to the target directory.

> [!question] Interview Alert
> "What is the difference between soft links and hard links?"
> - **Answer**: A soft link (symbolic link) is a pointer file that references the path of another file; deleting the source file breaks the link. A hard link is a direct reference to the file's data block on the storage disk (inode); deleting the source file does not affect the hard link, as the data remains accessible until all links to it are deleted.

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| SELinux blocking services | File contexts mismatch after moving files | Run `restorecon -Rv /path` to apply correct SELinux labels to files and directories. |
| Lost partition mappings | Mounting by device name (e.g., `/dev/sdb1`) which can change on boot | Use the partition's unique UUID (`blkid`) to identify drives in `/etc/fstab`. |
| Missing network configurations | Forgetting to set interfaces to auto-start | Edit the interface parameters using `nmcli connection modify [Name] connection.autoconnect yes`. |

---

## Lab Exercise

**Objective**: Configure a Logical Volume, format it with XFS, and mount it persistently using UUID.
**Time Required**: 25 minutes
**Environment Needed**: RHEL/CentOS VM with a second physical disk (`/dev/sdb`) added.

**Steps**:
1. Log in to the terminal as root.
2. Initialize the physical disk partition `/dev/sdb1` using `gdisk /dev/sdb`.
3. Create the Physical Volume and Volume Group:
   ```bash
   pvcreate /dev/sdb1
   vgcreate vg_data /dev/sdb1
   ```
4. Deploy the Logical Volume named `lv_data` with a size of 2GB:
   ```bash
   lvcreate -n lv_data -L 2G vg_data
   ```
5. Format the new volume with XFS filesystem:
   ```bash
   mkfs.xfs /dev/vg_data/lv_data
   ```
6. Create the target mount directory:
   ```bash
   mkdir /mnt/data
   ```
7. Find the UUID of the logical volume:
   ```bash
   blkid /dev/vg_data/lv_data
   ```
   - Copy the UUID string.
8. Edit `/etc/fstab` and append the mount configuration:
   `UUID=your-uuid-string /mnt/data xfs defaults 0 0`
9. Test the configuration:
   ```bash
   mount -a
   df -h /mnt/data
   ```

**Pass Criteria**: The logical volume mounts successfully, does not show errors when running `mount -a`, and displays in `df -h`.

---

## Interview Questions & Answers

### L1 Level

#### Q1: How do you search for a specific word inside a text file using the command line?
**A**: I use the `grep` command. For example, to search for the word "Failed" in the log file `/var/log/secure`, I run the command:
`grep "Failed" /var/log/secure`
I can append `-i` to make the search case-insensitive, or `-n` to show matching line numbers.

#### Q2: How do you check if a service (like SSH) is running on a system?
**A**: I run the command:
`systemctl status sshd`
This displays if the service is active, running, stopped, or disabled, and shows the latest log entries.

---

### L2 Level

#### Q3: Explain the difference between user permissions `chmod 755` and `chmod 644`.
**A**:
- `755` (rwxr-xr-x) grants the owner read, write, and execute permissions, and grants group members and others read and execute permissions only. This is used for directories and executable scripts.
- `644` (rw-r--r--) grants the owner read and write permissions, and grants group members and others read-only permissions. This is used for standard text and configuration files.

#### Q4: How do you verify network connectivity and trace routing paths from a Linux host?
**A**: I use the `ping` command to test basic connection response. To trace routing paths and identify gateway hops, I run the `traceroute` (or `tracepath`) command. I can also use `ss -tulpn` to check active listening ports on the local host.

---

### L3 Level

#### Q5: Explain how you would troubleshoot an application service that crashes on startup, returning permission denied errors.
**A**:
- **Situation**: An Apache web server failed to start after changing its document root directory, throwing permission denied logs even though permissions were set to `755`.
- **Task**: Identify the security control blocking service access to the new directory.
- **Action**: I ran `tail -n 100 /var/log/messages` and noticed SELinux security alerts. I checked the directory context using `ls -dZ /new_web_root` and confirmed it was labeled `default_t` instead of `httpd_sys_content_t`. I ran `semanage fcontext -a -t httpd_sys_content_t "/new_web_root(/.*)?"` and applied the context using `restorecon -R /new_web_root`.
- **Result**: Successfully updated file contexts, permitting Apache to start normally and restoring web access without disabling SELinux.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **Filesystems**: `/etc/fstab` controls persistent mounts. Run `mount -a` to test changes before rebooting.
> **Boot Recovery**: Append `rd.break` to the kernel boot line in GRUB, remount `/sysroot` in write mode, and reset root passwords.
> **LVM Flow**: Physical Disks -> PV -> VG -> LV -> Filesystem -> Mount.
> **Security**: Manage firewall zones using `firewall-cmd` and maintain SELinux contexts using `restorecon`.

---

## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/Filesystem|Linux Filesystem]]
- [[02-Operating-Systems/04-Linux-RHEL/Permissions|Linux Permissions]]
- [[02-Operating-Systems/04-Linux-RHEL/Systemctl|Systemctl Services]]

---

## Tags
#desktop-support #linux-administration #rhcsa #redhat-linux #lvm-storage #firewalld #selinux-security #certification-roadmap #L2 #L3

