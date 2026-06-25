---
tags: [desktop-support, active-directory, group-policy, GPO, L2]
aliases: [gpo-management, group-policy-objects]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Group Policy

---

## Concept Overview
- **What it is**: Group Policy is an Active Directory feature that allows administrators to define and enforce configurations for users and computers across the domain using Group Policy Objects (GPOs).
- **Why it matters for a support engineer**: GPOs control the security settings, software deployments, and system configurations of all corporate endpoints. A support engineer must know how to trace policy applications, resolve conflicts, and troubleshoot client-side update failures.
- **Where you encounter this in real job**: Troubleshooting why a security setting failed to apply to a workstation, generating GPO diagnostic reports (`gpresult`), mapping network drives, and managing loopback processing policies.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Runs basic client diagnostics (`gpupdate /force`, `gpresult /r`), and checks local policy settings.
  - **L2**: Creates and links GPOs to OUs, configures security filtering, and analyzes HTML policy reports (`gpresult /h`).
  - **L3**: Configures WMI filters, designs loopback processing policies, audits GPO processing times (slow logins), and manages the GPO backup registry.

---

## Technical Deep Dive

### 1. GPO Architecture & Client-Side Extensions (CSEs)
A GPO consists of two parts:
- **Group Policy Container (GPC)**: The Active Directory object that stores GPO properties, status, and version numbers (replicated via AD database replication).
- **Group Policy Template (GPT)**: The physical folder structure stored in the `SYSVOL` share (`\\domain.local\SYSVOL\domain.local\Policies\{GUID}\`) that contains the actual policy registry settings and scripts (replicated via DFS-R).
- **Client-Side Extensions (CSEs)**: Local Windows DLLs (located on client machines) that read GPO settings from `SYSVOL` and apply them to the local system (e.g., Folder Redirection CSE, registry CSE).

### 2. GPO Processing Order: LSDOU
Windows evaluates GPOs in a specific sequence:
1. **L (Local)**: Local computer policy.
2. **S (Site)**: GPOs linked to the Active Directory Site.
3. **D (Domain)**: GPOs linked to the Domain.
4. **OU (Organizational Unit)**: GPOs linked to the OU containing the user or computer object.
- **Rule of Conflict**: GPOs applied last override policies applied earlier. Therefore, an OU GPO will override a Domain GPO if they modify the same setting.

### 3. GPO Filtering & Loopback Processing
- **Security Filtering**: Restricts the GPO to specific users or computers within the linked OU (requires **Read** and **Apply Group Policy** permissions).
- **WMI Filtering**: Filters GPOs using WMI queries (e.g., apply only if OS version is Windows 11: `select * from Win32_OperatingSystem where Version like "10.0.22%"`).
- **Loopback Processing**: By default, User configuration settings only apply to User objects, and Computer configurations only apply to Computer objects. Loopback processing forces user settings to apply based on the *computer's* location (e.g., in a kiosk or terminal server environment):
  - **Merge Mode**: Applies the user's GPOs first, followed by the computer's GPOs. If conflicts exist, the computer's GPOs win.
  - **Replace Mode**: Ignores the user's standard GPOs, applying only the user settings defined in the GPOs linked to the computer's OU.

---

## Commands & Syntax

### PowerShell
```powershell
# Query all Group Policy Objects in the active domain
Get-GPO -All | Select-Object DisplayName, Id, GpoStatus

# Backup all GPOs in the domain to a designated directory
Backup-GPO -All -Path "C:\Backups\GPOs"
```

### CMD / Run Box
```cmd
REM Force an immediate client refresh of all group policies (User & Computer)
gpupdate /force
REM Generate a detailed HTML report of applied policies on the client machine
gpresult /h C:\Reports\gpreport.html
```

### GUI Path
> Server Manager -> Tools -> **Group Policy Management** console (`gpmc.msc`)
> Or: Start -> Run -> type `rsop.msc` (Resultant Set of Policy on client)

### Important Registry Paths
GPOs write their configurations to dedicated policy keys:
```
HKLM\SOFTWARE\Policies\
HKCU\Software\Policies\
```

### Key Event IDs
GPO events are logged in the System log under source `Microsoft-Windows-GroupPolicy`.

| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 1500 | Group Policy processing started | System Log |
| 8003 | Group Policy processing completed successfully | System Log |
| 7017 | Group Policy processing failed (includes error code) | System Log |

---

## Real-World Scenarios

### Scenario 1: Security GPO Fails to Apply on Workstation (Security Filtering Error)
**User Complaint:** "My laptop was added to the Finance department, but I am still not receiving the drive mapping or desktop folder access that other members have."
**Your First 3 Checks:**
1. Check the computer's OU location in ADUC.
2. Run `gpresult /r` on the client to check the applied GPOs list.
3. Check the GPO's security filtering configuration in GPMC.
**Diagnosis Steps:**
1. Open command prompt on the client as Administrator.
2. Run the policy report: `gpresult /r`
3. Inspect the output under **Filtered GPOs (Not Applied)**:
   - GPO Name: `Finance Drive Map GPO` -> Reason: `Denied (Security Filtering)`
4. Open the GPMC console on the DC. Select the GPO -> **Delegation** tab.
   - The security group `Global-Finance` has **Read** and **Apply Group Policy** permissions.
   - However, the **Authenticated Users** group was removed from the delegation list.
   - *Since Windows 10 update MS16-072, computer accounts must be able to read the GPO template from SYSVOL. If the computer account lacks read permissions, the policy fails.*
**Root Cause:** The GPO security filtering blocked the computer account from reading the policy because the "Authenticated Users" group (or the computer object) was missing read permissions.
**Fix:** In the GPO delegation properties, add the **Domain Computers** group with **Read** permissions, or add **Authenticated Users** with **Read** permissions (unchecking Apply Group Policy). Rerun `gpupdate /force`.
**Prevention:** Always ensure "Authenticated Users" retains Read permission when applying custom security filtering.
**Ticket Close Note:** "Added Authenticated Users back to GPO delegation with Read-only permissions. Ran gpupdate /force on client. verified successful drive mapping. Closing ticket."

### Scenario 2: Slow Login Times During Startup (Group Policy Processing Delay)
**User Complaint:** "When I boot my laptop in the morning, it takes over 3 minutes on the 'Please wait for Group Policy Client' screen before displaying the login page."
**Your First 3 Checks:**
1. Check the Event Viewer System log for Group Policy processing times.
2. Check if the workstation has a wired or wireless network connection during boot.
3. Identify which specific GPO extension is causing the delay.
**Diagnosis Steps:**
1. Open Event Viewer on the client -> `Applications and Services Logs` -> `Microsoft` -> `Windows` -> `Group Policy` -> `Operational`.
2. Filter the log for Event ID `5016` (GPO extension processing times).
   - Locate the extension with the highest duration:
   - Message: `Completed Folder Redirection extension processing in 120 seconds.`
3. Check the Folder Redirection GPO settings. The policy is configured to redirect the Desktop and Documents folders to a local file server, but the user is working from home over a slow VPN connection.
**Root Cause:** The Folder Redirection CSE was attempting to sync large files over a slow home connection during boot, blocking the login page.
**Fix:** Configure the Folder Redirection GPO to disable sync over slow links, or configure the policy to run asynchronously.
**Prevention:** Avoid running heavy file synchronization tasks during system startup.
**Ticket Close Note:** "Configured slow link detection on Folder Redirection GPO settings. Startup logon times returned to under 15 seconds. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Edit the **Default Domain Policy** or **Default Domain Controllers Policy** to configure optional settings like drive mappings, desktop wallpapers, or software installations.
> - Corrupting these default policies can break core domain services and authentication.
> - Always create separate, dedicated GPOs for custom settings, keeping the default policies clean.

> [!warning] Common Trap
> - Enabling **Block Inheritance** on an OU and assuming that critical domain security policies (like password complexity and lockout rules) still apply.
> - Block Inheritance blocks *all* policies, including default domain policies.
> - Configure the Default Domain Policy as **Enforced** to force security settings to apply across all sub-OUs regardless of inheritance blocks.

> [!tip] Senior Engineer Tip
> - If you need to troubleshoot complex GPO interactions on a client machine, generate a detailed HTML report using:
>   `gpresult /h C:\gpreport.html`
>   - The HTML report highlights conflicting settings, showing exactly which GPO won the conflict and which GPOs were overridden.

> [!success] Verification Steps
> - Run: `gpresult /r` on the client.
> - Verify the target GPO name is listed under **Applied Group Policy Objects**.

> [!question] Interview Alert
> - "Explain what Loopback Processing is and when you would use it."
> - Answer: "Loopback Processing is a GPO setting used to apply User configuration settings based on the *computer's* OU location rather than the user's location. It is used in shared environment scenarios, like kiosk terminals, laboratories, or Citrix servers, where we want any user logging into that specific machine to receive a locked-down interface."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Linking GPOs to default containers | Containers do not support GPO links | Move computer and user objects out of default containers into OUs before linking GPOs. |
| Forgetting Read permissions | Security filtering blocks computer access | Always grant the Domain Computers or Authenticated Users group Read permissions on custom filtered GPOs. |
| Confusing User and Computer configs | Applying settings to the wrong objects | Apply user-specific GPOs to OUs containing user accounts, and computer GPOs to OUs containing computer accounts. |

---

## Lab Exercise

**Objective:** Create a GPO to deploy a registry setting, configure security filtering, link it to an OU, and verify application on a client machine.
**Time Required:** 30 minutes
**Environment Needed:** A Windows Server 2022 VM (DC01) and a Windows 11 VM.
**Pre-requisites:** Active Directory domain configured, client VM joined.

**Steps:**
1. Create GPO: On `DC01`, open the Group Policy Management Console (`gpmc.msc`).
2. Right-click **Group Policy Objects** -> Select **New** -> Name it `GPO-DisableUSB`.
3. Edit GPO: Right-click `GPO-DisableUSB` -> Select **Edit**.
4. Navigate to:
   `Computer Configuration\Policies\Administrative Templates\System\Removable Storage Access`
5. Enable **All Removable Storage classes: Deny all access** -> Click **OK** -> Close editor.
6. Link GPO: Right-click your `Workstations` OU -> Select **Link an Existing GPO** -> Select `GPO-DisableUSB` -> Click **OK**.
7. Log into your Windows 11 VM. Open a command prompt as Administrator and run:
   ```cmd
   gpupdate /force
   ```
8. Verification: Check the applied GPOs using `gpresult`:
   ```cmd
   gpresult /r
   ```
   - Expected output: `GPO-DisableUSB` is listed under **Applied Group Policy Objects**.

**Success Criteria:** The GPO successfully applies to the client VM, blocking USB port access, and is listed in the `gpresult` report.
**Common Failures:** Policy fails if the computer object is located in an OU where inheritance is blocked.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: How do you force a Group Policy update on a Windows client?**
A: I open a command prompt and run the command `gpupdate /force`. This tells the client to immediately query the Domain Controller for new user and computer policy changes.

**Q: What is the difference between gpupdate and gpresult?**
A: `gpupdate` is used to trigger a refresh and apply group policies on the client machine. `gpresult` is a diagnostic tool used to generate reports showing which group policies actually applied to the user and computer, and which were blocked or filtered.

### Intermediate (L2 Level)
**Q: Explain the difference between GPO Block Inheritance and Enforce options.**
A: **Block Inheritance** is configured on an OU to prevent higher-level GPOs (from parent OUs or the domain) from applying to objects within that OU. **Enforce** is configured on a GPO link to override Block Inheritance, forcing the GPO settings to apply to all child OUs regardless of any inheritance blocks.

### Advanced (L3/Senior Level)
**Q: A user on a multi-domain forest cannot map network drives. `gpresult` shows the drive mapping GPO is applied, but the drive letter is missing. Explain your diagnostic and resolution strategy.**
A:
- **Situation**: A drive mapping GPO was applied, but the drive letter was missing on the client.
- **Task**: I needed to identify why the policy failed to map the drive.
- **Action**: I ran `gpresult /h` to check the execution logs. The report showed the Drive Mapping CSE failed with error code `0x80070005` (Access Denied). I inspected the GPO configuration and found the drive map path pointed to `\\server\share`. I checked the NTFS permissions on the share and observed that the user's security group lacked Read access. I added the group to the share permissions.
- **Result**: The GPO successfully mapped the drive on the next login.

### HR / Behavioral
**Q: Tell me about a time you had to deal with a GPO change that caused issues. How did you identify and resolve the problem?**
A: A newly deployed security GPO blocked a legacy accounting application. I checked the applied GPOs, identified the setting causing the block, moved the affected users to a sub-OU, blocked inheritance of the GPO, and worked with security to configure an exception rule.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The Active Directory feature used to define and enforce security and system configurations on clients.
> **Why**: Critical for automating client management, security baselines, and drive mappings.
> **How**: Link GPOs to OUs, configure security and WMI filters, and troubleshoot using `gpresult`.
> **Command**: `gpresult /r`
> **Interview Answer Starter**: "In my experience, Group Policy troubleshooting requires generating a gpresult report to identify which policies are active and which are blocked..."

**Key Numbers to Remember:**
- Default policy refresh interval: 90 minutes (with a random 30-minute offset)
- Default replication path: `\\domain.local\SYSVOL\`
- Successful update code: Event ID 8003

**3 Things Interviewer Wants to Hear:**
1. LSDOU processing order (Local -> Site -> Domain -> OU)
2. Enforcing read permissions for "Authenticated Users" on filtered GPOs
3. Loopback processing modes (Merge vs. Replace) for shared terminals

---

## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/Users-and-Groups|Users and Groups]] — Target objects for GPO configurations.
- [[03-Identity-and-Core-Services/06-Active-Directory/Organizational-Units|Organizational Units]] — GPO link boundaries.
- [[02-Operating-Systems/03-Windows-OS/Registry|Registry]] — Details where GPO settings are written.

---

## Tags
#desktop-support #active-directory #group-policy #GPO #L2 #interview-topic #lab-complete #daily-use

