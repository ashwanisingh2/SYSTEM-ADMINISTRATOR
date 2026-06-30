---
tags: [microsoft-365, intune, conditional-access, security, mdm, mam]
aliases: [conditional-access, ca-policies, intune-ca]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#advanced` `#md-102`

# INT-18: Conditional Access

> [!abstract] Overview
> *Conditional Access (CA) Azure AD / Entra ID ka wo advanced security feature hai jo "If-Then" logic pe kaam karta hai. Agar ek user (If) kisi specific location ya device se login kar raha hai, toh CA decide karega ki usse access dena hai, block karna hai, ya MFA (Multi-Factor Authentication) maangna hai. Ek system admin ya security engineer ke liye yeh zero-trust architecture implement karne ka sabse powerful aur essential tool hai. Iske bina modern security incomplete hai.*

---
## 🧠 Concept Overview

- **What it is** — Conditional Access policies are the if-then statements of Entra ID (formerly Azure AD). They bring signals together, make decisions, and enforce organizational policies automatically during authentication.
- **Why it matters** — Traditional network perimeters are no longer effective because users work from anywhere, on any device. CA provides identity-driven security, enforcing access controls based on real-time risk context instead of just a username and password.
- **Where you see this** — Enforcing MFA for external access, blocking logins from high-risk countries, requiring devices to be Intune compliant before accessing company emails, or restricting session timeouts for non-managed devices.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check if a user is blocked by a CA policy (Sign-in logs), guide users to register for MFA, or check device compliance status in Intune. *Basic log check karke reason find karna.* |
| **L2** | Add exclusions for specific users (break-glass accounts), troubleshoot why a CA policy didn't apply, test new policies in "Report-only" mode. *Exceptions manage karna aur policies ko modify karna.* |
| **L3** | Architect enterprise-wide CA strategy, define baseline security policies, integrate Defender for Cloud Apps session controls, configure risk-based access. *Poore organization ka zero-trust architecture design karna.* |

> [!tip] Seedha Simple Mein
> *Conditional Access bilkul ek smart aur strict security guard ki tarah hai. Wo sirf tumhara ID card (password) nahi dekhta, balki yeh bhi dekhta hai ki tum kahan se aaye ho (IP Location), tumhara device kaisa hai (Intune compliance), aur kya tumhara behavior suspicious lag raha hai (Risk score). Agar sab theek hai toh allow karega, warna aur proof (MFA) mangega ya seedha bahar nikal dega (Block).*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Conditional Access** is like a **High-Security Bank Vault Access System** because...
>
> - **[The Signal / Condition]** The security guard checks your ID (User), sees you are arriving at 2 AM (Time/Risk), and notices you aren't wearing the bank uniform (Device State).
> - **[The Decision]** The guard's protocol dictates that this combination of factors is highly unusual and potentially dangerous.
> - **[The Enforcement]** Instead of just letting you in (Allow) or sending you away (Block), the guard requires you to provide a fingerprint scan and calls your manager to verify it's really you (Require MFA & Compliance).

---
## 🔬 Technical Deep Dive

### 1. Signals (The "If" Statement)

> [!info] Key Concept
> Signals are the real-time data points Conditional Access uses to make a decision during a sign-in attempt.

- **User or Group membership:** Policies can target specific users, groups, or directory roles (like Global Admins). You can also target Guest or External users specifically.
- **IP Location:** You can define Trusted IPs (like your corporate network) and Untrusted IPs. You can block access from entire countries using named locations.
- **Device Platform:** Target specific operating systems: Windows, macOS, iOS, Android, or Linux.
- **Client Application:** Differentiate between browser access, mobile apps, desktop clients (like Outlook), or legacy authentication protocols (like IMAP/POP).
- **Sign-in / User Risk:** Integration with Entra ID Protection. It can detect leaked credentials, anonymous IP usage, or impossible travel scenarios and adjust the policy dynamically.

> [!danger] Common Mistake
> *Kabhi bhi aisi CA policy mat banao jo "All Users" aur "All Cloud Apps" par apply hoti ho bina kisi exclusion ke. Agar tumne galti se "Block" select kar diya, toh tum apna access bhi block kar doge aur tenant se hamesha ke liye lock out ho jaoge! Always use a Break-glass account.*

### 2. Decisions (The "Then" Action)

- **Block access:** The most restrictive action. The user cannot sign in under any circumstances if the conditions match.
- **Grant access:** The least restrictive. The user signs in without interruption.
- **Require one or more controls:** The sweet spot. You can require MFA, require the device to be marked as compliant (Intune), require Hybrid Azure AD joined device, or require an approved client app.

### 3. Session Controls

Session controls allow you to limit the experience *within* a cloud application once access is granted.
- **App Enforced Restrictions:** Restrict downloading attachments in Outlook on the web for unmanaged devices.
- **Conditional Access App Control:** Uses Microsoft Defender for Cloud Apps to monitor and control user sessions in real-time (e.g., block downloading sensitive files).
- **Sign-in Frequency:** Force a user to re-authenticate after a certain number of days or hours.

### 4. Policy Evaluation Logic

When a user signs in, all enabled CA policies are evaluated. 
- The evaluation uses a logical **AND** operator across different policies. 
- For access to be granted, **ALL** policies that apply to the sign-in scenario must be satisfied. 
- If even **ONE** applicable policy blocks access, the user is blocked immediately. 

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Entra ID P1 or P2 license (Azure AD Premium P1/P2).
> - Global Administrator, Security Administrator, or Conditional Access Administrator role.
> - A test user account and a dedicated Break-Glass Admin account.

### Step 1: Create a "Require MFA for Admins" Policy

1. Go to the **Entra admin center** (`entra.microsoft.com`).
2. Expand **Protection** and click on **Conditional Access**.
3. Click on **New policy** -> **Create new policy**.
4. Name the policy: `CA-01-Require MFA for Directory Roles`.

> [!success] Quick Win
> *Naming convention hamesha standard aur clear rakho. Best practice format hai: "[Target] - [Condition] - [Action]". Example: "All Admins - Any App - Require MFA". Isse policy list dekh kar hi samajh aa jayega ki kya ho raha hai.*

### Step 2: Configure Assignments (Users and Roles)

1. Under **Users**, click the link. Select **Select users and groups**.
2. Check the box for **Directory roles** and select highly privileged roles like *Global Administrator*, *Security Administrator*, *Exchange Administrator*, and *Privileged Role Administrator*.
3. **CRITICAL STEP:** Go to the **Exclude** tab. Select **Users and groups**, and add your "Break-glass" emergency access account!

### Step 3: Configure Target Resources

1. Under **Target resources**, click the link.
2. Ensure the dropdown is set to **Cloud apps**.
3. Choose **All cloud apps**. (Admins should have MFA everywhere).

### Step 4: Configure Conditions (Optional but recommended)

1. Under **Conditions**, click **Client apps**.
2. Set **Configure** to **Yes**.
3. Leave all options checked (Browser, Mobile apps and desktop clients).

### Step 5: Configure Grant Controls

1. Under **Access controls** > **Grant**, click the link.
2. Select **Grant access**.
3. Check the box for **Require multifactor authentication**.
4. Click **Select**.

### Step 6: Enable Policy in Report-only Mode

1. At the bottom of the screen, under **Enable policy**, select **Report-only**.
2. Click **Create**.

> [!success] Expected Output
> *Aapki nayi policy CA list mein "Report-only" state mein dikhegi. Iska matlab hai ki policy background mein sign-ins ko evaluate karegi aur logs mein result dikhayegi, par actually kisi ko block ya MFA ke liye prompt nahi karegi. Testing ke liye yeh best aur safest practice hai!*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command (PowerShell - Microsoft.Graph) | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `Get-MgIdentityConditionalAccessPolicy` | Tenant ki saari CA policies ko list karta hai. | `Get-MgIdentityConditionalAccessPolicy` |
| `Get-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId <ID>` | Kisi specific CA policy ki puri configuration details nikalta hai. | `Get-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId "123-abc"` |
| `New-MgIdentityConditionalAccessPolicy` | Script ke through ek nayi CA policy create karta hai (JSON/Hash table use karke). | `New-MgIdentityConditionalAccessPolicy -BodyParameter $PolicyConfig` |
| `Update-MgIdentityConditionalAccessPolicy` | Existing policy ko modify karta hai (jaise Report-only se On karna). | `Update-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId "xyz" -State "enabled"` |
| `Remove-MgIdentityConditionalAccessPolicy` | Kisi CA policy ko permanently delete karta hai. | `Remove-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId "123"` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| User ko baar-baar MFA prompt aa raha hai, even on corporate network. | User is hitting a policy that requires MFA for every session, or the "Sign-in frequency" is set too low. Or, corporate IP is not added to Trusted Locations. | Check Azure AD Sign-in logs -> Conditional Access tab. Identify the triggering policy. Add corporate public IP to Named Locations and exclude it from the policy if needed. |
| User "Device not compliant" error se login mein block ho raha hai. | Device is not enrolled in Intune, or it is enrolled but failing a compliance policy (e.g., BitLocker is off, or OS is outdated). | Intune admin center mein device ki compliance state check karo. User ko Windows mein "Company Portal" app open karke "Sync" pe click karne bolo. |
| Administrator galti se apne account se lock out ho gaya! (Tenant Lockout) | Admin created a CA policy that blocked all access, applied it to all users/admins, and forgot to exclude themselves or a break-glass account. | Use the "Break-glass" emergency account to log in and disable the faulty policy. If no break-glass exists, you must open a critical Microsoft Support ticket (can take days to resolve). |
| Policy is enabled but targeted users are NOT getting prompted for MFA. | Policy might be scoped incorrectly (wrong AD group), or user might be coming from a "Trusted IP" that is explicitly excluded from the policy. | Use the "What If" diagnostic tool in the Conditional Access blade to simulate the user's login scenario and see exactly why the policy was bypassed. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Executive traveling to an unapproved country

> [!example] Ticket
> "URGENT: Our CEO is traveling to China for an industry conference and cannot access his corporate email. It says his login is blocked by organization policy. Please fix immediately."

**L1 Response:** *Sabse pehle Entra ID Sign-in logs check karo for the CEO. Wahan clearly dikhega ki kaunsi CA policy block kar rahi hai. Usually enterprise environments mein ek "Block logins from outside approved countries" ya "Block High Risk Locations" policy hoti hai.*
**Escalation Trigger:** Modifying company-wide CA policies usually requires Security Admin or L2/L3 approval to avoid compliance breaches.
**L2 Resolution:** Add a temporary exclusion for the CEO to the geolocation blocking CA policy for the exact duration of his trip. Alternatively, create a specific, tightly-scoped policy that allows his access from that country but stringently requires a compliant managed device AND multi-factor authentication.

### 🎫 Scenario 2: Legacy Email Client Blocking

> [!example] Ticket
> "User updated their iPhone to a new OS, and now the native Apple Mail app is asking them to sign in constantly, but it fails saying access is blocked by the IT department."

**L1 Response:** *Verify if the user is using the native iOS Mail app. Explain to the user that the organization blocks legacy authentication protocols and non-approved apps for security reasons.*
**Escalation Trigger:** If the user is a VIP, insists they have an exception, or if it's a specialized service account that needs IMAP.
**L2 Resolution:** Confirm that a "Block Legacy Authentication" or "Require Approved Client App (Outlook)" CA policy is actively enforcing this. Do NOT disable the policy. Instruct the user to download the official Microsoft Outlook app from the App Store, which supports Modern Authentication, and sign in there instead.

### 🎫 Scenario 3: Impossible Travel Alert (Identity Protection)

> [!example] Ticket
> "Security Alert: Sign-in risk detected for user John Doe - Impossible travel to atypical locations. User logged in from New York, and 30 minutes later from Moscow."

**L1 Response:** *Check the Entra ID sign-in logs. Verify if the Conditional Access policy "Require MFA for medium/high user risk" was successfully triggered by this event.*
**Escalation Trigger:** If the suspicious Moscow login actually succeeded (meaning MFA was bypassed, completely missing, or the user blindly approved an MFA fatigue attack).
**L2/L3 Resolution:** Since CA caught it and prompted for MFA, check the result. If the MFA prompt was failed or blocked by the user, the system worked perfectly. Contact John Doe out-of-band (call him) to confirm his actual location. As a precaution, force a password reset and revoke all existing Entra ID refresh tokens/sessions to secure the account.

### 🎫 Scenario 4: User cannot access SharePoint from personal laptop

> [!example] Ticket
> "I am working from home on my personal MacBook. I can log into Office.com, but when I click on SharePoint or OneDrive, it says 'You cannot access this resource from an unmanaged device'."

**L1 Response:** *Check the CA policies. Verify that there is a policy requiring a 'Compliant Device' or 'Hybrid Azure AD Joined Device' for accessing SharePoint/OneDrive.*
**Escalation Trigger:** If the user claims it is a company-issued laptop but is still getting blocked.
**L2 Resolution:** If it is a personal device, the policy is working as intended. Explain the BYOD (Bring Your Own Device) policy to the user. If they need to work, they must use a corporate-managed device. If it IS a corporate device, troubleshoot Intune compliance on the MacBook (check Company Portal sync status).

---
## 🎤 Interview Questions

> [!question] Q1: What happens if two Conditional Access policies conflict with each other? For example, Policy A explicitly allows access, but Policy B explicitly blocks access.
> **Answer:** *Conditional Access policies are evaluated together. The most restrictive policy always wins. Is case mein, Policy B jisme "Block access" hai, wo override karegi, aur user block ho jayega. Block is the ultimate authority in CA.*
> ==**Exam Tip:** The implicit or explicit "Block" action will ALWAYS override any "Allow" or "Require MFA" action. Remember this rule for scenario questions.==

> [!question] Q2: How do you safely test a newly created Conditional Access policy without causing a massive outage for all users?
> **Answer:** *Create the policy and set its enforcement state to "Report-only" mode. Phir kuch din wait karo aur Entra ID Sign-in logs ya Conditional Access Insights workbook mein jake dekho ki agar yeh policy enabled hoti, toh kitne users pe kya impact padta. Jab testing successful ho, tabhi usko "On" karo.*
> ==**Exam Tip:** "Report-only" mode is a highly tested concept in Microsoft certifications. It is the only safe way to deploy CA policies at an enterprise scale.==

> [!question] Q3: What is a "Break-glass" account, and why is it absolutely critical when configuring Conditional Access?
> **Answer:** *Yeh ek ya do emergency global admin accounts hote hain jinhe tenant ki saabhi CA policies se explicitly exclude kiya jata hai. Agar koi admin galti se aisi policy bana de jo sabko block kar de (e.g., require MFA for all users, but the Microsoft MFA service goes down), toh in break-glass accounts ka use karke tenant ka access wapas liya ja sakta hai aur policy disable ki ja sakti hai.*
> ==**Exam Tip:** Break-glass accounts should be cloud-only (`*.onmicrosoft.com`), have insanely strong passwords (30+ characters) stored securely in a physical safe, and should NOT require MFA through CA (though they should use physical FIDO2 security keys).==

> [!question] Q4: How can you use Conditional Access to effectively block legacy authentication protocols across the organization?
> **Answer:** *Create a new CA policy. Under 'Conditions' -> 'Client apps', configure it to select ONLY 'Other clients' (which represents legacy protocols like IMAP, POP, SMTP). Under the 'Grant' control, select 'Block access'. Apply this policy broadly to 'All users' (excluding break-glass).*
> ==**Exam Tip:** Blocking legacy authentication is the number one most effective way to stop password spray and brute-force attacks, because legacy auth does not support MFA.==

> [!question] Q5: You have a policy requiring a compliant device. A user complains they are blocked, but you see their device is enrolled in Intune. What is the issue?
> **Answer:** *Enrolled doesn't necessarily mean Compliant. The device might be violating an Intune Compliance Policy. For example, it might not have an active antivirus, the OS version might be too old, or BitLocker encryption might be disabled. The Intune portal will show the exact reason for non-compliance.*
> ==**Exam Tip:** CA relies on Intune's compliance signal. If Intune says "Not Compliant", CA will strictly enforce the block.==

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Intune Architecture & Enrollment|INT-01 Intune Architecture & Enrollment]] — Learn how devices actually get enrolled to become "Compliant" for Conditional Access to check.
- [[04-Cloud-and-Security/07-Microsoft-365/SEC-03 Multi-Factor Authentication (MFA)|SEC-03 Multi-Factor Authentication (MFA)]] — Deep dive into the most common "Grant" control used in CA policies.
- [[04-Cloud-and-Security/07-Microsoft-365/ID-05 Azure AD Identity Protection|ID-05 Azure AD Identity Protection]] — Provides the crucial "Sign-in Risk" and "User Risk" AI-driven signals directly into Conditional Access.
- [[04-Cloud-and-Security/07-Microsoft-365/SEC-06 Defender for Cloud Apps|SEC-06 Defender for Cloud Apps]] — Learn how Conditional Access App Control works for real-time session monitoring.
