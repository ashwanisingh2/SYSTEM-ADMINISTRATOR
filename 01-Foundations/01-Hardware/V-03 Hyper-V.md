---
tags: [sysadmin, virtualization, hyper-v, comparison]
difficulty: Intermediate
lab-required: Yes
read-time: 10 mins
---

# V-03: Hyper-V Virtualization Comparison and PowerShell Management

> [!abstract] Overview
> Advanced content only — basics in [[03-Identity-and-Core-Services/05-Windows-Server/WS-12 Hyper-V Virtualization|WS-12 Hyper-V Virtualization]]. This note covers Hyper-V client-side deployment, hypervisor comparisons, nested virtualization enabling processes, and core PowerShell cmdlets.

---
## Concept
Think of Hyper-V on your Windows 10/11 client laptop like having a built-in virtual apartment manager that resides in the motherboard itself. 
Unlike VirtualBox or VMware Workstation, which act as software apps you install on top of Windows, client Hyper-V turns your host OS into a manager partition running directly on the CPU virtualization layers on boot. 

- **Nested Virtualization** is like building a guest house inside a guest house: you rent out a room (a VM), and allow the tenant to rent out their closets as smaller rooms (running containers or VMs inside the VM).

*Seedha simple mein: Windows 10/11 Pro par Hyper-V use karne ke liye hum use Windows Features se enable karte hain. PowerShell cmdlets (Get-VM, Start-VM) and nested virtualization extensions hyper-v automation and nested labs ko enable karte hain.*

---
## Technical Deep Dive

### 1. Hypervisor Comparison Matrix

| Feature | Hyper-V (Client/Server) | VMware Workstation Pro | Oracle VM VirtualBox |
|---|---|---|---|
| **Hypervisor Type** | **Type 1** (Bare-Metal) | **Type 2** (Hosted) | **Type 2** (Hosted) |
| **Host Integration** | Deeply integrated into Windows OS kernel | Independent application | Independent application |
| **OS Compatibility** | Windows only | Windows, Linux | Windows, Linux, macOS, Solaris |
| **Direct Hardware Access**| Yes (VMBus synthetic drivers) | No (Translated via Host OS) | No (Translated via Host OS) |
| **Hyper-V Conflict** | N/A (Is the primary hypervisor) | Yes (Requires nested/CSM bypass) | Yes (Requires nested/CSM bypass) |

### 2. Enabling Client Hyper-V on Windows 10/11 Pro/Enterprise
Client Hyper-V is built into Windows 10/11 Pro, Enterprise, and Education editions (Not Home edition).
- **Enabling via GUI:** Open `optionalfeatures.exe` -> Check **Hyper-V** (Hyper-V Platform & Hyper-V Management Tools). Click OK and reboot.
- **Enabling via CLI (PowerShell Admin):**
  ```powershell
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All -LimitAccess -Restart
  ```

### 3. Nested Virtualization (Exposing Hardware virtualization)
Nested virtualization allows you to run Hyper-V inside a guest virtual machine (e.g., running Docker containers or Hyper-V VMs inside a VM).
- **Requirements:** Intel or AMD CPU with nested virtualization support on the host, Hyper-V VM Generation 2, and VM configuration modifications while the VM is stopped.
- **Enabling command:**
  ```powershell
  Set-VMProcessor -VMName "MyLabDC" -ExposeVirtualizationExtensions $true
  ```

---
## PowerShell for Hyper-V Cmdlets Reference

### VM Monitoring
```powershell
Get-VM                            # List all virtual machines on the host
Get-VM -Name "Prod-Web" | Format-List * # Show detailed parameters of a specific VM
Get-VMNetworkAdapter -VMName *    # List MAC and IP addresses of all VMs
```

### VM Control
```powershell
Start-VM -Name "Prod-Web"         # Start VM
Stop-VM -Name "Prod-Web" -Force   # Force stop (power off) VM
Stop-VM -Name "Prod-Web" -Save    # Suspend VM and save active memory state
Restart-VM -Name "Prod-Web"       # Reboot VM
```

### VM Creation & Configuration
```powershell
# Create a new virtual switch
New-VMSwitch -Name "Internal_Switch" -SwitchType Internal

# Configure VM memory properties dynamically
Set-VMMemory -VMName "Prod-Web" -DynamicMemoryEnabled $true -MinimumBytes 512MB -StartupBytes 2GB -MaximumBytes 4GB

# Set CPU Core allocation
Set-VMProcessor -VMName "Prod-Web" -Count 4
```

### Checkpoints Management
```powershell
Checkpoint-VM -Name "Prod-Web" -SnapshotName "Pre-Patch_Backup" # Create checkpoint
Get-VMSnapshot -VMName "Prod-Web"                               # List active checkpoints
Restore-VMSnapshot -VMName "Prod-Web" -Name "Pre-Patch_Backup"   # Revert VM to checkpoint
Remove-VMSnapshot -VMName "Prod-Web" -Name "Pre-Patch_Backup"    # Delete and merge checkpoint
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows 11 Pro physical machine with Hyper-V enabled, and an active PowerShell console running as Administrator.

### Step 1: Verify Hyper-V Requirements
1. Open PowerShell as Administrator.
2. Check if virtualization features are active:
   ```cmd
   systeminfo
   ```
3. Scroll to the bottom. Under "Hyper-V Requirements," confirm that **Virtualization Enabled In Firmware** reads **Yes**.

### Step 2: Enable Nested Virtualization
1. Create a template VM named `HyperV_Host_VM` in Hyper-V. Stop the VM.
2. Run the following command in PowerShell on your host machine to expose CPU virtualization to the guest VM:
   ```powershell
   Set-VMProcessor -VMName "HyperV_Host_VM" -ExposeVirtualizationExtensions $true
   ```
3. Enable MAC Address Spoofing (Required if the nested guest VMs need network access through the parent VM's adapter):
   ```powershell
   Get-VMNetworkAdapter -VMName "HyperV_Host_VM" | Set-VMNetworkAdapter -MacAddressSpoofing On
   ```
4. Start the VM. You can now install Hyper-V and run nested VMs inside `HyperV_Host_VM`.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Attempting to enable the Hyper-V optional feature on a laptop running Windows 11 Home edition fails. The option is missing from the features GUI.
- **Root Cause:** Microsoft restricts Hyper-V platform activation to Pro, Enterprise, and Education SKU editions.
- **Fix:**
  1. Upgrade the laptop operating system to Windows 11 Pro.
  2. Alternatively, if a upgrade is not possible, use a batch script to force-install the packages from the Windows component packages payload repository (unsupported but functional):
     ```cmd
     dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\Microsoft-Hyper-V-Hypervisor-Package~31bf3856ad364e35~amd64~~.mum"
     :: (Followed by all other Hyper-V packaging files, then reboot).
     ```

**Scenario 2:**
- **Problem:** After enabling Hyper-V, standard emulator tools (like Android Studio Emulator or Nox Player) fail to launch or report "Virtualization engine is locked."
- **Root Cause:** Conflict of hypervisors. Hyper-V runs as a Type 1 hypervisor on boot, preventing Android emulators (which expect Type 2 access) from accessing the VT-x registers.
- **Fix:**
  1. Configure Android Studio to use the Hyper-V Hypervisor Platform backend.
  2. Alternatively, create a boot configuration menu option to disable Hyper-V temporarily when running emulators:
     ```cmd
     bcdedit /copy {current} /d "Windows 11 (No Hyper-V)"
     :: Copy the GUID returned and run:
     bcdedit /set {GUID} hypervisorlaunchtype off
     ```
  3. Reboot the machine and select the "No Hyper-V" boot option when working with emulators.

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-12 Hyper-V Virtualization|WS-12 Hyper-V Virtualization]] — Core Hyper-V architecture and server setup.
- [[01-Foundations/01-Hardware/V-01 VMware Workstation Complete Guide|V-01 VMware Workstation Complete Guide]] — Comparison with VMware Workstation.
- [[01-Foundations/01-Hardware/V-04 Virtualization Lab Environment Setup|V-04 Virtualization Lab Environment Setup]] — Sizing multi-VM virtual environments.

