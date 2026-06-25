---
tags: [sysadmin, hardware, pc-build, bios]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# H-05: PC Assembly and BIOS Configuration

> [!abstract] Overview
> This note outlines the step-by-step procedure for physical PC assembly, component verification, BIOS initialization, and baseline configuration. It details best practices for static prevention, hardware installation, and initial system testing.

---
## Concept
Think of building a PC like constructing a Lego model of a power plant. First, you prepare a clean workspace (ESD safety). Second, you lay down the foundation blocks (Motherboard). Third, you install the reactor core (CPU) and fuel rods (RAM). Fourth, you plug in the power generator (SMPS) and build out the storage warehouses (SSD/HDD). Finally, you configure the control room computer (BIOS) to manage water cooling flow rates (fan curves) and optimize energy output (XMP).

*Seedha simple mein: PC assembly parts ko aapas mein jodne ka ek step-by-step process hai. Sabse pehle CPU, RAM aur M.2 SSD ko motherboard par lagaya jata hai, fir motherboard ko case ke andar daal kar power supply aur cables connect kiye jaate hain. Pehli baar boot karne par BIOS mein settings configure karni hoti hain.*

---
## Technical Deep Dive

### 1. Pre-Assembly Prep & Anti-Static Precautions
Electrostatic discharge (ESD) can destroy sensitive silicon components without visible sparks.
- **Safety Setup:** Assemble the PC on a clean, hard, non-conductive surface (wood or plastic). Avoid carpets.
- **ESD Wrist Strap:** Wear an ESD wrist strap. Clamp the alligator clip to a bare metal part of the PC case, and plug the PC's power supply into the wall (with the power switch **O** Off). This links your body to the building's electrical ground, equalizing static potential.

### 2. CPU Installation: LGA vs. PGA/LIF Sockets
Advanced content only — basics in [[CPU basics, cores, threads]]

- **LGA (Land Grid Array - Intel & AMD AM5):** Pins are on the motherboard socket; pads are on the CPU bottom.
  1. Open the metal load lever and lift the socket frame cover.
  2. Hold the CPU by its edges. Align the triangle mark on the CPU corner with the triangle mark on the motherboard socket. Match the alignment notches on the sides of the CPU.
  3. Lower the CPU straight down into the socket. Do not slide or drop it.
  4. Lower the metal load lever and lock it under the hook. The plastic socket protection cap will pop off automatically. Save this cap for RMAs.
- **PGA (Pin Grid Array - AMD AM4):** Pins are on the CPU bottom; holes are on the motherboard socket.
  1. Lift the socket lever to the vertical position.
  2. Align the gold triangle on the CPU corner with the socket triangle.
  3. Lower the CPU into the socket; it should fall into place without force.
  4. Lower the lever to lock pins in place.

### 3. RAM Installation (Dual Channel Configuration)
Advanced content only — basics in [[RAM types DDR3/DDR4/DDR5]]

To run memory in Dual-Channel mode (doubling memory bus bandwidth), RAM modules must be installed in matching channels.
- **Slot Naming:** Typically labeled A1, A2, B1, B2 (from left to right closest to CPU).
- **Rule of Thumb (4 Slots):** For 2 sticks, install in slots **A2 and B2** (Slots 2 and 4). This places the modules at the end of the memory traces (daisy chain topology), reducing signal reflections and improving stability.
- **Installation:** Open the locking tabs on the slot. Align the notch on the memory stick with the key in the slot. Press down firmly on both ends until the locking tabs snap closed with a click.

### 4. Storage Installation (M.2 & SATA)
- **M.2 NVMe SSD:**
  1. Locate the M.2 slot (use the slot closest to the CPU for direct lanes). Remove the motherboard's M.2 heatsink.
  2. Insert the M.2 card into the slot at a $30^\circ$ angle until it is fully seated.
  3. Push the SSD flat. Secure it using the tiny M.2 screw or toolless rotating latch (e-clip).
  4. Peel the protective plastic film off the heatsink's thermal pad and reinstall the heatsink.
- **SATA SSD/HDD:**
  1. Mount the drive in the chassis bay.
  2. Connect a SATA data cable from the drive to the motherboard SATA port.
  3. Connect a SATA power cable from the SMPS to the drive.

### 5. Cable Management Best Practices
- Route cables through the rubber grommets to the back of the motherboard tray.
- Group cables using Velcro straps or zip ties. Keep high-voltage lines (24-pin, 8-pin CPU) separated from sensitive data lines (SATA, front panel audio) where possible to reduce EMI.
- Ensure no cables hang near or touch cooling fans.

### 6. First Boot Verification
1. Connect power, monitor (to GPU, not motherboard, if a discrete GPU is installed), keyboard, and mouse.
2. Flip SMPS switch to **I** (On). Press case Power button.
3. Check for **POST (Power-On Self-Test)**. Watch the debug LEDs (CPU, DRAM, VGA, BOOT) near the 24-pin connector. If an LED stays lit, that component is failing initialization.
4. Verify that the CPU fan and system fans are spinning.

### 7. BIOS Baseline Configuration
- **Boot Priority:** Set the boot priority to read the OS installation USB first.
- **XMP/EXPO Memory Profile:** Enable XMP (Intel) or EXPO (AMD) to apply the RAM's rated speed, timings, and voltage.
- **Hardware Monitoring:** Verify CPU temperatures (idle should be $30^\circ\text{C}-45^\circ\text{C}$).
- **Fan Curves:** Set fan control modes to **PWM** (for 4-pin fans) or **DC** (for 3-pin fans). Adjust curves so fans spin up gradually when CPU temperature crosses $60^\circ\text{C}$.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A complete set of PC components (Case, Motherboard, CPU, Cooler, RAM, M.2 SSD, PSU) and a magnetic Phillips #2 screwdriver.

### Step 1: Motherboard Prep (Out of Case Assembly)
1. Place the motherboard box on the desk. Place the motherboard on top of the cardboard box. Do not place it on the anti-static bag (which is conductive on the outside).
2. Install the CPU into the socket (LGA/AM socket guidelines).
3. Install the M.2 NVMe SSD into the top slot.
4. Install two RAM sticks into slots A2 and B2.
5. Install the CPU cooler (apply thermal paste if it is not pre-applied). Connect the cooler fan wire to the **CPU_FAN** header.

### Step 2: Chassis Installation
1. Install the motherboard backplate (I/O shield) into the chassis back opening (if not pre-integrated).
2. Install brass standoffs in the case aligning with the motherboard size (ATX/mATX).
3. Lower the motherboard into the chassis, ensuring the ports slide into the I/O shield.
4. Secure the motherboard using the screws provided with the chassis.

### Step 3: Power and Front Panel Hookup
1. Slide the PSU into the bottom chamber and screw it in.
2. Route and plug in:
   - 24-pin ATX power cable to motherboard.
   - 8-pin EPS/CPU power cable to top-left motherboard header.
   - 6+2 pin PCIe power cable(s) to GPU.
3. Connect the chassis front panel cables:
   - **Power SW, Reset SW, HDD LED, Power LED** to the motherboard header (refer to manual for pin polarity).
   - **HD Audio** to the bottom-left header.
   - **USB 3.0** 19-pin cable.

### Step 4: First Boot Verification
1. Insert a bootable Windows installer USB drive into a rear port.
2. Turn on the PC and press **Del** to enter UEFI BIOS.
3. Verify that the CPU and RAM capacity are detected correctly.
4. Navigate to the Exit tab, choose **Save Changes & Reset**, and allow the system to boot into the Windows Installer.

---
## Commands Reference
Operating system tools to verify hardware configuration after installation:

```powershell
# Windows PowerShell
Get-PhysicalDisk                                              # Check if all installed storage drives are detected
Get-CimInstance Win32_PhysicalMemory | Select-Object Capacity, Speed, ConfiguredClockSpeed, DeviceLocator  # Verify RAM slots and active speeds
Get-WmiObject Win32_Processor | Select-Object Name, MaxClockSpeed, NumberOfCores  # Check CPU model and basic stats

# Linux Bash
lsusb                  # List USB buses and connected devices
lspci | grep -i vga    # Confirm discrete GPU is detected on PCIe
lscpu                  # Read complete CPU specifications and architecture
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** The PC powers on, the fans spin at maximum speed, but the screen remains black. The motherboard **DRAM debug LED** stays lit red/yellow.
- **Root Cause:** Memory training failed, the RAM modules are not fully seated, or they are installed in the wrong slots (e.g., A1 and B1 instead of A2 and B2).
- **Fix:**
  1. Shut down the system and unplug the power cable.
  2. Remove the RAM modules. Inspect slots for dust or debris.
  3. Reinsert one RAM stick into slot A2. Push down firmly until both ends click.
  4. Power on. If it boots, power down, add the second stick to slot B2, and boot again.
  5. If DRAM light persists, clear CMOS to reset memory settings.

**Scenario 2:**
- **Problem:** The system starts boot, but after 30 seconds it shuts down completely. Attempting to power on immediately fails, but waiting 5 minutes allows it to boot for another 30 seconds.
- **Root Cause:** The CPU cooler is not making contact with the CPU integrated heat spreader (IHS) or the thermal paste/cooler sticker was left on, causing thermal overload shutdown.
- **Fix:**
  1. Shut down the PC, unplug power, and remove the CPU cooler.
  2. Check the bottom of the cooler heatsink. Verify that the protective plastic shipping film ("PEEL OFF BEFORE INSTALLATION") was removed.
  3. Clean the old thermal paste with isopropyl alcohol.
  4. Reapply paste and remount the cooler. Ensure the mounting brackets are tightened evenly in a cross pattern.
  5. Verify that the fan cable is plugged into **CPU_FAN**, not SYS_FAN or AIO_PUMP.

---
## Common Mistakes
> [!warning] Avoid These
> **Connecting the monitor to the motherboard video out:** Plugging the HDMI/DisplayPort cable into the motherboard's rear ports instead of the discrete graphics card (GPU) ports. This causes the PC to run on weak integrated graphics, or show no display at all if the CPU lacks an integrated GPU.
> **Correct approach:** Always plug display cables directly into the horizontal ports on the GPU located lower down the chassis back.

---
## Pro Tips
> [!tip] Field Experience
> When assembling a system, always perform a "breadboard test" first. Assemble the CPU, cooler, RAM, and GPU on the motherboard outside of the chassis (resting on the motherboard cardboard box) and attempt a boot. It is much easier to troubleshoot a faulty component on a desk than after routing all cables inside a tight chassis.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | ESD Prevention | Ground yourself using a wrist strap connected to a metal chassis plugged into a grounded outlet. |
| 2 | LGA Installation | Align notches and corner triangles; lower the CPU straight down without force. |
| 3 | Memory Slots | For dual-channel performance with 2 sticks, always use slots A2 and B2 (2 and 4). |
| 4 | Debug LEDs | Motherboard check-lights (CPU, DRAM, VGA, BOOT) that pinpoint which component failed POST. |
| 5 | XMP/EXPO | BIOS settings that must be enabled to run RAM at its rated high performance frequency. |

---
## Interview Q&A

**Q1: What is memory training and why does a newly built PC take up to 2 minutes to display an image on its first boot?**
A: Memory training is a process where the motherboard BIOS/UEFI scans the installed RAM modules and adjusts electrical parameters (voltages, timing delays, impedance) to ensure stable signal transmission across the memory bus. On the very first boot, or after enabling XMP/EXPO, the system must run a sequence of read/write tests to optimize these signal traces. During this time, the DRAM debug LED may flash and the system may seem unresponsive. Once trained, these parameters are stored in CMOS, and subsequent boots will be fast.

**Q2: A technician installs a new PCIe Gen 4 GPU into a motherboard's secondary PCIe slot. Performance is significantly lower than expected. Explain how you would troubleshoot this.**
A: 
- **Situation:** A new GPU is underperforming in the secondary expansion slot.
- **Task:** Identify the bus configuration of the slot and reallocate resources for maximum performance.
- **Action:** I will inspect the motherboard user manual. Most motherboards only route the full x16 PCIe lanes directly to the top PCIe slot closest to the CPU. The lower PCIe slots are usually wired electrically for x4 or x8 and route through the PCH chipset, sharing bandwidth and adding latency. I will check the GPU bus width in CPU-Z or HWiNFO.
- **Result:** Moving the GPU to the top PCIe x16 slot restores full bandwidth and matches expected performance.

**Q3: What is the difference between PWM and DC fan control modes in BIOS?**
A: **PWM (Pulse Width Modulation)** uses a constant 12V supply via a 4-pin header, controlling fan speed by sending high-frequency pulses to a control chip inside the fan motor. This allows precise speed control down to very low RPMs (e.g., $10\%-20\%$). **DC (Direct Current)** mode uses a 3-pin header, controlling fan speed by adjusting the actual voltage supplied (varying between 5V and 12V). If the voltage drops below a certain threshold (usually 5V), the fan motor stalls and stops spinning entirely, limiting low-end speed control.

---
## Related Notes
- [[01-Foundations/01-Hardware/H-01 Motherboard Architecture|H-01 Motherboard Architecture]] — Motherboard form factors, chipsets, and slot designs.
- [[01-Foundations/01-Hardware/H-03 SMPS Power Supply Complete Guide|H-03 SMPS Power Supply Complete Guide]] — PSU calculation and cable connectors.
- [[01-Foundations/01-Hardware/H-06 Hardware Troubleshooting Masterclass|H-06 Hardware Troubleshooting Masterclass]] — Troubleshooting POST failures and diagnostic steps.
