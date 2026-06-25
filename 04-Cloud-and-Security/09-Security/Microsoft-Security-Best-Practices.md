---
tags: [desktop-support, security, best-practices, compliance, L2]
aliases: [secure-score, security-baselines, cis-benchmarks]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Microsoft Security Best Practices

---

## Concept Overview
- **What it is**: Microsoft Security Best Practices are standard configurations recommended to secure Windows endpoints and cloud environments. These are evaluated using **Microsoft Secure Score** (an audit percentage indicating security posture), and implemented using **Security Baselines** (pre-configured groups of Microsoft settings) and **CIS Benchmarks** (independent hardening standards).
- **Why it matters for a support engineer**: A support engineer must know how to maintain workstation security compliance. This requires applying standard Group Policy (GPO) baselines, auditing devices for configuration drift, and resolving user issues caused by hardened security rules.
- **Where you encounter this in real job**: Reviewing the Secure Score dashboard for recommendations, deploying Security Baselines in Intune, auditing local security settings using `secedit`, and hardening systems prior to compliance audits.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Verifies basic device compliance, checks local security center settings, and assists users with standard password resets.
  - **L2**: Audits local policy drift using command-line diagnostic tools (`secedit`), imports security templates, and resolves application blocks caused by security hardening.
  - **L3**: Configures global Microsoft Security Baselines, designs CIS Benchmark compliance frameworks, manages Microsoft Defender policy profiles, and presents security posture audits to leadership.

---

## Technical Deep Dive

### 1. Microsoft Secure Score
Microsoft Secure Score evaluates your tenant across four key categories: Identity, Devices, Applications, and Data. It provides:
- **Improvement Actions**: Recommendations to raise your score (e.g., "Require MFA for administrative roles").
- **Current Score vs. Benchmark**: Compares your organization's security posture to similar companies.
- *Tip*: Maximizing Secure Score is a balance; you must avoid turning on settings that break critical business applications.

### 2. Microsoft Security Baselines
Security Baselines are pre-configured packages of Group Policy settings designed by Microsoft security teams. Instead of configuring hundreds of individual registry settings, admins deploy the baseline:
- **Baseline Components**: Includes recommended settings for BitLocker, Credential Guard, Windows Defender, User Account Control (UAC), and Local Firewall rules.
- **Intune Baselines**: In cloud environments, these are deployed in a single click under the Endpoint Security blade, and are updated automatically as Microsoft updates its security recommendations.

### 3. CIS (Center for Internet Security) Benchmarks
CIS Benchmarks are globally recognized, vendor-neutral security guidelines developed by industry experts:
- **Level 1 Profile**: Baseline security recommendations. Easy to implement with minimal impact on business productivity.
- **Level 2 Profile**: High security, defense-in-depth hardening. Intended for highly secure environments (finance, government) where security outweighs convenience. May cause minor user friction or application incompatibilities.

---

## Commands & Syntax

### CMD / Run Box (`secedit` tool)
`secedit` is the command-line utility used to configure, analyze, and audit local security policies on a Windows client.
```cmd
:: Export the current local security policy settings to a text file for analysis
secedit /export /cfg C:\Temp\local_policy.inf

:: Analyze the current system configuration against a template policy database
secedit /analyze /db C:\Windows\security\database\secedit.sdb /cfg C:\Temp\security_template.inf /log C:\Temp\analysis_log.txt

:: Import and apply a security template configuration to the local machine immediately
secedit /configure /db C:\Windows\security\database\secedit.sdb /cfg C:\Temp\security_template.inf /log C:\Temp\configure_log.txt

:: Force the system to reapply all local security configurations immediately
secedit /configure /db C:\Windows\security\database\secedit.sdb /apply
```

### PowerShell
```powershell
# Check if Credential Guard is active (a key Microsoft Security Baseline requirement)
(Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard).SecurityServicesRunning

# Query local password complexity configuration using secedit export parsing
secedit /export /cfg C:\Temp\sec.inf | Out-Null
Select-String -Path C:\Temp\sec.inf -Pattern "PasswordComplexity"
```

### GUI Path
- **Secure Score Dashboard**: Go to **security.microsoft.com** -> **Microsoft Secure Score**.
- **Intune Baselines**: Go to **intune.microsoft.com** -> **Endpoint security** -> **Security baselines** -> Select target baseline (e.g., Windows 10 and later Security Baseline).

---

## Real-World Scenarios

### Scenario 1: Auditing Workstation for Local Security Policy Drift
**User Complaint:** A security auditor flags: *"Our company policy requires all laptops to enforce a 15-minute screen lock timeout. Workstation 'DESKTOP-FIN04' has been identified as non-compliant. We need to verify if its local security policy has deviated from our baseline."*
**Your First 3 Checks:**
1. Check the active Screen Saver lock settings in the local registry.
2. Export the current local security policy configurations using `secedit`.
3. Compare the exported policy against the corporate security template.
**Diagnosis Steps:**
1. Open Command Prompt as Administrator on `DESKTOP-FIN04`.
2. Export the active security settings:
   `secedit /export /cfg C:\Temp\policy_export.txt`
3. Open `C:\Temp\policy_export.txt` in Notepad. Search for the screen timeout or account lockout thresholds:
   - Check value: `ScreenSaverGracePeriod` and `ScreenSaverTimeout`.
   - Find: `ScreenSaverTimeout = 0` (Disabled).
4. The local user (who had local administrator access) modified their local registry to disable the automatic screen lock so they could watch long compile screens without logging back in.
**Root Cause:** Local policy drift caused by a local administrator manual registry modification.
**Fix:**
1. Re-apply the corporate security baseline:
   `gpupdate /force`
2. If the machine is off-network and cannot reach a Domain Controller, import the standard corporate configuration template:
   `secedit /configure /db C:\Windows\security\database\secedit.sdb /cfg C:\Temp\Corporate_Baseline.inf /log C:\Temp\fix_log.txt`
3. Verify that the timeout settings in the registry restore to `900` seconds (15 minutes).
**Prevention:** Remove local administrator rights from standard user profiles to block manual policy modifications.
**Ticket Close Note:** "Exported policy using secedit. Identified manual registry drift. Reapplied corporate baseline template. Verified screen lock active. Closed."

### Scenario 2: Application Fails to Run After Deploying Intune Security Baseline
**User Complaint:** An engineering lead complains: *"Our main design software 'CadEngine.exe' was working yesterday. Today, it fails to open, throwing a Windows error: 'The application was unable to start correctly'. This began after IT pushed a security update last night."*
**Your First 3 Checks:**
1. Check the recently applied policies in Microsoft Intune or the local Event Viewer.
2. Verify if the application block is caused by an **Attack Surface Reduction (ASR)** rule or **Credential Guard** conflict.
3. Review the Windows Defender Exploit Protection logs.
**Diagnosis Steps:**
1. Open Event Viewer on the client PC. Go to `Applications and Services Logs -> Microsoft -> Windows -> Windows Defender -> Operational`.
2. Locate the warning logs matching the application failure timestamp.
   - Find **Event ID 1121**: *"Windows Defender Exploit Guard blocked an audit event. Process CadEngine.exe was blocked from running due to: Block executable files from running unless they meet a prevalence, age, or trusted list criterion."*
3. The newly applied **Windows 10/11 Security Baseline** in Intune activated the "Block untrusted executables" ASR rule. The custom-compiled design software was blocked because it was not digitally signed or listed in Microsoft's database.
**Root Cause:** A newly deployed Microsoft Security Baseline ASR rule blocked an unsigned application process.
**Fix:**
1. Go to the Intune Endpoint Security portal. Go to **Security baselines** -> **Windows 10 and later Security Baseline**.
2. Select the active profile -> **Properties** -> **Configuration settings** -> **Attack Surface Reduction**.
3. Locate the rule: `Block executable files from running unless they meet a prevalence, age, or trusted list criterion`.
4. *Decision*: Instead of disabling the rule for everyone, add a specific exclusion path for the Cad folder (`C:\Program Files\CadEngine\`) in the baseline properties, or set the rule to **Audit** mode temporarily while the developer signs the binary.
5. Apply the baseline update and force a device sync. The software launches successfully.
**Prevention:** Test all security baseline profiles on a pilot group before deploying to production.
**Ticket Close Note:** "Identified ASR rule block in Defender logs. Added folder exclusion path in Intune Security Baseline. App runs successfully. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never deploy a Level 2 CIS Benchmark or a fresh Microsoft Security Baseline directly to production machines without a testing phase.
> - Level 2 baselines configure strict settings (like disabling legacy protocols, forcing smartcard logins, and blocking unsigned drivers) that will break legacy apps, printer connections, and internal networks if not tested first.

> [!warning] Common Trap
> - Assuming that raising your Microsoft Secure Score to 100% is the goal of IT security.
> - Secure Score is an indicator, not a absolute target. Configuring all recommendations (such as blocking all external sharing or enforcing weekly password changes) will block business productivity, leading users to use shadow IT workarounds.

> [!tip] Senior Engineer Tip
> - When applying local security policies on non-domain joined machines (such as standalone kiosk PCs), use the Microsoft utility **LGPO.exe** (Local Group Policy Object utility). It allows you to script and apply local policy baselines via a single command, avoiding manual GUI setup.

> [!success] Verification Steps
> - Run `secedit /compare` to check if a local configuration matches the secure baseline template.
> - Verify in the Intune dashboard that the device status displays `Succeeded` under the Security Baseline profile deployment.

> [!question] Interview Alert
> - "What is a Microsoft Security Baseline, and why should you use it?"
> - Answer: "A Microsoft Security Baseline is a pre-configured group of security settings recommended by Microsoft engineers. It standardizes settings for BitLocker, Windows Defender, local firewalls, and UAC. Using baselines ensures your systems adhere to industry-vetted security profiles out-of-the-box, saving time and preventing configuration gaps."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Manually configuring GPO settings one-by-one | Inefficient administrative habits | Download and import Microsoft's official Security Baseline GPO packages. |
| Ignoring application logs after a baseline push | Attributing errors to software bugs | Check Windows Defender Operational logs (Event 1121) for ASR block warnings. |
| Hardening development environments to CIS Level 2 | Overzealous security enforcement | Keep developer environments at CIS Level 1 to avoid blocking custom script executions. |

---

## Lab Exercise

**Objective:** Use `secedit` to export the current local security policy on Windows, locate the password history setting, simulate a policy modification, and verify configuration.
**Time Required:** 30 minutes
**Environment Needed:** A Windows 10/11 client VM.
**Pre-requisites:** Local Administrator access.

**Steps:**
1. Open Command Prompt as Administrator.
2. Export your current security configuration:
   ```cmd
   secedit /export /cfg C:\Windows\Temp\sec_config.inf
   ```
3. Open the exported `sec_config.inf` file in Notepad:
   ```cmd
   notepad C:\Windows\Temp\sec_config.inf
   ```
4. Scroll down to the `[System Access]` section. Locate the setting **PasswordHistorySize**:
   - Check the value (e.g., `PasswordHistorySize = 24`).
5. Change the value in Notepad to `12` (for test purposes) and save the file.
6. Configure the local database using the modified file:
   ```cmd
   secedit /configure /db C:\Windows\security\database\sec_test.sdb /cfg C:\Windows\Temp\sec_config.inf /log C:\Windows\Temp\sec_log.txt
   ```
7. Verification: Export the active settings again and check the value to confirm the change applied:
   ```cmd
   secedit /export /cfg C:\Windows\Temp\sec_verify.inf
   findstr "PasswordHistorySize" C:\Windows\Temp\sec_verify.inf
   ```
   - Confirm it outputs `PasswordHistorySize = 12`.

**Success Criteria:** The local security policy is exported, successfully modified, reconfigured via `secedit`, and verified.
**Common Failures:** The configure command fails if the specified `.sdb` database file path is locked by another MMC snap-in console.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is Microsoft Secure Score and how do you use it?**
A: Microsoft Secure Score is a security measurement dashboard in the Microsoft Defender portal. It gives a percentage rating of the organization's security posture and provides a list of recommended actions (like enabling MFA) to help improve security.

**Q: Where in Windows do you go to view local security policy settings like password rules?**
A: I press `Win + R`, type `secpol.msc` and click OK. This opens the Local Security Policy console, where I can navigate to Account Policies -> Password Policy.

### Intermediate (L2 Level)
**Q: What is the difference between CIS Benchmark Level 1 and Level 2 profiles?**
A: CIS Level 1 profile settings are designed to provide essential security hardening that can be implemented with minimal impact on user workflows and application compatibility. CIS Level 2 profile settings provide higher security hardening (defense-in-depth) but can cause minor user disruption or block custom-developed business software.

**Q: How do you use the 'secedit' command to troubleshoot local policy configurations?**
A: I open Command Prompt as Administrator and run `secedit /export /cfg C:\Temp\policy.txt`. This exports all active password rules, account lockouts, user rights assignments, and registry values to a text file, allowing me to search for discrepancies or deviations from our security template.

### Advanced (L3/Senior Level)
**Q: A compliance audit reveals that our servers have security configuration drift. How do you automate baseline compliance monitoring?**
A:
- **Situation**: Servers experiencing security policy drift over time.
- **Task**: Implement automated baseline monitoring and enforcement.
- **Action**: First, I build a standard security baseline template using the **LGPO** tool. I configure a scheduled task on our servers that runs a script daily. The script executes `secedit /analyze` comparing the running system to the template `sec_baseline.inf`. If deviations are found, the script triggers an alert to our monitoring system and executes `secedit /configure` to re-apply the baseline automatically. In cloud environments, I utilize **Azure Policy** guest configuration rules to automate this drift auditing.
- **Result**: Server policy configuration drift is automatically detected and remediated, maintaining 100% audit compliance.

### HR / Behavioral
**Q: Describe a time you had to implement a security policy that made users unhappy. How did you handle the situation?**
A: We implemented an Intune Security Baseline that locked computers after 10 minutes of inactivity. Our call center agents complained it logged them out between customer calls, slowing their metrics. I didn't ignore their frustration. I went to the call center floor, observed their workflow, and configured a dynamic exclusion in the policy: if an agent had their headset active or telephony software open, the idle timer was extended to 15 minutes. This maintained device security while addressing their core daily performance concerns.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The standard configuration baselines (Intune, GPO, CIS) used to harden Windows and Microsoft 365 environments.
> **Why**: Standardizes security, prevents configuration gaps, and meets audit compliance.
> **How**: Review Secure Score recommendation actions, deploy GPO/Intune Security Baselines, and audit local systems using `secedit`.
> **Command**: `secedit /export` / `secedit /configure` / `gpupdate /force`
> **Interview Answer Starter**: "To implement security best practices, I deploy Microsoft Security Baselines in Intune, verify device settings against CIS Benchmarks, and use secedit to audit..."

**Key Numbers to Remember:**
- Standard password history size recommendation: 24 passwords
- Default CIS Level 1 lock screen timeout: 15 minutes
- Command-line tool for local security databases: `secedit.exe`
- SECEDIT configuration file format: `.inf`

**3 Things Interviewer Wants to Hear:**
- Secure Score is a guide for prioritization, not a race to 100% at the cost of operations
- Microsoft Security Baselines simplify deployment of vetted configurations in Intune
- Using `secedit` to analyze and configure offline Windows devices

---

## Related Notes
- [[04-Cloud-and-Security/09-Security/CIA-Triad-and-Zero-Trust|CIA Triad and Zero Trust]] — The security principles behind baseline configurations.
- [[03-Identity-and-Core-Services/06-Active-Directory/Group-Policy|Group Policy]] — Outlines the primary deployment engine for Windows baselines.
- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]] — The cloud policy gatekeeper enforcing compliance standards.

---

## Tags
#desktop-support #security #best-practices #compliance #L2 #interview-topic #lab-complete #daily-use

