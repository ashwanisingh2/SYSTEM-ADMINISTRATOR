---
tags: [desktop-support, active-directory, identity, L3]
aliases: [ad-replication, ad-replication]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# Active Directory Replication

---

---
## Concept Overview
- **What it is**: Active Directory Replication is the mechanism by which changes made on one Domain Controller (like password resets, user creations, or group policy modifications) are copied and synchronized across all other Domain Controllers in the forest. It uses a multi-master database replication model.
- **Why it matters for a support engineer**: Replication delays are a frequent cause of ticket escalations, such as when a user resets their password in one office branch but cannot log in from a secondary branch because the local DC has not yet received the update.
- **Where you encounter this in real job**: Troubleshooting password synchronization delays across geographic locations, validating new Group Policy deployments, and resolving replication blocks when a DC goes offline.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Collects user symptoms (e.g., password reset doesn't work on one machine but works on web mail), determines the client's local Domain Controller, and escalates if needed.
  - **L2**: Uses standard cmdlets (`repadmin`) to verify basic replication status and forces replication manually to resolve immediate logon blocks.
  - **L3**: Configures Active Directory Sites and Services, manages IP subnets and site links, fixes complex synchronization issues (like USN Rollbacks and Lingering Objects), and monitors replication health forest-wide.

---

---
## Technical Deep Dive
### 1. Replication Topology: Intra-Site vs. Inter-Site
Active Directory organizes replication based on **Sites** (which represent physical IP subnets connected by high-speed networks):

| Feature | Intra-Site Replication | Inter-Site Replication |
|---|---|---|
| **Location** | Within the same AD Site (local LAN) | Between different AD Sites (WAN / VPN) |
| **Trigger** | **Change Notification**: DC notifies partners 15 seconds after a change is made. | **Scheduled**: Replication occurs at specified intervals (default: 180 minutes, min: 15 minutes). |
| **Compression** | Uncompressed (saves CPU resources) | Highly compressed (saves WAN bandwidth) |
| **Protocols** | RPC over IP | IP (RPC) or SMTP (only for Schema/Configuration) |
| **Topology Maker** | Knowledge Consistency Checker (KCC) | Inter-Site Topology Generator (ISTG) |

- **KCC (Knowledge Consistency Checker)**: A built-in process that runs every 15 minutes on all DCs. It automatically generates and maintains the replication topology (connection objects) to ensure there are no single points of failure.
- **Subnets**: In AD, subnets are mapped to specific Sites. If a computer's IP address falls into a subnet not associated with any site, it defaults to the primary site, which can cause replication and logon delays.

### 2. Update Sequence Numbers (USN) & USN Rollback
- **USN**: Each Domain Controller maintains an Update Sequence Number (USN) in its registry. Every change increments this number. DCs use USNs to track what updates they have already received from replication partners.
- **USN Rollback**: Occurs when a DC virtual machine is restored from an unsupported VM snapshot or backup. 
  1. The restored DC reverts to an older database state with a lower USN.
  2. The restored DC makes changes, assigning USN values it previously used.
  3. Partner DCs detect the duplicate USN sequences but refuse to replicate, recognizing a database inconsistency.
  4. The rolled-back DC becomes isolated, failing to replicate incoming or outgoing changes.

### 3. Lingering Objects
- **Tombstone Lifetime (TLT)**: When an object is deleted in AD, it is not erased immediately. Instead, it is marked as a "Tombstone" and kept for a set period (default 180 days) to replicate the deletion to all DCs.
- **Lingering Object**: If a DC is kept offline longer than the Tombstone Lifetime and then brought back online:
  1. It missed the deletion replication.
  2. It still contains the deleted objects in its database.
  3. It may replicate these obsolete "lingering" objects back to other healthy DCs, causing deleted users or computers to mysteriously reappear in the directory.

---


## Real-World Scenarios

### Scenario 1: Password Reset Fails to Apply in Branch Office
**User Complaint:** A sales manager in the New York branch resets their password via self-service. They immediately try to log in to their local desktop in the Chicago branch but get: *"The password you entered is incorrect."* They can log in successfully to the web interface.
**Your First 3 Checks:**
1. Identify the user's authenticating DC in Chicago.
2. Check replication latency between the New York DC and the Chicago DC.
3. Determine if the PDC Emulator is reachable from the Chicago DC.
**Diagnosis Steps:**
1. Run `gpresult /r` on the client PC to identify the local DC. (Output: `DC-CHI-01`).
2. Run `repadmin /replsummary` to check replication status between the hubs.
   - NYC and Chicago are in different AD Sites. 
   - Inter-site replication is scheduled for every 180 minutes. The password was reset 5 minutes ago, so the update has not arrived.
3. *Why did the PDC fallback fail?* The WAN link between Chicago and the PDC Emulator in NYC was saturated, causing the local Chicago DC to timeout when attempting to verify the credentials against the PDC Emulator.
**Root Cause:** Inter-site replication schedule delay, combined with temporary WAN latency preventing real-time PDC credential verification.
**Fix:**
1. Force an immediate replication block sync from the NYC DC to Chicago:
   `repadmin /syncall DC-CHI-01 /Adpe`
2. Once the replication finishes, have the user log in.
**Prevention:** Reduce the inter-site replication interval for critical branch office site links from 180 minutes to 15 minutes, or enable "Change Notification" on WAN site links to replicate immediately.
**Ticket Close Note:** "Forced replication from NYC to Chicago DC using repadmin. Verified user could log in. Recommended shortening site link replication intervals. Closed."

### Scenario 2: Active Directory Replication Fails with Event ID 2042 (Tombstone Lifetime Exceeded)
**User Complaint:** The replication monitoring system alerts that `DC-BRANCH-05` has stopped replicating. Standard `repadmin /showrepl` displays the error: *"The backup latch limit has been exceeded or the tombstone lifetime has expired."*
**Your First 3 Checks:**
1. Check the date/time on the failing Domain Controller.
2. Find the date of the last successful replication in `repadmin /showrepl`.
3. Check the configured Tombstone Lifetime of the forest.
**Diagnosis Steps:**
1. Run `repadmin /showrepl DC-BRANCH-05`.
   - Error: `Last replication attempt was 192 days ago.`
2. The forest Tombstone Lifetime is set to the default `180 days`.
3. The server was powered down during a building renovation for over 6 months. Now that it is powered back on, it has been offline longer than the Tombstone Lifetime. Replication is blocked by safety controls to prevent deleted objects from resurrecting (Lingering Objects).
**Root Cause:** A Domain Controller was offline longer than the Tombstone Lifetime, prompting AD safety mechanisms to disable its replication.
**Fix:**
1. **Do not attempt to replicate.** Force-demote the offline Domain Controller.
2. Run Metadata Cleanup on a healthy DC to remove all traces of `DC-BRANCH-05` from the Active Directory database.
3. Clean-install the OS on the branch server, rename it, and promote it as a fresh Domain Controller.
**Prevention:** Monitor Domain Controller health. If a DC is offline for more than 60 days, alert administrators, and demote it before it hits the Tombstone limit.
**Ticket Close Note:** "Forcefully demoted DC-BRANCH-05 due to Tombstone Lifetime expiry. Completed metadata cleanup on DC-01. Re-promoted branch server as a clean DC. Closed."

---


## Critical Points

> [!danger] Never Do This
> - Never restore a virtualized Domain Controller using standard hypervisor snapshots unless the virtualized DC supports VM Generation ID (Windows Server 2012 and later, running on supported hypervisors like Hyper-V 2012 or ESXi 5.0+).
> - Restoring an older snapshot on an unsupported platform causes a **USN Rollback**, isolating the DC and risking silent corruption of the entire AD database.

> [!warning] Common Trap
> - Assuming that forcing replication with `repadmin /syncall` works instantly if DNS is broken.
> - If name resolution between DCs is failing, `repadmin` will fail with "RPC Server Unavailable" or "DNS Lookup Failed". Always verify DNS resolution using `nslookup` before running replication sync commands.

> [!tip] Senior Engineer Tip
> - Enable **Strict Replication Consistency** on all Domain Controllers. This setting prevents a DC from receiving updates from a partner that contains lingering objects. Instead of accepting old, deleted objects, the healthy DC will terminate replication with that partner, isolating the issue.
> - Registry Path: `HKLM\System\CurrentControlSet\Services\NTDS\Parameters\Strict Replication Consistency` set to `1`.

> [!success] Verification Steps
> - Run `repadmin /replsummary` to get a quick matrix of all DCs. Check for `0` source/destination fails.
> - Check that local changes (e.g., adding a test security group) appear on a remote DC within the expected replication window.

> [!question] Interview Alert
> - "What tool do you use to force replication between two Domain Controllers, and what is the syntax?"
> - Answer: "I use `repadmin.exe`. To force replication across the entire domain, I run `repadmin /syncall /Adpe`. The `/A` syncs all partitions, `/d` identifies servers by distinguished name, `/p` pushes changes, and `/e` syncs across different AD sites."

---


## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Restoring DC from raw VM snapshots | Lack of backup training | Restore Domain Controllers using System State backups, or deploy a clean server and let AD replicate. |
| Neglecting to configure Subnets in Sites | Assuming AD auto-detects topography | Always map local subnets to their respective AD sites so clients find the closest DC and replication routes are correct. |
| Forcing sync while network is down | Misunderstanding WAN failure root causes | Identify physical network outages or firewall blocks (RPC ports blocked) before forcing replication. |

---


## Tags
#desktop-support #active-directory #system-administration #replication #L3 #interview-topic #lab-complete #daily-use

---
## Step-by-Step Lab
**Objective:** Inspect AD Replication status, create a test account, and force replication to a secondary Domain Controller using Command Line tools.
**Time Required:** 20 minutes
**Environment Needed:** Two Windows Server VMs acting as Domain Controllers in a test lab.
**Pre-requisites:** AD replication currently active between both servers.

**Steps:**
1. Open Command Prompt as Administrator on `DC-01`.
2. Run a replication health check:
   ```cmd
   repadmin /showrepl
   ```
   - Verify that all inbound replication connections show "Successful".
3. Check the replication summary:
   ```cmd
   repadmin /replsummary
   ```
4. Create a test user named "Repl Test" in Active Directory Users and Computers on `DC-01`.
5. Open PowerShell on `DC-02` and search for the user:
   ```powershell
   Get-ADUser -Filter "Name -like 'Repl Test'"
   ```
   - If not found immediately due to replication delay, run the sync command on `DC-02`:
   ```cmd
   repadmin /syncall /Adpe
   ```
6. Re-run the search on `DC-02` to verify that the user now appears.

**Success Criteria:** Replication checks report zero errors, and the newly created test user replicates from `DC-01` to `DC-02` successfully.
**Common Failures:** The sync command fails with RPC error if firewall rules block TCP ports 135 and the dynamic RPC range (49152-65535) between DCs.

---

---
## Cheat Sheet / Quick Reference
### PowerShell
```powershell
# Get replication metadata status for a Domain Controller
Get-ADReplicationPartnerMetadata -Target "DC-01.company.local"

# Force replication of a specific Active Directory object immediately
Sync-ADObject -Object "CN=John Doe,OU=Users,DC=company,DC=local" -Source "DC-01" -Destination "DC-02"

# View replication failures for all Domain Controllers in the forest
Get-ADReplicationFailure -Target "company.local" -Scope Forest
```

### CMD / Run Box (Repadmin Tool)
`repadmin` is the primary command-line tool for diagnosing and managing Active Directory replication.
```cmd
:: Check replication summary across all Domain Controllers
repadmin /replsummary

:: Show detailed replication partner status and last replication time
repadmin /showrepl

:: Force a pull replication on a specific DC from all its replication partners
repadmin /syncall /Adpe

:: Verify if there are any lingering objects on a target DC
repadmin /removelingeringobjects DC-01.company.local dc-guid CN=Configuration,DC=company,DC=local
```

### GUI Path
- Open **Active Directory Sites and Services** (`dssite.msc`):
  - Expand **Sites** -> Expand Target Site -> **Servers** -> Expand Server Name -> Click **NTDS Settings**.
  - Right-click connection objects in the right pane -> Click **Replicate Now**.

### Key Event IDs
Replication errors are logged in the **Directory Service** log in Event Viewer:

| Event ID | Cause / Meaning | Troubleshooting Action |
|----------|-----------------|------------------------|
| 1311 | KCC cannot compute a replication topology (Network partition) | Check physical network routing and site link configurations. |
| 1864 | Replication latency warning (A DC hasn't replicated in a long time) | Verify the target DC is powered on and DNS is working. |
| 2042 | Tombstone Lifetime exceeded (Lingering Object danger) | Disconnect DC immediately; remove lingering objects or demote/rebuild. |
| 2095 | USN Rollback detected on a virtualized DC | Immediately demote the DC and perform metadata cleanup; do not attempt repair. |

---

> [!info] 60-Second Summary
> **What**: The process of synchronizing changes across all Domain Controllers in an AD forest.
> **Why**: Ensures database consistency, allows global logons, and applies policy updates.
> **How**: Local subnets are mapped to Sites. KCC automatically builds the replication topology.
> **Command**: `repadmin /replsummary` / `repadmin /showrepl` / `repadmin /syncall /Adpe`
> **Interview Answer Starter**: "To troubleshoot Active Directory replication issues, my first step is always to run `repadmin /replsummary` to isolate failing partners and then check the Directory Service event logs..."

**Key Numbers to Remember:**
- Default Tombstone Lifetime: 180 days
- Default Inter-Site replication interval: 180 minutes
- Minimum Inter-Site replication interval: 15 minutes
- Intra-Site replication change notification delay: 15 seconds

**3 Things Interviewer Wants to Hear:**
- Using `repadmin` commands to force and verify replication
- The danger of restoring DC VM snapshots (USN Rollbacks)
- The concept of Active Directory Sites, Subnets, and Site Links

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
**Q: Why does a password reset sometimes take 15-30 minutes to work at another office?**
A: This occurs due to replication delay between office locations. If the offices are in different Active Directory Sites, replication happens on a schedule (usually every few hours or 15-30 minutes minimum). Until the replication cycle completes, the local DC in the other office does not have the new password.

**Q: What GUI snap-in do you use to manage IP subnets and physical replication links?**
A: I use the **Active Directory Sites and Services** console. It allows me to define corporate subnets, assign them to sites, and configure inter-site replication links and schedules.

### Intermediate (L2 Level)
**Q: How do you verify and troubleshoot replication health from the command line?**
A: I use the `repadmin` utility. First, I run `repadmin /replsummary` to get a high-level view of replication status and identify any DCs with failures. Then, I run `repadmin /showrepl` on the failing DC to see the exact error codes, such as DNS lookup errors or access denied.

**Q: What is a "Lingering Object" in Active Directory?**
A: A lingering object is a deleted object that remains on a Domain Controller because that DC was kept offline longer than the Tombstone Lifetime (usually 180 days). During this offline period, the DC missed the deletion notification. When it reconnects, it still has the object, which can replicate back to healthy DCs, causing deleted items to reappear.

### Advanced (L3/Senior Level)
**Q: What is a USN Rollback, how does it occur, and how do you resolve it?**
A:
- **Situation**: A virtualized Domain Controller was improperly restored from a snapshot.
- **Task**: Detect USN Rollback and recover the AD database.
- **Action**: A USN Rollback occurs when an administrator restores a DC VM from an old snapshot on an unsupported hypervisor. The DC's database returns to an older state, but its partner DCs believe they have already received updates up to the pre-restored USN. The partner DCs block replication to prevent corruption, generating Event ID 2095 in the Directory Service log.
- **Result**: The rolled-back DC must be forcefully demoted, its metadata cleaned up from AD using `ntdsutil` or ADUC, and then rebuilt from scratch. You should never attempt to force replicate a rolled-back DC.

### HR / Behavioral
**Q: Describe a situation where you had to collaborate with another team to solve a complex network or identity problem.**
A: We had a recurring issue where users in a new remote warehouse experienced login delays of up to 20 minutes. I coordinated with the Network Infrastructure team and checked the IP routing. I found the warehouse subnet had not been added to Active Directory Sites and Services. I worked with our team to add the subnet and bind it to the local warehouse site. This routed client requests to the local DC instead of the headquarter DC, eliminating the login delays.

---

---
## Seedha Simple Mein
*Seedha simple mein: AD-Replication ke bare mein seekhta hai. Yeh active-directory infrastructure aur system settings ko properly implement karne aur support tickets ko runbooks ke help se standard templates me clear karne me help karta hai.*

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/FSMO-Roles|FSMO Roles]] — Details the single-master roles, including PDC Emulator time authority.
- [[01-Foundations/02-Networking/DNS|DNS]] — Outlines the resolution system that allows DCs to find each other.
- [[03-Identity-and-Core-Services/06-Active-Directory/Group-Policy|Group Policy]] — Explains GPO replication via DFS-R alongside AD replication.

---
