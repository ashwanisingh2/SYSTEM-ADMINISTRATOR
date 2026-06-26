---
tags: [sysadmin, hardware, troubleshooting, diagnostics]
aliases: [H-06]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-yellow]
> 🔧 **HARDWARE**

`#complete` `#advanced`

# H-06: Hardware Troubleshooting Masterclass

> [!abstract] Overview
> This note outlines advanced diagnostic workflows for identifying hardware failures, thermal anomalies, system instability, and peripheral errors. It includes diagnostic flowcharts, POST beep code matrices, tool applications, and documentation templates.

---
## 🧠 Concept Overview

- **What it is** — Hardware troubleshooting is a systematic diagnostic process to identify physical hardware failures.
- **Why it matters** — Proper diagnosis prevents unnecessary component replacement and minimizes downtime.
- **Where you see this** — No power, blue screens, random reboots, high temperatures, failed boot sequences.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check basic connections, power source, and run basic diagnostics like MemTest. |
| **L2** | Perform detailed part isolation, deep dive SMART analysis, component swap testing. |
| **L3** | Advanced board-level analysis, kernel memory dump debugging with WinDbg. |

> [!tip] Seedha Simple Mein
> *Hardware troubleshooting ek step-by-step detective work hai. Agar computer chal nahi raha, toh hum pehle sabse basic chizon ko check karte hain (power source, cables) aur dheere-dheere complex parts ki taraf badhte hain (RAM, CPU, Motherboard), diagnostic tools aur error codes ka use karke.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Troubleshooting hardware** is like **being a medical examiner** because...
>
> - The PC cannot tell you what hurts. You must gather vital signs.
> - Checking the power supply is like checking blood pressure.
> - Monitoring temperatures is like checking for a fever.
> - Reading BIOS beep codes is like listening to vocal symptoms.
> - Running stress tests is like an echo stress test.

---
## 🔬 Technical Deep Dive

### 1. Diagnostic Workflows (No Power & No Display)

> [!info] Key Concept
> Workflows help isolate the exact point of failure systematically rather than guessing. 

**Workflow A: No Power (System is Electrically Dead)**
Start with outlet power. If it's fine, do the Paperclip Test on the PSU. If the PSU fan spins, reconnect only the 24-pin and 8-pin CPU, then short the PW_SW header. If it turns on, the front-panel switch or external devices are at fault. If it doesn't, inspect the Motherboard for shorts.

**Workflow B: No Display (POST Fails)**
Verify display cables are in the GPU. Match monitor input source. Check Debug LEDs. Perform single-stick RAM test, moving across slots. Clear CMOS memory. Test with known good GPU or integrated graphics.

> [!danger] Common Mistake
> Replacing components before testing basic things like RAM seating or clearing CMOS. Always start with the easiest, lowest-cost checks.

### 2. Physical Thermal Overheating Diagnosis

- **Symptoms:** System throttling, loud fan noise, sudden thermal shutdowns (temperatures pegging at $100^\circ\text{C}$).
- **Causes:** Dried thermal paste, dust-clogged radiator fins, failing pump in an AIO liquid cooler, or fan failure.
- **Diagnostics:** Use HWiNFO64 and Prime95. Watch for instant $100^\circ\text{C}$ spikes indicating mounting/paste issues. If AIO pump RPM is 0 or one tube is hot/cold, the pump failed.

### 3. USB Device Issues: Power vs. Driver

- **Bandwidth/Power Overload:** USB 3.0 provides 900mA at 5V. Multiple high-draw devices cause brownouts.
- **Fix:** Connect directly to rear I/O or use an externally powered active USB hub.

### 4. AMI BIOS POST Beep Code Chart

| ⌨️ Beeps | 🛠️ Pattern | 📝 Meaning & Action |
|-----------|-----------------|-----------|
| **1 Short** | `.` | Normal POST. System healthy. |
| **1 Long, 2 Short** | `- . .` | Video adapter error. Reseat GPU. |
| **1 Long, 3 Short** | `- . . .` | Memory failure. Reseat RAM. |
| **5 Short** | `. . . . .` | CPU error. Check bent pins. |
| **8 Short** | `. . . . . . . .` | GPU memory failed. Replace GPU. |
| **Continuous** | `---` | RAM or Power Supply failure. |

### 5. Hardware Diagnostic Toolset Matrix

| ⌨️ Tool | 🛠️ Diagnostic Target | 📝 Usage Pattern |
|-----------|-----------------|-----------|
| **MemTest86** | System RAM | Run for at least 4 passes from bootable USB. |
| **CrystalDiskInfo** | Storage Health | Check SMART status in Windows. |
| **HWiNFO64** | Voltages & Temps | Monitor sensor maximums in Windows. |
| **Prime95** | CPU & VRM | Small FFTs test to generate heat. |
| **FurMark** | GPU Thermal | Stress test core and VRMs. |

### 6. Troubleshooting BSOD System Crashes

> [!info] Key Concept
> `WHEA_UNCORRECTABLE_ERROR` points to hardware failures like CPU voltage drops or NVMe disconnects.

Run WinDbg:
```cmd
windbg -z C:\Windows\Minidump\xxxxxx-xx.dmp
!analyze -v
```
Look for `MODULE_NAME` to find the hardware driver causing the crash (e.g., `nvlddmkm.sys`, `ntoskrnl.exe`).

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A desktop PC
> - Rufus software & an empty USB drive
> - Access to HWiNFO64, Prime95, and CrystalDiskInfo

### Step 1: Create a MemTest86 Bootable USB & Test RAM

```bash
# 1. Extract MemTest86, use USB Image Writer to create bootable drive.
# 2. Boot from USB, press S to start tests. 
# 3. Let it run 1 full pass.
```

> [!success] Expected Output
> `Errors: 0`
> Any error means faulty RAM or bad voltage configs.

### Step 2: Check Disk SMART Attributes

```bash
# Open CrystalDiskInfo and check the primary OS drive.
```

> [!success] Expected Output
> Health Status should be "Good". If "Caution" or "Bad" (IDs 05 and C5 are non-zero), replace the drive.

### Step 3: Run CPU Thermal Stress Test

```bash
# Open HWiNFO64 (Sensors-only) and Prime95 (Small FFTs).
```

> [!success] Expected Output
> CPU temp stabilizes below 85°C. If above 95°C and throttling, cooling has failed.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `sfc /scannow` | Check System File integrity | `sfc /scannow` |
| `DISM /Online ...` | Restore local component store | `DISM /Online /Cleanup-Image /RestoreHealth` |
| `dmesg -T \| grep ...` | Read Linux kernel messages for hardware errors | `sudo dmesg -T \| grep -i -E "error\|fail\|critical\|mce"` |
| `stress --cpu 8 ...` | Run CPU stress test on Linux for 60s | `stress --cpu 8 --timeout 60` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Workstation random reboots, no BSOD | Failing SMPS 12V rail dropping under load. | Monitor +12V sensor in HWiNFO64. If it drops below 11.4V under Prime95 load, replace PSU. |
| External USB HDD corrupts/disconnects | USB port power throttling (can't supply enough current). | Disable "Allow the computer to turn off this device" in Device Manager USB Root Hub, or use powered hub/Y-cable. |
| POST Fails (1 Long, 2 Short beeps) | Video adapter card error. | Reseat GPU, check PCIe power connections, replace GPU. |
| POST Fails (Continuous beep) | RAM or Power Supply failure. | Check power lines, test RAM, replace SMPS. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Unstable PC Rebooting

> [!example] Ticket
> "My computer randomly turns off and reboots while I'm working. No blue screen."

**L1 Response:** Verify power cable connections. Ask if it happens during specific tasks.
**Escalation Trigger:** If power cables and basic checks are fine, escalate to L2.
**L2 Resolution:** Run HWiNFO64 + Prime95 to monitor +12V rail. Found voltage dropping to 11.2V under load. Replaced the PSU.

### 🎫 Scenario 2: USB Disconnections

> [!example] Ticket
> "My external hard drive keeps disappearing from my PC in the middle of copying files."

**L1 Response:** Change the USB port, ideally to a rear motherboard port.
**Escalation Trigger:** If it still drops on rear ports.
**L2 Resolution:** Disabled power management on USB Root Hubs. Issue resolved. Alternatively, provided an active USB hub.

---
## 🎤 Interview Questions

> [!question] Q1: What does a WHEA_UNCORRECTABLE_ERROR (0x124) BSOD indicate, and what is your process for troubleshooting it?
> **Answer:** A critical hardware error occurred. Often caused by failing CPU core, insufficient CPU voltage, or failing PCIe device (like NVMe SSD). Analyze the memory dump file using WinDbg. Check if it points to CPU (`GenuineIntel`) or a PCIe storage controller.

==**Exam Tip:** WHEA always means HARDWARE. Stop troubleshooting software and check voltages/components.==

> [!question] Q2: A server in the data center boots, but the chassis fan controller spins all system fans at 100% continuously. Explain how you would troubleshoot this.
> **Answer:** Check the Out-of-Band management (iDRAC/iLO). This usually happens if the chassis cover is open (intrusion switch), a thermal sensor is failing, or an uncertified PCIe card was installed.

> [!question] Q3: What is the difference between a soft memory error and a hard memory error?
> **Answer:** Soft error is a temporary flip of a data bit (e.g., from cosmic rays/EMI). Hard error is a permanent physical defect in the silicon RAM chip, requiring replacement.

---
## 🔗 Related Notes

- [[01-Foundations/01-Hardware/H-01 Motherboard Architecture|H-01 Motherboard Architecture]] — Motherboard designs and BIOS recovery.
- [[01-Foundations/01-Hardware/H-02 Storage Deep Dive|H-02 Storage Deep Dive]] — SMART indicators and drive failures.
- [[01-Foundations/01-Hardware/H-03 SMPS Power Supply Complete Guide|H-03 SMPS Power Supply Complete Guide]] — Power rails and electrical troubleshooting.
