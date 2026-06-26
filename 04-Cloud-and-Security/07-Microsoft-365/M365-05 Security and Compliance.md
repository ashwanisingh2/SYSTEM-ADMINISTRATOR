---
tags: [desktop-support, m365, collaboration, L1]
aliases: [m365-05-security-and-compliance, m365-05]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# M365-05: Security and Compliance

> [!abstract] Overview
> This note details Microsoft 365 security and compliance tools, focusing on Defender for Office 365, Data Loss Prevention (DLP), Information Protection (Sensitivity Labels), eDiscovery, and identity protection via Conditional Access.

---

---
## Concept Overview
Think of Microsoft 365 Security and Compliance as a high-security corporate facility. **Conditional Access** acts as the security guards checking IDs and health passes at the gate. **Defender for Office 365** behaves like the automated scan tunnel checking mail packages for explosives (Safe Attachments) and verifying sender badges (Anti-Phishing). **DLP** functions as guards monitoring the exits to make sure staff don't walk out with confidential blueprints, while **Sensitivity Labels** stamp physical documents with security levels like "SECRET" that restrict who can open them even after they leave the building.

---

---
## Technical Deep Dive
### 1. Microsoft Defender for Office 365
Protects email, files, and collaboration channels (Teams, SharePoint, OneDrive) from cyber threats:
- **Plan 1**: Focuses on core prevention. Includes Safe Attachments (detonating attachments in sandboxes), Safe Links (rewriting URLs to scan them at time-of-click), and Anti-Phishing protection (validating SPF/DKIM/DMARC and scanning for impersonation).
- **Plan 2**: Extends Plan 1 to include active post-breach investigation and response. Includes Threat Tracker, Threat Explorer (interactive reporting tools), Automated Investigation and Response (AIR), and Attack Simulation training (phishing tests).

### 2. Data Loss Prevention (DLP)
- Identifies, monitors, and protects sensitive information (like SSNs, Credit Cards, or custom keywords) across Exchange, Teams, SharePoint, and endpoints.
- Works by scanning for policy matches (e.g., detecting credit card patterns), displaying user policy tips to educate users, and blocking the sharing action if threshold rules are breached.

### 3. Microsoft Purview Information Protection (Sensitivity Labels)
- Allows classification and encryption of documents, emails, and sites.
- **Publishing**: Created in Microsoft Purview compliance portal and deployed via label policies.
- **Rights Management (RMS)**: Stays embedded inside the file metadata. If a labeled file is copied to a USB drive or emailed outside the company, only users authorized by the embedded RMS policy can decrypt and read it.

### 4. Retention Policies vs. Retention Labels
- **Retention Policies**: Applied broadly (e.g., tenant-wide or site-wide) to keep data for a set duration, then delete it. Users cannot bypass these settings.
- **Retention Labels**: Applied granularly at the folder or file level. Can be manually selected by users or auto-applied based on content analysis (e.g., recognizing financial reports).

### 5. Identity Protection: Conditional Access (CA)
- Evaluates signals (user location, device health, application accessed, risk levels) before granting access.
- **Per-User MFA**: Static, binary control. A user either always requires MFA or never does. Obsolete.
- **Conditional Access (Modern)**: Dynamic. Requires MFA only when signals indicate a need (e.g., accessing from an unfamiliar IP address, unmanaged device, or during a high-risk logon event).

---

## Common Mistakes
> [!warning] Avoid These
> Enabling Conditional Access policies targeting "All Users" to require MFA without excluding a "Break-Glass" (Emergency Access) account. If the Azure identity service encounters a configuration loop, you risk locking yourself out of the tenant permanently.
> Creating sensitivity labels and choosing to encrypt files without training users first. This results in users locking themselves out of their own documents, or sharing encrypted attachments that external vendors cannot open.
> Overlapping broad Retention Policies with granular Retention Labels. The longest retention period and the most restrictive deletion rules always win, causing confusion when files fail to purge.

---

## Pro Tips
> [!tip] Field Experience
> Always implement a Report-Only period (using `State = reportOnly`) for at least 14 days before enforcing any new Conditional Access policy. Use the Entra Sign-in logs to review exactly which users will be impacted by the changes before they go live.
> Enable **DKIM** and **DMARC** alongside SPF for outbound mail flow. Without these, your outbound emails will fail phishing evaluation filters on target domains (like Gmail or Yahoo) and land in the recipient's spam folder.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Microsoft 365 Tenant with Global Administrator or Compliance/Security Administrator roles. Access to the Microsoft Purview portal.

### Step 1: Create a Data Loss Prevention (DLP) Policy
Create a DLP policy to block external sharing of credit card numbers:
1. Navigate to the **Microsoft Purview Portal** (`https://purview.microsoft.com`) -> `Solutions` -> `Data Loss Prevention` -> `Policies`.
2. Click **Create policy** -> Choose **Custom** category -> Name it `Credit Card Block Policy`.
3. Choose Locations: Check **Exchange email**, **SharePoint sites**, **OneDrive accounts**, and **Teams chat and channel messages**.
4. Define Rules:
   - Condition: **Content contains** -> **Sensitive info types** -> Add **Credit Card Number**.
   - Condition: **Content is shared from Microsoft 365** -> **with people outside my organization**.
   - Action: **Restrict access or block the content** -> Choose **Block only people outside the organization**.
   - User notification: **Turn on notifications and show policy tips**.
5. Save the policy in **Test mode** first, then click **Submit** to publish.

### Step 2: Create and Publish a Retention Label
Configure a label to retain tax documents for 7 years:
1. Under **Microsoft Purview** -> `Solutions` -> `Data Lifecycle Management` -> `Microsoft 365` -> `Labels`.
2. Click **Create a label** -> Name: `Tax Records 7 Years`.
3. Label Settings: Select **Retain items forever or for a specific period** -> Duration: **7 years**.
4. Choose action after duration: **Delete items automatically**.
5. Go to **Label Policies** -> Click **Publish labels** -> Select the `Tax Records 7 Years` label.
6. Target Locations: Choose **SharePoint** and **OneDrive**.
7. Name the policy `Tax Retention Publish Policy` and submit it.

### Step 3: Create a Conditional Access Policy via PowerShell
Deploy a CA policy requiring MFA for all admin accounts using Microsoft Graph PowerShell:
```powershell
# Install and Connect using Microsoft Graph
Install-Module Microsoft.Graph -Force
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess", "Application.Read.All"

# Define Policy Parameters
$policy = @{
    DisplayName = "Require MFA for Directory Admins"
    State = "enabledForReportingButNotEnforced" # Always deploy in Report-Only first
    Conditions = @{
        Users = @{
            IncludeRoles = @(
                "62e90394-69f5-4237-9190-012177145e10", # Global Administrator
                "74946d75-4ed3-468a-a10e-ceceecff9f63"  # Security Administrator
            )
        }
        Applications = @{
            IncludeApplications = @("All")
        }
    }
    GrantControls = @{
        Operator = "OR"
        BuiltInControls = @("mfa")
    }
}

# Create the Policy
New-MgIdentityConditionalAccessPolicy -BodyParameter $policy
```

---

---
## Cheat Sheet / Quick Reference
```powershell
Get-DlpCompliancePolicy                             # Retrieves all active DLP policies
Get-RetentionCompliancePolicy                        # Lists data retention policies
Start-HistoricalSearch -ReportTitle "eDiscovery1"   # Triggers background search for audit logs
Get-MgIdentityConditionalAccessPolicy               # Lists Entra Conditional Access policies
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Safe Links | Rewrites and scans URLs at time-of-click to block malicious targets. |
| 2 | DLP | Scans data formats to prevent unauthorized exfiltration of sensitive information. |
| 3 | Sensitivity Label | Classification tag that embeds encryption and access permissions inside files. |
| 4 | eDiscovery | Discovery tool to search, hold, and export data across Microsoft 365 locations. |
| 5 | Conditional Access | Rules engine evaluating logon signals to grant or deny system access. |

---

---
## Troubleshooting
**Scenario 1:**
- Problem: Users complain they cannot open legitimate invoices sent by customers because the links inside the email are replaced with long `nam06.safelinks.protection.outlook.com` URLs, and clicking them redirects to a Microsoft warning page.
- Root Cause: Defender for Office 365 Safe Links has flagged the destination domain due to bad sender reputation, or it is a false-positive detection.
- Fix:
  1. Go to the **Microsoft Defender Portal** (`https://security.microsoft.com`) -> `Policies & rules` -> `Threat Policies` -> `Tenant Allow/Block Lists`.
  2. Search for the domain or URL under the `URLs` tab.
  3. Click **Add** -> Add the URL, set action to **Allow**, and define an expiration time (e.g., 30 days) to allow temporary bypass.
  4. Submit the URL to Microsoft for analysis to clear its bad reputation score permanently.

**Scenario 2:**
- Problem: The eDiscovery administrator ran a Content Search for a critical legal investigation but the search results came up empty, even though the employee confirmed sending emails containing the search string.
- Root Cause: The search query used incorrect KQL (Keyword Query Language) syntax, or the mailbox was not included in the search scope.
- Fix:
  1. Navigate to **Microsoft Purview** -> `eDiscovery` -> `Standard` -> Select the Case -> `Searches`.
  2. Edit the search properties and double-check the **Locations** scope. Ensure the user's specific Exchange Mailbox and OneDrive accounts are checked.
  3. Correct the search query. For example, search for `subject:"Project Phoenix"` instead of just `Phoenix` to narrow the results.
  4. Rerun the search and monitor progress under the `Status` column until it reads `Completed`.

---

---
## Interview Questions
**Q1: Explain the difference between SPF, DKIM, and DMARC. How do they prevent email spoofing?**
A: SPF (Sender Policy Framework) is a DNS TXT record listing all IP addresses authorized to send emails on behalf of your domain. DKIM (DomainKeys Identified Mail) adds a cryptographic signature to the email header, validating that the mail was sent by the domain owner and not altered in transit. DMARC (Domain-based Message Authentication, Reporting, and Conformance) uses both SPF and DKIM checks to instruct the receiving server on what to do with failures (e.g., none, quarantine, or reject).

**Q2: A company is hit by a ransomware attack. Some SharePoint sites have encrypted files. How does M365 protect against this?**
A:
- **Situation**: A ransomware script accessed a user's sync folder and encrypted 50,000 files in a SharePoint document library.
- **Task**: I needed to restore the document library to its pre-encrypted state with minimal downtime and zero data loss.
- **Action**: I used SharePoint's **"Restore a library"** feature. I navigated to the document library settings, clicked "Restore this library", selected a point-in-time just before the ransomware encryption started, and monitored the restoration.
- **Result**: Because versioning is active by default (500+ versions), SharePoint rolled back all modified versions to the previous unencrypted version in under 20 minutes, neutralizing the ransom threat.

**Q3: How does Conditional Access evaluate a sign-in request when multiple policies are active?**
A: Conditional Access evaluates all applicable policies simultaneously. If a user matches the criteria of multiple policies, the controls of *all* matching policies must be satisfied. For example, if Policy A requires MFA, and Policy B requires a compliant device, the user must perform MFA *and* use a compliant device to successfully authenticate. If any policy results in a "Block" control, access is immediately blocked regardless of other policies.

---

---
## Seedha Simple Mein
*Seedha simple mein: M365 Security aur Compliance system identities (Conditional Access, MFA), data channels (Defender, Safe Links), aur document boundaries (DLP, Sensitivity Labels) ko protect karne ka ek unified framework hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-01 Microsoft 365 Administration|M365-01 Microsoft 365 Administration]] — Sets up tenant-wide user and billing scopes.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-02 Exchange Online Administration|M365-02 Exchange Online Administration]] — Manages mail flow routing and basic connectors.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-07 Endpoint Security in Intune|INT-07 Endpoint Security in Intune]] — Manages Windows Defender and compliance metrics.
