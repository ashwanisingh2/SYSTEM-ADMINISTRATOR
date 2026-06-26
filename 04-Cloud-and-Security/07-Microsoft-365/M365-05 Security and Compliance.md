---
tags: [m365, security, compliance, purview, defender, advanced, conditional-access, dlp]
aliases: [m365-security, m365-compliance, purview, defender-o365, m365-defender]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#advanced` `#none`

# M365-05: Security and Compliance

> [!abstract] Overview
> Microsoft 365 Security and Compliance (ab Microsoft Purview aur Microsoft Defender) cloud data aur users ko protect karne ka ecosystem hai. Ek system administrator ke liye yeh samajhna zaroori hai ki data leak kaise rokein, phishing emails se kaise bachein, aur company ke compliance standards (jaise GDPR, HIPAA) ko kaise meet karein. Yeh note M365 ke advanced security features aur compliance policies ko cover karta hai.

---
## 🧠 Concept Overview

- **What it is** — M365 Security & Compliance do alag par connected portals hain: **Microsoft Defender** (Threat protection) aur **Microsoft Purview** (Data governance & compliance).
- **Why it matters** — Data breaches aur cyber attacks se company ko bachane ke liye. *Agar koi employee galti se confidential client data bahar bhej de, toh company ko millions ka fine lag sakta hai. Yeh tools usko prevent karte hain.*
- **Where you see this** — Jab kisi user ko phishing email aati hai, ya jab koi sensitive credit card information email ke through bhejne ki koshish karta hai (DLP policy trigger hoti hai).

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check alerts, user ki blocked email check karna, MFA reset karna. |
| **L2** | Configure DLP policies, handle eDiscovery cases, analyze Safe Links/Attachments logs. |
| **L3** | Architecture, design enterprise security posture, configure complex Conditional Access, integrate SIEM/Sentinel. |

> [!tip] Seedha Simple Mein
> *Security matlab bahar ke hackers aur virus se bachana (Defender). Compliance matlab apne hi employees ko rules follow karwana aur company data leak hone se rokna (Purview).*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Airport Security System** is like **M365 Security & Compliance** because...
>
> - **Microsoft Defender (Threat Protection) is like the Security Scanner:** *Jaise scanner check karta hai ki aapke paas koi dangerous item (gun/knife) toh nahi hai, waise hi Defender check karta hai ki email mein koi malware ya phishing link toh nahi hai.*
> - **Microsoft Purview (Data Loss Prevention) is like Customs/Immigration:** *Jaise customs check karta hai ki aap country se koi illegal saaman bahar toh nahi le jaa rahe, waise hi Purview (DLP) check karta hai ki aap company se koi sensitive data (credit cards, PII) bahar toh nahi bhej rahe.*
> - **Conditional Access is like the Boarding Pass Check:** *Agar boarding pass aur ID match nahi hota, ya visa invalid hai, toh entry denied. Waise hi agar user unknown device ya location se aaye, toh MFA prompt hota hai ya access block hota hai.*

---
## 🔬 Technical Deep Dive

### 1. Microsoft Defender for Office 365 (MDO)

> [!info] Key Concept
> Defender for Office 365 email, links (URLs), aur collaboration tools (Teams, SharePoint) ko protect karta hai.

- **Safe Attachments:** Email ke attachments ko ek virtual environment (sandbox) mein open karke check karta hai. *Agar virus hua toh user tak nahi pahuchega.*
- **Safe Links:** Email ya Teams mein aane wale links ko scan karta hai. Agar link pe click karte time wo malicious mila, toh block screen aayegi.
- **Anti-Phishing & Anti-Spam:** Spoofing aur spam emails ko rokta hai aur quarantine mein daal deta hai.

> [!danger] Common Mistake
> Admin quarantine notifications ko on karna bhool jaate hain. *User wait karta rehta hai email ka aur usko pata hi nahi hota ki mail quarantine mein phasi hai.*

### 2. Microsoft Purview (Compliance)

> [!info] Key Concept
> Data ki lifecycle ko manage karna, usko protect karna aur legal rules follow karna Purview ka kaam hai.

- **Data Loss Prevention (DLP):** Policies jo sensitive info (jaise Aadhar card, SSN, Credit card) ko bahar bhejne se rokti hain. Aap policy bana sakte ho: *Agar email mein 3 se zyada credit card numbers hain, toh mail send hi nahi hogi.*
- **Retention Policies:** Data ko kitne time tak save rakhna hai. *Example: Company policy kehta hai ki sabhi emails 7 saal tak delete nahi honi chahiye, chahe user apne mailbox se delete kar de.*
- **eDiscovery:** Legal matters ke liye emails ya chats ko search aur export karna. *Agar koi legal case hota hai, toh lawyers bolenge "Mr. X ki pichle 2 saal ki saari mails nikalo jisme 'Project Alpha' word use hua ho". eDiscovery yahi kaam karta hai.*

### 3. Conditional Access (Entra ID / Azure AD)

> [!info] Key Concept
> If-Then logic for logins. "If user is from outside India, require MFA".

- **Signals:** User, Location, Device, Application, Real-time risk.
- **Decisions:** Block access, Grant access, Require MFA, Require compliant device.

> [!warning] Pre-requisites / Caution
> Conditional access policies banate time 'Exclude' list mein at least ek 'Break-glass' (Emergency Access) global admin account zaroor rakhein. *Warna agar policy galat ban gayi, toh aap khud hi poore tenant se lock out ho jaoge!*

### 4. Secure Score & Compliance Score

- **Secure Score:** Ek metric (percentage) jo batata hai aapka M365 tenant kitna secure hai. Microsoft recommendations deta hai (e.g., Turn on MFA for admins) jisse score badhta hai.
- **Compliance Score:** Same concept, but for regulatory compliance (jaise HIPAA, GDPR ke rules follow ho rahe hain ya nahi).

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Global Admin or Security Admin role in M365.
> - E5 license or Microsoft Defender for Office 365 Plan 1/2 license.

### Step 1: Create a Safe Links Policy

1. Go to **Microsoft Defender Portal** (security.microsoft.com).
2. Navigate to **Email & collaboration** -> **Policies & rules** -> **Threat policies**.
3. Click on **Safe Links** under Policies.
4. Click **+ Create**.
5. **Name your policy:** `Global-SafeLinks-Policy`. Click Next.
6. **Users and domains:** Select your domain (e.g., `contoso.com`). Click Next.
7. **Protection settings:**
   - Turn ON "On for unrecognised potentially malicious URLs".
   - Turn ON "Apply Safe Links to messages sent within the organization".
   - Turn ON "Do not let users click through to original URL" (Very important!).
8. Click Next, review, and Submit.

> [!success] Expected Output
> Ab se koi bhi user email mein aayi hui link pe click karega, toh backend mein pehle Microsoft us link ko verify karega ki wo safe hai ya nahi.

### Step 2: Create a Data Loss Prevention (DLP) Policy

1. Go to **Microsoft Purview Portal** (compliance.microsoft.com).
2. Go to **Data loss prevention** -> **Policies** -> **+ Create policy**.
3. Select **Financial** -> **U.S. Financial Data** (or custom for Indian PAN/Aadhar).
4. Name the policy `Block-Credit-Card-Data`.
5. Select locations: Exchange email, SharePoint sites, OneDrive accounts.
6. Set Rule: If content contains "Credit Card Number", Action: "Restrict access or encrypt the content".
7. Turn policy on or Test it out first.

### Step 3: Require MFA for External Access (Conditional Access)

1. Go to **Entra ID Admin Center** (entra.microsoft.com).
2. Go to **Protection** -> **Conditional Access** -> **Policies** -> **New policy**.
3. Name: `Require MFA for All Users`.
4. Users: Include **All users**. Exclude **Emergency Access Admin**.
5. Target resources: Include **All cloud apps**.
6. Conditions: *No specific condition, applies anywhere.*
7. Grant: **Require multifactor authentication**.
8. Set state to **On** and Save.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Connect-ExchangeOnline` | Exchange/Security modules connect karta hai | `Connect-ExchangeOnline -UserPrincipalName admin@domain.com` |
| `Get-AntiPhishPolicy` | Tenant ki anti-phishing policies list karta hai | `Get-AntiPhishPolicy` |
| `Get-SafeLinksPolicy` | Safe links rules show karta hai | `Get-SafeLinksPolicy -Identity "Global"` |
| `Get-DlpCompliancePolicy` | Purview DLP policies ki list deta hai | `Get-DlpCompliancePolicy` |
| `Search-UnifiedAuditLog` | M365 mein kya kya activity hui, uska log nikalta hai | `Search-UnifiedAuditLog -RecordType ExchangeAdmin -StartDate 01/01/2026 -EndDate 01/10/2026` |
| `Get-RetentionPolicy` | Org-wide retention policies fetch karta hai | `Get-RetentionPolicy` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Legit emails going to Junk/Quarantine** | Strict anti-spam policy or bad sender reputation. | *Defender portal mein Threat Explorer use karke check karein email kis policy se block hui. Sender ko Tenant Allow/Block list mein add karein.* |
| **User unable to send email (Blocked)** | User ka account compromise ho gaya aur wo spam bhej raha tha. | *Restricted Users page (Defender) mein jaakar user ko unblock karein, uska password reset karein aur MFA enforce karein.* |
| **DLP policy not triggering** | Policy abhi sync nahi hui hai ya condition match nahi ho rahi. | *M365 policies ko apply hone mein 1 se 24 ghante lag sakte hain. Sync time ka wait karein.* |
| **Cannot find old deleted email** | Retention policy nahi thi, ya purge ho gayi. | *eDiscovery chala kar recover karne ki koshish karein. Agar 30 din (default) se upar ho gaye bina litigation hold ke, toh data permanently lost.* |
| **MFA prompt not appearing** | CA Policy user pe apply nahi ho rahi ya excluded hai. | *Entra ID mein 'What If' tool use karein test karne ke liye ki us user pe policy trigger kyun nahi hui.* |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Blocked from Sending Emails

> [!example] Ticket
> "Hi IT, main email bhej nahi pa raha hu. Mujhe error aa raha hai 'Your message couldn't be delivered because you weren't recognized as a valid sender'."

**L1 Response:** Check the "Restricted entities" page in Defender. *User spam bhej raha tha isliye Microsoft ne block kar diya.*
**Escalation Trigger:** Agar account baar baar compromise ho raha hai.
**L2 Resolution:**
1. Block user sign-in immediately.
2. Force password reset.
3. Check email forwarding rules (hackers set karte hain chupke se mails padhne ke liye).
4. Remove user from restricted users list.
5. Re-enable sign-in.

### 🎫 Scenario 2: Phishing Link Clicked by Employee

> [!example] Ticket
> "I clicked on an email link saying 'Salary Update 2026' and entered my password, but nothing happened. Was it a scam?"

**L1 Response:** Immediately reset the user's password and clear all active sessions from Entra ID. *User ne credential de diye hain.*
**Escalation Trigger:** Agar user finance ya executive department ka hai (High Risk).
**L2 Resolution:**
1. Run a Message Trace to see who else received the "Salary Update 2026" email.
2. Use Defender Threat Explorer to permanently delete/purge those emails from all other users' mailboxes (ZAP - Zero-hour Auto Purge).
3. Check Azure AD sign-in logs to see if attackers have already logged in from a foreign country.
4. Block the sender domain in Tenant Allow/Block lists.

### 🎫 Scenario 3: eDiscovery Request for Leaning Employee

> [!example] Ticket
> From HR: "Mr. Sharma is resigning tomorrow. We suspect he is sharing client data with competitors. Please pull all his emails sent to external domains in the last 30 days."

**L1 Response:** Route ticket to Security/Compliance team. L1 typically does not have eDiscovery Manager roles.
**Escalation Trigger:** Immediate routing to L2/L3 Compliance Admin.
**L2 Resolution:**
1. Go to Purview Compliance Portal.
2. Create an eDiscovery Standard case.
3. Set condition: Sender = `sharma@company.com`, Recipient domain `!= company.com`, Date `> 30 days ago`.
4. Export the results to a PST file and hand it over to HR/Legal securely.

### 🎫 Scenario 4: User Complains About Blocked Email (DLP)

> [!example] Ticket
> "I am trying to send an Excel sheet to our vendor, but Outlook gives me a 'Policy Tip' saying message is blocked due to sensitive info."

**L1 Response:** Ask user what kind of data is in the sheet. *User shayad PAN ya credit card numbers bhej raha hai.*
**Escalation Trigger:** If it's a legitimate business requirement and the system is giving a false positive.
**L2 Resolution:**
1. Review the DLP policy matches in Purview compliance portal.
2. If it's a false positive, adjust the confidence level of the DLP policy or add an exception for that specific vendor domain.
3. Instruct user to encrypt the email or use a secure file transfer method if the data is genuinely sensitive.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Retention Policy and Litigation Hold?
> **Answer:** **Retention Policy** ek broad policy hai jo pure company ya specific groups pe lagti hai data ko ek fix time tak rakhne ya delete karne ke liye. **Litigation Hold** specific user mailbox pe lagaya jata hai jab koi legal investigation chal rahi ho, taaki us mailbox ka koi bhi data delete na ho sake (purges and hard deletes block ho jate hain).
> ==**Exam Tip:** Use Litigation hold for legal cases, Retention for compliance standards.==

> [!question] Q2: How does Safe Attachments work in Microsoft Defender?
> **Answer:** Safe Attachments har incoming email ke attachment ko ek isolated virtual environment (sandbox) mein open karta hai aur uska behavior analyze karta hai. Agar file malicious nikli, toh usko block kar deta hai aur user tak sirf ek warning message pahuchta hai.
> ==**Exam Tip:** Dynamic Delivery is a feature of Safe Attachments where the user gets the email body immediately, while the attachment is still being scanned.==

> [!question] Q3: You created a new Conditional Access Policy but locked yourself out of the tenant. How could you have prevented this?
> **Answer:** By creating a "Break-glass" or Emergency Access account. Yeh ek highly secure global admin account hota hai (with long complex password, no MFA policy applied directly but secured physically) jisko CA policies se exclude rakha jata hai.
> ==**Exam Tip:** Always exclude at least one global admin account when creating block/MFA Conditional Access policies.==

> [!question] Q4: How does Data Loss Prevention (DLP) identify sensitive information?
> **Answer:** DLP uses **Sensitive Information Types (SIT)**. Yeh built-in ya custom regular expressions (Regex) aur keywords hote hain. For example, ek credit card SIT check karega ki data 16 digit ka hai aur usme Luhn algorithm pass hota hai, plus aas paas "Visa", "Mastercard" jaise keywords hain.
> ==**Exam Tip:** DLP can not only detect but also encrypt, block, or send policy tips to users.==

> [!question] Q5: What is ZAP (Zero-hour Auto Purge) in Exchange Online?
> **Answer:** ZAP ek aisi technology hai jo retrospectively emails ko delete karti hai. Agar ek email user ke inbox mein deliver ho gayi, aur 2 ghante baad Microsoft ko pata chala ki wo spam/malware thi, toh ZAP automatically us mail ko user ke inbox se nikal kar quarantine/junk mein daal dega.
> ==**Exam Tip:** ZAP works even after the message has been delivered to the Exchange Online mailbox.==

> [!question] Q6: Can we track admin actions or file deletions in M365?
> **Answer:** Yes, using the **Unified Audit Log (UAL)** in the Purview portal. Yahan pe har admin activity, file modification, login attempt aur permission changes track hote hain.
> ==**Exam Tip:** Unified Audit Log needs to be turned ON initially for a tenant. It is not enabled by default for older tenants.==

---
## 🔗 Related Notes

- [[M365-01 Introduction and Architecture]] — M365 overview
- [[M365-02 Exchange Online Admin]] — Mail flow and transport rules
- [[AZ-03 Azure Active Directory Entra ID]] — Learn more about Conditional Access
- [[WIN-10 PowerShell for Admins]] — Connecting to Exchange via PS
