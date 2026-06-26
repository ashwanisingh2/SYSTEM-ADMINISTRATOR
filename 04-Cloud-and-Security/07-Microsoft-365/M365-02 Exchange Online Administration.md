---
tags: [desktop-support, m365, collaboration, L1]
aliases: [m365-02-exchange-online-administration, m365-02]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#none`

# M365-02: Exchange Online Administration

> [!abstract] Overview
> Yeh note Exchange Online Administration pe focus karta hai. Isme Mailbox types (User, Shared, Resource), Delegate permissions, Mail Flow rules, aur Message Tracing cover kiye gaye hain.

---
## 🧠 Concept Overview

- **What it is** — Microsoft ka cloud-based enterprise email aur calendaring system.
- **Why it matters** — Email corporate communication ki backbone hai. Agar mail routing ya spam filtering fail hui, toh business impact hoga.
- **Where you see this** — Jab kisi user ko kisi dusri id (jaise info@) se email bhejna ho, ya jab kisi important email ko trace karna ho ki deliver hua ya nahi.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Shared mailboxes pe access (FullAccess/SendAs) dena, aliases add karna. |
| **L2** | Message Trace run karna troubleshoot karne ke liye, aur Mail Flow/Transport rules banana. |
| **L3** | Advanced EOP (Exchange Online Protection) configure karna, Litigation holds aur eDiscovery manage karna. |

> [!tip] Seedha Simple Mein
> *Exchange Online aapke organization ka global post office hai. Alag-alag tarah ke mailboxes (user, shared, room) aur dynamic routing rules (transport rules) ke sath aap email flow control karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Exchange Online** is like **the physical Postal Service** because...
>
> - **User Mailbox**: Aapka personal ghar ka letterbox.
> - **Shared Mailbox**: Ek common office desk jahan kai log letters padh aur bhej sakte hain.
> - **Resource Mailbox**: Conference room ke bahar tanga hua sign-up sheet.
> - **Transport Rules**: Post office ki sorting machine jo address/weight ke hisaab se letters alag karti hai.

---
## 🔬 Technical Deep Dive

### 1. Mailbox Types & Permissions

> [!info] Key Concept
> Har mailbox ka alag purpose hota hai. Shared Mailbox pe login ID nahi hoti, usme delegates ko permissions di jati hain.

- **Full Access**: User doosre ka mailbox read/write kar sakta hai (Send As automatically nahi milta).
- **Send As**: Delegate aise email bhejta hai ki receiver ko lagega original mailbox se aaya hai.
- **Send on Behalf**: Receiver ko dikhega "Delegate on behalf of Mailbox".

> [!danger] Common Mistake
> Directing users to log into shared mailboxes using a dedicated password. Shared mailboxes ka password disable rehna chahiye security (MFA) ki wajah se. Sirf delegation (FullAccess) use karein.

### 2. Mail Flow & Tracing

- **Transport Rules**: Conditions ke basis pe emails ko block, modify ya redirect karta hai (e.g. prepending `[EXTERNAL]` to subject).
- **Message Trace**: Email kidhar atak gaya, spam me gaya ya drop hua, ye track karne ke liye use hota hai.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Exchange Admin Center access
> - Exchange Online PowerShell Module

### Step 1: Create a Shared Mailbox and Delegate Permissions via PowerShell

```powershell
# Authenticate
Install-Module -Name ExchangeOnlineManagement -Force
Connect-ExchangeOnline -UserPrincipalName admin@yourtenant.com

# Create Shared Mailbox & Assign Permissions
New-Mailbox -Name "Customer Support" -Alias "support" -Shared
Add-MailboxPermission -Identity "support" -User "jdoe@yourtenant.com" -AccessRights FullAccess -InheritanceType All
Add-RecipientPermission -Identity "support" -Trustee "jdoe@yourtenant.com" -AccessRights SendAs -Confirm:$false
```

> [!success] Expected Output
> ```
> Mailbox created. Permissions applied successfully. (Note: It may take up to 60 mins to sync to Outlook clients).
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-MessageTrace` | Pichle 10 dino ke email delivery status track karta hai | `Get-MessageTrace -RecipientAddress "support@domain.com"` |
| `Add-MailboxPermission` | Kisi mailbox par (jaise FullAccess) delegate permission lagata hai | `Add-MailboxPermission -Identity "sales" -User "tom" -AccessRights FullAccess` |
| `Set-Mailbox` | Mailbox ke settings (forwarding/hold) change karta hai | `Set-Mailbox -Identity "tom" -DeliverToMailboxAndForward $true -ForwardingAddress "other@domain.com"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| User has Send As permission but gets bounce-back when sending from Shared Mailbox | Outlook OAB (Offline Address Book) cache purana hai | Outlook me `Send/Receive` -> `Download Address Book` karein. Or manually type address in From field. |
| External vendor emails failing with 550 5.7.1 Service unavailable | Vendor ka SPF/DKIM fail ho raha hai, EOP unhe spam maan raha hai | Message Trace run karein, vendor ko SPF fix karne bolen, temporary EOP Tenant Allow/Block list me IP daalein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: External Client Did Not Receive Contract

> [!example] Ticket
> "I sent a critical PDF to a client an hour ago, and they say they haven't received it!"

**L1 Response:** Message Trace run karke sender aur recipient detail daalna aur result check karna.
**Escalation Trigger:** Agar Message Trace me status "Failed" ya "Quarantined" dikhaye aur policy rule complex ho.
**L2 Resolution:** Trace me "Delivered" dikha raha hai with a '250 OK' response code. Ticket me log attach karna proving Microsoft handoff was successful, and advise client to check their own spam filter.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Send As and Send on Behalf permissions?
> **Answer:** 'Send As' mein receiver ko lagta hai ki mail directly us mailbox owner se aaya hai (e.g. from Info). 'Send on Behalf' mein receiver ko pata chal jata hai ki kisne bheja hai (e.g. John on behalf of Info).

==**Exam Tip:** 'FullAccess' permission auto-mapping ($true) ke sath aati hai, jisse Outlook restart hote hi mailbox automatically mount ho jata hai. Performance issues rokne ke liye isko disable ($false) kiya ja sakta hai.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Configures global tenants.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-05 Security and Compliance|M365-05 Security and Compliance]] — Integrates DLP and mail flow restrictions.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-02 PowerShell for Active Directory|PS-02 PowerShell for Active Directory]] — Handles AD attributes.
