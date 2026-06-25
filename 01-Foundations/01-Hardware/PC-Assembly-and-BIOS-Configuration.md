---
tags: [desktop-support, hardware, pc-assembly, bios, L1]
aliases: [pc-building, bios-setup]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# PC Assembly and BIOS Configuration

---

## Concept Overview
- **What it is**: The physical assembly of computer hardware components (CPU, Motherboard, RAM, Storage, Cooling, PSU) and the subsequent configuration of the system BIOS/UEFI firmware for optimal boot behavior.
- **Why it matters for a support engineer**: A support engineer must know how to build, configure, and optimize systems from individual parts, ensuring that systems pass the Power-On Self-Test (POST) and boot reliably.
- **Where you encounter this in real job**: Building custom workstations for CAD or developers, replacing failed motherboards or CPUs, upgrading memory configurations, and configuring BIOS security profiles.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Assembles components, verifies cable connections, checks POST status, and configures basic boot priorities.
  - **L2**: Troubleshoots POST failure loops, configures advanced storage configurations (RAID/AHCI) in BIOS, and performs BIOS flashing.
  - **L3**: Establishes system configuration standards (Secure Boot profiles, remote management tools), and reviews hardware vendor contracts.

---

## Technical Deep Dive

### 1. Pre-Assembly Checklist & Static Safety
Before assembly, establish a static-safe workspace:
- **ESD Protection**: Electrostatic discharge (ESD) can damage microchip circuits silently. Use an **ESD wrist strap** connected to a grounded metal surface, work on a static-dissipative mat, and avoid working on carpet.
- **Pre-Boot Check**: Verify component compatibility (motherboard socket matches CPU, RAM type matches motherboard specifications, PSU capacity meets the system power budget).

### 2. Component Installation Standards
- **CPU Sockets**:
  - **Intel LGA (Land Grid Array)**: Sockets contain delicate pins. Align the CPU using the side notches and the gold triangle on the corner. Lower the load plate and secure the lever.
  - **AMD PGA/AM5**: PGA sockets (legacy AM4) contain pins on the CPU. The modern AM5 socket uses an LGA design. Ensure the CPU is seated flat before lowering the retention arm.
- **Dual-Channel RAM**: Motherboards trace RAM slots in pairs (typically slots A1, A2, B1, B2). To run in dual-channel mode with two modules, install them in slots **A2 and B2** (slots 2 and 4, counting outward from the CPU socket).
- **M.2 & SATA Storage**: Install M.2 NVMe SSDs into the primary M.2 slot (wired directly to the CPU) at a 30-degree angle, lay it flat, and secure it with the mounting screw or toolless latch. Connect SATA drives to SATA Port 0 or Port 1.

### 3. BIOS/UEFI Configurations
- **Boot Priority**: Configures the sequence in which the system searches storage devices for a bootloader (e.g., Boot 1: NVMe SSD, Boot 2: USB Network Boot PXE).
- **XMP/DOCP**: Memory profiles that must be enabled to configure RAM to its rated speeds (e.g., configuring RAM to run at 3200 MHz instead of the default 2133 MHz).
- **Thermal Profiles (Fan Curves)**: Controls CPU and system fan speeds relative to temperature sensors to balance cooling and noise.

---

## Commands & Syntax

### PowerShell
```powershell
# Retrieve BIOS serial number and current version details
Get-CimInstance -ClassName Win32_BIOS |
    Select-Object SerialNumber, SMBIOSBIOSVersion, Manufacturer, ReleaseDate
```

### CMD / Run Box
```cmd
REM Force the local system to restart directly into the UEFI Firmware settings menu
shutdown /r /fw /t 0
```

### GUI Path
> Start -> Settings -> System -> Recovery -> Advanced Startup -> Click **Restart Now** -> Troubleshoot -> Advanced Options -> **UEFI Firmware Settings**

### Important Registry Paths
```
HKLM\System\CurrentControlSet\Control\SystemInformation\
HKLM\HARDWARE\DESCRIPTION\System\BIOS\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 1 | System time synchronized with time server (RTC check) | System Log |
| 12 | Secure Boot status changed | System Log |
| 41 | System rebooted without clean shutdown (thermal/power off) | System Log |

---

## Real-World Scenarios

### Scenario 1: New Workstation Fails to POST (RAM Warning Code)
**User Complaint:** "I just completed assembling the new developer desktop. When I power it on, the fans spin, but the screen stays black, and the motherboard sounds 3 long beeps repeatedly."
**Your First 3 Checks:**
1. Check the motherboard manual to identify the beep code meaning.
2. Inspect the RAM slot assignments.
3. Check the diagnostic debug LEDs on the motherboard.
**Diagnosis Steps:**
1. Refer to the manual: "3 long beeps = No memory detected or memory initialization failed."
2. Open the case and inspect the RAM. The modules are installed in slots A1 and A2.
3. Check if the modules are fully seated. The lower clips of the RAM slots are open.
**Root Cause:** The RAM modules were not pushed down fully until the retention clips clicked locked, causing a contact error.
**Fix:** Remove the RAM. Reseat the modules into slots **A2 and B2** (slots 2 and 4) and push down until the retaining clips click shut on both sides.
**Prevention:** Always verify that both memory slot clips snap locked during RAM installation.
**Ticket Close Note:** "Resated RAM modules into slots A2 and B2. Verified retention clips locked. System successfully completed POST and booted. Closing ticket."

### Scenario 2: Workstation Freezes During Heavy Processing (Incorrect Fan Curve)
**User Complaint:** "The computer runs fine initially. But when I start compiling code or rendering video, the system freezes or shuts down unexpectedly after 10 minutes."
**Your First 3 Checks:**
1. Check the CPU temperature using diagnostic software (e.g., HWiNFO).
2. Check the CPU fan activity and thermal paste application.
3. Check the BIOS fan control settings.
**Diagnosis Steps:**
1. Monitor temperatures under load: CPU temp climbs rapidly to 100°C and thermal throttles before the system shuts down.
2. Open the BIOS. Navigate to the Hardware Monitor tab.
3. Observe the CPU fan configuration: The fan curve is set to a constant "Silent" profile, spinning at only 20% speed even as CPU temperatures rise.
**Root Cause:** The CPU fan curve was restricted, failing to ramp up fan speeds to dissipate heat under load, triggering a thermal shutdown.
**Fix:** Set the CPU fan curve to a dynamic **Standard** or **Performance** profile in the BIOS, ensuring the fan ramps to 100% speed when the CPU exceeds 75°C.
**Prevention:** Configure thermal warning alarms in the BIOS to alert users before critical thermal limits are reached.
**Ticket Close Note:** "Reconfigured CPU fan curve profile to Standard in UEFI settings. Ran stress test; CPU temperature stabilized at 72°C under full load. Verified no shutdowns. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Drop or slide the CPU into an LGA socket at an angle.
> - LGA socket pins are fragile and easily bent; bending motherboard pins can cause permanent damage to the board.
> - Lower the CPU straight down into the socket, verifying alignment markers before locking the retention frame.

> [!warning] Common Trap
> - Enabling XMP/DOCP profiles on memory without verifying CPU and motherboard compatibility.
> - Running memory frequencies beyond the limits of the motherboard chipset or CPU memory controller causes boot instability and BSOD errors.
> - Verify compatibility on the motherboard's Qualified Vendors List (QVL) first.

> [!tip] Senior Engineer Tip
> - If you cannot access the BIOS because of a forgotten supervisor password, locate the **CLRTC** pins on the motherboard. Shorting these pins with a screwdriver for 10 seconds (with the power cord unplugged) resets the CMOS and clears the password lock.

> [!success] Verification Steps
> - Verify that the system passes POST on every boot.
> - Run: `Get-CimInstance Win32_BIOS` to confirm that the BIOS version matches the latest release.

> [!question] Interview Alert
> - "Explain how to verify if RAM is running in dual-channel mode from within the operating system."
> - Answer: "I open Task Manager, navigate to the Performance tab -> Memory, and check the speed and slot count. For detailed channel analysis, I run tools like CPU-Z or query the system using CIM commands to check the memory bus width (128-bit indicates dual-channel)."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Forgetting Motherboard Standoffs | Direct screw mounting to case | Install brass standoffs in the case holes to prevent ground short-circuits. |
| Leaving plastic cover on heatsink base | Oversight during assembly | Remove the protective plastic film from the cooler block before applying thermal paste. |
| Plug CPU fan to SYS_FAN header | Wrong header selection | Connect the CPU cooler fan directly to the designated `CPU_FAN` motherboard header. |

---

## Lab Exercise

**Objective:** Install a CPU, seat RAM modules in dual-channel configuration, verify POST, and configure boot order settings in UEFI.
**Time Required:** 30 minutes
**Environment Needed:** Anti-static workspace, motherboard, CPU, RAM, CPU cooler, thermal paste, and an active SMPS.
**Pre-requisites:** ESD safety strap active.

**Steps:**
1. Ground yourself. Place the motherboard on a non-conductive surface (e.g., motherboard box).
2. Open the CPU socket load plate. Align the triangle marker on the CPU corner with the socket triangle, seat the CPU, close the plate, and lock the lever.
3. Install two RAM modules into slots **A2 and B2** (slots 2 and 4). Verify the tabs click shut.
4. Apply a pea-sized amount of thermal paste to the CPU center, and install the CPU cooler. Connect the fan to the `CPU_FAN` header.
5. Connect the 24-pin ATX and 8-pin CPU power cables from the PSU.
6. Connect a monitor to the motherboard video port. Short the `PWR_SW` pins to power on.
   - Expected output: Motherboard lights up, fan spins, and the monitor displays "Press Del to enter Setup."
7. Press `Del` to enter BIOS -> Go to **Boot** tab -> Set **Boot Option #1** to your target drive -> Save and Exit (`F10`).
8. Verification: Verify the system boots to the OS installation menu.

**Success Criteria:** System successfully passes POST, memory is detected in dual-channel configuration, and the target drive is set as the primary boot device.
**Common Failures:** Boot failure if power connectors are not fully seated.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What does POST stand for and what is its role?**
A: POST stands for Power-On Self-Test. It is a diagnostic routine run by the BIOS/UEFI immediately after power-on to verify that the CPU, RAM, storage controllers, and graphics card are functioning before booting the operating system.

**Q: How do you enter the BIOS menu on a corporate PC?**
A: During system startup, I press the manufacturer's designated hotkey (typically `F2` or `Delete` for desktops, `F12` for Dell, or `F10` for HP) when the logo screen appears. Alternatively, I can run `shutdown /r /fw /t 0` in Windows command prompt to reboot directly into UEFI settings.

### Intermediate (L2 Level)
**Q: Explain the difference between AHCI and RAID SATA controller modes, and the impact of changing them after OS installation.**
A: AHCI (Advanced Host Controller Interface) enables standard SATA features like hot-plugging and Native Command Queuing (NCQ). RAID mode enables storage virtualization features across multiple drives. Changing this setting in the BIOS *after* the OS is installed causes the OS to load the wrong storage driver, resulting in a boot failure and a blue screen (`INACCESSIBLE_BOOT_DEVICE`).

### Advanced (L3/Senior Level)
**Q: A critical engineering workstation is failing to boot after a user attempted to upgrade the RAM. The system loops on boot and the EZ-Debug LEDs show a solid DRAM light. Explain your diagnostic process.**
A:
- **Situation**: A workstation was stuck in a boot loop with a solid DRAM debug light after a RAM upgrade.
- **Task**: I needed to identify the failure point and restore the system to service.
- **Action**: I disconnected the power cord, opened the chassis, and verified that the RAM modules were installed in the correct slots (A2 and B2) and fully locked. I removed the new RAM and tested the machine with only a single original RAM module. I then cleared the CMOS settings to reset timings to factory defaults. I verified that the new RAM part number was listed on the motherboard QVL list.
- **Result**: The CMOS reset cleared unstable custom memory configurations, allowing the system to boot and recognize the new RAM capacity.

### HR / Behavioral
**Q: Tell me about a time you had to assemble or upgrade multiple systems under a tight deadline.**
A: We had to prepare 20 custom developer workstations within 48 hours. I set up an assembly line workflow, pre-configuring a single BIOS profile with PXE boot enabled. I assembled the hardware, booted the systems, and initiated simultaneous network OS deployments, completing the project on time.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The physical assembly of computer hardware and configuration of system BIOS/UEFI firmware.
> **Why**: Ensures hardware is installed correctly and optimized for stable OS boot behavior.
> **How**: Reseat components (RAM, CPU), configure boot order, and enable security features (Secure Boot, TPM).
> **Command**: `shutdown /r /fw /t 0`
> **Interview Answer Starter**: "In my experience, PC assembly requires static-safe procedures, followed by optimizing the BIOS settings for Secure Boot and TPM..."

**Key Numbers to Remember:**
- CMOS battery voltage: 3V
- Memory Dual-Channel slots: A2 & B2 (slots 2 and 4)
- CPU Fan header: CPU_FAN (supplies variable power based on temperature)

**3 Things Interviewer Wants to Hear:**
1. ESD protection habits (anti-static wrist strap usage)
2. Motherboard slot assignment strategies for memory and storage
3. Core UEFI security settings (Secure Boot, TPM 2.0 configuration)

---

## Related Notes
- [[01-Foundations/01-Hardware/Motherboard-Architecture|Motherboard Architecture]] — Connects all assembled components.
- [[01-Foundations/01-Hardware/Storage-Deep-Dive|Storage Deep Dive]] — Details drive installation and RAID setups.
- [[01-Foundations/01-Hardware/Hardware-Troubleshooting-Masterclass|Hardware Troubleshooting Masterclass]] — Troubleshooting flows for boot failures.

---

## Tags
#desktop-support #hardware #pc-assembly #bios #L1 #interview-topic #lab-complete #daily-use
