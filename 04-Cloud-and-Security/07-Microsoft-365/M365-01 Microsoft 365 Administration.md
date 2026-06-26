---
tags: [microsoft-365, m365-admin, cloud, administration, office-365]
aliases: [M365 Admin, Microsoft 365 Administration, O365 Admin]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#intermediate` `#none`

# M365-01: Microsoft 365 Administration

> [!abstract] Overview
> Microsoft 365 Administration is the centralized portal (`admin.microsoft.com`) for managing users, licenses, groups, domains, and security across all Microsoft cloud services including Exchange Online, SharePoint, Teams, and OneDrive. *M365 ek cloud-based ecosystem hai aur Admin Center uska control panel hai jahan se hum poori company ka IT infrastructure manage karte hain.* Ek support engineer ke liye user lifecycle aur daily access requests ke liye ye portal master karna sabse basic aur zaroori skill hai.

---
## 🧠 Concept Overview

- **What it is** — The web-based console where IT admins configure and monitor the Microsoft 365 tenant. It acts as the central hub connecting all other admin centers (Exchange, Teams, SharePoint, Security).
- **Why it matters** — *Poori company ke emails, files, aur communication Teams ke through M365 par hi chalte hain. Ek choti si mistake kisi ka bhi access block kar sakti hai ya data leak kar sakti hai.*
- **Where you see this** — Daily operations like onboarding new employees, recovering deleted emails, assigning software licenses, resetting passwords, and configuring custom domains.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Password resets, account unlocking, assigning basic licenses, checking mail flow, adding users to groups. |
| **L2** | Creating shared mailboxes, managing Exchange routing rules, setting up MFA policies, troubleshooting DirSync issues. |
| **L3** | Tenant-to-tenant migrations, conditional access policies, hybrid AD Connect architecture, overall security posture. |

> [!tip] Seedha Simple Mein
> *Microsoft 365 Admin Center ek society ke secretary office jaisa hai. Kon society mein aayega (Users), kisko kaunsi parking milegi (Licenses), aur kaun se rules follow honge (Policies) sab yahan se decide hota hai.* Agar aapko IT Support mein career banana hai, toh is portal ko apna best friend bana lo.

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **An Airport Terminal** is like **M365 Admin Center** because...
>
> - The **Check-in Desk** is like **User Management** (Creating identities and verifying boarding passes before anyone can do anything).
> - The **Baggage Allowance** is like **Licensing** (E3, E5, F3 determining what features, apps, and storage limits a user gets).
> - **Security Screening** is like **Microsoft Entra ID / MFA** (Ensuring only authenticated and safe users enter the environment).
> - **Different Gates and Lounges** represent **Teams, SharePoint, and Exchange** (Different services accessible once you are inside the M365 ecosystem).
> - **Air Traffic Control** is like the **Global Admin** (Having visibility and control over everything happening in the environment).

---
## 🔬 Technical Deep Dive

### 1. Identity and Access (Entra ID Connection)

> [!info] Key Concept
> M365 relies entirely on Microsoft Entra ID (formerly Azure AD) for identity management. *Jo bhi user hum M365 Admin Center mein banate hain, wo actually background mein Entra ID mein save hota hai.*

There are three main types of identities in M365:
- **Cloud-Only:** Created directly in M365. Passwords managed completely in the cloud. Easy to manage for small businesses.
- **Synchronized (Hybrid):** Created in On-Premise Active Directory and synced using Azure AD Connect. *Aise accounts ko hum direct M365 se delete ya modify nahi kar sakte, changes hamesha on-prem AD mein karne padte hain.*
- **Guest Users:** External users invited to collaborate (e.g., via Teams or SharePoint).

### 2. Licensing and Subscriptions

Licenses dictate the features a user can access. M365 offers various tiers:
- **Business Basic / Standard / Premium:** Generally for SMBs (<300 users).
- **Enterprise E3 / E5:** For large organizations. Includes advanced security, compliance, and limitless archiving.
- **F-Series (F1, F3):** For frontline workers (kiosk workers, limited needs, mostly web-based access).

> [!danger] Common Mistake
> Forgetting to remove licenses from disabled/terminated users. *License ka paisa monthly lagta hai. Agar user chhod gaya hai aur license unassigned nahi kiya, toh company ka hazaro dollars ka loss ho sakta hai.* Always convert the mailbox to a Shared Mailbox and remove the license.

### 3. Groups in Microsoft 365

- **Microsoft 365 Groups:** Creates a shared workspace. It automatically provisions a Teams chat, SharePoint site, shared calendar, and Planner board.
- **Distribution Lists:** Only for sending emails to multiple people at once. No shared file storage or collaboration features.
- **Security Groups:** Used purely to control access to resources like SharePoint sites or Intune policies. No email functionality by default.
- **Mail-Enabled Security Groups:** Acts as a distribution list AND can be used to assign permissions.

### 4. Admin Roles

> [!info] Key Concept
> M365 uses Role-Based Access Control (RBAC). You only give the access needed for the job (Least Privilege).

- **Global Administrator:** Has access to everything. *Sabse powerful role, ek tenant mein sirf 2-4 trusted logo ke paas hona chahiye.*
- **Exchange Administrator:** Manages mailboxes, mail flow, anti-spam.
- **User Administrator:** Can reset passwords, manage users/groups. Cannot manage Global Admins.
- **Helpdesk Administrator:** Can force password resets and manage service requests. *L1 support ko mostly yahi role milta hai.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Global Admin or User Admin role access.
> - Access to `admin.microsoft.com`.
> - A valid available license in the tenant.

### Step 1: Create a New User and Assign License

1. Login to **Microsoft 365 Admin Center**.
2. Navigate to **Users** > **Active Users** on the left menu.
3. Click on **Add a user**.
4. Fill in the basics: First name, Last name, Display name, and Username (e.g., `john.doe@company.com`).
5. Choose password settings (auto-generate vs manual) and check "Require this user to change their password when they first sign in". *Naye user ko humesha force password change option dena chahiye.*

### Step 2: Assign Product Licenses

1. In the next screen, select the location (e.g., United States). **Location is mandatory before assigning a license.**
2. Check the box for the desired license (e.g., **Microsoft 365 E3**).
3. (Optional) Expand **Apps** to disable specific services like Yammer or Sway if company policy restricts them.

### Step 3: Review and Finish

1. Review the details summary.
2. Click **Finish adding**.

> [!success] Expected Output
> The user is created and will appear in the Active Users list within a few seconds. An email with temporary credentials can be securely sent to their manager.

### Step 4: Configuring Multi-Factor Authentication (MFA)

1. Go to **Users** > **Active Users**.
2. Click on **Multi-factor authentication** at the top navigation.
3. Select the user and click **Enable**.
4. *Security ke liye MFA hamesha enabled hona chahiye taaki passwords leak hone par bhi account safe rahe. Aaj kal ye Conditional Access se manage hota hai.*

### Step 5: Converting to a Shared Mailbox

1. Go to **Users** > **Active Users**, select the user.
2. Go to the **Mail** tab and click **Convert to shared mailbox**.
3. *Shared mailbox convert karne se data aur history save rehti hai bina monthly license pay kiye.*

---
## ⌨️ Command Cheat Sheet

We use the `Connect-MgGraph` module (formerly MSOnline / AzureAD) to manage M365 identities and licenses via PowerShell.

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Connect-MgGraph` | Connects your PowerShell session to the M365 tenant. | `Connect-MgGraph -Scopes "User.ReadWrite.All"` |
| `Get-MgUser` | Lists all users or specific user details. | `Get-MgUser -UserId "john@domain.com"` |
| `New-MgUser` | Creates a new cloud-only user. | `New-MgUser -DisplayName "John" -UserPrincipalName...` |
| `Update-MgUser` | Modifies existing user properties. | `Update-MgUser -UserId "john@domain.com" -Department "IT"` |
| `Revoke-MgUserSignInSession` | Forces sign-out from all active devices immediately. | `Revoke-MgUserSignInSession -UserId "user@domain.com"` |
| `Get-MgGroup` | Lists all groups in the tenant. | `Get-MgGroup -Filter "DisplayName eq 'IT Dept'"` |
| `Get-MgUserLicenseDetail` | Views assigned licenses for a specific user. | `Get-MgUserLicenseDetail -UserId "john@domain.com"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| User cannot access Exchange/Teams. | License is not assigned or is still provisioning. | Assign license. *Wait for 15-30 mins for backend Microsoft sync to complete.* |
| User password reset button greyed out. | User is synced from On-Prem AD (DirSync enabled). | Reset password in local On-Premise Active Directory, then force AD sync (`Start-ADSyncSyncCycle`). |
| Cannot add a custom domain. | TXT/MX records are not verified in external DNS host. | Go to DNS registrar (GoDaddy, Route53) and add the specific M365 TXT verification record. |
| Global admin locked out (MFA failure). | Device lost, no alternative method registered. | Use Break-Glass account, or contact another Global Admin to reset MFA sessions for that admin. |
| "This username is already in use" error. | Alias exists on a deleted user, contact, or group. | Check soft-deleted users in Entra ID or run a message trace/PowerShell search to find the conflicting proxy address. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Termination / Offboarding

> [!example] Ticket
> "Hi IT, employee Sarah from HR has resigned and today is her last day. Please block her access, convert her mailbox, and give her manager access to her emails."

**L1 Response:** 
1. Reset Sarah's password and block sign-in via the M365 Admin Center.
2. Run `Revoke-MgUserSignInSession` to kick her out of active mobile/web sessions. *Sirf password reset kaafi nahi hai, uske active device tokens ko expire karna padta hai.*

**Escalation Trigger:** If litigation hold or eDiscovery is required before deleting the account for legal reasons.

**L2 Resolution:** 
1. Convert Sarah's mailbox to a **Shared Mailbox** (via Exchange Admin Center).
2. Remove her M365 E3 license (saves cost for the company).
3. Grant "Full Access" delegation to the HR Manager so they can read her past emails.

### 🎫 Scenario 2: Shared Mailbox Access Issue

> [!example] Ticket
> "I was given access to the 'info@company.com' shared mailbox yesterday by HR, but it's not showing up in my Outlook client."

**L1 Response:** 
1. Verify in Exchange Admin Center that the user actually has "Full Access" permissions.
2. Explain that Auto-mapping takes time. *Permissions assign hone ke baad Outlook app mein auto-map hone mein 1 se 2 ghante lag sakte hain.*
3. Quick Fix: Tell user to access it immediately via Outlook on the Web (OWA) by clicking their profile picture -> "Open another mailbox".

**Escalation Trigger:** If it's still not syncing to desktop Outlook after 24 hours.

**L2 Resolution:** Manually add the shared mailbox as a secondary account in the Outlook desktop profile or disable/re-enable Auto-mapping via Exchange PowerShell.

### 🎫 Scenario 3: Email Delivery Blocked (High Risk User)

> [!example] Ticket
> "My outgoing emails are bouncing back with an NDR error saying 'User is restricted from sending email'."

**L1 Response:** Check the user's sent items and sign-in logs for spam/phishing activity. *Agar kisi ne user ka account hack karke bulk spam bheja hai, toh Microsoft security feature (EOP) usse auto-block kar deta hai.*

**Escalation Trigger:** Account compromise is confirmed and the user needs to be unblocked urgently.

**L2 Resolution:** 
1. Reset password, clear malicious inbox forwarding rules, and enforce strict MFA.
2. Go to **Microsoft Defender Security Center > Email & collaboration > Review > Restricted entities**.
3. Unblock the user from the restricted users portal (can take up to 1 hour to take effect).

### 🎫 Scenario 4: User Needs Access to Another User's Calendar

> [!example] Ticket
> "I am the new assistant. I need to manage my director's calendar to schedule and cancel meetings on his behalf."

**L1 Response:**
1. Identify this as a calendar delegation request.
2. Direct the user to ask the director to share the calendar directly from their own Outlook client with "Editor" or "Delegate" permissions. *L1 mostly direct users ko guide karta hai self-service ke liye taaki security maintain rahe.*

**Escalation Trigger:** The director is unavailable (on leave/traveling) and HR/Management has explicitly approved the request.

**L2 Resolution:**
1. Open Exchange Admin Center or use PowerShell to bypass user action.
2. Grant "Editor" permissions to the user on the director's calendar folder.
`Add-MailboxFolderPermission -Identity "director@domain.com:\Calendar" -User "assistant@domain.com" -AccessRights Editor`

### 🎫 Scenario 5: User Needs to Send as a Shared Mailbox

> [!example] Ticket
> "I can read emails from the Sales shared mailbox, but when I try to reply, it comes from my personal email address instead of sales@company.com."

**L1 Response:**
1. Check the shared mailbox permissions in the Exchange Admin Center.
2. Ensure the user has "Send As" or "Send on Behalf" permissions, not just "Full Access". *Full access se sirf read kar sakte hain, send karne ke liye alag permission chahiye.*

**Escalation Trigger:** Permissions are granted but the user still receives an NDR when sending.

**L2 Resolution:**
1. Force download the Offline Address Book (OAB) in the user's Outlook client.
2. Remove the auto-complete cache for the "From" address and manually select the shared mailbox from the Global Address List (GAL).

---
## 🎤 Interview Questions

> [!question] Q1: What happens to a user's data when you remove their Microsoft 365 license?
> **Answer:** The data (Exchange mailbox, OneDrive files) goes into a grace period (typically 30 days). After 30 days, the data is permanently deleted unless a retention policy is applied or the mailbox was converted to a Shared Mailbox *before* removing the license.

> [!question] Q2: How do you differentiate between a Distribution Group and a Microsoft 365 Group?
> **Answer:** A Distribution Group is purely for routing emails to multiple recipients at once. A Microsoft 365 Group provisions an entire collaboration space, including a shared SharePoint site, a shared calendar, a Planner board, and optionally a Teams workspace. *Agar sirf email bhejni hai to DL use karo, file sharing aur chat bhi karni hai to M365 Group.*

> [!question] Q3: You are unable to edit a user's profile information (like Job Title or Department) in the M365 Admin Center. The fields are completely greyed out. Why?
> **Answer:** The organization is running a Hybrid AD environment. The user account is authoritative in the On-Premises Active Directory and is being synced via Azure AD Connect (DirSync). Changes must be made in the local on-prem AD and synced to the cloud.

> [!question] Q4: What is the difference between Soft Delete and Hard Delete in Microsoft 365?
> **Answer:** 
> - **Soft Delete:** When you delete a user in M365 Admin Center, they are moved to "Deleted users" and can be fully restored (along with their mailbox and OneDrive data) within 30 days.
> - **Hard Delete:** Permanently removing the user using PowerShell (`Remove-MgUser` or clearing from the recycle bin) before the 30-day window expires. Data is unrecoverable unless third-party backups exist.

> [!question] Q5: A user reports they are not receiving any emails from external senders, but internal emails work perfectly fine. Where do you start troubleshooting?
> **Answer:** 
> 1. Check the domain's MX (Mail Exchanger) records in the public DNS to ensure they correctly point to Microsoft 365 (`domain-com.mail.protection.outlook.com`).
> 2. Run a Message Trace in Exchange Admin Center to see if the external emails are hitting the M365 tenant and being dropped, delayed, or quarantined by spam filters.

> [!question] Q6: How do you protect administrative accounts in Microsoft 365 from brute-force or credential stuffing attacks?
> **Answer:** Implement Conditional Access policies requiring MFA for all admin roles, restrict admin access to trusted locations (e.g., corporate office IPs), and monitor sign-in logs. Additionally, use Privileged Identity Management (PIM) for Just-In-Time access so admin roles are not permanent.

==**Exam Tip:** Always remember the 30-day retention rule for deleted users and removed licenses. This is a very common exam and interview topic for any cloud administrator role!==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/M365-02 Exchange Online Admin|M365-02 Exchange Online Admin]] — Deep dive into mail flow, transport rules, and routing.
- [[04-Cloud-and-Security/06-Azure/AZ-03 Azure Active Directory|AZ-03 Azure Active Directory (Entra ID)]] — Identity management, Conditional Access, and SSO foundation.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-05 SharePoint and OneDrive|M365-05 SharePoint and OneDrive]] — File sharing, permissions, and collaboration.
- [[01-Windows-Server/WS-04 Active Directory Basics|WS-04 Active Directory Basics]] — On-premises AD for understanding hybrid setups.
