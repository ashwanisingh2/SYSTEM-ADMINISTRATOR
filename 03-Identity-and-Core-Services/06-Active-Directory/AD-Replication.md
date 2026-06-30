---
tags: [active-directory, replication, identity, infrastructure]
aliases: [ad-replication, ad-sync, domain-controller-replication]
created: 2026-06-25
status: "#complete"
difficulty: "#intermediate"
cert-relevant: "#az-104"
---

> [!NOTE|color-purple]
> 🔐 **ACTIVE DIRECTORY**

`#complete` `#intermediate` `#az-104`

# AD-06: Active Directory Replication

> [!abstract] Overview
> Yeh note Active Directory Replication ko cover karta hai — yaani kaise ek Domain Controller (DC) par kiya gaya koi bhi change (password reset, user creation, GPO update) doosre saare DCs par automatically copy hota hai. Yeh **multi-master replication** model follow karta hai. Ek L1 support engineer ko yeh samajhna zaroori hai kyunki replication delay sabse common reason hai password sync failures, login issues, aur GPO apply na hone ka.

---

## 🧠 Concept Overview

- **What it is** — Active Directory Replication wo mechanism hai jisme ek DC par kiya gaya change (jaise password reset, user creation, group policy modification) automatically baaki saare Domain Controllers par copy/sync hota hai. Yeh **multi-master database replication** model use karta hai.
- **Why it matters** — Replication delays sabse zyada ticket escalations ka reason hai. Jaise agar user ne ek office mein password reset kiya, lekin doosre branch mein login nahi ho raha — kyunki local DC ko update nahi mila.
- **Where you see this** — Password sync delays across geographic locations, new Group Policy deployments ka validation, DC offline hone par replication blocks resolve karna.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | User symptoms collect karta hai (e.g., password reset ek machine par work nahi kar raha lekin web mail par work karta hai), client ka local DC identify karta hai, aur escalate karta hai agar zaroorat ho |
| **L2** | Standard cmdlets (`repadmin`) use karke basic replication status verify karta hai aur manual replication force karta hai immediate logon blocks resolve karne ke liye |
| **L3** | AD Sites and Services configure karta hai, IP subnets aur site links manage karta hai, complex sync issues fix karta hai (jaise USN Rollbacks aur Lingering Objects), aur forest-wide replication health monitor karta hai |

> [!tip] Seedha Simple Mein
> *Socho Active Directory ek WhatsApp group hai jisme saare Domain Controllers members hain. Jab koi ek DC par change karta hai, toh woh message baaki saare DCs ko bhi jaata hai — lekin agar kisi ka internet slow hai (WAN link) toh message late pahunchta hai. Yehi hai replication delay!*

---

## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Active Directory Replication** is like a **chain of post offices** in different cities.
>
> - Jab koi ek post office mein naya address register hota hai, toh woh update baaki saare post offices ko bhejna padta hai
> - **Intra-Site** = Same city ke post offices — update turant chala jaata hai (15 seconds mein notification)
> - **Inter-Site** = Doosre city ke post offices — update scheduled truck se jaata hai (default 180 minutes mein)
> - Agar koi post office 6 mahine se band tha aur phir khula, toh uske paas purani records hain — yeh hai **Lingering Objects**
> - Agar kisi ne galti se purani diary restore kar di, toh numbers confuse ho jaate — yeh hai **USN Rollback**

---

## 🔬 Technical Deep Dive

### 1. Replication Topology: Intra-Site vs. Inter-Site

Active Directory replication ko **Sites** ke basis par organize karta hai *(Sites = physical IP subnets jo high-speed networks se connected hain)*:

| 🏷️ Feature | 🏠 Intra-Site Replication | 🌍 Inter-Site Replication |
|-----------|------------------------|------------------------|
| **Location** | Same AD Site ke andar (local LAN) | Different AD Sites ke beech (WAN / VPN) |
| **Trigger** | **Change Notification**: DC apne partners ko 15 seconds baad notify karta hai | **Scheduled**: Specified intervals par hota hai (default: 180 min, min: 15 min) |
| **Compression** | Uncompressed *(CPU bachata hai)* | Highly compressed *(WAN bandwidth bachata hai)* |
| **Protocols** | RPC over IP | IP (RPC) ya SMTP (sirf Schema/Configuration ke liye) |
| **Topology Maker** | Knowledge Consistency Checker (KCC) | Inter-Site Topology Generator (ISTG) |

> [!info] Key Concept
> **KCC (Knowledge Consistency Checker)** — Yeh ek built-in process hai jo har 15 minutes mein saare DCs par chalta hai. Yeh automatically replication topology (connection objects) generate aur maintain karta hai taaki koi single point of failure na ho.

> [!info] Key Concept
> **Subnets** — AD mein subnets ko specific Sites se map kiya jaata hai. Agar kisi computer ka IP address kisi bhi site se associated subnet mein nahi aata, toh woh default/primary site par chala jaata hai — isse replication aur logon delays ho sakti hain.

==**Exam Tip:** Yaad rakho — Intra-Site = Change Notification (15 sec), Inter-Site = Scheduled (default 180 min). Yeh frequently poocha jaata hai!==

---

### 2. Update Sequence Numbers (USN) & USN Rollback

> [!info] Key Concept
> **USN (Update Sequence Number)** — Har Domain Controller apne registry mein ek USN maintain karta hai. Har change iss number ko increment karta hai. DCs USNs use karte hain track karne ke liye ki unhe apne replication partners se kaunse updates already mil chuke hain.

**USN Rollback — Kaise hota hai:**

> [!danger] Common Mistake
> Kabhi bhi ek virtualized Domain Controller ko standard hypervisor snapshot se restore mat karo (jab tak DC VM Generation ID support nahi karta — Windows Server 2012+ with Hyper-V 2012 or ESXi 5.0+). Purane snapshot se restore karne par **USN Rollback** hota hai!

1. Restored DC purane database state par revert ho jaata hai ek lower USN ke saath
2. Restored DC changes karta hai, pehle se use ho chuke USN values assign karta hai
3. Partner DCs duplicate USN sequences detect karte hain lekin replicate karne se mana kar dete hain — database inconsistency recognize karte hain
4. Rolled-back DC isolated ho jaata hai — na incoming na outgoing changes replicate hoti hain

==**Exam Tip:** USN Rollback ka fix = Immediately demote + metadata cleanup + rebuild from scratch. Kabhi force replicate mat karo!==

---

### 3. Lingering Objects & Tombstone Lifetime

> [!info] Key Concept
> **Tombstone Lifetime (TLT)** — Jab AD mein koi object delete hota hai, toh woh turant erase nahi hota. Usse "Tombstone" mark kiya jaata hai aur ek set period (default **180 days**) tak rakha jaata hai taaki deletion saare DCs par replicate ho sake.

**Lingering Object — Kaise banta hai:**

1. Ek DC **Tombstone Lifetime se zyada** time tak offline rehta hai
2. Usne deletion replication miss kar diya
3. Uske database mein abhi bhi deleted objects hain
4. Woh inn purane "lingering" objects ko healthy DCs par replicate kar sakta hai — jisse deleted users ya computers mysteriously directory mein wapas aa jaate hain!

> [!danger] Common Mistake
> Agar koi DC 180 days se zyada offline raha toh usse directly replicate mat karo. Pehle demote karo, metadata cleanup karo, phir fresh promote karo!

---

### 4. Key Numbers to Remember

| 🔢 Parameter | 📊 Value |
|-------------|---------|
| Default Tombstone Lifetime | **180 days** |
| Default Inter-Site replication interval | **180 minutes** |
| Minimum Inter-Site replication interval | **15 minutes** |
| Intra-Site change notification delay | **15 seconds** |
| KCC run interval | **15 minutes** |

==**Exam Tip:** 180 days (Tombstone), 180 min (Inter-Site default), 15 min (Inter-Site minimum), 15 sec (Intra-Site notification) — inn numbers ko ratt lo!==

---

### 5. Key Event IDs

Replication errors **Directory Service** log mein Event Viewer mein logged hoti hain:

| 🆔 Event ID | ⚠️ Cause / Meaning | 🛠️ Troubleshooting Action |
|------------|-------------------|--------------------------|
| **1311** | KCC replication topology compute nahi kar paa raha (Network partition) | Physical network routing aur site link configurations check karo |
| **1864** | Replication latency warning (DC ne bahut time se replicate nahi kiya) | Target DC powered on hai aur DNS kaam kar raha hai verify karo |
| **2042** | Tombstone Lifetime exceeded (Lingering Object danger) | DC ko disconnect karo; lingering objects remove karo ya demote/rebuild karo |
| **2095** | USN Rollback detected on a virtualized DC | Immediately DC ko demote karo aur metadata cleanup karo; repair attempt mat karo |

---

## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Do Windows Server VMs as Domain Controllers ek test lab mein
> - AD replication currently active between both servers
> - Administrator access dono servers par
> - Approx. 20 minutes ka time

### Step 1: Replication Health Check Karo

```cmd
:: DC-01 par Command Prompt as Administrator kholo
:: Replication partner status check karo
repadmin /showrepl
```

> [!success] Expected Output
> ```
> DC-01 via RPC
>     DC object GUID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
>     Last attempt @ 2026-06-26 10:15:30 was successful.
> ```
> *Saari inbound replication connections "Successful" dikhni chahiye.*

### Step 2: Replication Summary Dekho

```cmd
:: Saare DCs ka quick matrix dikhata hai
repadmin /replsummary
```

> [!success] Expected Output
> ```
> Source DSA       largest delta    fails/total %%    error
> DC-01            00m:15s          0 /   5     0
> DC-02            00m:30s          0 /   5     0
> ```
> *Source aur Destination dono mein 0 fails hone chahiye.*

### Step 3: Test User Create Karo DC-01 Par

```powershell
# DC-01 par Active Directory Users and Computers kholke
# Ya PowerShell se test user banao
New-ADUser -Name "Repl Test" -SamAccountName "repltest" -Enabled $true -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force)
```

### Step 4: DC-02 Par User Search Karo

```powershell
# DC-02 par PowerShell kholke user search karo
Get-ADUser -Filter "Name -like 'Repl Test'"
```

> *Agar user turant nahi mila (replication delay ki wajah se), toh Step 5 chalo.*

### Step 5: Manual Replication Force Karo

```cmd
:: DC-02 par yeh command chalaake replication force karo
repadmin /syncall /Adpe
```

> [!success] Expected Output
> ```
> Syncing all NC's held on DC-02.
> Syncing partition: DC=company,DC=local
> CALLBACK MESSAGE: The following replication completed successfully:
> ```

### Step 6: Verify — User Ab Dikhna Chahiye

```powershell
# Dobara search karo — ab user milna chahiye
Get-ADUser -Filter "Name -like 'Repl Test'"
```

> [!tip] Pro Tip
> *Agar sync command fail ho jaaye RPC error ke saath, toh check karo ki firewall rules TCP ports **135** aur dynamic RPC range (**49152-65535**) dono DCs ke beech allow kar rahi hain ya nahi.*

---

## ⌨️ Command Cheat Sheet

### PowerShell Commands

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-ADReplicationPartnerMetadata` | DC ka replication metadata status dikhata hai | `Get-ADReplicationPartnerMetadata -Target "DC-01.company.local"` |
| `Sync-ADObject` | Specific AD object ko immediately replicate karta hai | `Sync-ADObject -Object "CN=John,OU=Users,DC=company,DC=local" -Source "DC-01" -Destination "DC-02"` |
| `Get-ADReplicationFailure` | Forest ke saare DCs ki replication failures dikhata hai | `Get-ADReplicationFailure -Target "company.local" -Scope Forest` |

### Repadmin Commands (CMD)

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `repadmin /replsummary` | Saare DCs ka replication summary matrix | `repadmin /replsummary` |
| `repadmin /showrepl` | Detailed replication partner status aur last replication time | `repadmin /showrepl DC-01` |
| `repadmin /syncall /Adpe` | Saare partitions, saare sites mein replication force karo | `repadmin /syncall DC-01 /Adpe` |
| `repadmin /removelingeringobjects` | Lingering objects detect aur remove karo | `repadmin /removelingeringobjects DC-01.company.local dc-guid CN=Configuration,DC=company,DC=local` |

> [!tip] Pro Tip
> *`/Adpe` ka matlab yaad rakho:*
> - `/A` = All partitions sync karo
> - `/d` = Servers ko Distinguished Name se identify karo
> - `/p` = Push changes
> - `/e` = Enterprise — across different AD Sites

### GUI Path

```
Active Directory Sites and Services (dssite.msc):
├── Sites
│   ├── Target Site
│   │   ├── Servers
│   │   │   ├── Server Name
│   │   │   │   └── NTDS Settings
│   │   │   │       └── Right-click connection → "Replicate Now"
```

---

## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `repadmin /syncall` fails with **"RPC Server Unavailable"** | DNS resolution broken hai DCs ke beech ya firewall RPC ports block kar raha hai | Pehle `nslookup DC-02.company.local` run karo. Firewall mein TCP 135 aur 49152-65535 allow karo |
| Password reset doosre branch mein kaam nahi karta | Inter-site replication schedule delay (default 180 min) | `repadmin /syncall /Adpe` se immediate sync force karo. Long-term: site link interval 15 min karo |
| **Event ID 2042** — Tombstone Lifetime exceeded | DC 180 days se zyada offline tha | DC ko demote karo, metadata cleanup karo, fresh rebuild karo. Directly replicate mat karo! |
| **Event ID 2095** — USN Rollback detected | DC VM ko purane snapshot se restore kiya | Immediately DC demote karo, `ntdsutil` se metadata cleanup karo, scratch se rebuild karo |
| **Event ID 1311** — KCC topology compute fail | Network partition ya site link configuration galat hai | Physical network routing check karo, site link configurations verify karo |
| Deleted users mysteriously wapas aa rahe hain | Lingering Objects — offline DC ne purane objects replicate kar diye | `repadmin /removelingeringobjects` run karo, Strict Replication Consistency enable karo |
| Login delays 15-20 minutes naye branch office mein | Warehouse/branch subnet AD Sites and Services mein add nahi kiya gaya | Subnet add karo aur correct site se map karo `dssite.msc` mein |

> [!tip] Pro Tip
> **Strict Replication Consistency** enable karo saare DCs par. Yeh setting prevent karti hai ki koi DC lingering objects wale partner se updates accept kare. Instead, healthy DC replication terminate kar dega us partner ke saath.
> ```cmd
> :: Registry path for Strict Replication Consistency
> reg add "HKLM\System\CurrentControlSet\Services\NTDS\Parameters" /v "Strict Replication Consistency" /t REG_DWORD /d 1 /f
> ```

==**Exam Tip:** Troubleshooting ka pehla step hamesha `repadmin /replsummary` hona chahiye — yeh sabse fast overview deta hai!==

---

## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Password Reset Branch Office Mein Fail

> [!example] Ticket
> "Main New York branch se apna password reset kiya. Ab Chicago office mein login karne ki koshish kar raha hoon — 'The password you entered is incorrect' aa raha hai. Web mail par toh login ho gaya."

**L1 Response:**
1. User ka authenticating DC identify karo Chicago mein — `gpresult /r` run karo client PC par (Output: `DC-CHI-01`)
2. Verify karo ki web mail kaam kar raha hai (confirms password reset successful tha)
3. User ko batao ki sync mein thoda time lagega, aur L2 ko escalate karo

**Escalation Trigger:** User 30+ minutes se wait kar raha hai aur abhi bhi login nahi ho raha

**L2 Resolution:**
1. `repadmin /replsummary` run karo — NYC aur Chicago different AD Sites mein hain, inter-site replication 180 min scheduled hai, password 5 min pehle reset hua
2. *PDC fallback fail kyun hua?* — WAN link between Chicago aur PDC Emulator (NYC) saturated tha, Chicago DC ne PDC se credential verify karte waqt timeout kar diya
3. Immediate fix: `repadmin /syncall DC-CHI-01 /Adpe` — NYC se Chicago DC par force sync
4. User se login karwao — kaam kar gaya

**Prevention:** Critical branch office site links ka inter-site replication interval 180 min se 15 min karo, ya WAN site links par "Change Notification" enable karo

**Ticket Close Note:** *"Forced replication from NYC to Chicago DC using repadmin. Verified user login successful. Recommended shortening site link replication intervals. Closed."*

---

### 🎫 Scenario 2: Event ID 2042 — Tombstone Lifetime Exceeded

> [!example] Ticket
> "Replication monitoring system alert: DC-BRANCH-05 ne replication band kar diya hai. repadmin /showrepl mein error: 'The backup latch limit has been exceeded or the tombstone lifetime has expired.'"

**L1 Response:**
1. Alert acknowledge karo aur basic details note karo
2. DC-BRANCH-05 ka physical status check karo (powered on hai ya nahi)
3. Immediately L2/L3 ko escalate karo — yeh critical issue hai

**Escalation Trigger:** Event ID 2042 = automatic L3 escalation

**L2/L3 Resolution:**
1. `repadmin /showrepl DC-BRANCH-05` — Error: `Last replication attempt was 192 days ago`
2. Forest Tombstone Lifetime = default 180 days. Server building renovation ke dauran 6+ months se powered down tha
3. **Direct replicate mat karo!** Force-demote karo offline DC ko
4. Healthy DC par Metadata Cleanup run karo — `DC-BRANCH-05` ke saare traces AD database se remove karo
5. Branch server par clean OS install, rename, aur fresh DC promote karo

**Prevention:** DC health monitor karo. Agar koi DC 60 days se zyada offline ho, toh administrators ko alert karo aur Tombstone limit hit hone se pehle demote kar do

**Ticket Close Note:** *"Forcefully demoted DC-BRANCH-05 due to Tombstone Lifetime expiry. Completed metadata cleanup on DC-01. Re-promoted branch server as a clean DC. Closed."*

---

### 🎫 Scenario 3: Naye Warehouse Mein Login Delays 15-20 Minutes

> [!example] Ticket
> "Naye warehouse mein users ko login karne mein 15-20 minutes lag rahe hain. Bahut slow hai. Koi kaam nahi ho pa raha."

**L1 Response:**
1. Confirm karo ki yeh sirf naye warehouse ke users ke saath ho raha hai
2. Client PC par `gpresult /r` run karo — dekho kaunsa DC authenticate kar raha hai
3. Agar DC headquarter wala dikh raha hai (local DC nahi), toh L2 ko escalate karo

**Escalation Trigger:** Client PCs remote/headquarter DC se authenticate ho rahe hain instead of local DC

**L2 Resolution:**
1. Network team ke saath coordinate karo — warehouse ka IP subnet check karo
2. **Active Directory Sites and Services** (`dssite.msc`) mein dekho — warehouse subnet add nahi kiya gaya!
3. Subnet add karo aur local warehouse site se map karo
4. Client requests ab local DC par route hongi — login delays eliminate

**Prevention:** Jab bhi naya office/branch/warehouse setup ho, toh pehle din hi subnet AD Sites and Services mein add karo

**Ticket Close Note:** *"Added warehouse subnet to AD Sites and Services and mapped to local site. Client authentication now routes to local DC. Login time reduced from 15-20 min to under 10 seconds. Closed."*

---

## 🎤 Interview Questions

> [!question] Q1: Password reset doosre office mein 15-30 minutes kyun lagta hai kaam karne mein?
> **Answer:** Yeh replication delay ki wajah se hota hai. Agar offices different Active Directory Sites mein hain, toh replication scheduled hota hai (usually har few hours ya minimum 15-30 minutes). Jab tak replication cycle complete nahi hota, doosre office ka local DC naya password nahi jaanta. PDC Emulator fallback mechanism hai lekin WAN issues ki wajah se woh bhi fail ho sakta hai.

> [!question] Q2: Kaunsa GUI snap-in use karte ho IP subnets aur physical replication links manage karne ke liye?
> **Answer:** **Active Directory Sites and Services** console use karta hoon (`dssite.msc`). Isme corporate subnets define kar sakte hain, unhe sites se assign kar sakte hain, aur inter-site replication links aur schedules configure kar sakte hain.

> [!question] Q3: Command line se replication health kaise verify aur troubleshoot karte ho?
> **Answer:** `repadmin` utility use karta hoon. Pehle `repadmin /replsummary` run karta hoon high-level view ke liye aur failing DCs identify karne ke liye. Phir failing DC par `repadmin /showrepl` run karta hoon exact error codes dekhne ke liye — jaise DNS lookup errors ya access denied.

==**Exam Tip:** Interview mein hamesha bolo — "Mera pehla step hota hai `repadmin /replsummary` run karna to isolate failing partners, phir Directory Service event logs check karna."==

> [!question] Q4: "Lingering Object" kya hota hai Active Directory mein?
> **Answer:** Lingering object ek deleted object hai jo ek Domain Controller par rehta hai kyunki woh DC Tombstone Lifetime (usually 180 days) se zyada time tak offline tha. Offline period mein DC ne deletion notification miss kar diya. Jab woh reconnect hota hai, uske paas abhi bhi woh object hai — jo healthy DCs par replicate hokar deleted items ko wapas la sakta hai.

> [!question] Q5: USN Rollback kya hai, kaise hota hai, aur kaise resolve karte ho?
> **Answer:**
> - **Situation**: Ek virtualized DC ko improperly snapshot se restore kiya gaya
> - **Task**: USN Rollback detect karna aur AD database recover karna
> - **Action**: Jab administrator DC VM ko purane snapshot se restore karta hai unsupported hypervisor par, DC ka database purani state mein aa jaata hai. Partner DCs believe karte hain ki unhe already updates mil chuke hain, toh woh replication block kar dete hain. Event ID **2095** generate hota hai
> - **Result**: Rolled-back DC ko forcefully demote karna padta hai, metadata cleanup karni padti hai `ntdsutil` ya ADUC se, aur scratch se rebuild karna padta hai. ==Kabhi bhi rolled-back DC ko force replicate mat karo!==

> [!question] Q6: Do Domain Controllers ke beech replication force karne ke liye kaunsa tool aur syntax use karte ho?
> **Answer:** `repadmin.exe` use karta hoon. Poore domain mein replication force karne ke liye `repadmin /syncall /Adpe` run karta hoon. `/A` saare partitions sync karta hai, `/d` servers ko distinguished name se identify karta hai, `/p` changes push karta hai, aur `/e` different AD sites ke across sync karta hai.

> [!question] Q7: Describe a situation where you collaborated with another team to solve a complex identity problem.
> **Answer:** Humare naye remote warehouse mein users ko 20 minutes tak login delays aa rahe the. Maine Network Infrastructure team ke saath coordinate kiya aur IP routing check kiya. Pata chala warehouse subnet AD Sites and Services mein add nahi kiya gaya tha. Humne subnet add kiya aur local warehouse site se bind kiya. Isse client requests local DC par route hone lagi aur login delays eliminate ho gaye.

==**Exam Tip:** 3 cheezein interviewer sunna chahta hai: (1) `repadmin` commands se force aur verify replication, (2) DC VM snapshots restore karne ka danger (USN Rollbacks), (3) AD Sites, Subnets, aur Site Links ka concept.==

---

## 🔗 Related Notes

- [[03-Identity-and-Core-Services/06-Active-Directory/WS-08 FSMO Roles|WS-08 FSMO Roles]] — Single-master roles ka detail, including PDC Emulator time authority
- [[01-Foundations/02-Networking/DNS|DNS]] — Resolution system jo DCs ko ek doosre ko find karne deta hai
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-05 Group Policy — Complete Guide|WS-05 Group Policy — Complete Guide]] — GPO replication via DFS-R alongside AD replication

---
