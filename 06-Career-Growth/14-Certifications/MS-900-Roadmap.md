---
tags: [desktop-support, career, certification, m365, L1]
aliases: [ms-900-roadmap, roadmap-ms900]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

# MS-900: Microsoft 365 Fundamentals Roadmap

> [!note] Overview
> This roadmap note covers preparation for the Microsoft 365 Fundamentals (MS-900) exam. It details the core concepts of M365 cloud services, administration, licensing options, security and compliance architectures, and collaboration tools.

---
## Concept Overview
- **What it is** — The MS-900 exam validates foundational-level knowledge of Microsoft 365 cloud service offerings, including SaaS collaboration tools, licensing, security, and cloud administration.
- **Why it matters** — Microsoft 365 is the standard collaboration platform in the corporate world. Support engineers must understand how Exchange Online, Teams, SharePoint, OneDrive, and device management (Intune) work together to resolve user issues.
- **Real job encounter** — Resetting user passwords in the M365 Admin Center, creating shared mailboxes for team collaborations, and explaining licensing options (e.g., E3 vs. E5) to management.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Create user accounts, assign licensing, create standard mailboxes, configure aliases, and assist with Outlook sync issues.
  - **Escalation Trigger**: Escalate when email delivery fails globally, tenant-wide licensing shortages occur, or when mail flow rules block legitimate emails.
  - **L2 Resolution**: Manage Shared Mailbox permissions, configure Microsoft Teams channels, configure OneDrive synchronization settings, and manage quarantine email lists.
  - **L3 Resolution**: Implement Intune MDM policies, configure Exchange mail flow rules (transport rules), manage eDiscovery hold search cases, and manage hybrid directory synchronization.

*Seedha simple mein: MS-900 exam Microsoft 365 SaaS products ka foundation level test hai. Isme hum seekhte hain ki Exchange (Emails), Teams (Collaboration), SharePoint (Data Storage), and Intune (Device management) kaise kaam karte hain, aur business requirements ke according kaun sa license (E3 ya E5) recommend karna chahiye.*

---
## Technical Deep Dive

### 1. Exam Domains and Weightage
- **Describe Cloud Concepts (10–15%)**: Public, Private, and Hybrid clouds, SaaS, PaaS, IaaS benefits, and cloud cost management.
- **Describe Microsoft 365 Core Services and Concepts (50–55%)**: Exchange Online, SharePoint Online, OneDrive for Business, Microsoft Teams, Microsoft 365 Apps, and Microsoft Endpoint Manager (Intune).
- **Describe Security, Compliance, Privacy, and Trust in Microsoft 365 (15–20%)**: Zero Trust, Conditional Access, Multi-Factor Authentication, Unified Audit Log, and Service Trust Portal.
- **Describe Microsoft 365 Pricing and Support (10–15%)**: Licensing (E3, E5, F3, Business Premium), billing management, service life cycles (Private Preview, Public Preview, General Availability), and SLAs.

### 2. Core Concepts: SharePoint vs. OneDrive
Support engineers must understand the storage division:
- **OneDrive for Business**: Designed for personal work storage. Files are private by default but can be shared manually.
- **SharePoint Online**: Designed for team collaboration and organization-wide storage. Permissions are typically managed by site/group membership.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - Access to a Microsoft 365 Admin Center (e.g., via a free M365 Developer sandbox subscription).
> - Permissions of at least **User Administrator** or **Global Administrator**.

### Step 1: Create a Shared Mailbox
Shared mailboxes allow multiple users to monitor and send emails from a common address (e.g., `info@mybrand.com` or `support@mybrand.com`). They do not require a separate license.

1. Log into the **Microsoft 365 Admin Center** (`admin.microsoft.com`).
2. Navigate to **Teams & groups** -> **Shared mailboxes** in the left navigation pane.
3. Click **+ Add a shared mailbox**:
   - **Name**: `IT Helpdesk Support`
   - **Email**: `helpdesk@mybrand.com` (Select your verified domain).
4. Click **Save changes**.
**Expected Result:** The shared mailbox is created successfully.

### Step 2: Assign Access Permissions
1. Once created, click on the shared mailbox name.
2. Under **Members**, click **Edit** next to **Read and manage permissions**.
3. Click **+ Add members**, select the support engineers (e.g., `Ashwani Singh` and `Priya Sharma`), and click **Save**.
4. Repeat the step for **Send as permissions** to allow engineers to reply using the helpdesk email address.
**Expected Result:** The shared mailbox is automatically mapped to the users' Outlook clients within a few hours.

---
## Cheat Sheet / Quick Reference

| Licensing | Target Audience | Key Features Included |
|---|---|---|
| **Microsoft 365 F3** | Frontline workers | Mobile apps, web-only mail (2GB mailbox), Teams |
| **Microsoft 365 E3** | Information workers | Desktop Apps, 100GB Mailbox, Basic Security & Compliance |
| **Microsoft 365 E5** | Enterprise Power users | Everything in E3 + Advanced Security (Defender), Power BI, Phone System |
| **Business Premium** | Small & Medium Business | Up to 300 users. Includes Intune device management and Azure AD P1 |
| **General Availability**| Release state | Fully supported production-ready release |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command / URL |
|---|---|---|---|
| Synced Shared Mailbox does not appear in user's Outlook. | Auto-mapping is delayed or the user lacks "Read and Manage" permission. | Verify permissions in admin center; manually add the shared mailbox account in Outlook settings. | N/A |
| "License shortage" error when creating a user. | All purchased licenses in the tenant are currently allocated to active users. | Purchase more licenses, or unassign licenses from disabled/inactive user profiles. | Navigate to Billing -> Licenses |
| User cannot send emails from Shared Mailbox: "SendAsDenied." | The user has "Read and Manage" access but lacks the "Send As" permission. | Assign the "Send As" permission in the Shared Mailbox settings under members. | `Add-RecipientPermission` |
| Email bounces back with: "Remote Server returned '550 5.7.501 Access denied'." | The user's account has been compromised and blocked by Microsoft for sending spam. | Unblock the user in the Microsoft Defender Portal under Email & Collaboration Security. | security.microsoft.com |

---
## Interview Questions

> [!question] L1 Question
> **Q:** Do Shared Mailboxes require a paid Microsoft 365 license?
> **A:** No, **Shared Mailboxes** do not require a separate paid license as long as their total storage stays under **50 GB**. If the mailbox exceeds 50 GB, an Exchange Online Plan 2 or Microsoft 365 E3/E5 license must be assigned to keep it active.

> [!question] L2 Question
> **Q:** What is the difference between "Send As" and "Send on Behalf" permissions on an Exchange mailbox?
> **A:** **Send As** allows the delegate to send an email appearing exactly as if it came from the mailbox owner (the recipient only sees `helpdesk@mybrand.com`). **Send on Behalf** allows the delegate to send mail, but the recipient sees: `Ashwani Singh on behalf of helpdesk@mybrand.com`.

> [!question] L3/Scenario Question
> **Q:** Your company wants to implement mobile application management (MAM) to ensure that users cannot copy corporate email data from Outlook to personal applications (like WhatsApp or Notes) on their mobile phones. Which Microsoft 365 tool and feature should you use?
> **A:** 
> - **Situation:** Restricting data exfiltration from corporate apps to personal apps on mobile devices.
> - **Task:** Define the MAM solution within Microsoft 365.
> - **Action:** 
>   1. **Select Tool**: Use **Microsoft Intune** (included in E3/E5 and Business Premium licenses).
>   2. **Define Feature**: Create an **App Protection Policy (MAM)**.
>   3. **Configure Restricting Rules**: In the policy settings, set "Restrict cut, copy, and paste with other apps" to **Blocked** or **Policy managed apps only**. Configure "Prevent Save As" to restrict local saving of attachments to OneDrive for Business only.
>   4. **Deploy**: Apply the policy to target user groups across iOS and Android devices.
> - **Result:** Corporate data remains encrypted within the managed Outlook sandbox, preventing data leakage to personal apps.

---
## Seedha Simple Mein
*Seedha simple mein: MS-900 exam Microsoft 365 cloud products ke services aur licensing structures ka check-point hai. Isme hum seekhte hain ki Exchange mailboxes, OneDrive personal data, aur SharePoint group sites ko kaise administer kiya jata hai. Shared mailboxes without license free space dete hain (upto 50GB) and teams setup me client communication ke liye best hote hain.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Admin center procedures.
- [[04-Cloud-and-Security/07-Microsoft-365/OneDrive-Administration|OneDrive-Administration]] — User storage management.
- [[06-Career-Growth/14-Certifications/AZ-900-Roadmap|AZ-900-Roadmap]] — Azure Fundamentals certification comparison.

---
*Tags: #desktop-support #career #certification #m365 #L1*
