---
tags: [sysadmin, hardware, pc-build, bios]
aliases: [pc-assembly]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-yellow]
> 🔧 **HARDWARE**

`#complete` `#intermediate` `#sysadmin`

# H-05: PC Assembly and BIOS Configuration

> [!abstract] Overview
> This note outlines the step-by-step procedure for physical PC assembly, component verification, BIOS initialization, and baseline configuration. Ek support engineer ko yeh jaanna zaroori hai taaki hardware failures aur POST issues ko confidently resolve kiya ja sake.

---
## 🧠 Concept Overview

- **What it is** — Plain English mein definition: Physical PC assembly aur BIOS setup ka step-by-step process.
- **Why it matters** — Real job mein kyun zaroori hai: Faulty components ko diagnose aur replace karne ke liye, ya naya workstation setup karne ke liye.
- **Where you see this** — Actual scenarios jahan yeh aata hai: Jab user ka system boot nahi hota, RAM upgrade karni hoti hai, ya thermal throttling issues aate hain.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — POST check karna, cable connections verify karna, aur debug LEDs dekhna. |
| **L2** | Configure, fix, escalate kab karta hai — BIOS update karna, XMP enable karna, RAM/storage replace karna. |
| **L3** | Architecture, design, enterprise-level — Advanced hardware compatibility, server assembly, custom cooling loops. |

> [!tip] Seedha Simple Mein
> *PC assembly parts ko aapas mein jodne ka ek step-by-step process hai. Sabse pehle CPU, RAM aur M.2 SSD ko motherboard par lagaya jata hai, fir motherboard ko case ke andar daal kar power supply aur cables connect kiye jaate hain. Pehli baar boot karne par BIOS mein settings configure karni hoti hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Building a PC** is like **Constructing a Lego model of a power plant** because...
>
> - You lay down the foundation blocks (Motherboard).
> - You install the reactor core (CPU) and fuel rods (RAM).
> - You plug in the power generator (SMPS) and build out the storage warehouses (SSD/HDD).
> - You configure the control room computer (BIOS) to manage operations.

---
## 🔬 Technical Deep Dive

### 1. Pre-Assembly Prep & Anti-Static Precautions

> [!info] Key Concept
> Electrostatic discharge (ESD) can destroy sensitive silicon components without visible sparks.

- **Safety Setup:** Assemble the PC on a clean, hard, non-conductive surface (wood or plastic). Avoid carpets.
- **ESD Wrist Strap:** Wear an ESD wrist strap. Clamp the alligator clip to a bare metal part of the PC case, and plug the PC's power supply into the wall (with the power switch **O** Off). This links your body to the building's electrical ground, equalizing static potential.

### 2. CPU Installation: LGA vs. PGA Sockets
- **LGA (Land Grid Array - Intel & AMD AM5):** Pins are on the motherboard socket; pads are on the CPU bottom. Align notches and corner triangles; lower the CPU straight down without force.
- **PGA (Pin Grid Array - AMD AM4):** Pins are on the CPU bottom; holes are on the motherboard socket. Lower the CPU into the socket; it should fall into place without force.

### 3. RAM Installation (Dual Channel Configuration)
> [!info] Key Concept
> To run memory in Dual-Channel mode, RAM modules must be installed in matching channels.

- For 2 sticks, install in slots **A2 and B2** (Slots 2 and 4). This places the modules at the end of the memory traces, reducing signal reflections and improving stability.

### 4. Storage Installation (M.2 & SATA)
- **M.2 NVMe SSD:** Insert the M.2 card into the slot at a $30^\circ$ angle until it is fully seated. Push the SSD flat and secure it.

### 5. BIOS Baseline Configuration
- **XMP/EXPO Memory Profile:** Enable XMP (Intel) or EXPO (AMD) to apply the RAM's rated speed, timings, and voltage.
- **Fan Curves:** Set fan control modes to **PWM** (for 4-pin fans) or **DC** (for 3-pin fans).

> [!danger] Common Mistake
> **Connecting the monitor to the motherboard video out:** Plugging the display cable into the motherboard's rear ports instead of the discrete graphics card (GPU) ports. Always plug display cables directly into the horizontal ports on the GPU.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A complete set of PC components (Case, Motherboard, CPU, Cooler, RAM, M.2 SSD, PSU)
> - A magnetic Phillips #2 screwdriver
> - ESD Wrist Strap

### Step 1: Motherboard Prep (Out of Case Assembly)

> [!tip] Pro Tip
> Always perform a "breadboard test" first. Assemble the CPU, cooler, RAM, and GPU on the motherboard outside of the chassis (resting on the motherboard cardboard box) and attempt a boot.

1. Place the motherboard on top of the cardboard box.
2. Install the CPU into the socket.
3. Install the M.2 NVMe SSD into the top slot.
4. Install two RAM sticks into slots A2 and B2.
5. Install the CPU cooler and connect the fan wire to **CPU_FAN**.

### Step 2: Chassis Installation and Power

1. Install the motherboard into the chassis.
2. Slide the PSU into the bottom chamber and screw it in.
3. Connect 24-pin ATX, 8-pin CPU, and PCIe power cables.
4. Connect the chassis front panel cables (**Power SW, Reset SW, HD Audio**).

### Step 3: First Boot Verification

```bash
# Power on the PC and press Del/F2 to enter BIOS
# Check for POST (Power-On Self-Test) using debug LEDs (CPU, DRAM, VGA, BOOT)
```

> [!success] Expected Output
> ```
> System boots into BIOS, CPU and RAM capacity are detected correctly, temperatures are stable (30°C-45°C).
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-PhysicalDisk` | Check if all installed storage drives are detected in Windows. | `Get-PhysicalDisk` |
| `Get-CimInstance Win32_PhysicalMemory` | Verify RAM slots, capacity, and active speeds. | `Get-CimInstance Win32_PhysicalMemory \| Select-Object Capacity, Speed` |
| `lsusb` | List USB buses and connected devices in Linux. | `lsusb` |
| `lspci \| grep -i vga` | Confirm discrete GPU is detected on PCIe in Linux. | `lspci \| grep -i vga` |
| `lscpu` | Read complete CPU specifications and architecture in Linux. | `lscpu` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| System powers on, fans spin 100%, black screen, DRAM LED is lit | Memory training failed or RAM not fully seated / wrong slots | Reinsert RAM firmly into slots A2 and B2. Clear CMOS if persists. |
| System boots but shuts down after 30 seconds. | Thermal overload due to poor CPU cooler contact or forgotten thermal paste sticker | Remove cooler, peel plastic film, apply thermal paste, remount securely. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: New GPU Underperforming

> [!example] Ticket
> "I installed a new PCIe Gen 4 GPU into my secondary slot, but the performance is significantly lower than expected."

**L1 Response:** Verify the GPU model and display connections. Ask user to check Task Manager for GPU utilization.
**Escalation Trigger:** If GPU is running at full load but FPS is low, pass to L2.
**L2 Resolution:** Inspect motherboard manual. Explain that lower PCIe slots run at x4/x8 via PCH. Move the GPU to the top PCIe x16 slot for full bandwidth.

---
## 🎤 Interview Questions

> [!question] Q1: What is memory training and why does a newly built PC take up to 2 minutes to display an image on its first boot?
> **Answer:** Memory training is a process where the BIOS scans RAM modules and adjusts electrical parameters for stable signal transmission. On the first boot, it runs read/write tests to optimize these traces, which takes time.

==**Exam Tip:** Once trained, parameters are stored in CMOS, making subsequent boots fast.==

> [!question] Q2: What is the difference between PWM and DC fan control modes in BIOS?
> **Answer:** **PWM** uses a 4-pin header with constant 12V, controlling speed via high-frequency pulses for precise control. **DC** uses a 3-pin header, adjusting the actual voltage (5V-12V), which stalls the fan if voltage drops too low.

---
## 🔗 Related Notes

- [[01-Foundations/01-Hardware/H-01 Motherboard Architecture|H-01 Motherboard Architecture]] — Motherboard form factors, chipsets, and slot designs.
- [[01-Foundations/01-Hardware/H-03 SMPS Power Supply Complete Guide|H-03 SMPS Power Supply Complete Guide]] — PSU calculation and cable connectors.
- [[01-Foundations/01-Hardware/H-06 Hardware Troubleshooting Masterclass|H-06 Hardware Troubleshooting Masterclass]] — Troubleshooting POST failures and diagnostic steps.
