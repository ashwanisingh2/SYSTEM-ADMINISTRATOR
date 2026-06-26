---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-08-windows-update-management, int-08]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#md-102`

# INT-08: Windows Update Management

> [!abstract] Overview
> Yeh note Windows Update for Business (WUfB) ke through Intune mein updates manage karne ko cover karta hai. Isme Update Rings, deferral periods, driver updates aur deadlines ka concept samjhaya gaya hai.

---
## 🧠 Concept Overview

- **What it is** — Intune ka update management system jisse aap Windows quality aur feature updates ko schedule aur control kar sakte hain.
- **Why it matters** — Bina soche samjhe sabko ek sath update bhej dena systems crash kar sakta hai. Controlled rollout zaroori hai.
- **Where you see this** — Jab company ke saare laptops pe har mahine naye security patches bina kaam disturb kiye install karne hote hain.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | User ki machine pe updates check karna aur manual `usoclient` command chalana. |
| **L2** | Update Rings configure karna, deferral periods set karna aur specific updates ko pause karna agar bugs aayein. |
| **L3** | Enterprise patching strategy design karna, telemetry configure karna aur driver update policies manage karna. |

> [!tip] Seedha Simple Mein
> *Windows Update Management ke zariye aap devices par automatic updates aur driver patches ko schedule karte hain. Deferral periods se system crashes ko control kiya jata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Update Rings** are like **a campus water filtration system** because...
>
> - Aap raw paani (new updates) sabko ek sath nahi pilate.
> - Pehle Ring 1 (IT team) test karta hai.
> - Agar theek lage, toh 10 din baad Ring 3 (Production) ko supply kholi jati hai.
> - Koi bimar pada (bug aaya), toh Emergency lever (Pause) kheench diya jata hai.

---
## 🔬 Technical Deep Dive

### 1. Windows Update for Business (WUfB) Concepts

> [!info] Key Concept
> Quality Updates (monthly security patches) aur Feature Updates (annual major upgrades like 23H2) ko manage karne ki service.

- **Deferral Period**: Kitne din wait karna hai naya update deploy karne se pehle.
- **Active Hours**: Wo time jisme system auto-reboot nahi hoga (e.g. 8 AM to 5 PM).
- **Deadline settings**: Maximum din jab tak user restart delay kar sakta hai, uske baad force reboot hoga.

> [!danger] Common Mistake
> Setting a quality update deadline to `0` days. Isse machine immediate restart ho jati hai aur user ka active work lost ho sakta hai.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Intune Admin Center access
> - Test Windows 11 clients

### Step 1: Create the IT Testing Update Ring (Fast Ring)

```powershell
# Force local client machine to search for updates immediately
usoclient StartScan
```

> [!success] Expected Output
> ```
> Command runs silently and triggers background update scan.
> ```

1. Intune -> `Devices` -> `Manage updates` -> `Windows 10 and later update rings`.
2. Name it `IT Testing - Fast Update Ring`.
3. Set **Quality update deferral** to `0` days.
4. Set **Active hours** to 8 AM - 5 PM.
5. Assign to the IT group.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `usoclient StartScan` | Background mein turant naye updates scan karta hai | `usoclient StartScan` |
| `usoclient StartInstall` | Downloaded updates ko install karna shuru karta hai | `usoclient StartInstall` |
| `wusa /uninstall /kb:NUMBER` | Specific update ko command line se remove karta hai | `wusa /uninstall /kb:5032190 /quiet /norestart` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Update causing BSOD on production machines | Monthly update has a bug conflicting with hardware | Intune Update Ring mein Pause click karein, aur `wusa` se uninstall karein. |
| Devices show "Not Started" in Intune reports despite being updated | Diagnostic/Telemetry data blocked | Settings Catalog mein System\Allow Telemetry ko Basic/Full karein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Unplanned Reboots During Meetings

> [!example] Ticket
> "My laptop restarted automatically during my client presentation to install an update!"

**L1 Response:** User ke local 'Active Hours' check karna.
**Escalation Trigger:** Agar Active Hours centrally theek se enforce nahi ho rahe.
**L2 Resolution:** Intune Update Ring configuration mein User Experience settings check karke Active Hours (e.g. 8 AM - 6 PM) set karna aur deadline grace period allow karna.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between update deadlines and grace periods?
> **Answer:** Deadline ka matlab hai ki kitne din baad system force reboot karega (e.g. 3 days). Grace period wo extra buffer time hai jo un logon ko milta hai jo kai dino se offline the (jaise chutti se wapas aayein), taaki unka PC khulte hi restart na ho jaye.

==**Exam Tip:** Windows Update reports require Diagnostic Data (Telemetry) to be enabled on clients, otherwise devices will show as 'No Status'.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Sets up the tenant MDM authority.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details telemetry settings.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-07 Endpoint Security in Intune|INT-07 Endpoint Security in Intune]] — Manages Windows Defender updates.
