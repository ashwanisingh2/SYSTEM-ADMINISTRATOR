---
tags: [desktop-support, active-directory, identity, L2]
aliases: [ws-05-group-policy-complete-guide, ws-05]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# WS-05: Group Policy — Complete Guide

> [!abstract] Overview
> This note covers Group Policy Object (GPO) architecture, inheritance processing, security filtering, and client troubleshooting. It contains configuration paths for twenty key administrative policies and diagnostic command guides.

---

---
## Concept Overview
Think of Group Policy as the central remote control and law enforcement database of your organization's IT department. 

Instead of a sysadmin walking to 500 computers to block USB drives, configure security settings, and set the corporate desktop wallpaper, the admin writes a law (GPO) in the central database. 

When a computer boots up (Computer Configuration) or a user logs in (User Configuration), the system queries the Domain Controller: "What laws apply to me?" The DC hands over the linked policies. The operating system instantly enforces those settings, locking down registry paths, changing files, and configuring applications automatically.


---

---
## Technical Deep Dive
### 1. GPO Architecture: LSDOU Processing Order
GPOs are processed in a specific hierarchical order, commonly remembered by the acronym **LSDOU**:
1. **L**ocal Policies: Configured on the physical client machine (`gpedit.msc`). Processed first.
2. **S**ite Policies: Linked to the physical AD Site location. Processed second.
3. **D**omain Policies: Linked to the domain root (e.g., Default Domain Policy). Processed third.
4. **O**rganizational Unit (OU) Policies: Linked to the OU level. Processed last.
- **Conflict Rule:** **The last policy processed wins**. If a setting in a Domain GPO enables USB access, but an OU GPO denies it, the OU GPO wins because it is processed last.

### 2. GPO Inheritance Rules & Overrides
- **Block Inheritance:** Applied to an OU. Blocks all policies linked *higher* up the chain (Local, Site, Domain, parent OUs) from applying to objects inside that OU.
- **Enforced (formerly No Override):** Applied to a GPO link. Forces the GPO settings down the chain, overriding any downstream block inheritance settings.
- **Security Filtering:** Limits GPO application to specific security groups rather than the entire OU.
- **WMI Filtering:** Uses WMI queries to target specific hardware states (e.g., only apply if OS is Windows 11, or if laptop runs on battery).

### 3. User Configuration vs. Computer Configuration
- **Computer Configuration:** Applies when the machine boots up. Governs network, security, OS configurations, and startup scripts. Applies to the physical device regardless of who logs in.
- **User Configuration:** Applies when the user enters credentials. Governs desktop settings, mapped drives, printers, folder redirection, and application choices. Applies to the user profile on any machine.

---

## 20 Key Group Policies for Sysadmins

### 1. Map Network Drives
- **Path:** `User Configuration > Preferences > Windows Settings > Drive Maps`
- **Action:** Maps shared folders (e.g., `\\SVR-FS01\Sales`) to standard letters (e.g., `S:`).

### 2. Deploy Network Printers
- **Path:** `User Configuration > Preferences > Control Panel Settings > Printers`
- **Action:** Dynamically pushes shared office printers based on user locations.

### 3. Set Corporate Desktop Wallpaper
- **Path:** `User Configuration > Policies > Administrative Templates > Desktop > Desktop > Desktop Wallpaper`
- **Action:** Forces a specific local/UNC path JPG as background, locking out changes.

### 4. Disable USB Storage Drives
- **Path:** `Computer Configuration > Policies > Administrative Templates > System > Removable Storage Access > All Removable Storage classes: Deny all access`
- **Action:** Blocks USB mass storage devices from mounting to prevent data exfiltration.

### 5. Password Complexity Policy
- **Path:** `Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy`
- **Action:** Forces minimum length (e.g., 12 chars), complexity, and age limits. Must be configured in the Default Domain Policy unless using Fine-Grained Password Policies (FGPP).

### 6. Software Restriction Policies (Block executables)
- **Path:** `Computer Configuration > Policies > Windows Settings > Security Settings > Software Restriction Policies`
- **Action:** Prevents execution of files in user-writable directories (e.g., blocks `*.exe` running out of `%appdata%` to stop ransomware).

### 7. Redirect "My Documents" Folder
- **Path:** `User Configuration > Policies > Windows Settings > Folder Redirection > Documents`
- **Action:** Syncs local User Documents to a secure network home folder (e.g., `\\SVR-FS01\Home\%username%`).

### 8. Deploy Software (MSI Installer)
- **Path:** `Computer Configuration > Policies > Software Settings > Software installation`
- **Action:** Installs business applications (.msi files) silently on computer boot.

### 9. Configure Edge Home Page URL
- **Path:** `Computer Configuration > Policies > Administrative Templates > Microsoft Edge > Configure the home page URL`
- **Action:** Forces Microsoft Edge to open the corporate intranet portal by default.

### 10. Run Computer Startup/Shutdown Scripts
- **Path:** `Computer Configuration > Policies > Windows Settings > Scripts (Startup/Shutdown)`
- **Action:** Executes batch/PowerShell scripts using system privileges during boot/shutdown.

### 11. Account Lockout Policy
- **Path:** `Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout Policy`
- **Action:** Locks domain accounts for 30 minutes after 5 failed login attempts to prevent brute-forcing.

### 12. Disable Control Panel & Settings access
- **Path:** `User Configuration > Policies > Administrative Templates > Control Panel > Prohibit access to Control Panel and PC settings`
- **Action:** Blocks users from modifying local settings or launching administrative tools.

### 13. Enable Windows Defender Firewall
Advanced content only — basics in [[Windows Defender, BitLocker basics]]

- **Path:** `Computer Configuration > Policies > Windows Settings > Security Settings > Windows Defender Firewall with Advanced Security`
- **Action:** Configures firewall rules and enforces stateful packet filtering.

### 14. Configure Windows Update Client (WSUS target)
- **Path:** `Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Update > Manage updates offered from Windows Server Update Service`
- **Action:** Directs workstations to pull updates from local WSUS servers instead of Microsoft Update.

### 15. Turn Off Microsoft Store Application
- **Path:** `Computer Configuration > Policies > Administrative Templates > Windows Components > Store > Turn off the Store application`
- **Action:** Disables the consumer App Store, preventing unapproved downloads.

### 16. Restrict Local Administrators Group (Restricted Groups)
- **Path:** `Computer Configuration > Policies > Windows Settings > Security Settings > Restricted Groups`
- **Action:** Replaces local admin group membership with explicit global AD groups (e.g., `Domain Admins`), removing rogue users.

### 17. Rename Local Administrator Account
- **Path:** `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Accounts: Rename administrator account`
- **Action:** Changes the default "Administrator" login name to a custom name (e.g., `sysadmin_local`) to mitigate automated scans.

### 18. Enable Remote Desktop (RDP)
- **Path:** `Computer Configuration > Policies > Administrative Templates > Windows Components > Remote Desktop Services > Remote Desktop Session Host > Connections > Allow users to connect remotely by using Remote Desktop Services`
- **Action:** Enables RDP server services on workstations for administrative support access.

### 19. Configure Time Synchronization (NTP Client)
- **Path:** `Computer Configuration > Policies > Administrative Templates > System > Windows Time Service > Time Providers > Configure Windows NTP Client`
- **Action:** Directs all domain computers to synchronize time with the domain PDC emulator (critical for Kerberos authentication).

### 20. Prevent Access to Command Prompt / PowerShell
- **Path:** `User Configuration > Policies > Administrative Templates > System > Prevent access to the command prompt`
- **Action:** Blocks standard users from launching `cmd.exe` or executing scripts.

---

## Common Mistakes
> [!warning] Avoid These
> **Modifying the Default Domain Policy for general software deployment:** Adding dozens of custom drive maps, registry edits, and software deployments directly into the `Default Domain Policy` or `Default Domain Controllers Policy`. If a configuration breaks, recovering these core default policies is difficult and impacts the entire forest.
> **Correct approach:** Leave default policies at their default settings (except password limits). Always create dedicated, modular GPOs for custom configurations.

---

## Pro Tips
> [!tip] Field Experience
> When configuring GPO preferences (like drive maps or file copies), use **Item-Level Targeting** (located under the Common tab of the preference properties). This allows you to restrict the policy setting to apply only if the user matches highly specific criteria (e.g., only if they belong to a specific security group, have a specific IP address range, or use a laptop chassis).

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> A Domain Controller (`SVR-DC01`), a member workstation, and an OU named `Accounts_OU` containing a test user (`jdoe`).

### Step 1: Open Group Policy Management
1. On `SVR-DC01`, open Server Manager -> Tools -> **Group Policy Management** (`gpmc.msc`).
2. Expand your Forest -> Domains -> `company.local`.

### Step 2: Create and Link a GPO to Map a Network Drive
1. Right-click **Group Policy Objects** (the storage container) and select **New**.
2. Name: `Map_Drive_S_GPO`. Click OK.
3. Right-click the new GPO and select **Edit**. The Group Policy Management Editor opens.
4. Navigate to: `User Configuration` -> `Preferences` -> `Windows Settings` -> `Drive Maps`.
5. Right-click **Drive Maps** -> **New** -> **Mapped Drive**.
6. Configuration:
   - Action: **Create**
   - Location: `\\SVR-FS01\Accounts`
   - Label: `Accounts Shared`
   - Drive Letter: Use **S**
7. Click Apply, then click OK. Close the Editor.
8. Right-click your target OU **`Accounts_OU`** and select **Link an Existing GPO**.
9. Select `Map_Drive_S_GPO` and click OK.

### Step 3: Verify the GPO on Client Workstation
1. Log into a domain workstation using the test user `jdoe` account.
2. Open Command Prompt. Force policy evaluation:
   ```cmd
   gpupdate /force
   ```
3. Open File Explorer.
4. **Verify:** Check "This PC." Confirm that the network drive `S:` pointing to `\\SVR-FS01\Accounts` is visible and mapped.

---

---
## Cheat Sheet / Quick Reference
```cmd
:: Windows Command Prompt (Client Side)
:: Force immediate Group Policy update evaluation (runs in background)
gpupdate /force

:: Generate simple, text-based Group Policy execution report for the active session
gpresult /r

:: Generate a detailed HTML-based Group Policy report to analyze failures
gpresult /h C:\temp\gpreport.html && start C:\temp\gpreport.html
```

### Group Policy Management Console (GPMC) Diagnostic Tools
- **Group Policy Modeling:** Runs a wizard simulating the Resultant Set of Policy (RSoP) for a user/computer before deploying. Great for testing LSDOU inheritance and block rules.
- **Group Policy Results:** Queries a remote target online client machine and reads its active database to report which policies actually applied.

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | LSDOU | Local, Site, Domain, OU. The order of policy processing where downstream links override upstream. |
| 2 | Enforced | Link option that forces GPO settings down the OU tree, overriding block inheritance rules. |
| 3 | gpupdate /force | Command line utility that triggers an immediate refresh and evaluation of all active policies. |
| 4 | gpresult /h | Generates a detailed HTML report showing exactly which GPO settings applied or failed. |
| 5 | Item-Level Target | Preference filter targeting policies based on specific security groups or environment parameters. |

---

---
## Troubleshooting
**Scenario 1:**
- **Problem:** A security GPO to block USB access has been configured and linked to the `Finance_OU`, but the settings are not applying to client workstations. The client reports "GP successfully applied" but USB drives still open.
- **Root Cause:** Target mismatch. The USB deny policy is located under **Computer Configuration**, but the `Finance_OU` only contains **User Objects**. GPOs configured under Computer Configuration only apply if linked to OUs containing Computer Objects.
- **Fix:**
  1. Open `gpmc.msc`. Locate the GPO link.
  2. Move the target client computer objects in Active Directory into the `Finance_OU` (or a sub-OU named `Finance_Computers`).
  3. Link the USB block GPO to that computer OU.
  4. Run `gpupdate /force` on the client and reboot.

**Scenario 2:**
- **Problem:** A critical GPO linked to the Domain root is not applying to a specific OU. Running `gpresult /r` on the client lists the GPO under "Filtered Out (Reason: Blocked)".
- **Root Cause:** The target OU has **Block Inheritance** enabled.
- **Fix:**
  1. Open Group Policy Management.
  2. Locate the Domain Root GPO link (e.g., `Default Domain Policy`).
  3. Right-click the GPO link and check **Enforced**.
  4. This forces the policy settings down the tree, overriding the block inheritance configuration on the target OU.
  5. Verify application on the client using `gpresult`.

---

---
## Interview Questions
**Q1: Explain the difference between Group Policy Policies (Settings) and Group Policy Preferences.**
A: **Group Policy Policies** are restrictive settings enforced by the OS. They modify locked system registry keys (such as `HKEY_CURRENT_USER\Software\Policies`). The user cannot override these settings; they are greyed out in the GUI. If the GPO is unlinked, the setting is removed (no tattooing). **Group Policy Preferences** are configurations (like drive maps or registry edits) that configure initial settings but are not strictly enforced. The user can modify these settings later. If the GPO is unlinked, the setting remains (tattooing), unless configured to "Remove this item when it is no longer applied."

**Q2: A client workstation has conflicting wallpaper settings. Policy A (linked to Domain) sets Red wallpaper. Policy B (linked to local OU) sets Blue wallpaper. If Policy A is marked "Enforced" and Policy B has normal link settings, which wallpaper applies?**
A: 
- **Situation:** Conflicting GPO links are applied to a workstation with one link marked as Enforced.
- **Task:** Apply the GPO processing hierarchy rules to determine the winning configuration.
- **Action:** Under normal LSDOU processing, the OU-linked Policy B would win because it is processed last. However, because the upstream Policy A is marked **Enforced**, it overrides any conflicting downstream policies.
- **Result:** Policy A wins, and the workstation displays the **Red wallpaper**.

**Q3: Describe how Group Policy Loopback Processing works, and when you would use it.**
A: By default, User Configuration settings apply based on where the *user object* is located in AD. Loopback Processing is used when you want User Configuration settings to apply based on the location of the *computer object* instead. It is used on shared environments like terminal servers (Remote Desktop Session Hosts) or kiosk PCs. When enabled (in either Merge or Replace mode), it forces the GPO to read the user policies linked to the computer's OU and apply them to any user logging into that specific machine, preventing normal user GPOs (like drive maps) from running on the kiosk.

---

---
## Seedha Simple Mein
*Seedha simple mein: Group Policy ke zariye hum hazaron computers aur users par security settings, registry values aur application configurations ek hi click mein enforce karte hain. LSDOU rule iski processing order ko control karta hai.*

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — Active Directory containers and OUs.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-06 Active Directory — Users Groups OUs|WS-06 Active Directory — Users Groups OUs]] — Managing security groups for filtering.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-08 FSMO Roles|WS-08 FSMO Roles]] — Domain PDC emulator role running time synchronization.
