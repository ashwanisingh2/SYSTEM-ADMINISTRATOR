---
tags: [desktop-support, active-directory, security, password-policy, L2]
aliases: [fgpp-guide, password-security]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Password Policies

---

## Concept Overview
- **What it is**: Active Directory Password Policies are security configurations that define domain-wide rules for password complexity, minimum length, history limits, expiration age, and account lockout thresholds.
- **Why it matters for a support engineer**: A support engineer must know how password rules are evaluated to troubleshoot lockout loops, reset expired accounts, and configure custom security profiles for different departments.
- **Where you encounter this in real job**: Resolving password reset loop tickets, unlocking accounts (Event ID 4740), configuring Fine-Grained Password Policies (FGPP) for administrative accounts, and auditing weak configurations.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Resets user passwords, unlocks locked accounts, and explains password complexity rules to users.
  - **L2**: Manages account lockout policies, creates Password Settings Objects (PSOs), and configures fine-grained policies.
  - **L3**: Establishes identity security baselines, manages password synchronization tools, and audits authentication security logs.

---

## Technical Deep Dive

### 1. Default Domain Password Policy
- **The Rule of One**: Historically, Active Directory only supported a single password policy per domain, defined in the **Default Domain Policy** linked to the domain root. If you attempted to link a separate GPO containing different password settings to an OU, the settings were ignored by local workstations.
- **Scope**: The Default Domain Policy applies to all domain accounts, setting baseline rules (e.g., minimum 8 characters, complexity enabled, 90-day expiration).

### 2. Fine-Grained Password Policies (FGPP) & PSOs
Windows Server 2008 introduced FGPP to allow applying different password rules to different users:
- **Password Settings Object (PSO)**: A directory object (class `msDS-PasswordSettings`) that stores password and lockout settings.
- **Targeting**: PSOs are applied directly to User objects or Global Security Groups, completely bypassing the Default Domain Policy for those users.
- **Precedence Value**: If multiple PSOs apply to a user (due to overlapping group memberships), the PSO with the **lowest Precedence number** (e.g., 1 overrides 10) wins and applies its settings.

### 3. Account Lockout Policies
Defines the parameters used to block access after failed login attempts:
- **Account Lockout Threshold**: The number of failed login attempts allowed before the account locks (e.g., 5 attempts). Set to `0` to disable lockouts.
- **Account Lockout Duration**: The time the account remains locked before automatically unlocking (e.g., 30 minutes). Set to `0` to require an administrator to unlock it manually.
- **Reset Account Lockout Counter After**: The time window during which failed login attempts accumulate (e.g., 15 minutes). Must be less than or equal to the lockout duration.

---

## Commands & Syntax

### PowerShell
```powershell
# Create a new Fine-Grained Password Policy (PSO) for domain administrators
New-ADFineGrainedPasswordPolicy -Name "AdminPasswordPolicy" -Precedence 10 -ComplexityEnabled $true -MinPasswordLength 16 -LockoutThreshold 5 -LockoutDuration "00:30:00" -LockoutObservationWindow "00:15:00"

# Subject the new PSO to the Domain Admins security group
Add-ADFineGrainedPasswordPolicySubject -Identity "AdminPasswordPolicy" -Subjects "Domain Admins"
```

### CMD / Run Box
```cmd
REM Query current domain-wide password policy settings from cmd
net accounts
```

### GUI Path
> Server Manager -> Tools -> **Active Directory Administrative Center** (ADAC) -> Select Domain -> **System** container -> **Password Settings Container**.

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\NTDS\Parameters\
```

### Key Event IDs
Password policy auditing events are logged in the Security log of Domain Controllers:

| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 4723 | An attempt was made to change an account's password (by the user) | Security Log |
| 4724 | An attempt was made to reset an account's password (by an administrator) | Security Log |
| 4740 | A user account was locked out | Security Log |
| 4725 | A user account was disabled | Security Log |

---

## Real-World Scenarios

### Scenario 1: User Account Locks Out Instantly After Unlock (Conflict in Lockout Reset Counter)
**User Complaint:** "My account was locked out. The helpdesk unlocked it, but before I could type in my password, it locked out again. This has happened three times."
**Your First 3 Checks:**
1. Check the user's active login status and bad logon count.
2. Query the DC Security log for Event ID 4740 to find the lockout source.
3. Verify if the lockout reset window is too long compared to client sync times.
**Diagnosis Steps:**
1. Log into the DC. Filter the Security log for Event ID `4740` matching the user.
   - Message: `A member account was locked out. Caller Computer Name: TABLET-JDOE.`
2. The user has a tablet that is sending cached bad credentials in the background.
3. Check the default domain lockout settings:
   - Account Lockout Threshold: `3`
   - Reset Account Lockout Counter After: `30` minutes.
   - *Because the reset counter is 30 minutes, if the user makes 2 bad attempts on their laptop, and their tablet sends 1 bad request 20 minutes later, the threshold of 3 is breached and the account locks.*
**Root Cause:** A secondary device (tablet) sending cached bad credentials, combined with an overlapping lockout reset counter window, caused immediate re-lockouts.
**Fix:** Unlock the user account, isolate the tablet from Wi-Fi, update the cached password, and reduce the reset lockout counter setting to 15 minutes.
**Prevention:** Educate users to turn off Wi-Fi on secondary devices when changing passwords.
**Ticket Close Note:** "Isolated the lockout source to user's mobile device (tablet). Updated cached Wi-Fi credentials. Reset lockout count. Verified normal operation. Closing ticket."

### Scenario 2: Administrative Account Fails to Accept Password (Precedence Conflict)
**User Complaint:** "I am trying to change my domain administrator password to a new 12-character secure password, but the system rejects it, stating it does not meet length requirements, even though the policy is set to 8 characters."
**Your First 3 Checks:**
1. Verify the Default Domain Policy minimum password length setting.
2. Check if a Password Settings Object (PSO) is applied to the user.
3. Determine the precedence of active PSOs.
**Diagnosis Steps:**
1. Run PowerShell to check the effective password policy for the user:
   `Get-ADUserResultantPasswordPolicy -Identity "admin-user"`
   - Output: `MinPasswordLength: 16`
   - Active Policy: `HighSecPolicy` (Precedence 5).
2. The user belongs to the `Domain Admins` group, which has a custom PSO (`HighSecPolicy`) requiring a 16-character minimum length.
3. The user assumed only the Default Domain Policy (8 characters) applied.
**Root Cause:** A Fine-Grained Password Policy (PSO) with a lower precedence value overrode the Default Domain Policy, enforcing a stricter 16-character minimum length.
**Fix:** Instruct the user to select a secure password that meets the 16-character minimum requirement defined by the active PSO.
**Prevention:** Maintain clear documentation of all active PSOs and target groups.
**Ticket Close Note:** "Explained that the user's account is subject to a custom 16-character minimum PSO policy. User set compliant password. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Configure an Account Lockout Threshold value of `1` or `2` in the Default Domain Policy.
> - This allows attackers to perform simple denial of service (DoS) attacks on corporate accounts by sending a single incorrect password request, locking out the entire workforce.
> - Enforce a threshold of at least 5 to 10 attempts.

> [!warning] Common Trap
> - Creating a GPO with custom password settings and linking it to a user OU, expecting it to apply.
> - GPO-based password settings only apply when linked to the Domain Root. To apply different settings to OUs, you must use Fine-Grained Password Policies (PSOs) configured in ADAC or PowerShell.
> - Avoid linking custom password GPOs to sub-OUs.

> [!tip] Senior Engineer Tip
> - If you need to audit domain password strength compliance without reading passwords, use the **Active Directory Password Filter** DLL to block users from setting passwords that appear in known breached databases (e.g., HaveIBeenPwned API list).

> [!success] Verification Steps
> - Run: `Get-ADUserResultantPasswordPolicy -Identity "username"` to verify the active policy.
> - Confirm the settings match the required parameters.

> [!question] Interview Alert
> - "Explain how Fine-Grained Password Policies (FGPP) work in Active Directory."
> - Answer: "FGPP allows applying different password and lockout settings to different users and groups within the same domain. This is done by creating a Password Settings Object (PSO) and linking it directly to User objects or Global groups. PSOs override the Default Domain Policy, and conflicts are resolved using a Precedence value where the lowest number wins."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Linking password GPOs to sub-OUs | Misunderstanding GPO boundaries | Configure custom password rules using PSOs targeted to security groups. |
| Duplicate PSO precedence values | Unplanned deployments | Assign unique precedence numbers to all PSOs to avoid replication conflicts. |
| Short lockout reset windows | Resetting counts too quickly | Configure the reset counter window to at least 15 to 30 minutes to capture brute-force attempts. |

---

## Lab Exercise

**Objective:** Create a Password Settings Object (PSO) using ADAC/PowerShell, target it to a security group, and verify the effective policy on a test user.
**Time Required:** 20 minutes
**Environment Needed:** A Windows Server 2022 VM (Domain Controller).
**Pre-requisites:** Active Directory Domain Services active.

**Steps:**
1. Open PowerShell on the DC as Administrator.
2. Create the Test User and Security Group:
   ```powershell
   New-ADGroup -Name "G-Sec-Admins" -GroupScope Global -GroupCategory Security -Path "CN=Users,DC=lab,DC=local"
   New-ADUser -Name "Sec Admin" -SamAccountName "sadmin" -Path "CN=Users,DC=lab,DC=local" -Enabled $true
   Add-ADGroupMember -Identity "G-Sec-Admins" -Members "sadmin"
   ```
3. Create the PSO:
   ```powershell
   New-ADFineGrainedPasswordPolicy -Name "SecureAdminPolicy" -Precedence 5 -ComplexityEnabled $true -MinPasswordLength 14 -LockoutThreshold 3 -LockoutDuration "00:15:00" -LockoutObservationWindow "00:10:00" -Force
   ```
4. Assign the PSO to the Security Group:
   ```powershell
   Add-ADFineGrainedPasswordPolicySubject -Identity "SecureAdminPolicy" -Subjects "G-Sec-Admins"
   ```
5. Verification: Check the resultant password policy for the user:
   ```powershell
   Get-ADUserResultantPasswordPolicy -Identity "sadmin"
   ```
   - Expected output: `MinPasswordLength : 14`, confirming the PSO successfully applied.

**Success Criteria:** The custom PSO is created, targeted to the group, and verified as the effective policy on the test user.
**Common Failures:** Query returns empty if the user is not a direct or indirect member of the targeted group.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What are the main settings configured in a standard domain password policy?**
A: A standard domain password policy configures minimum password length, password complexity (requiring uppercase, lowercase, numbers, and special characters), password history (preventing reuse of previous passwords), and maximum/minimum password age.

**Q: What is the default account lockout threshold and how do you unlock an account?**
A: The default threshold varies by company but is typically set to 5 or 10 failed attempts. To unlock an account, I locate the user in ADUC, open their properties, go to the Account tab, check the unlock box, and click Apply.

### Intermediate (L2 Level)
**Q: What is a Password Settings Object (PSO), and why is it used?**
A: A PSO is a directory object used to define a Fine-Grained Password Policy. It is used to apply custom, stricter password and lockout settings to specific users or security groups (such as domain administrators) within the same domain, bypassing the Default Domain Policy.

### Advanced (L3/Senior Level)
**Q: A user belongs to two groups that have different PSOs applied. PSO-A has a precedence of 20 and requires 10 characters. PSO-B has a precedence of 10 and requires 16 characters. Which policy applies, and why?**
A:
- **Situation**: A user was subject to two conflicting PSOs due to overlapping group memberships.
- **Task**: I needed to determine the effective policy.
- **Action**: Active Directory resolves PSO conflicts using the Precedence value. The PSO with the lowest numerical Precedence value wins.
- **Result**: PSO-B applies because its precedence value of 10 is lower than PSO-A's value of 20, enforcing the 16-character minimum requirement.

### HR / Behavioral
**Q: Tell me about a time you had to enforce a security policy change that users complained about. How did you handle it?**
A: We increased the minimum password length to 14 characters. Users complained about difficulty remembering passwords. I conducted training sessions explaining how to create secure "passphrases" (e.g., using complete sentences with spaces), which improved security and reduced user frustration.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The Active Directory security configurations defining password complexity, length, age, and lockout rules.
> **Why**: Critical for securing user identities and preventing unauthorized access.
> **How**: Configure the Default Domain Policy for standard users, and deploy PSOs for privileged accounts.
> **Command**: `Get-ADUserResultantPasswordPolicy`
> **Interview Answer Starter**: "In my experience, managing password policies requires implementing Fine-Grained Password Policies to secure administrative credentials while maintaining standard rules for users..."

**Key Numbers to Remember:**
- Default Domain Policy link level: Domain Root
- Active Directory lockout threshold event ID: 4740
- Custom PSO class name: `msDS-PasswordSettings`

**3 Things Interviewer Wants to Hear:**
1. Using PSOs (Fine-Grained Password Policies) to apply different rules
2. Conflict resolution rules for PSOs (lowest precedence value wins)
3. Using audit logs (Event IDs 4724, 4740) to trace password resets and lockouts

---

## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/Users-and-Groups|Users and Groups]] — Target objects for password policies.
- [[03-Identity-and-Core-Services/06-Active-Directory/Group-Policy|Group Policy]] — Details GPO deployments.
- [[04-Cloud-and-Security/09-Security/MFA-and-Identity-Protection|MFA and Identity Protection]] — Details integration with multi-factor authentication.

---

## Tags
#desktop-support #active-directory #security #password-policy #L2 #interview-topic #lab-complete #daily-use

