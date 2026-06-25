---
tags: [#desktop-support, #hardware, #smps, #power-supply, #L1]
aliases: [power-supply-guide, psu-guide]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# SMPS Power Supply Complete Guide

---

## Concept Overview
- **What it is**: The Switched-Mode Power Supply (SMPS) is the hardware component that converts incoming high-voltage Alternating Current (AC) from the wall outlet into regulated low-voltage Direct Current (DC) required by the computer components (12V, 5V, 3.3V rails).
- **Why it matters for a support engineer**: Power instability causes intermittent crashes, random shutdowns, and component failures. Diagnosing power supply faults is essential to avoid replacing functional motherboards or GPUs.
- **Where you encounter this in real job**: Troubleshooting systems that refuse to power on, diagnosing random reboots under graphical load, upgrading GPUs requiring higher wattage, and testing suspicious power units using hardware testers.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Performs basic power cord checks, tests wall outlets, executes the paperclip test, and replaces standard desktop PSUs.
  - **L2**: Evaluates system power budgets for hardware upgrades, diagnoses rail voltage drops using multimeters, and troubleshoots modular cable compatibility.
  - **L3**: Establishes data center power distribution strategies, selects uninterruptible power supply (UPS) backups, and handles electrical safety compliance standards.

---

## Technical Deep Dive

### 1. PSU Form Factors
- **ATX**: The standard desktop power supply size (typically 150mm wide × 86mm high × 140mm deep). Compatible with ATX and Micro-ATX motherboards and cases.
- **SFX (Small Form Factor)**: Designed for compact ITX builds (125mm wide × 63.5mm high × 100mm deep). Requires an adapter bracket to fit standard ATX cases.
- **TFX (Thin Form Factor)**: Long, narrow design used in custom low-profile slim business desktops (e.g., Dell OptiPlex SFF).

### 2. Efficiency Ratings: The 80 Plus Standard
The 80 Plus certification rates power supply efficiency at 20%, 50%, and 100% loads. Higher efficiency reduces power draw from the wall, decreases heat generation, and extends component lifespan:
- **80 Plus Standard**: 80% efficiency across all loads.
- **Bronze**: 82% to 85% efficiency.
- **Silver**: 85% to 88% efficiency.
- **Gold**: 87% to 90% efficiency.
- **Platinum**: 89% to 92% efficiency.
- **Titanium**: 90% to 94% efficiency (adds efficiency evaluation at 10% load).

### 3. Connector Pinouts & Voltage Rails
Components require different DC voltages (rails) from the SMPS:
- **12V Rail (Yellow wires)**: Supplies power to high-draw components (CPU, GPU, fans).
- **5V Rail (Red wires)**: Supplies power to storage drives, logic circuits, and USB ports.
- **3.3V Rail (Orange wires)**: Supplies power to motherboard logic circuits and M.2 NVMe drives.
- **Standard Connectors**:
  - **24-pin ATX**: Main motherboard power.
  - **4+4 pin (8-pin) EPS**: Dedicated CPU power.
  - **6+2 pin (8-pin) PCIe**: Auxiliary power for graphics cards.
  - **12VHPWR (PCIe Gen 5)**: High-power 12+4 pin connector supplying up to 600W to modern GPUs.
  - **SATA Power**: Flat 15-pin connector for storage drives.

---

## Commands & Syntax

### PowerShell
```powershell
# Query current power plan settings and active configurations
Get-CimInstance -Namespace root\cimv2\power -ClassName Win32_PowerPlan |
    Select-Object ElementName, IsActive
```

### CMD / Run Box
```cmd
REM Generate a detailed HTML report of system power states and battery usage
powercfg /energy
powercfg /batteryreport
```

### GUI Path
> Control Panel -> Hardware and Sound -> Power Options
> Or: Device Manager -> System Devices -> Double-click **ACPI Multiprocessor PC**

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Control\Power\
HKLM\SYSTEM\CurrentControlSet\Control\Power\User\PowerSchemes\
```

### Key Event IDs
| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 41 | Kernel-Power (System rebooted without clean shutdown) | System Log |
| 1074 | System shutdown initiated by user or process | System Log |
| 6008 | Previous system shutdown was unexpected | System Log |

---

## Real-World Scenarios

### Scenario 1: Desktop Shuts Down Randomly Under Graphical or Heavy Load
**User Complaint:** "My computer works fine when I'm checking emails. But when I start compiling code or run GPU tasks, the machine turns off completely without a blue screen."
**Your First 3 Checks:**
1. Check the CPU and GPU temperatures to rule out thermal throttling and shutdown.
2. Check the Event Viewer System log for unexpected shutdown codes (Event ID 6008).
3. Verify the total wattage rating of the installed SMPS compared to the hardware components.
**Diagnosis Steps:**
1. Review the Event Viewer logs. Event ID 41 is logged, but there are no corresponding driver or memory dump logs, indicating a hardware-triggered power drop.
2. Run a stress test (using Prime95 and FurMark) while monitoring system power draw.
3. If the power shuts down immediately when the GPU load climbs:
   - The power draw has exceeded the SMPS capacity, triggering the Over-Current Protection (OCP) or Over-Power Protection (OPP) shutdown.
**Root Cause:** An underpowered 450W SMPS was installed in a machine upgraded with a high-draw GPU requiring a 650W PSU minimum.
**Fix:** Upgrade the SMPS to a certified 750W 80 Plus Gold power supply.
**Prevention:** Always calculate and document system power budgets before performing hardware upgrades.
**Ticket Close Note:** "Diagnosed system shutdown caused by PSU overload (Over-Power Protection trigger). Replaced 450W PSU with a 750W 80 Plus Gold PSU. Ran stress test for 1 hour. No failures. Closing ticket."

### Scenario 2: Machine Fails to Power On at All (Paperclip Test Verification)
**User Complaint:** "I press the power button and absolutely nothing happens. No lights, no fan spin, no sound."
**Your First 3 Checks:**
1. Verify that the power cord is plugged into the wall and the outlet has active power.
2. Check if the physical switch on the back of the SMPS is in the "I" (ON) position.
3. Disconnect internal components and perform a manual PSU startup check.
**Diagnosis Steps:**
1. Test the wall outlet with another working device to confirm power delivery.
2. Open the chassis and unplug the 24-pin ATX power connector from the motherboard.
3. Perform the paperclip test: Insert a jumper wire between **Pin 16 (PS_ON - Green wire)** and **Pin 17 (COM - Black wire)**.
   - Connect a fan as a load and turn on the PSU switch.
   - If the PSU fan does not spin:
     - The SMPS is dead and must be replaced.
**Root Cause:** Internal failure of the SMPS converter circuits, preventing voltage generation.
**Fix:** Replace the faulty SMPS with a standard ATX power supply.
**Prevention:** Connect corporate desktops to surge protectors to prevent electrical damage during storms.
**Ticket Close Note:** "PSU failed paperclip test. Replaced dead SMPS with new 550W ATX PSU. Verified successful boot to Windows. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Open the metal casing of an SMPS to attempt internal repair of capacitors or coils.
> - Power supply capacitors store high-voltage electrical charges for days after being unplugged, presenting a risk of fatal electrical shock.
> - Never attempt component-level repair on a power supply; replace the entire unit.

> [!warning] Common Trap
> - Mixing modular cables from different power supply brands or models.
> - Pinouts on the power supply side of modular cables are not standardized. Using a cable from a different brand can route 12V power into ground or 5V pins, frying motherboards or storage drives.
> - Only use the cables that shipped with the specific PSU model.

> [!tip] Senior Engineer Tip
> - When troubleshooting erratic system behavior, check the 12V rail voltage. If a multimeter shows the 12V rail dropping below 11.4V (more than a 5% tolerance drop) under load, the PSU is failing and should be replaced.

> [!success] Verification Steps
> - Verify that the system powers on, passes POST, and boots to the OS.
> - Run the command: `Get-CimInstance -Namespace root\cimv2\power -ClassName Win32_PowerPlan` to confirm the active power profile.

> [!question] Interview Alert
> - "How do you calculate the required wattage for a new custom desktop configuration?"
> - Answer: "I calculate the maximum power draw of the CPU and GPU, add 100W to 150W for secondary components (motherboard, RAM, storage, fans), and select a power supply that operates at roughly 50% to 70% capacity under maximum load to optimize efficiency."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Leaving the PSU switch set to "O" | Oversight during assembly | Ensure the back panel switch is set to "I" (On). |
| Forgetting CPU EPS 8-pin connector | Assuming 24-pin main power is sufficient | Always connect the dedicated 4+4/8-pin CPU power cable. |
| Selecting PSU solely on cheap price | Budget cutting | Choose reputable brands with active safety protections (OCP, OVP, SCP). |

---

## Lab Exercise

**Objective:** Calculate the power requirements of a system configuration, perform a manual paperclip test, and check output voltage rails using a digital multimeter.
**Time Required:** 30 minutes
**Environment Needed:** Anti-static workspace, a digital multimeter, a paperclip/jumper wire, and a target SMPS unit.
**Pre-requisites:** Safety goggles, SMPS disconnected from the wall and components.

**Steps:**
1. Calculate system power requirements:
   - Target CPU: Core i7 (125W TDP)
   - Target GPU: RTX 4070 (200W TDP)
   - Extras: Motherboard, 4 fans, 2 NVMe drives (approx. 100W)
   - Total Load: `425W` -> Recommended PSU size: `650W` (leaving safety headroom).
2. Safety Check: Disconnect the SMPS from the wall. Unplug all internal components from the motherboard and storage drives.
3. Paperclip Test: Connect a wire between the **Green pin (PS_ON)** and a **Black pin (Ground)** on the 24-pin ATX connector.
4. Plug the PSU into the wall and turn the switch to "I".
   - Expected output: The PSU fan spins, indicating the unit is active.
5. Multimeter Test: Set the multimeter to DC Voltage mode.
   - Insert the black probe into a Black pin (Ground).
   - Insert the red probe into a **Yellow pin (12V)** -> Expected: `~12.0 V` (11.4V to 12.6V range).
   - Insert the red probe into a **Red pin (5V)** -> Expected: `~5.0 V` (4.75V to 5.25V range).
   - Insert the red probe into an **Orange pin (3.3V)** -> Expected: `~3.3 V` (3.14V to 3.46V range).
6. Verification: Confirm all rails are within the 5% tolerance range.

**Success Criteria:** The SMPS powers on, and all output voltages measure within the 5% specification limits.
**Common Failures:** The PSU fan may stop spinning if the unit has a semi-passive fan curve; attach a chassis fan to a Molex connector to verify activity.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is the difference between modular, semi-modular, and non-modular power supplies?**
A: Non-modular PSUs have all cables permanently attached. Semi-modular PSUs have essential cables (24-pin and CPU power) fixed, while peripheral cables are detachable. Fully modular PSUs have all cables detachable, allowing for cleaner cable management.

**Q: What are the main voltages provided by a desktop power supply?**
A: A standard desktop SMPS provides three primary voltages: +12V (used by CPU, GPU, and fans), +5V (used by storage drives and USB controllers), and +3.3V (used by motherboard chips and M.2 devices).

### Intermediate (L2 Level)
**Q: How do you identify a power supply issue versus a motherboard or RAM issue during a POST failure?**
A: I isolate components. If the fans spin but there is no POST, I check the motherboard debug LEDs. If the LEDs do not light up and there are no signs of life, I check the PSU using a digital power supply tester or a multimeter. If the PSU rails provide correct voltages, the issue likely resides with the motherboard or VRMs.

### Advanced (L3/Senior Level)
**Q: A critical storage server shuts down unexpectedly during backup operations. How would you diagnose the power supply system?**
A:
- **Situation**: A production backup server shut down unexpectedly, causing database corruption risks.
- **Task**: I needed to identify if the shutdown was caused by power supply failure, system thermal limits, or load anomalies.
- **Action**: I reviewed the system log (Event ID 41, Event ID 6008) to verify a hard power-off occurred. I connected a multimeter to the 12V rail and monitored voltage during a simulated workload. I observed the 12V rail drop from 12.1V to 11.2V under load, triggering the motherboard's under-voltage protection shutdown.
- **Result**: I replaced the failing SMPS with a redundant dual-power supply unit (1+1 configuration) to prevent single-point-of-failure shutdowns.

### HR / Behavioral
**Q: Tell me about a time you had to deal with an equipment failure that impacted users, and how you managed the situation.**
A: A department power outage damaged several desktop power supplies on the morning of a key sales event. I set up a triage desk, used our inventory of spare PSUs to replace the damaged units, and tested each system using a hardware tester before returning them. All sales desks were online within two hours.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The hardware component that converts AC electricity to low-voltage DC rails.
> **Why**: Unstable power causes erratic system behavior, storage corruption, and component failures.
> **How**: Diagnose failures using the paperclip test or a multimeter to measure the 12V, 5V, and 3.3V rails.
> **Command**: `powercfg /energy`
> **Interview Answer Starter**: "In my experience, diagnosing power supply issues requires isolating the PSU from the motherboard and verifying rail voltages under load..."

**Key Numbers to Remember:**
- Voltage rail tolerance limit: +/- 5% (12V rail must stay between 11.4V and 12.6V)
- Standard ATX width/height: 150mm / 86mm
- 12VHPWR pin count: 16 pins (12 power + 4 signal)

**3 Things Interviewer Wants to Hear:**
1. Safe testing procedures (multimeter checks, avoiding opening the casing)
2. Danger of mixing modular cables
3. Understanding power calculations and efficiency ratings

---

## Related Notes
- [[01-Foundations/01-Hardware/Motherboard-Architecture|Motherboard Architecture]] — Recipient of 24-pin and CPU power connectors.
- [[01-Foundations/01-Hardware/Hardware-Troubleshooting-Masterclass|Hardware Troubleshooting Masterclass]] — Troubleshooting flow for power failures.
- [[04-Cloud-and-Security/09-Security/Microsoft-Security-Best-Practices|Microsoft Security Best Practices]] — Power plan configurations for security compliance.

---

## Tags
#desktop-support #hardware #smps #power-supply #L1 #interview-topic #lab-complete #daily-use
