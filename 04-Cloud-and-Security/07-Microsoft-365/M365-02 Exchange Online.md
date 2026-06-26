---
tags: [microsoft-365, exchange-online, email, email-routing]
aliases: [Exchange, EXO]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#intermediate` `#none`

# M365-02: Exchange Online

> [!abstract] Overview
> Exchange Online is the enterprise-grade, cloud-based email, calendaring, and contacts solution provided by Microsoft 365. Ek system administrator ya IT support engineer ke liye iska in-depth knowledge hona bahut zaroori hai kyunki maximum corporate communication isi par chalta hai. Agar email down hai, toh poori company ka kaam ruk jata hai. Yeh note aapko mail flow, user mailboxes, spam filtering, aur email routing concepts sikhayega.

---
## 🧠 Concept Overview

- **What it is** — Exchange Online is a hosted messaging solution that delivers the capabilities of Microsoft Exchange Server as a cloud-based service.
- **Why it matters** — Email is critical for business operations. Managing mailboxes, distribution lists, shared mailboxes, and ensuring security (anti-spam/anti-phishing) are core daily tasks.
- **Where you see this** — Password resets, email not receiving issues, creating new user mailboxes, managing calendar sharing, and setting up out-of-office replies.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Password reset, adding users to distribution lists, checking if an email was delivered using message trace, helping users configure Outlook on their mobile or desktop. |
| **L2** | Configure, fix, escalate kab karta hai — Creating shared mailboxes, setting up forwarding rules, troubleshooting complex mail flow issues, managing Exchange admin center (EAC) policies. |
| **L3** | Architecture, design, enterprise-level — Hybrid Exchange setup, large-scale email migrations (e.g., from Google Workspace to M365), configuring DKIM/DMARC/SPF records, creating transport rules for compliance. |

> [!tip] Seedha Simple Mein
> *Exchange Online ek digital post office ki tarah hai. Jab bhi koi employee email bhejta hai, yeh post office us email ko check karta hai (spam toh nahi), aur sahi destination tak pahunchata hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Exchange Online** is like **a high-tech corporate Post Office** because...
>
> - [Mailboxes] Every user has their own personal P.O. Box (Mailbox) with a specific size limit.
> - [Shared Mailboxes] Sometimes multiple people share one P.O. Box for a department, like "sales@company.com".
> - [Mail Flow/Transport Rules] The post office has strict rules on what packages (emails) are allowed in and out, screening for dangerous items (malware/spam).
> - [Message Trace] If a package is lost, you can use a tracking number (Message Trace) to see exactly where it is stuck in the delivery process.

---
## 🔬 Technical Deep Dive

### 1. Types of Mailboxes

> [!info] Key Concept
> Exchange Online offers different mailbox types tailored for specific organizational needs.

- **User Mailbox**: A personal mailbox assigned to an individual user. Requires a paid M365 license. *Yeh regular employee ka email account hota hai.*
- **Shared Mailbox**: A mailbox used by multiple people (e.g., info@company.com). It does not require a license as long as it stays under 50GB, and users access it using their own credentials. *Yeh department level email ke liye best hai, free hota hai agar size limit cross na kare.*
- **Resource Mailbox**: Used for company resources like meeting rooms or equipment. Users can book them through Outlook calendars. *Yeh meeting rooms (Room mailbox) ya projectors (Equipment mailbox) book karne ke liye use hota hai.*

> [!danger] Common Mistake
> Assigning a paid license to a shared mailbox when it's under 50GB. *Company ka paisa waste mat karo, shared mailbox bina license ke free mein chalta hai!*

### 2. Groups in Exchange

- **Distribution Groups**: Used to send emails to a group of people. *Jaise "All Employees" ko ek sath mail bhejna ho.*
- **Security Groups**: Can be used for email distribution, but primarily used to grant access to resources like SharePoint sites.
- **Microsoft 365 Groups**: The modern group that provides a shared inbox, calendar, document library, and Teams integration.

### 3. Mail Flow and DNS Records

To make Exchange Online work with a custom domain (e.g., company.com), DNS records are required:
- **MX Record (Mail Exchanger)**: Points to Exchange Online to receive emails. *Yeh record batata hai ki is domain ki incoming emails kahan jaani chahiye.*
- **SPF (Sender Policy Framework)**: A TXT record that lists IP addresses authorized to send email on behalf of your domain. *Yeh prevent karta hai ki koi doosra aapke naam se fake email bhej sake.*
- **DKIM (DomainKeys Identified Mail)**: Adds a digital signature to outgoing emails to verify authenticity.
- **DMARC (Domain-based Message Authentication, Reporting, and Conformance)**: Works with SPF and DKIM to tell receiving servers what to do if an email fails authentication.

> [!warning] Pre-requisites / Caution
> Without proper SPF and DKIM records, your company's outgoing emails will likely end up in the recipient's spam/junk folder.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Global Administrator or Exchange Administrator role.
> - Access to the Microsoft 365 Admin Center (`admin.microsoft.com`) or Exchange Admin Center (`admin.exchange.microsoft.com`).
> - PowerShell with `ExchangeOnlineManagement` module installed.

### Step 1: Connect to Exchange Online via PowerShell

Connecting via PowerShell is often faster than using the GUI for bulk operations.

```powershell
# Yeh command Exchange Online management module install karta hai (one-time only)
Install-Module -Name ExchangeOnlineManagement -Force

# Yeh command apko portal mein login karwayega
Connect-ExchangeOnline -UserPrincipalName admin@yourdomain.com
```

> [!success] Expected Output
> ```
> An interactive login window will appear. After entering your password and completing MFA, you will be connected and returned to the PowerShell prompt with no red error text.
> ```

### Step 2: Convert a User Mailbox to a Shared Mailbox

Often, when an employee leaves, you convert their mailbox to shared so the manager can access old emails without paying for a license.

```powershell
# Yeh command user mailbox ko shared mailbox mein convert kar deta hai
Set-Mailbox -Identity "john.doe@company.com" -Type Shared

# Manager ko us shared mailbox ka full access dena
Add-MailboxPermission -Identity "john.doe@company.com" -User "manager@company.com" -AccessRights FullAccess -InheritanceType All
```

### Step 3: Run a Message Trace

Agar user complaint kare ki usko email nahi aayi, toh hum Message Trace chalate hain.

```powershell
# Last 48 hours mein is user ko aayi hui emails ka status check karna
Get-MessageTrace -RecipientAddress "user@company.com" -StartDate (Get-Date).AddDays(-2) -EndDate (Get-Date)
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Connect-ExchangeOnline` | Exchange Online se connect karta hai | `Connect-ExchangeOnline -UserPrincipalName admin@domain.com` |
| `Get-Mailbox` | Saare mailboxes ki list dikhata hai | `Get-Mailbox -ResultSize Unlimited` |
| `Set-Mailbox` | Mailbox ki properties change karta hai | `Set-Mailbox -Identity "user" -Type Shared` |
| `Add-MailboxPermission` | Kisi aur user ko mailbox ka access deta hai | `Add-MailboxPermission -Identity "shared" -User "admin" -AccessRights FullAccess` |
| `Get-MessageTrace` | Email ka delivery status track karta hai | `Get-MessageTrace -SenderAddress "a@b.com" -RecipientAddress "c@d.com"` |
| `Set-MailboxAutoReplyConfiguration` | Kisi user ka Out of Office (OOF) message set karta hai | `Set-MailboxAutoReplyConfiguration -Identity "user" -AutoReplyState Enabled -InternalMessage "On leave"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| User is not receiving any external emails. | MX Record is missing or incorrect. | Verify the MX record in DNS points to `company-com.mail.protection.outlook.com`. *DNS settings check karo ki traffic sahi jagah route ho raha hai.* |
| Outgoing emails are going to recipients' spam folders. | SPF or DKIM is not configured or is failing. | Add or update the SPF TXT record: `v=spf1 include:spf.protection.outlook.com -all`. Configure DKIM in Microsoft 365 Defender. |
| User cannot send email, gets NDR (Non-Delivery Report). | User might be blocked for sending spam (compromised account). | Go to Microsoft 365 Defender -> Review -> Restricted users, and unblock the account after resetting their password. *User ka account compromise ho gaya hoga, isliye Microsoft ne block kar diya.* |
| Changes in EAC (like adding a user to a group) are not showing up in Outlook. | Offline Address Book (OAB) sync delay. | Ask the user to manually download the address book in Outlook (`Send/Receive -> Send/Receive Groups -> Download Address Book`), or wait 24 hours. |
| Shared Mailbox is not appearing in Outlook. | Auto-mapping issue or sync delay. | Restart Outlook. If it still doesn't appear, manually add it via Account Settings or check if `AutoMapping` was set to `$false` during permission grant. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Employee Left the Company

> [!example] Ticket
> "Sarah has left the company. Please block her sign-in, free up her license, and give her manager (David) access to her emails."

**L1 Response:** Reset Sarah's password, block sign-in in the Microsoft 365 admin center, and log out of all sessions.
**Escalation Trigger:** No escalation usually needed, but if the mailbox is on legal hold, might consult L2/Compliance team.
**L2 Resolution:**
1. Convert Sarah's mailbox to a Shared Mailbox.
2. Remove her Microsoft 365 license (since shared mailboxes under 50GB don't need one).
3. Grant David `FullAccess` and `SendAs` permissions to the shared mailbox. *License bachane ka sabse best tareeka.*

### 🎫 Scenario 2: Missing Important Email from Vendor

> [!example] Ticket
> "I was expecting a critical contract from our vendor (vendor@external.com), but I haven't received it yet."

**L1 Response:** Check the user's Junk folder and Outlook rules. Run a basic Message Trace in Exchange Admin Center for the past 24-48 hours with the sender's address.
**Escalation Trigger:** The message trace shows the email was quarantined due to anti-spam policies or advanced threat protection.
**L2 Resolution:** Go to Microsoft 365 Defender portal, review the Quarantine section. If it's a false positive, release the email to the user and consider adding the vendor's domain to the allow-list if it happens frequently. *Hamesha check karo ki spam filter ne email block toh nahi ki.*

### 🎫 Scenario 3: Bulk Email Sending Issues

> [!example] Ticket
> "Our marketing team is trying to send a newsletter to 1,000 clients, but they are getting bounce backs saying 'Message rejected'."

**L1 Response:** Verify the exact error message in the NDR. Discover that the user has hit the Exchange Online sending limits (max 10,000 recipients per day, 30 messages per minute).
**Escalation Trigger:** The marketing team demands a workaround because they need to send this today.
**L2 Resolution:** Explain to the marketing team that Exchange Online is not designed for bulk marketing emails. Recommend and assist in setting up a third-party service like Mailchimp or SendGrid, and configure the necessary SPF/DKIM records to allow that service to send on the company's behalf. *Exchange marketing ke liye nahi bana hai, limits cross karne par tenant block ho sakta hai.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between a User Mailbox and a Shared Mailbox?
> **Answer:** A user mailbox requires a paid M365 license and has its own username/password. A shared mailbox is free (up to 50GB), does not have its own credentials (users log in with their own accounts and access it), and is used for generic addresses like `info@company.com`.
> ==**Exam Tip:** Converting a user mailbox to a shared mailbox is the standard practice for retaining data of ex-employees without consuming a paid license.==

> [!question] Q2: How do you troubleshoot an issue where external emails are not coming in, but internal emails are working fine?
> **Answer:** I would first check the domain's MX records using tools like `mxtoolbox.com` to ensure it correctly points to Microsoft 365 (`company-com.mail.protection.outlook.com`). Then, I would run a Message Trace in the Exchange Admin Center to see if the emails are hitting our tenant but getting dropped or quarantined.
> ==**Exam Tip:** Internal emails do not rely on public MX records; they are routed internally by Exchange. MX records are strictly for incoming external mail.==

> [!question] Q3: What are SPF, DKIM, and DMARC? Why do we need them?
> **Answer:** They are email authentication protocols used to prevent spoofing and ensure deliverability.
> - **SPF**: Lists authorized IP addresses that can send on behalf of the domain.
> - **DKIM**: Digitally signs outgoing emails.
> - **DMARC**: Uses SPF and DKIM to tell the receiving server what to do with emails that fail authentication (e.g., reject or quarantine).
> ==**Exam Tip:** Always configure SPF before configuring DMARC, otherwise legitimate emails might get rejected.==

> [!question] Q4: A user's account is compromised and they sent out 5,000 spam emails. Microsoft blocked them. How do you recover the account?
> **Answer:** First, reset the user's password and enforce MFA. Then, check for any malicious inbox forwarding rules. Finally, go to the Microsoft 365 Defender portal -> Email & Collaboration -> Review -> Restricted Users, and unblock the account.
> ==**Exam Tip:** Never unblock a restricted user without first securing the account, otherwise they will just get blocked again.==

---
## 🔗 Related Notes

- [[M365-01 Office 365 Administration|M365-01 Office 365 Administration]] — Basics of user and license management.
- [[Security-03 Email Security and Defender|Security-03 Email Security and Defender]] — Deep dive into Anti-Phishing, Safe Links, and Safe Attachments.
- [[Networking-04 DNS Deep Dive|Networking-04 DNS Deep Dive]] — Detailed explanation of MX, TXT, and CNAME records.
