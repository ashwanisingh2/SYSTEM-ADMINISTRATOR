---
tags: [desktop-support, interview-prep, l2-support, career, L2]
aliases: [l2-interview-questions, sysadmin-qa]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Top 30 L2 Technical Interview Questions and Answers

---

## Concept Overview
- **What it is**: A repository of the top 30 L2 Desktop Support and Systems Administrator interview questions with deep technical model answers. It spans advanced Windows OS diagnostics, Group Policy loopback processing, hybrid Entra ID sync operations, and intermediate TCP/IP routing issues.
- **Why it matters for a support engineer**: Moving from L1 to L2 requires demonstrating command-line competence, understanding *why* systems fail, and managing identity/GPO architectures. This file provides comprehensive technical answers.
- **Where you encounter this in real job**: Preparing for promotion assessments, auditing technical knowledge, and drafting team hiring guides.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Reviews the questions to understand intermediate system flows and prepare for career progression.
  - **L2**: Uses these Q&As to study for technical interviews and validate daily operational troubleshooting steps.
  - **L3**: Conducts candidate interviews, using these questions to evaluate engineering problem-solving skills.

---

## Interview Questions & Answers

### Module 1: Advanced Windows & Driver Diagnostics

#### Q1: What causes a device to display "Code 43" in Device Manager, and how do you resolve it?
**A**: Device Manager displays **Error Code 43** when Windows stops a hardware device because the device driver reported a communication failure to the operating system. To resolve this:
1. I right-click the device, click Uninstall, and restart the PC to let Windows reinstall the driver.
2. If it fails, I download the latest manufacturer driver package, uninstall utility software, and run a clean install.
3. If it persists, I test the card in another slot or check for hardware thermal damage, as Code 43 often indicates physical GPU or USB controller failure.

#### Q2: Explain the difference between SFC and DISM, and why you run DISM first if SFC fails.
**A**: **SFC (System File Checker)** scans local Windows protected system files and replaces corrupted files using a cached backup folder located at `%WinDir%\System32\dllcache`. **DISM (Deployment Image Servicing and Management)** scans and repairs the actual Windows component store image. If the local dllcache cache itself is corrupted, SFC will fail because its source files are bad. I run `dism /online /cleanup-image /restorehealth` to download and rebuild the local cache from Microsoft Update servers, allowing SFC to run successfully.

#### Q3: What is Windows Credential Guard and how does it protect OS memory?
**A**: Credential Guard is a virtualization-based security feature that isolates secrets (like NTLM password hashes and Kerberos tickets) inside a virtual container running on the Hyper-V hypervisor. The standard Windows OS kernel cannot access this memory directly, preventing attackers who gain root access from executing memory dumping attacks (like Mimikatz) to steal credentials.

#### Q4: A user's computer is stuck in a boot loop installing Windows Updates. How do you recover it?
**A**: I boot the computer into the Windows Recovery Environment (WinRE).
1. Go to Troubleshoot -> Advanced Options -> Command Prompt.
2. I rename the Windows Update cache directory to clear corrupted updates:
   `ren C:\Windows\SoftwareDistribution SoftwareDistribution.old`
3. I use the DISM command-line utility to uninstall recently installed update packages offline:
   `dism /image:C:\ /get-packages`
   `dism /image:C:\ /remove-package /packagename:[PackageName]`
4. Exit, reboot the computer, and verify normal startup.

#### Q5: Where are Windows Update installation logs located, and how do you analyze them?
**A**: The active logs are recorded in the **Component Based Servicing (CBS) log** at:
`C:\Windows\Logs\CBS\CBS.log`
To extract update failures, I run a PowerShell command to filter for `[ERROR]` or `[FATAL]` tags:
`Select-String -Path C:\Windows\Logs\CBS\CBS.log -Pattern "Failed", "Error"`
Alternatively, I run `Get-WindowsUpdateLog` in PowerShell to compile the local ETW event logs into a readable desktop text file.

---

### Module 2: Active Directory & Group Policy

#### Q6: Explain what Group Policy Loopback Processing is and when you configure it.
**A**: Group Policy Loopback Processing is a GPO setting used when you need to apply User GPO settings to any user who logs into a specific computer (like a shared classroom computer or terminal server), bypassing their standard user OU GPOs:
- **Replace Mode**: Ignores the user's standard GPOs completely, applying only the GPOs linked to the computer's OU.
- **Merge Mode**: Combines the user's GPOs with the computer's GPOs. If there are conflicts, the computer's GPO settings win.

#### Q7: What is a WMI Filter in Group Policy and how is it used?
**A**: A WMI (Windows Management Instrumentation) Filter is a query linked to a GPO that evaluates system properties before applying the policy. If the query returns `$true`, the GPO applies.
- *Example*: `SELECT * FROM Win32_OperatingSystem WHERE Version LIKE "10.%"` ensures the GPO only applies to Windows 10/11 computers, skipping Windows Server systems.

#### Q8: What is the default order of GPO precedence if there are conflicting settings?
**A**: GPOs are processed in the **LSDOU** order:
1. **L**ocal Policies (processed first, lowest priority)
2. **S**ite GPOs
3. **D**omain GPOs
4. **O**rganizational **U**nit (OU) GPOs (processed last, highest priority)
- *Precedence*: The GPO closest to the object wins. If a setting is set to Enable at the Domain level but Disable at the OU level, the OU setting overrides and applies.

#### Q9: What happens to a domain user's login experience if the PDC Emulator FSMO role holder goes offline?
**A**: Users can still log in because standard Domain Controllers can authenticate password requests. However:
1. If a user recently reset their password, login might fail at remote branches because the PDC is responsible for immediate password sync.
2. Users who make multiple bad attempts may not trigger account lockouts correctly, as DCs replicate lockouts to the PDC emulator immediately.
3. System times will drift over time, eventually exceeding the 5-minute Kerberos skew limit, blocking all logons.

#### Q10: How do you identify which Domain Controller authenticated a client computer during login?
**A**: I open Command Prompt on the client machine and run:
`echo %LOGONSERVER%`
Alternatively, I run `gpresult /r` and check the "Group Policy was applied from" field to identify the authenticating DC.

---

### Module 3: Intermediate Enterprise Networking

#### Q11: Explain the difference between VPN Split Tunneling and Force Tunneling.
**A**:
- **Split Tunneling**: Only traffic destined for corporate servers (e.g., `10.0.0.0/8`) is routed through the encrypted VPN tunnel. Standard internet traffic (like streaming video) routes directly through the user's home ISP. This saves corporate gateway WAN bandwidth.
- **Force Tunneling**: All network traffic (corporate and internet) is forced through the VPN tunnel. Internet traffic egresses from the corporate firewall, allowing security teams to inspect and filter all user web traffic, but consumes high bandwidth.

#### Q12: What is a DHCP Relay Agent and why is it used?
**A**: A DHCP Relay Agent (Helper Address) is a routing configuration that forwards DHCP broadcast packets (DORA) from a local client subnet across a router to a centralized DHCP server on a different subnet. Since DHCP broadcasts cannot cross standard routers, the relay agent converts the broadcast to a unicast packet, allowing one DHCP server to service multiple subnets.

#### Q13: Explain how DNS SRV records locate Domain Controllers.
**A**: When a computer joins a domain, it queries its primary DNS server for **SRV (Service) records** under the `_msdcs` zone (e.g., `_ldap._tcp.dc._msdcs.domain.com`). The DNS server returns a list of active Domain Controllers, their IP addresses, and port numbers (port 389), allowing the client to contact the DC.

#### Q14: How does NAT (Network Address Translation) differ from PAT (Port Address Translation)?
**A**:
- **NAT (Static/Dynamic)**: Maps one private IP address to one public IP address. It requires a pool of public IPs.
- **PAT (NAT Overload)**: Maps multiple private IP addresses to a single public IP address by assigning unique port numbers (e.g. `192.168.1.10:1050` and `192.168.1.11:1051` share public IP `203.0.113.1`). Standard for home and office internet access.

#### Q15: How do you isolate network packet loss between a client laptop and a remote server?
**A**: I use the `pathping` command or run a trace route using `tracert`. `pathping` runs a ping test to every router hop along the path over 5 minutes, mapping the exact percentage of packet loss at each router interface, helping isolate if the latency is on the user's home network, the ISP, or the corporate gateway.

---

### Module 4: Cloud & Hybrid Administration

#### Q16: How does Password Hash Synchronization (PHS) differ from Pass-Through Authentication (PTA)?
**A**:
- **PHS**: A hash of the user's password hash is synchronized from local Active Directory to Microsoft Entra ID. Authentication occurs entirely in Microsoft's cloud, offering high availability.
- **PTA**: Microsoft Entra ID does not hold the password hash. When a user logs in, Entra passes the credentials to an on-premises agent, which validates them against the local Domain Controller, enforcing on-premises security controls.

#### Q17: A user is synchronized from on-premises AD, but they do not appear in the Entra ID Portal. How do you troubleshoot?**
**A**:
1. Check the on-premises Entra Connect server's Synchronization Service Manager (`miisclient.exe`) for sync run status and check for errors like `attribute-value-must-be-unique` or duplicate UPNs.
2. Verify the user is located in an OU configured to sync in the Entra Connect wizard.
3. Check the user's UPN suffix in ADUC to verify it uses a routable public domain name (e.g., `@company.com`), not a local non-routable suffix (e.g., `@company.local`).

#### Q18: What is the Microsoft Graph PowerShell SDK, and what scopes are required to reset a user's password?
**A**: The Microsoft Graph PowerShell SDK is the modern module used to manage Microsoft Entra ID resources via Microsoft Graph REST APIs. To reset a user's password, I run `Connect-MgGraph` specifying the scope:
`-Scopes "User.ReadWrite.All"` or `Directory.AccessAsUser.All`.

#### Q19: Explain the difference between Azure AD Joined and Hybrid Azure AD Joined devices.**
**A**:
- **Azure AD (Entra) Joined**: The device is registered directly in Entra ID and managed by Intune. It is cloud-native and does not communicate with local Domain Controllers.
- **Hybrid Azure AD Joined**: The device is joined to the on-premises Active Directory domain *and* registered in Entra ID, allowing access to legacy Kerberos fileshares and cloud-based SSO.

#### Q20: What is Shared Computer Activation (SCA) in Microsoft 365, and when must you enable it?
**A**: SCA is a licensing mode used when deploying Microsoft 365 Apps to shared computers, such as terminal servers (RDS) or shared clinic terminals. It allows multiple users to log into Office on the same machine without subtracting from their standard 5-device personal install limit.

---

### Module 5: Security, Encryption, and Access Management

#### Q21: A BitLocker-encrypted laptop repeatedly prompts for the recovery key on every single boot. How do you resolve this?
**A**: This occurs because the TPM PCR registers have lost their secure boot baseline measurement. To fix this:
1. Enter the recovery key to boot into Windows.
2. Open Command Prompt as Administrator and suspend BitLocker:
   `manage-bde -protectors -disable C: -rebootcount 1`
3. Reboot the computer. The TPM chip automatically reads the new system configurations and registers them as the new secure baseline.
4. Upon the second reboot, the prompt stops appearing.

#### Q22: What is the purpose of User Account Control (UAC) and how does it protect against privilege escalation?
**A**: UAC runs all applications (including administrative accounts) in standard user privilege tokens by default. When an application attempts a system-level change (like editing registry files or installing software), UAC halts execution, prompting the user for approval or admin credentials to elevate the token, blocking malware from executing silent installations.

#### Q23: Explain the difference between asymmetric and symmetric encryption.
**A**:
- **Symmetric Encryption**: Uses a single shared key to encrypt and decrypt data (e.g. AES). It is fast and efficient for encrypting large volumes of data (like BitLocker hard drives).
- **Asymmetric Encryption**: Uses a mathematically linked key pair: a Public Key to encrypt data, and a private Key to decrypt data (e.g. RSA, HTTPS certificates). It is slower but secures key exchanges over untrusted networks.

#### Q24: What is Microsoft Sentinel?
**A**: Microsoft Sentinel is a cloud-native **SIEM (Security Information and Event Management)** and **SOAR (Security Orchestration, Automated Response)** platform. It aggregates log data from across the enterprise (servers, firewalls, Azure, M365) and uses AI to detect and alert on active security threats.

#### Q25: What is the difference between RBAC (Role-Based Access Control) and ABAC (Attribute-Based Access Control)?
**A**:
- **RBAC**: Grants access permissions based on a user's assigned role (e.g., John is in the "Finance Contributor" group, so John has access to the billing server).
- **ABAC**: Grants access based on attributes of the user, the resource, and the environment (e.g., John has access to the billing server *only if* he is connecting from a corporate device *and* the current time is between 9:00 AM and 5:00 PM).

---

### Module 6: ITSM, Scripting, and Operations

#### Q26: What is a "Known Error" in ITIL, and how does it differ from a Workaround?
**A**: A **Known Error** is a problem record that has a documented root cause and a workaround, but no permanent fix has been applied yet (often stored in the KEDB). A **Workaround** is the actual step-by-step procedure used to restore service to users, bypassing the bug temporarily.

#### Q27: How do you handle a non-terminating error in a PowerShell script to ensure it triggers a Try-Catch block?
**A**: By default, non-terminating errors do not trigger the `Catch` block. I must append the parameter **`-ErrorAction Stop`** to the cmdlet inside the `Try` block. This forces the cmdlet to treat the failure as a terminating error, redirecting the execution flow to the `Catch` block immediately.

#### Q28: What is a Configuration Item (CI) and how does it differ from an Asset?
**A**:
- **Configuration Item (CI)**: Any component managed to deliver an IT service, tracked inside the CMDB (e.g., a virtual network, software license, or database). Focuses on **relationships**.
- **Asset**: A component with financial value to the company, tracked for depreciation and auditing (e.g., a physical server chassis, a monitor). Focuses on **financial cost**.

#### Q29: What is the purpose of the Change Advisory Board (CAB) in Change Control?
**A**: The CAB is a group of stakeholders that meets to review, evaluate, and schedule high-risk Normal Changes. They assess the business risk, verify that testing and rollback plans are complete, and coordinate the change calendar to prevent scheduling conflicts.

#### Q30: How do you use the command `secedit` to analyze policy drift on a Windows client?
**A**: I open Command Prompt as Administrator and run:
`secedit /export /cfg C:\Temp\policy.txt`
This exports all active local security database configurations (password rules, account lockouts, user rights) to a text file, which I compare against our corporate baseline template `corporate_security.inf` to identify unauthorized local modifications.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The top 30 intermediate technical questions covering driver issues, GPO loopbacks, advanced networking, hybrid Entra ID syncs, and security baselines.
> **Why**: Critical preparation to demonstrate L2 competence during technical interviews and promotion boards.
> **Command**: `dism /online /cleanup-image /restorehealth` / `secedit /export` / `manage-bde -status`
> **Interview Answer Starter**: "To resolve intermediate system and network issues, my approach is to verify the directory sync states, evaluate GPO precedence chains, and check Event IDs..."

**3 Things Interviewer Wants to Hear:**
1. Clear understanding of system dependencies (e.g., why GPO loopback replaces user settings).
2. Command-line first troubleshooting logic (PowerShell Graph/Az modules, manage-bde, secedit).
3. Distinguishing between quick workarounds (incidents) and permanent root causes (problems).

---

## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/Group-Policy|Group Policy]] — Focuses on GPO precedence and loopbacks (Q6, Q7, Q8).
- [[04-Cloud-and-Security/07-Microsoft-365/Microsoft-Entra-ID|Microsoft Entra ID]] — Explains hybrid sync operations (Q16, Q17, Q19).
- [[04-Cloud-and-Security/09-Security/BitLocker-Deep-Dive|BitLocker Deep Dive]] — Outlines boot screen recovery fixes (Q21).

---

## Tags
#desktop-support #interview-prep #l2-support #career #L2 #interview-topic #lab-complete #daily-use

