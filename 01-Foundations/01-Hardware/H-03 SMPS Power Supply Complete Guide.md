---
tags: [hardware, smps, power-supply, troubleshooting]
aliases: [SMPS Guide, PSU]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-yellow]
> 🔧 **HARDWARE**

`#complete` `#intermediate` `#none`

# H-03: SMPS Power Supply Complete Guide

> [!abstract] Overview
> Yeh note Switched-Mode Power Supply (SMPS) ke form factors, efficiency standards, connectors, aur wattage calculation ko cover karta hai. Ek support engineer ko troubleshooting ke liye paperclip bypass aur multimeter validation aana chahiye, kyunki hardware failures mein SMPS ek common culprit hota hai.

---
## 🧠 Concept Overview

- **What it is** — SMPS is the component that converts high-voltage AC from your wall outlet into the low-voltage DC required by computer parts.
- **Why it matters** — A failing PSU can cause random shutdowns, blue screens, or completely prevent the PC from turning on. 
- **Where you see this** — In every desktop PC, server, and many network appliances, providing 3.3V, 5V, and 12V rails.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check cables, verify wall power, test power switch. |
| **L2** | Perform paperclip bypass test, check voltages with a multimeter, replace PSU. |
| **L3** | Calculate enterprise power budgets, plan redundant PSU load balancing. |

> [!tip] Seedha Simple Mein
> *SMPS aapke ghar ki AC power (220V/110V) ko DC power (12V, 5V, 3.3V) mein badalta hai jo computer ke sensitive chips ko chahiye hoti hai. Agar SMPS kharab hoga, toh PC switch on hi nahi hoga ya chalti class mein band ho jayega.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **An SMPS** is like the **heart and blood vessels** of the computer because...
>
> - The wall outlet supplies high-pressure AC (unfiltered blood/water).
> - The SMPS regulates and pumps clean, low-pressure DC to the organs (CPU, GPU).
> - If the heart is weak or irregular, the organs experience brownouts or shut down to protect themselves.

---
## 🔬 Technical Deep Dive

### 1. PSU Form Factors & Efficiency

> [!info] Key Concept
> **ATX** is the standard desktop size. **SFX** is for small cases. **80 Plus** rating measures how efficiently the PSU converts AC to DC without losing power as heat.

- **ATX:** Standard desktop (150mm x 86mm x 140mm+).
- **SFX:** Compact size (125mm x 63.5mm x 100mm). Needs adapter for ATX case.
- **80 Plus Ratings:** Higher ratings (Bronze, Gold, Titanium) mean better efficiency. **Gold** achieves 90% efficiency at 50% load.

### 2. Connectors Explained

- **24-Pin ATX Main:** Powers motherboard, PCIe slots, and RAM (3.3V, 5V, 12V).
- **4+4 Pin EPS/CPU:** Pure 12V power for CPU VRMs.
- **6+2 Pin PCIe:** 12V power to graphics cards (up to 150W for 8-pin).
- **SATA Power:** For HDDs and SSDs (3.3V, 5V, 12V).

> [!danger] Common Mistake
> **Mixing modular PSU cables:** Using modular cables from one brand (like EVGA) on another (like Corsair). Pinouts vary wildly on the PSU side. Doing this will fry your SSDs, motherboard, and GPU! Only use cables that came with that exact unit.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A disconnected SMPS
> - Metal paperclip (or PSU bridge tool)
> - Digital Multimeter

### Step 1: The Paperclip Bypass Test

This test tricks the PSU into turning on without a motherboard by shorting the **PS_ON** pin to **Ground**.

```bash
# 1. Unplug PSU from wall. Switch to 'O' (Off).
# 2. Disconnect all motherboard/GPU cables.
# 3. Locate 24-Pin ATX connector. Find Green Wire (Pin 16).
# 4. Find adjacent Black Wire (Pin 15 or 17).
# 5. Insert paperclip into Green and Black wire slots.
# 6. Plug in and switch on 'I'.
```

> [!success] Expected Output
> The PSU fan should spin up. This means basic startup logic works. If it doesn't spin, the PSU is completely dead.

### Step 2: Testing Voltages with Multimeter

Keep the paperclip inserted so the PSU runs.

```bash
# 1. Set Multimeter to DC Voltage (20V range).
# 2. Black probe -> Black Wire (COM).
# 3. Red probe -> Yellow Wire: should read ~12.00V.
# 4. Red probe -> Red Wire: should read ~5.00V.
# 5. Red probe -> Orange Wire: should read ~3.30V.
```

> [!success] Expected Output
> Voltages should be within a $\pm 5\%$ tolerance range.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-WmiObject -Namespace root\OpenHardwareMonitor -Class Sensor` | Windows: Read motherboard voltage sensors (needs OHM) | `... \| Where-Object {$_.SensorType -eq "Voltage"}` |
| `sudo sensors-detect --auto` | Linux: Detect sensor chips | `sudo sensors-detect --auto` |
| `sensors` | Linux: Read current voltages and temps | `sensors \| grep -E "12V|5V|3.3V"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **System completely dead (No LEDs, no fans)** | Wall outlet dead, faulty power cable, or completely dead SMPS. | Verify wall power. Perform paperclip bypass test. Replace if dead. |
| **Powers on for 1-2 sec, then shuts off** | Short circuit somewhere in the PC, or faulty PSU protection tripping. | Unplug components one by one to find the short, or test PSU standalone. |
| **Random restarts under heavy 3D load** | PSU lacks enough capacity (wattage) for GPU transient spikes, tripping OCP. | Calculate total wattage. Upgrade to a higher wattage/quality PSU (e.g., 750W Gold). |
| **Whining noise / grinding** | Coil whine from components, or failing PSU cooling fan bearing. | Replace PSU if noise is unbearable or fan is failing. (Do not open PSU to fix). |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Random Shut Downs Under Load

> [!example] Ticket
> "Hi IT, my PC shuts down instantly and completely whenever I launch a modern 3D rendering job. The screen goes black."

**L1 Response:** Check basic connections, ensure PC is plugged directly into wall (bypass faulty power strip).
**Escalation Trigger:** If issue persists only under heavy load, pass to L2 for hardware diagnosis.
**L2 Resolution:** Calculate system wattage. The GPU transient power spikes are triggering the PSU's OCP/OPP because the PSU is degrading or underpowered. Replace with a high-quality 80 Plus Gold unit.

### 🎫 Scenario 2: Motherboard LED On, But Won't Boot

> [!example] Ticket
> "My PC won't turn on when I press the button, but I can see a green light inside on the motherboard."

**L1 Response:** Ask user to check if the front panel power button feels broken.
**Escalation Trigger:** Needs physical hardware inspection.
**L2 Resolution:** The 5VSB (Standby) rail is working (lighting the LED), but the main 12V rail is dead. Run paperclip test. If PSU fails test, replace SMPS. If it passes, short the motherboard power pins directly to verify if the chassis button is broken.

---
## 🎤 Interview Questions

> [!question] Q1: What is the purpose of the PG (Power Good) signal on Pin 8 of the 24-Pin ATX connector?
> **Answer:** It's a +5V signal sent by the SMPS to the motherboard after it completes self-tests and stabilizes its voltages (3.3V, 5V, 12V). If voltages drop, the PG signal falls to 0V, causing the motherboard to instantly reset or shut down to prevent damage.

> [!question] Q2: What are transient response and ripple in an SMPS?
> **Answer:** **Transient response** is how quickly the PSU adjusts its output to sudden load changes (e.g., GPU jumping from 50W to 350W). **Ripple** is the residual AC voltage fluctuation on the DC rails. Low ripple (<50mV on 12V rail) is critical to protect sensitive chips.

==**Exam Tip:** Never open a power supply case to replace parts! The large primary capacitors hold lethal voltages (380V+) for days after unplugging.==

---
## 🔗 Related Notes

- [[01-Foundations/01-Hardware/H-01 Motherboard Architecture|H-01 Motherboard Architecture]] — Explains motherboard power delivery interfaces.
- [[01-Foundations/01-Hardware/H-05 PC Assembly and BIOS Configuration|H-05 PC Assembly and BIOS Configuration]] — Safe installation and cable routing.
- [[01-Foundations/01-Hardware/H-06 Hardware Troubleshooting Masterclass|H-06 Hardware Troubleshooting Masterclass]] — Diagnosing power and POST issues.
