---
tags:
  - desktop-support
  - azure
  - virtual-machines
  - infrastructure
  - L2
aliases:
  - azure-vms-guide
  - vm-diagnostics
  - vm-redeploy
created: 2026-06-25
status:
difficulty:
cert-relevant:
---

# Azure Virtual Machines

---

## Concept Overview
- **What it is**: Azure Virtual Machines (VMs) are on-demand, scalable, virtualized computing resources (IaaS) hosted in Microsoft's global data centers, running operating systems like Windows Server, Windows 10/11, or Linux.
- **Why it matters for a support engineer**: Organizations host servers, databases, and remote desktops (Azure Virtual Desktop) in Azure. Support engineers manage VM lifecycle actions, expand virtual disks, and troubleshoot boot failures or connection drops.
- **Where you encounter this in real job**: Resizing VMs to optimize performance, recovering unresponsive VMs using the Redeploy tool, mounting data disks, and analyzing boot screenshots.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Performs basic power cycles (Start, Stop, Restart), checks VM status, and escalates connection failures.
  - **L2**: Resizes VMs, adds and formats virtual data disks, configures Boot Diagnostics, and redeploys stuck VMs.
  - **L3**: Designs Auto-Scaling rules, manages Azure Dedicated Hosts, configures VM backup retention strategies, and orchestrates migration batches using Azure Migrate.

---

## Technical Deep Dive

### 1. VM Sizing & Families
Azure categorizes VM sizes by workload performance:
- **A-Series / B-Series (Burstable)**: General purpose, budget-friendly. B-series is ideal for workloads (like development servers) that run idle but need occasional high CPU bursts.
- **D-Series (General Purpose)**: Balanced CPU and memory. Used for standard application servers and Active Directory Domain Controllers.
- **F-Series (Compute Optimized)**: High CPU-to-memory ratio. Best for web servers, batch processing, and media encoding.
- **E-Series / M-Series (Memory Optimized)**: High memory-to-CPU ratio. Required for database servers (SQL Server) and SAP HANA.

### 2. Disk Storage Architecture
A standard Azure VM has three default disk categories:

```
                            [Azure Virtual Machine]
                                  /    |    \
                                 /     |     \
                                v      v      v
                          [OS Disk] [Temp Disk] [Data Disk]
```

1. **OS Disk**: Typically the `C:` drive. Stores the operating system. Configured on persistent storage.
2. **Temporary Disk**: Typically the `D:` drive. Local physical storage on the host server. **WARNING**: This disk is ephemeral. Any data saved on the D: drive is permanently lost during host maintenance, redeployment, or deallocation reboots.
3. **Data Disks**: Persistent SCSI disks attached separately (e.g., `E:`, `F:`) to store databases, logs, and files.

#### Storage Disk Tiers:
- **Standard HDD**: Budget magnetic storage for backup/archives.
- **Standard SSD**: Moderate performance for lightweight web servers.
- **Premium SSD**: High-performance, low-latency SSD storage for database VMs.
- **Ultra Disk**: Sub-millisecond latency for heavy enterprise workloads (SQL / SAP).

### 3. VM Extensions & Diagnostic Tools
- **Azure Virtual Machine Agent (VM Agent)**: A lightweight background service installed inside the Guest OS. It manages interactions between the VM and the Azure fabric (e.g., password resets, custom scripts).
- **Boot Diagnostics**: Captures console serial log outputs and a screenshot of the VM's desktop environment during boot. Essential to diagnose BSODs or stuck Windows updates.
- **Redeploy**: An administrative action that shuts down the VM, moves it to a completely different physical hardware host inside the Azure datacenter, and boots it back up. Preserves the OS and Data disks.

---

## Commands & Syntax

### PowerShell
VM management uses the `Az.Compute` module.
```powershell
# Connect to Azure
Connect-AzAccount

# Start a stopped Virtual Machine
Start-AzVM -ResourceGroupName "RG-Prod-Servers" -Name "VM-Web-01"

# Stop and deallocate a Virtual Machine (Force parameter releases billing charges)
Stop-AzVM -ResourceGroupName "RG-Prod-Servers" -Name "VM-Web-01" -Force -Confirm:$false

# Resize a VM to Standard_D4s_v5 size
$VM = Get-AzVM -ResourceGroupName "RG-Prod-Servers" -Name "VM-Web-01"
$VM.HardwareProfile.VmSize = "Standard_D4s_v5"
Update-AzVM -ResourceGroupName "RG-Prod-Servers" -VM $VM

# Redeploy a stuck VM to a new hardware host
Restart-AzVM -ResourceGroupName "RG-Prod-Servers" -Name "VM-Web-01" -Redeploy
```

### Azure CLI
```bash
# Start a Virtual Machine
az vm start --resource-group RG-Prod-Servers --name VM-Web-01

# Stop and deallocate a VM
az vm deallocate --resource-group RG-Prod-Servers --name VM-Web-01

# Redeploy the VM
az vm redeploy --resource-group RG-Prod-Servers --name VM-Web-01
```

### GUI Path
- **Power & Redeploy**: Azure Portal -> Go to **Virtual Machines** -> Select VM -> Overview page -> Click **Start / Stop / Restart** or **Redeploy + reapply** under Help.
- **Diagnostics**: Select VM -> Scroll to Help section -> Click **Boot diagnostics**.

---

## Real-World Scenarios

### Scenario 1: Remote Desktop Connection (RDP) Fails (Unresponsive VM)
**User Complaint:** A developer complains: *"I cannot connect to our testing server 'VM-Dev-04' via RDP. It times out. The Azure Portal says the VM status is 'Running', but I cannot reach it."*
**Your First 3 Checks:**
1. Check the VM's public/private IP connectivity.
2. Open Boot Diagnostics to view the console screenshot and serial log.
3. Check the Network Security Group (NSG) port rules (Port 3389 inbound).
**Diagnosis Steps:**
1. Open Azure Portal -> Go to `VM-Dev-04` -> Scroll down to **Boot diagnostics**.
2. *Review the Screenshot*: It shows the Windows boot screen is stuck at `100% - Working on updates. Don't turn off your PC.`
3. *Why did RDP fail?* The OS is locked updating system files, preventing the RDP network listener service from launching.
4. *Decision*: Do not reboot; doing so during an active update cycle can corrupt the OS.
5. In another scenario, if the screenshot is completely black or showing a crash dump (BSOD), we can use the **Redeploy** action to force the VM onto a healthy host node.
**Root Cause:** The VM Guest OS was unresponsive due to a stuck Windows Update installation process.
**Fix:**
1. Since the update is hung, wait another 30 minutes. If still stuck, initiate a **Redeploy** action:
   - Go to `Redeploy + reapply` -> Click **Redeploy**.
2. Azure shuts down the VM, migrates the virtual disks to a fresh host hypervisor, and restarts it.
3. Once booted, the update completes, and RDP access is restored.
**Prevention:** Schedule Windows Updates using Azure Update Manager during off-hours.
**Ticket Close Note:** "Redeployed VM-Dev-04 to a fresh host. Stuck Windows Update resolved. Verified RDP access is functional. Closed."

### Scenario 2: Data Disappears After VM Deallocation (The Temporary Disk Trap)
**User Complaint:** A database administrator reports: *"I restarted our SQL server 'VM-SQL-Prod' yesterday. Today, our backup directory 'D:\Backups' has completely vanished, along with all the test backup files. The D: drive is completely empty."*
**Your First 3 Checks:**
1. Check the Disk configuration of the VM in the Azure Portal.
2. Review the purpose of the `D:` drive on Azure Virtual Machines.
3. Verify if the files were saved on the ephemeral Temporary Disk.
**Diagnosis Steps:**
1. Log in to the Azure Portal. Go to `VM-SQL-Prod` -> Click **Disks**.
2. Review the list of attached disks:
   - OS Disk: `LUN 0` (`C:`)
   - Data Disk: None.
3. The server administrator saved files on the local `D:` drive.
4. In Azure VMs, the `D:` drive is the default **Temporary Storage** (labeled `Temporary Storage` in File Explorer).
5. When a VM is stopped and deallocated, it releases its physical hardware host. When started again, it boots on a new physical server. The temporary disk is wiped clean during this move.
**Root Cause:** The administrator saved critical files on the ephemeral Temporary Storage disk (`D:`), which was wiped during a deallocation reboot.
**Fix:**
1. Explain to the database administrator that files on the `D:` drive cannot be recovered; it is physical host scratch space.
2. Provision a new persistent Data Disk:
   - Go to VM properties -> **Disks** -> Click **Create and attach a new disk**.
   - Set size to `100GB`, type to `Premium SSD`. Click **Save**.
3. Log in to the VM. Open Disk Management.
4. Format the new disk and assign it a permanent drive letter (e.g., `E:` or `F:`).
5. Reconfigure the SQL backup path to point to the new persistent drive.
**Prevention:** Use Group Policy to hide the `D:` drive or place a `DONT_USE_THIS_DRIVE.txt` warning file in the root directory.
**Ticket Close Note:** "Explained the ephemeral nature of the D: drive. Attached a new 100GB Premium SSD data disk (E:) for persistent backup storage. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never store active databases, system state files, or documents on the `D:` drive (Temporary Storage).
> - This drive is physically local to the hypervisor host server. When the VM is restarted after deallocation or undergoes Azure platform maintenance, **all data on the temporary disk is permanently deleted.**

> [!warning] Common Trap
> - Confusing "Stopping" a VM in the Guest OS with "Deallocating" it in Azure.
> - If you log in to a VM and click Shut Down, the OS stops, but the VM remains "Stopped (Allocated)" in Azure. **You are still billed for the CPU/RAM allocation.** You must stop the VM via the Azure Portal or PowerShell to change the status to "Stopped (Deallocated)" and halt billing.

> [!tip] Senior Engineer Tip
> - If a VM becomes completely unreachable via RDP or SSH and standard resets fail, go to **Redeploy + reapply** and try clicking **Reapply** first. Reapply refreshes the VM connection state metadata with the Azure fabric without restarting the OS. If that fails, run **Redeploy** to move the VM to a healthy host.

> [!success] Verification Steps
> - Run `Get-AzVM -ResourceGroupName "RG-Name" -Name "VM-Name" -Status` to verify the VM's power state is `VM deallocated`.
> - Check that Boot Diagnostics shows a clean Windows logon screen screenshot.

> [!question] Interview Alert
> - "A Windows VM is stuck on boot and you cannot log in. How do you troubleshoot this?"
> - Answer: "I check the **Boot Diagnostics** blade in the Azure Portal. This displays a screenshot of the virtual monitor, which lets me see if the server is performing updates, run checking disk utility, or has crashed with a BSOD. I also review the serial console logs to audit OS services boot phases."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Storing data on the D: drive | Assuming D: is a standard data disk | Use D: only for swap files; attach persistent Premium SSDs for data storage. |
| Paying for idle VMs during weekends | Shutting down inside Guest OS only | Shut down VMs via the Azure Portal or automate deallocation schedules using Azure Automation. |
| Using standard HDDs for SQL database workloads | Seeking lowest disk cost | Always attach Premium SSDs or Ultra Disks for I/O intensive database servers to avoid bottlenecks. |

---

## Lab Exercise

**Objective:** Use PowerShell to query VM status, resize a VM, attach a new virtual data disk, and check boot diagnostics.
**Time Required:** 30 minutes
**Environment Needed:** Azure subscription / Sandbox.
**Pre-requisites:** Azure Az PowerShell module logged in.

**Steps:**
1. Open PowerShell. Get the status of a lab VM:
   ```powershell
   Get-AzVM -ResourceGroupName "RG-Lab" -Name "VM-Test" -Status | Select-Object Name, PowerState
   ```
2. Deallocate the VM to prep for resizing (resizing virtual machines can cause a reboot, so stopping is safest):
   ```powershell
   Stop-AzVM -ResourceGroupName "RG-Lab" -Name "VM-Test" -Force -Confirm:$false
   ```
3. Change the VM size to Standard B2s:
   ```powershell
   $VM = Get-AzVM -ResourceGroupName "RG-Lab" -Name "VM-Test"
   $VM.HardwareProfile.VmSize = "Standard_B2s"
   Update-AzVM -ResourceGroupName "RG-Lab" -VM $VM
   ```
4. Attach a new 50GB Premium SSD data disk:
   ```powershell
   $DiskConfig = New-AzDiskConfig -Location $VM.Location -CreateOption Empty -DiskSizeGB 50 -SkuName Premium_LRS
   $NewDisk = New-AzDisk -DiskName "VM-Test-Data01" -Disk $DiskConfig -ResourceGroupName "RG-Lab"
   $VM = Add-AzVMDataDisk -VM $VM -Name "VM-Test-Data01" -CreateOption Attach -ManagedDiskId $NewDisk.Id -Lun 1
   Update-AzVM -ResourceGroupName "RG-Lab" -VM $VM
   ```
5. Start the VM:
   ```powershell
   Start-AzVM -ResourceGroupName "RG-Lab" -Name "VM-Test"
   ```

**Success Criteria:** The VM is successfully resized, a new 50GB disk is attached, and the VM transitions to the "VM running" state.
**Common Failures:** The resize fails if the target size family (e.g. Standard_B2s) is not available in that specific Azure datacenter region or subscription quota limit is exceeded.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the default temporary disk (usually D: drive) on an Azure VM, and what is its main restriction?**
A: The temporary disk is local storage built directly on the physical host hypervisor server. Its main restriction is that it is ephemeral; if the VM is stopped, deallocated, or moved to a new host during maintenance, all data stored on the temporary disk is permanently deleted.

**Q: What is the difference between "Stopping" a VM in the portal and shutting it down inside the guest OS?**
A: Shutting down inside the guest OS leaves the VM in a "Stopped (Allocated)" state in Azure, meaning the resources are still reserved on the hardware and you are still billed. Stopping the VM in the Azure Portal transitions it to "Stopped (Deallocated)", releasing the hardware reservation and stopping the billing charges.

### Intermediate (L2 Level)
**Q: A VM is running, but you cannot connect to it, and RDP is unresponsive. What Azure diagnostic feature do you check first?**
A: I check the **Boot Diagnostics** blade. This shows me a screenshot of the guest OS screen and the serial log output. It allows me to verify if the VM is hung installing updates, checking the disk, or has encountered a Blue Screen of Death (BSOD), isolating the issue from network or firewall problems.

**Q: What is the "Redeploy" action in Azure, and when would you use it?**
A: The Redeploy action is a troubleshooting tool that shuts down a VM, migrates its virtual disks to a different physical host server inside the Azure data center, and restarts it. I use it when a VM becomes completely unresponsive, shows provisioning errors, or when the underlying physical hypervisor hardware has failed.

### Advanced (L3/Senior Level)
**Q: A critical application VM is showing extremely high disk latency, causing database queries to timeout. The VM is configured with standard SSDs. Describe your resolution steps.**
A:
- **Situation**: Application VM suffering database timeouts due to disk IOPS limits.
- **Task**: Upgrade the disk performance without data loss or prolonged downtime.
- **Action**: First, I analyze the VM's disk performance metrics in Azure Monitor to verify if the disk is hitting its IOPS and throughput caps. I note that Standard SSDs are limited in performance. I deallocate the VM using PowerShell. I go to the disk settings, select the database OS/Data disk, change the SKU from `Standard_SSD_LRS` to `Premium_SSD_LRS`, and select an appropriate size that scales IOPS. Finally, I restart the VM.
- **Result**: Upgrading to Premium SSD increased the IOPS from 500 to 5000, reducing disk latency and resolving the database timeouts.

### HR / Behavioral
**Q: Tell me about a time you had to deal with a critical user error that resulted in system downtime. How did you handle it?**
A: A developer deleted our staging database files because they had saved them on the VM's D: drive, which was wiped when they deallocated the VM over the weekend. They were panicked. I didn't blame them; instead, I explained the ephemeral nature of the D: drive. I checked our recovery vault, located the last snapshot of the OS disk, and restored the database structure. I then attached a persistent Premium SSD data disk for them and set up a GPO to block users from saving files to the D: drive, preventing future errors.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: Scalable, on-demand virtualized computing servers hosted in Azure (IaaS).
> **Why**: Hosts application servers, Active Directory DCs, and virtual desktops.
> **How**: Provision appropriate VM sizes, manage persistent OS/Data disks, avoid the temporary disk trap, and utilize Boot Diagnostics for recovery.
> **Command**: `Restart-AzVM -Redeploy` / `Stop-AzVM -Force`
> **Interview Answer Starter**: "To provision and support Azure VMs, I ensure workloads are sized correctly, map databases to persistent Premium SSDs, and monitor health using Boot Diagnostics..."

**Key Numbers to Remember:**
- Ephemeral Drive letter: `D:` (Temporary Storage)
- Default VM shutdown state that stops billing: Stopped (Deallocated)
- Max IOPS for Premium SSD v2: Up to 80,000 IOPS
- Port required for SSH: TCP 22 / RDP: TCP 3389

**3 Things Interviewer Wants to Hear:**
- Never save persistent data on the D: drive
- Stopping a VM inside the guest OS does not stop billing; it must be deallocated in Azure
- Using Redeploy to resolve virtual machine hypervisor errors

---

## Related Notes
- [[04-Cloud-and-Security/08-Azure/Azure-Identity|Azure Identity]] — Manages the RBAC roles to administer virtual machines.
- [[04-Cloud-and-Security/08-Azure/Azure-Networking|Azure Networking]] — Outlines the virtual subnets and NSG rules connecting VMs.
- [[04-Cloud-and-Security/09-Security/BitLocker-Deep-Dive|BitLocker Deep Dive]] — Discusses VM disk encryption keys backed up to Azure.

---

## Tags
#desktop-support #azure #virtual-machines #infrastructure #L2 #interview-topic #lab-complete #daily-use

