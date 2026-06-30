---
tags: [desktop-support, windows-os, device-manager, drivers, L1]
aliases: [driver-troubleshooting, hardware-devices]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS**

`#complete` `#beginner` `#md-102`

# WIN-02: Device Manager

> [!abstract] Overview
> Yeh note Device Manager ko cover karta hai, jo ek built-in administrative tool hai jisse physical aur virtual hardware devices manage hote hain. Ek support engineer ke liye yeh janna zaroori hai taaki drivers update kar sake, hardware problems diagnose kar sake aur errors (jaise Code 10, Code 43) fix kar sake.

---
## 🧠 Concept Overview

- **What it is** — Windows Device Manager is a built-in administrative tool that provides a graphical representation of the physical and virtual hardware devices connected to a computer, allowing administrators to manage drivers, change hardware properties, and troubleshoot device errors.
- **Why it matters** — Operating system interaction with physical hardware relies on device drivers. A support engineer must know how to diagnose driver errors, roll back updates that cause instability, and locate drivers for unrecognized hardware.
- **Where you see this** — Troubleshooting network adapter failures (code 10), resolving USB controller disconnects (code 43), identifying unrecognized hardware components (code 28), and rolling back graphics drivers.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Identifies basic driver errors (yellow warning flags), updates drivers from local folders, and performs basic device resets. |
| **L2** | Troubleshoots driver conflicts, locates driver packages using hardware IDs, and rolls back updates. |
| **L3** | Establishes driver deployment baselines for enterprise OS deployments, configures Driver Signature Enforcement policies, and manages driver updates via Intune. |

> [!tip] Seedha Simple Mein
> *Device Manager computer hardware ka control panel hai. Agar koi hardware parts jaise keyboard ya Wi-Fi nahi chal raha toh idhar aakar drivers update ya rollback karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Operating System** is like a **busy manager** and **Hardware Devices** are like **foreign workers** because...
>
> - **Device Drivers** are the translators that allow the OS and hardware to understand each other. If the translator is missing or bad, communication breaks down.
> - **Device Manager** is the HR portal where you hire, fire, and manage these translators.

---
## 🔬 Technical Deep Dive

### 1. The Hardware Abstraction Layer (HAL) & Drivers

> [!info] Key Concept
> The HAL acts as a translation layer between the OS and hardware. The **Device Driver** is the translator script.

Windows does not communicate with hardware directly. The HAL acts as a translation layer. The **Device Driver** is the translator script. If the driver is missing, outdated, or corrupt, the translator fails, and the operating system cannot control the hardware.

> [!danger] Common Mistake
> Assuming hardware is broken without checking if the driver is simply missing or corrupt.

### 2. Common Device Manager Error Codes

When a device fails to initialize, Device Manager displays a yellow warning icon next to the device. Double-clicking the device reveals the specific error code:
- **Code 10 (This device cannot start)**: The device failed to initialize. Commonly caused by outdated, incompatible, or corrupt drivers.
- **Code 28 (The drivers for this device are not installed)**: The device is unrecognized. Typically occurs after a clean OS install when chipset or custom device drivers are missing.
- **Code 43 (Windows has stopped this device because it has reported problems)**: The hardware device or its driver reported a physical failure to the operating system. Can indicate a loose connection, power issue, or hardware fault.

### 3. Driver Signature Enforcement

To prevent rootkits and kernel-level malware from exploiting hardware, Windows enforces **Driver Signature Enforcement**. Only drivers signed with digital certificates from Microsoft's Windows Hardware Quality Labs (WHQL) can be loaded into memory. Unsigned drivers are blocked automatically during boot.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 11 VM or physical machine with local administrator rights.

### Step 1: Export Driver Package via PowerShell

```powershell
# Export the active driver package from the local Windows driver store to a backup folder
Export-WindowsDriver -Online -Destination "C:\Backups\Drivers"
```

> [!success] Expected Output
> ```
> Driver packages are successfully exported to C:\Backups\Drivers with .inf and .sys files present.
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `devmgmt.msc` | Launch the Device Manager console directly | `devmgmt.msc` |
| `pnputil /enum-drivers` | List all active third-party drivers installed | `pnputil /enum-drivers` |
| `pnputil.exe /add-driver` | Install a local driver package using pnputil | `pnputil.exe /add-driver "C:\Drivers\network.inf" /install` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Workstation has no network connection after Windows Update | Generic driver pushed via Windows Update was incompatible | Roll back the driver to the previous stable version in Device Manager. |
| Unrecognized Device in Device Manager (Code 28) | Missing required controller driver | Search the Vendor and Device IDs on PCI Lookup databases, download driver from manufacturer, and install. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Workstation Has No Network Connection After Windows Update

> [!example] Ticket
> "I restarted my computer to apply Windows updates, and now my internet connection is gone. The network icon shows a red X."

**L1 Response:** Check network adapter status in Device Manager and inspect physical connections.
**Escalation Trigger:** If rollback is unavailable or manual installation fails.
**L2 Resolution:** Diagnose Code 10 on NIC caused by Windows Update driver override. Navigate to the Driver tab and select 'Roll Back Driver' to restore the previous stable version. Connection is restored.

---
## 🎤 Interview Questions

> [!question] Q1: How do you identify an unrecognized hardware device in Device Manager if no driver is installed?
> **Answer:** I double-click the unrecognized device, navigate to the Details tab, select 'Hardware Ids' from the dropdown, and note the Vendor (VEN) and Device (DEV) codes. I then search these codes in public PCI lookup databases to identify the manufacturer and model to download the correct driver.

==**Exam Tip:** Code 10 means the device failed to initialize, often a driver issue. Code 28 means the driver is missing. Code 43 means the hardware reported a physical or power issue.==

---
## 🔗 Related Notes

- [[01-Foundations/01-Hardware/H-01 Motherboard Architecture|H-01 Motherboard Architecture]] — Connects physical interfaces under driver control.
- [[01-Foundations/01-Hardware/H-06 Hardware Troubleshooting Masterclass|H-06 Hardware Troubleshooting Masterclass]] — Diagnostic flows for component failures.
- [[02-Operating-Systems/03-Windows-OS/WIN-04 Registry|WIN-04 Registry]] — Details device class configuration keys.
