---
tags: [sysadmin, windows-server, active-directory, domain-controller]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# WS-02: Active Directory Domain Services (AD DS)

> [!abstract] Overview
> This note covers Active Directory Domain Services logical and physical architectures, Domain Controller promotion protocols, replication topologies, and directory databases (NTDS.dit).

---
## Concept
Think of Active Directory as the centralized identity registry and security office of a massive corporation. 

Before AD, every computer had its own local registry of keys. If a new employee joined, the sysadmin had to physically walk to every single computer, desk, and server to create a local username and password for them (Workgroup environment). 

With Active Directory, there is a central corporate registry database (**NTDS.dit**) managed by security offices (**Domain Controllers**). When a user plugs their laptop into the network and logs in, their laptop asks the Domain Controller, "Is this password valid?" The DC checks the central registry, verifies the identity, and grants access to files, emails, and printers throughout the entire corporate network without the user needing local accounts on any of those individual resources.

*Seedha simple mein: Active Directory pure organization ke users, computers, aur permissions ko ek jagah se manage karne ka centralized management tool hai. Domain Controller woh server hota hai jo is directory database (NTDS.dit) ko run karta hai.*

---
## Technical Deep Dive

### 1. Active Directory Logical vs. Physical Structures

#### Logical Components (Logical Architecture)
- **Forest:** The top-level container boundary in AD. It defines the security boundary and shares a common Schema, Configuration partition, and Global Catalog.
- **Tree:** A collection of one or more domains that share a contiguous namespace (e.g., `corp.com`, `uk.corp.com`).
- **Domain:** A logical administrative partition containing users, groups, and computers. It serves as a replication boundary and security policy scope.
- **Organizational Unit (OU):** Sub-containers inside a domain used to group users, computers, and groups. It is the smallest scope to which Group Policies (GPOs) can be linked or control delegated.

#### Physical Components (Physical Architecture)
- **Domain Controller (DC):** A physical or virtual server running the AD DS role. It stores the directory database and authenticates logons.
- **Sites:** A physical group of IP subnets representing a high-bandwidth local network. Used to control replication traffic across slow WAN connections (e.g., London site replicating to New York site over a scheduled link).

### 2. The Active Directory Database (NTDS.dit)
The directory database is located in the `%systemroot%\NTDS\` folder.
- **`ntds.dit`:** The core database file. It uses the Extensible Storage Engine (ESE) database engine. It stores all user objects, hashes, computers, group memberships, and schema metadata.
- **`SYSVOL`:** A shared directory on the DC (replicated via DFS-R to other DCs). It stores Group Policy Templates, logon/logoff scripts, and template files. Labeled share: `\\domain\SYSVOL`.
- **`NETLOGON`:** A legacy folder (now a subfolder inside SYSVOL) used to store startup scripts for backward compatibility. Labeled share: `\\domain\NETLOGON`.

### 3. Basic AD User Management (Reference)
Advanced content only — basics in [[Basic AD user management]]

For standard user modifications, password resets, and account unlocking, refer to the ADUC snap-in or active directories commands.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A clean virtual machine running Windows Server 2022 (Desktop Experience) with a static IP (`192.168.10.10`) and hostname set to `SVR-DC01`.

### Step 1: Install the AD DS Role
1. Open **Server Manager**. Click **Add roles and features**.
2. Select **Role-based or feature-based installation**. Click Next.
3. Select your local server.
4. Check **Active Directory Domain Services**. In the pop-up, click **Add Features**.
5. Click Next through features and confirmation pages, then click **Install**.

### Step 2: Promote Server to a Domain Controller
1. Once installation completes, click the **Notification flag** (yellow triangle) at the top of Server Manager.
2. Click **Promote this server to a domain controller**.
3. In the Deployment Configuration wizard:
   - Select **Add a new forest** (since this is our first DC).
   - Root domain name: `company.local`. Click Next.
4. Domain Controller Options:
   - Select Forest and Domain functional levels (Default: Windows Server 2016).
   - Ensure **Domain Name System (DNS) server** and **Global Catalog (GC)** are checked.
   - Enter a **Directory Services Restore Mode (DSRM)** password (used for offline database recovery, e.g., `RestoreP@ssword1`). Click Next.
5. Skip DNS Options (DNS delegation warning is normal). Click Next.
6. Verify NetBIOS Name (Auto-fills as `COMPANY`). Click Next.
7. Paths: Leave database, log, and SYSVOL paths at default (`C:\Windows\NTDS` and `C:\Windows\SYSVOL`). Click Next.
8. Review options, verify prerequisites pass (Warnings about default security settings are normal), and click **Install**.
9. The server will automatically reboot to complete the promotion.

### Step 3: Verify the Installation
1. Log in using domain credentials: `COMPANY\Administrator` with the local admin password.
2. Open Server Manager -> Tools. Verify that new snap-ins are visible:
   - Active Directory Users and Computers (ADUC)
   - Active Directory Domains and Trusts
   - Active Directory Sites and Services
3. Open CMD. Verify DNS registration of domain services:
   ```cmd
   nslookup company.local
   ```
   It should return your DC's static IP `192.168.10.10`.

---
## Commands Reference
Active Directory management command sequences:

```powershell
# Windows PowerShell
# Install AD DS role binaries
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote server to a new domain controller in a new forest (unattended script)
Install-ADDSForest -DomainName "company.local" -SafeModeAdministratorPassword (ConvertTo-SecureString "RestoreP@ssword1" -AsPlainText -Force) -Force -Restart

# Get Domain Controller configuration details
Get-ADDomainController -Identity "SVR-DC01"
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Attempting to promote a second domain controller to an existing domain fails, returning error: "Active Directory Domain Services could not locate a Domain Controller for the domain company.local."
- **Root Cause:** DNS configuration mismatch. The new server is targeting an external DNS server (e.g., `8.8.8.8`) on its network adapter, so it cannot resolve the internal `company.local` SRV records that point to the primary DC.
- **Fix:**
  1. Open Network Connections on the second server. Right-click the Ethernet adapter -> Properties -> IPv4.
  2. Modify the **Preferred DNS Server** IP to point to the primary Domain Controller IP (`192.168.10.10`).
  3. Run `ipconfig /flushdns` in CMD.
  4. Rerun the DC Promotion wizard; the domain is now detected.

**Scenario 2:**
- **Problem:** Users report they cannot log in to domain workstations. Event Viewer logs error: "The Security Accounts Manager failed to initialize the database because of a physical error on disk NTDS.dit."
- **Root Cause:** Corrupted `ntds.dit` database file caused by sudden power loss or storage controller crash.
- **Fix:**
  1. Reboot the DC. Press **F8** (or use msconfig) to boot into **Directory Services Repair Mode (DSRM)** using the DSRM credentials configured during setup.
  2. Open Command Prompt as Administrator. Run database integrity check:
     ```cmd
     ntdsutil
     activate instance ntds
     files
     info
     ```
  3. If corruption is found, recover database using `esentutl`:
     ```cmd
     esentutl /g C:\Windows\NTDS\ntds.dit   :: Check integrity
     esentutl /r C:\Windows\NTDS\ntds       :: Attempt recovery
     ```
  4. Reboot the DC normally. If recovery fails, restore from a System State backup.

---
## Common Mistakes
> [!warning] Avoid These
> **Setting dynamic IPs on Domain Controllers:** Allowing a Domain Controller to obtain its IP address via DHCP. If the IP address changes on DHCP lease renewal, all member workstations lose connection to the DC, breaking domain logins, group policies, and DNS lookups.
> **Correct approach:** Always assign a fixed, static IP address to every Domain Controller before installing the AD DS role.

---
## Pro Tips
> [!tip] Field Experience
> When designing an Active Directory environment, never name your internal domain identical to your public website (e.g., using `company.com` for both internal and external). This creates a "Split-Brain DNS" nightmare where internal users cannot resolve the external corporate website unless you manually duplicate web server A-records on the AD DNS zone. Use a dedicated sub-domain (e.g., `corp.company.com` or `ad.company.com`).

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Forest | The top-level administrative and security boundary within an Active Directory hierarchy. |
| 2 | NTDS.dit | The core database file storing all directory objects, user accounts, and password hashes. |
| 3 | SYSVOL | Replicated share folder storing GPOs, logon scripts, and AD system template configurations. |
| 4 | Sites | Logical groups of subnets defining physical network boundaries for replication control. |
| 5 | DSRM | Boot mode used to restore or repair the offline Active Directory database files. |

---
## Interview Q&A

**Q1: What is the difference between a Workgroup and an Active Directory Domain environment?**
A: In a **Workgroup**, the network is decentralized. Every computer maintains its own local security database (SAM) containing local user accounts. There is no centralized management; if a user needs access to three computers, they must have accounts created manually on all three. In an **Active Directory Domain**, the network is centralized. A central database (NTDS.dit) on a Domain Controller stores all accounts and permissions. Users authenticate once against the DC, and can access any authorized domain member workstation or server.

**Q2: A client wants to deploy a new domain controller. During prerequisites check, the installer warns that the Forest Functional Level is Windows Server 2016. Explain what this means.**
A: 
- **Situation:** A new Domain Controller installation reports functional level limits.
- **Task:** Explain functional levels and how they impact server compatibility.
- **Action:** I will explain that Domain/Forest Functional Levels dictate which Windows Server operating systems can be deployed as Domain Controllers in the forest, and which advanced AD features are active. A Windows Server 2016 functional level means only servers running Windows Server 2016 or newer (including 2019 and 2022) can act as DCs. Legacy servers (like Server 2012 R2) are blocked.
- **Result:** Keeping the functional level at 2016 supports modern Server 2022 DCs while maintaining compatibility if the client still runs active 2016 DCs.

**Q3: Describe the role of the Global Catalog (GC) in an Active Directory forest.**
A: The Global Catalog is a Domain Controller that stores a full replica of all objects in its local domain, plus a partial, read-only replica of all objects in all other domains in the forest. It indexes commonly searched attributes (like logon names, emails, and phone numbers). The GC is critical because: 1) It enables users to search for directory information across the entire forest regardless of which domain they are in, and 2) It verifies Universal Group memberships during the logon process to generate the user's security token.

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-01 Windows Server 2022 Introduction|WS-01 Windows Server 2022 Introduction]] — Base server setup and Admin Center tools.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-03 DNS Server — Install and Configure|WS-03 DNS Server — Install and Configure]] — Resolving DC SRV records.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-06 Active Directory — Users Groups OUs|WS-06 Active Directory — Users Groups OUs]] — User accounts and security structures.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-08 FSMO Roles|WS-08 FSMO Roles]] — Directory role management.

