---
tags: [sysadmin, hardware, smps, power]
difficulty: Intermediate
lab-required: Yes
read-time: 12 mins
---

# H-03: SMPS Power Supply Complete Guide

> [!abstract] Overview
> This note covers Switched-Mode Power Supply (SMPS) form factors, efficiency standards, connector configurations, and capacity calculation. It provides troubleshooting processes, including physical bypass testing and electrical multimeter validation.

---
## Concept
Think of the SMPS as the heart and blood vessels of the computer. The wall outlet supplies high-pressure alternating current (AC) power, which is like raw, unfiltered water. The SMPS acts as a water treatment facility and pumping station, converting that high-pressure AC into clean, regulated, low-pressure direct current (DC) streams (3.3V, 5V, and 12V rails) and distributing it to each organ (CPU, GPU, storage) at the exact pressure they need. 

If the heart is weak or irregular, the organs will experience brownouts, behave erratically, or shut down entirely to protect themselves.

*Seedha simple mein: SMPS aapke ghar ki AC power (220V/110V) ko DC power (12V, 5V, 3.3V) mein badalta hai jo computer ke sensitive chips ko chahiye hoti hai. Agar SMPS kharab hoga, toh PC switch on hi nahi hoga ya chalti class mein band ho jayega.*

---
## Technical Deep Dive

### 1. PSU Form Factors
- **ATX (Advanced Technology Extended):** The standard desktop power supply size. Dimensions are typically 150mm wide, 86mm high, and 140mm (or deeper) long. Fits standard mid-tower and full-tower chassis.
- **SFX (Small Form Factor):** Designed for compact SFF/Mini-ITX cases. Standard dimensions are 125mm wide, 63.5mm high, and 100mm long. Requires an adapter bracket to fit in ATX cases.
- **TFX (Thin Form Factor):** Long and narrow profiles, typical in low-profile office PCs (e.g., Dell Optiplex SFF, HP ProDesk). Dimensions are 85mm wide, 65mm high, and 175mm long.

### 2. 80 Plus Efficiency Ratings
The 80 Plus certification rates power supply efficiency at 20%, 50%, and 100% workloads. Higher efficiency means less electricity is lost as heat.

| Rating | 20% Load | 50% Load | 100% Load |
|---|---|---|---|
| **80 Plus Standard** | 80% | 80% | 80% |
| **80 Plus Bronze** | 82% | 85% | 82% |
| **80 Plus Silver** | 85% | 88% | 85% |
| **80 Plus Gold** | 87% | 90% | 87% |
| **80 Plus Platinum** | 90% | 92% | 89% |
| **80 Plus Titanium** | 92% | 94% | 90% |
| *Note: Numbers above represent 115V internal non-redundant configurations. 230V EU ratings are ~2% higher.* |

### 3. Connectors Explained
All modern PSU cables supply specific DC voltages:

- **24-Pin ATX Main Power Connector:**
  ```
  [========================]  <- Delivers 3.3V, 5V, 12V, -12V, and 5VSB (Standby)
  ```
  Powers the motherboard chipset, PCIe slots, and RAM modules.

- **4+4 Pin EPS/CPU Connector:**
  ```
  [====][====]  <- Delivers pure 12V power
  ```
  Plugs near the CPU socket to power the CPU VRMs. Can split into a single 4-pin for lower-end boards.

- **6+2 Pin PCIe Connector:**
  ```
  [======][==]  <- Delivers 12V power to graphics cards
  ```
  Features a detachable 2-pin section. A 6-pin provides up to 75W, and an 8-pin provides up to 150W.

- **SATA Power Connector:**
  ```
  [__________________] L-Shape  <- Delivers 3.3V, 5V, and 12V
  ```
  Powers mechanical HDDs, SATA SSDs, and optical drives.

- **4-Pin Peripheral (Molex):**
  ```
  ( O O O O )  <- Delivers 5V (Red) and 12V (Yellow)
  ```
  Legacy connector now used for case fans, pump controls, and RGB hubs.

### 4. Wattage Calculation
To size a PSU, calculate the total power consumption of the components, focusing primarily on the 12V rail (which powers the CPU and GPU):
1. **CPU TDP:** (e.g., Intel Core i7-13700K: ~125W base, up to 253W boost).
2. **GPU TDP:** (e.g., NVIDIA RTX 4070 Ti: ~285W).
3. **Motherboard & RAM:** ~50W.
4. **Storage & Fans:** ~10W per HDD, ~5W per SSD, ~3W per fan.
5. **Headroom/Efficiency Buffer:** Add 20% to 30% to prevent the PSU from running at 100% capacity constantly.

### 5. Modular vs. Semi-Modular vs. Non-Modular
- **Non-Modular:** All cables are soldered to the internal PCB. Unused cables must be bundled and hidden in the case. Cheap, but restricts airflow.
- **Semi-Modular:** Core cables (24-pin and 8-pin CPU) are fixed; SATA and PCIe cables are detachable. Good compromise of price and cable management.
- **Modular:** Every cable is detachable. Simplifies custom cabling, improves airflow, and allows easier PSU replacement.

### 6. Ten Real Symptoms of PSU Failure
1. The system does not power on at all (no fans, no LEDs).
2. The computer starts, but immediately shuts down after 1-2 seconds.
3. Random Blue Screens of Death (BSOD) under 3D gaming or rendering workloads.
4. Spontaneous restarts without any BSOD or warning logs.
5. The smell of burning ozone or plastic coming from the back of the case.
6. A high-pitched whining noise (coil whine) or grinding fan bearings.
7. USB peripherals disconnect and reconnect randomly due to unstable 5V rail.
8. Storage drives drop offline or fail to spin up (caused by weak 5V or 12V rails).
9. Graphical artifacts or driver crashes during GPU load.
10. The chassis feels warm to the touch near the PSU mounting area (failed PSU fan).

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A disconnected SMPS, a paperclip (or PSU bridge tool), and a digital multimeter.

### Step 1: The Paperclip Bypass Test
This test tricks the PSU into turning on without being connected to a motherboard by shorting the **PS_ON** pin to a ground pin.

1. Ensure the PSU is completely unplugged from the wall outlet and its power switch is set to **O** (Off).
2. Disconnect all cables from the motherboard, GPU, and drives.
3. Locate the 24-Pin ATX connector. Find the **Green Wire** (Pin 16 - `PS_ON#`).
4. Find any **Black Wire** (Pin 15 or 17 - `COM/Ground`).
5. Bend a metal paperclip into a U-shape. Insert one end into the pin slot with the Green wire, and the other end into an adjacent Black wire slot.
6. Plug the PSU into the wall outlet and switch it to **I** (On).
7. **Verify:** The PSU fan should spin up. If the fan spins, the PSU basic startup logic is functioning. If it does not spin, the PSU is dead.

```
24-Pin ATX Connector Pinout Example (Clip facing up):
Pins 1-12 on top, 13-24 on bottom.
Pin 16 (Green - PS_ON) is on the bottom row, 4th from the left.
Pin 17 (Black - COM) is right next to it.
[  13  14  15 [16] [17]  18  19  20  21  22  23  24  ]
                ^----^  <-- Insert paperclip here
```

### Step 2: Testing Voltages with a Multimeter
1. Keep the paperclip inserted so the PSU remains powered on.
2. Turn on your digital multimeter and set the dial to **DC Voltage (20V range)**.
3. Insert the **Black probe** of the multimeter into any COM (Black wire) pin on the 24-pin connector.
4. Insert the **Red probe** into the following pins to verify output:
   - **Yellow Wire** (Pin 10/11/etc): Should read between **11.4V and 12.6V** (Target: 12.00V).
   - **Red Wire** (Pin 4/6/etc): Should read between **4.75V and 5.25V** (Target: 5.00V).
   - **Orange Wire** (Pin 1/2/etc): Should read between **3.14V and 3.46V** (Target: 3.30V).
   - **Purple Wire** (Pin 9 - 5VSB): Should read **5V** even when the PSU is switched on but PS_ON is not shorted.
5. If any rail reads outside the $\pm 5\%$ tolerance range, the PSU must be replaced.

---
## Commands Reference
Operating systems cannot directly read SMPS voltage rails because standard power supplies do not have data interfaces. However, if using a smart PSU (e.g., Corsair i-Series) with a USB header connection, utility tools query stats:

```powershell
# Windows: Read motherboard voltage sensors (requires manufacturer WMI providers or OpenHardwareMonitor APIs)
Get-WmiObject -Namespace root\OpenHardwareMonitor -Class Sensor | Where-Object {$_.SensorType -eq "Voltage"}

# Linux: Read sensor chips (requires lm-sensors installed)
sudo sensors-detect --auto
sensors | grep -E "12V|5V|3.3V"
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** User reports the PC shuts down instantly and completely whenever they launch a modern 3D game. There are no BSODs; the screen goes black, and the power LED on the chassis goes off.
- **Root Cause:** The GPU transient power spikes are triggering the PSU's **OCP (Over-Current Protection)** or **OPP (Over-Power Protection)** because the PSU capacity is insufficient or degrading.
- **Fix:** 
  1. Calculate the system wattage: CPU (125W) + GPU (285W) + system overhead (50W) = 460W.
  2. Inspect the PSU label. If it's a budget 500W unit, it lacks headroom for transient power spikes (which can double GPU power draw for milliseconds).
  3. Replace the PSU with a high-quality 750W 80 Plus Gold unit.
  4. Test by running FurMark and Prime95 simultaneously to simulate maximum system load.

**Scenario 2:**
- **Problem:** A PC fails to power on. Pressing the power button does nothing. However, the green LED on the motherboard remains illuminated.
- **Root Cause:** The 5VSB (5V Standby) rail of the SMPS is functioning (lighting up the motherboard LED), but the main 12V or 5V rail is dead, or the motherboard's power switch header is disconnected.
- **Fix:**
  1. Unplug the PC, open the side panel, and ensure the front panel Power SW cable is securely connected to the motherboard header.
  2. Perform the Paperclip Bypass Test on the PSU.
  3. If the PSU fan spins up during the paperclip test, use a screwdriver to short the Power SW motherboard pins directly. If the PC starts, the chassis power button is broken.
  4. If the PSU fan does not spin during the paperclip test, the SMPS is faulty and must be replaced despite the motherboard standby light being on.

---
## Common Mistakes
> [!warning] Avoid These
> **Mixing modular PSU cables:** Using modular cables from an EVGA power supply on a Corsair power supply. The connector that plugs into the PSU side is not standardized; pinouts vary wildly between brands and models. Doing this will send 12V down a 5V or ground wire, instantly frying your SSDs, motherboard, and GPU.
> **Correct approach:** Only use the cables that came in the box with that specific PSU model. If replacements are needed, buy cables explicitly certified for that exact model.

---
## Pro Tips
> [!tip] Field Experience
> Never open a power supply case to replace a fan or check capacitors unless you are a certified electronics technician. The large primary capacitors inside an SMPS hold lethal voltages (380V+) for days or weeks after the unit has been unplugged from the wall.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | ATX vs SFX | ATX is the standard desktop PSU size; SFX is a compact version for mini-ITX small form factor cases. |
| 2 | 80 Plus Rating | Measures efficiency; Gold means $\ge 90\%$ of wall power is converted to DC, reducing heat loss. |
| 3 | 12V Rail | The most critical rail; powers the CPU (via 8-pin EPS) and the GPU (via 6+2 pin PCIe). |
| 4 | Paperclip Test | Shorting Pin 16 (Green) to Pin 17 (Black) tricks the PSU into turning on for diagnostic testing. |
| 5 | Modular PSU | Power supply where all cables detach, simplifying cable management and routing. |

---
## Interview Q&A

**Q1: What is the purpose of the PG (Power Good) signal on Pin 8 of the 24-Pin ATX connector?**
A: The Power Good (PG) signal is a +5V signal generated by the SMPS once it has completed its internal self-tests and stabilized its output voltages (3.3V, 5V, and 12V rails). This signal is sent to the motherboard (usually within 100ms to 500ms after power-on). As long as the motherboard receives this +5V signal, the CPU will run. If the power supply outputs drop or fluctuate, the PG signal falls to 0V, causing the motherboard to instantly reset or shut down the CPU to prevent data corruption or component damage.

**Q2: A server room experiences random reboots on a hypervisor under peak load. The ups shows no utility power drops. Explain how you would troubleshoot the power supply setup.**
A: 
- **Situation:** A physical host is rebooting spontaneously during peak loads, and utility power is verified stable.
- **Task:** Identify if the issue lies in power capacity, PSU degradation, or phase configuration.
- **Action:** First, I would check the server's management interface (e.g., iDRAC/iLO) logs to check for Power Supply Unit (PSU) redundancy alerts or thermal warnings. Second, I would verify the configuration of the redundant PSUs. If they are configured in "Active/Passive" mode and load spikes exceed the capacity of a single PSU, it could trigger OCP during a switchover. Third, I would check if both PSUs are plugged into different power phases or PDUs.
- **Result:** If one PSU was failing under load, replacing the degraded unit and balancing the load across both PSUs in "Active/Active" mode resolves the spontaneous reboots.

**Q3: What are transient response and ripple in the context of an SMPS?**
A: **Transient response** is the speed and stability with which a power supply can adjust its voltage output in response to sudden, extreme changes in load (such as a GPU jumping from 50W to 350W in microseconds). If the response is poor, the voltage drops too low, causing a crash. **Ripple** refers to the residual AC voltage fluctuations (measured in millivolts peak-to-peak) superimposed on the DC output rails after rectification. Low ripple (under 50mV on the 12V rail) is critical for protecting the delicate transistors on memory chips and GPUs.

---
## Related Notes
- [[01-Foundations/01-Hardware/H-01 Motherboard Architecture|H-01 Motherboard Architecture]] — Explains motherboard 24-pin and 8-pin power delivery interfaces.
- [[01-Foundations/01-Hardware/H-05 PC Assembly and BIOS Configuration|H-05 PC Assembly and BIOS Configuration]] — Safe installation and cable routing.
- [[01-Foundations/01-Hardware/H-06 Hardware Troubleshooting Masterclass|H-06 Hardware Troubleshooting Masterclass]] — Diagnosing power and POST issues.
