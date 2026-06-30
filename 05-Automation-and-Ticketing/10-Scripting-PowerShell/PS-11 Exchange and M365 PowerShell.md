---
tags: [powershell, m365, exchange-online, automation]
aliases: [PS-11, Exchange PowerShell, M365 PS]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-purple]
> 📜 **POWERSHELL**

`#complete` `#advanced` `#none`

# PS-11: Exchange and M365 PowerShell

> [!abstract] Overview
> Yeh note Microsoft 365 (M365) aur Exchange Online ko PowerShell ke through manage karne par focus karta hai. Ek System Administrator ke liye GUI (Admin Center) se zyada fast aur efficient tareeka PowerShell use karna hai, especially jab bulk operations (jaise 100 users ke mailboxes create karna ya permissions assign karna) karne hon. Is note mein M365 modules, connection establishment, aur day-to-day administrative tasks ko automate karne ki techniques detail mein cover ki gayi hain.

---
## 🧠 Concept Overview

- **What it is** — It’s a set of PowerShell modules (like ExchangeOnlineManagement, MSOnline, AzureAD, or Microsoft.Graph) used to interact directly with Office 365 services via command-line.
- **Why it matters** — Jab aapke paas large enterprise environment hota hai, toh har ek user ki setting web portal se change karna impossible aur time-consuming hota hai. PowerShell aapko script ke through mass-changes karne ki power deta hai. *Matlab ghanto ka kaam minutes mein ho jata hai.*
- **Where you see this** — Mailbox creation, setting forwarding rules, assigning M365 licenses, tracking email delivery logs (Message Trace), aur bulk user creation mein iska roz use hota hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Password reset karna, user ka mailbox size check karna, licenses verify karna. |
| **L2** | Configure, fix, escalate — Shared mailboxes create karna, permissions (Send As, Full Access) modify karna, Message Trace run karke email flow issues debug karna. |
| **L3** | Architecture, design — Complex automation scripts likhna, Graph API ke sath integration karna, enterprise-level security aur retention policies deploy karna. |

> [!tip] Seedha Simple Mein
> *M365 Admin Center ek normal gaadi hai jo aapko aaraam se le jaati hai, par PowerShell M365 ka jet engine hai. Ek command dalo aur hazaro users ka data ek second mein update ho jata hai. Galti ki toh sabka data bhi ud sakta hai, isliye commands carefully chalani padti hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Exchange Online PowerShell** is like **a bulk remote control for a massive apartment building (your company)** because...
>
> - Portal (GUI) se aap har ek flat (user) ki door-bell baja kar andar jate ho, jo bahut slow hai.
> - PowerShell se aap building ke master control room mein baithe ho, aur ek switch dabate hi saare flats ki lights (settings) ek sath on ya off kar sakte ho.

---
## 🔬 Technical Deep Dive

### 1. The Core Modules

> [!info] Key Concept
> M365 management has evolved. We moved from MSOnline (v1) and AzureAD to **ExchangeOnlineManagement (EXO v3)** and **Microsoft Graph PowerShell**.

**ExchangeOnlineManagement Module:**
The V3 module is REST API based, which makes it much faster and more reliable than older remote PowerShell sessions. It uses Modern Authentication (OAuth 2.0).

- **Connecting:** `Connect-ExchangeOnline -UserPrincipalName admin@domain.com`
- **Disconnecting:** `Disconnect-ExchangeOnline`

> [!danger] Common Mistake
> Session ko close na karna. Hamesha `Disconnect-ExchangeOnline` run karein apna kaam khatam hone ke baad, warna active sessions ki limit cross ho sakti hai aur naye connections block ho jayenge. *Bina session disconnect kiye script chhod dena error ka sabse bada reason hai.*

### 2. Managing Mailboxes

Mailbox management is the bread and butter of Exchange administration. Using `Get-Mailbox` and `Set-Mailbox`, you can control almost every aspect of a user's mailbox.

- `Get-Mailbox -Identity "John Doe"`: Returns mailbox properties.
- `Set-Mailbox -Identity "John Doe" -DeliverToMailboxAndForward $true -ForwardingSmtpAddress "external@domain.com"`: Sets up a forwarding rule.

### 3. Mailbox Permissions

Delegating access is frequently requested via helpdesk tickets.
- **Full Access:** Allows the delegate to open the mailbox and act as the owner.
- **Send As:** Allows the delegate to send emails that appear to come directly from the mailbox.
- **Send on Behalf:** Similar to Send As, but clearly shows "Delegate on behalf of Owner".

```powershell
Add-MailboxPermission -Identity "Sales" -User "JohnDoe" -AccessRights FullAccess -InheritanceType All
```

### 4. Message Trace

When users complain "I didn't get the email", Message Trace is your ultimate tool.

```powershell
Get-MessageTrace -SenderAddress "sender@domain.com" -RecipientAddress "user@domain.com" -StartDate (Get-Date).AddDays(-2) -EndDate (Get-Date)
```
*Yeh command pichle 2 din mein is sender aur recipient ke beech ki saari emails ka status dikhayegi.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Administrator access (Global Admin or Exchange Admin role) in an M365 Tenant.
> - Windows 10/11 with PowerShell 5.1 or PowerShell 7.
> - Internet connectivity.

### Step 1: Install and Import the Module

```powershell
# Yeh command Exchange Online Management module install karega
Install-Module -Name ExchangeOnlineManagement -Force -AllowClobber

# Module ko session mein import karna
Import-Module ExchangeOnlineManagement
```

> [!success] Expected Output
> ```
> (No output if successful. If it asks for untrusted repository, press Y)
> ```

### Step 2: Connect to Exchange Online

```powershell
# Tenant connect karne ki command. Yeh aapse login prompt poochega.
Connect-ExchangeOnline -UserPrincipalName admin@yourdomain.onmicrosoft.com
```

> [!success] Expected Output
> ```
> ----------------------------------------------------------------------------------------
> This V3 EXO PowerShell module contains new REST API backed Exchange Online cmdlets...
> ----------------------------------------------------------------------------------------
> ```

### Step 3: Create a Shared Mailbox and Assign Permissions

```powershell
# Naya shared mailbox create karna
New-Mailbox -Shared -Name "IT Support" -DisplayName "IT Support Desk" -Alias itsupport

# John Doe ko Full Access dena without automapping
Add-MailboxPermission -Identity "itsupport" -User "JohnDoe" -AccessRights FullAccess -AutoMapping $false

# John Doe ko Send As permission dena
Add-RecipientPermission -Identity "itsupport" -Trustee "JohnDoe" -AccessRights SendAs -Confirm:$false
```

> [!success] Expected Output
> ```
> Identity             User                 AccessRights                                                IsInherited Deny
> --------             ----                 ------------                                                ----------- ----
> itsupport            domain\JohnDoe       {FullAccess}                                                False       False
> ```

### Step 4: Disconnect the Session

```powershell
# Session ko gracefully close karna
Disconnect-ExchangeOnline -Confirm:$false
```

> [!success] Expected Output
> ```
> (Session disconnected successfully. No output returned.)
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Connect-ExchangeOnline` | Connects PowerShell to your M365 tenant. | `Connect-ExchangeOnline` |
| `Get-Mailbox` | Retrieves mailbox details (size, type, alias). | `Get-Mailbox -ResultSize Unlimited` |
| `Set-Mailbox` | Modifies mailbox settings (like forwarding, quotas). | `Set-Mailbox "user" -HiddenFromAddressListsEnabled $true` |
| `New-Mailbox` | Creates a new user, shared, or room mailbox. | `New-Mailbox -Shared -Name "HR"` |
| `Add-MailboxPermission` | Grants Full Access to a mailbox. | `Add-MailboxPermission -Identity "HR" -User "Admin" -AccessRights FullAccess` |
| `Add-RecipientPermission` | Grants Send As permission. | `Add-RecipientPermission -Identity "HR" -Trustee "Admin" -AccessRights SendAs` |
| `Get-MessageTrace` | Tracks email delivery status (Delivered, Failed, Quarantined). | `Get-MessageTrace -SenderAddress "a@b.com"` |
| `Disconnect-ExchangeOnline`| Closes the session securely. | `Disconnect-ExchangeOnline -Confirm:$false` |
| `Set-MailboxAutoReplyConfiguration` | Configures Out of Office (OOF) messages for a user. | `Set-MailboxAutoReplyConfiguration -Identity "user" -AutoReplyState Enabled -InternalMessage "On leave"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Cannot load module `ExchangeOnlineManagement` | Execution Policy is set to Restricted or module is not installed properly. | Run `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`, then `Install-Module ExchangeOnlineManagement -Force`. |
| Login prompt loop or MFA failure | Legacy authentication blocked or Conditional Access blocking PowerShell. | Ensure you are using the V3 module and Modern Auth. *Check karo ki CA policy script login block toh nahi kar rahi.* |
| `Access Denied` when running `New-Mailbox` | Account does not have Exchange Admin or Global Admin roles. | Assign the correct admin role in the M365 Admin Center and wait 15 minutes for replication. |
| `Get-MessageTrace` returns no data | Email is older than 10 days, standard trace only holds 10 days of quick data. | Use `Start-HistoricalSearch` for emails older than 10 days. |
| "The operation couldn't be performed because object 'X' couldn't be found" | M365 replication delay or incorrect UPN/Alias. | Wait 5-10 minutes for M365 backend sync after creating a user, then try again. Check spelling. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Bulk Mailbox Hide from GAL

> [!example] Ticket
> "We have terminated 50 contractors today. Please hide their mailboxes from the Global Address List (GAL) immediately."

**L1 Response:** Manually go to Exchange Admin Center and hide them one by one. (Inefficient)
**Escalation Trigger:** The number of users is too large for manual processing within SLA.
**L2 Resolution:** Save the 50 UPNs in a CSV file and use a PowerShell script.
```powershell
$users = Import-Csv "C:\temp\terminated.csv"
foreach ($user in $users) {
    Set-Mailbox -Identity $user.UPN -HiddenFromAddressListsEnabled $true
    Write-Host "Hidden $($user.UPN) from GAL" -ForegroundColor Green
}
```
*CSV file import karke loop chalaya aur sabko ek sath hide kar diya.*

### 🎫 Scenario 2: Trace a Missing Email for CEO

> [!example] Ticket
> "The CEO says he didn't receive an important contract from a vendor (vendor@external.com). He needs to know if our spam filter blocked it."

**L1 Response:** Ask CEO to check the Junk folder.
**Escalation Trigger:** User confirms it is not in Junk. Needs server-level verification.
**L2 Resolution:** Run a Message Trace to find the exact fate of the email.
```powershell
Get-MessageTrace -RecipientAddress "ceo@ourcompany.com" -SenderAddress "vendor@external.com" | Format-List
```
Check the `Status` property. If it says `FilteredAsSpam` or `Quarantined`, release it from M365 Defender portal. If `Delivered`, the CEO might have moved it to a folder. *Trace chala ke prove karo email kidhar ruka hai.*

### 🎫 Scenario 3: Shared Mailbox Permission Removal

> [!example] Ticket
> "Please remove Alex's access from the Finance shared mailbox as he has moved to the IT department."

**L1 Response:** Check the current permissions on the Finance mailbox.
**Escalation Trigger:** If L1 doesn't have the appropriate rights to modify Exchange permissions.
**L2 Resolution:** Remove both Full Access and Send As permissions via PowerShell.
```powershell
Remove-MailboxPermission -Identity "Finance" -User "Alex" -AccessRights FullAccess -Confirm:$false
Remove-RecipientPermission -Identity "Finance" -Trustee "Alex" -AccessRights SendAs -Confirm:$false
```
*Dono permissions (Full Access aur Send As) remove karni padti hain separately, log aksar sirf Full Access hata ke ticket close kar dete hain.*

---
## 🎤 Interview Questions

> [!question] Q1: How do you connect to Exchange Online using PowerShell with MFA enabled?
> **Answer:** You use the `Connect-ExchangeOnline` cmdlet from the ExchangeOnlineManagement (EXO V2/V3) module. It supports Modern Authentication (OAuth) out of the box and will automatically prompt for MFA through a browser-based popup.

> [!question] Q2: What is the difference between `Add-MailboxPermission` and `Add-RecipientPermission`?
> **Answer:** `Add-MailboxPermission` is used to grant mailbox-level access (like Full Access or Read Permission), allowing someone to open the mailbox. `Add-RecipientPermission` is specifically used to grant 'Send As' permission, which allows someone to send emails that look like they came directly from that mailbox.

> [!question] Q3: How do you find out the size of all mailboxes in a tenant and export it to a CSV?
> **Answer:** You use `Get-Mailbox` piped into `Get-MailboxStatistics`.
> ```powershell
> Get-Mailbox -ResultSize Unlimited | Get-MailboxStatistics | Select-Object DisplayName, TotalItemSize, ItemCount | Export-Csv "C:\MailboxSizes.csv" -NoTypeInformation
> ```

> [!question] Q4: A user says an email they sent is delayed. How can you investigate this using PowerShell?
> **Answer:** I would use the `Get-MessageTrace` cmdlet to track the specific email by providing the `SenderAddress` and `RecipientAddress`. I can use `Get-MessageTraceDetail` on the Message ID to see the exact hops and where the delay is occurring (e.g., a specific transport rule or spam filtering).

==**Exam Tip:** Always remember to run `Disconnect-ExchangeOnline` at the end of your scripts. This is a common best practice question!==

---
## 🔗 Related Notes

- [[PS-01 Introduction to PowerShell|PS-01 Intro to PowerShell]] — Basics of cmdlets and pipelines.
- [[M365-02 Exchange Admin Center|M365-02 EAC]] — Performing these same tasks via the GUI.
- [[PS-03 PowerShell Loops and Arrays|PS-03 Loops]] — Needed to understand bulk mailbox operations.
