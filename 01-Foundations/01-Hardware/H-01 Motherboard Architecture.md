---
tags: [hardware, motherboard, sysadmin]
aliases: [mobo, H-01]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #comptia-aplus
---

> [!NOTE|color-yellow]
> 🔧 **HARDWARE**

`#complete` `#intermediate` `#comptia-aplus`

# H-01: Motherboard Architecture

> [!abstract] Overview
> Yeh note modern motherboards ki physical aur logical architecture cover karta hai. Ek support engineer ke liye chipsets, BIOS/UEFI, PCIe lanes aur form factors samajhna zaroori hai for hardware selection, PC building, aur low-level troubleshooting.

---
## 🧠 Concept Overview

- **What it is** — The main circuit board that connects all computer components together.
- **Why it matters** — CPU, RAM, aur GPU ki aapas mein communication aur power delivery motherboard par hi depend karti hai.
- **Where you see this** — During PC builds, hardware upgrades, and troubleshooting "No POST" or dead system issues.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check basic power cables, verify POST codes/beep codes, clear CMOS. |
| **L2** | Flash BIOS/UEFI updates, configure boot order, replace faulty motherboards. |
| **L3** | Server/Workstation platform architecture design, PCIe lane mapping, hardware compatibility testing. |

> [!tip] Seedha Simple Mein
> *Motherboard ek bada junction board hai jahan computer ka har ek part aakar connect hota hai. Agar CPU brain hai, toh motherboard body ki arteries aur veins hai jo sabhi parts ko aapas mein baat karne aur power share karne ki permission deti hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Motherboard** is like a **City's Infrastructure** because...
>
> - **CPU** is the City Hall (brain).
> - **Data buses/traces (PCIe)** are the roads connecting districts.
> - **VRMs/Power phases** are the power grids supplying electricity.
> - **BIOS/UEFI** are the security gates deciding who enters when the city boots up.

---
## 🔬 Technical Deep Dive

### 1. Form Factors

> [!info] Key Concept
> Motherboard form factors dictate physical size, case compatibility, and expansion capacity.

| 📏 Form Factor | 📐 Dimensions | 🔧 Expansion Slots (Max) | 💻 Best Use Case |
|---|---|---|---|
| **ATX (Standard)** | 12" × 9.6" | 7 | Standard Desktops, Workstations |
| **Micro-ATX (mATX)** | 9.6" × 9.6" | 4 | Budget builds, compact office PCs |
| **Mini-ITX (ITX)** | 6.7" × 6.7" | 1 | Small Form Factor (SFF) builds, HTPCs |

> [!danger] Common Mistake
> Installing an ATX motherboard into a case without checking standoff alignment. Unaligned standoffs can cause short circuits against the PCB traces.

### 2. Chipset Architecture (SoC & PCH)

Modern processors integrate the old Northbridge directly into the CPU (**System-on-Chip or SoC**). Slower I/O is handled by the **PCH (Platform Controller Hub)** on Intel or **Chipset** on AMD.

- **Intel Chipsets:** `Z-Series` (Overclocking, Max PCIe), `B-Series` (Mid-range), `H-Series` (Entry-level).
- **AMD Chipsets:** `X-Series` (Enthusiast, Max PCIe), `B-Series` (Mainstream), `A-Series` (Entry-level).

### 3. BIOS vs. UEFI

> [!info] Key Concept
> Firmware initializes hardware during the power-on sequence.

- **Legacy BIOS:** 16-bit, text interface, uses **MBR** (max 2.2 TB drives).
- **UEFI:** 32/64-bit, GUI/mouse support, uses **GPT** (up to 9.4 ZB).
  - **Secure Boot:** Verifies bootloader signatures to prevent boot-time malware.
  - **TPM 2.0:** Cryptographic module required for Windows 11.

### 4. CMOS & Power Delivery

- **CMOS:** Volatile RAM storing BIOS settings (clock, boot order). Powered by a `CR2032` 3V battery.
- **Power Connectors:** `24-pin ATX` for main board, `8-pin EPS` for CPU VRMs.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A physical PC to access UEFI/BIOS.

### Step 1: Accessing BIOS/UEFI

```bash
# Power on PC and immediately tap the BIOS hotkey repeatedly
# Common keys: Del (Custom PCs), F2 (Dell/Laptops), F10 (HP)
```

> [!success] Expected Output
> You will enter the motherboard's GUI settings screen.

### Step 2: Enable XMP / EXPO

1. Go to the **OC** or **Extreme Tweaker** section.
2. Enable **XMP** (Intel) or **EXPO / DOCP** (AMD) to run RAM at rated speeds.
3. Press **F10** to save and exit.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-CimInstance Win32_BaseBoard` | Windows mein motherboard details (model, serial) laata hai | `Get-CimInstance Win32_BaseBoard \| Format-List` |
| `Get-CimInstance Win32_Bios` | Windows mein BIOS version aur name check karta hai | `Get-CimInstance Win32_Bios` |
| `Confirm-SecureBootUEFI` | Check karta hai if Secure Boot is active | `Confirm-SecureBootUEFI` |
| `sudo dmidecode -t baseboard` | Linux mein motherboard ki details nikalta hai | `sudo dmidecode -t baseboard` |
| `sudo dmidecode -t bios` | Linux mein BIOS features aur version show karta hai | `sudo dmidecode -t bios` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| No POST (Blank Screen) | Corrupt BIOS settings or failed RAM OC | Clear CMOS via `CLRTC`/`JBAT1` pins or remove battery for 5 mins. |
| Clock resets to midnight | CMOS battery is dead | Replace the `CR2032` 3V lithium coin-cell battery. |
| NVMe SSD runs at half speed | SSD is in a chipset slot, not CPU-direct slot | Move SSD to the top M.2 slot closest to the CPU socket. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Post-BIOS Update BSOD

> [!example] Ticket
> "I updated the BIOS on the engineering workstation, and now Windows throws an 'Inaccessible Boot Device' BSOD on startup."

**L1 Response:** Verify that the system powers on and can access the BIOS screen.
**Escalation Trigger:** If boot order is correct but Windows still crashes.
**L2 Resolution:** A BIOS flash resets settings to defaults. Change the storage controller mode back to **AHCI/RAID** (whichever was used during OS install) and verify if **CSM (Legacy Boot)** needs to be toggled off to boot the UEFI Windows install.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between a PCIe lane routed to the CPU versus one routed to the PCH/Chipset?
> **Answer:** CPU-routed lanes connect directly to the processor, giving lowest latency and maximum bandwidth (used for primary GPU/NVMe). Chipset-routed lanes share a multiplexed uplink to the CPU, which can bottleneck if multiple devices transmit simultaneously.

==**Exam Tip:** PCIe Gen 4 bandwidth is double that of Gen 3 per lane (~1.97 GB/s vs ~985 MB/s).==

> [!question] Q2: How do you hardware-reset a motherboard if you can't access the BIOS menu?
> **Answer:** Shut down, unplug power, and short the CLRTC / JBAT1 jumper pins for 15 seconds. Alternatively, remove the CR2032 CMOS battery for 5 minutes.

---
## 🔗 Related Notes

- [[01-Foundations/01-Hardware/H-02 Storage Deep Dive|H-02 Storage Deep Dive]] — Detail on SATA, NVMe, and partition tables.
- [[01-Foundations/01-Hardware/H-05 PC Assembly and BIOS Configuration|H-05 PC Assembly and BIOS Configuration]] — Practical guide to system building and BIOS setup.
- [[01-Foundations/01-Hardware/H-06 Hardware Troubleshooting Masterclass|H-06 Hardware Troubleshooting Masterclass]] — Motherboard diagnostic beep codes and POST failures.
