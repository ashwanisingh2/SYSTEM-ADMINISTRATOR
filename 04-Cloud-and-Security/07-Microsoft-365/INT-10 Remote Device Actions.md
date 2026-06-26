---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-10-remote-device-actions, int-10]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#md-102`

# INT-10: Remote Device Actions

> [!abstract] Overview
> Yeh note Microsoft Intune ke remote device actions cover karta hai, jaise Sync, Restart, Wipe, Retire aur Remote Lock. Ek support engineer ko aana chahiye taaki wo remote laptops ko manage, secure aur decommission kar sake bina device physically apne paas laye.

---
## 🧠 Concept Overview

- **What it is** — Cloud console se commands bhejna jo remote devices (Windows, iOS, Android) par execute hoti hain.
- **Why it matters** — Jab koi laptop chori ho jaye, ya user resign kar de, toh data remotely secure ya wipe karna bohot zaroori hai.
- **Where you see this** — Troubleshooting ke liye remote restart/sync karna, ya terminated employee ke BYOD phone se company data nikalna (Retire).

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | User ki request par device Sync karna ya remote lock karna. |
| **L2** | Diagnostic logs collect karna, Fresh Start/Autopilot Reset run karna aur stolen laptops wipe karna. |
| **L3** | Remote assistance integrations (TeamViewer/Remote Help) configure karna aur wipe policies define karna. |

> [!tip] Seedha Simple Mein
> *Remote actions ke zariye aap devices ko cloud console se control kar sakte hain, jaise phone/laptop lock karna, diagnostic reports retrieve karna, aur device ka poora data wipe karna.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Remote Device Actions** are like **a fleet management system for delivery cars** because...
>
> - **Sync**: Jaise car ka radio signal check karna ki naye route updates aaye hain ya nahi.
> - **Retire**: Driver ki personal car se sirf company ka logo nikalna (personal data safe rehta hai).
> - **Wipe**: Car ka self-destruct mechanism on karna jisse sab kuch erase ho jaye.
> - **Fresh Start**: Car ka engine nikal ke ek naya factory clean engine laga dena.

---
## 🔬 Technical Deep Dive

### 1. Device Clean Wipes: Key Differences

> [!info] Key Concept
> Har wipe action ka alag purpose hota hai. Retire BYOD ke liye hai, Wipe stolen devices ke liye.

- **Retire (Selective Wipe)**: Sirf corporate data aur policies nikalta hai. Personal photos/files safe rehte hain.
- **Wipe (Factory Reset)**: Pura device clean karke factory default pe lata hai. Sab delete ho jata hai.
- **Fresh Start**: OEM bloatware remove karke ek clean Windows OS dalta hai.
- **Autopilot Reset**: Local data wipe karke next user ke liye ready karta hai, but Entra ID join status aur Autopilot enrollement banaye rakhta hai.

> [!danger] Common Mistake
> Kisi BYOD user ke phone par 'Wipe' click kar dena. Isse uski personal photos aur files bhi delete ho jayengi aur company pe liability aa sakti hai. Hamesha **Retire** use karein.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Intune Admin Center access
> - Enrolled test Windows 11 device

### Step 1: Trigger a Remote Device Sync

```powershell
# Force manual sync on Windows client to pull down remote actions
C:\Windows\System32\deviceenroller.exe /c /mobiledeviceenrollmenttoday
```

> [!success] Expected Output
> ```
> System checks in with Intune immediately in the background.
> ```

1. Intune -> `Devices` -> `All devices` pe jaayein.
2. Apne test Windows 11 device pe click karein.
3. Top bar mein **Sync** button click karein aur confirm karein.
4. Kuch minute baad 'Last contact' timestamp verify karein.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `deviceenroller.exe /c /mobiledeviceenrollmenttoday` | Client PC se MDM sync force karta hai | `deviceenroller.exe /c /mobiledeviceenrollmenttoday` |
| `reagentc /info` | Windows Recovery Environment status check karta hai | `reagentc /info` |
| `reagentc /enable` | Windows Recovery Environment ko enable karta hai (Fresh Start fail hone par) | `reagentc /enable` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Wipe command stuck on "Pending" indefinitely | Device offline hai ya turned off hai | Wait karein. Device Entra ID se delete na karein varna trust link toot jayega. Sirf device object ko disable karein Entra ID mein. |
| Fresh Start fails with error `0x80070002` | Local Windows Recovery Environment (WinRE) disabled ya corrupted hai | Run `reagentc /info` and `reagentc /enable` locally on the client. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Stolen Corporate Laptop

> [!example] Ticket
> "My laptop was stolen from my car yesterday. I had some important spreadsheets saved on the desktop."

**L1 Response:** Incident log karein aur immediately L2/Security team ko inform karein.
**Escalation Trigger:** Stolen laptop with potential corporate data exposure.
**L2 Resolution:** Intune mein jake **Wipe** command trigger karna aur 'secure wipe' option select karna. Phir Entra ID mein user ka password reset karna, active sessions revoke karna, aur device object disable karna.

---
## 🎤 Interview Questions

> [!question] Q1: Explain the difference between Retire and Wipe.
> **Answer:** Retire is a selective wipe jo sirf corporate apps, certificates aur data remove karta hai, ideal for BYOD. Wipe is a full factory reset jo device ka poora data erase karke usko default setup screen pe le aata hai, ideal for lost/stolen corporate devices.

==**Exam Tip:** Autopilot Reset Entra ID aur Intune connection barkarar rakhta hai jabki Wipe device ko console se bhi unenroll kar deta hai!==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes identity joins and MDM authority.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-02 Device Enrollment|INT-02 Device Enrollment]] — Details device onboarding flows.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-05 Application Management|INT-05 Application Management]] — Details application deployment profiles.
