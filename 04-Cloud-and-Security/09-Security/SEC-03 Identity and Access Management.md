---
tags: [security, iam, identity, access, difficulty-intermediate]
aliases: [IAM, Identity and Access Management, SEC-03]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-red]
> 🛡️ **SECURITY**

`#complete` `#intermediate` `#none`

# SEC-03: Identity and Access Management

> [!abstract] Overview
> Identity and Access Management (IAM) is the security discipline that enables the right individuals to access the right resources at the right times for the right reasons. *Yeh ek security framework hai jo ensure karta hai ki sahi insaan (ya system) ko sahi resources ka access mile, aur galat log bahar rahein. Ek support engineer ke liye IAM samajhna bahot zaroori hai kyunki 60% se zyada helpdesk tickets password reset ya access denied ke aate hain.*

---
## 🧠 Concept Overview

- **What it is** — A system of policies, processes, and technologies that manage digital identities and user access to data, systems, and resources. *Yeh decide karta hai ki network mein kaun enter kar sakta hai (Authentication) aur andar aane ke baad kya kar sakta hai (Authorization).*
- **Why it matters** — Without IAM, organizations risk data breaches, compliance violations, and operational inefficiencies. *Agar IAM theek nahi hai, toh koi bhi sensitive data chura sakta hai, aur employees ko apna kaam karne mein dikkat aayegi.*
- **Where you see this** — Active Directory, Azure AD, AWS IAM, Okta, Ping Identity, and everyday logins like SSO (Single Sign-On). *Jab aap office laptop me login karte ho, ya kisi internal portal pe jaate ho bina naya password daale, wahan IAM kaam kar raha hota hai.*

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Password resets, account unlock, basic user provisioning, checking if account is active. *Dekhna ki user locked toh nahi hai, aur password reset karna.* |
| **L2** | Configure, fix, escalate kab karta hai — RBAC (Role-Based Access Control) assignments, troubleshooting MFA (Multi-Factor Authentication) issues, configuring basic SSO policies. *Access groups modify karna aur MFA issues fix karna.* |
| **L3** | Architecture, design, enterprise-level — Designing identity federation, implementing Zero Trust architecture, integrating complex directory services. *Pura architecture design karna aur nayi security policies lagana.* |

> [!tip] Seedha Simple Mein
> *IAM basically ek club ka bouncer hai. Pehle wo aapki ID dekhta hai (Authentication) aur phir decide karta hai ki aap VIP lounge mein jaa sakte ho ya sirf regular area mein (Authorization).*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **An Airport** is like **Identity and Access Management** because...
>
> - **Authentication (Identity Check):** When you show your passport and ticket at the entrance or check-in counter. *Jab aap apna ID dikhate ho, woh prove karta hai ki aap kaun ho.*
> - **Authorization (Access Level):** Your boarding pass determines which terminal and gate you can access. You can't just walk into the cockpit! *Aapki ticket decide karti hai ki aap kahan jaa sakte ho, same as user permissions.*
> - **MFA (Multi-Factor Authentication):** Scanning your fingerprint at immigration along with your passport. *Ek se zyada proof dena.*
> - **Auditing/Logging:** The airport security cameras and boarding scans track everywhere you went. *Logs batate hain ki aapne system mein kya kya kiya.*

---
## 🔬 Technical Deep Dive

### 1. The Triad: Identification, Authentication, and Authorization

> [!info] Key Concept
> AAA Model: Authentication, Authorization, and Accounting (or Auditing).

- **Identification:** Claiming an identity (e.g., entering a username). *System ko batana ki "Main Rahul hoon".*
- **Authentication:** Proving the identity (e.g., entering a password). *Prove karna ki sach mein main hi Rahul hoon (password daal ke).*
- **Authorization:** Determining what the authenticated user can do (e.g., read vs. write access). *System decide karega ki Rahul ko kya access milna chahiye.*
- **Accounting:** Tracking user actions for security and compliance. *User ne kya kiya uska hisaab rakhna logs ke through.*

> [!danger] Common Mistake
> Confusing Authentication with Authorization. *Log aksar in dono me confuse hote hain. Yaad rakho, login karna authentication hai, par file open karna authorization hai.*

### 2. Multi-Factor Authentication (MFA)

MFA requires two or more pieces of evidence to authenticate a user.
- **Something you know:** Password, PIN, Security Question. *Jo aapke dimaag mein hai.*
- **Something you have:** Smartphone (App/SMS), Smartcard, Hardware token (YubiKey). *Jo aapki pocket mein hai.*
- **Something you are:** Biometrics (Fingerprint, Retina scan, FaceID). *Jo aap khud ho.*
- **Somewhere you are:** Location-based (GPS, IP Address network). *Aap kahan se login kar rahe ho.*
- **Something you do:** Keystroke dynamics, voice print. *Aap kaise type karte ho ya bolte ho.*

### 3. Access Control Models

- **Discretionary Access Control (DAC):** Data owner decides who gets access. Highly flexible but risky. *File ka owner khud decide karta hai ki kisko access dena hai.*
- **Mandatory Access Control (MAC):** Strict OS-level control based on security labels (e.g., Top Secret). Used in military. *System decide karta hai security clearance ke hisaab se.*
- **Role-Based Access Control (RBAC):** Access is based on user's job role (e.g., HR, IT, Finance). Best for business. *Aapke role ke hisaab se automatically access milta hai.*
- **Attribute-Based Access Control (ABAC):** Access based on policies combining multiple attributes (e.g., Time of day, device type, location). *Dynamic rules, jaise office hours me hi access milega aur office device se hi.*

### 4. Single Sign-On (SSO) and Federation

> [!info] Key Concept
> **SSO** allows a user to authenticate once and access multiple related but independent software systems.

- **SAML (Security Assertion Markup Language):** XML-based, traditionally used for enterprise SSO.
- **OAuth 2.0 / OIDC (OpenID Connect):** JSON-based, used mainly for web apps and APIs to authorize access.
- **Identity Federation:** Extending SSO across different organizations (e.g., logging in with Google on a 3rd party site).
*SSO se user ko baar baar password yaad nahi rakhna padta, ek baar login kiya aur sabhi internal tools ka access mil jata hai.*

### 5. Zero Trust Architecture (ZTA)

"Never trust, always verify."
No user or device is trusted by default, even if they are inside the corporate network. Every access request is fully authenticated, authorized, and encrypted. *Network ke andar ho iska matlab yeh nahi ki aap safe ho. Har kadam pe checking hogi.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux server or Windows Server with Active Directory.
> - Root or Administrator access.
> - Basic understanding of user management commands.

### Step 1: Creating a User and Group (Linux IAM Basics)

Let's implement RBAC locally on a Linux machine.

```bash
# Add a new group for developers
groupadd developers
groupadd hr_team

# Create a new user 'alex' and assign to the 'developers' group
useradd -m -g developers -c "Alex from Dev Team" alex

# Create a new user 'sara' for HR
useradd -m -g hr_team -c "Sara from HR" sara

# Set a password for 'alex'
passwd alex
```

> [!success] Expected Output
> ```
> Changing password for user alex.
> New password:
> Retype new password:
> passwd: all authentication tokens updated successfully.
> ```

### Step 2: Implementing RBAC via Sudoers

Instead of giving full root access, we give specific access based on the role (group).

```bash
# Edit the sudoers file safely using visudo to prevent syntax errors
visudo

# Add the following lines at the end of the file:
# Allow developers to restart the web server only without password
%developers ALL=(ALL) NOPASSWD: /bin/systemctl restart httpd

# Allow HR team to only manage users (just an example, usually risky)
%hr_team ALL=(ALL) /usr/sbin/useradd, /usr/sbin/usermod
```

> [!success] Expected Output
> The file is saved without syntax errors, and the user 'alex' can now run `sudo systemctl restart httpd` without entering a password, but cannot run other privileged commands. Sara can manage users but cannot restart services.

### Step 3: File Permissions (DAC to RBAC transition)

```bash
# Create a shared project directory
mkdir /var/www/project

# Change group ownership to developers
chown root:developers /var/www/project

# Give read, write, execute permissions to the group
chmod 770 /var/www/project
```

### Step 4: Checking Effective Permissions (Windows AD PowerShell equivalent)

If you were on Windows, you would do this:

```powershell
# Find all groups a user belongs to
Get-ADPrincipalGroupMembership -Identity "alex" | Select-Object Name

# Check account lockout status
Get-ADUser -Identity "alex" -Properties LockedOut | Select-Object Name, LockedOut
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `useradd` | Creates a new user account | `useradd -m john` |
| `passwd` | Changes user password | `passwd john` |
| `usermod` | Modifies user account properties (like adding to a group) | `usermod -aG sudo john` |
| `userdel` | Deletes a user account | `userdel -r john` |
| `chage` | Changes user password expiry information | `chage -l john` |
| `Get-ADUser` | (PowerShell) Retrieves AD user info | `Get-ADUser -Identity smithj` |
| `Unlock-ADAccount` | (PowerShell) Unlocks a locked AD account | `Unlock-ADAccount -Identity smithj` |
| `Reset-ADAccountPassword` | (PowerShell) Resets user password | `Reset-ADAccountPassword -Identity smithj` |
| `aws iam list-users` | (AWS CLI) Lists IAM users in AWS | `aws iam list-users` |
| `az ad user list` | (Azure CLI) Lists users in Azure AD | `az ad user list` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **User cannot login** | Wrong password, Account locked out due to too many failed attempts, Account disabled, or Expired password. | *Check logs to see if account is locked. Reset password or unlock account.* (`passwd -u username` or `Unlock-ADAccount`) |
| **User logs in but gets "Access Denied" on a folder** | Missing authorization/permissions, User not in the correct RBAC group, or propagation delay. | *Check user's group membership. Add user to the correct group and ask them to re-login to refresh their access token.* |
| **MFA prompt not appearing** | Phone lost, Sync issue with authenticator app, policy not applied correctly, or conditional access bypassed it. | *Re-register MFA device for the user, or provide a temporary bypass code if authorized. Check if time is synced on phone.* |
| **SSO Login Fails (SAML Error)** | Time synchronization issue between servers, Expired certificate, wrong ACS URL mapping. | *Check server time (NTP is crucial), verify IdP and SP certificates are valid and matched.* |
| **Password keeps expiring immediately** | "User must change password at next logon" is set incorrectly, or max password age policy is misconfigured. | *Review AD/Linux password policies. Check `chage -l username` on Linux.* |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Account Lockout Nightmare

> [!example] Ticket
> "Hi IT, I am trying to login to my laptop but it says 'The referenced account is currently locked out'. I didn't even enter my password wrong today!"

**L1 Response:** Verify user identity via phone call or manager. Check AD or Identity Portal to confirm the lockout status and unlock the account. *Pehle check karo account sach me lock hai ya nahi, aur unlock kar do.*
**Escalation Trigger:** The account keeps locking out every 5-10 minutes automatically after being unlocked. *Agar unlock karne ke baad bhi baar baar lock ho raha hai.*
**L2 Resolution:** Investigate where the bad login attempts are coming from (e.g., stale cached credentials on a mobile device syncing email, a scheduled task using an old password, or a brute-force attack). Check Windows Event Viewer logs (Event ID 4740 on the Domain Controller) to find the source machine.

### 🎫 Scenario 2: Promotion and Access Request

> [!example] Ticket
> "John Doe has been promoted to Senior Financial Analyst. Please grant him access to the 'Confidential-Finance' folder and the ERP Admin portal."

**L1 Response:** Request written approval from the Data Owner / Finance Manager as per IAM policy. *Bina manager ke approval ke access nahi dena. Audit me fas jayenge.*
**Escalation Trigger:** The ERP Admin portal requires custom role mapping that L1 doesn't have access to modify. *Agar L1 ke paas portal pe role assign karne ka access nahi hai.*
**L2 Resolution:** Add the user to the appropriate AD Security Group for the folder access. Log into the ERP system and assign the "Admin" role using RBAC policies. Ask the user to log out and log back in to refresh their kerberos token.

### 🎫 Scenario 3: MFA Device Lost or Damaged

> [!example] Ticket
> "I dropped my phone in the pool over the weekend and just bought a new one. I cannot access the VPN because it requires the Authenticator app."

**L1 Response:** Verify user identity strictly (e.g., via video call, manager confirmation, or HR data) because attackers often use this excuse for social engineering (MFA fatigue/bypass). *Bohat dhyan se verify karna, ye social engineering attack ho sakta hai.*
**Escalation Trigger:** If L1 doesn't have permissions to reset MFA tokens in the central identity provider (like Azure AD or Okta).
**L2 Resolution:** Revoke the old MFA device sessions in the identity provider. Require re-registration of MFA on the next login, and guide the user through setting up the authenticator app on their new phone.

### 🎫 Scenario 4: Contractor Termination (Urgent)

> [!example] Ticket
> "Contractor XYZ's contract ended today. Terminate all access immediately."

**L1 Response:** Disable the Active Directory account, reset the password to a random value, hide from GAL (Global Address List), and force terminate active sessions in Microsoft 365. *Turant account disable karna taaki access band ho jaye.*
**Escalation Trigger:** The contractor had access to third-party cloud apps that aren't integrated with AD/SSO.
**L2/L3 Resolution:** Manually log into isolated systems (like specific AWS accounts or standalone databases) to remove access. Ensure an offboarding script runs to check for any orphaned accounts left behind.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Authentication and Authorization?
> **Answer:** Authentication is verifying WHO you are (e.g., entering a username and password). Authorization is determining WHAT you are allowed to do once you are authenticated (e.g., having read, write, or delete permissions on a specific file). *Ek club ka entry pass hai, dusra access level hai ki aap VIP lounge jaa sakte ho ya nahi.*
> ==**Exam Tip:** Remember AAA - Authentication (Who), Authorization (What), Accounting (What you did).==

> [!question] Q2: Can you explain RBAC and give an enterprise example?
> **Answer:** RBAC stands for Role-Based Access Control. Instead of assigning permissions directly to individual users, permissions are assigned to roles (or security groups), and users are assigned to those roles. For example, creating an "HR_Managers" group that has access to employee records, and simply putting all HR staff in that group. *Seedhe user ko permission dene ke bajaye, group ko permission do aur user ko group mein daal do. Isse management aasan hota hai.*
> ==**Exam Tip:** RBAC is the most common and manageable access model in enterprise environments. It scales easily.==

> [!question] Q3: What is the principle of least privilege (PoLP)?
> **Answer:** It means giving a user the minimum levels of access – or permissions – needed to perform his/her job functions, and nothing more. If someone only needs to read a document, they shouldn't have edit rights. *Kisi ko bhi zaroorat se zyada rights nahi dena. Default is always 'implicit deny'.*
> ==**Exam Tip:** This is a core security principle. It minimizes the potential damage of a compromised account.==

> [!question] Q4: How does Single Sign-On (SSO) work at a high level?
> **Answer:** SSO allows a user to authenticate once with a central Identity Provider (IdP). The IdP then provides a trusted token (like a SAML assertion) to various Service Providers (SP). Because the SP trusts the IdP, the user doesn't have to log in separately to each application. *Ek baar login karo, aur sab jagah access mil jayega bina password daale.*
> ==**Exam Tip:** Understand the difference between IdP (who holds the identities) and SP (who provides the service/app). SAML and OIDC are common SSO protocols.==

> [!question] Q5: What is Multi-Factor Authentication (MFA) and what are its factors?
> **Answer:** MFA requires two or more independent pieces of evidence to authenticate. The factors are: Something you know (password), Something you have (phone/hardware token), Something you are (biometric fingerprint/face), Somewhere you are (location/IP), Something you do (typing pattern). *Kam se kam do alag alag tareeke ka proof chahiye.*
> ==**Exam Tip:** Two passwords is NOT MFA (it's two of the same factor). A password and a text message code IS MFA.==

---
## 🔗 Related Notes

- [[SEC-01 Cyber Security Fundamentals|SEC-01: Cyber Security Fundamentals]] — Basics of security CIA triad.
- [[SEC-02 Cryptography and PKI|SEC-02: Cryptography and PKI]] — How encryption works, often used in IAM.
- [[AD-01 Active Directory Basics|AD-01: Active Directory Basics]] — Microsoft's primary IAM solution.
- [[AZ-01 Azure AD (Entra ID)|AZ-01: Azure AD (Entra ID)]] — Cloud-based identity management.
- [[LNX-03 User and Group Management|LNX-03: User and Group Management]] — Deep dive into Linux user administration.
