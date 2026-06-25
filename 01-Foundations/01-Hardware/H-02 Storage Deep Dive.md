---
tags: [sysadmin, hardware, storage, raid]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# H-02: Storage Deep Dive — HDD, SSD, NVMe, RAID

> [!abstract] Overview
> This note covers storage media architecture (HDD, SATA SSD, NVMe SSD), M.2 form factors, partition tables, disk health analysis (SMART), and RAID configurations. Mastery of these concepts is essential for server provisioning, data redundancy, and resolving storage performance issues.

---
## Concept
Think of computer storage as the files and cabinets in an office. 
- **HDD** is like a large filing cabinet located in the basement. It holds massive amounts of files cheap, but a mechanical runner (the read/write head) must physically walk down the aisle, locate the right drawer, pull the paper out, and carry it back.
- **SATA SSD** is a filing cabinet right next to your desk. There are no moving parts; you open a electronic folder, and there it is. However, the files are passed to you through a narrow chute (the SATA bus).
- **NVMe SSD** is having the files projected onto your desk via a multi-lane high-speed pneumatic tube system (PCIe lanes) straight from the core archives.
- **RAID** is like having duplicate copies of all folders spread across multiple drawers so that if a cabinet catches fire, you can recreate the missing documents instantly from the surviving files.

*Seedha simple mein: HDD mechanical disk hoti hai jo ghoomti hai aur dheemi hoti hai. SSD chips par chalti hai aur bohot fast hoti hai. NVMe sabse tez hoti hai kyunki yeh PCIe slots par directly CPU se baat karti hai. RAID alag-alag drives ko jodkar speed aur redundancy badhane ka tareeqa hai.*

---
## Technical Deep Dive

### 1. HDD Internals & Mechanical Bottlenecks
Hard Disk Drives (HDDs) store data magnetically. Key mechanical components:
- **Platters:** Aluminum or glass discs coated with a magnetic layer. Data is stored in concentric circles called **tracks**, which are divided into **sectors** (typically 512 bytes or 4KB/Advanced Format).
- **Read/Write Head:** Electromagnetic tip attached to an actuator arm that hovers nanometers above the spinning platter.
- **RPM (Rotations Per Minute):** Common speeds are 5400 RPM (laptops/consumer storage), 7200 RPM (desktops/entry NAS), and 10,000 to 15,000 RPM (legacy enterprise/SAS drives). 
  - **Latency:** Higher RPM decreases rotational latency (time taken for the required sector to spin under the head).
  - **Seek Time:** The time it takes for the actuator arm to move to the correct track.

### 2. SSD vs. HDD Comparison
Solid State Drives (SSDs) use NAND flash memory (volatile charge trap or floating gate transistors) to store data electrically.

| Attribute | HDD (Hard Disk Drive) | SSD (Solid State Drive) |
|---|---|---|
| **Mechanism** | Mechanical (Spinning Platters, Heads) | Semiconductor (NAND Flash Memory) |
| **Random I/O (IOPS)**| ~75 - 200 IOPS | 50,000 - 1,000,000+ IOPS |
| **Sequential Read** | ~100 - 250 MB/s | ~500 - 7,500+ MB/s |
| **Access Latency** | ~5 - 15 ms | ~0.02 - 0.1 ms |
| **Lifespan** | Subject to mechanical wear/shock | Limited write endurance (measured in TBW) |
| **Best Use Case** | Cold data archives, mass backups | OS drives, databases, application virtualization |

### 3. NVMe vs. SATA SSD
- **SATA (Serial ATA):** Uses the legacy AHCI (Advanced Host Controller Interface) protocol designed for spinning disks. SATA III caps out at a theoretical speed of 6 Gbps (~550 - 600 MB/s real-world limit). Queue depth is limited to 32 commands in 1 queue.
- **NVMe (Non-Volatile Memory Express):** A protocol designed specifically for flash storage. Runs directly over PCIe lanes. Standard NVMe SSDs use 4 PCIe lanes (x4).
  - **Speed:** Gen 3 NVMe runs up to ~3,500 MB/s; Gen 4 up to ~7,500 MB/s; Gen 5 up to ~14,000 MB/s.
  - **Concurrency:** Supports up to 64,000 queues, with 64,000 commands per queue.

### 4. M.2 Form Factor & Keying
M.2 is a physical form factor (formerly NGFF). An M.2 drive can be SATA-based or PCIe (NVMe)-based.
- **Sizes:** Described by width and length (in mm). E.g., **2280** means 22mm wide and 80mm long. Other sizes include 2242, 2260, and 22110 (common in enterprise servers with power loss protection capacitors).
- **Physical Keys (Notches):**
  - **B-Key:** Supports SATA or PCIe x2. (Deprecated for NVMe).
  - **M-Key:** Supports PCIe x4 (NVMe) or SATA.
  - **B+M Key:** Has both notches; compatible with either slot, limited to PCIe x2 speeds or SATA speed.

```
M.2 M-Key Notch (Right Side): [=======   ===]
M.2 B-Key Notch (Left Side):  [===   =======]
M.2 B+M Key (Dual Notch):     [===   ===   ===]
```

### 5. RAID Levels (Redundant Array of Independent Disks)
RAID combines multiple physical hard drives into a single logical volume for performance, redundancy, or both.

| RAID Level | Description | Min Drives | Space Efficiency | Fault Tolerance | Read/Write Speed |
|---|---|---|---|---|---|
| **RAID 0** | Striping (No parity) | 2 | 100% | 0 Drives (Any failure = data loss) | Fast Read, Fast Write |
| **RAID 1** | Mirroring | 2 | 50% | 1 Drive | Fast Read, Normal Write |
| **RAID 5** | Striping + Distributed Parity | 3 | $(N-1)/N$ | 1 Drive | Fast Read, Slow Write (Parity overhead) |
| **RAID 10** | Striped Mirrors (RAID 1+0) | 4 | 50% | Up to 2 Drives (No two in same mirror) | Very Fast Read, Fast Write |

### 6. Disk Health Monitoring & SMART Attributes
**S.M.A.R.T. (Self-Monitoring, Analysis, and Reporting Technology)** allows drives to monitor health indicators. Tools like CrystalDiskInfo (Windows) or `smartctl` (Linux) check:
- **Reallocated Sectors Count:** Count of damaged sectors that have been remapped to spare sectors. Non-zero indicates drive failure is imminent.
- **Power-On Hours:** Total hours the drive has been powered.
- **Wear Leveling Count (SSDs):** Percentage of remaining NAND write endurance cycles.
- **Current Pending Sector Count:** Unstable sectors waiting to be remapped. Indicates physical degradation.

### 7. Partition Schemes: MBR vs. GPT
- **MBR (Master Boot Record):**
  - Legacy layout located at the very first sector of the drive.
  - Max disk size: 2.2 TB.
  - Max partitions: 4 primary partitions (or 3 primary + 1 extended containing logical drives).
- **GPT (GUID Partition Table):**
  - Modern layout using globally unique identifiers.
  - Max disk size: 9.4 ZB.
  - Max partitions: 128 primary partitions (Windows standard).
  - Features redundancy: stores a backup partition table at the end of the disk.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows 10/11 or Windows Server environment with at least two unallocated disks (or virtual disks attached to a VM).

### Step 1: Open Disk Management
1. Press `Win + X` and select **Disk Management** (or run `diskmgmt.msc`).
2. If new disks are detected, you will be prompted to initialize them. Choose **GPT** and click OK.

### Step 2: Create a Mirrored Volume (RAID 1)
1. Locate the two unallocated disk blocks in the lower panel (e.g., Disk 1 and Disk 2).
2. Right-click on **Disk 1** (unallocated space) and select **New Mirrored Volume**.
3. In the wizard, click **Next**.
4. Select the second disk (e.g., **Disk 2**) from the left list and click **Add**.
5. Set the volume size to the maximum available and click **Next**.

### Step 3: Assign Drive Letter and Format
1. Choose an available drive letter (e.g., `R:`). Click **Next**.
2. Select File System: **NTFS**.
3. Allocation Unit Size: **Default**.
4. Volume Label: `RAID1_DATA`.
5. Check **Perform a quick format**. Click **Next**, then **Finish**.
6. A warning prompt will state that basic disks will be converted to **Dynamic Disks**. Click **Yes**.

### Step 4: Verify Synchronization
1. The drives will turn a dark red/brown color (indicating dynamic volumes) and state "Resynching...".
2. Once complete, the status will show **Healthy**. Open File Explorer and verify that drive `R:` behaves as a single storage volume.

---
## Commands Reference
```powershell
# Windows PowerShell
Get-PhysicalDisk                                              # List all physical disks, status, and media type
Get-PhysicalDisk | Select-Object DeviceId, FriendlyName, OperationalStatus, HealthStatus, Size
Get-Volume                                                    # Show all logical volumes and health
Initialize-Disk -Number 1 -PartitionStyle GPT                 # Initialize Disk 1 using GPT partition style
New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel "Data"

# Linux Bash
lsblk                                                         # List block devices (disks and partitions)
sudo fdisk -l                                                 # List partition tables on all disks
sudo smartctl -i /dev/sda                                     # Get SMART capabilities for /dev/sda
sudo smartctl -A /dev/sda                                     # View SMART attributes for /dev/sda
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** A server alerts that a RAID 5 array has degraded. The server remains online, but disk performance is slow.
- **Root Cause:** A single physical drive in the RAID 5 array has failed. The system is reading missing data by calculating XOR parity on the fly across the remaining healthy drives, causing high CPU and I/O overhead.
- **Fix:** 
  1. Identify the failed drive slot using the RAID controller GUI (e.g., Dell iDRAC, HP ILO) or `smartctl`. Locate the physical blinking orange amber light.
  2. Hot-swap the failed drive with an identical model of equal or greater capacity.
  3. Access the controller utility and verify that the rebuild process starts automatically.
  4. Monitor rebuild status. Do not reboot or stress the server during rebuild to avoid a secondary drive failure which would destroy the array.

**Scenario 2:**
- **Problem:** An operating system installation media fails to see a newly installed NVMe M.2 drive, or it appears as offline and read-only.
- **Root Cause:** BIOS SATA mode is set to RAID/Intel RST (Rapid Storage Technology) instead of AHCI, or the partition contains a corrupted raw Linux/Mac layout.
- **Fix:**
  1. Boot into BIOS/UEFI. Locate SATA/Storage configuration. Convert storage mode from RAID/RST to **AHCI**.
  2. If the drive is still not writable, boot using installation media, press `Shift + F10` to open Command Prompt, and run:
     ```cmd
     diskpart
     list disk
     select disk [number of NVMe]
     clean
     convert gpt
     exit
     ```
  3. Refresh the installer; the drive will now show as clean unallocated space.

---
## Common Mistakes
> [!warning] Avoid These
> **Using RAID as a backup solution:** Thinking that because a server has RAID 1 or RAID 5, data is safe from corruption, deletion, or malware. RAID only provides hardware redundancy. If a user deletes a file, or ransomware encrypts it, the action is instantly mirrored across all drives.
> **Correct approach:** Maintain a dedicated external backup system (e.g., 3-2-1 backup rule) completely separate from the production RAID storage.

---
## Pro Tips
> [!tip] Field Experience
> When configuring RAID 5 or 6, always purchase replacement disks from different batches or vendors. Disks from the same production batch have highly correlated lifespans. If one drive fails and the rebuild process puts intense write stress on the remaining identical-age disks, a second drive is highly likely to fail during the rebuild, causing catastrophic data loss.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | NVMe vs SATA | NVMe runs over PCIe lanes utilizing high-speed queues; SATA is bottlenecked at 600MB/s by the AHCI protocol. |
| 2 | RAID 0 | Striping. Boosts performance but has zero redundancy; one drive failure wipes the entire volume. |
| 3 | RAID 10 | Nested RAID. Combines mirroring (RAID 1) and striping (RAID 0), offering high performance and high safety. |
| 4 | SMART Attributes| Self-monitoring markers; watch "Reallocated Sectors Count" closely to preempt mechanical disk failure. |
| 5 | GPT vs MBR | MBR is legacy, limited to 2.2TB and 4 partitions; GPT is modern, support sizes up to ZBs and 128 partitions. |

---
## Interview Q&A

**Q1: What is the write penalty of a RAID 5 array, and how does it affect write performance?**
A: The write penalty of RAID 5 is 4. When a write occurs on a RAID 5 array, the controller must perform 4 disk operations: 1) Read the old data, 2) Read the old parity, 3) Write the new data, and 4) Write the new parity. This calculation ($Parity = Data \oplus Parity$) means random writes on a RAID 5 array are significantly slower than writes on a RAID 10 array, which only has a write penalty of 2 (writing the data and its single mirror clone).

**Q2: A developer demands an M.2 SSD for their workstation. You have a SATA M.2 and an NVMe M.2 in stock. Explain how you would decide which one to install.**
A: 
- **Situation:** A workstation requires an M.2 upgrade, and both SATA and NVMe M.2 drives are available.
- **Task:** Analyze the workstation hardware motherboard capabilities and user requirements to select the correct drive.
- **Action:** I would first inspect the workstation's motherboard specifications. I need to check if the M.2 slot supports PCIe NVMe, SATA only, or both. If the motherboard supports NVMe, I would evaluate the developer's workload. If they are compiling code, running local Docker containers, or working with databases, they will benefit from NVMe's high IOPS and read/write speeds.
- **Result:** If the slot supports NVMe, I install the NVMe M.2 because it uses the high-performance PCIe bus. If the slot is legacy SATA-only, I install the SATA M.2. Installing an NVMe SSD into a SATA-only M.2 slot results in the drive not being detected at all.

**Q3: Explain the difference between TBW and DWPD when measuring SSD lifespan.**
A: Both measure SSD write endurance. **TBW (Terabytes Written)** is the total cumulative amount of data (in terabytes) that a drive can write before its NAND flash blocks degrade and can no longer reliably store data. **DWPD (Drive Writes Per Day)** measures how many times you can write the equivalent of the drive's total capacity every single day of its warranty period (usually 5 years). The conversion is: $DWPD = \frac{TBW \times 1000}{Capacity (GB) \times Warranty (Days) \times 365}$.

---
## Related Notes
- [[01-Foundations/01-Hardware/H-01 Motherboard Architecture|H-01 Motherboard Architecture]] — Details on PCIe lanes and BIOS configurations.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-11 Storage — Disk Management and RAID|WS-11 Storage — Disk Management and RAID]] — Server-level software and hardware RAID configurations.
- [[02-Operating-Systems/04-Linux-RHEL/L-11 File Systems and Storage in Linux|L-11 File Systems and Storage in Linux]] — Linux partition formatting, LVM, and mounting storage.

