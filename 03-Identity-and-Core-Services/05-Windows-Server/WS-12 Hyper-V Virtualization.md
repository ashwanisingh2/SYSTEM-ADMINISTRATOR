---
tags: [sysadmin, windows-server, virtualization, hyper-v]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# WS-12: Hyper-V Virtualization

> [!abstract] Overview
> This note covers Windows Server Hyper-V virtualization. It details Type-1 hypervisor architecture, VM generation differences, virtual switch modes (External, Internal, Private), virtual disk formats, checkpoints, and Live Migration.

---
## Concept
Think of virtualization like renting out rooms in a large hotel building (the physical server hardware). 
- A **Type 2 Hypervisor** is a hotel operated inside another business (running virtual machines inside Windows 11 using VirtualBox). You must talk to the shopkeeper (Host OS) to get keys, which adds delay.
- A **Type 1 Hypervisor** is a purpose-built hotel (Hyper-V/ESXi). There is no host shopkeeper; the manager (Hypervisor) sits directly on the foundation concrete (hardware). The rooms (virtual machines) run directly on the foundations, separated by soundproof walls.
- The **Virtual Switch** is the internal wiring. You can link rooms to the street outside (External Switch), link rooms only to each other and the front office (Internal Switch), or seal them off so they can only talk to neighboring rooms (Private Switch).

*Seedha simple mein: Hyper-V Microsoft ka built-in Type-1 Hypervisor hai. Yeh physical hardware ko multiple virtual machines mein divide karta hai. Generation 2 VMs modern features jaise UEFI aur Secure Boot use karti hain.*

---
## Technical Deep Dive

### 1. Hyper-V Architecture: Parent & Child Partitions
Hyper-V is a **Type 1 (Bare-Metal)** hypervisor. When you install the Hyper-V role, the original Windows Server OS is converted into the **Parent Partition** (Management OS) running on top of the hypervisor. 
- **Management OS:** Responsible for managing VM configurations and hardware drivers.
- **Child Partitions:** The Guest Virtual Machines. They execute virtual workloads and access physical CPU and RAM directly under hypervisor scheduling, while I/O calls (disk, network) are routed via the Management OS using VMBus.

### 2. VM Generation 1 vs. Generation 2
When creating a VM, you must choose its generation:

| Feature | Generation 1 | Generation 2 |
|---|---|---|
| **Firmware** | BIOS (Legacy) | UEFI (Modern) |
| **Boot Disk** | MBR | GPT (Supports Secure Boot) |
| **Virtual Bus** | Emulated IDE/Legacy NIC | Native VMBus (SCSI/Synthetic NIC) |
| **PXE Boot** | Requires Legacy Network Adapter | Native support on standard NIC |
| **Max Virtual RAM** | 1 TB | 12 TB |
| **Max vCPUs** | 64 | 240 |

### 3. Hyper-V Virtual Switches (vSwitches)
- **External Switch:** Binds to a physical network adapter. VMs can communicate with other VMs, the host Management OS, and external physical networks (internet/LAN).
- **Internal Switch:** Connects VMs to each other and to the host Management OS. No external network access. Used for local testing labs.
- **Private Switch:** Connects VMs only to other VMs on the same host. The host Management OS has no network interface on this switch. Best for isolated lab networks.

### 4. Virtual Hard Disk Formats
- **VHD:** Legacy format. Max size 2 TB.
- **VHDX:** Modern format. Max size 64 TB. Features power failure resiliency logs to prevent corruption and supports live resizing.
- **Disk Allocation Profiles:**
  - **Dynamically Expanding:** Starts small and grows as data is written. Saves host disk space but adds minor write latency.
  - **Fixed Size:** Allocates the full virtual disk size immediately. Offers maximum performance and guarantees space availability.

### 5. Checkpoints (Snapshots)
- **Standard Checkpoint:** Captures the virtual disk state and the active memory state of the VM. Restoring rolls the VM back in time instantly, but can cause database replication desyncs (e.g., AD or SQL).
- **Production Checkpoint (Default):** Uses **VSS (Volume Shadow Copy Service)** inside the guest OS to create a application-consistent backup of the disk before generating the checkpoint. It does not capture memory states, ensuring safe rollbacks for transaction-based applications.

### 6. Live Migration
Live Migration moves a running VM from one physical Hyper-V cluster host to another without any downtime or loss of connection.
- **Requirements:** Shared storage (SAN or SMB 3.0 share), matching Active Directory domain memberships, and dedicated high-speed network interfaces for migration traffic.

---
## Windows Server CLI Configuration Commands

### Installing Hyper-V Role
```powershell
# Install Hyper-V role and management tools via PowerShell (Requires server reboot)
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

### PowerShell VM Configuration
```powershell
# Create a new External Virtual Switch
New-VMSwitch -Name "External_Switch" -NetAdapterName "Ethernet 1" -AllowManagementOS $true

# Create a new Generation 2 VM
New-VM -Name "VM-Prod-Web" -MemoryStartupBytes 4GB -Generation 2 -NewVHDPath "C:\Hyper-V\Disks\ProdWeb.vhdx" -NewVHDSizeBytes 80GB -SwitchName "External_Switch"

# Start the virtual machine
Start-VM -Name "VM-Prod-Web"
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows Server 2022 VM with Nested Virtualization enabled (or a physical server) and an ISO file for Windows 10/11.

### Step 1: Create a Virtual Switch
1. Open Server Manager -> Tools -> **Hyper-V Manager**.
2. In the right pane, click **Virtual Switch Manager**.
3. Select **Private** switch. Click **Create Virtual Switch**.
4. Name: `Isolated_Lab_Switch`. Click Apply and OK.

### Step 2: Create a Generation 2 VM
1. In Hyper-V Manager, right-click your Server -> **New** -> **Virtual Machine**.
2. Click Next. Name: `Lab_Client_01`. Click Next.
3. Choose **Generation 2**. Click Next.
4. Memory: Startup RAM `2048 MB`. Uncheck **Use Dynamic Memory** for this lab. Click Next.
5. Connection: Select `Isolated_Lab_Switch`. Click Next.
6. Virtual Hard Disk: Choose **Create a virtual hard disk**. Set size to `40 GB`. Click Next.
7. Installation Options: Select **Install an operating system from a bootable image file**. Browse to your Windows 10/11 ISO.
8. Click Finish.

### Step 3: Configure Settings and Boot
1. Right-click `Lab_Client_01` -> **Settings**.
2. Go to **Security** -> verify **Enable Secure Boot** is checked.
3. Go to **Processor** -> increase virtual processors to `2`. Click OK.
4. Double-click the VM to open the connection window. Click **Start**.
5. Press any key to boot from the CD/DVD. Install the operating system normally.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Attempting to start a VM fails, returning error: "Failed to start virtual machine. The virtual machine could not be started because the hypervisor is not running."
- **Root Cause:** Intel VT-x or AMD-V virtualization features are disabled in the physical host BIOS, or you are trying to run Hyper-V inside a VM without enabling nested virtualization.
- **Fix:**
  1. If running on physical hardware, reboot into BIOS/UEFI. Enable **Intel Virtualization Technology** (VT-x) or **AMD-V** and **Execute Disable Bit**.
  2. If running nested inside a VM, shut down the VM, open PowerShell on the physical host, and run:
     ```powershell
     Set-VMProcessor -VMName "MyParentVM" -ExposeVirtualizationExtensions $true
     ```
  3. Boot the host VM and start the Hyper-V role.

**Scenario 2:**
- **Problem:** A VM configured with Dynamically Expanding VHDXs crashes. The host reports the VM status as "Paused-Critical."
- **Root Cause:** The physical storage volume hosting the virtual hard disks has run out of space. When a dynamic disk expands and the host has zero disk space, the hypervisor pauses the VM immediately to prevent data corruption.
- **Fix:**
  1. Locate the physical volume hosting the `.vhdx` files.
  2. Free up space on the volume by deleting unused files, old standard checkpoints, or migrating other VMs off the volume using Storage Migration.
  3. Right-click the paused VM and select **Resume**. The VM will continue running without data loss.

---
## Common Mistakes
> [!warning] Avoid These
> **Using Standard Checkpoints as backups in Active Directory domains:** Taking standard checkpoints of Domain Controllers before changes, and restoring them days later. This rolls back the database clock, creating duplicate USNs (Update Sequence Numbers) and causing a **USN Rollback** which permanently halts Active Directory replication.
> **Correct approach:** Use Production Checkpoints or proper AD-aware system state backups that support Generation ID tracking to roll back databases safely.

---
## Pro Tips
> [!tip] Field Experience
> When configuring virtual switches, never uncheck "Allow management operating system to share this network adapter" on your only physical NIC unless you have a dedicated secondary NIC for host management. Doing so will cut off all remote connectivity (RDP/Admin Center) to the host server.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Type 1 Hypervisor| Hyper-V bare-metal hypervisor that sits directly on hardware, providing maximum speed. |
| 2 | Gen 2 VM | Modern VM profile utilizing UEFI, GPT boot disks, Secure Boot, and native synthetic drivers. |
| 3 | Private Switch | Virtual switch isolating VM traffic completely; host management OS has no adapter access. |
| 4 | VHDX | Modern virtual disk format supporting sizes up to 64TB and power loss protection logs. |
| 5 | Production Check| Checkpoint utilizing VSS inside guest OS to snapshot disks without capturing memory states. |

---
## Interview Q&A

**Q1: What is the difference between a Type 1 and a Type 2 Hypervisor? Provide examples of each.**
A: A **Type 1 (Bare-Metal) Hypervisor** installs directly on the physical hardware. There is no host operating system; the hypervisor acts as the kernel, managing hardware resources directly. It offers low latency, high security, and high performance. Examples include **Microsoft Hyper-V**, **VMware ESXi**, and **Proxmox VE**. A **Type 2 (Hosted) Hypervisor** runs as an application inside a host operating system (like Windows or macOS). All hardware calls must pass through the host OS first, adding latency and overhead. Examples include **Oracle VirtualBox** and **VMware Workstation**.

**Q2: A client demands high availability for their virtual machines across two physical Hyper-V servers. They want VMs to move automatically if a host crashes. Explain how you would implement this.**
A: 
- **Situation:** Virtual machines require automatic failover high availability.
- **Task:** Recommend and configure a Hyper-V clustering solution.
- **Action:** I will configure a **Failover Cluster** using both Hyper-V servers as nodes. First, I will provision shared storage (such as a Fiber Channel SAN, iSCSI SAN, or a shared SMB 3.0 file share) to host the virtual machine files. Second, I will configure a **Clustered Shared Volume (CSV)** so both hosts can access the storage simultaneously. Third, I will configure the VMs as clustered resources and set up a cluster witness (file share or disk quorum).
- **Result:** If Host A crashes, the cluster detects the failure and automatically restarts the VMs on Host B within seconds.

**Q3: Explain how Hyper-V Dynamic Memory works and the purpose of Memory Buffer and Memory Weight.**
A: Dynamic Memory allows Hyper-V to dynamically adjust the RAM allocated to a VM based on its active workload. 
- **Startup RAM:** The initial RAM allocated to boot the VM.
- **Minimum RAM:** The lowest RAM limit the hypervisor can reclaim from the VM during resource contention.
- **Memory Buffer:** A percentage value (default $20\%$) specifying how much extra RAM Hyper-V should attempt to reserve as standby memory based on the VM's active demands.
- **Memory Weight:** A priority rating determining which VM gets memory allocated first if the physical host runs out of physical RAM.

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-01 Windows Server 2022 Introduction|WS-01 Windows Server 2022 Introduction]] — Initial host setup.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-11 Storage — Disk Management and RAID|WS-11 Storage — Disk Management and RAID]] — Creating physical volumes for virtual disks.
- [[01-Foundations/01-Hardware/V-03 Hyper-V|V-03 Hyper-V]] — Virtualization client side and hypervisor comparisons.

