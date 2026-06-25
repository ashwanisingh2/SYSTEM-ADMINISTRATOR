---
tags: [desktop-support, interview-prep, career, star-method, L2, L3]
aliases: [star-scenarios, behavioral-scenarios]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

# Top 20 Scenario-Based Interview Questions (STAR Method)

---

## Concept Overview
- **What it is**: A repository of the top 20 scenario-based interview questions answered using the **STAR Method** (Situation, Task, Action, Result). Each scenario represents a real ticket an IT engineer receives, focusing on logical troubleshooting, risk management, and communication.
- **Why it matters for a support engineer**: Interviewers use scenario questions to test how you solve problems under pressure, handle difficult users, and implement security policies. The STAR method provides a structured way to highlight your experience.
- **Where you encounter this in real job**: Writing incident root-cause analysis (RCA) reports, post-mortem reviews, and presenting project achievements to management.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Studies these scenarios to learn logical troubleshooting patterns and prepare for promotion boards.
  - **L2**: Uses these STAR models to formulate clear answers during engineering interviews.
  - **L3**: Analyzes these workflows to design standard operating procedures (SOP) and run technical interviews.

---

## The STAR Method Framework
When answering scenario-based questions, structure your response as follows:
- **Situation (S)**: Set the context. Describe the specific event, the urgency, and the impact on the business.
- **Task (T)**: Identify the challenge. What was your role, and what needed to be resolved?
- **Action (A)**: Detail the steps you took. What commands did you run, who did you coordinate with, and why?
- **Result (R)**: Share the outcome. What was restored, how fast was it fixed, and what did you implement to prevent it from happening again?

---

## Technical Scenarios (STAR Answers)

### Scenario 1: The Broken Domain Secure Channel
**Question: "Tell me about a time a workstation lost connection to the domain. How did you resolve it?"**
- **Situation**: A sales manager returned from a 3-month leave, turned on their desktop, and was blocked at logon with: *"The trust relationship between this workstation and the primary domain failed."* They had client meetings scheduled that morning.
- **Task**: I needed to repair the computer's secure channel connection to the Domain Controller immediately without destroying the user's local profile or SID configurations.
- **Action**: Instead of leaving and rejoining the domain (which would recreate the local profile and require hours of migration), I logged into the workstation using a local administrator account. I verified network connectivity to the DC, opened PowerShell as Administrator, and executed the repair cmdlet:
  `Test-ComputerSecureChannel -Repair -Credential (Get-Credential)`
  I entered the domain admin credentials to reset the machine password on the DC and sync it with the local workstation.
- **Result**: The trust relationship was restored within 2 minutes. The user logged in successfully with their domain account, preserving all local settings, files, and shortcuts. The ticket was closed, and I documented the cmdlet in our helpdesk wiki.

### Scenario 2: Endpoint Ransomware Outage
**Question: "How would you handle a workstation flagged with an active ransomware payload?"**
- **Situation**: A security alert indicated that a laptop (`DESKTOP-FIN02`) in our finance department had triggered a warning for the `WannaCrypt` ransomware worm, with files actively being encrypted.
- **Task**: Contain the infection instantly to prevent lateral spreading across our corporate subnets and recover the user's data.
- **Action**: I accessed the Microsoft Defender for Endpoint portal, located the device, and initiated **Device Isolation**, cutting off all network traffic except for the secure tunnel to our management portal. I contacted the user to shut down the laptop, retrieved it, booted it into the Windows Recovery Environment, and triggered a pre-boot offline scan:
  `Start-MpWDOScan`
  After cleaning the registry and deleting the malicious binaries, I re-installed the OS and logged the user into OneDrive, where I ran the "Restore your OneDrive" feature to roll back all their files by 24 hours.
- **Result**: The worm was isolated within 5 minutes of the alert, preventing any spread to our local servers. The user's files were restored to their pre-encrypted state, resulting in zero data loss.

### Scenario 3: Lost Files Recovery After Disk Failure
**Question: "A user's hard drive crashed and they lost local files. How did you recover their data?"**
- **Situation**: A designer's laptop suffered a physical SSD failure. They had not backed up files manually and feared losing a week's worth of client design drafts.
- **Task**: Recover the user's files and configure a new system to prevent future data loss.
- **Action**: I provisioned a replacement laptop. Since we had implemented **Known Folder Move (KFM)** via Microsoft Intune, the user's Desktop, Documents, and Pictures folders were automatically synced to OneDrive. I logged the user into their new laptop using their corporate M365 account. OneDrive began synchronizing, downloading all their design files back to their desktop.
- **Result**: The designer recovered all files within 15 minutes of logging in, resulting in zero data loss. The user praised the automated backup system.

### Scenario 4: VIP blocked by Geo-Blocking conditional access
**Question: "A VIP user is blocked from logging in during travel. How do you resolve this?"**
- **Situation**: Our VP of Sales traveled to Germany and was blocked at logon with error `53003` (Access Denied by Conditional Access). They needed access to present a proposal to a client.
- **Task**: Temporarily bypass the geo-blocking policy for the VP's account without exposing the tenant to attacks.
- **Action**: I audited the Entra ID Sign-in logs, locating the failed attempt. I verified that the block was caused by our `Block-Foreign-Logins` policy. After calling the VP on their registered phone to verify identity, I edited the policy properties in the Entra portal, added the VP's account to the **Exclusions** list, and set a calendar reminder to remove the exclusion in 7 days when they returned.
- **Result**: The VP logged in successfully and completed the client presentation. The exclusion was automatically audited and removed upon their return, maintaining security.

### Scenario 5: IP Address Conflict in Subnet
**Question: "Describe a time you resolved an IP address conflict on a local network."**
- **Situation**: Users in our warehouse reported that their terminals randomly lost connection to the inventory server, and Windows threw warnings about duplicate IP addresses.
- **Task**: Locate the device causing the IP address conflict and restore subnet stability.
- **Action**: I logged into the core switch and queried the ARP table to find the MAC address bound to the conflicting IP (`10.0.1.50`). I matched the MAC address to a physical network port on the switch. I found a user had brought in a personal router and plugged it into a wall jack, causing its DHCP server to distribute overlapping IP addresses. I disabled the network port on the switch.
- **Result**: The conflict was resolved, and the warehouse terminals resumed normal operations. I worked with the team to enable **DHCP Snooping** on the switches to block unauthorized rogue DHCP servers permanently.

### Scenario 6: Corrupted Profile for an Executive
**Question: "An executive's Windows profile is corrupted. How do you fix it under time pressure?"**
- **Situation**: The CFO's Windows profile corrupted, showing a temporary profile desktop, right before an annual budget review meeting. They could not access their local folders.
- **Task**: Rebuild the profile and restore access to their files within 30 minutes.
- **Action**: I logged in as local administrator. I backed up the CFO's folder from `C:\Users\CFO`. I opened Registry Editor and navigated to the ProfileList key:
  `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`
  I located the SID matching the CFO's account, deleted the key, and renamed the old user folder to `.old`. I had the CFO log in, which forced Windows to build a fresh, clean profile. I then copied their Desktop and Document folders back to the new profile directory.
- **Result**: The profile was rebuilt and files restored in 15 minutes, allowing the CFO to present their budget reports on schedule.

### Scenario 7: Outgoing Mail Flow Blocked (SPF Failure)
**Question: "What did you do when external clients complained they weren't receiving emails from your company?"**
- **Situation**: External partners reported that our corporate emails were going straight to their spam folders or being blocked entirely with NDRs.
- **Task**: Audit our domain DNS records to identify why outbound emails were failing verification.
- **Action**: I used `nslookup -type=TXT company.com` to analyze our public DNS records. I found that marketing had recently contracted a third-party CRM service to send emails on our behalf but had created a *second* SPF record in DNS. Because DNS specifications allow only one SPF record per domain, receiving servers failed the checks. I combined the two records into a single TXT entry:
  `v=spf1 include:spf.protection.outlook.com include:crmvendor.com -all`
- **Result**: The DNS change replicated, and outbound emails delivered normally within 2 hours. I set up a DNS auditing process to prevent duplicate record entries.

### Scenario 8: Teams Quality Outage in Branch Office
**Question: "How did you resolve poor call and video quality in a remote branch office?"**
- **Situation**: Employees at our new branch office complained that Teams video calls lagged, dropped constantly, and audio was choppy.
- **Task**: Identify the network bottleneck causing the poor call quality.
- **Action**: I reviewed the **Call Quality Dashboard (CQD)** in the Teams Admin Center. I found that calls in the branch had high packet loss (5%) and latency (250ms). I checked their network routing and found their corporate VPN client was forcing all internet traffic (including Teams media) to route to our headquarters firewall before exiting to the web. I configured **VPN Split Tunneling** to allow Teams UDP ports (3478-3811) to bypass the VPN and route directly to Microsoft's cloud.
- **Result**: Packet loss dropped below 1%, latency fell to 30ms, and call quality restored to excellent.

### Scenario 9: Recovering a Deleted VM from Azure Backup
**Question: "A critical virtual machine was accidentally deleted in Azure. How did you restore it?"**
- **Situation**: A junior admin deleted a production application VM (`VM-App-01`) from a resource group, causing a service outage for our logistics team.
- **Task**: Rebuild the VM from our latest recovery point with minimal downtime.
- **Action**: I went to the Recovery Services Vault, located the backup items, and selected the deleted VM's history. Thanks to **Soft Delete**, the backup data was preserved. I selected the latest recovery point (taken 6 hours prior), selected the option **Create Virtual Machine**, and mapped it to the original virtual network and resource group.
- **Result**: The VM was restored and booted within 25 minutes. I applied a `CanNotDelete` Resource Lock on all production resource groups to prevent future accidental deletions.

### Scenario 10: Stuck Windows Update Loop
**Question: "How did you fix a computer stuck at 'Working on updates' boot loop?"**
- **Situation**: A user's desktop was stuck on a boot loop for 3 hours, displaying: *"Working on updates. 100% complete. Don't turn off your computer."*
- **Task**: Force the update to abort or complete without reinstalling the OS.
- **Action**: I booted the computer into Safe Mode. I opened Command Prompt as Administrator, stopped the Windows Update service (`net stop wuauserv`), and renamed the cache folder to wipe the pending update data:
  `ren C:\Windows\SoftwareDistribution SoftwareDistribution.old`
  I restarted the service, rebooted the computer normally, and ran a clean manual update search.
- **Result**: The computer booted successfully to the logon screen, and the user returned to work.

### Scenario 11: Mailbox Offboarding Workflow
**Question: "How do you manage email offboarding for a terminated employee?"**
- **Situation**: HR submitted an offboarding ticket for a terminated sales director: *"Block login immediately, convert their mailbox to shared, and grant access to their manager."*
- **Task**: Secure the account, preserve the emails, and reclaim the license.
- **Action**: I reset the user's password in Active Directory and blocked their sign-in. I opened Exchange PowerShell and converted the mailbox:
  `Set-Mailbox -Identity "director@company.com" -Type Shared`
  I granted the manager Full Access and Send As permissions, waited for replication, and removed the M365 license in the admin portal to reclaim the cost.
- **Result**: The emails were preserved for the manager, access was secured, and the license was reclaimed, saving the company billing costs.

### Scenario 12: Duplicate UPN Sync Conflict
**Question: "Describe a time you resolved a synchronization error in Entra ID Connect."**
- **Situation**: A newly hired manager's account failed to sync to M365, throwing a `Duplicate UPN` error in the sync dashboard.
- **Task**: Identify the conflicting attribute and complete the sync.
- **Action**: I ran the **IDFix** tool on our local Domain Controller. It identified that a retired contractor account had the exact same user principal name (UPN) prefix and SMTP address cached in the directory. I modified the retired account's UPN suffix to `@company.local` and removed the SMTP alias from its proxyAddresses. I then ran a manual delta sync:
  `Start-ADSyncSyncCycle -PolicyType Delta`
- **Result**: The new manager's account synchronized successfully, and their mailbox was provisioned within 10 minutes.

### Scenario 13: Recovering a Seized FSMO Role
**Question: "What do you do if your primary Domain Controller suffers a catastrophic hardware failure?"**
- **Situation**: Our primary Domain Controller (holding the PDC Emulator and RID Master roles) suffered a motherboard blowout and was permanently unrecoverable.
- **Task**: Force the transfer of the FSMO roles to the secondary Domain Controller to restore AD health.
- **Action**: I logged into the secondary DC (`DC-02`). Since the primary DC was dead, I had to **Seize** the roles. I opened PowerShell as Administrator and executed:
  `Move-ADDirectoryServerOperationMasterRole -Identity "DC-02" -OperationMasterRole PDCEmulator, RIDMaster -Force -Confirm:$false`
  *Crucial safety step*: I instructed the hardware team to format the old DC's drives to ensure it could never be powered back online, preventing metadata corruption.
- **Result**: The roles were seized, time synchronization stabilized, and the helpdesk could resume creating new user accounts.

### Scenario 14: Restoring Replication After Tombstone Expiry
**Question: "A branch Domain Controller was offline for 7 months. How did you bring it back?"**
- **Situation**: A branch DC was powered on after being offline for 7 months. Replication failed with Event ID `2042` (Tombstone Lifetime Exceeded).
- **Task**: Safely recover the domain replication without risking lingering objects.
- **Action**: Since the DC had been offline longer than the 180-day tombstone limit, AD had blocked replication to prevent deleted items from reappearing. Instead of attempting to force replication, I ran a forced demotion on the branch DC, cleaned up its metadata on our primary DC using `ntdsutil`, re-imaged the branch server, renamed it, and promoted it as a clean new DC.
- **Result**: Replication was restored, and security integrity was maintained across all sites.

### Scenario 15: Resolving Local Group Policy Drift
**Question: "How do you handle a computer that has drifted from our corporate security policy?"**
- **Situation**: A compliance audit revealed that a remote kiosk computer was missing critical security settings (firewall disabled, guest account enabled).
- **Task**: Audit and re-apply our secure baseline to the device.
- **Action**: I logged into the kiosk PC as administrator. I used the `secedit` tool to export the active settings:
  `secedit /export /cfg C:\Temp\kiosk_policy.inf`
  Comparing the file to our corporate baseline template `sec_baseline.inf` revealed manual registry overrides. I ran the configure command to force-apply the baseline:
  `secedit /configure /db C:\Windows\security\database\secedit.sdb /cfg C:\Temp\sec_baseline.inf /log C:\Temp\sec_log.txt`
- **Result**: The security baseline was re-applied, the guest account was disabled, the firewall was turned on, and the device passed the compliance audit.

### Scenario 16: Handling an Executive Policy Bypass Request
**Question: "How do you handle an executive who demands you bypass a security policy?"**
- **Situation**: A director requested I whitelist their personal tablet so they could sync corporate OneDrive files, bypassing our Conditional Access policy.
- **Task**: Deny the insecure request while maintaining a professional, helpful relationship.
- **Action**: I explained the security risk: whitelisting an unmanaged device allows data leaks if the tablet is lost or compromised by malware. I offered the approved alternative: "I cannot bypass the policy, but I can configure **App Protection Policies (MAM)** on your tablet. This allows you to view files in a secure sandboxed browser session, protecting corporate data while letting you work."
- **Result**: The director accepted the alternative, keeping our data secure while enabling their remote productivity.

### Scenario 17: RDP Connection Timeout to Azure VM
**Question: "An Azure VM is running, but you cannot connect via RDP. How do you troubleshoot?"**
- **Situation**: A developer could not connect to `VM-Prod-01` via RDP, even though the portal status showed "Running".
- **Task**: Identify the cause of the connection timeout and restore access.
- **Action**: I checked the **Boot Diagnostics** blade in the Azure Portal. The console screenshot showed a Blue Screen of Death (BSOD) with error `Inaccessible Boot Device`. I navigated to the **Redeploy + reapply** blade and clicked **Redeploy**. This moved the VM to a healthy physical host inside the Azure data center.
- **Result**: The VM booted successfully on the new host, and the developer restored their RDP connection.

### Scenario 18: Printer Spooler Crash Loop
**Question: "A print server's spooler service crashes continuously. How do you resolve it?"**
- **Situation**: The Print Spooler service on our main print server crashed every time a user attempted to print, halting all office print jobs.
- **Task**: Locate the corrupted print job or driver and restore the spooler.
- **Action**: I stopped the Spooler service. I navigated to the spool directory:
  `C:\Windows\System32\spool\PRINTERS`
  I deleted all cached `.shd` and `.spl` files (corrupted print jobs in queue). I restarted the service. When it crashed again, I inspected the event logs, identified a corrupted legacy printer driver, removed the driver using `Print Management`, and replaced it with a modern v4 driver.
- **Result**: The spooler stabilized, and all users resumed printing.

### Scenario 19: Auditing Unlicensed M365 Users
**Question: "How did you help the company reduce Microsoft 365 licensing costs?"**
- **Situation**: The company was paying for 100 excess M365 Enterprise E5 licenses, wasting IT budget.
- **Task**: Identify and remove licenses from inactive or terminated accounts.
- **Action**: I wrote a PowerShell script using the Microsoft Graph SDK that queried all users. The script filtered for accounts that had been disabled for over 30 days but still had active E5 licenses assigned.
- **Result**: The script identified 42 stale accounts. I removed the licenses, saving the company over $1,500 per month. I automated this cleanup to run monthly.

### Scenario 20: DNS Lookup Failures on Client Laptops
**Question: "Users on a subnet cannot access the internet, but they can ping the gateway. How do you troubleshoot?"**
- **Situation**: 20 users on a local office subnet reported they could not open any websites, although local network printers were reachable.
- **Task**: Identify the name resolution failure and restore connectivity.
- **Action**: I logged into a client PC and ran `ipconfig /all`. I noted that their primary DNS server was set to `192.168.1.5` (an old local DNS server that had been decommissioned). I checked the DHCP server scope options on our router and found the DNS server IP had not been updated. I changed the DHCP option 6 to point to our active Domain Controllers (`10.0.1.10` and `10.0.1.11`) and forced a IP release/renew on the clients.
- **Result**: DNS resolution was restored, and all users resumed internet access.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: Twenty advanced support scenarios structured in the STAR methodology (Situation, Task, Action, Result).
> **Why**: Critical preparation to showcase logical technical leadership and secure L2/L3 roles.
> **Key Cmdlets**: `Test-ComputerSecureChannel` / `Start-MpWDOScan` / `Set-Mailbox` / `Get-MgUser`
> **Interview Answer Starter**: "To address scenario questions, I structure my answers using the STAR method—focusing first on the business impact, then detailing the exact command line actions and security controls I implemented..."

**3 Things Interviewer Wants to Hear:**
1. A focus on minimizing business downtime (restoring service first).
2. Explicit CLI commands and registry/event diagnostic tools.
3. Proactive preventative actions taken after the incident is resolved to stop it from happening again.

---

## Related Notes
- [[06-Career-Growth/12-Interview-Prep/Top-30-L2-Questions|Top 30 L2 Questions]] — Hard technical question bank.
- [[05-Automation-and-Ticketing/11-ITSM-Ticketing/Incident-and-Problem-Management|Incident and Problem Management]] — Explains ITIL incident lifecycles.
- [[04-Cloud-and-Security/08-Azure/Azure-Backup|Azure Backup]] — Covers virtual machine recovery processes.

---

## Tags
#desktop-support #interview-prep #career #star-method #L2 #L3 #interview-topic #lab-complete #daily-use

