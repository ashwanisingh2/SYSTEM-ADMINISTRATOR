---
tags: [desktop-support, m365, sspr, security, L2]
aliases: [sspr-guide, password-writeback, password-reset]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Self-Service Password Reset (SSPR)

---

## Concept Overview
- **What it is**: Self-Service Password Reset (SSPR) is a Microsoft Entra ID feature that allows users to reset their forgotten passwords without contacting the IT helpdesk. It verifies identity using pre-registered security methods (such as the Authenticator app, SMS, or security questions).
- **Why it matters for a support engineer**: Password resets account for up to 30% of helpdesk ticket volume. Implementing SSPR reduces ticket queues, empowers users to resolve lockouts immediately, and secures the reset workflow against social engineering.
- **Where you encounter this in real job**: Enrolling users in SSPR, troubleshooting SSPR verification blocks, monitoring password writeback errors on hybrid servers, and enabling the reset link on Windows login screens.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Guides users through registering SSPR contact methods, clears locked account flags, and walks users through the reset portal (`aka.ms/sspr`).
  - **L2**: Enrolls users in SSPR security groups, troubleshoots cloud-to-on-premises password sync failures, and configures Windows 10/11 login screen reset links.
  - **L3**: Configures SSPR tenant-wide policies, enables and maintains **Password Writeback** on Entra Connect servers, manages AD delegation permissions for the sync account, and audits SSPR security logs.

---

## Technical Deep Dive

### 1. SSPR Verification Methods
To reset their password, users must satisfy a configured number of authentication methods (typically 1 or 2):
- **Microsoft Authenticator**: Push notification or verification code (TOTP). Very secure.
- **Mobile/Office Phone**: Verification via SMS or voice call code.
- **Alternative Email**: A non-work personal email address that receives a verification code.
- **Security Questions**: pre-defined questions and answers (e.g., "What city were you born in?").
  - *Security Warning*: Security questions are vulnerable to social engineering and are deprecated or restricted in high-security environments.

### 2. Password Writeback (Hybrid Architecture)
For organizations using local Active Directory synced to the cloud, SSPR requires **Password Writeback**. This allows a password changed in the cloud to be written back to the local AD Domain Controller in real-time.

```
[User resets password at aka.ms/sspr] ---> [Microsoft Entra ID (Cloud)]
                                                     |
                                                     | (Encrypted over port 443 via Service Bus)
                                                     v
[Local Domain Controller (AD DS)] <--- [Entra Connect Sync Server (On-Premises)]
```

#### The Writeback Process:
1. The user requests a password reset at `passwordreset.microsoftonline.com`.
2. Entra ID verifies the user's identity via SSPR methods.
3. The new password is encrypted using a tenant-specific symmetric key.
4. Entra ID sends the encrypted password over a secure channel (HTTPS port 443 via Azure Service Bus) to the on-premises Entra Connect server.
5. The local Entra Connect agent decrypts the password and submits it to the local AD DC using the Windows ResetPassword API.
6. The DC validates the password against local password policies (complexity, history). If it passes, the password is changed in AD.
7. The change replicates to other DCs and is confirmed back to the cloud.

---

## Commands & Syntax

### PowerShell
SSPR and writeback configurations can be verified using the `Microsoft.Graph.Identity.SignIns` and local Active Directory modules.
```powershell
# Connect to Microsoft Graph with Directory Setting read/write permissions
Connect-MgGraph -Scopes "Directory.AccessAsUser.All", "User.ReadWrite.All"

# Query the status of SSPR configurations on your tenant
Get-MgDirectorySetting | Select-Object -ExpandProperty Values

# Verify the password writeback status on your on-premises Entra Connect Sync Server
Import-Module ADSync
Get-ADSyncAADPasswordResetConfiguration

# Enable password writeback on the Entra Connect server
Set-ADSyncAADPasswordResetConfiguration -Enable $true
```

### CMD / Run Box
```cmd
:: Test if the on-premises sync server can reach the Azure Service Bus endpoint for writeback
powershell -Command "Test-NetConnection -ComputerName 'servicebus.windows.net' -Port 443"
```

### GUI Path
- **SSPR Enablement**: Entra Admin Center -> **Protection** -> **Password reset** -> **Properties** (Select "All" or "Selected" groups).
- **Authentication Methods**: Entra Admin Center -> **Protection** -> **Password reset** -> **Authentication methods**.

### Important Registry Paths
- Enabling the "Reset password" link on the Windows 10/11 Lock Screen:
  ```
  HKLM\SOFTWARE\Policies\Microsoft\SecondaryAuthenticationFactor\
  (Create DWORD "AllowSelfServicePasswordReset" set to 1)
  ```

### Key Event IDs
On the on-premises Entra Connect server, check the **Application** log:

| Event ID | Source | Meaning | Action |
|----------|--------|---------|--------|
| **33005** | Directory Synchronization | Password writeback succeeded | Success event; user's local AD password was updated. |
| **33006** | Directory Synchronization | Password writeback failed | Check AD delegation permissions or local complexity rules. |
| **33001** | Directory Synchronization | Connection to Service Bus failed | Check proxy settings and firewall outbound port 443. |

---

## Real-World Scenarios

### Scenario 1: User Receives "Your organization hasn't enabled password reset for you" Error
**User Complaint:** A field technician attempts to log in to office.com from their tablet, forgets their password, and clicks "Forgot my password". They are blocked by a screen: *"Get back into your account. We're sorry, but your organization hasn't enabled password reset for you."*
**Your First 3 Checks:**
1. Check if the user is in the designated SSPR security group.
2. Verify the global SSPR enablement state in the Entra ID portal.
3. Check if the user has registered their SSPR authentication methods.
**Diagnosis Steps:**
1. Log in to the Entra Admin Center. Go to **Protection** -> **Password reset**.
2. Under **Properties**, note that SSPR is set to **Selected** (restricted to members of the group `G-SSPR-Users`).
3. Search for the technician's account in Entra ID. Check their group memberships.
   - The user is *not* a member of the `G-SSPR-Users` security group.
4. Because they are not in the targeted group, the cloud SSPR engine rejects their self-service request.
**Root Cause:** The user was not added to the security group authorized to use Self-Service Password Reset.
**Fix:**
1. Add the user to the `G-SSPR-Users` security group.
2. Wait 5-10 minutes for token replication.
3. Instruct the user to browse back to `passwordreset.microsoftonline.com`.
4. Walk the user through resetting their password using their mobile phone authentication code.
**Prevention:** Configure SSPR properties to **All** (enabling SSPR for the entire organization) or automate group membership for all new hires.
**Ticket Close Note:** "Added technician to SSPR authorization security group. Verified user could access the SSPR portal and reset their password. Closed."

### Scenario 2: SSPR Password Reset Fails with Writeback Error (Event ID 33006)
**User Complaint:** A finance clerk tries to reset their password using SSPR. The portal prompts for MFA, they enter the code and type a new password, but the reset fails with: *"We're sorry, but we cannot reset your password at this time. Please contact your administrator."*
**Your First 3 Checks:**
1. Check the Entra ID Audit logs for password reset failures.
2. RDP into the Entra Connect server and check the Application event logs.
3. Verify the Active Directory permissions delegated to the Entra Connect sync service account.
**Diagnosis Steps:**
1. In Entra ID Audit logs, locate the failed reset event.
   - Error: `PasswordWritebackError`.
2. Log into the on-premises Entra Connect server. Open Event Viewer -> Application log.
   - Find **Event ID 33006**: *"The password reset request failed. Details: Access Denied. The synchronization service account does not have sufficient permissions to reset passwords in Active Directory."*
3. Open ADUC on a DC. Locate the user account. Click Properties -> Security tab -> Advanced.
4. *Check inheritance*: The user belongs to a protected administrative group (e.g., Account Operators), which disabled permission inheritance. The Entra Connect sync account (`MSOL_xxxxxxxxxx`) lost its delegated "Reset Password" and "Change Password" rights on this user object.
**Root Cause:** The Entra Connect service account lacked delegated reset permissions on a user object that had permission inheritance disabled.
**Fix:**
1. In ADUC, delegate **Reset Password**, **Write lockoutTime**, and **Write pwdLastSet** permissions to the sync account (`MSOL_xxxx`) on the target Organizational Unit (OU).
2. For protected accounts, re-enable inheritance or remove the user from the protected group if they do not require administrative privileges.
3. Re-test SSPR on the web. The writeback succeeds, generating Event ID 33005 on the sync server.
**Prevention:** Regularly run scripts to audit sync account permissions across all OUs holding synchronized users.
**Ticket Close Note:** "Re-delegated Reset Password permissions to MSOL sync service account in Active Directory. Verified writeback now succeeds. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never assign SSPR security questions as the only authentication method for resets.
> - Security questions (e.g., "What was your first car?") are easily researched by cybercriminals on social media, allowing them to bypass password security easily. Combine them with push notifications or disable them.

> [!warning] Common Trap
> - Enabling SSPR writeback on the sync server, but forgetting to check the box "User must change password at next logon" compatibility.
> - If a user resets their password via SSPR, the password writeback updates local AD. However, if the local AD policy requires password changes on next login, the user's next PC login will prompt them to change it again, causing confusion.

> [!tip] Senior Engineer Tip
> - Integrate SSPR directly into the Windows 10/11 lock screen using Microsoft Intune or a registry group policy. This places a "Reset password" link right below the password field. Users who are locked out of their PCs can reset their password using their mobile phone directly from their locked workstation.

> [!success] Verification Steps
> - Run the PowerShell command `Get-ADSyncAADPasswordResetConfiguration` on the sync server to confirm writeback is active.
> - Check that the Application log shows Event ID 33005 when a test user resets their password via the cloud.

> [!question] Interview Alert
> - "Explain how SSPR Password Writeback works in a hybrid Active Directory environment."
> - Answer: "When a user resets their password in the cloud, Entra ID validates their MFA. The new password is encrypted and sent over a secure outbound HTTPS channel (port 443) via Azure Service Bus to the on-premises Entra Connect server. The sync agent decrypts the request and uses the Windows APIs to change the password in the local Active Directory, enforcing local complexity rules in real-time."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Forgetting to configure outbound port 443 for Service Bus | Firewalls block sync server traffic | Ensure the on-premises sync server can communicate outbound to `*.servicebus.windows.net`. |
| Leaving SSPR active for all users without writeback enabled | Hybrid sync is active but incomplete | Always enable SSPR Password Writeback simultaneously with cloud SSPR registration for hybrid tenants. |
| Using weak security questions | Creating easy-to-use recovery paths | Enforce Microsoft Authenticator app push notification as the primary verification factor. |

---

## Lab Exercise

**Objective:** Configure and enable Self-Service Password Reset (SSPR) for a test group, verify authentication methods, and inspect writeback settings.
**Time Required:** 30 minutes
**Environment Needed:** A Windows Server VM running Entra Connect Sync and access to a Microsoft 365 Developer tenant.
**Pre-requisites:** Entra Connect Sync active, and Microsoft Graph PowerShell module installed.

**Steps:**
1. Open a browser and log in to the **Microsoft Entra Admin Center** (`entra.microsoft.com`).
2. Go to **Protection** -> **Password reset**.
3. Under **Properties**, select **All** to enable SSPR for the entire tenant (or select your pilot security group). Click **Save**.
4. Go to **Authentication methods**. Check the boxes for **Email** and **Mobile phone**. Set the number of methods required to reset to `1`. Click **Save**.
5. RDP into your on-premises Entra Connect server. Open PowerShell as Administrator.
6. Check if writeback is currently enabled on the sync agent:
   ```powershell
   Import-Module ADSync
   Get-ADSyncAADPasswordResetConfiguration
   ```
7. If writeback is disabled, enable it:
   ```powershell
   Set-ADSyncAADPasswordResetConfiguration -Enable $true
   ```
8. Go to your local Domain Controller. Open Event Viewer -> Application log, filter by source `Directory Synchronization`, and look for Event ID `33005` to confirm successful communication.

**Success Criteria:** SSPR is enabled in the cloud, SSPR writeback is configured on the sync server, and communications to the service bus are verified.
**Common Failures:** The writeback command fails if the Entra Connect wizard is open in the background (wizard locks configuration databases).

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is Self-Service Password Reset (SSPR) and how does it help a user?**
A: SSPR is a feature that allows users to reset their forgotten password on the web (`aka.ms/sspr`) using registered security methods like a text code or the Authenticator app, without having to wait on the phone for a helpdesk technician.

**Q: Where do users go to register their SSPR security contact information?**
A: Users go to `aka.ms/setup` or `myaccount.microsoft.com`, log in with their corporate credentials, click Security Info, and add their phone number, alternative email, or register their Microsoft Authenticator app.

### Intermediate (L2 Level)
**Q: What is SSPR Password Writeback and why is it critical for hybrid AD networks?**
A: Password Writeback is a feature of Entra Connect Sync that takes a password reset in the cloud and writes it back to the local Active Directory Domain Controller in real-time. It is critical because if writeback is disabled, a user resetting their password in the cloud will only update their M365 profile, leaving their on-premises computer login with the old password.

**Q: How do you enable SSPR on the Windows 10/11 login screen?**
A: I configure it using Microsoft Intune or a Group Policy object. I enable the policy setting "Allow self-service password reset" under the Secondary Authentication Factor container. This writes a registry key that displays a "Reset password" link on the lock screen below the login field.

### Advanced (L3/Senior Level)
**Q: A hybrid user attempts to reset their password via SSPR. The reset fails in the cloud. You locate Event ID 33006 (Access Denied) in the sync server logs. Describe your resolution steps.**
A:
- **Situation**: User's SSPR writeback is failing due to permissions issues.
- **Task**: Identify why the sync account is blocked from resetting the user's password.
- **Action**: Event ID 33006 indicates the Entra Connect sync account (`MSOL_xxxx`) lacks "Reset Password" permissions on the user object in local AD. I open Active Directory Users and Computers, turn on Advanced Features, locate the user, and check their Security tab. I find the user has permission inheritance disabled, likely due to membership in a protected administrative group. I restore permission inheritance, or manually delegate "Reset Password" and write permissions to the MSOL account on the parent OU.
- **Result**: The permissions sync, and the user's next SSPR writeback attempt succeeds.

### HR / Behavioral
**Q: Tell me about a time you had to roll out a change that required user training. How did you ensure success?**
A: We decided to deploy SSPR to reduce helpdesk calls. Knowing that users might ignore emails about security registration, I worked with the communications team to create a 30-second video showing how to register. I set up a pilot group first to find common questions. When we enabled it tenant-wide, I set up a "help desk booth" in the cafeteria for two days to help users scan the QR codes. As a result, we reached a 95% registration rate within two weeks and cut password reset tickets by 40%.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The self-service cloud portal allowing users to reset their AD/Entra passwords.
> **Why**: Reduces helpdesk ticket volumes and secures the password reset workflow.
> **How**: Configure SSPR targets, set auth methods, and enable Password Writeback on the Entra Connect server.
> **Command**: `Get-ADSyncAADPasswordResetConfiguration` / `aka.ms/sspr`
> **Interview Answer Starter**: "SSPR enables secure password recovery using multi-factor signals. In hybrid networks, I configure Password Writeback so cloud resets update the local Active Directory..."

**Key Numbers to Remember:**
- Default sync port for writeback: TCP 443 (Outbound)
- Event ID for writeback success: `33005`
- Event ID for writeback failure: `33006`
- SSPR setup web address: `aka.ms/setup`

**3 Things Interviewer Wants to Hear:**
- SSPR writeback communicates over outbound HTTPS, eliminating inbound firewall rules
- Why to disable weak verification methods like security questions
- Resolving Event ID 33006 by checking AD permission delegation for the sync account

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]] — The core directory managing user accounts.
- [[03-Identity-and-Core-Services/06-Active-Directory/Password-Policies|Password Policies]] — The local rules validating the writeback password.
- [[04-Cloud-and-Security/07-Microsoft-365/MFA|MFA]] — The authentication platform utilized by SSPR.

---

## Tags
#desktop-support #m365 #sspr #security #L2 #interview-topic #lab-complete #daily-use

