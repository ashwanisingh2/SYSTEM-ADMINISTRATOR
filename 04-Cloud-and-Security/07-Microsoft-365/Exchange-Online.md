---
tags: [desktop-support, m365, exchange-online, mail-flow, L2]
aliases: [exchange-guide, mail-troubleshooting, message-trace]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Exchange Online

---

## Concept Overview
- **What it is**: Exchange Online is Microsoft's cloud-based email, calendar, and collaboration service. It is the enterprise cloud equivalent of an on-premises Exchange Server, hosting user mailboxes, shared mailboxes, and routing email traffic globally.
- **Why it matters for a support engineer**: Email is a business-critical system. Support engineers spend hours running message traces to locate lost emails, managing shared mailbox access, resolving Outlook sync issues, and releasing legitimate emails caught in the quarantine filter.
- **Where you encounter this in real job**: Troubleshooting "email bounce back" messages (NDRs), converting departing employee mailboxes to shared mailboxes, setting up delegate access for assistants, and verifying email delivery status.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Creates email aliases, resets mailbox passwords, grants Full Access/Send As permissions, and configures basic Outlook profiles.
  - **L2**: Runs Exchange Message Traces to locate missing emails, manages mobile device access policies, releases files from quarantine, and handles shared mailbox conversions.
  - **L3**: Configures mail flow rules (transport rules), designs mail connectors, manages hybrid migration batches, and configures security records (SPF, DKIM, DMARC).

---

## Technical Deep Dive

### 1. DNS Records for Email Delivery & Security
Email delivery and security depend on four DNS records configured on your public domain registrar:
1. **MX (Mail Exchanger) Record**: Points to the mail server responsible for accepting emails on behalf of your domain. For Exchange Online, it looks like: `company-com.mail.protection.outlook.com`.
2. **SPF (Sender Policy Framework)**: A TXT record listing all authorized IP addresses/servers allowed to send emails from your domain. Example: `v=spf1 include:spf.protection.outlook.com -all`.
3. **DKIM (DomainKeys Identified Mail)**: Adds a digital cryptographic signature to emails. The receiving server validates this signature against your public DNS key to verify the email was not modified in transit.
4. **DMARC (Domain-based Message Authentication, Reporting, and Conformance)**: A TXT record instructing receiving servers what to do if an incoming email fails SPF or DKIM checks (Options: `none` to monitor, `quarantine` to send to spam, `reject` to block).

### 2. Recipient Types in Exchange Online
Exchange Online uses several recipient types to manage collaborative workspaces:
- **User Mailbox**: Standard mailbox assigned to a single licensed user (includes email, calendar, contacts).
- **Shared Mailbox**: Used by multiple users to read and send email from a common address (e.g., `info@company.com`). Does not require a license if kept under 50GB, but users must have delegate permissions.
- **Resource / Room Mailbox**: Used to reserve physical meeting spaces or equipment (e.g., projector). Includes auto-accept/decline rules based on calendar availability.
- **Distribution Group**: A list of email addresses. Sending to the group forwards the message to all members.
- **Mail Contact**: An external email address listed in the Global Address List (GAL) so users can find them easily.

### 3. Message Tracing Flow
When you run a Message Trace in the Exchange Admin Center, you search the mail flow logs. The trace results display status indicators:

```
[Incoming Email] --> [Exchange Online Protection (EOP)] --> [Spam/Phish Check] --> [Inbox/Junk]
                                                             | (Detected)
                                                             v
                                                      [Quarantine]
```
- **Delivered**: The message was successfully placed in the user's Inbox or Junk Folder.
- **Quarantined**: The message was flagged as spam, phishing, or malware and redirected to the quarantine portal.
- **Failed**: The message was rejected and blocked. An NDR (Non-Delivery Report) was sent to the sender.
- **Expanded**: The email was sent to a distribution list and split into separate messages for each recipient.

---

## Commands & Syntax

### PowerShell
Exchange Online is administered using the `ExchangeOnlineManagement` module.
```powershell
# Install and connect to Exchange Online
Install-Module -Name ExchangeOnlineManagement -Force
Connect-ExchangeOnline -UserPrincipalName admin@company.com

# Convert a standard user mailbox to a Shared Mailbox
Set-Mailbox -Identity "retired.user@company.com" -Type Shared

# Grant a user full access and send-as permissions to a Shared Mailbox
Add-MailboxPermission -Identity "info@company.com" -User "jdoe@company.com" -AccessRights FullAccess -InheritanceType All
Add-RecipientPermission -Identity "info@company.com" -Trustee "jdoe@company.com" -AccessRights SendAs -Confirm:$false

# Run a message trace to find email from a specific sender to a recipient in the last 2 days
Get-MessageTrace -SenderAddress "client@external.com" -RecipientAddress "jdoe@company.com" -StartDate (Get-Date).AddDays(-2) -EndDate (Get-Date)
```

### CMD / Run Box
```cmd
:: Verify the public MX record for a target domain from the command line
nslookup -type=MX company.com

:: Verify the public SPF TXT record
nslookup -type=TXT company.com
```

### GUI Path
> Open browser -> Go to **admin.exchange.microsoft.com** -> **Mail flow** -> **Message trace** -> **Start a trace**.
> For Quarantine: Go to **security.microsoft.com** -> **Email & collaboration** -> **Review** -> **Quarantine**.

### Important Registry Paths
Outlook cache and profile Autodiscover bypass:
```
HKCU\Software\Microsoft\Office\16.0\Outlook\AutoDiscover\
(Create DWORD "ExcludeHttpsRootDomain" set to 1 to resolve lookup loops)
```

---

## Real-World Scenarios

### Scenario 1: External Client Email is Missing (Caught in Quarantine)
**User Complaint:** A VP of Finance complains: *"An external vendor claims they sent me the invoice contract hours ago, but I have not received it. I checked my Junk folder and it is empty. I need this contract immediately."*
**Your First 3 Checks:**
1. Check the sender's exact email address.
2. Run a message trace in the Exchange Admin Center (EAC).
3. Check the Security and Compliance Quarantine portal.
**Diagnosis Steps:**
1. Run a Message Trace in EAC for the sender `vendor@external.com` targeted to the VP `vp@company.com`.
2. Look at the trace results. Locate the message.
   - Status: `Failed` or `Quarantined`.
   - Error: `Spam filter matched. Reason: High Confidence Phish.`
3. The email contained a PDF attachment with suspicious redirect links, causing Exchange Online Protection (EOP) to divert it to the Quarantine portal instead of the user's mailbox.
**Root Cause:** The external sender's email was incorrectly classified as "High Confidence Phishing" by EOP rules due to a suspicious link inside the attached file.
**Fix:**
1. Go to the Microsoft Defender Quarantine portal (`security.microsoft.com/quarantine`).
2. Search for the quarantined message. Select the email and click **Release email**.
3. Check the box **Release to all recipients** and check **Submit the message to Microsoft for analysis (False Positive)**.
4. Verify with the VP that the email has arrived in their inbox.
**Prevention:** If the sender is verified, advise them to correct the link structures in their files, or add their domain to the Safe Senders list if it's a persistent partner (use caution with phish overrides).
**Ticket Close Note:** "Located email in Quarantine. Classified as false positive phish. Released message to user's inbox and submitted false positive report. Verified receipt. Closed."

### Scenario 2: Employee Termination Mailbox Offboarding Workflow
**User Complaint:** HR submits a ticket: *"Jane Doe is leaving the company today. Please block access to her account, disable login, convert her mailbox to a shared mailbox, and grant her manager 'John Smith' access to view her emails."*
**Your First 3 Checks:**
1. Confirm the manager's identity and required access level (Full Access vs Send As).
2. Verify that the user's license is active (needed during the conversion process).
3. Identify if the mailbox is a hybrid mailbox or pure cloud mailbox.
**Diagnosis Steps:**
1. In M365 Admin Center, block the sign-in for `jdoe@company.com` and reset her password.
2. Run PowerShell to convert the mailbox to a shared type:
   `Set-Mailbox -Identity "jdoe@company.com" -Type Shared`
3. Assign Full Access and Send As permissions to the manager, `jsmith@company.com`.
4. *Crucial Decision*: Wait for the conversion to complete, then remove the M365 license. A shared mailbox under 50GB does not require a license to exist, saving the company money.
**Root Cause:** Standard employee offboarding procedure to preserve business correspondence.
**Fix:**
1. Convert mailbox to shared type using Exchange PowerShell.
2. Grant manager permissions.
3. Remove user license in M365 Admin Center.
4. Hide the mailbox from the Global Address List (GAL) if requested: `Set-Mailbox "jdoe@company.com" -HiddenFromAddressListsEnabled $true`.
**Prevention:** Automate this process using a standardized PowerShell offboarding script.
**Ticket Close Note:** "Blocked logon for jdoe@company.com. Converted mailbox to Shared type. Assigned Full Access and Send As rights to manager jsmith@company.com. Removed license. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never add wildcards (e.g., `*`) or allow major public domains (like `gmail.com` or `outlook.com`) to the Tenant Allow/Block List (Bypass Anti-Spam).
> - This completely disables spam and phishing scanning for those domains, allowing spoofed emails to enter users' inboxes directly, leading to ransomware or credential theft.

> [!warning] Common Trap
> - Removing a user's license BEFORE converting their mailbox to a shared mailbox.
> - If you strip the license first, the mailbox will be marked for deletion. You must convert the mailbox to Shared type while it is still active/licensed. Once converted, you can safely remove the license.

> [!tip] Senior Engineer Tip
> - When troubleshooting Outlook connection failures or slow syncing, bypass Outlook altogether by using Microsoft's free tool: **Remote Connectivity Analyzer** (`testconnectivity.microsoft.com`). This runs a simulation of Autodiscover and ActiveSync from Microsoft servers, isolating whether the issue is on-premises (workstation/network) or in the cloud.

> [!success] Verification Steps
> - Run: `Get-Mailbox -Identity "username@company.com" | Select-Object RecipientTypeDetails` to verify the conversion worked.
> - Verify in the manager's Outlook client that the Shared Mailbox automaps and appears in their folder pane (may require a restart or up to 1 hour).

> [!question] Interview Alert
> - "What is the difference between 'Full Access' and 'Send As' permissions on an Exchange mailbox?"
> - Answer: "Full Access allows a delegate to open the mailbox, view calendars, and read emails, but does not allow them to send messages. Send As allows the delegate to send emails appearing as if they came directly from that mailbox owner. To do both, you must assign both permissions."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Deleting a departed user's account to save licensing costs | Unaware of data loss | Convert the mailbox to Shared type, assign delegate permissions, and remove the license. |
| Forgetting to update SPF record when adding mailing services (e.g., Mailchimp) | Senders lack DNS knowledge | Add `include:servers.mcsv.net` inside the existing SPF TXT record instead of creating a second SPF record. |
| Creating multiple SPF records | Misunderstanding DNS specs | A domain must only have ONE SPF TXT record. Combine multiple records into a single TXT entry. |

---

## Lab Exercise

**Objective:** Connect to Exchange Online via PowerShell, convert a mailbox to a shared mailbox, assign permissions, and check SPF health.
**Time Required:** 20 minutes
**Environment Needed:** A computer with internet access and a Microsoft 365 test tenant with administrative credentials.
**Pre-requisites:** The `ExchangeOnlineManagement` module installed.

**Steps:**
1. Open PowerShell as Administrator.
2. Install the module if not present: `Install-Module -Name ExchangeOnlineManagement -Force`.
3. Connect to your tenant:
   ```powershell
   Connect-ExchangeOnline -UserPrincipalName admin@yourtestdomain.onmicrosoft.com
   ```
4. Find a test user mailbox and convert it to Shared type:
   ```powershell
   Set-Mailbox -Identity "testuser@yourtestdomain.onmicrosoft.com" -Type Shared
   ```
5. Assign Full Access rights to your administrator account:
   ```powershell
   Add-MailboxPermission -Identity "testuser@yourtestdomain.onmicrosoft.com" -User "admin@yourtestdomain.onmicrosoft.com" -AccessRights FullAccess -InheritanceType All
   ```
6. Verify the changes:
   ```powershell
   Get-Mailbox -Identity "testuser@yourtestdomain.onmicrosoft.com" | Select-Object Name, RecipientTypeDetails
   ```
   - Expected Output: `RecipientTypeDetails: SharedMailbox`.

**Success Criteria:** The target user mailbox is converted to a shared type, and permissions are successfully assigned and verified via PowerShell.
**Common Failures:** The command fails if the user account is synchronized from on-premises Active Directory (must be modified on-premises in hybrid setups).

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a shared mailbox and does it require a Microsoft 365 license?**
A: A shared mailbox is an email folder that multiple users can access to read and send messages. It does not require a license as long as it has less than 50GB of data and does not have an active user logging in directly.

**Q: A user did not receive an email. What is the first tool you use in Microsoft 365 to investigate?**
A: I use the **Message Trace** tool in the Exchange Admin Center. I enter the sender's email, the recipient's email, and search the log to see if the message was delivered, rejected, or sent to quarantine.

### Intermediate (L2 Level)
**Q: What is a Non-Delivery Report (NDR) and which details do you look for inside it?**
A: An NDR is an automatic notification sent to a sender when their email fails to deliver. I look for the status code (e.g., `5.7.1` for access denied, or `5.1.1` for user unknown) and the host server information to determine if the issue is with the sender's DNS records, a local block list, or a misspelled address.

**Q: Why is it bad practice to create two SPF records on a domain?**
A: DNS RFC specifications dictate that a domain can only have a single SPF record. If a domain contains multiple SPF records, receiving mail servers will fail the validation check automatically, rejecting or flagging all emails sent from that domain.

### Advanced (L3/Senior Level)
**Q: A client reports that emails sent from our domain are going to their spam folder. How do you troubleshoot this email delivery issue?**
A:
- **Situation**: Outbound corporate emails are classified as spam by external partners.
- **Task**: Audit outbound email verification records and server reputation.
- **Action**: I run an nslookup to verify our SPF record contains our sending systems. Next, I inspect our DKIM keys to ensure messages are signed. I then check our DMARC policy using public analyzers, and query international email blacklist databases (like MXToolbox) to see if our sending IP addresses have been blacklisted. Finally, I inspect the email header of a failed message sent to a test account to review the spam confidence level (SCL) scores.
- **Result**: I identified a third-party CRM server sending email on our behalf without an SPF record entry. Adding the server to the SPF record resolved the issue.

### HR / Behavioral
**Q: Tell me about a time you had to deal with an demanding user who wanted an immediate bypass of a security rule.**
A: An executive was blocked from receiving an email containing a macro-enabled Excel sheet that was quarantined. They demanded I whitelist the sender and bypass the scan. I calmly explained the risk of ransomware associated with macros and offered to download the file in a sandbox environment, strip the macro, verify the contents, and send them the clean file. They appreciated the quick solution while keeping the company safe.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: Microsoft's cloud-based messaging platform managing corporate email.
> **Why**: Core communication platform; critical for monitoring mail flow and troubleshooting delivery.
> **How**: Configure MX, SPF, DKIM, and DMARC DNS records, and manage user/shared mailboxes via the Exchange Admin Center.
> **Command**: `Get-MessageTrace` / `Set-Mailbox` / `Add-MailboxPermission`
> **Interview Answer Starter**: "To troubleshoot email routing and delivery issues, I rely on Exchange Message Traces and check domain security configurations like SPF and DKIM..."

**Key Numbers to Remember:**
- Max storage for shared mailbox without license: 50 GB
- Default replication time for license changes: 15-30 minutes
- Maximum days back you can run a quick message trace: 10 days (requires extended trace after that)
- Outlook cache file extension: `.ost`

**3 Things Interviewer Wants to Hear:**
- How MX, SPF, DKIM, and DMARC work together to prevent spoofing
- Converting departing user accounts to shared mailboxes to save licensing costs
- Using the Security and Compliance portal to manage quarantined phish

---

## Related Notes
- [[01-Foundations/02-Networking/DNS|DNS]] — Details the public records required for email delivery.
- [[04-Cloud-and-Security/07-Microsoft-365/Licensing|Licensing]] — Explains M365 SKUs and mailbox storage capacities.
- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]] — Covers MFA enforcement when accessing Outlook.

---

## Tags
#desktop-support #m365 #exchange-online #mail-flow #L2 #interview-topic #lab-complete #daily-use

