---
tags: [desktop-support, security, zero-trust, cryptography, L1]
aliases: [cia-triad, zero-trust-guide, integrity-hashing]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #az-900
---

# CIA Triad and Zero Trust

---

## Concept Overview
- **What it is**: The **CIA Triad** is the foundational security model consisting of Confidentiality (protecting data from unauthorized access), Integrity (ensuring data is accurate and not altered), and Availability (guaranteeing reliable access to data). **Zero Trust** is a modern security framework based on three principles: Verify explicitly, Use least privilege access, and Assume breach.
- **Why it matters for a support engineer**: A support engineer is the first line of defense. Every action—from resetting a password (confidentiality) to restoring a backup (availability) or validating a file hash (integrity)—is mapped directly to the CIA Triad. Zero Trust dictates how support engineers manage remote users and verify client devices.
- **Where you encounter this in real job**: Troubleshooting MFA prompts, explaining file permission blocks, running file hash integrity checks, and checking device compliance status before granting access.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Performs user identity checks before resetting passwords, walks users through MFA setups, and monitors backup availability.
  - **L2**: Configures device encryption (BitLocker), audits file integrity using hashing tools, and manages access permissions based on role definitions.
  - **L3**: Designs tenant-wide Zero Trust architectures, enforces conditional access rules, establishes identity protection baselines, and coordinates disaster recovery availability plans.

---

## Technical Deep Dive

### 1. The CIA Triad in Enterprise Operations
The CIA Triad is a balancing act; increasing security in one pillar can occasionally impact another (e.g., heavy encryption protects confidentiality but can slow down availability).

```
                            [CIA Triad Security]
                                 /    |    \
                                /     |     \
                               v      v      v
              [Confidentiality]   [Integrity]   [Availability]
              - Encryption (AES)  - Hashing     - Backups (RSV)
              - RBAC / Permissions - Signatures  - Redundancy (RA-GRS)
              - MFA Challenges    - Versioning  - Load Balancing
```

- **Confidentiality**: Access is restricted to authorized entities.
  * *Controls*: AES encryption for data at rest (BitLocker), SSL/TLS for data in transit, MFA, and Role-Based Access Control (RBAC).
- **Integrity**: Data must be accurate, complete, and protected from unauthorized modification.
  * *Controls*: Cryptographic hashing (SHA-256), digital signatures, database transaction logs, and version history.
- **Availability**: Authorized users must have uninterrupted access to data when needed.
  * *Controls*: RAID storage arrays, geo-redundant backups, load balancers, UPS power backups, and DDoS protection.

### 2. The Three Principles of Zero Trust
Zero Trust abandons the legacy "Castle-and-Moat" model (where anything inside the corporate network was trusted) in favor of:
1. **Verify Explicitly**: Always authenticate and authorize based on all available data points (identity, location, device health, service, and anomalies).
2. **Use Least Privilege Access**: Limit user access with Just-in-Time (JIT) and Just-Enough-Access (JEA), data protection policies, and role boundaries.
3. **Assume Breach**: Minimize blast radius by segmenting networks, encrypting end-to-end, and using analytics to obtain threat visibility.

### 3. The Six Pillars of Zero Trust Architecture
Zero Trust integrates controls across six critical infrastructure domains:
1. **Identities**: Verify users with strong MFA, risk policies, and single sign-on (SSO).
2. **Devices**: Audit and monitor device compliance status before allowing access (via Intune).
3. **Applications**: Secure APIs, shadow IT apps, and enforce runtime access controls.
4. **Data**: Classify, label, and encrypt documents (Sensitivity Labels / DLP).
5. **Infrastructure**: Monitor for configuration drifts and isolate host servers.
6. **Networks**: Segment network resources using micro-perimeters (VNets, NSGs).

---

## Commands & Syntax

### PowerShell
PowerShell includes built-in cmdlets to verify file integrity and audit security configurations.
```powershell
# Calculate the SHA-256 cryptographic hash of a file to verify its integrity
Get-FileHash -Path "C:\Downloads\update-agent.msi" -Algorithm SHA256

# Compare a calculated file hash against an expected vendor hash
$Hash = (Get-FileHash -Path "C:\Downloads\update-agent.msi" -Algorithm SHA256).Hash
$ExpectedHash = "A567BCE902F1C34A999DB88F45D10E34F5C02D1E2A34F5678BCEF90D1C2A34B2"
if ($Hash -eq $ExpectedHash) {
    Write-Host "Integrity Verified: The file has not been modified." -ForegroundColor Green
} else {
    Write-Warning "INTEGRITY FAILURE: The file has been modified or corrupted!"
}

# Verify BitLocker volume encryption status (Confidentiality control)
Get-BitLockerVolume -MountPoint "C:"
```

### CMD / Run Box
```cmd
:: Check local network availability routing to DNS servers
ping 8.8.8.8

:: Verify certificate path verification for web endpoints
certutil -verify C:\Temp\corporate_root.cer
```

### GUI Path
- **Security Portal**: Go to **security.microsoft.com** -> **Microsoft Secure Score** to audit Zero Trust compliance metrics.
- **Entra Center**: Go to **entra.microsoft.com** -> **Identity** -> **Users** -> **Sign-in logs** to verify explicit authentication signals.

---

## Real-World Scenarios

### Scenario 1: Auditing File Integrity Prior to Software Deployment
**User Complaint:** A system engineer reports: *"I downloaded the new system patch installer 'patch.msi' from a vendor's mirror site. Before I deploy this to all 500 employee desktops, I need to verify that the file was not corrupted during transit or tampered with by a malicious third party."*
**Your First 3 Checks:**
1. Locate the official SHA-256 hash provided by the software vendor on their secure website.
2. Calculate the cryptographic hash of the downloaded installer file.
3. Compare the calculated hash against the vendor's published hash.
**Diagnosis Steps:**
1. Open the vendor's official release page. Copy the published SHA-256 hash string:
   `D2E555C09F200A526E6F22BD02B1DE028E3A8EE65FFD2A87CE65A029FA9E3D8F`
2. Open PowerShell on your computer. Run the file hash cmdlet:
   `Get-FileHash -Path "C:\Temp\patch.msi" -Algorithm SHA256`
   - Output: `Hash : D2E555C09F200A526E6F22BD02B1DE028E3A8EE65FFD2A87CE65A029FA9E3D8F`
3. The calculated hash matches the vendor's hash character-for-character.
4. *Decision*: The file has complete integrity. If the hash differed by even a single character, it would mean the file was altered, corrupted, or infected with malware, requiring deletion.
**Root Cause:** Requirement to verify file integrity (Integrity pillar of CIA).
**Fix:**
1. Document the matching hash values in the deployment change ticket.
2. Proceed with the software deployment using the verified installer.
**Prevention:** Always calculate and document file hashes for all software packages added to the company's software center.
**Ticket Close Note:** "Calculated SHA-256 file hash. Value matched vendor's published signature. Integrity verified. Proceeding with deployment. Closed."

### Scenario 2: Restoring Database Availability After a Ransomware Outage
**User Complaint:** The sales department submits an urgent ticket: *"Our Customer Relations database server is showing an error: 'Connection Refused'. The main application is down. We cannot process any client orders today."*
**Your First 3 Checks:**
1. Verify if the database VM is running and accessible in the cloud.
2. Check the integrity of the database file structures.
3. Locate the latest recovery point in the Azure Backup Vault.
**Diagnosis Steps:**
1. SSH into the database host server.
2. Inspect the database file directories. Note that all database files (e.g., `sales.db`) have been renamed to `sales.db.locked` and are unreadable.
3. The server was compromised via a weak administrative credential, and a script encrypted the active database files, breaking **Integrity** and **Availability**.
4. *Decision*: Do not attempt decryption. To restore **Availability** quickly, we must recover the database from a clean, offline backup snapshot taken prior to the compromise.
**Root Cause:** A ransomware attack compromised database integrity, causing service unavailability.
**Fix:**
1. Format the infected virtual machine OS drive to remove malware.
2. Navigate to the Recovery Services Vault. Select the database volume.
3. Select the latest clean snapshot recovery point (from 8 hours ago).
4. Run the restore task.
5. Once restored, verify the database files are in a clean state, restart the database service, and confirm the sales application connects successfully.
**Prevention:** Enforce Zero Trust principles: restrict access to the DB port using NSGs, require MFA for all admin accounts, and encrypt backups.
**Ticket Close Note:** "Database restored from clean recovery point to resolve ransomware outage. Restored Availability. Enforced strict NSG rules on DB port. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never bypass identity verification checks when a user calls saying: *"I am locked out of my account and need my password reset immediately, I am about to walk into a client meeting."*
> - Threat actors use high-pressure social engineering tactics to bypass verification checks. Resetting a password without verifying identity violates the **Verify Explicitly** principle of Zero Trust, leading to credential theft. Always call back on a verified number or use verified manager channels.

> [!warning] Common Trap
> - Assuming that a secure virtual network (VPN) is sufficient to protect resources under a Zero Trust model.
> - Under Zero Trust, network location does not grant trust. An attacker who compromises a user's home Wi-Fi and connects via VPN can access the internal network. You must still verify device compliance and enforce MFA for every transaction.

> [!tip] Senior Engineer Tip
> - When implementing file integrity checking for automation scripts, compile your code and sign it with a corporate code-signing certificate. Configure execution policies on client computers to `AllSigned`, which prevents unsigned or modified scripts from running, blocking local file tampering.

> [!success] Verification Steps
> - Run `Get-FileHash` on software installers to verify file integrity.
> - Verify that access to sensitive portals displays the Entra ID MFA prompt, confirming that explicit validation is active.

> [!question] Interview Alert
> - "Explain the three main principles of the Zero Trust security model."
> - Answer: "The three principles are: 1. **Verify Explicitly**, which means always authenticate and authorize based on all available signals like identity, location, and device health. 2. **Use Least Privilege Access**, which limits access with JIT and JEA permissions. 3. **Assume Breach**, which limits the blast radius by segmenting networks, encrypting data, and monitoring sessions."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Trusting users because they connect from the office network | Relying on perimeter security | Enforce MFA and check device compliance regardless of network location. |
| Skipping file hash validation for minor patches | Assuming vendor sites are secure | Always verify the SHA-256 hash of any executable downloaded from the web. |
| Neglecting backup verification tests | Assuming backups are healthy | Run quarterly mock restores to verify the Availability of recovery files. |

---

## Lab Exercise

**Objective:** Calculate SHA-256 file hashes in PowerShell, modify a file, recalculate to verify integrity failure, and inspect Zero Trust compliance status.
**Time Required:** 20 minutes
**Environment Needed:** A Windows 11 workstation.
**Pre-requisites:** PowerShell access.

**Steps:**
1. Open PowerShell. Create a folder and a test document:
   ```powershell
   New-Item -Path "C:\Temp" -ItemType Directory -ErrorAction SilentlyContinue
   Set-Content -Path "C:\Temp\contract.txt" -Value "This is the original contract text."
   ```
2. Calculate and store the original hash value:
   ```powershell
   $OriginalHash = (Get-FileHash -Path "C:\Temp\contract.txt" -Algorithm SHA256).Hash
   Write-Host "Original Hash: $OriginalHash"
   ```
3. Simulate unauthorized file tampering (modify a single character):
   ```powershell
   Set-Content -Path "C:\Temp\contract.txt" -Value "This is the original contract text!"
   ```
4. Calculate the new hash value:
   ```powershell
   $NewHash = (Get-FileHash -Path "C:\Temp\contract.txt" -Algorithm SHA256).Hash
   Write-Host "New Hash: $NewHash"
   ```
5. Verification: Compare the values in PowerShell:
   ```powershell
   if ($OriginalHash -ne $NewHash) {
       Write-Host "INTEGRITY BREAK DETECTED: The file was modified!" -ForegroundColor Red
   }
   ```
   - Confirm that the script detects the hash mismatch.

**Success Criteria:** The script successfully calculates both hashes, identifies the mismatch caused by the single character change, and logs the integrity failure.
**Common Failures:** The command fails if the path directory is locked or permissions block file edits.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What does the CIA Triad stand for in IT security?**
A: The CIA Triad stands for Confidentiality (ensuring only authorized users can see data), Integrity (ensuring data is accurate and not altered), and Availability (guaranteeing users can access data when needed).

**Q: Why does the helpdesk call users back on their registered phone number before resetting their password?**
A: This is done to enforce the "Verify Explicitly" security principle. It prevents attackers from calling in, pretending to be an employee, and stealing credentials using social engineering.

### Intermediate (L2 Level)
**Q: How does a cryptographic hash (like SHA-256) help ensure data integrity?**
A: A cryptographic hash algorithm takes a file and runs it through an algorithm to generate a unique, fixed-length string of characters. If even a single bit or character inside the file changes, the resulting hash will change. By comparing the calculated hash of a file to the vendor's published hash, we verify the file has not been altered or corrupted.

**Q: How does the Zero Trust model differ from the legacy Castle-and-Moat network model?**
A: The legacy Castle-and-Moat model assumed that once a user connected to the internal corporate network (inside the moat), they were trusted and had broad access. The Zero Trust model assumes that no user or device is trusted by default, regardless of their network location. Every sign-in and access request must be explicitly verified, authorized, and checked for security compliance.

### Advanced (L3/Senior Level)
**Q: You are designing a Zero Trust authentication flow for remote finance employees accessing a SQL database hosted in Azure. Describe the security controls you would implement.**
A:
- **Situation**: Designing a secure access flow for finance remote workers under Zero Trust.
- **Task**: Configure authentication, device, and network controls to secure access.
- **Action**: First, under the Identity pillar, I enforce modern authentication requiring Microsoft Authenticator MFA with Number Matching. Under the Device pillar, I configure a Conditional Access policy requiring that the user's laptop is enrolled in Intune and marked as "Compliant". Under the Network pillar, I restrict access to the SQL database using an NSG that only allows traffic from our Azure Virtual Desktop subnet. Finally, under the Data pillar, I classify SQL logs using Sensitivity Labels.
- **Result**: The finance data is secured end-to-end, and only verified users on compliant corporate devices can access the database.

### HR / Behavioral
**Q: Tell me about a time you had to handle a situation where business convenience clashed with security compliance. How did you resolve it?**
A: A director wanted to bypass MFA for their team because they claimed it was slowing down their client work. Instead of simply citing the security policy, I sat down with the director. I showed them statistics on recent credential stuffing attacks in our sector. I then helped them configure the Authenticator app, setting up passwordless phone sign-in and number matching, which made logins fast. This maintained our security baseline while addressing their efficiency concerns.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The core security structures: CIA Triad (Confidentiality, Integrity, Availability) and the Zero Trust model (Verify explicitly, Least privilege, Assume breach).
> **Why**: Foundation of all IT security; prevents data leaks, system compromises, and credential theft.
> **How**: Enforce encryption (Confidentiality), use hashing (Integrity), configure backups (Availability), and require MFA + compliance (Zero Trust).
> **Command**: `Get-FileHash` / `Get-BitLockerVolume`
> **Interview Answer Starter**: "To implement secure IT operations, I align all controls with the CIA Triad—using encryption for confidentiality, SHA-256 hashing for integrity, and backups for availability—under a Zero Trust model..."

**Key Numbers to Remember:**
- Zero Trust principles: `3` (Verify, Least Privilege, Assume Breach)
- Zero Trust architecture pillars: `6`
- Default hash algorithm for file checks: SHA-256
- Port required for secure web verification: TCP 443 (HTTPS)

**3 Things Interviewer Wants to Hear:**
- Trust is never granted based on network location under Zero Trust
- Using file hashing (SHA-256) to verify software integrity before installation
- How the CIA Triad pillars balance security and usability

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/Conditional-Access|Conditional Access]] — The primary policy engine enforcing Zero Trust rules.
- [[04-Cloud-and-Security/09-Security/Access-Management|Access Management]] — Focuses on the Least Privilege principle.
- [[04-Cloud-and-Security/09-Security/MFA-and-Identity-Protection|MFA and Identity Protection]] — Details Identity verification controls.

---

## Tags
#desktop-support #security #zero-trust #cryptography #L1 #interview-topic #lab-complete #daily-use

