---
tags: [security, cia-triad, zero-trust, threat-protection, cryptography]
aliases: [cia-triad, zero-trust-model, cia-zero-trust]
created: 2026-06-25
status: "#complete"
difficulty: "#beginner"
cert-relevant: "#sc-900"
---

> [!NOTE|color-red]
> 🛡️ **SECURITY**

`#complete` `#beginner` `#sc-900`

# SEC-01: CIA Triad and Zero Trust

> [!abstract] Overview
> Yeh note IT security ke **do sabse important foundations** cover karta hai — **CIA Triad** (Confidentiality, Integrity, Availability) aur **Zero Trust Model** (Verify Explicitly, Least Privilege, Assume Breach). Har ek support engineer ko yeh samajhna zaroori hai kyunki tumhara har ek action — password reset se lekar backup restore tak — isi framework ke andar aata hai. Agar tum yeh nahi samajhte, toh tum unknowingly security breach ka door khol sakte ho.

---

## 🧠 Concept Overview

- **What it is** — **CIA Triad** IT security ka foundation model hai jo teen pillars pe based hai: **Confidentiality** (data ko unauthorized access se bachana), **Integrity** (data accurate aur untampered rahe), aur **Availability** (authorized users ko reliable access mile). **Zero Trust** ek modern security framework hai — *"Never trust, always verify"* — jo teen principles pe chalta hai: Verify Explicitly, Use Least Privilege, aur Assume Breach.
- **Why it matters** — Ek support engineer pehli line of defense hai. Tumhara har ek action directly CIA Triad se mapped hai — password reset (Confidentiality), file hash check (Integrity), backup restore (Availability). Zero Trust dictate karta hai ki tum remote users ko kaise manage karo aur devices ko kaise verify karo.
- **Where you see this** — MFA prompts troubleshoot karna, file permissions blocks explain karna, file hash integrity checks run karna, device compliance status check karna before access dena, Conditional Access policy issues debug karna.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | User identity verify karke password reset karta hai, MFA setup mein walk-through deta hai, backup availability monitor karta hai |
| **L2** | BitLocker encryption configure karta hai, file integrity audit (hashing tools) karta hai, role-based access permissions manage karta hai |
| **L3** | Tenant-wide Zero Trust architecture design karta hai, Conditional Access policies enforce karta hai, identity protection baselines establish karta hai, DR availability plans coordinate karta hai |

> [!tip] Seedha Simple Mein
> *Socho aise — CIA Triad matlab: **C** = Data koi dekh na sake jo dekhna nahi chahiye, **I** = Data koi badal na sake bina permission ke, **A** = Data jab chahiye tab mile. Aur Zero Trust matlab — chahe tu office mein baith ke kaam kar ya ghar se, har baar prove kar ki tu kaun hai. Koi bhi by default trusted nahi hai!*

---

## 💡 Real-World Analogy

> [!info] Think of it like this...
> **CIA Triad** is like a **bank locker system**:
>
> - 🔐 **Confidentiality** = Locker ki **key sirf tumhare paas** hai, bank manager bhi nahi khol sakta — *jaise AES encryption sirf authorized users ko data dekhne deta hai*
> - ✅ **Integrity** = Bank maintain karta hai ki locker mein jo **jewellery rakhi hai woh waisi ki waisi** rahe, koi tamper na kare — *jaise SHA-256 hash ensure karta hai file alter nahi hui*
> - 🟢 **Availability** = Bank **opening hours mein guaranteed accessible** hai, chahe festival ho ya nahi — *jaise geo-redundant backups ensure karte hain data recover ho sake*
>
> **Zero Trust** is like **airport security**:
>
> - ✈️ Chahe tum **frequent flyer** ho, har baar boarding pass dikhana padega (**Verify Explicitly**)
> - 🎫 Tumhe sirf **apni seat tak access** milta hai, cockpit mein nahi ja sakte (**Least Privilege**)
> - 🔎 Security assume karti hai ki **koi bhi dangerous ho sakta hai**, isliye sab ko scan karte hain (**Assume Breach**)

---

## 🔬 Technical Deep Dive

### 1. 🔺 The CIA Triad in Enterprise Operations

*CIA Triad ek balancing act hai — ek pillar mein security badhao toh doosre pe impact pad sakta hai (e.g., heavy encryption Confidentiality protect karta hai lekin Availability slow ho sakti hai).*

```
                            [CIA Triad Security]
                                 /    |    \
                                /     |     \
                               v      v      v
              [Confidentiality]   [Integrity]   [Availability]
              - Encryption (AES)  - Hashing     - Backups (RSV)
              - RBAC / Permissions - Signatures  - Redundancy (RA-GRS)
              - MFA Challenges    - Versioning  - Load Balancing
```

> [!info] Key Concept — Confidentiality
> Access sirf **authorized entities** tak restricted hona chahiye.
> - *Controls*: **AES encryption** for data at rest (BitLocker), **SSL/TLS** for data in transit, **MFA**, aur **Role-Based Access Control (RBAC)**

> [!info] Key Concept — Integrity
> Data **accurate, complete, aur unauthorized modification se protected** hona chahiye.
> - *Controls*: **Cryptographic hashing (SHA-256)**, digital signatures, database transaction logs, aur version history

> [!info] Key Concept — Availability
> Authorized users ko **uninterrupted access** milna chahiye jab bhi zaroori ho.
> - *Controls*: **RAID** storage arrays, **geo-redundant backups**, load balancers, **UPS** power backups, aur **DDoS protection**

> [!danger] Common Mistake
> Bahut log **Confidentiality aur Privacy** ko same samajhte hain. **Privacy** user ka right hai data control karna, **Confidentiality** ek technical control hai jo unauthorized access block karta hai. Interview mein yeh galti mat karna!

==**Exam Tip:** CIA ka C = Confidentiality, **na ki Central Intelligence Agency!** SC-900 mein yeh trick question aata hai.==

---

### 2. 🛡️ The Three Principles of Zero Trust

*Zero Trust purane **"Castle-and-Moat"** model ko replace karta hai (jahan corporate network ke andar sab trusted tha).*

| 🛡️ Principle | 📋 Description | 🛠️ Real Example |
|-------------|---------------|-----------------|
| **Verify Explicitly** | Har access request pe authenticate + authorize karo based on all data points (identity, location, device health) | MFA prompt har sign-in pe, chahe office se ho ya ghar se |
| **Use Least Privilege** | Sirf utna access do jitna kaam ke liye zaroori hai — JIT (Just-in-Time) + JEA (Just-Enough-Access) | Finance team ko sirf finance SharePoint access, HR portal nahi |
| **Assume Breach** | Maan lo ki breach ho chuka hai — blast radius minimize karo, segment karo, encrypt karo | Network micro-segmentation + end-to-end encryption + analytics |

> [!danger] Common Mistake
> Bahut log sochte hain ki **VPN lagaya toh Zero Trust ho gaya**. ❌ Zero Trust mein network location se trust nahi milta. Agar attacker tumhari home Wi-Fi compromise karke VPN se connect ho jaye, toh bhi internal network access ho jayega. **Device compliance + MFA har transaction pe zaroori hai!**

==**Exam Tip:** SC-900 mein pucha jayega — "Which Zero Trust principle recommends JIT and JEA?" Answer: **Use Least Privilege Access**==

---

### 3. 🏛️ The Six Pillars of Zero Trust Architecture

*Zero Trust sirf identity tak limited nahi hai — yeh **6 critical domains** mein controls integrate karta hai:*

| 🏛️ Pillar | 🛡️ Security Focus | 🛠️ Microsoft Tool |
|-----------|-------------------|-------------------|
| **1. Identities** | Strong MFA, risk policies, SSO se users verify karo | Microsoft Entra ID |
| **2. Devices** | Device compliance audit karo before access dena | Microsoft Intune |
| **3. Applications** | Shadow IT apps secure karo, runtime controls enforce karo | Defender for Cloud Apps |
| **4. Data** | Documents classify, label, aur encrypt karo | Sensitivity Labels / DLP |
| **5. Infrastructure** | Configuration drifts monitor karo, host servers isolate karo | Microsoft Defender for Cloud |
| **6. Networks** | Micro-perimeters se segment karo | VNets, NSGs, Azure Firewall |

==**Exam Tip:** SC-900 ke liye yeh 6 pillars yaad rakhna — **I-D-A-D-I-N** (Identities, Devices, Applications, Data, Infrastructure, Networks)==

---

## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Windows 11 workstation
> - PowerShell access (Run as Administrator recommended)
> - **Time Required:** 20 minutes

**🎯 Objective:** SHA-256 file hash calculate karo PowerShell mein, file modify karo, recalculate karke integrity failure verify karo.

### Step 1: Test folder aur document create karo

```powershell
# Step 1: Temp folder banao aur ek test file create karo
New-Item -Path "C:\Temp" -ItemType Directory -ErrorAction SilentlyContinue
Set-Content -Path "C:\Temp\contract.txt" -Value "This is the original contract text."
```

> [!success] Expected Output
> ```
> Directory: C:\
> Mode                 LastWriteTime         Length Name
> ----                 -------------         ------ ----
> d-----         6/26/2026  2:30 PM                Temp
> ```

### Step 2: Original file ka hash calculate karo

```powershell
# Step 2: SHA-256 hash generate karo aur store karo
$OriginalHash = (Get-FileHash -Path "C:\Temp\contract.txt" -Algorithm SHA256).Hash
Write-Host "Original Hash: $OriginalHash" -ForegroundColor Cyan
```

> [!success] Expected Output
> ```
> Original Hash: 7E2B5F3A9C1D4E8F6B0A2D5C7E9F1B3D4A6C8E0F2D4B6A8C0E2F4D6B8A0C2E
> ```

### Step 3: File mein unauthorized tampering simulate karo

```powershell
# Step 3: Sirf ek character change karo (period -> exclamation mark)
Set-Content -Path "C:\Temp\contract.txt" -Value "This is the original contract text!"
```

### Step 4: Naya hash calculate karo

```powershell
# Step 4: Modified file ka naya hash generate karo
$NewHash = (Get-FileHash -Path "C:\Temp\contract.txt" -Algorithm SHA256).Hash
Write-Host "New Hash: $NewHash" -ForegroundColor Yellow
```

### Step 5: Dono hashes compare karo

```powershell
# Step 5: Compare karo — agar match nahi karta toh INTEGRITY BREAK!
if ($OriginalHash -ne $NewHash) {
    Write-Host "⚠️ INTEGRITY BREAK DETECTED: The file was modified!" -ForegroundColor Red
} else {
    Write-Host "✅ File integrity intact." -ForegroundColor Green
}
```

> [!success] Expected Output
> ```
> ⚠️ INTEGRITY BREAK DETECTED: The file was modified!
> ```

**✅ Success Criteria:** Script successfully dono hashes calculate kare, mismatch detect kare (sirf ek character change se), aur integrity failure log kare.

> [!danger] Common Mistake
> Agar `Get-FileHash` fail ho raha hai toh check karo — kya file path sahi hai? Kya folder permissions block kar rahe hain? `Test-Path "C:\Temp\contract.txt"` se verify karo pehle.

---

## ⌨️ Command Cheat Sheet

### PowerShell

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-FileHash` | File ka SHA-256 hash calculate karta hai (Integrity check) | `Get-FileHash -Path "C:\Downloads\patch.msi" -Algorithm SHA256` |
| `Get-BitLockerVolume` | BitLocker encryption status dikhata hai (Confidentiality check) | `Get-BitLockerVolume -MountPoint "C:"` |
| `Get-MgUser` | Entra ID user details pull karta hai (Identity verification) | `Get-MgUser -UserId "user@domain.com"` |
| `Test-NetConnection` | Network availability test karta hai (Availability check) | `Test-NetConnection -ComputerName "server01" -Port 443` |
| `Get-ExecutionPolicy` | Script execution policy check karta hai (Integrity control) | `Get-ExecutionPolicy -List` |

```powershell
# File hash calculate karo aur vendor hash se compare karo
$Hash = (Get-FileHash -Path "C:\Downloads\update-agent.msi" -Algorithm SHA256).Hash
$ExpectedHash = "A567BCE902F1C34A999DB88F45D10E34F5C02D1E2A34F5678BCEF90D1C2A34B2"
if ($Hash -eq $ExpectedHash) {
    Write-Host "✅ Integrity Verified: File has not been modified." -ForegroundColor Green
} else {
    Write-Warning "⚠️ INTEGRITY FAILURE: File has been modified or corrupted!"
}
```

```powershell
# BitLocker volume encryption status check karo
Get-BitLockerVolume -MountPoint "C:"
```

### CMD / Run Box

```cmd
:: Network availability check — DNS server reachable hai ya nahi
ping 8.8.8.8

:: Certificate verification — corporate root certificate valid hai ya nahi
certutil -verify C:\Temp\corporate_root.cer
```

### GUI Path

| 🖥️ Tool | 🔗 Path | 🛠️ Kya karta hai |
|---------|--------|-----------------|
| **Security Portal** | `security.microsoft.com` → **Microsoft Secure Score** | Zero Trust compliance metrics audit karo |
| **Entra Center** | `entra.microsoft.com` → **Identity** → **Users** → **Sign-in logs** | Explicit authentication signals verify karo |
| **Intune Portal** | `intune.microsoft.com` → **Devices** → **Compliance** | Device compliance status check karo |

---

## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix | ⌨️ Command |
|-----------|----------------|-------|-----------|
| `Get-FileHash` error: "File not found" | File path galat hai ya file exist nahi karti | `Test-Path` se pehle verify karo ki file exist karti hai | `Test-Path "C:\Temp\file.msi"` |
| BitLocker status "Protection Off" | TPM chip disabled hai ya Group Policy ne enforce nahi kiya | BIOS mein TPM enable karo, phir BitLocker re-enable karo | `Enable-BitLocker -MountPoint "C:" -RecoveryPasswordProtector` |
| MFA prompt nahi aa raha sign-in pe | Conditional Access policy mein user excluded hai ya legacy auth use ho raha hai | Entra ID mein CA policy check karo, legacy auth block karo | Entra Portal → CA Policies → Check assignments |
| Service connection timeout | Firewall ya routing traffic block kar raha hai | Network route check karo, target ports firewall pe enable karo | `Test-NetConnection -ComputerName <ip> -Port <port>` |
| Access Denied error on file/resource | User account ke paas permissions nahi hain ya invalid credentials hain | Account access permissions verify karo ya password reset karo | `Get-Acl "C:\Path\To\Resource"` |
| Resource/path not found error | Object path misspelled hai ya resource delete ho chuka hai | Target path spelling verify karo aur active objects query karo | `Test-Path "C:\Path\To\Resource"` |
| Hash mismatch but file looks same | File encoding change ho gayi (UTF-8 vs UTF-16) ya invisible characters add hue | File encoding explicitly set karo while saving | `Get-Content file.txt \| Format-Hex` |

---

## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: File Integrity Verification Before Software Deployment

> [!example] Ticket
> "I downloaded the new system patch installer 'patch.msi' from a vendor's mirror site. Before I deploy this to all 500 employee desktops, I need to verify that the file was not corrupted during transit or tampered with by a malicious third party."

**L1 Response:**
1. Vendor ki official release page se **published SHA-256 hash** locate karo
2. PowerShell mein downloaded file ka hash calculate karo:
   ```powershell
   Get-FileHash -Path "C:\Temp\patch.msi" -Algorithm SHA256
   ```
3. Dono hashes **character-by-character compare** karo

**Escalation Trigger:** Agar hash mismatch ho toh file delete karo aur **immediately L2 ko escalate** karo — possible supply chain attack ho sakta hai.

**L2 Resolution:**
1. Vendor se direct download link leke fresh file download karo
2. Hash re-verify karo → match confirm hone pe deployment ticket mein document karo
3. Software Center mein verified installer upload karo

**Ticket Close Note:** *"Calculated SHA-256 file hash. Value matched vendor's published signature. Integrity verified. Proceeding with deployment. Closed."*

---

### 🎫 Scenario 2: Ransomware Outage — Database Availability Restore

> [!example] Ticket
> "Our Customer Relations database server is showing an error: 'Connection Refused'. The main application is down. We cannot process any client orders today." — Sales Department (URGENT P1)

**L1 Response:**
1. Database VM ka status check karo cloud portal pe — running hai ya nahi
2. Basic connectivity test karo: `Test-NetConnection -ComputerName "db-server01" -Port 1433`
3. Agar VM down hai ya unreachable hai toh **immediately L2 escalate** karo

**Escalation Trigger:** VM reachable hai lekin DB files encrypted/renamed hain (`.db.locked` extension) → **Security incident** — L2/L3 + Security team involve karo

**L2 Resolution:**
1. Infected VM ka OS drive format karo — malware remove karo
2. **Recovery Services Vault** mein navigate karo → database volume select karo
3. Latest clean snapshot recovery point select karo (pre-compromise)
4. Restore task run karo → DB files clean state mein verify karo
5. Database service restart karo → sales application connection confirm karo
6. **Root Cause:** Weak admin credential se server compromise hua — Integrity + Availability break

**Prevention:** Zero Trust enforce karo — DB port pe NSG restrict karo, admin accounts pe MFA mandatory karo, backups encrypt karo

**Ticket Close Note:** *"Database restored from clean recovery point to resolve ransomware outage. Restored Availability. Enforced strict NSG rules on DB port. Security incident report filed. Closed."*

---

### 🎫 Scenario 3: Social Engineering Attack — Unauthorized Password Reset Attempt

> [!example] Ticket
> Phone call: "Hi, I am Rahul from Finance. I'm locked out of my account and about to walk into a board meeting in 5 minutes. Please reset my password right now — my manager approved it verbally."

**L1 Response:**
1. ❌ **KABHI BHI pressure mein password reset mat karo!**
2. Caller se registered employee ID aur manager ka name pucho
3. Verified phone number pe **callback karo** — direct number se nahi, registered HR directory number se
4. Agar caller refuse kare ya details na de sake toh **social engineering attempt** flag karo

**Escalation Trigger:** Caller verification fail ho jaye ya suspicious behaviour dikhaye toh **Security team ko immediately escalate** karo

**L2 Resolution:**
1. Security team alert — possible social engineering attack log karo
2. Target user account pe **temporary sign-in block** lagao as precaution
3. Real employee se contact karo verified channels pe — actual situation confirm karo
4. Agar legitimate request hai toh verified identity ke baad password reset karo with MFA re-registration

**Ticket Close Note:** *"Received suspicious password reset call. Followed Verify Explicitly protocol. Caller could not verify identity. Security incident logged. No password was reset. Closed."*

==**Exam Tip:** SC-900 mein "Verify Explicitly" ka real-world example puchte hain — yeh scenario yaad rakhna!==

---

## 🎤 Interview Questions

> [!question] Q1: What does the CIA Triad stand for in IT security?
> **Answer:** CIA Triad stands for **Confidentiality** (ensuring only authorized users can see data), **Integrity** (ensuring data is accurate and not altered), aur **Availability** (guaranteeing users can access data when needed). Yeh IT security ka fundamental framework hai.

> [!question] Q2: Explain the three main principles of the Zero Trust security model.
> **Answer:** Teen principles hain: 1. **Verify Explicitly** — always authenticate and authorize based on all available signals like identity, location, aur device health. 2. **Use Least Privilege Access** — JIT aur JEA permissions se access limit karo. 3. **Assume Breach** — blast radius limit karo by segmenting networks, encrypting data, aur monitoring sessions.

> [!question] Q3: How does a cryptographic hash (like SHA-256) help ensure data integrity?
> **Answer:** Cryptographic hash algorithm ek file ko process karke unique, fixed-length string generate karta hai. Agar file mein **ek bit bhi change** ho, toh hash completely change ho jata hai. Calculated hash ko vendor ke published hash se compare karke verify karte hain ki file alter ya corrupt nahi hui. ==**Exam Tip:** SHA-256 produces a **256-bit hash** — yeh number yaad rakhna!==

> [!question] Q4: How does Zero Trust differ from the legacy Castle-and-Moat model?
> **Answer:** Castle-and-Moat model assume karta tha ki once a user is **inside the corporate network** (inside the moat), they are trusted with broad access. Zero Trust **kisi ko bhi by default trust nahi karta**, chahe network location koi bhi ho. Har sign-in aur access request explicitly verified, authorized, aur security compliance checked honi chahiye.

> [!question] Q5: Why does the helpdesk call users back on their registered phone number before resetting passwords?
> **Answer:** Yeh **"Verify Explicitly"** principle enforce karta hai. Attackers phone call karke employee ban sakte hain aur social engineering se credentials steal kar sakte hain. Registered phone number pe callback karke hum ensure karte hain ki request legitimate hai.

> [!question] Q6: You are designing a Zero Trust authentication flow for remote finance employees accessing a SQL database in Azure. Describe your approach.
> **Answer (STAR Format):**
> - **Situation:** Designing secure access flow for remote finance workers under Zero Trust
> - **Task:** Configure authentication, device, aur network controls
> - **Action:** **Identity pillar** — Microsoft Authenticator MFA with Number Matching enforce karo. **Device pillar** — Conditional Access policy se Intune-enrolled "Compliant" device require karo. **Network pillar** — NSG se SQL database access sirf Azure Virtual Desktop subnet se allow karo. **Data pillar** — SQL logs pe Sensitivity Labels se classification karo.
> - **Result:** Finance data end-to-end secured, sirf verified users on compliant corporate devices database access kar sakte hain.

> [!question] Q7 (Behavioral): Tell me about a time when business convenience clashed with security compliance.
> **Answer:** Ek director ne apni team ke liye MFA bypass karna chahta tha kyunki "client work slow ho raha hai." Maine directly security policy cite karne ki jagah, unke saath baithke recent credential stuffing attacks ke statistics dikhaye. Phir Maine unhe **Authenticator app configure karke passwordless phone sign-in + number matching** setup kiya, jisse logins fast ho gaye. Security baseline maintain raha aur efficiency concern bhi address hua.

==**Exam Tip:** Interview mein STAR format (Situation-Task-Action-Result) use karo — senior roles ke liye yeh mandatory hai!==

---

## 🔢 Key Numbers to Remember

| 🔢 Number | 📋 Kya hai |
|-----------|-----------|
| **3** | Zero Trust principles (Verify, Least Privilege, Assume Breach) |
| **6** | Zero Trust architecture pillars (I-D-A-D-I-N) |
| **256** | SHA-256 hash output bits |
| **443** | HTTPS port — secure web verification ke liye required |
| **3** | CIA Triad pillars (Confidentiality, Integrity, Availability) |

---

## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]] — Zero Trust rules enforce karne ka primary policy engine
- [[04-Cloud-and-Security/09-Security/Access-Management|Access Management]] — Least Privilege principle pe focus karta hai
- [[04-Cloud-and-Security/09-Security/MFA-and-Identity-Protection|MFA and Identity Protection]] — Identity verification controls detail mein

---
