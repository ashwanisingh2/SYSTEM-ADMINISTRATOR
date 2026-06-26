---
tags: [desktop-support, m365, collaboration, L1]
aliases: [m365-01-microsoft-365-administration, m365-01]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **CLOUD & MICROSOFT 365**

`#complete` `#intermediate` `#none`

# M365-01: Microsoft 365 Administration

> [!abstract] Overview
> Yeh note Microsoft 365 tenant administration par focus karta hai. Isme licensing plans, admin center scopes, custom domain configuration (DNS requirements), identity management aur RBAC roles cover hain.

---
## 🧠 Concept Overview

- **What it is** — Microsoft 365 ka admin center jahan se company ke saare cloud tools (Exchange, Teams, SharePoint) control hote hain.
- **Why it matters** — Foundation hai kisi bhi modern company ki IT ka. Yahan se users, licenses aur security handle hoti hai.
- **Where you see this** — Jab company ke naam ka custom domain (company.com) setup karna ho ya naye employees ke accounts banane hon.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Users create karna, passwords reset karna, aur licenses assign karna M365 admin portal se. |
| **L2** | Custom domains add karna, DNS records (MX, TXT) verify karna, aur basic RBAC roles assign karna. |
| **L3** | Tenant architecture, global security settings, aur hybrid identity sync (Entra Connect) manage karna. |

> [!tip] Seedha Simple Mein
> *M365 tenant setup karne ke liye hum custom domain add karte hain aur DNS records (MX, SPF, DKIM) verify karte hain. Fir admin centers se services aur RBAC roles control karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **M365 Tenant** is like **renting a corporate office building** because...
>
> - **Licensing Plans**: Desk rental packages (kuch me sirf phone hai, kuch me poora cabin).
> - **Custom Domain Setup**: Building ke bahar apni company ka board aur logo lagana.
> - **Global Administrator**: Building ka owner jiske paas saari chabiyan (Master keys) hain.

---
## 🔬 Technical Deep Dive

### 1. Custom Domain Setup: DNS Requirements

> [!info] Key Concept
> Custom domain use karne ke liye aapko prove karna hota hai ki aap domain ke owner hain aur mail routing configure karni hoti hai.

Public DNS mein records add hote hain:
- **TXT Record**: Verification (Proves ownership like MS=ms12345).
- **MX Record**: Mail routing (Mails M365 servers pe aayenge).
- **CNAME Record**: Autodiscover (Outlook auto-configure karega).
- **TXT Record (SPF)**: Spoofing se bachane ke liye (Authorized sender M365 hai).

### 2. Admin Roles (Least Privilege)

- **Global Administrator**: Full access (MFA compulsory, sirf 2-4 accounts hone chahiye).
- **User Administrator**: Users aur passwords manage karta hai.
- **Exchange Administrator**: Mailboxes aur mail flow rules manage karta hai.

> [!danger] Common Mistake
> Har helpdesk engineer ko 'Global Administrator' role de dena. Hamesha Least Privilege ka rule follow karein aur specifically 'User Administrator' ya 'Helpdesk Admin' dein.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - M365 Admin Center access
> - Access to public DNS registrar (e.g. GoDaddy, Cloudflare)

### Step 1: Add a Custom Domain

```bash
# General network ping check for domain
ping company.com
```

> [!success] Expected Output
> ```
> Reply from [IP Address]: bytes=32 time=20ms TTL=115
> ```

1. M365 Admin Center (`admin.microsoft.com`) mein login karein.
2. `Settings` -> `Domains` -> `+ Add domain` pe jaayein.
3. Domain name dalein (e.g., company.com).
4. `Add a TXT record to the domain's DNS records` select karein aur MS=... value copy karein.
5. Apne DNS provider me TXT record daalein, 5 min wait karein aur Verify click karein.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Resolve-DnsName -Type TXT company.com` | PowerShell mein TXT records check karta hai (SPF/Verification verify karne ke liye) | `Resolve-DnsName -Type TXT google.com` |
| `nslookup -type=mx company.com` | CMD mein MX records check karta hai | `nslookup -type=mx company.com` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Domain verification failing in M365 portal | DNS propagation issue ya TXT record galat jaga add kiya hai | 1-2 hours wait karein (TTL) aur `nslookup` / `mxtoolbox.com` se manually verify karein. |
| Inbound emails failing after domain setup | MX record add nahi hua ya incorrect pointing hai | Ensure MX record `company-com.mail.protection.outlook.com` pe point kar raha ho with Priority 10. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: New Domain Setup

> [!example] Ticket
> "We just bought a new subsidiary company. Can you add their domain `newbrand.com` to our M365 tenant so they can receive emails?"

**L1 Response:** Domain ownership details (GoDaddy/Registrar login) arrange karna.
**Escalation Trigger:** Agar DNS settings L2 team karti hai network changes ke policy ke mutabik.
**L2 Resolution:** M365 admin center mein domain add karna, TXT se verify karna, aur phir MX, CNAME, SPF records public DNS mein update karna.

---
## 🎤 Interview Questions

> [!question] Q1: Explain the purpose of the MX and SPF records in Microsoft 365 domain setup.
> **Answer:** MX (Mail Exchanger) record batata hai ki incoming emails ko kis server pe bhejna hai (Exchange Online Protection). SPF (Sender Policy Framework) ek TXT record hai jo define karta hai ki kaunse IP/servers domain ke naam se email bhej sakte hain, taaki spam filters use reject ya flag na karein (spoofing protection).

==**Exam Tip:** Global Admin role should be heavily restricted and protected by Conditional Access / MFA. Daily tasks should be done using scoped roles like User Administrator.==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — DNS records configurations.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-02 Exchange Online Administration|M365-02 Exchange Online Administration]] — Advanced mailbox provisioning.
- [[04-Cloud-and-Security/07-Microsoft-365/M365-05 Security and Compliance|M365-05 Security and Compliance]] — Conditional Access and MFA security.
