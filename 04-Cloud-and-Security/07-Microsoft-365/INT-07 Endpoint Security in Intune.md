---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-07-endpoint-security-in-intune, int-07]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#advanced` `#md-102`

# INT-07: Endpoint Security in Intune

> [!abstract] Overview
> Yeh note Endpoint Security in Microsoft Intune par focus karta hai. Isme Defender AV policies, Attack Surface Reduction (ASR) rules, BitLocker encryption, aur Security Baselines cover hote hain, jo device security maintain karne ke liye zaroori hain.

---
## 🧠 Concept Overview

- **What it is** — Intune ka feature jo devices ki security configurations ko centralized manage karta hai (AV, Firewall, Encryption).
- **Why it matters** — Data breaches aur malware se bachane ke liye endpoints ka secure hona sabse important hai.
- **Where you see this** — Jab company ke laptops par automatically antivirus chalana ho, ya BitLocker encryption lagana ho bina user ko pareshan kiye.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check karna ki BitLocker on hai ya nahi, aur Entra ID se recovery key nikal kar user ko dena. |
| **L2** | Endpoint security policies banana, Defender AV rules set karna, aur ASR rules audit mode mein test karna. |
| **L3** | Complete zero-trust architecture design karna, Defender for Endpoint (MDE) integration aur custom baselines configure karna. |

> [!tip] Seedha Simple Mein
> *Endpoint Security blade ke zariye aap device ke local windows defender settings, firewall rules, ASR rules, aur BitLocker encryption ko direct cloud se lock down karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Endpoint Security** is like **a corporate fort's defense system** because...
>
> - Antivirus Policies are like guards checking photos of known bad actors.
> - Firewall Policies are like checkpoint controls for delivery trucks.
> - BitLocker is like a locked safe — agar koi safe chura le (drive chori ho jaye), bina combination (Recovery Key) ke nahi khulega.

---
## 🔬 Technical Deep Dive

### 1. Microsoft Defender Integration & ASR

> [!info] Key Concept
> Intune Defender for Endpoint (MDE) ke sath integrate karta hai to enforce security rules like Antivirus, Firewall, and Attack Surface Reduction (ASR).

ASR (Attack Surface Reduction) common malware entry paths ko block karta hai, jaise Office apps se child processes run hone dena, ya email attachments se scripts launch karna.

> [!danger] Common Mistake
> Enabling all ASR rules in "Enforce" mode instantly. ASR rules common behaviors block kar dete hain, jisse business apps ruk sakte hain. Always start with **Audit mode**.

### 2. BitLocker Encryption & Key Escrowing

> [!info] Key Concept
> Key Escrowing means backing up the recovery key to Microsoft Entra ID before starting encryption.

Requires TPM 2.0. Yeh silent encryption enforce karta hai bina user ke input ke.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Intune Admin Center access
> - A licensed Windows 11 client device with TPM 2.0 enabled

### Step 1: Configure BitLocker with Escrowing

```powershell
# Verify TPM and encryption status locally on client
manage-bde -status c:
```

> [!success] Expected Output
> ```
> Conversion Status:    Fully Encrypted
> Protection Status:    Protection On
> ```

1. Intune mein `Endpoint Security` -> `Disk encryption` pe jayein.
2. Create Policy -> Windows 10 and later -> BitLocker.
3. **Configure recovery folder** ko **Require backup to Azure AD** set karein.
4. **Compatible TPM startup PIN** ko **Blocked** karein (silent encryption ke liye).

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `manage-bde -status c:` | Encryption percentage aur TPM details dikhata hai | `manage-bde -status c:` |
| `manage-bde -protectors -get c:` | Recovery password ID aur key dikhata hai | `manage-bde -protectors -get c:` |
| `Get-MpComputerStatus` | Local Defender AV ka real-time protection status batata hai | `Get-MpComputerStatus \| Select-Object RealTimeProtectionEnabled` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| BitLocker policy error `0x8018001a` (TPM not found) | Device mein TPM chip disabled hai ya nahi hai | BIOS/UEFI mein jaakar TPM 2.0 ya Intel PTT enable karein, phir `tpm.msc` check karein. |
| Legacy app crashing after ASR rule enabled | App is reading `lsass.exe` which is blocked | Event Viewer log 1121/1122 check karein, aur app path ko ASR Exclusions mein daalein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Developer App Blocked by ASR

> [!example] Ticket
> "I can't run my internal testing script, Windows Defender keeps giving me a notification that the action is blocked."

**L1 Response:** Event Viewer `Operational` logs check karna for Event ID 1121/1122.
**Escalation Trigger:** Agar path confirm ho jaye ki genuine business app hai jo block ho raha hai.
**L2 Resolution:** Intune mein ASR profile open karke us specific developer directory path ko Exclusions list mein add karna.

---
## 🎤 Interview Questions

> [!question] Q1: How do you configure silent BitLocker deployment in Intune?
> **Answer:** Disk Encryption policy banakar, 'Require Device Encryption' enable karte hain, TPM require karte hain, 'Compatible TPM startup PIN' ko Blocked karte hain, aur Entra ID mein recovery key backup mandate karte hain.

==**Exam Tip:** BitLocker silent encryption fails if the device doesn't have an active TPM chip.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes identity joins and console controls.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Details compliance health checks.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details device configuration templates.
