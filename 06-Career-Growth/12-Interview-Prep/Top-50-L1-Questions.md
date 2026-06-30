---
tags: [desktop-support, interview-prep, l1-helpdesk, career, L1]
aliases: [l1-interview-questions, helpdesk-qa]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #md-102
---

# Top 50 L1 Helpdesk Interview Questions and Answers

---

## Concept Overview
- **What it is**: A comprehensive, interview-focused repository of the top 50 L1 Helpdesk and IT Support Engineer questions with model answers. It covers hardware troubleshooting, basic networking, Windows OS utilities, Active Directory resets, and daily customer support interactions.
- **Why it matters for a support engineer**: Landing an L1 position requires demonstrating fundamental technical literacy, logical troubleshooting patterns, and strong customer-service skills. This note provides clean, professional answers to common questions.
- **Where you encounter this in real job**: Preparing for technical interviews, onboarding junior helpdesk technicians, and standardizing support scripts.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Studies this question bank to master fundamental concepts, troubleshoot basic client issues, and clear entry-level technical interviews.
  - **L2**: Evaluates candidates using these questions during team hiring assessments and verifies foundational competence.
  - **L3**: Maintains the technical hiring rubric and structures onboarding training files.

---

## Interview Questions & Answers

### Module 1: Hardware & Basic Troubleshooting

#### Q1: What is the POST process during computer startup?
**A**: POST stands for **Power-On Self-Test**. It is a diagnostic test run by the computer's BIOS/UEFI firmware immediately after powering on. It checks if essential hardware components—specifically the CPU, RAM, GPU, and storage controllers—are present and functioning. If POST fails, the system output beep codes or diagnostic LED lights.

#### Q2: What do you do if a computer turns on, fans spin, but the screen remains black?
**A**: First, I verify that the monitor is powered on, connected to the correct GPU port (not the motherboard onboard port if a discrete card is installed), and set to the correct input source. Second, I check for diagnostic beep codes or motherboard LED status lights. If no codes are present, I power down the PC, reseat the RAM sticks in their slots, and try booting with a single stick to isolate a memory failure.

#### Q3: What is the difference between RAM and ROM?
**A**: RAM (Random Access Memory) is volatile storage used by the CPU to hold active program files and data; it is wiped completely when the computer loses power. ROM (Read-Only Memory) is non-volatile storage containing the boot firmware (BIOS/UEFI); it retains its data permanently, even when the system is powered off.

#### Q4: How do you check the temperature of a CPU if a user complains of sudden shutdowns?
**A**: I reboot the system and enter the BIOS/UEFI console to check the hardware monitor page. Inside Windows, I run a third-party diagnostic utility like HWMonitor or Core Temp. If the CPU temperature exceeds 80-90°C under load, it indicates fan failure or dried thermal paste, causing the system to trigger thermal shutdowns.

#### Q5: What is a SSD and how does it compare to a traditional HDD?
**A**: An SSD (Solid-State Drive) stores data on flash memory chips and has no moving parts, offering high read/write speeds, shock resistance, and silent operation. A traditional HDD (Hard Disk Drive) uses spinning magnetic platters and read/write heads, which is slower, susceptible to physical drop damage, but cheaper for large storage capacities.

---

### Module 2: Basic Networking & Connectivity

#### Q6: What is an IP address and what is the difference between IPv4 and IPv6?
**A**: An IP (Internet Protocol) address is a unique numerical identifier assigned to each device on a network. IPv4 uses 32-bit addresses written in decimal format (e.g., `192.168.1.1`), providing about 4.3 billion unique combinations. IPv6 uses 128-bit addresses written in hexadecimal format (e.g., `2001:db8::1`), providing an almost infinite address space to support the growth of internet devices.

#### Q7: What does the command `ping` do?
**A**: `ping` sends ICMP (Internet Control Message Protocol) Echo Request packets to a target IP address or domain name and waits for an Echo Reply. It is used to test basic network connectivity and measure the round-trip latency in milliseconds.

#### Q8: What does it mean if a computer has an IP address of `169.254.X.X`?
**A**: It means the computer was unable to contact a DHCP server to obtain an IP address and automatically assigned itself an **APIPA (Automatic Private IP Addressing)** address. This indicates a network connection failure, a disconnected cable, or a stopped DHCP helper service.

#### Q9: What is DNS and what happens if it fails?
**A**: DNS (Domain Name System) translates human-readable domain names (like `google.com`) into computer-readable IP addresses (like `142.250.190.46`). If DNS fails, users cannot access websites using their names, though they can still connect if they type the target IP address directly.

#### Q10: How do you flush the DNS cache on a Windows client?
**A**: I open Command Prompt or PowerShell and run the command:
`ipconfig /flushdns`
This clears locally cached DNS records, forcing the OS to query DNS servers for updated IP mappings.

#### Q11: What is the difference between a Hub, a Switch, and a Router?
**A**: A Hub is a basic physical layer device that broadcasts incoming packets to all ports. A Switch operates at the Data Link layer (Layer 2) and forwards traffic to specific ports using MAC addresses. A Router operates at the Network layer (Layer 3) and routes traffic between different networks using IP addresses.

#### Q12: What is the default port used for HTTP, HTTPS, and RDP?
**A**: HTTP uses port **80**, HTTPS uses secure port **443**, and RDP (Remote Desktop Protocol) uses port **3389**.

#### Q13: What does the `tracert` command do?
**A**: `tracert` (traceroute) displays the hop-by-hop path packets take to reach a destination IP, listing the response times of each router along the way. It is used to identify where network traffic is lagging or being blocked.

#### Q14: What is a MAC address?
**A**: A MAC (Media Access Control) address is a physical 48-bit hardware identifier burned into a network card during manufacturing. It is globally unique and operates at Layer 2 (Data Link) of the OSI model.

#### Q15: What is the difference between a Public IP and a Private IP?
**A**: Private IPs (ranges like `192.168.X.X` or `10.X.X.X`) are used inside a local home or office network and are not routable on the public internet. Public IPs are assigned by ISPs, are globally unique, and route traffic across the internet.

---

### Module 3: Windows OS & Profile Management

#### Q16: How do you fix a corrupted user profile in Windows 10/11?
**A**: I log in as local administrator, backup the user's data from `C:\Users\Username`, rename the corrupted profile folder, and delete the corresponding SID registry key under `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`. When the user logs in again, Windows builds a fresh, clean profile, and I copy their files back.

#### Q17: What are the SFC and DISM commands used for?
**A**: `sfc /scannow` scans and repairs corrupted Windows system files using local backup cache repositories. If SFC fails to fix files, I run `dism /online /cleanup-image /restorehealth` to download and replace corrupted components directly from Microsoft Update servers.

#### Q18: What is the "Blue Screen of Death" (BSOD) and how do you troubleshoot it?
**A**: A BSOD is a critical system stop error that occurs when the Windows kernel encounters a fatal error it cannot recover from. I check the error code displayed on screen, boot into Safe Mode, and read the crash dump file using **BlueScreenView** to find which driver file (e.g., graphics or network card driver) caused the crash.

#### Q19: How do you open the local Services console in Windows?
**A**: I open the Run dialog (`Win + R`), type `services.msc`, and press Enter.

#### Q20: What is the difference between a Workgroup and a Domain?
**A**: A Workgroup is a peer-to-peer network where each computer manages its own local user database and security settings. A Domain is a centralized network managed by an Active Directory Domain Controller, allowing unified user logons, GPOs, and permissions across all computers.

#### Q21: What does `gpupdate /force` do?
**A**: It forces an immediate background pull and refresh of all User and Computer Group Policy Objects (GPOs) from the Domain Controller, overriding the standard 90-minute delay refresh cycle.

#### Q22: What is the Registry in Windows and why is it dangerous to edit?
**A**: The Registry is a database that stores low-level configuration settings for the Windows OS, hardware drivers, and installed software. Editing it is dangerous because a typo or deleting a critical registry key can corrupt the OS boot files, forcing a re-install.

#### Q23: How do you start a Windows computer in Safe Mode?
**A**: I hold down the Shift key while clicking Restart in the Windows start menu, navigate to Troubleshoot -> Advanced Options -> Startup Settings -> click Restart, and press the number key corresponding to Safe Mode (usually F4 or F5).

#### Q24: What is the purpose of Task Manager?
**A**: Task Manager displays active processes, CPU/RAM/Disk utilization metrics, startup programs, and running services. It is used to terminate hung applications (`End Task`) and identify performance bottlenecks.

#### Q25: How do you check a laptop's battery health report in Windows?
**A**: I open Command Prompt as Administrator and run the command:
`powercfg /batteryreport`
This generates an HTML report showing battery design capacity versus actual charge capacity.

---

### Module 4: Active Directory & Account Administration

#### Q26: A user is locked out of their AD account. How do you unlock it?
**A**: I open Active Directory Users and Computers (ADUC) or log into the AD Administrative Center, search for the user, open their properties, select the Account tab, check the box "Unlock account", and click Apply.

#### Q27: What is the difference between a Security Group and a Distribution Group?
**A**: A Security Group is used to assign access permissions to network shares, printers, and AD folders. A Distribution Group is used exclusively as an email list to send messages to multiple users simultaneously and cannot be assigned folder permissions.

#### Q28: How do you check what Group Policies (GPOs) are currently applied to a user?
**A**: I open Command Prompt on the user's PC and run:
`gpresult /r`
This outputs a report listing applied computer and user policies, security group memberships, and the authenticating Domain Controller.

#### Q29: What is an OU (Organizational Unit) in Active Directory?
**A**: An OU is a logical container folder in Active Directory used to group users, computers, and security groups. It is primarily used to organize the directory structure, delegate admin rights, and link Group Policy Objects (GPOs).

#### Q30: How do you verify if a user's account password has expired in Active Directory?
**A**: In ADUC, I check the user's Account tab properties. Alternatively, I run the PowerShell command:
`Get-ADUser -Identity "username" -Properties PasswordLastSet, PasswordExpired`
This returns the exact date the password was set and its current expiration state.

---

### Module 5: Office 365, Outlook, and Email

#### Q31: What is the first thing you do if Outlook crashes on startup?
**A**: I try to open Outlook in Safe Mode by holding down the Ctrl key while launching the application. If it opens successfully in Safe Mode, it indicates that a third-party add-in is corrupt or conflicting, and I disable all active add-ins.

#### Q32: What is the difference between an OST and a PST file in Outlook?
**A**: An `.ost` file is an offline cached replica of the user's Exchange or M365 mailbox stored locally. A `.pst` file is a personal archive file used to store local copies of emails, calendars, and contacts offline to free up mailbox space.

#### Q33: How do you rebuild a user's Outlook profile if mail syncing is broken?
**A**: I open Control Panel, search for **Mail**, click **Show Profiles**, click **Add** to create a new profile name, enter the user's credentials to sync their mailbox, and set the new profile as the default launch option.

#### Q34: What is Autodiscover in Exchange / Microsoft 365?
**A**: Autodiscover is a service that automatically configures email client profiles using a user's email address and password. It queries DNS SRV or CNAME records to locate the correct Exchange XML configurations, saving users from inputting manual server URLs.

#### Q35: How do you grant a user "Full Access" to a colleague's mailbox?
**A**: In the Exchange Admin Center (EAC) or Exchange PowerShell, I locate the mailbox, click Delegation, add the delegate user under **Read and Manage (Full Access)** permissions, and click Save.

#### Q36: What is a Shared Mailbox and does it require an active license?
**A**: A Shared Mailbox is a common mailbox accessed by multiple users (e.g., `sales@company.com`). It does not require a license if it is under 50GB and does not have an active user logging in directly; access is granted via delegation.

#### Q37: What is the quarantine folder in Microsoft 365 security?
**A**: The Quarantine portal holds incoming emails flagged as spam, phishing, or malware by Exchange Online Protection (EOP). Admins inspect and release false-positive messages from the quarantine dashboard.

#### Q38: A user gets an "Attachment size limit exceeded" error in Outlook. What is the default limit and how do you resolve it?**
**A**: The default attachment limit in Outlook is **20MB** for POP/IMAP accounts and **35-150MB** for Exchange/M365 accounts. I resolve this by teaching the user to upload large files to OneDrive and share a secure link instead of attaching the file directly.

#### Q39: What is Safe Mode in Office applications?
**A**: Safe Mode runs Office applications without loading any templates, custom toolbars, startup files, or third-party add-ins. It is used to bypass corruption and diagnose startup crashes.

#### Q40: What do you do if a user's Outlook folder shows "Disconnected" in the bottom status bar?**
**A**: First, I verify the computer has active internet access. Second, I check if the "Work Offline" button is toggled on in the Send/Receive tab. If it's connected to the web and offline is off, I close Outlook, end any residual `OUTLOOK.EXE` tasks in Task Manager, and restart the app.

---

### Module 6: Customer Service & Scenario Handling

#### Q41: How do you handle an angry user who calls screaming about a broken computer?
**A**: I remain calm, listen without interrupting, and validate their frustration by saying, "I understand this is blocking your work, and I am here to help you get this resolved." I focus on the technical issue, document the details, and guide them through step-by-step troubleshooting, keeping a professional tone.

#### Q42: What do you do if a user asks you to reset a password for another colleague who is out sick?
**A**: I politely decline, citing our security policy. I explain that we can only reset passwords for the account owner after verifying their identity. If their manager requires access to their files, they must submit a formal request ticket approved by HR or the Security team.

#### Q43: A user has a meeting in 5 minutes and their printer won't print. How do you triage this?**
**A**: Because time is critical, I offer a workaround first: I print the document from my helpdesk terminal, or guide them to send the file to a nearby colleague who can print it. Once their meeting starts, I log a ticket and troubleshoot the local driver or spooler issue on their workstation.

#### Q44: What is an SLA (Service Level Agreement) in a ticketing system?**
**A**: An SLA defines committed response and resolution time targets for support tickets based on priority (e.g., resetting a password within 1 hour). It ensures the helpdesk meets service expectations.

#### Q45: What do you do if you do not know the answer to a user's technical question?
**A**: I admit that I need to research the issue to ensure we apply the correct fix. I tell the user: "I want to verify the correct path for this configuration. Let me research this, and I will call you back in 15 minutes." I then check our internal wiki, search knowledge bases, or consult senior engineers.

#### Q46: How do you prioritize your ticket queue if you have 10 open tickets?
**A**: I prioritize tickets using the corporate Priority Matrix, which combines **Impact** (users affected) and **Urgency** (business criticality). A VIP lockout or server offline incident takes precedence over a single user requesting a software shortcut.

#### Q47: What is the purpose of documenting a ticket resolution?
**A**: Documenting the resolution creates a searchable history. If the issue occurs again, other technicians can find the fix instantly in our knowledge base, reducing resolution times (MTTR) and keeping the team aligned.

#### Q48: What is "Phishing" and what do you do if a user clicks a suspicious link?
**A**: Phishing is a social engineering attack where malicious actors send emails pretending to be trusted companies to steal passwords or download malware. If a user clicks a link, I immediately reset their password, revoke their active login sessions, scan their PC using Defender, and isolate the device if malware is detected.

#### Q49: What is dynamic IP addressing and how does it compare to static IP addressing?
**A**: Dynamic IP addresses are assigned automatically by a DHCP server for a set lease time and can change. Static IP addresses are manually configured on a device, remain constant, and are used for servers, printers, and network gateways.

#### Q50: Why are you interested in working as an IT Support Engineer at our company?
**A**: I enjoy solving technical puzzles and helping users overcome daily technical blocks. Your company is known for using modern cloud-based systems (like Intune and Azure), and I want to apply my troubleshooting skills here while growing from L1 to L2 and systems administration.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The core 50 entry-level questions covering hardware, basic networking, OS, active directory, and customer service.
> **Why**: Essential prep to verify technical literacy and secure support analyst roles.
> **Command**: `ipconfig /flushdns` / `gpupdate /force` / `sfc /scannow`
> **Interview Answer Starter**: "To resolve entry-level IT support issues, my approach is always to check the physical connection layer first, query diagnostic logs, and follow structured..."

**3 Things Interviewer Wants to Hear:**
1. A logical, step-by-step troubleshooting methodology (Hardware -> Network -> OS -> Software).
2. Verification of user identity before executing credential changes.
3. Calm, professional customer service during high-pressure user calls.

---

## Related Notes
- [[02-Operating-Systems/03-Windows-OS/WIN-07 User Profiles|WIN-07 User Profiles]] — Focuses on profile corruption fixes (Q16).
- [[02-Operating-Systems/03-Windows-OS/WIN-05 SFC and DISM|WIN-05 SFC and DISM]] — Details file repair utilities (Q17).
- [[01-Foundations/02-Networking/DNS|DNS]] — Explains name resolution flows (Q9, Q10).

---

## Tags
#desktop-support #interview-prep #l1-helpdesk #career #L1 #interview-topic #lab-complete #daily-use
