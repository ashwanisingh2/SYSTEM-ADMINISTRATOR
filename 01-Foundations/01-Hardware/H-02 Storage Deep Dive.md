---
tags: [sysadmin, hardware, storage, raid]
aliases: [H-02]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-yellow]
> 🔧 **HARDWARE**

`#complete` `#intermediate`

# H-02: Storage Deep Dive — HDD, SSD, NVMe, RAID

> [!abstract] Overview
> This note covers storage media architecture (HDD, SATA SSD, NVMe SSD), M.2 form factors, partition tables, disk health analysis (SMART), and RAID configurations. Mastery of these concepts is essential for server provisioning, data redundancy, and resolving storage performance issues.

---
## 🧠 Concept Overview

- **What it is** — Computer storage components ranging from slow mechanical HDDs to high-speed PCIe-based NVMe SSDs, and how multiple disks are organized for performance/redundancy via RAID.
- **Why it matters** — Proper storage architecture directly dictates a server's speed, data safety, and redundancy in the event of hardware failure.
- **Where you see this** — Provisioning physical servers, upgrading storage for performance, monitoring for disk failures, or recovering data from a degraded RAID array.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Identify failed drives physically, check disk health alerts via monitoring tools. |
| **L2** | Format partitions, hot-swap failed drives, configure basic RAID mirroring. |
| **L3** | Architect enterprise storage solutions, select NVMe/SSD/HDD blends, design complex RAID configurations for massive databases. |

> [!tip] Seedha Simple Mein
> *HDD mechanical disk hoti hai jo ghoomti hai aur dheemi hoti hai. SSD chips par chalti hai aur bohot fast hoti hai. NVMe sabse tez hoti hai kyunki yeh PCIe slots par directly CPU se baat karti hai. RAID alag-alag drives ko jodkar speed aur redundancy badhane ka tareeqa hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Storage** is like **an office filing system** because...
>
> - **HDD** is like a large filing cabinet in the basement. Huge capacity, but the runner (read/write head) must physically fetch the file.
> - **SATA SSD** is a cabinet right next to your desk. No moving parts, but files pass through a narrow chute (SATA bus).
> - **NVMe SSD** is having files projected onto your desk via a multi-lane high-speed pneumatic tube system (PCIe lanes) straight from the core archives.
> - **RAID** is like having duplicate copies of all folders spread across multiple drawers so that if a cabinet catches fire, you can instantly recreate the missing documents.

---
## 🔬 Technical Deep Dive

### 1. HDD Internals & Mechanical Bottlenecks

> [!info] Key Concept
> Hard Disk Drives (HDDs) store data magnetically on spinning physical disks.

- **Platters:** Aluminum or glass discs coated with a magnetic layer. Data is stored in concentric circles called **tracks**, which are divided into **sectors** (typically 512 bytes or 4KB/Advanced Format).
- **Read/Write Head:** Electromagnetic tip attached to an actuator arm that hovers nanometers above the spinning platter.
- **RPM (Rotations Per Minute):** Common speeds are 5400 RPM, 7200 RPM, and 10,000 to 15,000 RPM. 
  - **Latency:** Higher RPM decreases rotational latency.
  - **Seek Time:** The time it takes for the actuator arm to move to the correct track.

### 2. SSD vs. HDD Comparison

| Attribute | HDD (Hard Disk Drive) | SSD (Solid State Drive) |
|---|---|---|
| **Mechanism** | Mechanical (Spinning Platters) | Semiconductor (NAND Flash) |
| **Random I/O**| ~75 - 200 IOPS | 50,000 - 1,000,000+ IOPS |
| **Sequential Read** | ~100 - 250 MB/s | ~500 - 7,500+ MB/s |
| **Access Latency** | ~5 - 15 ms | ~0.02 - 0.1 ms |
| **Lifespan** | Mechanical wear/shock | Limited write endurance (TBW) |
| **Best Use Case** | Cold data archives, backups | OS drives, databases, virtualization |

### 3. NVMe vs. SATA SSD

- **SATA (Serial ATA):** Uses the legacy AHCI protocol designed for spinning disks. SATA III caps out at ~550 - 600 MB/s. Queue depth is limited to 32 commands.
- **NVMe (Non-Volatile Memory Express):** Designed for flash storage. Runs directly over PCIe lanes (x4).
  - **Speed:** Gen 3 (~3,500 MB/s); Gen 4 (~7,500 MB/s); Gen 5 (~14,000 MB/s).
  - **Concurrency:** Supports up to 64,000 queues.

### 4. M.2 Form Factor & Keying

M.2 is a physical form factor. An M.2 drive can be SATA-based or PCIe (NVMe)-based.
- **Sizes:** Described by width and length (in mm). E.g., **2280** means 22mm x 80mm.
- **Physical Keys (Notches):**
  - **B-Key:** Supports SATA or PCIe x2.
  - **M-Key:** Supports PCIe x4 (NVMe) or SATA.
  - **B+M Key:** Has both notches; PCIe x2 speeds or SATA.

```
M.2 M-Key Notch (Right Side): [=======   ===]
M.2 B-Key Notch (Left Side):  [===   =======]
M.2 B+M Key (Dual Notch):     [===   ===   ===]
```

### 5. RAID Levels (Redundant Array of Independent Disks)

| 💽 RAID Level | 🛠️ Description | 📦 Min Drives | 📊 Space Efficiency | 🛡️ Fault Tolerance | 🚀 Read/Write Speed |
|---|---|---|---|---|---|
| **RAID 0** | Striping (No parity) | 2 | 100% | 0 Drives (Data loss risk) | Fast Read, Fast Write |
| **RAID 1** | Mirroring | 2 | 50% | 1 Drive | Fast Read, Normal Write |
| **RAID 5** | Striping + Distributed Parity | 3 | $(N-1)/N$ | 1 Drive | Fast Read, Slow Write |
| **RAID 10** | Striped Mirrors (1+0) | 4 | 50% | Up to 2 Drives | Very Fast Read, Fast Write |

### 6. Disk Health Monitoring & SMART Attributes

**S.M.A.R.T. (Self-Monitoring, Analysis, and Reporting Technology)** allows drives to monitor health indicators. 
- **Reallocated Sectors Count:** Damaged sectors remapped to spare sectors. Non-zero indicates drive failure is imminent.
- **Current Pending Sector Count:** Unstable sectors waiting to be remapped. 

### 7. Partition Schemes: MBR vs. GPT

- **MBR (Master Boot Record):** Legacy, max disk size 2.2 TB, max 4 primary partitions.
- **GPT (GUID Partition Table):** Modern, max disk size 9.4 ZB, max 128 partitions, stores a backup partition table.

> [!danger] Common Mistake
> **Using RAID as a backup solution:** RAID only provides hardware redundancy. If a user deletes a file, or ransomware encrypts it, the action is instantly mirrored across all drives. Always maintain a dedicated external backup.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 10/11 or Windows Server environment with at least two unallocated disks.

### Step 1: Open Disk Management
1. Press `Win + X` and select **Disk Management** (or run `diskmgmt.msc`).
2. If new disks are detected, initialize them as **GPT**.

### Step 2: Create a Mirrored Volume (RAID 1)
1. Right-click on **Disk 1** (unallocated space) and select **New Mirrored Volume**.
2. Click **Next** in the wizard.
3. Select the second disk (**Disk 2**) from the left list and click **Add**.
4. Set volume size to maximum and click **Next**.

### Step 3: Assign Drive Letter and Format
1. Choose an available drive letter (e.g., `R:`).
2. Format as **NTFS** with label `RAID1_DATA`.
3. Quick format and click **Finish**. Convert to **Dynamic Disks** if prompted.

> [!success] Expected Output
> The drives will turn a dark red/brown color stating "Resynching...". Once **Healthy**, drive `R:` behaves as a single volume.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-PhysicalDisk` | List all physical disks, status, and media type | `Get-PhysicalDisk` |
| `Initialize-Disk` | Initializes a new disk | `Initialize-Disk -Number 1 -PartitionStyle GPT` |
| `lsblk` | List block devices (disks and partitions) in Linux | `lsblk` |
| `smartctl -A` | View SMART attributes for drive health | `sudo smartctl -A /dev/sda` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| RAID 5 array degraded | A single drive has failed | Hot-swap the failed drive with identical model. Rebuild starts automatically. |
| OS installer fails to see NVMe | BIOS SATA mode set to RAID/RST instead of AHCI | Change BIOS to AHCI. Or use `diskpart` to clean and convert to GPT. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: RAID Array Degraded

> [!example] Ticket
> "Server PRD-DB01 is flashing an amber light on Drive Bay 2. Monitoring shows RAID 5 array is degraded."

**L1 Response:** Verify the alert in monitoring tools. Identify the physical server and the exact drive bay.
**Escalation Trigger:** Pass to L2 if hot-swap spare is not available or array rebuild does not start automatically.
**L2 Resolution:** Hot-swap the failed drive with an identical one. Monitor the RAID controller utility to ensure the rebuild process completes successfully without rebooting the server.

---
## 🎤 Interview Questions

> [!question] Q1: What is the write penalty of a RAID 5 array, and how does it affect write performance?
> **Answer:** The write penalty of RAID 5 is 4. When a write occurs, the controller performs 4 operations: Read old data, Read old parity, Write new data, and Write new parity. This makes random writes significantly slower than RAID 10.

> [!question] Q2: Explain the difference between TBW and DWPD when measuring SSD lifespan.
> **Answer:** **TBW (Terabytes Written)** is the total cumulative data a drive can write before NAND flash degrades. **DWPD (Drive Writes Per Day)** measures how many times you can write the drive's total capacity daily over its warranty period.

==**Exam Tip:** Always know the difference between RAID 0 (speed, no redundancy) and RAID 1 (redundancy, normal speed).==

---
## 🔗 Related Notes

- [[01-Foundations/01-Hardware/H-01 Motherboard Architecture|H-01 Motherboard Architecture]] — Motherboard PCIe lanes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-11 Storage — Disk Management and RAID|WS-11 Storage — Disk Management and RAID]] — Windows Server storage
- [[02-Operating-Systems/04-Linux-RHEL/L-11 File Systems and Storage in Linux|L-11 File Systems and Storage in Linux]] — Linux file systems
