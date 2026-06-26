---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-04-configuration-profiles, int-04]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#md-102`

# INT-04: Configuration Profiles

> [!abstract] Overview
> Yeh note Intune Configuration Profiles cover karta hai. Isme settings catalog, device restrictions, Wi-Fi/VPN configuration, policy assignments (filters ke through) aur conflict resolution rules explain kiye gaye hain. Ek support engineer ko yeh aana chahiye kyunki yahi se saari corporate policies remote devices par push hoti hain.

---
## 🧠 Concept Overview

- **What it is** — Configuration Profiles aapke organization ke devices ke remote control settings hote hain. Yeh cloud mein ek blueprint hota hai jise device connect hotey hi download kar leta hai.
- **Why it matters** — Manual configuration (jaise Wi-Fi password dalna, USB block karna) ko eliminate karta hai aur devices ko scale par secure karta hai.
- **Where you see this** — Jab naye devices enroll hote hain ya security policies enforce karni hoti hain, tab yeh use hota hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check karna ki profile apply hua hai ya pending/conflict mein hai. |
| **L2** | Assignments aur filters configure karna, conflicts resolve karna. |
| **L3** | Enterprise level pe OMA-URI custom profiles aur policy architecture design karna. |

> [!tip] Seedha Simple Mein
> *Configuration Profiles ke zariye aap devices ki OS settings remote control karte ho, jaise USB block karna, Wi-Fi profile push karna ya updates control karna. Jaise GPO on-premise kaam karta hai, waise hi yeh cloud mein.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Configuration Profiles** are like **House Rules for a Rented Apartment** because...
>
> - Tenant (Device) ko building (Company) mein ghuste hi list milti hai (blueprint).
> - Woh rules tay karte hain ki tenant kya kar sakta hai (Wi-Fi, updates) aur kya nahi kar sakta (USB block).

---
## 🔬 Technical Deep Dive

### 1. Configuration Profile Types

> [!info] Key Concept
> Intune mein multiple tarike se policies deploy hoti hain, lekin aajkal **Settings Catalog** primary method hai.

- **Settings Catalog**: Modern, search-first interface jahan sab settings milti hain.
- **Templates**: Legacy grouped profiles (e.g., Device Restrictions, VPN).
- **Administrative Templates (ADMX)**: Cloud mein available ported Windows GPOs.
- **Custom Profiles (OMA-URI)**: Custom paths unn settings ke liye jo GUI mein nahi hain.

### 2. Core Configurations

- **Removable Storage (USB Block)**: Write access ko block karna taaki data leak na ho. (OMA-URI: `./Device/Vendor/MSFT/Policy/Config/Storage/RemovableDiskDenyWriteAccess`)
- **Wi-Fi & VPN Profiles**: Network parameters deploy karta hai aur auto-connect triggers set karta hai.
- **Update Rings**: Quality/Feature updates ko defer karta hai aur active hours control karta hai.

### 3. Assignments & Conflicts

- **Groups vs. Filters**: Groups Entra ID based hote hain (slow evaluation). Filters real-time evaluation dete hain (e.g., target only Windows 11).
- **Conflict Rules**: Compliance settings Configuration ko override karte hain. Agar 2 Configuration Profiles alag value push karein, toh state **Conflict** ho jaati hai aur existing OS setting retain rehti hai.

> [!danger] Common Mistake
> Mixing different settings configurations for the same feature across Templates aur Settings Catalog. Yeh troubleshooting ko bahut mushkil bana deta hai.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Microsoft Intune Admin Center with Intune Administrator permissions
> - Test Windows 11 client device

### Step 1: Create a Device Restriction Profile

```bash
# GUI mein steps:
# 1. Devices -> Configuration -> Create -> New policy
# 2. Platform: Windows 10 and later, Profile type: Settings catalog
# 3. Add settings -> Search "Removable Storage" -> Deny write access (Enable)
# 4. Search "Lock Screen" -> Image URL set karna
```

> [!success] Expected Output
> Profile create hone ke baad, jab yeh device par sync hoga, USB drive mein write access "Access Denied" ho jayega.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-Content "C:\...\MDMDiagReport.xml"` | Client par Intune MDM logs check karta hai | `Get-Content -Path "C:\Users\Public\Documents\MDMDiagnostics\MDMDiagReport.xml"` |
| `Get-ChildItem "HKLM:\SOFTWARE\Microsoft\PolicyManager\current\device\"` | Registry path jahan Intune policies likhi jaati hain, use check karna | `Get-ChildItem -Path "HKLM:\...\device\"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Target devices "Pending" status mein fass gaye hain | Device offline hai ya internet nahi chal raha | Device ka internet check karein aur Company Portal se manual sync push karein. |
| Lock screen policy "Conflict" show kar rahi hai | Koi doosri profile ya GPO same setting override kar rahi hai | Intune dashboard mein conflict count par click karein, conflicting policy identify karein aur ek ko exclude karein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: VPN Auto-connect Issue

> [!example] Ticket
> "Company VPN is not connecting automatically when opening the web browser."

**L1 Response:** Check karega ki device par profile aayi hai ya nahi (Settings -> Accounts -> Info).
**Escalation Trigger:** Agar profile "Success" dikha rahi hai par phir bhi trigger kaam nahi kar raha.
**L2 Resolution:** Profile mein jayenge aur dekhenge ki **App-triggered VPN** rule mein browser ka sahi executable path configured hai ya nahi, aur certificate deploy hua hai ki nahi.

---
## 🎤 Interview Questions

> [!question] Q1: How does Intune resolve conflicts when two different configuration profiles apply to the same device with different values?
> **Answer:** Jab do profiles alag values push karti hain, toh state "Conflict" ho jaati hai. Device koi bhi value apply nahi karta aur existing OS state par hi rehta hai, jab tak admin conflict resolve na kare.

==**Exam Tip:** Settings Catalog is the recommended modern approach to building Intune Configuration Profiles over Legacy Templates.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Management authority and console layout
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Health states that override configuration profiles
- [[04-Cloud-and-Security/07-Microsoft-365/INT-05 Application Management|INT-05 Application Management]] — Application deployment profiles
