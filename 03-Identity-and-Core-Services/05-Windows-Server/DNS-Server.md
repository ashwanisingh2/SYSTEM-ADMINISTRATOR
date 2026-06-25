---
tags: [desktop-support, windows-server, dns, L2]
aliases: [dns-server-role, active-directory-dns]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# DNS Server

---

## Concept Overview
- **What it is**: The Windows Server DNS Server is a network service role that manages the lookup databases of hostnames and IP addresses, providing name resolution services across local Active Directory environments and forwarding external queries to the internet.
- **Why it matters for a support engineer**: Without a healthy DNS server, Active Directory cannot locate Domain Controllers, users cannot log in, and local web applications become unreachable. A support engineer must know how to configure lookup zones, manage dynamic updates, and troubleshoot resolution timeouts.
- **Where you encounter this in real job**: Resolving Active Directory replication failures caused by missing SRV records, configuring conditional forwarders for partner connections, cleaning stale records using scavenging, and adding A/CNAME records for new web servers.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Adds standard host A records, checks local DNS cache status, and runs basic lookup queries.
  - **L2**: Manages Forward and Reverse lookup zones, configures conditional forwarders, and manages zone record scavenging.
  - **L3**: Designs DNS zone replication topologies, configures DNSSEC security validation, and manages parent zone integrations in multi-domain forests.

---

## Technical Deep Dive

### 1. Zone Types & Active Directory Integration
Windows Server DNS supports three primary zone types:
- **Primary Zone**: The master copy of the zone database, stored as a local text file (`zone.dns`) or integrated directly into Active Directory. Only the primary zone can write modifications.
- **Secondary Zone**: A read-only copy of the zone database located on a partner DNS server. It synchronizes changes from the primary zone using **Zone Transfers** (TCP port 53).
- **Stub Zone**: A read-only copy containing only records required to locate the authoritative servers for that zone (SOA, NS, and A records).
- **Active Directory-Integrated Zone**: The zone database is stored as an object inside the Active Directory database (`ntds.dit`). It replicates automatically across all Domain Controllers, utilizes secure dynamic updates, and eliminates the single-point-of-failure of a standard primary zone.

### 2. Dynamic Updates
- **Secure Only**: Only domain-joined computers can register and update their DNS records. This prevents unauthorized systems from hijacking existing records.
- **Nonsecure and Secure**: Allows any client to register its IP address in the DNS database. Represents a security risk and should be disabled in enterprise environments.

### 3. DNS Scavenging (Aging & Cleaning)
Ensures the DNS database does not bloat with stale records (e.g., from old client DHCP leases):
- **No-Refresh Interval**: The duration during which a client cannot refresh its timestamp. Prevents excessive replication traffic (defaults to 7 days).
- **Refresh Interval**: The duration during which a client can refresh its timestamp (defaults to 7 days).
- **Scavenging Period**: The interval at which the server runs a sweep to delete records whose timestamps have expired (No-Refresh + Refresh + Scavenging).

---

## Commands & Syntax

### PowerShell
```powershell
# Create an Active Directory integrated DNS Forward Lookup Zone
Add-DnsServerPrimaryZone -Name "dev.lab.local" -ReplicationScope Forest

# Add a static A record to the local DNS zone
Add-DnsServerResourceRecordA -Name "gitlab" -ZoneName "dev.lab.local" -IPv4Address "192.168.10.75" -CreatePtr
```

### CMD / Run Box
```cmd
REM Launch the DNS Management console directly
dnsmgmt.msc
REM Force the DNS server to register its Active Directory SRV records
ipconfig /registerdns
```

### GUI Path
> Server Manager -> Tools -> **DNS** -> Expand Server node -> Expand **Forward Lookup Zones**.

### Important Registry Paths
```
HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters\
HKLM\SYSTEM\CurrentControlSet\Services\DNS\Zones\
```

### Key Event IDs
DNS audits events in the log path: `Applications and Services Logs\DNS Server`

| Event ID | Meaning | Where to Check |
|----------|---------|----------------|
| 4013 | DNS server is waiting for Active Directory to complete initial replication | DNS Server Log |
| 4015 | DNS server encountered a critical error in Active Directory | DNS Server Log |
| 9943 | DNSSEC validation failed on a queried record | DNS Server Log |

---

## Real-World Scenarios

### Scenario 1: DNS Server Fails to Start on DC Reboot (Event ID 4013 Loop)
**User Complaint:** "We restarted the primary domain controller, and now users cannot log in. The DNS Server service is stuck in a starting state, and Event Viewer shows error 4013."
**Your First 3 Checks:**
1. Check the Event Viewer DNS Server log for Event ID 4013.
2. Check if the domain has another active Domain Controller online.
3. Check the DNS server IP configurations on the DC network adapter.
**Diagnosis Steps:**
1. Open Event Viewer on the DC -> DNS Server log -> Event ID `4013`:
   - Message: `The DNS server was unable to open the Active Directory. The DNS server will check for Active Directory integration every 2 minutes.`
2. Check the DC IP configuration. The preferred DNS server is set to its loopback address (`127.0.0.1`).
3. Since this DC is the only domain controller, it cannot sync its AD database with any partner server, causing the DNS service to hang waiting for AD initialization.
**Root Cause:** The DNS server failed to start because it was waiting for AD DS to initialize, but AD DS was waiting for DNS to resolve name queries, creating a deadlock.
**Fix:** Modify the registry to bypass the initial replication check on startup:
1. Open Registry Editor (`regedit`).
2. Navigate to: `HKLM\SYSTEM\CurrentControlSet\Services\NTDS\Parameters`
3. Create a String value named `Repl Perform Initial Synchronizations` and set it to `0`.
4. Restart the DNS Server service.
**Prevention:** Always maintain at least two Domain Controllers in an enterprise Active Directory domain to avoid startup deadlocks.
**Ticket Close Note:** "Bypassed initial AD replication check via registry key. DNS service started successfully. Verified domain logins active. Closing ticket."

### Scenario 2: Active Directory Replication Fails Due to Missing DNS Records
**User Complaint:** "Changes made on DC01 are not replicating to DC02. The replication report shows a target name resolution error."
**Your First 3 Checks:**
1. Check the replication status using `repadmin /showrepl`.
2. Check if the DC02 hostname resolves to the correct IP address.
3. Test if the Active Directory GUID SRV records are present in DNS.
**Diagnosis Steps:**
1. Run: `repadmin /showrepl` on DC01:
   - Output: `Error: 8524 (The DSA operation is unable to proceed due to a DNS lookup failure).`
2. Run a lookup query for the GUID CNAME record of DC02:
   `nslookup [DC02-GUID]._msdcs.lab.local`
   - Output: `*** Name Server can't find GUID: Non-existent domain.`
3. Open DNS Manager -> Expand `_msdcs.lab.local` zone. The GUID record mapping to DC02 is missing from the database.
**Root Cause:** The DNS dynamic registration failed or the record was deleted, preventing DC01 from resolving the replication target GUID.
**Fix:** Force the Domain Controller to register its SRV and CNAME records in DNS:
1. Log into DC02.
2. Open a command prompt as Administrator.
3. Run the registration command:
   ```cmd
   ipconfig /registerdns
   net stop netlogon && net start netlogon
   ```
4. Verification: Rerun `nslookup` on DC01 to verify the record resolves.
**Prevention:** Ensure the DNS zone allows Secure Dynamic Updates to let DCs register their records automatically.
**Ticket Close Note:** "Forced DNS and Netlogon registration on DC02. Verified DC GUID SRV record registration. AD replication successfully restored. Closing ticket."

---

## Critical Points

> [!danger] Never Do This
> - Configure the DNS Server zone to allow **Nonsecure and Secure** dynamic updates in a corporate environment.
> - This allows any device (including unauthorized guest laptops or rogue systems) to overwrite DNS records, enabling DNS spoofing and man-in-the-middle attacks.
> - Enforce **Secure Only** dynamic updates.

> [!warning] Common Trap
> - Creating a secondary DNS zone and forgetting to configure **Zone Transfer** permissions on the primary server.
> - The secondary server will fail to sync database records, showing a "Zone not loaded by DNS server" error.
> - Explicitly list the secondary server's IP in the Zone Transfers tab on the primary server.

> [!tip] Senior Engineer Tip
> - If an enterprise network has multiple subnets, enable **DNS Round Robin** and **Netmask Ordering** in the DNS server properties. This ensures the DNS server returns the IP address closest to the client's subnet first, optimizing network traffic.

> [!success] Verification Steps
> - Run the command: `Resolve-DnsName -Name "_ldap._tcp.dc._msdcs.company.local" -Type SRV` to verify active directories.
> - Confirm the SRV records return the correct domain controllers.

> [!question] Interview Alert
> - "Explain the difference between a Stub Zone and a Conditional Forwarder."
> - Answer: "A Conditional Forwarder is a static rule that sends all queries for a specific domain to a designated DNS server IP. A Stub Zone is a dynamic zone copy that retrieves and maintains only the SOA, NS, and A records of the target zone, automatically updating if the target zone's authoritative servers change."

---

## Common Mistakes & Fixes
| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Unsynchronized DNS root hints | Outdated hints database | Keep Root Hints updated or configure standard public DNS Forwarders (e.g., `8.8.8.8`). |
| Inactive reverse zones | Creating records without PTR checks | Create Reverse Lookup zones for all subnets to support reverse name lookups. |
| Inactive Scavenging | Forgetting to enable aging | Enable scavenging at both the server level and the zone level to purge stale records. |

---

## Lab Exercise

**Objective:** Install the DNS server role, configure a primary lookup zone, enable secure dynamic updates, configure scavenging, and verify records.
**Time Required:** 30 minutes
**Environment Needed:** A Windows Server 2022 VM (DC01) and a Windows 11 VM.
**Pre-requisites:** Administrative access, static IP configured on the server.

**Steps:**
1. Open PowerShell as Administrator on `DC01`. Install the DNS role:
   ```powershell
   Install-WindowsFeature -Name "DNS" -IncludeManagementTools
   ```
2. Create the Forward Lookup Zone:
   ```powershell
   Add-DnsServerPrimaryZone -Name "lab.local" -ZoneFile "lab.local.dns"
3. Configure Dynamic Updates: Allow secure dynamic updates only.
   ```powershell
   Set-DnsServerPrimaryZone -Name "lab.local" -DynamicUpdate Secure
   ```
4. Configure DNS Scavenging: Set the No-Refresh and Refresh intervals to 7 days:
   ```powershell
   Set-DnsServerZoneAging -Name "lab.local" -Aging $true -NoRefreshInterval 7.00:00:00 -RefreshInterval 7.00:00:00
5. Log into the Windows 11 client VM. Configure its Preferred DNS server to point to the server IP.
6. Register the client record:
   ```cmd
   ipconfig /registerdns
   ```
7. Verification: Run verification command on the server to check the client record.
   ```powershell
   Get-DnsServerResourceRecord -ZoneName "lab.local" -Name "winclient"
   ```

**Success Criteria:** The DNS server registers the client's host record automatically, and the client successfully resolves name queries.
**Common Failures:** Registration failure if the client network connection status is set to Public instead of Domain.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What is a Forward Lookup Zone and a Reverse Lookup Zone?**
A: A Forward Lookup Zone translates hostnames into IP addresses (using A or AAAA records). A Reverse Lookup Zone translates IP addresses back into hostnames (using PTR records in a reverse lookup zone).

**Q: What are DNS Forwarders and where do you configure them?**
A: DNS Forwarders are external DNS server IPs (like public DNS) configured on the local DNS server. When the local server cannot resolve a query internally, it forwards the query to these servers. They are configured in the DNS Manager properties under the Forwarders tab.

### Intermediate (L2 Level)
**Q: Explain how DNS Scavenging works and why it is important in an Active Directory network.**
A: DNS Scavenging automatically purges stale DNS records from the zone database. It uses aging timestamps (No-Refresh and Refresh intervals) to verify record activity. If a record has not been updated after the intervals expire, scavenging deletes it. This is important to prevent IP address mismatches and database bloat.

### Advanced (L3/Senior Level)
**Q: A newly promoted Domain Controller cannot replicate with the forest root DC. Dcdiag logs show name resolution errors. Explain your diagnostic and resolution strategy.**
A:
- **Situation**: A new DC failed to replicate with the forest root DC due to name resolution errors.
- **Task**: I needed to verify and restore the required Active Directory SRV records.
- **Action**: I ran `dcdiag /test:dns` to analyze the DNS health. The report showed missing SRV records for the LDAP service. I opened a command prompt on the new DC and restarted the Netlogon service:
   `net stop netlogon && net start netlogon`.
   I then ran `ipconfig /registerdns` to force record registration. I checked DNS Manager to confirm the records were registered.
- **Result**: The SRV records were registered successfully, resolving the name errors and restoring replication.

### HR / Behavioral
**Q: Tell me about a time you had to deal with a major DNS failure. How did you handle the situation?**
A: A misconfigured DNS change caused all internal users to lose access to the intranet. I logged into the primary DC, analyzed the zone transfer logs, identified the incorrect record, reverted the change, and flushed the DNS cache on client systems, restoring access.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The Windows Server role that manages the name resolution database.
> **Why**: Critical for domain authentication, Active Directory replication, and routing.
> **How**: Configure Active Directory integrated zones, manage secure updates, and configure Forwarders.
> **Command**: `ipconfig /registerdns`
> **Interview Answer Starter**: "In my experience, Windows DNS management requires configuring AD integrated zones to support secure dynamic updates and automatic replication..."

**Key Numbers to Remember:**
- Default Scavenging Aging period: 14 days (7 days No-Refresh + 7 days Refresh)
- DNS Server Service Name: `DNS`
- Replication Event ID: Event ID 4013

**3 Things Interviewer Wants to Hear:**
1. Enforcing Secure Only dynamic updates on AD zones
2. Using DNS Forwarders instead of Root Hints to speed up queries
3. Restarting the Netlogon service to force DC SRV record registration

---

## Related Notes
- [[01-Foundations/02-Networking/DNS|DNS]] — Details client-side DNS protocols.
- [[03-Identity-and-Core-Services/05-Windows-Server/Roles|Roles]] — Details server role deployment settings.
- [[03-Identity-and-Core-Services/06-Active-Directory/Domain-Join|Domain Join]] — Details domain join resolution requirements.

---

## Tags
#desktop-support #windows-server #dns #L2 #interview-topic #lab-complete #daily-use

