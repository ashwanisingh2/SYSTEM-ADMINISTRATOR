---
tags: [security, windows, fundamentals, uac, defender, firewall]
aliases: [sec-01]
created: 2026-06-26
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-red]
> 🛡️ **SECURITY**

`#complete` `#beginner` `#none`

# SEC-01: Windows Security Fundamentals

> [!abstract] Overview
> *Yeh note Windows system security ke basics ko cover karta hai. Ek L1 support engineer ko pata hona chahiye ki Local Security Policies kya hoti hain, User Account Control (UAC) kaise kaam karta hai, aur Windows Defender ka kya role hai. Windows machines ko secure rakhne ke liye in fundamentals ka knowledge zaroori hai. Is note mein hum core security components aur unke daily IT support mein uses ko samjhenge.*

---
## 🧠 Concept Overview

- **What it is** — Windows Security Fundamentals is a collection of built-in features and policies designed to protect the OS, user data, and network from unauthorized access and malicious software. It encompasses UAC, Defender, Firewall, and Local Security Policies.
- **Why it matters** — *System ko hack hone se bachane aur company ka data secure rakhne ke liye yeh bahut important hai. End-users aksar aisi mistakes karte hain jisse security compromise ho sakti hai (jaise phishing emails ya unauthorized software). In fundamentals ke properly configured hone se risk minimize hota hai.*
- **Where you see this** — Password expiration policies, UAC prompts when installing software, Windows Defender virus scans, and restricted access to sensitive folders.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — *Check basic security settings like UAC, Windows Firewall status, reset passwords, and run basic Defender scans.* |
| **L2** | Configure, fix, escalate kab karta hai — *Configure Local Security Policies (GPO), analyze Defender logs, troubleshoot complex permissions issues, and handle malware quarantine alerts.* |
| **L3** | Architecture, design, enterprise-level — *Design enterprise-wide Group Policy Objects (GPOs), implement Microsoft Defender for Endpoint (MDE), establish security baselines, and design Zero Trust architectures.* |

> [!tip] Seedha Simple Mein
> *Jaise ghar ki security ke liye main gate ka lock, CCTV, aur guard zaroori hai, waise hi Windows mein Firewall, UAC, aur Defender system ko safe rakhte hain. Har feature ka apna ek specific role hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Corporate Office Building Security System** is like **Windows Security Fundamentals** because...
>
> - **Security Guard at the Gate:** Is like **User Account Control (UAC)**. Yeh har baar confirm karta hai jab koi bada change (jaise naya software install karna) hota hai. *Agar ID card (Admin rights) nahi hai, toh entry denied.*
> - **CCTV Cameras & Patrolling Guards:** Is like **Windows Defender / Antivirus**. Yeh hamesha background mein monitor karta rehta hai ki koi malicious activity toh nahi ho rahi.
> - **Door Access Control (Swipe Cards):** Is like **NTFS Permissions**. Kis employee ko server room ka access dena hai aur kise sirf pantry ka.
> - **Rules for Residents/Employees:** Is like **Local Security Policies**. Yeh decide karte hain ki kaunsa user kya kar sakta hai (e.g., minimum password length, logon hours).

---
## 🔬 Technical Deep Dive

### 1. User Account Control (UAC)

> [!info] Key Concept
> User Account Control (UAC) helps prevent malware from damaging a PC and helps organizations deploy a better-managed desktop. It ensures apps run with standard privileges unless explicitly authorized.

*UAC ensure karta hai ki koi bhi software bina admin permission ke system level changes na kar sake. Agar user Standard hai, toh usko admin credentials dene padenge. Iske 4 levels hote hain slider mein (Always notify se lekar Never notify tak).*

> [!danger] Common Mistake
> *Kai baar log UAC ko puri tarah disable (Never notify) kar dete hain taaki unhe baar-baar prompts na aayein. Yeh ek bahut bada security risk hai kyunki isse malware bina permission ke install ho sakta hai.*

### 2. Windows Defender (Antivirus & Firewall)

> [!info] Key Concept
> Windows Defender is the built-in anti-malware and antivirus software in Windows, alongside Windows Defender Firewall which controls network traffic based on predefined rules.

*Windows Defender realtime protection provide karta hai. Agar koi infected file download hoti hai, toh yeh use turant block ya quarantine kar deta hai. Firewall incoming aur outgoing connections ko manage karti hai.*

### 3. Local Security Policy (`secpol.msc`)

> [!info] Key Concept
> Local Security Policy is used to edit account policies and local policies on a standalone computer. In domain environments, these are overridden by Group Policy Objects (GPOs).

*Isme Password Policies (jaise password history, length, complexity) aur Account Lockout Policies set ki jaati hain.*

> [!success] Quick Win
> *Agar aapko check karna hai ki kitni baar galat password dalne pe account lock hoga, toh seedhe `secpol.msc` kholo aur Account Lockout Policy check karo.*

### 4. BitLocker Drive Encryption

> [!info] Key Concept
> BitLocker is a full volume encryption feature included with Windows designed to protect data by providing encryption for entire volumes.

*Agar koi laptop chori ho jaye, toh hard drive nikal kar bhi data access nahi kiya ja sakta bina BitLocker Recovery Key ke.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Administrator access on a Windows 10/11 machine.
> - PowerShell basic knowledge.

### Step 1: Open Local Security Policy and Configure Password Rules

```powershell
# Yeh command Local Security Policy snap-in ko open karta hai
secpol.msc
```

1. Navigate to **Account Policies** -> **Password Policy**.
2. Double-click on **Password must meet complexity requirements**.
3. Set it to **Enabled**.
4. Click **OK**.
5. *Ab se koi bhi naya password alphabet, numbers, aur special characters mix karke banana padega.*

### Step 2: Check Windows Defender Status via PowerShell

```powershell
# Yeh command Defender ka real-time protection status check karta hai
Get-MpComputerStatus | Select-Object RealTimeProtectionEnabled, AntivirusSignatureAge
```

> [!success] Expected Output
> ```
> RealTimeProtectionEnabled AntivirusSignatureAge
> ------------------------- ---------------------
>                      True                     1
> ```
> *True ka matlab protection on hai. Age 1 ka matlab signatures recent hain.*

### Step 3: View Active Windows Firewall Profiles

```powershell
# Yeh command teeno Firewall profiles (Domain, Private, Public) ka status check karti hai
Get-NetFirewallProfile | Format-Table Name, Enabled
```

> [!success] Expected Output
> ```
> Name    Enabled
> ----    -------
> Domain     True
> Private    True
> Public     True
> ```

### Step 4: Review Event Viewer for Security Logs

```powershell
# Yeh command aakhiri 5 failed login attempts ko show karti hai
Get-EventLog -LogName Security -InstanceId 4625 -Newest 5
```

> [!success] Expected Output
> *A list of Event ID 4625 logs, showing time and user account that failed to log in.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `secpol.msc` | *Local Security Policy open karta hai* | `secpol.msc` |
| `wf.msc` | *Windows Defender Firewall with Advanced Security kholta hai* | `wf.msc` |
| `Get-MpComputerStatus` | *Windows Defender ka current status batata hai* | `Get-MpComputerStatus` |
| `Update-MpSignature` | *Defender ke definitions ko manually update karta hai* | `Update-MpSignature` |
| `UserAccountControlSettings.exe` | *UAC settings slider open karta hai* | `UserAccountControlSettings.exe` |
| `Get-NetFirewallProfile` | *Check karta hai konsi firewall profile active/enabled hai* | `Get-NetFirewallProfile` |
| `manage-bde -status` | *BitLocker encryption ka status check karta hai* | `manage-bde -status c:` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| UAC Prompt is blocking standard users | *Software require admin rights but user is standard.* | *Right-click -> Run as Administrator aur L2 se admin credentials enter karwao (via LAPS).* |
| Windows Defender is off | *Another 3rd party antivirus is installed, ya policy se disabled hai.* | *Check if McAfee/Symantec is installed. Ya fir `Get-MpPreference` check karo.* |
| Account locked out frequently | *User galat password enter kar raha hai ya koi background app purana password use kar raha hai.* | *Active Directory mein account unlock karo aur user ko Credential Manager clear karne ko bolo.* |
| Ping is not working between 2 PCs | *Windows Firewall ICMP packets ko block kar rahi hai by default.* | *Firewall mein "File and Printer Sharing (Echo Request - ICMPv4-In)" rule enable karo.* |
| User prompted for BitLocker Recovery Key | *Motherboard BIOS update hua ya hardware change hua.* | *Azure AD / Active Directory mein recovery key dhundo aur enter karo.* |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Account Lockout Loop

> [!example] Ticket
> "My Windows account keeps getting locked every 30 minutes even though I am not typing the wrong password."

**L1 Response:** *Check AD for lockout status. User se confirm karo ki unka email phone pe logged in toh nahi jiska password expire ho chuka ho.*
**Escalation Trigger:** *Agar account bar-bar lock ho raha hai without any clear reason (possible brute force or persistent stale credential issue).*
**L2 Resolution:** *Check Event Viewer (Security Log Event ID 4740) on the Domain Controller to find the source computer/device causing the lockout. Trace the IP address.*

### 🎫 Scenario 2: Software Installation Blocked by UAC

> [!example] Ticket
> "I am trying to install Adobe Reader but a popup asks for an IT admin password. I cannot proceed."

**L1 Response:** *Check software catalog if the user is allowed to have this software. Use LAPS (Local Administrator Password Solution) or admin credentials remotely to allow the installation.*
**Escalation Trigger:** *If the software is unapproved but user claims it's business critical.*
**L2 Resolution:** *Review software request process. Validate security risk. Deploy via SCCM/Intune if approved globally.*

### 🎫 Scenario 3: Windows Defender Flagged a Legitimate File

> [!example] Ticket
> "Windows Defender deleted an internal company tool saying it's a 'Trojan:Win32/Generic'. It's not a virus!"

**L1 Response:** *File ko quarantine se check karo. Ask user to hold on while it's investigated. Do NOT blindly restore it without checking.*
**Escalation Trigger:** *Security tools ka false positive is always an L2/L3 or Security Team issue.*
**L2 Resolution:** *Check Defender logs. Upload file to VirusTotal if allowed. Add the specific internal file hash to the Defender exclusion list via Group Policy / Intune.*

### 🎫 Scenario 4: User Cannot Access a Network Share

> [!example] Ticket
> "I am getting an Access Denied error when trying to open the HR Shared folder."

**L1 Response:** *Check NTFS permissions and Share permissions on the folder. Verify user's group memberships.*
**Escalation Trigger:** *If permissions seem correct but access is still failing, or if it involves cross-domain authentication.*
**L2 Resolution:** *Check Effective Access in Advanced Security Settings. Verify Kerberos tickets or token bloat issues.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the purpose of UAC (User Account Control)?
> **Answer:** *UAC ensures that administrative changes require explicit authorization. Yeh background malware ko automatically system files modify karne se rokata hai.*

==**Exam Tip:** UAC does not prevent users from running apps; it only prevents apps from elevating privileges without explicit consent.==

> [!question] Q2: How do you configure a password policy on a standalone Windows machine vs Domain?
> **Answer:** *On standalone, I would use `secpol.msc` to open the Local Security Policy. In a domain, I would configure a Group Policy Object (GPO) applied at the domain level, which overrides local policies.*

> [!question] Q3: What is the difference between Windows Firewall and Windows Defender?
> **Answer:** *Windows Firewall network traffic ko block/allow karta hai (ports and IP basis par). Windows Defender ek antivirus/anti-malware tool hai jo files aur processes ko scan karta hai for malicious code.*

> [!question] Q4: Where can you find audit logs if someone tries to log in with a wrong password?
> **Answer:** *Event Viewer ke andar Windows Logs -> Security section mein. Event ID 4625 indicates a failed login attempt. Event ID 4624 is for a successful login.*

> [!question] Q5: What is BitLocker and why is a TPM chip required?
> **Answer:** *BitLocker is a volume encryption tool. TPM (Trusted Platform Module) stores the encryption keys securely. Agar hard drive nikali jaaye, toh bina TPM ke ya bina recovery key ke data decrypt nahi hoga.*

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/09-Security/SEC-02 Windows Firewall Rules|SEC-02 Windows Firewall Rules]] — *Firewall concepts in detail*
- [[02-Windows/03-Active-Directory/AD-05 Group Policy Basics|AD-05 Group Policy Basics]] — *How security policies are applied across domain*
- [[04-Cloud-and-Security/09-Security/SEC-03 BitLocker Deep Dive|SEC-03 BitLocker Deep Dive]] — *Detailed note on disk encryption*
