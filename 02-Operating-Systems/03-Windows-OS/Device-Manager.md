---
tags: [desktop-support, windows-os, device-manager, drivers, L1]
aliases: [driver-troubleshooting, hardware-devices]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# Device Manager

---

## Concept Overview
- **What it is**: Windows Device Manager is a built-in administrative tool that provides a graphical representation of the physical and virtual hardware devices connected to a computer, allowing administrators to manage drivers, change hardware properties, and troubleshoot device errors.
- **Why it matters for a support engineer**: Operating system interaction with physical hardware relies on device drivers. A support engineer must know how to diagnose driver errors, roll back updates that cause instability, and locate drivers for unrecognized hardware.
- **Where you encounter this in real job**: Troubleshooting network adapter failures (code 10), resolving USB controller disconnects (code 43), identifying unrecognized hardware components (code 28), and rolling back graphics drivers.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Identifies basic driver errors (yellow warning flags), updates drivers from local folders, and performs basic device resets.
  - **L2**: Troubleshoots driver conflicts, locates driver packages using hardware IDs, and rolls back updates.
  - **L3**: Establishes driver deployment baselines for enterprise OS deployments, configures Driver Signature Enforcement policies, and manages driver updates via Intune.

---

## Technical Deep Dive

### 1. The Hardware Abstraction Layer (HAL) & Drivers
Windows does not communicate with hardware directly. The HAL acts as a translation layer. The **Device Driver** is the translator script. If the driver is missing, outdated, or corrupt, the translator fails, and the operating system cannot control the hardware.

### 2. Common Device Manager Error Codes
When a device fails to initialize, Device Manager displays a yellow warning icon next to the device. Double-clicking the device reveals the specific error code:
- **Code 10 (This device cannot start)**: The device failed to initialize. Commonly caused by outdated, incompatible, or corrupt drivers.
- **Code 28 (The drivers for this device are not installed)**: The device is unrecognized. Typically occurs after a clean OS install when chipset or custom device drivers are missing.
- **Code 43 (Windows has stopped this device because it has reported problems)**: The hardware device or its driver reported a physical failure to the operating system. Can indicate a loose connection, power issue, or hardware fault.

### 3. Driver Signature Enforcement
To prevent rootkits and kernel-level malware from exploiting hardware, Windows enforces **Driver Signature Enforcement**. Only drivers signed with digital certificates from Microsoft's Windows Hardware Quality Labs (WHQL) can be loaded into memory. Unsigned drivers are blocked automatically during boot.

---

## Commands & Syntax

### PowerShell
```powershell
# Retrieve all network adapters that have a non-healthy status in Device Manager
Get-PnpDevice -Class "Net" | Where-Object {$_.Status -ne "OK"} |
    Select-Object FriendlyName, InstanceId, Status, Problem

# Install a local driver package using pnputil.exe via PowerShell
pnputil.exe /add-driver "C:\Drivers\network.inf" /install
```

### CMD / Run Box
```cmd
REM Launch the Device Manager console directly
devmgmt.msc
REM List all active third-party drivers installed on the system
pnputil /enum-drivers
```

### GUI Path
> Start -> Run -> type `devmgmt.msc`
> Or: Control Panel -> Hardware and Sound -> **Device Manager**

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Control\Class\
HKLM\SYSTEM\CurrentControlSet\Enum\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 20001 | Driver package installation started | System Log |
| 20003 | Service added for device installation (driver service registered) | System Log |
| 411 | Device failed to start (matches Code 10/43) | System Log |

---

## Real-World Scenarios

### Scenario 1: Workstation Has No Network Connection After Windows Update (Driver Roll Back)
**User Complaint:** "I restarted my computer to apply Windows updates, and now my internet connection is gone. The network icon shows a red X."
**Your First 3 Checks:**
1. Check the network adapter status in Device Manager.
2. Inspect the link lights on the physical NIC port.
3. Check the Windows Update history logs.
**Diagnosis Steps:**
1. Open Device Manager (`devmgmt.msc`).
2. Locate **Network adapters** -> Double-click **Intel(R) Ethernet Connection**.
   - Status: `This device cannot start. (Code 10)`
3. Go to the **Driver** tab -> Check the **Driver Date** and **Driver Version**.
   - The driver was updated yesterday by Windows Update, replacing the OEM driver.
**Root Cause:** A generic driver pushed via Windows Update was incompatible with the specific NIC chipset revision, causing a startup failure (Code 10).
**Fix:** Roll back the driver to the previous working version:
1. Double-click the network adapter in Device Manager.
2. Navigate to the **Driver** tab.
3. Click **Roll Back Driver** -> Select a reason -> Click **Yes**.
**Prevention:** Block driver updates from being distributed via Windows Update using Group Policy.
**Ticket Close Note:** "Diagnosed Code 10 on Intel NIC caused by Windows Update driver override. Rolled back driver to the previous stable version. Connection restored. Closing ticket."

### Scenario 2: Unrecognized Device in Device Manager (Hardware ID Verification)
**User Complaint:** "I installed a new PCIe expansion card, but it doesn't work, and Device Manager just shows a generic 'PCI Device' with a yellow question mark."
**Your First 3 Checks:**
1. Verify if the device displays Code 28 (Drivers not installed).
2. Retrieve the Hardware IDs from the device properties.
3. Check if the device requires external power from the SMPS.
**Diagnosis Steps:**
1. Open Device Manager -> Locate the generic **PCI Device** (under Other Devices).
   - Status: `The drivers for this device are not installed. (Code 28)`
2. Right-click the device -> **Properties** -> Navigate to the **Details** tab.
3. Select **Hardware Ids** from the Property dropdown menu.
   - Output: `PCI\VEN_10EC&DEV_8168&SUBSYS_816810EC`
   - Vendor ID (VEN): `10EC` (Realtek Semiconductor)
   - Device ID (DEV): `8168` (RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller)
**Root Cause:** The system lacked the required controller driver, preventing the OS from recognizing the expansion card.
**Fix:** Search the Vendor and Device IDs on PCI Lookup databases, download the driver package from the manufacturer's site, and install it.
**Prevention:** Maintain a centralized repository of approved driver packages for all deployed workstation hardware.
**Ticket Close Note:** "Identified unrecognized device as Realtek RTL8168 NIC using Hardware IDs. Downloaded and installed the matching driver package. Device active. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Install driver packages downloaded from third-party driver utility websites or unverified download portals.
> - These utilities often bundle adware, spyware, or outdated drivers that cause system instability or compromise endpoint security.
> - Only download drivers from the official hardware manufacturer's support site.

> [!warning] Common Trap
> - Attempting to force-install a 32-bit (x86) driver package on a 64-bit (x64) Windows operating system.
> - The installation will fail, or if forced via compatibility bypass, will cause a critical kernel-level blue screen (`SYSTEM_THREAD_EXCEPTION_NOT_HANDLED`).
> - Always match the driver architecture to the operating system version.

> [!tip] Senior Engineer Tip
> - If a device is hidden in Device Manager (e.g., disconnected USB controllers), click **View** -> **Show hidden devices** on the top menu bar to display greyed-out legacy devices, allowing you to uninstall clean-up configurations.

> [!success] Verification Steps
> - Run the command: `Get-PnpDevice -InstanceId "instance-id"` to check status.
> - Confirm the status reads **OK** and the problem code is 0.

> [!question] Interview Alert
> - "How do you identify an unrecognized hardware device in Device Manager if no driver is installed?"
> - Answer: "I double-click the unrecognized device, navigate to the Details tab, select 'Hardware Ids' from the dropdown, and note the Vendor (VEN) and Device (DEV) codes. I then search these codes in public PCI lookup databases to identify the manufacturer and model to download the correct driver."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Uninstalling devices blindly | Attempting to fix configuration errors | Use the **Update Driver** or **Roll Back** options first before completely removing device registrations. |
| Forgetting to unplug hardware | Swapping internal cards on live systems | Shut down and unplug the system before physical hardware installations. |
| Installing unsigned drivers on production | Bypassing driver signature enforcement | Use WHQL-certified signed drivers to prevent Windows security blocks. |

---

## Lab Exercise

**Objective:** Locate an active device, identify its Hardware and Vendor IDs, export its driver package using PowerShell, and verify signature status.
**Time Required:** 20 minutes
**Environment Needed:** A Windows 11 VM or physical machine with local administrator rights.
**Pre-requisites:** Administrative access.

**Steps:**
1. Open PowerShell as Administrator.
2. Query the primary network adapter device details:
   ```powershell
   Get-PnpDevice -Class "Net" | Select-Object FriendlyName, InstanceId, Status
   ```
3. Copy the `InstanceId` of the network adapter.
4. Retrieve the Hardware IDs:
   ```powershell
   (Get-PnpDeviceProperty -InstanceId "[InstanceId]" -KeyName "DEVPKEY_Device_HardwareIds").Value
   ```
   - Expected output: A string containing the `VEN` (Vendor) and `DEV` (Device) identifiers.
5. Export the active driver package from the local Windows driver store to a backup folder:
   ```powershell
   Export-WindowsDriver -Online -Destination "C:\Backups\Drivers"
   ```
6. Verification: Check the folder `C:\Backups\Drivers` to confirm the `.inf` and `.sys` files exist.
7. Open Device Manager -> Locate the adapter -> Right-click **Properties** -> Verify the driver is active and healthy.

**Success Criteria:** The adapter's Hardware IDs are successfully queried, and the active driver is exported to the backup directory.
**Common Failures:** Access denied errors if PowerShell is not run with elevated administrator privileges.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the meaning of a yellow warning triangle next to a device in Device Manager?**
A: A yellow warning triangle indicates that the device has a configuration error or driver issue that prevents it from functioning. Double-clicking the device opens the properties menu, showing the specific error code (e.g., Code 10, 28, or 43).

**Q: How do you roll back a device driver in Windows?**
A: I open Device Manager, double-click the target device to open its **Properties**, navigate to the **Driver** tab, and click the **Roll Back Driver** button, which restores the previously installed driver version.

### Intermediate (L2 Level)
**Q: What is the difference between a device driver and the BIOS?**
A: The BIOS is firmware stored on a motherboard chip that initializes basic hardware (POST) during boot. A device driver is software installed within the operating system that enables the OS to communicate with and control specific hardware components after boot.

### Advanced (L3/Senior Level)
**Q: A critical network adapter on a production server displays Code 10. The driver is up to date, and rolling back is grayed out. Explain your diagnostic and resolution process.**
A:
- **Situation**: A production server's network adapter failed to start with a Code 10 error, and no roll back option was available.
- **Task**: I needed to identify the root cause and restore the network interface.
- **Action**: I checked the Event Viewer System log and found Event ID 411 stating that the device driver failed to load due to a system resource conflict. I opened Registry Editor and checked the device class registry key:
   `HKLM\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}`.
   I checked for any corrupt `UpperFilters` or `LowerFilters` values. I removed the invalid filter entries and rebooted the server.
- **Result**: The driver loaded successfully, the Code 10 warning cleared, and the network connection was restored.

### HR / Behavioral
**Q: Tell me about a time you had to deal with an urgent user issue caused by a bad software or driver update. How did you handle it?**
A: An update caused several laptops to lose display output. I booted the affected laptops into Safe Mode, rolled back the display driver, and configured a policy to block the specific update until a patch was released.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The Windows utility used to view and manage physical and virtual hardware devices and drivers.
> **Why**: Critical for diagnosing connection errors, resolving driver conflicts, and configuring hardware.
> **How**: Identify yellow warning indicators, verify error codes (Code 10/28/43), and roll back unstable updates.
> **Command**: `devmgmt.msc`
> **Interview Answer Starter**: "In my experience, driver troubleshooting in Device Manager starts by checking the device status properties and looking at the Hardware IDs to find missing drivers..."

**Key Numbers to Remember:**
- Code 10: Device cannot start (driver issue)
- Code 28: Driver not installed (missing driver)
- Code 43: Device reported problems (hardware or connection failure)

**3 Things Interviewer Wants to Hear:**
1. Using Hardware IDs to identify unrecognized devices
2. Performing driver rollbacks in Safe Mode
3. Managing corporate driver updates (preventing generic overrides)

---

## Related Notes
- [[01-Foundations/01-Hardware/Motherboard-Architecture|Motherboard Architecture]] — Connects physical interfaces under driver control.
- [[01-Foundations/01-Hardware/Hardware-Troubleshooting-Masterclass|Hardware Troubleshooting Masterclass]] — Diagnostic flows for component failures.
- [[02-Operating-Systems/03-Windows-OS/Registry|Registry]] — Details device class configuration keys.

---

## Tags
#desktop-support #windows-os #device-manager #drivers #L1 #interview-topic #lab-complete #daily-use
