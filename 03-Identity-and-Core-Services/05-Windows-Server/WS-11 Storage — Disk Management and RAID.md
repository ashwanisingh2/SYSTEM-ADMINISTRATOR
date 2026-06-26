---
tags: [sysadmin, windows-server, storage, partition, diskpart]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# WS-11: Storage — Disk Management and RAID

> [!abstract] Overview
> This note covers storage administration within a Windows Server environment. It details disk profiles (Basic vs. Dynamic), partition tables (MBR vs. GPT), software RAID volume configurations, iSCSI initiator setup, and command diagnostics using `diskpart`.

---
## Concept
Think of your server's storage system like a warehouse floor. 
- **Basic Disks** are standard storage lockers. You can partition them into fixed rooms (Primary and Logical partitions) that cannot be easily stretched or merged across different lockers.
- **Dynamic Disks** are modular, expandable storage bays. You can merge space from different physical drives to form one massive logical room (Spanned), split data across drives to double your loading speed (Striped/RAID 0), or mirror drawers so data is written to two drives simultaneously (Mirrored/RAID 1).
- **iSCSI** is like renting storage space in a warehouse across the street (SAN storage array) and running a direct private tunnel (network cable) to route data so your server thinks the rented lockers are physically inside its own building.

*Seedha simple mein: Disk Management ke zariye hum storage initialize aur format karte hain. Dynamic disks software-based RAID options (0, 1, 5) offer karti hain. diskpart command utility complex storage automation aur formatting ke liye use hoti hai.*

---
## Technical Deep Dive

### 1. Basic Disks vs. Dynamic Disks
- **Basic Disk:** The default storage type in Windows. Uses standard partition tables compatible with all OS versions. Supports primary and extended logical partitions.
- **Dynamic Disk:** Allows advanced volume configurations that span multiple physical disks. 
  - *Deprecated warning:* Microsoft is transitioning dynamic disks to **Storage Spaces** (logical storage pools), but dynamic disks remain widely active in legacy and virtualized environments.

### 2. MBR vs. GPT Partition Tables
As reviewed in [[01-Foundations/01-Hardware/H-02 Storage Deep Dive|H-02 Storage Deep Dive]], MBR is legacy, limited to 2.2TB and 4 primary partitions. GPT is modern, supporting up to 9.4ZB and 128 partitions, featuring a backup partition table for reliability.

### 3. Dynamic Disk Volume Types
When using dynamic disks, Windows supports five volume profiles:
- **Simple Volume:** Uses disk space from a single physical disk. Can be extended within the same disk.
- **Spanned Volume:** Merges disk space from multiple physical disks (up to 32). Data is written sequentially: when Disk 1 is full, it writes to Disk 2. No performance increase or redundancy.
- **Striped Volume (RAID 0):** Alternates data blocks across 2 or more physical disks. Speeds increase because data reads/writes simultaneously across controllers. No redundancy; one disk failure destroys the volume.
- **Mirrored Volume (RAID 1):** Duplicates data on two physical disks. Offers fault tolerance.
- **RAID-5 Volume:** Stripes data and distributed parity across 3 or more physical disks. Offers fault tolerance (can survive 1 disk failure). Requires write calculations, adding minor CPU overhead.

### 4. Storage Spaces (Software-Defined Storage)
Microsoft's modern virtualization storage technology:
- **Storage Pool:** Combines physical disks of varying sizes and interfaces (SAS, SATA, NVMe) into a single logical pool.
- **Virtual Disks (Storage Spaces):** Carved out of the pool. Supports thin provisioning (allocating logical space larger than physical disks present) and resiliency types: Simple (no redundancy), Two-way mirror (RAID 1), Three-way mirror, or Parity (RAID 5).

### 5. iSCSI Storage Target & Initiator
- **iSCSI (Internet Small Computer System Interface):** A protocol that routes SCSI storage commands over IP networks, allowing servers to mount block-level storage from a SAN (Storage Area Network) over standard Ethernet.
- **iSCSI Target:** The server hosting the storage disks.
- **iSCSI Initiator:** The client server mounting the remote storage. It authenticates and mounts the disks, making them appear in Disk Management as local SCSI drives.

---
## Diskpart Commands Reference
`diskpart` is the built-in command-line utility for disk and partition management.

```cmd
:: Windows Command Prompt (Run as Administrator)
diskpart
  list disk                       :: List all connected physical drives
  select disk 1                   :: Select targeted disk 1
  clean                           :: Wipe all partition and volume metadata (destructive)
  convert gpt                     :: Initialize disk as GPT partition style
  create partition primary size=50000  :: Create a 50GB primary partition
  assign letter=S                 :: Assign drive letter S:
  format fs=ntfs quick label="Sales" :: Format using NTFS with label
  exit                            :: Exit diskpart utility
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows Server 2022 VM with 4 empty unallocated virtual hard disks (Disk 1, Disk 2, Disk 3, Disk 4) of 10GB each attached.

### Step 1: Open Disk Management and Initialize
1. Press `Win + X` -> select **Disk Management**.
2. A prompt will detect new disks. Select all disks, choose **GPT**, and click OK.
3. Verify that Disk 1, 2, 3, and 4 are initialized and show "Online - Unallocated."

### Step 2: Create a Mirrored Volume (RAID 1)
1. Right-click the unallocated space on **Disk 1** and select **New Mirrored Volume**. Click Next.
2. Select **Disk 2** from the left list. Click **Add**. Click Next.
3. Assign drive letter `M:`. Click Next.
4. Format: Choose **NTFS**, quick format, label: `Mirror_Data`. Click Next, then **Finish**.
5. Click **Yes** to allow the wizard to convert the basic disks to **Dynamic Disks**.
6. Verify that Disk 1 and Disk 2 display a dark red color and the status shows "Healthy."

### Step 3: Create a RAID-5 Volume
1. Right-click the unallocated space on **Disk 3** and select **New RAID-5 Volume**. Click Next.
2. Select **Disk 1** and **Disk 2**? No, those are already dynamic and full. Select **Disk 4** from the left list and click **Add**. Now select **Disk 3** and **Disk 4**? No, RAID 5 requires at least 3 disks.
3. We must allocate space on Disk 1, Disk 2, Disk 3, and Disk 4. Since Disk 1 and 2 are used for the mirror, let's verify if we have enough disks.
4. We have 4 disks: Disk 1 & 2 are in the Mirror. We have Disk 3 and Disk 4 left. RAID 5 needs 3 disks.
5. Let's adjust: We will delete the mirrored volume (or use smaller partition slices) to free up disks, or assume we attach a 5th disk.
6. Let's create the RAID-5 volume using **Disk 1, Disk 2, and Disk 3** from the start, or assume we have Disk 2, 3, and 4 available.
7. Right-click unallocated space on **Disk 2** -> **New RAID-5 Volume** -> Add **Disk 3** and **Disk 4**.
8. Set size to maximum, assign letter `R:`, quick format as NTFS with label `RAID5_Vol`. Click Finish. Confirm dynamic conversion.
9. Verify that the three disks display a cyan/light-blue color, status showing "Reformatting" then "Healthy."

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Attempting to extend a system volume (drive `C:`) fails because the "Extend Volume" option is greyed out in Disk Management, despite having 20GB of unallocated space on the physical disk.
- **Root Cause:** The unallocated space is not contiguous. It is located to the right of a recovery partition or another active volume. Windows can only extend basic disk partitions into contiguous unallocated space directly adjacent to the right.
- **Fix:**
  1. Open Command Prompt as Administrator. Run `diskpart`.
  2. If the recovery partition is blocking, you can delete it using diskpart override commands (ensure you have backups of recovery tools):
     ```cmd
     select disk 0
     list partition
     select partition [Recovery_Partition_Number]
     delete partition override
     ```
  3. Go back to Disk Management; the unallocated space is now adjacent, and "Extend Volume" is active.

**Scenario 2:**
- **Problem:** A server connected to an external iSCSI SAN fails to mount its storage drives after a reboot. The volume is missing from File Explorer.
- **Root Cause:** The iSCSI Initiator service failed to log into the target during boot because the network interfaces initialized slower than the storage service.
- **Fix:**
  1. Open Server Manager -> Tools -> **iSCSI Initiator**.
  2. Go to the **Targets** tab. Click **Quick Connect** targeting the SAN controller IP.
  3. If connection succeeds, go to the **Favorite Targets** tab. Ensure the target is added here so Windows attempts auto-mounts on boot.
  4. Change the **Microsoft iSCSI Initiator Service** startup type from Manual to **Automatic** in `services.msc`.
  5. Go to Disk Management and run "Rescan Disks" to mount the volumes.

---
## Common Mistakes
> [!warning] Avoid These
> **Converting a boot disk to Dynamic without checking backup support:** Converting a physical basic disk containing the boot OS (`C:`) to a dynamic disk without verifying if your third-party backup agent supports dynamic disk recovery. Many bare-metal restore tools cannot read dynamic volume layouts, blocking system recovery.
> **Correct approach:** Keep operating system boot drives as basic disks. Only use dynamic configurations or Storage Spaces on secondary data drives.

---
## Pro Tips
> [!tip] Field Experience
> When configuring virtual storage (such as Storage Spaces or SAN LUNs) on Hyper-V hosts, always format volumes using **ReFS (Resilient File System)** instead of NTFS. ReFS features built-in data integrity streams, accelerates virtual machine disk creations (fixed VHDX files initialize instantly), and performs faster merge operations on checkpoints.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Basic Disk | Standard storage disk profile supporting primary partitions and standard volume tables. |
| 2 | Dynamic Disk | Software-defined disk layout supporting spanning, striping (RAID 0), and mirroring (RAID 1). |
| 3 | diskpart | Powerful command-line utility used to initialize, wipe, and format physical and virtual disks. |
| 4 | iSCSI | Protocol routing block-level SCSI storage commands over standard IP network cables. |
| 5 | Storage Spaces | Modern software-defined storage pooling technology replacing legacy dynamic disks. |

---
## Interview Q&A

**Q1: What is the difference between a Spanned Volume and a Striped Volume (RAID 0) in Windows Disk Management?**
A: A **Spanned Volume** combines unallocated space from multiple physical disks sequentially. Data is written to the first disk until full, then continues onto the next disk. It does not provide any performance increase or redundancy. A **Striped Volume (RAID 0)** alternates data blocks evenly across all participating disks. Because write/read requests run simultaneously across multiple disk controllers, performance is significantly improved. Neither volume type provides redundancy: if one physical disk fails in either a spanned or striped volume, the entire volume is destroyed.

**Q2: A server loses connection to its iSCSI SAN. Walk through your troubleshooting process to identify where the link broke.**
A: 
- **Situation:** iSCSI target connection is lost, and storage volumes are offline.
- **Task:** Verify network pathing, initiator parameters, and target authentication.
- **Action:** First, I will verify L3 connection by pinging the iSCSI target IP from the initiator server. If pinging fails, I will check the storage network adapters and switches. Second, I will check the iSCSI Initiator properties. I need to check if the target status reads "Connecting" or "Inactive." Third, I will check if the SAN has blocked the connection due to an IQN (iSCSI Qualified Name) change or CHAP authentication mismatch.
- **Result:** Resolving the IP path or re-registering the initiator IQN on the SAN restores the target login status and mounts the volumes.

**Q3: Explain the difference between Thin Provisioning and Thick Provisioning in virtual disk deployment.**
A: **Thin Provisioning** allocates logical storage space to a virtual volume dynamically as data is written. For example, a 1TB thin volume initially consumes only a few megabytes on the physical array, growing only when files are added. This allows "over-provisioning" (allocating more logical space than physical space present). **Thick Provisioning** pre-allocates the entire designated storage space immediately. A 1TB thick volume consumes exactly 1TB of physical space on the array from day one, guaranteeing write availability and avoiding performance degradation caused by dynamic allocation overhead.

---

---
## Real-World Scenario: File Server Crash (500+ Users Affected)
Agar company ka main file server (500+ users connected) crash ho jaye, toh as a Senior Sysadmin aapka immediate action plan yeh hoga:
1. **Identify & Isolate**: Check if the server is pingable. Verify hardware status via Dell iDRAC / HPE iLO or check virtual machine status in vCenter/Hyper-V.
2. **Access Check & Lock**: If the storage volume is corrupt or unmapped, stop DFS Namespace referrals to the crashed target to redirect users to secondary replica servers if active.
3. **Trigger Disaster Recovery (DR)**: Start restoring the crashed system drive or volumes from the latest VSS snapshot or Azure Backup RSV restore point.
4. **Communication**: Broadcast status updates to the Incident Commander and helpdesk teams to manage incoming support volume.

### PowerShell Automation Snippet: Verify File Shares and Access Permissions
```powershell
# Get all active file shares on the server
Get-SmbShare | Format-Table -AutoSize

# Test local DFS Namespace server connection health
Test-DFSNamespaceTarget -Path "\\corp.local\DFSRoot\Public"
```

---
## Related Notes
- [[01-Foundations/01-Hardware/H-02 Storage Deep Dive|H-02 Storage Deep Dive]] — Base storage architectures and RAID concepts.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-09 File Server and FSRM|WS-09 File Server and FSRM]] — Hosting file shares on formatted NTFS volumes.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-12 Hyper-V Virtualization|WS-12 Hyper-V Virtualization]] — Virtual machine storage (VHDX) management.

