---
tags: [sysadmin, virtualization, vmware, hypervisor]
difficulty: Intermediate
lab-required: Yes
read-time: 12 mins
---

# V-01: VMware Workstation Complete Guide

> [!abstract] Overview
> This note covers VMware Workstation desktop virtualization. It details configuration parameters, hardware sizing, network modes (Bridged, NAT, Host-Only), checkpoint snapshots, cloning techniques, and OVA/OVF importing.

---
## Concept
Think of VMware Workstation as a sandbox testing laboratory located inside a physical office building (your physical PC/host OS). 
- The physical host supplies electricity, desk space, and AC.
- When you build a Virtual Machine, you build a custom glass office inside the lab. You decide how many desks (vCPU cores), how much file cabinet space (virtual disk), and how many filing systems (RAM) this office gets. 
- You can connect this office's telephone line to the street outside (Bridged Network), route their calls through the lab's main desk so they share the office phone number (NAT Network), or block their phone line so they can only call other offices inside the laboratory (Host-Only).

*Seedha simple mein: VMware Workstation ek Type-2 Hypervisor hai jo host OS ke upar multiple VMs run karne ki permission deta hai. Isme hum virtual network adapters (Bridged, NAT, Host-Only) configure karte hain and snapshots ke zariye state capture karte hain.*

---
## Technical Deep Dive

### 1. VMware Workstation vs. VMware ESXi
- **VMware Workstation:**
  - **Classification:** **Type 2 (Hosted) Hypervisor**.
  - **Interface:** Runs as an application inside Windows or Linux.
  - **Use Case:** Software testing, administration labs, local development.
- **VMware ESXi (vSphere):**
  - **Classification:** **Type 1 (Bare-Metal) Hypervisor**.
  - **Interface:** Installs directly on server hardware. Managed via a web-browser console over network.
  - **Use Case:** Production enterprise server virtualization.

### 2. VM Network Modes Explained
Virtual networks are managed via the **Virtual Network Editor (vmnetcfg)**.

- **Bridged Mode (VMnet0):**
  - **How it works:** Connects the VM's virtual NIC directly to the host's physical network adapter. 
  - **IP Allocation:** The VM behaves as a physical machine on the local LAN. It obtains an IP from the physical DHCP server (e.g., `192.168.1.100` alongside the host's `192.168.1.50`).
  - **Use Case:** Best when the VM needs to act as a server accessible to other physical machines on the local network.
- **NAT Mode (Network Address Translation - VMnet8):**
  - **How it works:** The VM connects to a private virtual network hosted by VMware. VMware runs a virtual DHCP and NAT router. 
  - **IP Allocation:** The VM gets a private IP (e.g., `192.168.150.128`). External networks only see the host's physical IP address.
  - **Use Case:** Default mode. Safe for internet browsing; protects the VM from external incoming scans.
- **Host-Only Mode (VMnet1):**
  - **How it works:** Connects the VM to a private network isolated from the physical adapter. 
  - **IP Allocation:** The VM can communicate only with other VMs on VMnet1 and the host OS loopback. No external internet access.
  - **Use Case:** Creating isolated lab networks (e.g., setting up active directory domains without interfering with the corporate LAN).

### 3. Snapshots
Snapshots save the exact state of a VM at a specific point in time (RAM state, disk delta, settings).
- **Mechanism:** When you take a snapshot, the primary `.vmdk` disk file becomes read-only, and a new delta disk file (`-000001.vmdk`) is created to store all new writes.
- **Merge/Delete:** Deleting a snapshot does not lose data; it merges the changes from the delta disk back into the base disk, reclaiming host space.

### 4. Linked Clones vs. Full Clones
Cloning replicates an existing VM template:
- **Linked Clone:** Points directly to the parent VM's base virtual disk. It only stores new delta writes on its local disk.
  - **Pros:** Creation is instant; consumes minimal disk space (a few MBs initially).
  - **Cons:** If the parent VM is deleted or moved, the linked clone breaks and is unrecoverable.
- **Full Clone:** A complete, independent duplicate copy of the parent VM files.
  - **Pros:** Safe and independent; parent can be deleted without issues.
  - **Cons:** Slow creation; consumes identical disk space as the parent (GBs).

### 5. VMware Tools
A bundle of drivers and utilities installed inside the guest OS.
- **Why it is critical:** Enables high-resolution graphics drivers, smooth mouse cursor movement, shared clipboard copy-pasting, drag-and-drop file transfers, and host-guest shared folder mounting.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A workstation running Windows with VMware Workstation Pro installed, and a Windows 10/11 ISO file.

### Step 1: Create a New Virtual Machine
1. Launch VMware Workstation. Click **File** -> **New Virtual Machine**.
2. Select **Typical (recommended)**. Click Next.
3. Select **Installer disc image file (iso)**. Browse and select your Windows ISO. Click Next.
4. Name: `Client_Lab_VM`. Location: Select a directory with at least 40GB free space. Click Next.
5. Disk Capacity: Set maximum disk size to `40 GB`. Select **Split virtual disk into multiple files** (simplifies moving VMs to other drives). Click Next.
6. Click **Customize Hardware**.
7. Memory: Set to `4096 MB` (4GB).
8. Network Adapter: Select **NAT**.
9. Click Close, then click **Finish**. The VM boots automatically to start OS installation.

### Step 2: Install VMware Tools
1. Once the guest OS (Windows) installation completes and boots to the desktop:
2. In the VMware Workstation menu, click **VM** -> **Install VMware Tools**.
3. A virtual DVD drive will mount inside the guest OS. Open File Explorer -> Double-click the virtual drive (`D:\`).
4. Run `setup64.exe`. Follow the wizard: select **Typical** installation. Click Next, then Install.
5. Once complete, reboot the VM. Confirm that mouse integration is smooth and the display window auto-resizes to match the application boundaries.

### Step 3: Configure Shared Folders
1. Shut down the VM.
2. In VMware, select the VM -> click **Edit virtual machine settings**.
3. Go to the **Options** tab. Click **Shared Folders**.
4. Folder Sharing: Select **Always enabled**.
5. Click **Add**. Map a folder on your host machine (e.g., `C:\HostShared`) and name it `SharedDir`. Click OK.
6. Start the VM.
7. Open File Explorer in the guest. Navigate to Network -> **`vmware-host`** -> **Shared Folders** -> `SharedDir`. Verify files can be read/written across the boundary.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** When attempting to launch VMware Workstation or start a VM, the host blue-screens (BSOD), or throws error: "VMware Workstation and Device/Credential Guard are not compatible."
- **Root Cause:** A hypervisor conflict. Microsoft Hyper-V is running on the host, locking out physical CPU virtualization registers from being accessed by the Type-2 VMware hypervisor.
- **Fix:**
  1. Open Windows Features on the host machine.
  2. Uncheck **Hyper-V**, **Virtual Machine Platform**, and **Windows Sandbox**. Click OK.
  3. Open Command Prompt as Administrator. Disable hypervisor launch on boot:
     ```cmd
     bcdedit /set hypervisorlaunchtype off
     ```
  4. Reboot the host computer. VMware Workstation can now launch VMs.

**Scenario 2:**
- **Problem:** A virtual machine is configured in Bridged mode, but it cannot obtain an IP address from the network DHCP server, defaulting to APIPA (`169.254.x.x`). NAT mode works.
- **Root Cause:** Bridged interface binding mismatch. VMware's virtual switch `VMnet0` is configured to "Auto-bridge," automatically binding to an inactive network adapter (e.g., a virtual VPN adapter or bluetooth interface) instead of the active physical Ethernet/Wi-Fi adapter.
- **Fix:**
  1. In VMware, click **Edit** -> **Virtual Network Editor**.
  2. Click **Change Settings** (admin credentials required).
  3. Select **VMnet0** (Bridged).
  4. Under "Bridged to", change from **Automatic** to the explicit name of your active physical interface card (e.g., `Intel(R) Ethernet Connection` or `Killer Wi-Fi 6`).
  5. Click Apply and OK. Re-run `ipconfig /renew` on the guest VM.

---
## Common Mistakes
> [!warning] Avoid These
> **Keeping long chains of snapshots active on production VMs:** Leaving dozens of snapshots active for months. Every snapshot generates a new delta disk file, causing disk read operations to chain through multiple files, severely degrading virtual disk performance and saturating host storage.
> **Correct approach:** Snapshots should only be used for short-term rollbacks (e.g., before applying system updates). Delete and merge snapshots back to the base disk within 24-72 hours.

---
## Pro Tips
> [!tip] Field Experience
> When migrating VMs between different physical hosts running VMware Workstation, export the VM as an **OVF (Open Virtualization Format)** package instead of just copying the files. An OVF package compresses the virtual disk and standardizes configuration metadata, preventing compatibility issues on import.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Type 2 Hypervisor| Virtualization application running on top of a host operating system (e.g., VMware Workstation). |
| 2 | Bridged Mode | Network profile where the VM binds to the physical NIC and acts as a separate host on the LAN. |
| 3 | NAT Mode | Network profile where the VM shares the host IP address via a virtual private DHCP router. |
| 4 | Linked Clone | A fast clone that points to the parent VM's base disk, only saving new data deltas. |
| 5 | VMware Tools | Core driver package installed in guest VMs to enable high resolution, mouse integration, and folders. |

---
## Interview Q&A

**Q1: Explain the difference between a Bridged, NAT, and Host-Only network configuration in VMware Workstation.**
A: **Bridged mode** connects the virtual NIC directly to the host's physical network adapter. The VM appears as an independent physical machine on the local network, obtaining an IP from the local router. **NAT mode** connects the VM to a private network managed by VMware. The VM shares the host's physical IP for outbound connections using port translation, protecting the VM from incoming internet scans. **Host-Only mode** isolates the VM completely: it can only communicate with other VMs on that virtual switch and the host OS. It has zero external WAN access.

**Q2: You want to deploy a template VM for developers. They need to create 10 duplicates immediately, but storage space is extremely limited. How do you implement this?**
A: 
- **Situation:** Duplicate VMs must be deployed rapidly under tight storage limits.
- **Task:** Recommend and configure the appropriate cloning mechanism.
- **Action:** I will create a base master VM, configure the OS and required developer tools, and shutdown the VM. I will generate a snapshot of this base state. Then, I will instruct the developers to create **Linked Clones** pointing to this snapshot.
- **Result:** Creation is instant, and each clone initially consumes less than 50MB of space, saving hundreds of gigabytes of host storage compared to full clones.

**Q3: Why are virtual machine disks split into multiple files (e.g., 2GB chunks) by default in VMware Workstation?**
A: Splitting virtual disks into multiple files makes them easier to manage on FAT32 and ExFAT external storage drives, which have a maximum single file size limit of 4GB. It also makes moving, copying, or backing up the VM files faster: if only a small section of the disk is modified, backup utilities only need to copy the specific modified 2GB chunk files instead of a single, massive 100GB `.vmdk` file.

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-12 Hyper-V Virtualization|WS-12 Hyper-V Virtualization]] — Alternative bare-metal Hyper-V configurations.
- [[01-Foundations/01-Hardware/V-02 VirtualBox Complete Guide|V-02 VirtualBox Complete Guide]] — Comparison with VirtualBox hypervisor.
- [[01-Foundations/01-Hardware/V-04 Virtualization Lab Environment Setup|V-04 Virtualization Lab Environment Setup]] — Building multi-VM labs.

