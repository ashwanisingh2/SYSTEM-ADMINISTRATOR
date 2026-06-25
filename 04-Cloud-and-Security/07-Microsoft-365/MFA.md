---
tags: [desktop-support, m365, mfa, security, L2]
aliases: [mfa-guide, multi-factor-auth, temporary-access-pass]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Multi-Factor Authentication (MFA)

---

## Concept Overview
- **What it is**: Multi-Factor Authentication (MFA) is a security process that requires users to provide two or more verification factors to gain access to Microsoft 365 services. It verifies identity using something you know (password), something you have (phone/token), and something you are (biometrics).
- **Why it matters for a support engineer**: Passwords are easily compromised. MFA blocks 99.9% of identity-based attacks. Helping users register, reset, and troubleshoot MFA devices is one of the most frequent support tickets in any modern enterprise.
- **Where you encounter this in real job**: Generating Temporary Access Passes (TAP) for new hires, resetting authentication methods for users who lost their phones, and troubleshooting "MFA prompt loop" authentication failures.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Resets user MFA registration methods (forces re-register), generates Temporary Access Passes, and assists with Microsoft Authenticator app setups.
  - **L2**: Troubleshoots MFA blockages in Entra Sign-in logs, audits registered methods, and configures SSPR-MFA synchronization.
  - **L3**: Enforces tenant-wide Authentication Policies, disables Legacy Authentication protocols, designs Phishing-Resistant MFA baselines (FIDO2 / WHfB), and monitors MFA fatigue attacks.

---

## Technical Deep Dive

### 1. MFA Verification Methods & Security Strengths
Microsoft Entra ID supports several authentication methods, ranked by their resistance to cyber-attacks:

| Method | Type | Attack Resistance | Support Considerations |
|---|---|---|---|
| **FIDO2 / Security Keys** | Cryptographic USB/NFC | **Phishing-Resistant** | Requires hardware distribution. Excellent for high-security environments. |
| **Windows Hello for Business** | Device-bound biometrics/PIN | **Phishing-Resistant** | Enrolled at device setup. Limited to that specific PC. |
| **Microsoft Authenticator (Number Matching)** | Push Notification + Entry | **High** | Prevents "MFA Fatigue" attacks. Requires smartphone. |
| **OATH Hardware Tokens** | Time-based code (TOTP) key fob | **Medium** | Used for users without company smartphones or in restricted zones. |
| **SMS / Voice Call** | Telephony code delivery | **Low** | Vulnerable to SIM-swapping and intercept attacks. Least secure. |

- **Temporary Access Pass (TAP)**: A time-limited passcode configured by an administrator. It acts as a strong authentication method, allowing a user to sign in and set up their permanent MFA methods without using a password.

### 2. Disabling Legacy Authentication
- *What it is*: Legacy authentication refers to old email protocols (such as IMAP, POP3, SMTP Auth, and MAPI) used by older clients (like Outlook 2010).
- *The Danger*: **Legacy protocols do not support MFA.** Even if you enforce MFA for a user, an attacker can bypass the MFA prompt entirely by logging into the user's account using an IMAP client.
- *The Fix*: Legacy authentication must be blocked tenant-wide using Entra ID Security Defaults or Conditional Access policies.

---

## Commands & Syntax

### PowerShell
MFA administration uses the `Microsoft.Graph.Identity.SignIns` module.
```powershell
# Connect to Microsoft Graph with authentication administration scopes
Connect-MgGraph -Scopes "UserAuthenticationMethod.ReadWrite.All"

# List all registered authentication methods for a target user
Get-MgUserAuthenticationMethod -UserId "jdoe@company.com"

# Generate a Temporary Access Pass (TAP) for a user (valid for 1 hour, one-time use)
$TAP = New-MgUserAuthenticationTemporaryAccessPassMethod -UserId "jdoe@company.com" -LifetimeInMinutes 60 -IsUsableOnce $true
$TAP.TemporaryAccessPass

# Revoke a user's MFA sessions (forces all their devices to prompt for login again)
Revoke-MgUserSignInSession -UserId "jdoe@company.com"
```

### CMD / Run Box
```cmd
:: Test connectivity to Microsoft's login portal authentication endpoints from the workstation
curl -I https://login.microsoftonline.com
```

### GUI Path
- **Reset User MFA Methods**: Entra Admin Center -> **Identity** -> **Users** -> **All Users** -> Select User -> **Authentication methods** -> Click **Require re-register MFA** or **Manage User features**.
- **Tenant-Wide Settings**: Entra Admin Center -> **Protection** -> **Authentication methods**.

### Important Registry Paths
- Enabling Modern Authentication in older Office 2013 installations:
  ```
  HKCU\Software\Microsoft\Office\16.0\Common\Identity
  (Create DWORD "EnableADAL" set to 1)
  ```

---

## Real-World Scenarios

### Scenario 1: User Lost Phone and is Locked Out of Microsoft 365
**User Complaint:** A project manager calls the helpdesk: *"I dropped my phone in the lake this weekend. I bought a new phone, but when I try to log in to my work email on my laptop, it sends a verification code to the Microsoft Authenticator app on my broken phone. I cannot access anything."*
**Your First 3 Checks:**
1. Verify the identity of the caller (always check corporate HR files or call back on a verified number).
2. Check if the user has secondary MFA methods registered (like SMS or office phone).
3. Access the Entra Admin Center to generate a Temporary Access Pass (TAP).
**Diagnosis Steps:**
1. Look up the user `jdoe@company.com` in the Microsoft Entra Admin Center.
2. Go to **Authentication methods**. Notice the only registered method is `Microsoft Authenticator app`.
3. To register a new phone, the user must log in. But they cannot log in without the old phone.
4. *We must bypass the MFA temporarily without disabling security.*
**Root Cause:** The user lost their sole registered MFA device, blocking authentication to the tenant.
**Fix:**
1. On the user's Authentication methods page, click **Add authentication method**.
2. Select **Temporary Access Pass**. Set lifetime to `60` minutes, and toggle **Require One-Time Use** to `Yes`.
3. Read the generated password to the user over the phone.
4. Instruct the user to open an Incognito window and navigate to `myaccount.microsoft.com`.
5. The user logs in using their UPN and the TAP passcode (bypassing the standard password and MFA prompt).
6. Under Security Info, the user deletes the old phone and registers the Microsoft Authenticator app on their new device.
**Prevention:** Educate users to register at least two MFA methods (e.g., Authenticator app + FIDO2 or OATH key) to prevent lockouts.
**Ticket Close Note:** "Verified user identity. Generated a Temporary Access Pass. Guided user through registering the Authenticator app on their new phone. Closed."

### Scenario 2: User Targeted by MFA Fatigue (Prompt Bombing) Attack
**User Complaint:** A HR supervisor contacts support: *"My phone has been buzzing every 5 minutes since 3:00 AM asking me to approve a sign-in. I did not approve any of them, but it is extremely annoying. What should I do?"*
**Your First 3 Checks:**
1. Check the Entra ID Sign-in logs for the user to find the source IP of the failed attempts.
2. Verify if the user's password has been compromised.
3. Check if Number Matching is active on the tenant's Authenticator policy.
**Diagnosis Steps:**
1. Open Microsoft Entra Admin Center -> **Monitoring & health** -> **Sign-in logs**.
2. Filter by user. Notice dozens of logons with status: `MFA mobile app notification sent`.
   - Source IP: `198.51.100.22` (located in a country the user is not in).
3. *Why did the attacker trigger MFA?* The attacker successfully guessed the user's actual password. They are now repeatedly sending push notifications to the user's phone, hoping the user will accidentally click "Approve" to stop the notifications (MFA Fatigue).
**Root Cause:** The user's password was compromised, allowing the attacker to trigger authentication push notifications repeatedly.
**Fix:**
1. Change the user's Active Directory/M365 password immediately to block the attacker from initiating requests.
2. Revoke the user's active sign-in sessions:
   `Revoke-MgUserSignInSession -UserId "hruser@company.com"`
3. Confirm that **MFA Number Matching** is enabled in the tenant authentication policies. With number matching, the user must look at the login screen and enter a displayed 2-digit number into their phone app. An attacker cannot guess the number, eliminating accidental approvals.
**Prevention:** Enforce MFA Number Matching globally and run identity threat workshops.
**Ticket Close Note:** "Compromised password reset. Revoked all active user sessions. Verified Number Matching policy is active. Attacker access blocked. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never disable MFA for a user account to resolve a temporary login issue, even if requested by an executive.
> - Disabling MFA leaves the account highly vulnerable to credential stuffing. If a user is facing login issues, generate a **Temporary Access Pass (TAP)** instead, keeping security controls active.

> [!warning] Common Trap
> - Assuming that legacy authentication is disabled automatically when you enable MFA.
> - Legacy clients bypass MFA. You must explicitly block legacy authentication using Entra Conditional Access policies or Security Defaults, or attackers will target legacy ports to compromise accounts.

> [!tip] Senior Engineer Tip
> - When onboarding new remote employees, generate a Temporary Access Pass (TAP) and send it via a secure secondary channel (e.g., encrypted SMS to their verified phone number). This allows them to complete their initial password setup and MFA registration securely without needing an initial temporary password.

> [!success] Verification Steps
> - Run: `Get-MgUserAuthenticationMethod -UserId "username@company.com"` to verify that the user has completed their MFA registration.
> - Check Entra Sign-in logs to confirm the sign-in status displays `MFA requirement satisfied by claim in token`.

> [!question] Interview Alert
> - "What is MFA Fatigue and how does Microsoft mitigate it?"
> - Answer: "MFA Fatigue is an attack where a threat actor who knows a user's password repeatedly sends MFA push notifications to their phone, hoping the user will get annoyed or distracted and approve the access. Microsoft mitigates this using 'Number Matching'. The user must enter the specific two-digit code displayed on the login screen into the Authenticator app, making accidental approvals impossible."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Keeping SMS as the primary MFA method | Easiest for users to set up | Enforce Microsoft Authenticator with Number Matching as the default verification method. |
| Forgetting to revoke sessions after a password change | Cached tokens remain active | Always click 'Revoke Sessions' in Entra ID when resetting a compromised account's password. |
| Leaving legacy authentication protocols active | Fearing compatibility issues with older apps | Upgrade email clients to modern versions and block legacy authentication tenant-wide. |

---

## Lab Exercise

**Objective:** Connect to Microsoft Graph via PowerShell, generate a Temporary Access Pass (TAP) for a test user, and verify its usability.
**Time Required:** 20 minutes
**Environment Needed:** A computer with internet access and a Microsoft 365 Developer tenant.
**Pre-requisites:** The Microsoft Graph PowerShell SDK installed.

**Steps:**
1. Open PowerShell as Administrator.
2. Connect to Microsoft Graph:
   ```powershell
   Connect-MgGraph -Scopes "UserAuthenticationMethod.ReadWrite.All"
   ```
3. Target a test user and check their current authentication methods:
   ```powershell
   Get-MgUserAuthenticationMethod -UserId "testuser@yourtenant.onmicrosoft.com"
   ```
4. Generate a one-time use Temporary Access Pass valid for 30 minutes:
   ```powershell
   $Pass = New-MgUserAuthenticationTemporaryAccessPassMethod -UserId "testuser@yourtenant.onmicrosoft.com" -LifetimeInMinutes 30 -IsUsableOnce $true
   Write-Host "TAP Passcode: $($Pass.TemporaryAccessPass)"
   ```
5. Open an Incognito browser window. Navigate to `https://myaccount.microsoft.com`.
6. Enter the test user's email. When prompted for password/MFA, enter the TAP passcode.
7. Verify that you log in directly to the security info page without a standard password prompt.

**Success Criteria:** The Temporary Access Pass is generated successfully via PowerShell, and grants direct access to the user account page.
**Common Failures:** The command fails if the Temporary Access Pass policy is disabled in the tenant's Global Authentication Methods settings.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a Temporary Access Pass (TAP) and when do you use it?**
A: A TAP is a temporary passcode generated by an admin that allows a user to sign in without using their password or standard MFA. We use it when onboarding new employees so they can set up their MFA securely, or when a user loses their phone and is locked out of their account.

**Q: How does a user set up Microsoft Authenticator on a new phone?**
A: The user logs in to `myaccount.microsoft.com`, goes to Security Info, clicks Add Method, and selects Authenticator App. They install the app on their phone, scan the QR code displayed on their computer screen, and complete the verification prompt.

### Intermediate (L2 Level)
**Q: Why is SMS authentication considered insecure compared to the Microsoft Authenticator app?**
A: SMS is vulnerable to SIM-swapping attacks, where an attacker tricks a carrier into porting the user's phone number to a new SIM card. SMS messages are also sent unencrypted over cellular networks and can be intercepted. The Authenticator app uses secure, encrypted push notifications and device-bound keys.

**Q: What is "Legacy Authentication" and why does it represent a major security risk?**
A: Legacy authentication refers to older communication protocols like IMAP, POP3, and SMTP that do not support modern interactive logon interfaces or multi-factor authentication. If legacy authentication is left active, attackers can bypass a user's MFA policy by logging in using these older protocols.

### Advanced (L3/Senior Level)
**Q: An organization is experiencing a high volume of successful password sprays, and several users have approved MFA prompts they did not initiate. How do you secure the tenant?**
A:
- **Situation**: Organization is vulnerable to password spray and accidental MFA approvals.
- **Task**: Enforce stronger authentication baselines and eliminate user-error approvals.
- **Action**: First, I implement a Conditional Access policy to block all Legacy Authentication protocols. Next, I enable **Microsoft Authenticator Number Matching** tenant-wide. I transition the default user authentication methods away from SMS/Voice towards the Authenticator app and FIDO2 keys. Finally, I configure Entra ID Protection sign-in risk policies to block sessions or force password changes if a logon attempt is flagged as high-risk.
- **Result**: Accidental MFA approvals were eliminated, and successful credential attacks dropped to zero.

### HR / Behavioral
**Q: Tell me about a time you had to handle an emergency security incident outside of normal business hours.**
A: At 11:00 PM, our monitoring system flagged that a user account was repeatedly attempting logins from suspicious locations, and the user's phone was receiving MFA prompts. I contacted the user directly to confirm they were not logging in. With their confirmation, I immediately locked the account, revoked all active login sessions via PowerShell, and reset their password. The next morning, I helped them register new MFA methods, securing the account and protecting the organization from a potential breach.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The security verification process requiring multiple credentials to access cloud identities.
> **Why**: Critical for preventing unauthorized access; blocks 99.9% of identity attacks.
> **How**: Enforce strong verification methods (Authenticator app, FIDO2), generate TAPs for lockouts, and block legacy protocols.
> **Command**: `New-MgUserAuthenticationTemporaryAccessPassMethod` / `Revoke-MgUserSignInSession`
> **Interview Answer Starter**: "To manage identity security effectively, I enforce multi-factor authentication using Microsoft Authenticator with number matching, while systematically blocking legacy protocols..."

**Key Numbers to Remember:**
- Minimum TAP duration: 10 minutes
- Maximum TAP duration: 30 days (default limit in policies is 24 hours)
- Standard device activation limit: 5 devices
- SSL port required for verification: TCP 443

**3 Things Interviewer Wants to Hear:**
- Number matching prevents MFA fatigue (push bombing) attacks
- Legacy protocols (IMAP/POP3) bypass MFA and must be blocked
- Using Temporary Access Passes (TAP) for secure user onboarding

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]] — The identity system managing MFA registrations.
- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]] — The rules engine that triggers MFA challenges.
- [[04-Cloud-and-Security/09-Security/MFA-and-Identity-Protection|MFA and Identity Protection]] — Covers advanced user and sign-in risk policies.

---

## Tags
#desktop-support #m365 #mfa #security #L2 #interview-topic #lab-complete #daily-use

