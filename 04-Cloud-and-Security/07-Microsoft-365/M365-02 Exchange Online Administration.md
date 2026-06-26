---
tags: [desktop-support, m365, collaboration, L1]
aliases: [m365-02-exchange-online-administration, m365-02]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# M365-02: Exchange Online Administration

> [!abstract] Overview
> This note covers mailboxes, mail flow, connectors, security filtering, retention policies, and administrative operations in Exchange Online. It explains how to manage user and resource environments, trace messages, and construct robust mail flow architectures.

---

---
## Concept Overview
Think of Exchange Online as the digital postal service of your organization. Every user has a personal physical mailbox (User Mailbox), while common rooms have a shared physical mailbox where anyone can pick up/drop off mail (Shared Mailbox), conference rooms have calendar sign-up sheets (Resource/Equipment Mailboxes), and sorting departments use delivery routes to broadcast letters to multiple people at once (Distribution Lists).

---

---
## Technical Deep Dive
### 1. Mailbox Types
- **User Mailbox**: Assigned to an individual user. Requires a license. Default storage is 50 GB or 100 GB based on the subscription tier (e.g., Business Premium vs. Enterprise E5).
- **Shared Mailbox**: Used for multiple users to read and send email from a common address (e.g., `info@company.com`). Does not require a license if under 50 GB and does not have an archive mailbox. Cannot be logged into directly; users must be delegated access.
- **Resource Mailbox (Room)**: Assigned to a physical location (e.g., Conference Room A). Used for scheduling. Automatically processes booking requests based on calendar availability.
- **Equipment Mailbox**: Assigned to a non-location resource (e.g., Company Car, Projector). Works similarly to room mailboxes.

### 2. Permissions Hierarchy
- **Full Access**: Allows a delegate to open the mailbox and read/write contents. It does *not* grant permission to send mail as that mailbox.
- **Send As**: Allows a delegate to send emails that appear to originate directly from the mailbox. The recipient sees only the mailbox's address in the `From` field.
- **Send on Behalf**: Allows a delegate to send emails on behalf of the mailbox. The recipient sees: `Delegate on behalf of Mailbox`.

### 3. Distribution Groups & Groups
- **Static Distribution List (DL)**: Members are manually added and managed.
- **Dynamic Distribution Group**: Membership is calculated automatically using recipient filters (e.g., all users with `Department -eq 'Sales'`).
- **Microsoft 365 Groups**: Collaborative group that includes a shared mailbox, calendar, SharePoint site, and Teams workspace.

### 4. Mail Flow & Security Architecture
- **Connectors**: Establish secure inbound/outbound mail flow routes between Exchange Online and on-premises email servers or third-party gateways (e.g., smart hosts).
- **Mail Flow (Transport) Rules**: Evaluate messages in transit. Conditions, exceptions, and actions allow administrators to block, modify, or redirect emails based on headers, subjects, body content, or attachment characteristics.
- **Message Trace**: Detailed tracking utility showing whether a message was received, deferred, rejected, or delivered by Exchange Online. Retention of logs is 10 days for immediate search, and up to 90 days via historical search.
- **Anti-Spam / Anti-Malware Policies**: Integrated filtering engines in Exchange Online Protection (EOP) that scan inbound, outbound, and internal mail for bulk mail, malicious attachments, and phishing attempts.
- **Public Folders**: Legacy hierarchy designed for shared access to files, calendars, and emails. Organized in a root mailbox and scaled using Public Folder mailboxes.
- **Litigation Hold vs. In-Place Hold**:
  - **Litigation Hold**: Places all mailbox content (including deleted items and original versions of modified items) on indefinite or specified duration hold.
  - **In-Place Hold**: A legacy granular hold based on search parameters (query-based). Modern environments use **Purview Retention Policies** to apply holds across multiple locations.

---

## Common Mistakes
> [!warning] Avoid These
> Assigning licenses to Shared Mailboxes that are under 50 GB. This wastes valuable subscription costs.
> Directing users to log into shared mailboxes using a dedicated username and password. This breaks security accountability, disables MFA tracking, and violates compliance.
>  Assign delegate permissions (`FullAccess`, `SendAs`) and expect them to work instantly. AD replication and Exchange Directory syncing can take anywhere from 15 minutes to 2 hours to propagate across Outlook.

---

## Pro Tips
> [!tip] Field Experience
> Always disable direct login on the Azure Active Directory user account associated with a Shared Mailbox. Block sign-in for the shared account to prevent credential cracking attempts, as it only needs delegate authentication.
> Use `Automapping` carefully. When granting `FullAccess` via PowerShell, it defaults to AutoMapping `$true`, which automatically mounts the shared mailbox in Outlook. If a user has access to 20+ shared mailboxes, it will bloat their Outlook profile and cause performance degradation. Use `-AutoMapping $false` in such cases.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Access to Microsoft 365 Admin Center with Exchange Administrator role, and Exchange Online PowerShell installed.

### Step 1: Connect to Exchange Online PowerShell
Open PowerShell as an Administrator and authenticate:
```powershell
Install-Module -Name ExchangeOnlineManagement -Force
Connect-ExchangeOnline -UserPrincipalName admin@yourtenant.onmicrosoft.com
```

### Step 2: Create a Shared Mailbox and Delegate Permissions
Create a shared mailbox named `Support` and grant Full Access and Send As permissions to a user `jdoe@yourtenant.onmicrosoft.com`:
```powershell
New-Mailbox -Name "Customer Support" -Alias "support" -Shared
Add-MailboxPermission -Identity "support" -User "jdoe@yourtenant.onmicrosoft.com" -AccessRights FullAccess -InheritanceType All
Add-RecipientPermission -Identity "support" -Trustee "jdoe@yourtenant.onmicrosoft.com" -AccessRights SendAs -Confirm:$false
```

### Step 3: Create a Custom Mail Flow Rule (Transport Rule)
Create a transport rule that prepends `[EXTERNAL]` to the subject line of any email coming from outside the organization, except if the sender is from trusted partner `trusteddomain.com`:
```powershell
New-TransportRule -Name "Prepend External Sender Warning" -FromScope NotInOrganization -ExceptIfSenderDomainIs "trusteddomain.com" -PrependSubject "[EXTERNAL] "
```

### Step 4: Perform a Message Trace using PowerShell
Trace emails sent to `support@yourtenant.onmicrosoft.com` in the last 24 hours to verify delivery:
```powershell
Get-MessageTrace -RecipientAddress "support@yourtenant.onmicrosoft.com" -StartDate (Get-Date).AddDays(-1) -EndDate (Get-Date)
```

---

---
## Cheat Sheet / Quick Reference
```powershell
Get-Mailbox -ResultSize Unlimited                    # Lists all mailboxes in the tenant
Get-MailboxPermission -Identity "support"            # Displays delegate permissions on a specific mailbox
Set-Mailbox -Identity "jdoe" -DeliverToMailboxAndForward $true -ForwardingAddress "backup@domain.com" # Configures forwarding
Set-Mailbox -Identity "sales" -LitigationHoldEnabled $true -LitigationHoldDuration 365                # Enables litigation hold for 1 year
Get-DynamicDistributionGroupMember -Identity "Sales" # Evaluates and lists current members of a dynamic group
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Shared Mailbox | Unlicensed mailbox up to 50 GB accessed solely by delegated permissions. |
| 2 | Send As | Delegate sends mail appearing directly from the shared address. |
| 3 | Send on Behalf | Delegate sends mail displaying "User on behalf of Shared Address". |
| 4 | Message Trace | Audit logs to track status, delivery paths, and EOP filtering results of emails. |
| 5 | Litigation Hold | Immutable preservation of all mailbox items for legal compliance and retention. |

---

---
## Troubleshooting
Advanced content only — basics in [[Outlook troubleshooting]]

**Scenario 1:**
- Problem: User reports they cannot send email from a shared mailbox they were recently granted permissions to. They get a "This message could not be sent. You do not have the permission to send the message on behalf of the specified user" bounce back.
- Root Cause: The client-side Outlook Address Book (OAB) cache has not synchronized since the permissions were added, or the user is trying to reply and Outlook is defaulting to "Send on Behalf" instead of "Send As".
- Fix:
  1. Verify the delegate has `SendAs` permissions via PowerShell using `Get-RecipientPermission`.
  2. In Outlook, instruct the user to force download the Address Book: `Send / Receive` -> `Send/Receive Groups` -> `Download Address Book`.
  3. When composing a new email, click `From` -> `Other Email Address...`, manually type the shared mailbox address to clear any bad autocomplete entry.

**Scenario 2:**
- Problem: Critical emails from a specific vendor are intermittently failing to deliver, and the vendor gets a "550 5.7.1 Service unavailable" bounce-back error.
- Root Cause: The vendor's domain lacks proper SPF, DKIM, or DMARC configurations, causing the Exchange Online Protection (EOP) inbound spam filters to reject the emails.
- Fix:
  1. Run a `Get-MessageTrace` on the incoming sender address to verify the exact failure status and sub-code.
  2. If blocked due to spoofing/authentication failure, analyze the email headers using the **Microsoft Message Analyzer (MXToolbox)**.
  3. Inform the vendor to correct their SPF record (adding their sending IP ranges).
  4. As a temporary workaround, configure an Exchange Tenant Allow/Block List entry or a mail flow rule to bypass filtering for their IP address (never whitelist wildcard domains for safety).

---

---
## Interview Questions
**Q1: What is the difference between Send As and Send on Behalf permissions, and how would you configure them using Exchange Online PowerShell?**
A: `Send As` allows a delegate to send mail appearing directly as the mailbox owner, and the recipient will not know a delegate sent it. `Send on Behalf` displays both names to the recipient (e.g., "John Doe on behalf of Support"). To configure them, I use `Add-RecipientPermission -Identity "support" -Trustee "jdoe" -AccessRights SendAs` for Send As, and `Set-Mailbox -Identity "support" -GrantSendOnBehalfTo "jdoe"` for Send on Behalf.

**Q2: A client reports that they sent a critical contract to an external vendor, but the vendor claims they never received it. How would you investigate this?**
A:
- **Situation**: An email sent to an external recipient was reported as missing, requiring validation of outbound mail delivery.
- **Task**: I needed to trace the mail flow path from the user's outbox through Exchange Online Protection (EOP) to the external mail transfer agent (MTA).
- **Action**: I initiated a Message Trace in the Exchange Admin Center for the sender and recipient addresses over the last 24 hours. The trace showed the status as "Delivered" and returned the exact Outbound IP, recipient MTA IP, and the "250 OK" response code with the transactional ID.
- **Result**: I provided the transmission logs containing the 250 OK status and timestamp to the client, proving Exchange successfully handed off the mail to the vendor's mail server. This shifted the troubleshooting focus to the vendor's local filters.

**Q3: How do Dynamic Distribution Groups evaluate membership, and why might an eligible user not receive an email sent to the group?**
A: Dynamic Distribution Groups use active filters against Azure AD attributes (like `Department` or `State`) to compile the recipient list at the moment of email dispatch. If a user is not receiving group mail, it's typically because: their AD attribute does not match the filter exactly (e.g., "Sales " with a trailing space instead of "Sales"), their account is hidden from the Global Address List (GAL), or they are configured with external forwarding that violates the tenant's outbound anti-spam forwarding limits.

---

---
## Seedha Simple Mein
*Seedha simple mein: Exchange Online aapke organization ka global post office hai. Alag-alag tarah ke mailboxes aur dynamic routing rules (transport rules) ke sath aap email flows, security rules aur compliance retention ko control karte ho.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Configures global tenants and identity configurations.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-05 Security and Compliance|M365-05 Security and Compliance]] — Integrates DLP, mail flow restrictions, and auditing.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-02 PowerShell for Active Directory|PS-02 PowerShell for Active Directory]] — Handles user attributes that feed into dynamic groups.
