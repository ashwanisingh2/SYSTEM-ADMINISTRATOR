---
tags: [sysadmin, hardware, motherboard]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# H-01: Motherboard Architecture

> [!abstract] Overview
> This note covers the physical and logical architecture of modern motherboards, detailng form factors, chipsets, BIOS/UEFI firmware, CMOS, expansion slots, and internal connectors. Understanding these components is critical for hardware selection, system building, and low-level troubleshooting.

---
## Concept
Think of the motherboard as the central nervous system and skeletal structure of a city. The CPU is the city hall (the brain), but the motherboard provides the roads, electrical grid, and zoning laws. 

The roads (data buses/traces) connect different districts (RAM, GPU, storage), the power lines supply electricity to each building through specific transformers (power phases/VRMs), and the security gates (BIOS/UEFI) control who gets to enter the city when it boots up.

*Seedha simple mein: Motherboard ek bada junction board hai jahan computer ka har ek part aakar connect hota hai. Agar CPU brain hai, toh motherboard body ki arteries aur veins hai jo sabhi parts ko aapas mein baat karne aur power share karne ki permission deti hai.*

---
## Technical Deep Dive

### 1. Motherboard Form Factors
Motherboard form factors dictate the physical size, mounting hole locations, power supply compatibility, and expansion slot capacity.

| Form Factor | Dimensions | Expansion Slots (Max) | RAM Slots (Typical) | Best Use Case |
|---|---|---|---|---|
| **ATX (Standard)** | 12" × 9.6" (305 × 244 mm) | 7 | 4 to 8 | Standard Desktops, Workstations |
| **Micro-ATX (mATX)** | 9.6" × 9.6" (244 × 244 mm) | 4 | 2 to 4 | Budget builds, compact office PCs |
| **Mini-ITX (ITX)** | 6.7" × 6.7" (170 × 170 mm) | 1 | 2 | Small Form Factor (SFF) builds, HTPCs |

*Note: Standoff placement varies by form factor. Installing an ATX motherboard into an ATX-compatible case requires checking that standoffs align perfectly to prevent short circuits.*

### 2. Chipset Architecture (Northbridge vs. Southbridge to Modern SoC)
In older architectures, the motherboard featured a **Northbridge** (controlling high-speed components: CPU, RAM, AGP/PCIe GPU) and a **Southbridge** (controlling slower I/O: SATA, PCI, USB, Audio). 

Modern processors have integrated the Northbridge onto the CPU die itself (System-on-Chip or SoC). The remaining Southbridge features are handled by a single chip on the motherboard, known as the **PCH (Platform Controller Hub)** on Intel or the **FCH (Fusion Controller Hub)/Chipset** on AMD.

- **Intel Chipsets (e.g., Z790, H770, B760):**
  - **Z-Series:** Fully featured, supports CPU and RAM overclocking, maximum PCIe lanes.
  - **B-Series:** Mid-range, supports RAM overclocking (on newer gens) but no CPU overclocking.
  - **H-Series:** Entry-level, locked down, minimal PCIe lanes.
- **AMD Chipsets (e.g., X670E, B650, A620):**
  - **X-Series:** Dual-chipset designs on AM5, maximum PCIe Gen 5 connectivity, CPU/RAM overclocking.
  - **B-Series:** Mainstream, CPU and RAM overclocking support, balanced I/O.
  - **A-Series:** Entry-level, no CPU overclocking, restricted I/O features.

### 3. BIOS vs. UEFI Deep Dive
Firmware initializes hardware during the power-on sequence and boots the operating system.

- **Legacy BIOS (Basic Input/Output System):**
  - Operates in 16-bit real mode.
  - Limits boot drives to **MBR (Master Boot Record)** partitions (maximum 2.2 TB, max 4 primary partitions).
  - Uses a basic text-based blue/gray interface.
- **UEFI (Unified Extensible Firmware Interface):**
  - Operates in 32-bit or 64-bit mode, allowing GUI interfaces, mouse support, and network connectivity.
  - Boot drives use **GPT (GUID Partition Table)** (supports up to 9.4 ZB, virtually unlimited partitions).
  - **Secure Boot:** Cryptographically verifies the bootloader, kernel, and drivers using pre-installed keys to prevent rootkits and boot-time malware.
  - **TPM 2.0 (Trusted Platform Module):** A dedicated microcontroller securing hardware by integrating cryptographic keys. Required for Windows 11. Can be physical (discrete dTPM) or firmware-based (Intel PTT or AMD fTPM).

### 4. CMOS (Complementary Metal-Oxide-Semiconductor)
The CMOS chip is a small volatile RAM memory block on the motherboard that stores BIOS/UEFI configuration settings (date, time, boot order, CPU/RAM frequency settings).
- **Power Source:** A CR2032 lithium coin-cell battery (3V) supplies continuous power when the computer is shut down and unplugged.
- **Signs of Depletion:** System clock resets to midnight on every reboot; "CMOS Checksum Error" on POST.

### 5. PCIe Slots (PCI Express)
PCIe uses serial point-to-point connections called "lanes." Each lane consists of two pairs of wires (one for sending, one for receiving).
- **Physical Sizes:** x1, x4, x8, x16.
- **Compatibility:** Upward and downward compatible. A physical x16 card can run in a physical x16 slot wired for x8 (electrical configuration). A physical x1 card can fit into a physical x16 slot.
- **Generations and Speeds (Per Lane, Unidirectional):**
  - Gen 3: ~985 MB/s
  - Gen 4: ~1.97 GB/s
  - Gen 5: ~3.94 GB/s
  - *Example:* A PCIe Gen 4 x16 slot has a theoretical maximum bandwidth of ~31.5 GB/s.

### 6. Motherboard Connectors
- **24-pin ATX Power:** Main motherboard power delivery (provides 3.3V, 5V, and 12V rails).
- **8-pin (4+4) EPS/CPU Power:** Dedicated 12V power supply to the CPU VRMs (Voltage Regulator Modules).
- **SATA (Serial ATA):** 6 Gbps port for connecting legacy HDDs and 2.5-inch SSDs.
- **M.2 Slots:** Compact slots for NVMe/SATA SSDs. May support PCIe lanes routed directly from the CPU or via the chipset (PCH).
- **USB Headers:** Internal pins connecting chassis front-panel USB 2.0, USB 3.2 Gen 1 (19-pin), and USB 3.2 Gen 2 Type-C (Key-A Type-E) ports.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A physical computer system (Intel or AMD) to access the UEFI/BIOS settings.

### Step 1: Accessing BIOS/UEFI
1. Shut down the computer completely.
2. Press the power button. Immediately tap the BIOS hotkey repeatedly. Common hotkeys:
   - **Del** (ASUS, Gigabyte, MSI)
   - **F2** (ASrock, Dell, laptops)
   - **F12** or **F10** (HP)
3. Confirm you have entered the BIOS/UEFI interface.

### Step 2: Configure Boot Order
1. Navigate to the **Boot** tab using keyboard arrow keys or mouse.
2. Locate **Boot Option Priorities**.
3. Select **Boot Option #1** and change it to the installation media (e.g., UEFI USB Flash Drive).
4. Select **Boot Option #2** and set it to the primary system storage drive.

### Step 3: Enable XMP / DOCP / EXPO
Advanced content only — basics in [[RAM types DDR3/DDR4/DDR5]]

1. Go to the **Extreme Tweaker**, **AI Overclock Tuner**, or **OC** section.
2. Locate the memory profile settings:
   - Intel: Enable **XMP (Extreme Memory Profile)**.
   - AMD AM4: Enable **DOCP (Direct Overclock Profile)** or **A-XMP**.
   - AMD AM5: Enable **EXPO (Extended Profiles for Overclocking)**.
3. Select Profile 1. Verify that memory frequency, timings, and voltage adjust to match the manufacturer's rated speed.

### Step 4: Save and Exit
1. Press **F10**.
2. Review the change log prompt showing the modifications made (e.g., Boot order updated, XMP enabled).
3. Confirm by selecting **Yes** to reboot the system.

---
## Commands Reference
No OS commands directly configure physical motherboard jumpers, but system information tools query properties:

```bash
# Windows (PowerShell)
Get-CimInstance Win32_BaseBoard | Format-List Product, Manufacturer, SerialNumber, Version  # Get motherboard hardware details
Get-CimInstance Win32_Bios | Format-List Name, Version, SMBIOSBIOSVersion                   # Check BIOS/UEFI firmware version
Confirm-SecureBootUEFI                                                                      # Verify if Secure Boot is active (True/False)

# Linux (Bash)
sudo dmidecode -t baseboard   # Read motherboard (baseboard) details from SMBIOS
sudo dmidecode -t bios        # Read BIOS version and features
[ -d /sys/firmware/efi ] && echo "UEFI" || echo "Legacy BIOS"  # Determine boot mode
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** The computer power-on cycle begins (fans spin, lights turn on), but the monitor remains blank and there is no POST (Power-On Self-Test).
- **Root Cause:** Corrupt or unstable BIOS settings (e.g., failed RAM overclocking/incorrect memory timings, incompatible hardware configuration).
- **Fix:** Clear the CMOS to reset the BIOS settings to factory defaults.
  1. Shut down the system, unplug the AC power cord from the SMPS, and hold the power button for 15 seconds to discharge internal capacitors.
  2. Locate the **CLRTC** / **JBAT1** jumper pins on the motherboard.
  3. Use a flathead screwdriver or jumper cap to short the two pins together for 10-15 seconds.
  4. Alternatively, remove the CR2032 coin-cell battery for 5 minutes, then reinstall it.
  5. Reconnect AC power, power on the system, and confirm the system boots with default BIOS settings.

**Scenario 2:**
- **Problem:** A newly installed NVMe PCIe Gen 4 SSD performs at half its rated speed (e.g., 3500 MB/s instead of 7000 MB/s).
- **Root Cause:** The SSD is installed in an M.2 slot routed through the Chipset configured at PCIe Gen 3 speeds, rather than the primary Gen 4 slot routed directly to the CPU.
- **Fix:**
  1. Refer to the motherboard user manual to identify the physical lane routing of each M.2 slot.
  2. Shut down the PC, discharge static electricity, and open the chassis.
  3. Relocate the NVMe drive to the top M.2 slot (closest to the CPU socket), which provides dedicated Gen 4 x4 PCIe lanes directly from the CPU.
  4. Boot the system and verify speeds with CrystalDiskMark.

---
## Common Mistakes
> [!warning] Avoid These
> **Forgetting to check standoff alignment:** Screwing a motherboard directly onto a case tray without brass standoffs, or leaving misaligned standoffs from an old build. This causes the PCB traces to short against the metal chassis, preventing POST or permanently damaging components.
> **Correct approach:** Always visually align motherboard screw holes with case standoffs. Remove any unused standoffs from the tray before installing the motherboard.

---
## Pro Tips
> [!tip] Field Experience
> Before updating a motherboard's BIOS in an enterprise or production environment, always export or write down custom RAID, storage controller (AHCI/NVMe), and Secure Boot settings. A BIOS flash clears these configurations, and booting a system with mismatched storage controller modes can cause BSODs or temporary array offline states.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | ATX vs. ITX | ATX is the standard size with full expansion; ITX is compact for SFF systems with one PCIe slot. |
| 2 | PCH/Chipset | Silicon controller acting as the hub for SATA, USB, audio, and chipset-connected PCIe lanes. |
| 3 | BIOS vs. UEFI | Legacy BIOS is 16-bit MBR-bound; UEFI is modern, 32/64-bit, supports GPT, TPM 2.0, and Secure Boot. |
| 4 | CMOS Reset | Done by shorting CLRTC pins or removing the battery to reset volatile BIOS settings to defaults. |
| 5 | PCIe Lanes | High-speed serial point-to-point buses where throughput doubles with each successive generation. |

---
## Interview Q&A

**Q1: What is the difference between a PCIe lane routed to the CPU versus one routed to the PCH/Chipset?**
A: CPU-routed PCIe lanes connect directly to the processor's PCIe controller, offering the lowest possible latency and maximum bandwidth. They are reserved for high-performance components like the primary GPU (slot 1) and primary NVMe drives. Chipset-routed lanes share a multiplexed uplink (e.g., DMI link on Intel platforms) to the CPU. If multiple high-speed devices (SATA controllers, secondary M.2 SSDs, NICs) connected to the chipset transmit data simultaneously, they can saturate this uplink, leading to latency and bottlenecking.

**Q2: A system administrator updates a motherboard BIOS. After rebooting, the OS fails to boot, reporting "Inaccessible Boot Device". Explain how you would troubleshoot this.**
A: 
- **Situation:** After a BIOS update, the OS drive is no longer recognized as bootable, or it blue-screens on boot.
- **Task:** Identify why the bootloader or OS partition is inaccessible and restore system functionality.
- **Action:** I would boot into the UEFI/BIOS configuration menu. First, I would verify the storage controller mode; bios updates reset storage modes from RAID/AHCI back to default values. Second, I would check the boot mode configuration (Legacy CSM vs. Native UEFI). If the OS was originally installed using UEFI with Secure Boot, but CSM was enabled during the bios reset (or vice versa), the bootloader will fail to initialize.
- **Result:** Changing the storage controller back to AHCI/RAID (matching the OS installation) and toggling CSM to its original state allows the OS to recognize the drive partition structure and boot successfully.

**Q3: What is TPM 2.0 and how does a motherboard implement it?**
A: TPM 2.0 (Trusted Platform Module) is an international standard for a secure cryptoprocessor designed to secure hardware by integrating cryptographic keys. It stores certificates, passwords, and encryption keys (like BitLocker keys) securely. A motherboard can implement it in two ways: as a discrete TPM (dTPM), which is a physical chip plugged into a dedicated header on the motherboard, or as a firmware-based TPM (fTPM/PTT), which runs inside the CPU's secure execution environment. Both options provide identical cryptographic capabilities required by modern operating systems.

---
## Related Notes
- [[01-Foundations/01-Hardware/H-02 Storage Deep Dive|H-02 Storage Deep Dive]] — Detail on SATA, NVMe, and partition tables.
- [[01-Foundations/01-Hardware/H-05 PC Assembly and BIOS Configuration|H-05 PC Assembly and BIOS Configuration]] — Practical guide to system building and BIOS setup.
- [[01-Foundations/01-Hardware/H-06 Hardware Troubleshooting Masterclass|H-06 Hardware Troubleshooting Masterclass]] — Motherboard diagnostic beep codes and POST failures.
