---
tags: [desktop-support, microsoft-365, email-security, spf-dkim-dmarc, L2]
aliases: [email-security-deep-dive, spf-dkim-dmarc-guide]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# M365-06: Email Security Deep Dive (SPF, DKIM, and DMARC)

> [!note] Overview
> This note covers the architecture and implementation of email authentication protocols: Sender Policy Framework (SPF), DomainKeys Identified Mail (DKIM), and Domain-based Message Authentication, Reporting, and Conformance (DMARC). It details DNS record formats, message header analysis, and policy migrations.

---
## Concept Overview
- **What it is** — SPF, DKIM, and DMARC are DNS-based email authentication standards. Together, they prevent email spoofing, phishing, and spam by verifying that the sending server is authorized by the domain owner and verifying the integrity of the email payload.
- **Why it matters** — Email spoofing allows malicious actors to impersonate corporate brands or execute business email compromise (BEC) attacks by sending messages that look like they originate from internal users. Implementing these protocols blocks fake emails from reaching customers or employees.
- **Real job encounter** — Configuring secure email delivery for a newly registered corporate domain, troubleshooting outbound emails getting rejected by external spam filters, and inspecting email headers to determine if SPF/DKIM checks passed.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Analyze email headers using message header analyzers (such as MXToolbox), verify basic DNS query records, and identify spam filtering classifications.
  - **Escalation Trigger**: Escalate if company emails are blocked globally due to SPF hard-fail policies or incorrect public DNS record lookups.
  - **L2 Resolution**: Author and update SPF TXT records, generate and publish DKIM keys in public DNS registries, enable DKIM signing in Exchange Online, and configure basic DMARC monitoring.
  - **L3 Resolution**: Implement strict DMARC enforcement policies (`p=reject`), analyze DMARC XML reports, manage email gateways, and resolve alignment errors with third-party bulk-mailing providers.

---
## Technical Deep Dive

### 1. Sender Policy Framework (SPF)
SPF allows domain owners to publish a list of IP addresses or subnets authorized to send email on behalf of their domain.
- **How it works**: The receiving mail server extracts the domain from the envelope sender (the `Return-Path` or `Mail From` address) and queries the public DNS of that domain for an SPF TXT record. If the sending IP matches one of the values, the check passes.
- **Limit**: SPF records **must not exceed 10 DNS lookups** (queries using `include`, `a`, `mx`, or `ptr`). Exceeding this limit causes an SPF PermError (Permanent Error), causing many spam filters to reject the mail.
- **Qualifiers**:
  - `+` (Pass): Authorizes the IP.
  - `-` (Fail / Hard Fail): Unlisted IPs are unauthorized and mail should be rejected.
  - `~` (Soft Fail): Unlisted IPs are suspicious; mail is flagged but allowed.
  - `?` (Neutral): No policy declared.

```
Example SPF Record:
v=spf1 ip4:192.168.1.10 include:spf.protection.outlook.com -all
  |       |                     |                          |
Version  IPv4 Address     Include M365 Subnet          Hard Fail (Reject others)
```

### 2. DomainKeys Identified Mail (DKIM)
DKIM uses public-key cryptography to add a digital signature to the headers of outgoing emails.
- **How it works**:
  1. The sending mail server signs key headers (like `From`, `To`, `Subject`) using a private key. The signature is inserted as a header (`DKIM-Signature`).
  2. The receiving server extracts the signature, reads the "selector" tag (`s=`) and domain tag (`d=`), and queries the domain's DNS for the public key (`selector._domainkey.domain.com`).
  3. It decrypts the signature using the public key. If the hash matches, the email's authenticity and integrity are verified, proving the email was not modified in transit.

### 3. DMARC (Message Alignment and Enforcement)
DMARC ties SPF and DKIM together. It introduces two main features:
- **Alignment**: DMARC requires that the domain in the visible `From:` header matches the domain used in the SPF check (envelope sender) and/or the domain validated by the DKIM signature.
- **Enforcement Policy**: Tells receiving servers what to do if an email fails both SPF and DKIM checks:
  - `p=none` (Monitoring): Deliver the mail normally; send audit reports to the domain owner.
  - `p=quarantine`: Send the failed emails to the recipient's Junk/Spam folder.
  - `p=reject`: Block the email completely at the gateway.

```
                  +-----------------------------------+
                  |      Incoming Mail Received       |
                  +-----------------------------------+
                                    |
                  +-----------------------------------+
                  |  Runs checks: SPF, DKIM, DMARC   |
                  +-----------------------------------+
                                    |
                  +-----------------------------------+
                  |     Do SPF or DKIM Pass and       |
                  |     Align with "From" Header?     |
                  +-----------------------------------+
                        /                       \
                     Yes                         No
                     /                             \
+-----------------------------------+     +-------------------------+
| Pass: Deliver to Inbox            |     | Look up DMARC Policy:   |
+-----------------------------------+     +-------------------------+
                                                /       |       \
                                           p=none  p=quar  p=reject
                                            /           |           \
                              +---------------+ +---------------+ +---------------+
                              | Deliver Inbox | | Deliver Spam  | | Block Mail    |
                              +---------------+ +---------------+ +---------------+
```

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A public domain registrar account (e.g., GoDaddy, Cloudflare) with DNS control.
> - A Microsoft 365 Tenant admin account.
> - Outbound Internet connectivity.

### Step 1: Verify Existing DNS Security Records via CLI
1. Open a command prompt and check the SPF record using `nslookup`:
```cmd
nslookup -type=txt corp.com
```
**Expected Output:** `corp.com text = "v=spf1 include:spf.protection.outlook.com -all"`

2. Query your DMARC record:
```cmd
nslookup -type=txt _dmarc.corp.com
```

---

### Step 2: Configure SPF for Microsoft 365
1. Log in to your DNS Hosting provider.
2. Edit your DNS zone file. Create a new TXT record:
   - Host/Name: `@`
   - Value: `v=spf1 include:spf.protection.outlook.com -all`
   - TTL: `3600` (1 hour)
3. Click **Save**.

### Step 3: Enable DKIM in Exchange Online
Before enabling, we must generate public keys and publish them to public DNS.

1. Log in to the **Microsoft Defender portal** (`security.microsoft.com`).
2. Navigate to **Email & collaboration** -> **Policies & rules** -> **Threat policies** -> **DKIM**.
3. Select your domain and click **Create DKIM keys**.
4. The system displays two CNAME records that must be published. Example:
   - CNAME 1: `selector1._domainkey.corp.com` pointing to `selector1-corp-com._domainkey.host.onmicrosoft.com`
   - CNAME 2: `selector2._domainkey.corp.com` pointing to `selector2-corp-com._domainkey.host.onmicrosoft.com`
5. Go to your public DNS control panel. Add both CNAME records.
6. Return to Microsoft Defender portal, select the domain, and toggle **Sign messages for this domain with DKIM signatures** to **Enabled**.

### Step 4: Publish DMARC Monitoring Record
1. In your DNS zone console, add a new TXT record:
   - Host/Name: `_dmarc` (creates `_dmarc.corp.com`)
   - Value: `v=DMARC1; p=none; pct=100; rua=mailto:dmarc-reports@corp.com;`
2. Save the record.
*Note: This starts monitoring (`p=none`) and sends daily XML compliance reports to `dmarc-reports@corp.com`. Once you verify all legitimate senders align, you should upgrade this policy to `p=quarantine` and eventually `p=reject`.*

---
## Cheat Sheet / Quick Reference

| Record Type | Hostname | Example Value | Purpose |
|---|---|---|---|
| **SPF (TXT)** | `@` | `v=spf1 include:spf.protection.outlook.com -all` | Authorizes email sending IPs |
| **DKIM (CNAME)**| `selector1._domainkey` | `selector1-domain-com._domainkey.host...` | Points to Microsoft public DKIM keys |
| **DMARC (TXT)** | `_dmarc` | `v=DMARC1; p=quarantine; pct=100; rua=mailto:log@domain.com` | Defines policy enforcement and report mailboxes |
| **SPF Qualifier** | `-all` | Hard Fail (Strict reject) | Rejects non-authorized servers |
| **SPF Qualifier** | `~all` | Soft Fail (Mark spam) | Flags non-authorized servers |
| **DMARC tag** | `p` | `p=none` / `p=quarantine` / `p=reject` | Enforcement actions |
| **DMARC tag** | `rua` | `rua=mailto:dmarc@corp.com` | Destination for aggregate XML reports |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Outbound emails fail with: "SPF PermError: Too many DNS lookups." | The domain's SPF record has exceeded the limit of 10 nested DNS queries. | Consolidate the SPF record. Remove unused `include:` statements or flatten nested IP ranges by replacing them with direct `ip4:` or `ip6:` ranges. |
| Emails fail DMARC checks, even though SPF is passing. | The sending application domain does not align with the visible `From:` header domain (SPF alignment failure). | Check email headers. Ensure the Return-Path address matches the domain in the `From:` header. If using third-party services (like Mailchimp), configure custom DKIM keys to achieve alignment. |
| Cannot enable DKIM in Exchange Online, returns error: "CNAME records not found." | Public DNS has not replicated the CNAME records yet, or the selector names contain typing errors. | Verify CNAME resolution using: `nslookup -type=cname selector1._domainkey.corp.com`. Ensure the selector name matches the exact spelling in the Exchange console. |
| Outlook displays warning: "This sender could not be verified" for incoming corporate emails. | External spoofing attempt, or internal application is spoofing domain without SPF registration. | Add the sending server's IP address to the SPF record. |
| DMARC reports show legitimate bulk mail (newsletters) failing checks. | Bulk mailing services do not have DKIM or SPF configurations mapped for the domain. | Access the configuration settings of the bulk mail service and configure a custom sending domain (DKIM key registration). |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What are SPF, DKIM, and DMARC used for?
> **A:** They are email authentication protocols published in public DNS records to prevent email spoofing, phishing, and spam. **SPF** lists the authorized IP addresses allowed to send mail for a domain. **DKIM** adds a digital signature to emails to prove the mail hasn't been altered. **DMARC** specifies how receiving servers should handle emails that fail SPF or DKIM checks (e.g., drop them or send them to spam).

> [!question] L2 Question
> **Q:** What is the difference between an SPF "-all" (Hard Fail) and an SPF "~all" (Soft Fail)?
> **A:** `-all` represents a **Hard Fail**. It tells receiving mail servers that if the sending IP address is not listed in the SPF record, the email should be blocked/rejected immediately. `~all` represents a **Soft Fail**. It tells receiving servers that unlisted IPs are suspicious and should not be blocked directly, but rather accepted and marked as spam or flagged for extra scrutiny.

> [!question] L3/Scenario Question
> **Q:** An enterprise has configured an SPF record with 12 DNS lookups. Outbound emails are being rejected by major providers with "SPF PermError". How do you resolve this without removing authorized servers?
> **A:** 
> - **Situation:** SPF record exceeds the 10 DNS lookup limit, causing delivery failures.
> - **Task:** Optimize the SPF record to bring the lookup count below 10.
> - **Action:** 
>   1. **Audit**: Trace all `include:`, `mx`, and `a` mechanisms. Identify which third-party senders are no longer used and delete them.
>   2. **Flattening**: Replace third-party `include:` lookups with their static public IP ranges (using `ip4:` or `ip6:` mechanisms) if the provider's IP ranges are stable. Static IP mechanisms do not count toward the 10 DNS lookup limit.
>   3. **Subdomain Delegation**: Delegate specific third-party applications (like marketing mailers) to send from a dedicated subdomain (e.g., `marketing.corp.com`) containing its own independent SPF record, isolating the lookup overhead.
> - **Result:** The primary domain's SPF record is reduced to under 10 lookups, resolving delivery errors.

---
## Seedha Simple Mein
*Seedha simple mein: Email security records (SPF, DKIM, DMARC) ye ensure karte hain ki aapke domain name ka use karke koi fraud email na bhej sake. SPF authorises IPs batata hai, DKIM email par digital signature lagata hai, aur DMARC bataata hai ki authentication fail hone par mail ko block (reject) karna hai ya spam folder me dalna hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-02 Exchange Online Administration|M365-02 Exchange Online Administration]] — Outbound and inbound mail routing flows.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Formatting public DNS records.
- [[04-Cloud-and-Security/09-Security/Incident-Response-Playbook|Incident-Response-Playbook]] — Mitigating phishing and compromised email attacks.

---
*Tags: #desktop-support #microsoft-365 #email-security #spf-dkim-dmarc #L2*
