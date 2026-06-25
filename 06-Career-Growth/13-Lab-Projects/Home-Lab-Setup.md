---
tags: [desktop-support, lab-setup, virtualization, vmware, hyper-v, L1, L2]
aliases: [home-lab, hypervisor-setup]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102, #az-104
---

# Home Lab Setup Guide

---

## Concept Overview
- **What it is**: A foundational guide for setting up a local physical or virtualized lab environment to practice enterprise desktop support, system administration, and network engineering.
- **Why it matters**: A home lab is the single best tool for hands-on learning. It allows engineers to test dangerous configurations, software deployments, and system upgrades without impacting production networks.
- **Where you encounter this in real job**: Building sandbox staging environments, testing group policies before active deployment, and validating patch updates.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Deploys client client operating systems (Windows 10/11) inside virtual environments and joins them to domains.
  - **L2**: Manages local server OS deployments, virtual switches, network routing paths, and virtual storage pools.
  - **L3**: Architectures bare-metal type-1 hypervisors (Proxmox VE, ESXi), sets up high availability, cluster storage (Ceph/SAN), and automated VM template provisioning.

---

## Technical Deep Dive

### Hypervisor Classification
Hypervisors are software platforms that manage and run virtual machines (VMs). They are divided into two main categories:
1. **Type-1 Hypervisor (Bare-Metal)**: Installs directly on the physical hardware host. It has no parent operating system, providing the highest performance and efficiency.
   - *Examples*: Proxmox VE, VMware ESXi, Hyper-V Server.
2. **Type-2 Hypervisor (Hosted)**: Installs as an application on top of an existing host operating system (e.g., Windows 11 or macOS).
   - *Examples*: VMware Workstation Player/Pro, Oracle VirtualBox.

```
+-----------------------------------+      +-----------------------------------+
|       Type-1 (Bare-Metal)         |      |          Type-2 (Hosted)          |
| +-------------------------------+ |      | +-------------------------------+ |
| |        Virtual Machines       | |      | |        Virtual Machines       | |
| +-------------------------------+ |      | +-------------------------------+ |
| |           Hypervisor          | |      | |           Hypervisor          | |
| +-------------------------------+ |      | +-------------------------------+ |
| |        Physical Hardware      | |      | |         Host OS (Win/Mac)     | |
| +-------------------------------+ |      | +-------------------------------+ |
+-----------------------------------+      | |        Physical Hardware      | |
                                           | +-------------------------------+ |
                                           +-----------------------------------+
```

### Lab Hardware Recommendations
To run a stable lab containing a Windows Server Domain Controller, a Linux server, and two Windows client workstations, target these specifications:
- **CPU**: Intel Core i5/i7 or AMD Ryzen 5/7 (minimum 6 Cores / 12 Threads).
- **RAM**: 16 GB minimum (32 GB highly recommended).
- **Storage**: 500 GB or 1 TB NVMe SSD. HDDs are too slow for running multiple concurrent VMs.
- **Network**: Gigabit Ethernet connection (for bridging configurations).

### Virtual Network Architectures
- **Bridged**: The VM binds directly to the host's physical network card. It receives an IP address from your home router (e.g., `192.168.1.X`), making it visible to all devices on your local home network.
- **NAT (Network Address Translation)**: The hypervisor creates a private internal virtual network. VMs can access the external internet through the host's IP address, but external home devices cannot contact the VMs directly.
- **Host-Only / Private**: A closed virtual network isolated from the host physical network and the internet. VMs can talk only to each other and the host. Ideal for testing active DHCP servers or domain controllers.

---

## Commands & Syntax

### PowerShell: Enabling Hyper-V on Windows 10/11 Pro/Enterprise
Run this command in an administrative PowerShell session to install the Hyper-V role locally:
```powershell
# Enable the Hyper-V platform and management tools
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All -NoRestart

# Query active virtual network adapters on the host
Get-NetAdapter | Where-Object Status -eq "Up"

# Create a private virtual switch for sandbox lab isolation
New-VMSwitch -Name "LabIsolatedSwitch" -SwitchType Private
```

### CMD / CLI: VirtualBox CLI Management (VBoxManage)
Manage virtual networks and VM states from the command line using Oracle VirtualBox tools:
```cmd
REM List all registered virtual machines in VirtualBox
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list vms

REM Power on a target lab VM in headless (background) mode
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "SRV-AD-01" --type headless
```

### GUI Path
> **VMware Workstation**: Edit -> Virtual Network Editor (to configure custom NAT and Host-only subnets)
> **Hyper-V Manager**: Action -> Virtual Switch Manager (to build External, Internal, or Private switches)

---

## Real-World Scenarios

### Scenario 1: Hyper-V VM Creation Fails (Nested Virtualization)
**User Complaint**: You are trying to install Hyper-V or Proxmox inside a virtual machine running on VMware Workstation to test a nested cluster, but the guest OS installer displays the error: "Hypervisor is not running or hardware virtualization is disabled."
**Your First 3 Checks**:
1. Check the physical BIOS/UEFI settings on the host machine to ensure Intel VT-x or AMD-V is enabled.
2. Check the CPU settings of the virtual machine inside VMware Workstation.
3. Check the guest OS configuration files.
**Diagnosis Steps**:
1. Confirm the physical host BIOS has Intel Virtualization Technology set to Enabled.
2. Open the VM settings in VMware Workstation and navigate to Hardware -> Processors.
3. Observe that the checkbox "Virtualize Intel VT-x/EPT or AMD-V/RVI" is unchecked.
**Root Cause**: The hypervisor VM was provisioned without passing the hardware virtualization flags from the physical CPU down into the guest VM processor configurations.
**Fix**:
1. Power off the target guest virtual machine.
2. In VMware Workstation, open the VM settings -> Processors -> Check the box **"Virtualize Intel VT-x/EPT or AMD-V/RVI"**.
3. Save settings, boot the guest VM, and complete the installation of the nested hypervisor role.
**Prevention**: Always select "Hyper-V" or "Proxmox" as the pre-configured guest operating system type when creating nested VMs, as this automatically enables virtualization passthrough.

### Scenario 2: Storage Pool Exhaustion on Test Host
**User Complaint**: Your local lab host running VMware Workstation suddenly freezes, and all active lab virtual machines show a suspended state. You cannot boot or resume any VMs.
**Your First 3 Checks**:
1. Check the physical host disk space using File Explorer or disk management.
2. Check if the lab VMs are using thin-provisioned disks.
3. Check for active virtual machine snapshots.
**Diagnosis Steps**:
1. Open File Explorer on the host machine: C: drive shows 0 bytes free.
2. Check the VM storage folders. Each VM is configured with a 60GB "thin-provisioned" virtual disk.
3. Locate VM directory contents: several VMs have multiple snapshot files (`.vmsn` / `.vmdk` delta files) taking up 30GB of space each.
**Root Cause**: Thin-provisioned virtual disks grew as data was written, and forgotten VM snapshots filled up the remaining physical host storage pool, causing the hypervisor to suspend all operations to prevent corruption.
**Fix**:
1. Delete or merge old VM snapshots using the VMware Workstation Snapshot Manager.
2. Empty the host's temp files, recycle bin, or download folders to reclaim at least 10GB of temporary physical disk space.
3. Resume the suspended VMs one by one and clean up internal logs, or run defragment/shrink disk commands in the hypervisor GUI.
**Prevention**: Avoid keeping VM snapshots for more than 48 hours. Monitor physical host disk capacity using automated alerting scripts.

---

## Critical Points

> [!danger] Never Do This
> Do not run a local lab DHCP server on a virtual network switch set to **Bridged** mode. Doing so will lease invalid local IP addresses to physical home devices (like family phones or smart TVs), breaking their internet access and disrupting your home network.

> [!warning] Common Trap
> Setting VM disk sizes to "Pre-allocated" (thick provisioning) unless you have excess storage capacity. Always use "Split virtual disk" or "Thin provisioning" so the VM files only take up the disk space actually used inside the guest OS.

> [!tip] Senior Engineer Tip
> Build base "Gold Images" for Windows Server, Windows Client, and Linux. Once installed, patched, and configured with basic tools, run a `sysprep` tool (for Windows) and export them as templates. Clone these templates to create new lab VMs in seconds.

> [!success] Verification Steps
> To verify a private lab network is safely isolated:
> 1. Set the VM switch to Private or Host-only.
> 2. Boot a lab VM and run `ping 8.8.8.8`.
> 3. Verify that the ping request times out (confirming no path to the public internet).

> [!question] Interview Alert
> "What is the difference between thin provisioning and thick provisioning of virtual disks?"
> - **Answer**: Thin provisioning allocates storage space on-demand as data is written inside the VM, keeping host file sizes small initially. Thick provisioning allocates the entire virtual disk capacity immediately upon creation, reserving that space on the host disk regardless of actual usage.

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Running out of host RAM | Over-allocating memory to VMs | Sum your VM memory requirements and keep them at least 4GB below your host's total RAM. |
| Duplicate SID issues | Cloning Windows VMs without running Sysprep | Run `sysprep /generalize /oobe /shutdown` before cloning base Windows images. |
| Lost VM network routing | Changing physical adapters (switching from Wi-Fi to Ethernet) | Reconfigure your virtual switch bindings in the Hypervisor Network Settings after changing adapters. |

---

## Lab Exercise

**Objective**: Install Oracle VM VirtualBox and Configure an Isolated Private Network.
**Time Required**: 20 minutes
**Environment Needed**: Windows 10/11 Workstation.

**Steps**:
1. Download and install Oracle VM VirtualBox (latest stable version).
2. Create a new virtual network:
   - Navigate to File -> Tools -> Network Manager.
   - Click the **Host-only Networks** tab.
   - Click **Create** to add `VirtualBox Host-Only Ethernet Adapter`.
   - In the Adapter tab, configure the IP manually: IP: `192.168.56.1`, Subnet Mask: `255.255.255.0`.
   - Select the **DHCP Server** tab and uncheck the **Enable Server** box (to prevent conflicts).
3. Create a test VM (no OS install needed) and go to Settings -> Network.
4. Set Adapter 1 to Attached to: **Host-only Adapter** and select your new adapter name.
5. Boot the VM and verify it does not receive an automatic IP.

**Pass Criteria**: The virtual network adapter is created, DHCP is disabled, and the VM associates with the isolated host-only interface.

---

## Interview Questions & Answers

### L1 Level

#### Q1: What is a VM snapshot and when should you use it?
**A**: A snapshot saves the complete state, disk contents, and memory of a virtual machine at a specific point in time. You should use snapshots before performing risky tasks like installing software updates, editing registry files, or running testing scripts. If the operation fails, you can quickly revert the VM to the saved snapshot state.

#### Q2: How do you access a virtual machine if it loses network connectivity?
**A**: I use the hypervisor's built-in console window (e.g., VMware Console or Hyper-V Connection window). This console acts as a virtual monitor plugged directly into the guest OS, allowing me to log in and troubleshoot local configuration errors even if network access is completely broken.

---

### L2 Level

#### Q3: What are guest additions (or VM Integration Services) and why are they necessary?
**A**: Guest additions are a package of drivers and system applications installed inside the guest operating system. They optimize VM performance by adding support for dynamic resolution scaling, shared folders with the host, clipboard sharing, mouse integration, and high-performance network/storage drivers.

#### Q4: Explain the difference between Type-1 and Type-2 hypervisors. When would you deploy a Type-1 hypervisor?
**A**: A Type-1 hypervisor runs directly on bare physical hardware and is used in production data centers (like Proxmox or ESXi) because it eliminates host OS overhead. A Type-2 hypervisor runs as an application on a host OS (like VMware Workstation or VirtualBox) and is used for local client-side testing, labs, and development.

---

### L3 Level

#### Q5: Explain how you would set up a nested virtualization lab environment to train junior administrators.
**A**:
- **Situation**: Junior support techs needed to practice installing Hyper-V cluster features, but we lacked physical server hardware.
- **Task**: Deploy a virtualized multi-node Hyper-V lab on a single physical workstation.
- **Action**: I deployed a Type-2 hypervisor (VMware Workstation) on a 32GB RAM workstation. I installed Windows Server 2022 as a VM. In the VM configuration file (`.vmx`), I added the parameters `vhv.enable = "TRUE"` to enable nested hardware virtualization. I then configured two guest Windows 10 VMs inside the virtual Windows Server.
- **Result**: Built a fully functional nested virtualization laboratory, allowing techs to practice clustering and GPO deployments at zero hardware cost.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **Hardware**: SSD is critical for running multiple VMs. Minimum 16GB RAM for basic server/client labs.
> **Networking**: Bridged connects to local LAN. NAT routes through host IP. Host-Only isolates the network (no internet).
> **Rule of Thumb**: Disable hypervisor DHCP servers if configuring custom AD Domain Controllers.
> **Template**: Build, patch, and run `sysprep` on a base template VM before cloning.

---

## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/Windows-Server-2022-Introduction|Windows Server 2022 Introduction]]
- [[02-Operating-Systems/04-Linux-RHEL/Filesystem|Linux Filesystem]]
- [[00-MOC/Master-Index|Master Index]]

---

## Tags
#desktop-support #virtualization #hyper-v #vmware #virtualbox #home-lab #system-admin #lab-ready #L1 #L2

