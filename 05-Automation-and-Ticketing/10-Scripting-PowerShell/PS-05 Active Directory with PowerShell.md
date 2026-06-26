---
tags: [powershell, activedirectory, automation, scripting]
aliases: [PS AD, AD PowerShell, Active Directory PS]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-purple]
> 📜 **POWERSHELL**

`#complete` `#intermediate` `#none`

# PS-05: Active Directory with PowerShell

> [!abstract] Overview
> Yeh note sikhata hai ki PowerShell ka use karke Active Directory (AD) ko kaise manage kiya jaye. Ek system administrator ke liye AD portal (dsa.msc) mein baar-baar click karna time-consuming hota hai. PowerShell scripts se hum bulk user creation, password resets, aur group management ko automate kar sakte hain, jo ki daily IT support mein sabse zyada use hota hai. 

---
## 🧠 Concept Overview

- **What it is** — Active Directory Module for Windows PowerShell ek command-line toolset hai jo AD DS (Active Directory Domain Services) ko manage karne ke liye cmdlets provide karta hai.
- **Why it matters** — *Jab aapko 100 naye employees ke accounts banane hon, toh GUI mein 3 ghante lagenge. PowerShell se yeh kaam sirf 2 minute mein ho jata hai.*
- **Where you see this** — New hire onboarding, bulk password expiry reporting, inactive user disablement, aur security audits mein.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Password reset karna, account unlock karna, group membership check karna. |
| **L2** | Configure, fix, escalate kab karta hai — Bulk user import scripts run karna, stale accounts cleanup karna, OUs create karna. |
| **L3** | Architecture, design, enterprise-level — Complex automation scripts likhna (e.g., HRMS to AD sync), domain controller replication issues resolve karna PowerShell ke through. |

> [!tip] Seedha Simple Mein
> *AD PowerShell tools ek remote control ki tarah hain jisse aap server par bina login kiye, command line se saare user aur computer objects ko control kar sakte ho.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Active Directory PowerShell** is like **a Factory Conveyor Belt** because...
>
> - GUI (Active Directory Users and Computers) mein aap ek-ek karke item banate ho, jaise ek artisan apne haath se ek time pe ek khilona banata hai.
> - PowerShell ek automated conveyor belt hai, jahan aap ek template (script) set kar dete ho aur wo hazaron accounts ek sath process kar deta hai.
> - Agar conveyor belt mein galti hui (wrong script), toh hazaron khilone ek sath galat ban jayenge! Isliye testing (`-WhatIf`) bohot zaroori hai.

---
## 🔬 Technical Deep Dive

### 1. The ActiveDirectory Module

> [!info] Key Concept
> RSAT (Remote Server Administration Tools) tools ke andar Active Directory module aata hai jo PowerShell mein AD cmdlets enable karta hai.

By default, normal Windows 10/11 machines mein AD module nahi hota. Ise install karna padta hai:
- **Windows 10/11:** Optional Features se "RSAT: Active Directory Domain Services and Lightweight Directory Services Tools" install karein.
- **Windows Server:** Server Manager se AD DS Tools install karein.

Module import karne ke liye:
```powershell
Import-Module ActiveDirectory
```

### 2. Cmdlet Naming Convention (Verb-Noun)

AD PowerShell cmdlets hamesha `Verb-ADNoun` format follow karte hain:
- **Verbs:** `Get` (Read), `New` (Create), `Set` (Modify), `Remove` (Delete), `Add`, `Clear`, `Enable`, `Disable`, `Unlock`.
- **Nouns:** `ADUser`, `ADComputer`, `ADGroup`, `ADGroupMember`, `ADOrganizationalUnit`.

> [!danger] Common Mistake
> *Bina `-WhatIf` parameter use kiye `Remove-ADUser` ya `Set-ADUser` bulk mein run karna bohot risky ho sakta hai. Galti se galat OU ke users delete ho sakte hain.* Hamesha pehle script ko `-WhatIf` ke sath test karein.

### 3. Filtering Data (The `-Filter` Parameter)

`Get-ADUser` mein saare users nikalna bohot resource-heavy hota hai aur Domain Controller ko crash kar sakta hai. Isliye filtering use karni chahiye:

```powershell
# Bad Practice (Pulls everything)
Get-ADUser -Filter *

# Good Practice (Only get enabled users)
Get-ADUser -Filter {Enabled -eq $true}
```

### 4. Advanced Properties Lookup
By default, `Get-ADUser` sirf kuch basic properties return karta hai (Name, SID, etc.). Agar aapko Email ya Department chahiye, toh `-Properties` switch zaroori hai.

```powershell
# Yeh EmailAddress nahi dega
Get-ADUser -Identity "ashwani.s" 

# Yeh poori details dega
Get-ADUser -Identity "ashwani.s" -Properties EmailAddress, Department, Title
```

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Ek Windows Server ya Client machine jisme RSAT installed ho.
> - Domain Admin ya Account Operator privileges.
> - PowerShell ran as Administrator.

### Step 1: Install and Import AD Module

```powershell
# Windows 10/11 par RSAT install karne ke liye
Add-WindowsCapability -Online -Name RsaT.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0

# Module Import karna
Import-Module ActiveDirectory
```

> [!success] Expected Output
> ```
> (Koi output nahi aayega, prompt wapas aa jayega, matlab module successfully load ho gaya)
> ```

### Step 2: Search for a User

```powershell
# Kisi user ki properties check karna
Get-ADUser -Identity "ashwani.singh" -Properties *
```

> [!success] Expected Output
> ```
> DistinguishedName : CN=Ashwani Singh,OU=IT,DC=domain,DC=com
> Enabled           : True
> GivenName         : Ashwani
> Name              : Ashwani Singh
> ObjectClass       : user
> SamAccountName    : ashwani.singh
> SID               : S-1-5-21-3623811015-3361044348-30300820-1013
> Surname           : Singh
> UserPrincipalName : ashwani.singh@domain.com
> ```

### Step 3: Create a New User

```powershell
# Naya account banana
New-ADUser -Name "Rahul Kumar" -SamAccountName "rahul.kumar" -UserPrincipalName "rahul.kumar@domain.com" -Path "OU=HR,DC=domain,DC=com" -Enabled $true -PasswordNeverExpires $true
```

> [!success] Expected Output
> ```
> (Koi error nahi aayega, account create ho jayega. Verify karne ke liye Get-ADUser run karein.)
> ```

### Step 4: Unlock an Account and Reset Password

```powershell
# Pehle unlock karna, fir naya password set karna
Unlock-ADAccount -Identity "rahul.kumar"
Set-ADAccountPassword -Identity "rahul.kumar" -NewPassword (ConvertTo-SecureString "NewP@ssw0rd!" -AsPlainText -Force) -Reset:$true
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Import-Module ActiveDirectory` | AD cmdlets ko PowerShell mein load karta hai. | `Import-Module ActiveDirectory` |
| `Get-ADUser` | User details fetch karta hai. | `Get-ADUser -Identity "johndoe"` |
| `Unlock-ADAccount` | Locked out AD account ko unlock karta hai. | `Unlock-ADAccount -Identity "johndoe"` |
| `Disable-ADAccount` | User account ko disable karta hai (jaise employee chhod de toh). | `Disable-ADAccount -Identity "johndoe"` |
| `Add-ADGroupMember` | Kisi user ko AD group mein add karta hai. | `Add-ADGroupMember -Identity "IT_Admins" -Members "johndoe"` |
| `Get-ADGroupMember` | Kisi group ke saare members list karta hai. | `Get-ADGroupMember -Identity "HR_Dept"` |
| `Get-ADComputer` | AD mein registered computers nikalta hai. | `Get-ADComputer -Filter * -SearchBase "OU=Laptops,DC=com"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `The term 'Get-ADUser' is not recognized` | AD PowerShell module installed ya loaded nahi hai. | `Import-Module ActiveDirectory` run karein. Agar phir bhi error aaye toh RSAT tools install karein. |
| `Access is denied` | Aapke paas required permissions nahi hain, ya PowerShell non-admin mode mein chal raha hai. | PowerShell ko "Run as Administrator" se kholiye, aur check karein ki aapka account AD groups (Account Operators/Domain Admins) mein hai. |
| `Cannot find an object with identity` | Jo user ya group aap dhoondh rahe hain wo exist nahi karta, ya spelling mistake hai. | Identity ke jagah SamAccountName use karein, ya wildcard se search karein: `Get-ADUser -Filter {Name -like "*John*"}` |
| `Server was unable to process request` | Domain Controller se connectivity issue ho sakta hai. | Check karein ki aap DC ko ping kar pa rahe hain, ya `-Server` parameter pass karein: `Get-ADUser -Identity "john" -Server "dc1.domain.com"` |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: New Employee Bulk Onboarding

> [!example] Ticket
> "Please create accounts for the 50 new graduates joining next Monday. Attached is the CSV file with their details."

**L1 Response:** *Check the CSV format to ensure it has columns like FirstName, LastName, Department, Title, etc.*
**Escalation Trigger:** *If the script fails to run or assigns wrong permissions.*
**L2 Resolution:** *L2 engineer ek PowerShell script use karega jo `Import-Csv` se file padhega aur `New-ADUser` loop mein run karke saare accounts banayega.*

```powershell
# Basic logic
$users = Import-Csv "C:\new_users.csv"
foreach ($user in $users) {
    New-ADUser -Name $user.FullName -SamAccountName $user.Username -Department $user.Department -Path "OU=Users,DC=domain,DC=com"
}
```

### 🎫 Scenario 2: Find Inactive Users (Security Audit)

> [!example] Ticket
> "Security team needs a list of all users who haven't logged in for the past 90 days so we can disable them."

**L1 Response:** *L1 shayad GUI mein jaakar manually last logon check kare, jo impossible hai 1000+ users ke liye.*
**Escalation Trigger:** *Pass to L2 for automated reporting.*
**L2 Resolution:** *PowerShell se ek single command mein ye data nikala ja sakta hai:*

```powershell
$date = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $date} -Properties LastLogonDate | Select-Object Name, LastLogonDate | Export-Csv "C:\InactiveUsers.csv" -NoTypeInformation
```

### 🎫 Scenario 3: Bulk Group Membership Update

> [!example] Ticket
> "Add all users from the Marketing department to the new 'MKT_Share_Access' security group."

**L1 Response:** *Ek-ek karke Marketing users dhoondhna aur group mein add karna.*
**Escalation Trigger:** *Agar users bohot zyada hain (e.g., 200+).*
**L2 Resolution:** *PowerShell pipeline use karke sabko ek sath add karenge.*

```powershell
Get-ADUser -Filter {Department -eq "Marketing"} -Properties Department | ForEach-Object {
    Add-ADGroupMember -Identity "MKT_Share_Access" -Members $_.SamAccountName
}
```

---
## 🎤 Interview Questions

> [!question] Q1: How do you import the Active Directory module in PowerShell?
> **Answer:** By running the `Import-Module ActiveDirectory` command. *Agar module pehle se server pe RSAT ke through installed hai toh yeh run karna padta hai.*

==**Exam Tip:** Windows Server 2012+ mein module autoload ho jata hai agar cmdlet use karo, but explicitly import karna best practice hai.==

> [!question] Q2: How can you find locked out users using PowerShell?
> **Answer:** Using the `Search-ADAccount` cmdlet. Specify the `-LockedOut` parameter. Command: `Search-ADAccount -LockedOut`.

> [!question] Q3: What is the difference between `Disable-ADAccount` and `Remove-ADUser`?
> **Answer:** `Disable-ADAccount` sirf account ko login karne se block karta hai (account AD mein rehta hai). `Remove-ADUser` account ko permanently delete kar deta hai. *Hamesha best practice hai ki ex-employees ke accounts pehle disable karein, kuch dino baad delete karein.*

> [!question] Q4: How would you export the list of all AD users to an Excel/CSV file?
> **Answer:** You can pipe the output of `Get-ADUser` to the `Export-Csv` cmdlet. For example: `Get-ADUser -Filter * | Export-Csv -Path "C:\Users.csv" -NoTypeInformation`.

==**Exam Tip:** The `-NoTypeInformation` flag is crucial to prevent unnecessary metadata at the top of the CSV file.==

> [!question] Q5: Why do we use `-WhatIf` parameter with PowerShell AD cmdlets?
> **Answer:** `-WhatIf` parameter is used to simulate a command without actually making any changes. *Yeh dikhata hai ki agar command run hui toh kya asar padega, jisse galat deletions ya updates roke ja sakte hain.*

---
## 🔗 Related Notes

- [[PS-01 Intro to PowerShell|PS-01: PowerShell Basics]] — *PowerShell ka basic syntax samajhne ke liye.*
- [[AD-01 Active Directory Basics|AD-01: Active Directory Core Concepts]] — *AD DS kya hai aur objects kaise kaam karte hain.*
- [[AD-03 Managing Users and Groups|AD-03: Users and Groups in GUI]] — *Same tasks GUI (dsa.msc) se kaise kiye jate hain.*
