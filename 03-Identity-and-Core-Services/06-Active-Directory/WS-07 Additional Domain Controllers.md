---
tags: [sysadmin, windows-server, active-directory, replication]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# WS-07: Additional Domain Controllers

> [!abstract] Overview
> This note covers deployment and replication of redundant Domain Controllers, AD Sites and Services scheduling, diagnostic commands (repadmin), Read-Only Domain Controllers (RODC), and Password Replication Policies (PRP).

---
## Concept
Think of a single Domain Controller as the only notary public officer in a city. If the office closes or gets busy, long queues form and people cannot sign their contracts. 

To solve this, you build second and third notary offices (Additional Domain Controllers) in different areas. To ensure they all certify the same documents, they send couriers back and forth to sync their registry logbooks (AD Replication). 

If you open a small branch office in a high-risk area where security is weak, you deploy a **Read-Only Domain Controller (RODC)**. This office gets a copy of the logbook but is not allowed to make any changes to it. To protect sensitive data, they do not even store the master keys (passwords) for corporate headquarters staff, unless specifically authorized (Password Replication Policy).

*Seedha simple mein: Redundancy aur load balancing ke liye domain mein multiple DCs deploy kiye jaate hain. AD replication in DCs ko sync rakhta hai. RODC unsafe locations (jaise branch offices) ke liye design kiya gaya read-only DC hai jahan security risk ho.*

---
## Technical Deep Dive

### 1. Multi-Master Replication & KCC
Active Directory uses a **Multi-Master Replication** model. Updates can be written to any Domain Controller (except RODCs) and will replicate to all other DCs.
- **KCC (Knowledge Consistency Checker):** A built-in process that runs on all DCs, automatically generating the most efficient replication path topology (ring structure). If a DC goes offline, the KCC automatically recalculates paths to route around the dead host.
- **Urgent Replication:** Certain critical updates (such as account lockouts or password changes) bypass normal replication schedules and are replicated immediately across the domain.

### 2. Active Directory Sites and Services
Sites represent the physical subnet boundaries of your network.
- **Intra-Site Replication:** Occurs between DCs within the same Site. Assumes high-bandwidth LAN connectivity. Replication occurs almost instantly (within 15 seconds) and is uncompressed.
- **Inter-Site Replication:** Occurs between DCs in different Sites connected by slow WAN links. Replication is compressed to save bandwidth and runs on a scheduled interval (Default: every 180 minutes, minimum 15 minutes).
- **IP Site Links:** Logical paths linking sites, configured with **Cost** values to control which WAN paths replication traffic should prefer.

### 3. Read-Only Domain Controllers (RODC)
An RODC holds a read-only partition of the Active Directory database.
- **Unidirectional Replication:** Updates replicate from writeable DCs to the RODC. No changes can be written directly to the RODC, securing the domain if the RODC is physically compromised.
- **No Private Keys by Default:** To protect credentials, an RODC does not cache user passwords. When a user authenticates, the RODC proxies the request to a writeable DC.
- **Password Replication Policy (PRP):** Defines which user/computer passwords can be cached on the RODC.
  - **Allowed List:** Typically contains branch office local users/computers so they can log in if the WAN link to headquarters goes down.
  - **Denied List (Default):** Contains Domain Admins, Enterprise Admins, Schema Admins, and other high-privilege groups. Their credentials are **never** cached on an RODC.

---
## Cisco/Windows Deployment Configuration Commands

### Promoting a Second DC (PowerShell Script)
Before running, ensure DNS on the second server points to the primary DC IP (`192.168.10.10`).
```powershell
# Install AD DS binaries
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote as additional DC in existing domain "company.local"
Install-ADDSDomainController `
    -DomainName "company.local" `
    -Credential (Get-Credential) `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "RestoreP@ssword1" -AsPlainText -Force) `
    -DatabasePath "C:\Windows\NTDS" `
    -LogPath "C:\Windows\NTDS" `
    -SysvolPath "C:\Windows\SYSVOL" `
    -Force -Restart
```

### Active Directory Replication Management (repadmin)
Use `repadmin` to monitor and force synchronization:

```cmd
:: Windows Command Prompt (Run on DC as Administrator)
:: Check replication status overview for all domain controllers
repadmin /replsummary

:: Show detailed replication partners and last success/failure status for target DC
repadmin /showrepl

:: Force immediate replication synchronization from all partners
repadmin /syncall /Adep
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A writeable Domain Controller (`SVR-DC01` at `192.168.10.10`) and a second Server (`SVR-RODC02` at `192.168.10.11`) configured with static IPs on the domain `company.local`.

### Step 1: Deploy an RODC
1. On `SVR-RODC02`, open Server Manager -> Install **Active Directory Domain Services** role.
2. Click **Promote this server to a domain controller**.
3. In Deployment Configuration:
   - Select **Add a domain controller to an existing domain**.
   - Domain: `company.local`. Enter Administrator credentials. Click Next.
4. Domain Controller Options:
   - Check **Read-only domain controller (RODC)**.
   - Select **Domain Name System (DNS) server** and **Global Catalog (GC)**.
   - Enter DSRM password. Click Next.
5. RODC Options:
   - Delegated administrator account: Enter the local branch technician username (allows them to manage local server OS without domain rights).
   - Review default **Password Replication Policy** list. Click Next.
6. Verify paths, pass checks, and click **Install**. Reboot completes promotion.

### Step 2: Configure Password Replication Policy (PRP)
1. Go to `SVR-DC01` (writeable DC). Open **Active Directory Users and Computers**.
2. Click the **Domain Controllers** container. Locate `SVR-RODC02`.
3. Right-click `SVR-RODC02` -> **Properties**.
4. Go to the **Password Replication Policy** tab.
5. Click **Add**.
6. Select **Allow passwords for the following accounts to be replicated to this RODC**.
7. Add a security group containing branch office users (e.g., `G_Branch_Users`).
8. Click OK and Apply.
9. **Verify:** Go to the **Advanced** tab of the PRP settings. Click **Show Authenticated Accounts** to monitor which branch passwords have been dynamically cached on the RODC after their first login.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Users at a branch office report they cannot log in if the WAN connection to headquarters is offline. The local RODC is online.
- **Root Cause:** The branch users' passwords are not cached on the RODC because they are not added to the Allowed list of the Password Replication Policy, or they have never logged in while the WAN was active.
- **Fix:**
  1. Log into a writeable DC at headquarters.
  2. Open ADUC -> Right-click the RODC properties -> Password Replication Policy.
  3. Ensure the branch users' group is added to the **Allowed** list.
  4. Force populate the password cache immediately before the WAN goes down:
     - Go to the **Advanced** tab. Click **Prepopulate Passwords**.
     - Enter the branch users group or usernames. Click OK.
     - The writeable DC securely pushes the hashes to the RODC cache immediately.

**Scenario 2:**
- **Problem:** Run `repadmin /showrepl` reveals replication errors: `Error 8452: The replication operation failed because the naming context is in the process of being removed or is not replicated from the specified server.`
- **Root Cause:** Stale Active Directory replication metadata or severe time synchronization drift (Kerberos time skew limit exceeded 5 minutes).
- **Fix:**
  1. Verify time on both DCs. Run `w32tm /query /status`. If mismatch exceeds 5 minutes, force time synchronization on the failing DC:
     ```cmd
     net stop w32time
     w32tm /config /syncfromflags:domhier /update
     net start w32time
     w32tm /resync /force
     ```
  2. If replication still fails, force KCC topology recalculation:
     ```cmd
     repadmin /kcc *
     ```
  3. Re-run `repadmin /syncall /Adep` and check results.

---
## Common Mistakes
> [!warning] Avoid These
> **Adding Domain Admins to the RODC Allowed PRP List:** Adding high-privilege administrators to the Allowed PRP list on an RODC in an unsecure physical branch office. If the RODC server is physically stolen, attackers can extract the Domain Admin hashes from the disk, compromising the entire forest.
> **Correct approach:** Keep high-privilege administrative groups on the Denied PRP list. Only allow low-privilege local branch user accounts to cache passwords on local RODCs.

---
## Pro Tips
> [!tip] Field Experience
> When removing a dead Domain Controller that cannot be powered back on, do not just delete the VM. You must perform a **Metadata Cleanup** in Active Directory Users and Computers (deleting the DC object in the Domain Controllers container triggers automatic cleanup in Server 2008 and newer) to ensure the remaining DCs stop attempting to replicate with the dead host, preventing replication timeouts.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | KCC | Built-in directory process that dynamically builds the replication topology ring for DCs. |
| 2 | Sites | Subnet-based containers that configure and schedule replication boundaries across WAN links. |
| 3 | RODC | Read-only Domain Controller that stores a non-writable AD database and lacks admin credentials. |
| 4 | PRP | Password Replication Policy; dictates which user hashes can be cached on a local RODC. |
| 5 | repadmin | The primary command line diagnostic suite used to monitor and force AD replication. |

---
## Interview Q&A

**Q1: How does Active Directory replication differ between Intra-Site and Inter-Site configurations?**
A: **Intra-Site replication** occurs between domain controllers within the same physical site (LAN). It assumes high bandwidth, does not compress data, and triggers replication dynamically via change notifications (almost instantly, within 15 seconds). **Inter-Site replication** occurs between domain controllers in different sites connected by slower WAN links. It compresses replication data up to $90\%$ to save bandwidth, does not use change notifications, and runs on a scheduled interval (Default 180 minutes, minimum 15 minutes) configured in AD Sites and Services.

**Q2: A branch office server room is located in a warehouse with poor physical security. What DC deployment model do you recommend, and how do you protect corporate credentials?**
A: 
- **Situation:** A domain controller must be deployed in a physically unsecure location.
- **Task:** Recommend the safest deployment profile and configure credential protections.
- **Action:** I will recommend deploying a **Read-Only Domain Controller (RODC)**. This ensures that no changes can be written back to the primary database if the server is compromised. To protect credentials, I will ensure that the default Password Replication Policy (PRP) is enforced, keeping Domain Admins and high-privilege accounts on the Denied List.
- **Result:** Only local warehouse workers are added to the Allowed PRP list, ensuring that a physical compromise of the server only exposes low-privilege warehouse hashes and does not compromise the main corporate forest.

**Q3: Describe what AD Metadata Cleanup is, and when you must perform it.**
A: Active Directory Metadata Cleanup is the process of manually removing stale registration records and database objects left behind by a Domain Controller that has been permanently retired or destroyed without running the standard promotion decommissioning process (e.g., due to a hardware crash or motherboard failure). Performing metadata cleanup removes the DC's NTDS Settings object, clean up replication links from the KCC, and removes stale SRV records from DNS, preventing healthy DCs from attempting to replicate with the dead host.

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — Core Domain Controller promotion steps.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-08 FSMO Roles|WS-08 FSMO Roles]] — Domain role management.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-02 PowerShell for Active Directory|PS-02 PowerShell for Active Directory]] — Scripting replication check runs.

