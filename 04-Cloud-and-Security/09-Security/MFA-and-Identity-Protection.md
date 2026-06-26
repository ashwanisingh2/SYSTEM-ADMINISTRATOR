---
tags: [desktop-support, security, threat-protection, L2]
aliases: [mfa-and-identity-protection, mfa-and-identity-protection]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# MFA and Identity Protection

---

---
## Concept Overview
- **What it is**: Microsoft Entra ID Identity Protection is a cloud service that automates the detection, investigation, and remediation of identity-based risks. It analyzes trillions of daily sign-in signals to flag anomalous behavior, categorizing threats into **User Risk** (compromised credentials) and **Sign-in Risk** (suspicious session characteristics).
- **Why it matters for a support engineer**: Attackers constantly target corporate logins. A support engineer must understand identity protection metrics to troubleshoot risk-based account lockouts, investigate "impossible travel" alerts, and clear risk flags to restore user access.
- **Where you encounter this in real job**: Resolving tickets where a user is blocked because their location changed, resetting accounts flagged with leaked credentials, and analyzing risk detections in the Entra ID Portal.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Confirms user identity, resets passwords for compromised accounts, and clears basic user risk flags in the portal.
  - **L2**: Audits sign-in logs for risk events (like unfamiliar locations or suspicious IPs), investigates risk detection details, and guides users on registering secure authentication methods.
  - **L3**: Configures tenant-wide User and Sign-in Risk Policies, integrates logs with SIEM systems (Microsoft Sentinel), and designs automated self-remediation workflows.

---

---
## Technical Deep Dive
### 1. User Risk vs. Sign-in Risk
Microsoft Entra ID divides security anomalies into two distinct risk classifications:

| Metric | User Risk | Sign-in Risk |
|---|---|---|
| **Definition** | The probability that a user's *identity* has been compromised. | The probability that a *specific sign-in attempt* is not authorized. |
| **Trigger Time** | Calculated **offline** using background data analytics. | Calculated **real-time** during the connection phase. |
| **Examples** | - Leaked credentials (username/password found on dark web).<br>- Anomalous user activity patterns. | - **Impossible travel** (sign-in from two locations too fast).<br>- Anonymous IP address (Tor browser, VPN).<br>- Unfamiliar sign-in properties. |
| **Typical Remediation** | Force **password change** via secure SSPR. | Enforce **MFA challenge** to verify user presence. |

### 2. Risk Detection Types & Analysis
Entra ID uses machine learning to detect several threat signatures:
- **Impossible Travel**: Flagged when a user signs in from New York, and then signs in from London 30 minutes later. The physical travel speed required between logs is impossible.
- **Unfamiliar Sign-in Properties**: Flagged when the user connects using a new browser, device, or location not matching their historical profile.
- **Malware Linked IP Address**: Flagged when the sign-in originates from an IP range known to host active botnet command servers.

### 3. Risk-Based Conditional Access Flow
Instead of prompting users for MFA constantly, organizations enforce risk-based policies:

```
[Sign-In Attempt] --> [Identity Protection Engine]
                             |
                  +----------+----------+
                  |                     |
         [Low Sign-In Risk]    [Medium/High Sign-In Risk]
                  |                     |
        (Allow standard login)          +---> [Require MFA Challenge]
                                                    |
                                                    +---> (Passed) ---> [Access Granted]
                                                    +---> (Failed) ---> [Access Blocked]
```

---


## Real-World Scenarios

### Scenario 1: Remote Worker Blocked due to "Impossible Travel" Risk Detection
**User Complaint:** A field consultant calls: *"I am trying to log in to my email from my hotel in Chicago. I type my password, but it blocks me instantly, stating suspicious activity was detected and I must contact my admin. I was working from our headquarter office in Dallas this morning."*
**Your First 3 Checks:**
1. Check the Entra ID Sign-in logs and Risky Sign-ins logs for the consultant.
2. Compare the timestamp and geographic locations of the consultant's latest logins.
3. Verify if the user completed an MFA challenge or was blocked outright.
**Diagnosis Steps:**
1. Log in to the Entra Portal. Go to **Protection** -> **Identity Protection** -> **Risky sign-ins**.
2. Locate the consultant's log entry.
   - Status: `Sign-in blocked`.
   - Risk Level: `High`.
   - Risk Detection: `Impossible travel`.
3. Check details:
   - Log 1: `9:00 AM` from Dallas, TX (Corporate IP: `198.51.100.10`).
   - Log 2: `9:25 AM` from Chicago, IL (Hotel Wi-Fi IP: `203.0.113.45`).
4. *Calculate speed*: Travel between Dallas and Chicago in 25 minutes is physically impossible.
5. *Investigation*: The user admits they logged into their corporate PC in Dallas, hopped on a flight, and turned on their corporate laptop at Chicago. However, they also had a virtual machine running a script in our Dallas server room under their account, which continued sending sign-in logs, triggering the impossible travel calculation.
**Root Cause:** Simultaneous active sign-ins from two physically distant geographic locations triggered the impossible travel risk engine.
**Fix:**
1. Confirm the user's identity via phone authentication.
2. In the Entra Portal, go to **Risky users** or **Risky sign-ins**. Select the user's log, and click **Dismiss user risk**.
3. Instruct the user to restart their browser and log in again.
4. The user completes their MFA challenge and gains access.
**Prevention:** Advise developers and power users to run service scripts using dedicated Service Principals rather than personal user accounts.
**Ticket Close Note:** "Dismissed false positive impossible travel risk. Verified user identity. User successfully logged in. Closed."

### Scenario 2: Executive Account Flagged with "Leaked Credentials" User Risk
**User Complaint:** A security administrator alerts the helpdesk: *"An automated alert indicates our VP of Finance has been flagged as a High User Risk in Microsoft Entra ID due to leaked credentials. We need immediate remediation to secure their account."*
**Your First 3 Checks:**
1. Check the Risky Users blade to confirm the risk detail type.
2. Verify if the user is blocked or if self-remediation (SSPR) is configured.
3. Audit the user's active login locations for unauthorized sessions.
**Diagnosis Steps:**
1. Navigate to **Identity Protection** -> **Risky users**. Locate the VP.
   - Risk Level: `High`.
   - Risk Detail: `Leaked credentials`.
2. *How did it happen?* Microsoft's background crawlers identified the VP's corporate email address and password hash in a public dark web dump originating from a breach on a third-party retail website where the VP reused their corporate password.
3. Because the user risk is high, our tenant-wide **User Risk Policy** has blocked active access until a secure password change is completed.
**Root Cause:** Password reuse on a compromised external website led to corporate credential leakage.
**Fix:**
1. Contact the VP immediately via corporate phone. Explain the security issue.
2. Revoke the VP's active sign-in sessions to boot any potential attackers off:
   `Revoke-MgUserSignInSession -UserId "vp@company.com"`
3. Instruct the VP to navigate to `passwordreset.microsoftonline.com` (SSPR) to reset their password.
4. The SSPR reset writes back to on-premises AD, syncs to Entra, and automatically closes the user risk flag in Identity Protection, changing their status to `Remediated`.
**Prevention:** Implement a strict password policy banning common passwords and reuse, and run regular security awareness training.
**Ticket Close Note:** "Revoked active sessions. VP completed password reset via SSPR. Risk status changed to Remediated. Closed."

---


## Critical Points

> [!danger] Never Do This
> - Never click "Dismiss user risk" without verifying the root cause of the risk flag.
> - Clicking "Dismiss" deletes the warning and tells the machine learning engine the activity was safe. If the user was actually compromised, dismissing the risk allows the attacker to remain active inside your network silently.

> [!warning] Common Trap
> - Assuming that standard MFA registration protects against all leaked credential attacks.
> - If an attacker has the correct password and triggers MFA, they may try to bypass it via push notification spam (MFA fatigue). Always combine MFA with **User Risk Policies** that force password changes when credentials are leaked.

> [!tip] Senior Engineer Tip
> - Configure **Self-Remediation** policies for both User and Sign-in risk. If a user is flagged with Medium Sign-in risk, allow them to unblock themselves by successfully completing an MFA prompt. If flagged with High User risk, allow them to unblock themselves by resetting their password via SSPR. This maintains security while reducing helpdesk ticket volumes.

> [!success] Verification Steps
> - Run: `Get-MgRiskyUser -Filter "UserPrincipalName eq 'user@company.com'"` to confirm their risk state has changed to `cleared` or `remediated`.
> - Check the Entra ID audit logs to verify the risk transition status.

> [!question] Interview Alert
> - "What is the difference between User Risk and Sign-in Risk in Entra ID Identity Protection?"
> - Answer: "User Risk represents the probability that the user's account identity itself has been compromised (e.g., leaked credentials found on the dark web). Sign-in Risk represents the probability that a specific sign-in attempt is unauthorized (e.g., impossible travel or anonymous IP). User risk is calculated offline, while sign-in risk is evaluated in real-time."

---


## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Dismissing risk flags to quickly close helpdesk tickets | Impatience and lack of audit checks | Always contact the user to verify login locations before dismissing flags. |
| Forgetting to revoke active sessions after a compromise | Assuming password change ends current sessions | Always run `Revoke-MgUserSignInSession` to force-expire active hacker tokens. |
| Permitting SMS as the only MFA for risk remediation | SMS is vulnerable to intercepts | Enforce Microsoft Authenticator or FIDO2 keys for secure self-remediation. |

---


## Tags
#desktop-support #security #mfa #identity-protection #L2 #interview-topic #lab-complete #daily-use

---
## Step-by-Step Lab
**Objective:** Connect to Microsoft Graph via PowerShell, query all risky users in the tenant, filter sign-in logs for active risk events, and simulate risk flag dismissal.
**Time Required:** 20 minutes
**Environment Needed:** A computer with internet access and a Microsoft 365 Developer tenant.
**Pre-requisites:** The Microsoft Graph PowerShell SDK installed, and global admin credentials.

**Steps:**
1. Open PowerShell. Connect to Microsoft Graph:
   ```powershell
   Connect-MgGraph -Scopes "IdentityRiskyUser.ReadWrite.All", "Directory.Read.All"
   ```
2. Retrieve all users currently flagged as risky:
   ```powershell
   Get-MgRiskyUser | Select-Object UserPrincipalName, RiskLevel, RiskState, RiskDetail
   ```
3. Find sign-ins that were flagged with a risk level of High or Medium:
   ```powershell
   Get-MgSignIn -Filter "RiskLevelDuringSignIn eq 'high' or RiskLevelDuringSignIn eq 'medium'" -Top 5 | Select-Object UserDisplayName, IPAddress, RiskLevelDuringSignIn
   ```
4. Simulate resolving a false positive risk flag for a test user:
   - Identify the User Object ID from the query.
   - Run the dismissal command:
   ```powershell
   Confirm-MgRiskyUserCompromised -UserIds @("your-test-user-guid")
   ```
5. Verification: Re-run the query from step 2 and confirm the user's risk state is no longer active.

**Success Criteria:** Risky users are queried, sign-in logs filtered for risk level, and the test user's risk state is cleared successfully via PowerShell.
**Common Failures:** The command fails if the test tenant does not have Microsoft Entra ID P2 licensing, which is required to run Identity Protection cmdlets.

---

---
## Cheat Sheet / Quick Reference
### PowerShell
Identity Protection management uses the `Microsoft.Graph.Identity.SignIns` and `Microsoft.Graph.Users` modules.
```powershell
# Connect to Microsoft Graph with Security Administration scopes
Connect-MgGraph -Scopes "IdentityRiskyUser.ReadWrite.All", "Directory.Read.All"

# Query the top 10 active risky users in the tenant
Get-MgRiskyUser -Top 10 | Select-Object Id, DisplayName, UserPrincipalName, RiskState, RiskLevel, RiskDetail

# Dismiss a risk flag for a user after confirming the activity was legitimate (e.g., user on vacation)
Confirm-MgRiskyUserCompromised -UserIds @("user-object-id-12345")

# List sign-ins flagged with active risk states
Get-MgSignIn -Filter "RiskState eq 'atRisk'" -Top 5 | Select-Object Id, UserDisplayName, CreatedDateTime, IPAddress, RiskLevelDuringSignIn
```

### CMD / Run Box
```cmd
:: Check routing hops to verify public egress IPs
tracert login.microsoftonline.com
```

### GUI Path
> Open browser -> Go to **entra.microsoft.com** -> **Protection** -> **Identity Protection** -> **Risky users** or **Risky sign-ins**.
> To configure: Go to **Identity Protection** -> **User risk policy** or **Sign-in risk policy**.

---

> [!info] 60-Second Summary
> **What**: Microsoft's identity security engine that automates threat detection and remediation.
> **Why**: Protects corporate identities from credential theft, brute-force attacks, and session hijacks.
> **How**: Categorize risks into User Risk and Sign-in Risk, and enforce self-remediation via Conditional Access.
> **Command**: `Confirm-MgRiskyUserCompromised` / `Get-MgRiskyUser`
> **Interview Answer Starter**: "To manage identity security, I leverage Microsoft Entra ID Identity Protection to evaluate User and Sign-in risks, automating unblocks using SSPR..."

**Key Numbers to Remember:**
- Default period to retain risk log details: 30 days
- Minimum license SKU required for Identity Protection: Entra ID P2
- Port required for connection: TCP 443 (HTTPS)
- SSPR reset unblocks user risk state: Yes (Remediated)

**3 Things Interviewer Wants to Hear:**
- User Risk is offline (identity compromised), Sign-in Risk is real-time (session compromised)
- Enabling self-remediation (MFA/SSPR) reduces helpdesk tickets while keeping security high
- Always revoke active sessions (`Revoke-MgUserSignInSession`) during account remediation

---

---
## Troubleshooting
| Problem | Cause | Fix | Command |
|---|---|---|---|
| Service connection timeout | Network firewall or routing blocking traffic | Check network route and enable target ports on firewall | `ping -c 4 <ip>` / `nc -zv <ip> <port>` |
| Access Denied error | User account lacks permissions or invalid credentials | Verify account access permissions or reset password | N/A |
| Resource not found | Object or path is misspelled or deleted | Verify spelling of target path or query active objects | N/A |

---
## Interview Questions
### Basic (L1 Level)
**Q: What is "Impossible Travel" in Microsoft Entra ID?**
A: Impossible travel is a security risk detection that occurs when a user logs in from two geographically distant locations within a time frame that is physically impossible to travel between (e.g., logging in from New York and then London 15 minutes later).

**Q: Where do you go in the Microsoft Entra Admin Center to check if a user is blocked by security risk flags?**
A: I go to the **Microsoft Entra Admin Center**, navigate to Protection, open **Identity Protection**, and check the **Risky users** or **Risky sign-ins** dashboards.

### Intermediate (L2 Level)
**Q: What is the difference between offline risk detection and real-time risk detection?**
A: Real-time risk detection occurs during the active sign-in attempt (e.g., anonymous IP or unfamiliar sign-in properties) and can be blocked instantly by Conditional Access. Offline risk detection is calculated in the background by data processors (e.g., discovering leaked credentials on the dark web) and flags the user account after the event, triggering a state change.

**Q: How does SSPR (Self-Service Password Reset) help automate risk remediation?**
A: By configuring a User Risk Policy, we can state: "If a user's risk level is High, then require a password change." If the user is flagged with leaked credentials, they are blocked until they successfully complete SSPR. Once they verify their identity via MFA and reset their password, Entra ID automatically marks the risk as remediated, unblocking their account without helpdesk intervention.

### Advanced (L3/Senior Level)
**Q: A critical user account is repeatedly flagged with "Suspicious IP address" risk events, but the user claims they are only working from home. How do you investigate this?**
A:
- **Situation**: User account repeatedly flagged with suspicious IP risk events.
- **Task**: Determine if the sign-ins are malicious or false positives.
- **Action**: I review the Entra ID Sign-in logs, filtering by the user's UPN and focusing on the IP addresses. I run a WHOIS lookup on the flagged IPs to check the ISP. I find the IPs belong to a residential broadband provider but are flagged because the user is running a third-party commercial VPN client or web accelerator that routes their traffic through anonymous proxy servers.
- **Result**: I instruct the user to bypass the VPN for corporate access, resolving the suspicious IP alerts.

### HR / Behavioral
**Q: Describe a time you had to investigate a potential security breach. How did you react?**
A: I received an alert indicating our CEO's account had flagged a High User Risk for leaked credentials at 2:00 AM. I did not wait for morning. I immediately logged in, revoked all active login sessions to kick out any active hackers, and disabled the account temporarily as a precaution. I then called the CEO to explain. I helped them complete an SSPR password reset and verified no unauthorized actions were logged in the audit trails. My prompt action secured the company's identity framework.

---

---
## Seedha Simple Mein
*Seedha simple mein: MFA-and-Identity-Protection ke bare mein seekhta hai. Yeh security infrastructure aur system settings ko properly implement karne aur support tickets ko runbooks ke help se standard templates me clear karne me help karta hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/MFA|MFA]] — The verification tool used to resolve Sign-in risk challenges.
- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]] — The rules engine that evaluates risk levels.
- [[04-Cloud-and-Security/09-Security/CIA-Triad-and-Zero-Trust|CIA Triad and Zero Trust]] — Explains the core identity security theory.

---
