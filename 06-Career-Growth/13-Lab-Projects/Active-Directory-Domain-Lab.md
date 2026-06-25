---
tags: [desktop-support, lab-setup, active-directory, windows-server, dns, dhcp, L1, L2, L3]
aliases: [ad-lab, active-directory-setup]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Active Directory Domain Lab Setup Guide

---

## Concept Overview
- **What it is**: A hands-on deployment guide for building a functional Microsoft Active Directory Domain Services (AD DS) network in an isolated lab environment.
- **Why it matters**: Active Directory is the directory service core for 90% of enterprise environments. Mastering domain controllers, DNS zone registration, DHCP lease ranges, and group policies is vital for growing from an L1 technician to an L3 administrator.
- **Where you encounter this in real job**: Managing corporate logins, joining workstations to a domain, troubleshooting failed group policy applications, and configuring network DHCP scopes.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Joins client PCs to the domain, resets user passwords, creates security groups, and checks applied local group policies (`gpresult`).
  - **L2**: Manages DNS zone replication, configures DHCP scopes, creates Group Policy Objects (GPOs), and manages organizational unit (OU) structures.
  - **L3**: Architectures multi-site replication topology, manages FSMO role placements, recovers deleted objects via AD Recycle Bin, and deploys automation scripts.

---

## Technical Deep Dive

### Active Directory Architecture Blueprint
To complete this lab, you will deploy a two-VM network configuration on an isolated host-only network switch:
```
[Isolated Host-Only Virtual Switch]
       |
       +--- VM 1: SRV-DC-01 (Windows Server 2022)
       |          - Role: Domain Controller (AD DS, DNS, DHCP)
       |          - Static IP: 192.168.10.10 / Subnet: 255.255.255.0
       |          - Domain: corp.contoso.com
       |
       +--- VM 2: CLI-WIN-10 (Windows 10/11 Client)
                  - Role: Client Workstation
                  - IP: Dynamic (leased from SRV-DC-01)
                  - DNS: Set to 192.168.10.10
```

### DNS and AD DS Integration
Active Directory cannot function without DNS.
- **SRV Records**: Domain Controllers register Service Location (SRV) records in DNS under the `_msdcs` zone. These records allow client machines to locate the Domain Controllers (DCs) for LDAP, Kerberos authentication, and directory queries.
- **Active Directory-Integrated DNS**: Zones are stored directly inside the Active Directory database (`NTDS.dit`), enabling automatic multi-master replication and secure dynamic updates.

---

## Commands & Syntax

### PowerShell: Installing AD DS and Domain Promotion
Run these commands on `SRV-DC-01` to install AD DS and promote the server to a new domain controller in a new forest:
```powershell
# 1. Install the Active Directory Domain Services Windows Feature
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# 2. Promote the server to a new Forest Controller
Import-Module ADDSDeployment
Install-ADDSForest `
    -CreateDnsDelegation:$false `
    -DatabasePath "C:\Windows\NTDS" `
    -DomainMode "WinThreshold" `
    -DomainName "corp.contoso.com" `
    -DomainNetbiosName "CORP" `
    -ForestMode "WinThreshold" `
    -InstallDns:$true `
    -LogPath "C:\Windows\NTDS" `
    -NoRebootOnCompletion:$false `
    -SysvolPath "C:\Windows\SYSVOL" `
    -Force:$true
```

### PowerShell: Configuring DHCP Scope
```powershell
# 1. Install the DHCP Server Role
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# 2. Add DHCP Scope and authorize it in Active Directory
Add-DhcpServerInDC -DnsName "SRV-DC-01.corp.contoso.com" -IPAddress 192.168.10.10
Add-DhcpServerv4Scope -Name "Corp-Scope" -StartRange 192.168.10.50 -EndRange 192.168.10.200 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionValue -DnsServer 192.168.10.10 -DnsDomain "corp.contoso.com" -ScopeId 192.168.10.0
```

### GUI Path
> Server Manager -> Add Roles and Features -> Active Directory Domain Services
> Server Manager -> Tools -> Active Directory Users and Computers (ADUC)
> Administrative Tools -> DNS Manager -> Forward Lookup Zones

---

## Real-World Scenarios

### Scenario 1: Domain Join Fails with Network Path Error
**User Complaint**: You are trying to join `CLI-WIN-10` to the domain `corp.contoso.com`, but the Windows client displays the error: "An Active Directory Domain Controller (AD DC) for the domain corp.contoso.com could not be contacted. Verify the name is typed correctly."
**Your First 3 Checks**:
1. Check the client's current IP address and configuration.
2. Verify you can ping the domain controller's IP address (`192.168.10.10`) from the client.
3. Check the DNS Server settings configured on the client's network adapter.
**Diagnosis Steps**:
1. Open Command Prompt on `CLI-WIN-10` and run `ipconfig /all`.
2. Observe that the client's IP is `192.168.1.55` and the DNS Server is pointing to `192.168.1.1` (your home internet router).
3. Ping `corp.contoso.com` and notice it fails to resolve.
**Root Cause**: The client workstation is using a home router as its DNS Server. The home router does not host the `_msdcs.corp.contoso.com` SRV records, making it impossible for the client to discover the Domain Controller.
**Fix**:
1. Open Network Connections (`ncpa.cpl`) on the client.
2. Edit IPv4 properties on the network adapter and set the Preferred DNS Server manually to the DC's IP: `192.168.10.10`.
3. Try joining the domain again. Once joined, reboot the client.
**Prevention**: Ensure all lab clients are bound to the Host-Only network where the Domain Controller’s DHCP server is configured to lease `192.168.10.10` as the primary DNS option.

### Scenario 2: GPO Wallpaper Deployment Fails on Client
**User Complaint**: You created a Group Policy Object to deploy a corporate wallpaper (`corp_bg.jpg`) to all workstations, but users in the "Marketing" OU are still seeing the default Windows background.
**Your First 3 Checks**:
1. Verify the GPO is linked to the correct OU containing the target user/computer objects.
2. Check if GPO inheritance is blocked on the target OU.
3. Check the client machine's policy logs using `gpresult /h`.
**Diagnosis Steps**:
1. Open Group Policy Management (`gpmc.msc`) on `SRV-DC-01`.
2. Confirm the policy "Deploy-Wallpaper" is linked to the "Marketing" OU.
3. On the client workstation, run `gpresult /h C:\temp\gpreport.html` and open it.
4. The report shows the "Deploy-Wallpaper" GPO status is marked as: **Denied (Security Filtering)**.
**Root Cause**: The GPO's security filtering was modified to target a specific user group, but the target users are not members of that group, blocking the policy from applying.
**Fix**:
1. Open Group Policy Management -> Select "Deploy-Wallpaper" GPO -> Delegation -> Advanced.
2. Verify "Authenticated Users" or the specific "Marketing-Users" group has both **Read** and **Apply group policy** permissions checked.
3. Run `gpupdate /force` on the client PC to pull the updated policies.
**Prevention**: Never remove "Authenticated Users" from GPO security filtering without explicitly adding the target security group and verifying that "Domain Computers" has read permissions to read the GPO files.

---

## Critical Points

> [!danger] Never Do This
> Do not configure a Domain Controller to use a dynamic IP address. If the DC's IP address changes via DHCP, all client machines will lose active DNS resolution, breaking the entire domain's authentication pipeline.

> [!warning] Common Trap
> Renaming the DNS domain prefix (e.g. from `.com` to `.org`) after promoting a Domain Controller. While possible using domain rename tools, it is extremely risky and can break internal application integrations. Set the FQDN carefully before running the promotion wizard.

> [!tip] Senior Engineer Tip
> Enable the **AD Recycle Bin** immediately after setting up a new domain. By default, it is disabled. Enabling it allows you to restore accidentally deleted users, groups, and OUs with their full SIDs and group memberships intact, without needing a full system backup restore.

> [!success] Verification Steps
> To verify successful Domain Controller promotion:
> 1. Run `dcdiag` in Command Prompt to execute directory diagnostic health checks.
> 2. Ensure all tests (specifically Connectivity, Advertising, and Services) pass with zero errors.

> [!question] Interview Alert
> "What are SYSVOL and NTDS.dit?"
> - **Answer**: `NTDS.dit` is the database file that stores all Active Directory objects, passwords, and security data. `SYSVOL` (System Volume) is a shared folder replicated to all DCs containing GPO templates, startup/shutdown scripts, and group policy files.

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| DNS records not updating | Dynamic updates set to None in DNS zone | Set DNS dynamic updates to Secure Only for Active Directory-integrated zones. |
| Time sync errors on clients | DC clock skew (time mismatch with hypervisor host) | Sync the DC's clock to a reliable external NTP server (e.g., `time.windows.com`). |
| Missing admin permissions | User account not added to Domain Admins | Add administrative users to the Domain Admins built-in security group under the Users container. |

---

## Lab Exercise

**Objective**: Install AD DS, Promote to Domain Controller, and Join a Client VM to the Domain.
**Time Required**: 45 minutes
**Environment Needed**: VM 1 (Windows Server 2022) and VM 2 (Windows 10/11) on a Host-Only virtual switch.

**Steps**:
1. **Configure SRV-DC-01**:
   - Set IP static to `192.168.10.10`, subnet `255.255.255.0`, DNS to `127.0.0.1`.
   - Install AD DS role and promote to forest: `corp.contoso.com`. Let the system reboot.
2. **Verify DNS Zone**:
   - Open DNS Manager and verify `corp.contoso.com` and `_msdcs.corp.contoso.com` zones are populated.
3. **Provision Client VM**:
   - Boot VM 2. Set IP manually to `192.168.10.50`, subnet `255.255.255.0`, DNS to `192.168.10.10`.
4. **Execute Domain Join**:
   - On VM 2, go to Settings -> About -> Rename this PC (advanced) -> Change.
   - Select Domain: `corp.contoso.com`.
   - Enter Domain Administrator credentials when prompted.
   - Reboot client VM.

**Pass Criteria**: VM 2 successfully prompts with "Welcome to the corp.contoso.com domain" and is visible inside the ADUC "Computers" container on VM 1.

---

## Interview Questions & Answers

### L1 Level

#### Q1: What is a Group Policy Object (GPO) and how do you force it to update?
**A**: A GPO is a set of rules and configuration templates used by administrators to manage user and computer settings across an Active Directory domain. By default, client PCs refresh these policies every 90 minutes. I can force an immediate update by running the command `gpupdate /force` in CMD or PowerShell on the client machine.

#### Q2: What is the difference between a Local User and a Domain User?
**A**: A Local User account is stored in the local Security Accounts Manager (SAM) database on a single computer and can only log in to that specific machine. A Domain User account is stored centrally in the Active Directory database on the Domain Controller, allowing the user to log in to any workstation joined to the domain.

---

### L2 Level

#### Q3: What is the difference between GPO Enforced and GPO Block Inheritance?
**A**:
- **Block Inheritance**: Applied to an Organizational Unit (OU) to prevent policies linked higher up the AD tree from inheriting down to the child OU.
- **Enforced (Link Enforced)**: Applied to a GPO link to override any inheritance blocks. An Enforced GPO will apply to users and computers inside an OU even if Block Inheritance is toggled on.

#### Q4: How does a client machine locate a Domain Controller to log in?
**A**: When a user inputs their domain credentials, the client workstation queries its configured DNS server for SRV records matching the domain name (specifically looking for `_ldap._tcp.dc._msdcs.DomainName`). The DNS server returns the IP addresses of available Domain Controllers. The client then contacts the DC via LDAP and Kerberos to authenticate the user.

---

### L3 Level

#### Q5: Explain how you would recover an accidentally deleted Active Directory user account without restoring from backup.
**A**:
- **Situation**: An administrator accidentally deleted a critical service account in ADUC, disrupting local printer integrations.
- **Task**: Restore the deleted user account immediately with its original permissions and Security Identifier (SID).
- **Action**: I opened Administrative Center (`dsac.exe`) and verified the AD Recycle Bin was enabled. I navigated to the Deleted Objects container, located the deleted service account, right-clicked, and selected Restore.
- **Result**: The account was restored in seconds, keeping its original group memberships and SID. This avoided access permission resets and prevented system downtime.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **DC Static Configuration**: Static IP is required. Loopback IP (`127.0.0.1`) is configured as primary DNS on the DC itself.
> **DNS Role**: Active Directory depends on SRV records stored in the `_msdcs` zone.
> **Dynamic Routing**: DHCP option 6 must point to the DC IP so clients receive DNS assignments automatically.
> **Diagnostic Tools**: `dcdiag` (DC health), `gpresult /r` (applied policies), `ipconfig /flushdns` (clear DNS cache).

---

## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/Domain-Join|Domain Join]]
- [[03-Identity-and-Core-Services/05-Windows-Server/Windows-Server-2022-Introduction|Windows Server 2022 Introduction]]
- [[03-Identity-and-Core-Services/06-Active-Directory/FSMO-Roles|FSMO Roles]]

---

## Tags
#desktop-support #active-directory #dns-configuration #dhcp-scope #domain-controller #windows-server #lab-complete #L2 #L3

