---
tags: [desktop-support, m365, collaboration, L1]
aliases: [m365-01-microsoft-365-administration, m365-01]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# M365-01: Microsoft 365 Administration

> [!abstract] Overview
> This note covers Microsoft 365 tenant administration. It details subscription licensing plans, admin center scopes, custom domain configuration (DNS requirements), identity management, RBAC administrator roles, and auditing.

---

---
## Concept Overview
Think of a Microsoft 365 Tenant as renting a complete virtual corporate office building in the Microsoft cloud:
- **Licensing Plans** are the desk rental packages: you can rent a basic desk with a phone (Business Basic), a standard desk with physical office tools (Business Standard), or a high-security executive suite with armored files and encryption keys (Business Premium/E5).
- **Custom Domain Setup** is putting your company logo and signpost on the front of the building (DNS records pointing mail and verification to your tenant).
- **Global Administrator** is the building owner holding the master key ring.
- **Service Health** is the dashboard showing if the city water main or power line (Microsoft servers) has an outage.


---

---
## Technical Deep Dive
### 1. Microsoft 365 Licensing Plans
M365 plans are categorized into Business (under 300 users) and Enterprise (unlimited users):

- **Business Basic:** Web and mobile Office apps only, Exchange Email, Teams, SharePoint, and 1TB OneDrive.
- **Business Standard:** Adds downloadable desktop Office applications (Outlook, Word, Excel).
- **Business Premium:** Adds advanced security controls: **Microsoft Intune** (device management) and **Azure Information Protection** (AIP).
- **Enterprise E3:** Advanced security, data loss prevention (DLP), and larger mailbox limits (100GB).
- **Enterprise E5:** Premium security, identity protection (Entra ID P2), Advanced Threat Protection (ATP), Power BI, and Teams phone capabilities.

### 2. Admin Center Directory
Administrators manage M365 services via dedicated web portals:
- **Microsoft 365 Admin Center (`admin.microsoft.com`):** User management, licensing, billing, domains, and global settings.
- **Exchange Admin Center (`admin.exchange.microsoft.com`):** Mailbox routing, flow rules, spam filtering.
- **SharePoint Admin Center (`admin.sharepoint.com`):** Site collections, storage limits, sharing policies.
- **Teams Admin Center (`admin.teams.microsoft.com`):** Calling policies, meeting configurations, device management.

### 3. Custom Domain Setup: DNS Requirements
To send and receive emails using a custom domain (e.g., `user@company.com`) instead of the default `company.onmicrosoft.com`, you must add your domain to the M365 tenant and verify ownership by adding the following DNS records at your public DNS registrar:

- **TXT Record (Verification):**
  - *Value:* `MS=msXXXXXXXX` (Proves ownership to Microsoft).
- **MX (Mail Exchanger) Record (Mail routing):**
  - *Host:* `@` | *Points to:* `company-com.mail.protection.outlook.com` | *Priority:* `10`
- **CNAME Record (Autodiscover):**
  - *Host:* `autodiscover` | *Points to:* `autodiscover.outlook.com` (Ensures Outlook auto-configures client profiles).
- **TXT Record (SPF - Sender Policy Framework):**
  - *Value:* `v=spf1 include:spf.protection.outlook.com -all` (Prevents spoofing by designating M365 as the only authorized sender for the domain).

### 4. Admin Roles (Least Privilege)
- **Global Administrator:** Full access to all administrative features. Limit this to 2 to 4 accounts per tenant, requiring MFA.
- **User Administrator:** Can create, edit, delete users, reset passwords for non-admin users, and manage licenses.
- **Exchange Administrator:** Full control over mailboxes, mail flow rules, and spam filtering.
- **SharePoint Administrator:** Manage site collections, file storage, and OneDrive policies.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Access to a new Microsoft 365 Tenant (or a free developer account from `developer.microsoft.com/en-us/microsoft-365/dev-program`).

### Step 1: Add a Custom Domain
1. Log into the **Microsoft 365 Admin Center** (`admin.microsoft.com`).
2. Go to **Settings** -> **Domains**. Click **+ Add domain**.
3. Domain name: `company.com` (or your registered test domain). Click **Use this domain**.
4. Verification: Select **Add a TXT record to the domain's DNS records**. Click Next.
5. Copy the TXT value (e.g., `MS=ms12345678`).
6. Log into your public DNS provider console (GoDaddy, Cloudflare, etc.).
7. Add the TXT record. Wait 5 minutes.
8. Go back to M365 Admin Center, click **Verify**.

### Step 2: Configure Exchange DNS Records
1. Once verified, M365 prompts: **How do you want to connect to your domain?** Select **Add your own DNS records**. Click Next.
2. The page lists the required MX, CNAME, and SPF records.
3. Add these records to your public DNS provider.
4. Click **Connect** in the M365 portal. The status should change to **Healthy**.

### Step 3: Create User and Assign License
1. In the M365 Admin Center, go to **Users** -> **Active users**. Click **Add user**.
2. Configurations:
   - First Name: `John` | Last Name: `Doe`.
   - Display Name: `John Doe`.
   - Username: `jdoe` | Domain: Select `company.com`.
3. Licenses: Select **Microsoft 365 Business Premium** (or your active trial license).
4. Roles: Leave as **User (no admin center access)**.
5. Click **Finish adding**.

---

---
## Cheat Sheet / Quick Reference
| Command / Configuration | Scope | Purpose / Example |
|---|---|---|
| `systemctl status <service>` | Linux | Check status of system service |
| `ip address show` | Linux | Display local interface network details |
| `Get-Service` | PowerShell | Verify service status on Windows hosts |
| `Test-NetConnection` | PowerShell | Check network path connectivity to target ports |

---
## Troubleshooting
| Problem | Cause | Fix | Command |
|---|---|---|---|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall | `ping -c 4 <ip>` / `nc -zv <ip> <port>` |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password | N/A |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects | N/A |

---
## Interview Questions
> [!question] L1 Question
> **Q:** How do you verify if the target service is running?
> **A:** On Linux, I would execute `systemctl status <service-name>`. On Windows, I would run `Get-Service <service-name>` in PowerShell or check Services.msc.

> [!question] L2 Question
> **Q:** Explain how you would troubleshoot a network connectivity issue to a remote server.
> **A:** I would verify local IP configuration, test routing gateway using `ping`, trace hops using `traceroute` or `tracert`, and check port accessibility using `telnet` or `Test-NetConnection` on target port.

---
## Seedha Simple Mein
*Seedha simple mein: M365 tenant setup karne ke liye hum custom domain add karte hain aur DNS records (MX, SPF, DKIM) verify karte hain. Licensing assignment and RBAC roles (Global Admin, User Admin) administrative boundaries ko regulate karte hain.*

---
## Related Notes
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — DNS records configurations.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-02 Exchange Online Administration|M365-02 Exchange Online Administration]] — Advanced mailbox provisioning.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-05 Security and Compliance|M365-05 Security and Compliance]] — Conditional Access and MFA security.
