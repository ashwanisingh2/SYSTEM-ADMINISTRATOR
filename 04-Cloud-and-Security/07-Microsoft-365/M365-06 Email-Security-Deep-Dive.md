---
tags: [desktop-support, microsoft-365, email-security, spf-dkim-dmarc, L2]
aliases: [email-security-deep-dive, spf-dkim-dmarc-guide]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#advanced` `#none`

# M365-06: Email Security Deep Dive (SPF, DKIM, and DMARC)

> [!abstract] Overview
> Yeh note email authentication protocols (SPF, DKIM, DMARC) ke baare mein hai. Yeh protocols spoofing aur phishing ko rokne mein help karte hain aur har L2 engineer ko iski samajh honi chahiye.

---
## 🧠 Concept Overview

- **What it is** — SPF, DKIM, and DMARC are DNS-based email authentication standards ensuring email authenticity and integrity.
- **Why it matters** — Spoofing attacks can impersonate domains. In protocols ke bina emails fake ho sakti hain ya spam mein ja sakti hain.
- **Where you see this** — Troubleshooting emails that bounce or go to spam folders, or configuring new domains.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Analyze headers using MXToolbox, check basic DNS records. |
| **L2** | Configure, fix, escalate — Create/update SPF TXT records, publish DKIM keys, manage simple DMARC policies. |
| **L3** | Architecture, design — Enforce strict DMARC (`p=reject`), analyze XML reports, resolve third-party alignment issues. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: Email security records (SPF, DKIM, DMARC) ye ensure karte hain ki aapke domain name ka use karke koi fraud email na bhej sake.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Email Authentication** is like **sending a secure package** because...
>
> - **SPF** is the authorized sender list checking if the delivery driver works for the company.
> - **DKIM** is the wax seal on the package proving it wasn't opened in transit.
> - **DMARC** is the instruction on what to do if the driver isn't on the list or the seal is broken.

---
## 🔬 Technical Deep Dive

### 1. Sender Policy Framework (SPF)

> [!info] Key Concept
> SPF allows domain owners to publish authorized sending IP addresses in a DNS TXT record.

- **Limit**: Must not exceed 10 DNS lookups.
- **Qualifiers**: `+` (Pass), `-` (Hard Fail), `~` (Soft Fail).

### 2. DomainKeys Identified Mail (DKIM)
Adds a digital signature to email headers using public-key cryptography. Receiver verifies via DNS public key.

### 3. DMARC (Message Alignment and Enforcement)
Requires domain alignment.
Policies:
- `p=none` (Monitoring)
- `p=quarantine` (Spam)
- `p=reject` (Block)

> [!danger] Common Mistake
> Exceeding the 10 DNS lookup limit in the SPF record causes an SPF PermError, rejecting legitimate mail.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Public domain registrar access
> - Microsoft 365 Tenant admin

### Step 1: Verify Existing DNS Security Records

```cmd
# Check SPF record
nslookup -type=txt corp.com
```

> [!success] Expected Output
> ```
> corp.com text = "v=spf1 include:spf.protection.outlook.com -all"
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `nslookup -type=txt domain.com` | Queries TXT records (SPF) | `nslookup -type=txt corp.com` |
| `nslookup -type=cname selector1._domainkey...` | Queries CNAME records (DKIM) | `nslookup -type=cname selector1._domainkey.corp.com` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| SPF PermError: Too many lookups | SPF record exceeded 10 nested lookups | Flatten IPs or remove unused `include:` statements |
| DKIM setup fails: "CNAME not found" | DNS propagation delay | Wait, or verify exact spelling of selector names |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Emails Going to Spam

> [!example] Ticket
> "Our marketing team is using Mailchimp and emails are going to customer's spam folders."

**L1 Response:** Check headers, verify if Mailchimp IPs are in SPF.
**Escalation Trigger:** Marketing tool requires custom DKIM keys.
**L2 Resolution:** Add Mailchimp's provided DKIM CNAME records to DNS and align domain for DMARC.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between an SPF "-all" and "~all"?
> **Answer:** `-all` is a Hard Fail (reject immediately). `~all` is a Soft Fail (flag as suspicious / spam but deliver).

==**Exam Tip:** DMARC enforces policy based on the alignment of the visible "From" header with either SPF or DKIM domains.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/M365-02 Exchange Online Administration|M365-02 Exchange Online Administration]] — Outbound routing
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — DNS records
