---
tags: [security, incident-response, playbook, sysadmin]
aliases: [incident-response-playbook]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# Incident-Response-Playbook

> [!abstract] Overview
> Incident Response (IR) is the structured process an organization uses to identify, contain, and mitigate cyber security breaches. This note defines the SANS/NIST 6-step Incident Response Lifecycle and outlines technical containment and recovery playbooks for Ransomware, Compromised Active Directory accounts, Data Exfiltration, and Phishing.

---

## Concept Overview
- **What it is** — An IR playbook is a step-by-step technical script executed by security analysts and sysadmins to identify, contain, and eliminate active security incidents.
- **Why it matters for a support engineer** — During an active breach (e.g., Ransomware spreading on a file share), seconds matter. Delays or incorrect steps (like rebooting a machine instead of isolating it) can lead to complete network-wide encryption.
- **Where you encounter this in real job** — Revoking active sessions for a user whose password was leaked, isolating a compromised server from the network, and checking backup files for indicators of compromise (IoCs).
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Detects basic antivirus alerts, isolates local workstations physically (unplugs ethernet) or via Intune, and files critical incident tickets.
  - **Escalation Trigger:** Escalate to L2 immediately upon detecting ransomware indicators, multi-account lockout alerts, or massive abnormal network traffic.
  - ****L2 Resolution:**** Disables compromised AD/Entra accounts, runs malware cleanup scanners, reviews firewall traffic logs, and restores data from isolated backups.
  - ****L3 Resolution:**** Leads forensic root-cause analysis, conducts memory forensics, traces exfiltration channels, coordinates with legal, and performs system rebuilds.

*Seedha simple mein: Security incident hone par (jaise virus attack ya account hack), hume kya steps lene hain yeh Incident Response Lifecycle tay karta hai. Is note me un checklists ko save kiya gaya hai taaki emergency me jaldi aur sahi action liya ja sake.*

---

## Technical Deep Dive

### 1. The NIST/SANS 6-Phase IR Lifecycle
A standard Incident Response procedure is divided into 6 distinct stages:

```
  +--------------+     +----------------+     +-----------------+
  | 1. Prep      | --> | 2. Identify    | --> | 3. Contain      |
  +--------------+     +----------------+     +-----------------+
                                                       |
  +--------------+     +----------------+     +-----------------+
  | 6. Lessons   | <-- | 5. Recover     | <-- | 4. Eradicate    |
  +--------------+     +----------------+     +-----------------+
```

1. **Preparation**: Developing playbooks, hardening assets, setting up log collections, and training users.
2. **Identification**: Detecting anomalies, analyzing alerts, and determining the scope of the incident.
3. **Containment**: Limiting the damage (Short-term: isolate server; Long-term: modify firewall rules, route traffic).
4. **Eradication**: Removing the cause of the breach (deleting malware, disabling compromised services).
5. **Recovery**: Restoring systems to normal production operations, verifying system health, and scanning for remaining vulnerabilities.
6. **Lessons Learned**: Documenting the incident, identifying security gaps, and updating playbooks.

---

## Step-by-Step Technical Incident Playbooks

### Playbook 1: Active Ransomware Infection

> [!warning] Critical Goal
> Stop encrypting files and isolate the infected host immediately. DO NOT reboot the computer as this can delete forensic evidence stored in RAM.

#### Phase 1: Containment
1. **Isolate the Endpoint**:
   * If on Wi-Fi: Disable wireless adaptor.
   * If on LAN: Unplug the network cable.
   * If virtual machine (Hyper-V / VMware): Disconnect virtual network switch cards (NICs).
2. **Isolate via Intune (if registered)**:
   * Navigate to Intune Portal -> **Devices** -> Select Device -> Click **Isolate**.
3. **Stop network shares propagation**:
   * Stop Server service on file servers to prevent encryption of network drives:
     ```powershell
     Stop-Service LanmanServer -Force
     ```

#### Phase 2: Eradication
1. Execute a full offline scan using Windows Defender:
   ```powershell
   Start-MpWDOScan
   ```
2. Scan running processes to dump memory of the encrypting process.

#### Phase 3: Recovery
1. Re-image/Clean rebuild the OS.
2. Restore file data from isolated, write-protected backups (verify backups are clean of IoCs).

---

### Playbook 2: Compromised Active Directory / Entra ID User Account

> [!warning] Critical Goal
> Prevent unauthorized access and terminate all active login sessions immediately.

#### Phase 1: Containment
1. **Disable the user account**:
   * Local AD:
     ```powershell
     Disable-ADAccount -Identity "alice.smith"
     ```
   * Azure AD / Entra ID:
     ```powershell
     Set-MgUser -UserId "alice.smith@company.com" -AccountEnabled $false
     ```
2. **Revoke Active OAuth Session Tokens**:
   * Run Entra ID PowerShell script to invalidate all refresh tokens:
     ```powershell
     Revoke-MgUserSignInSession -UserId "alice.smith@company.com"
     ```
3. **Block IP Address**: Block the attacker's source IP on the edge firewalls.

#### Phase 2: Eradication
1. Force password reset. Require a strong 16+ character password.
2. Review registered MFA options. Delete any foreign authenticators or phone numbers registered by the attacker:
   ```powershell
   Get-MgUserAuthenticationMethod -UserId "alice.smith@company.com"
   ```

#### Phase 3: Recovery
1. Re-enable user account.
2. Require registration of new MFA credentials upon next login.
3. Monitor sign-in logs for subsequent anomalies.

---

## Common Incident Tracking / Forensic Commands

| Command | Description | Example |
|---------|-------------|---------|
| `netstat -ano` | View all active network connections and matching PID | `netstat -ano` |
| `Get-Process` | List running processes and paths | `Get-Process | select Name, Path` |
| `Disable-ADAccount` | Instantly disable local AD account | `Disable-ADAccount -Identity "username"` |
| `Revoke-MgUserSignInSession` | Force log out Entra ID user | `Revoke-MgUserSignInSession -UserId "[id]"` |

---

## Troubleshooting / IR Escalation Matrix

```
**L1 Resolution:** Identifies antivirus alert, unplugs target workstation Ethernet, and disables local Wi-Fi.
**Escalation Trigger:** Alert contains indicators of ransomware (.locked files, ransom notes) or lateral movement alerts.
**L2 Resolution:** Runs `Revoke-MgUserSignInSession` on Entra ID, disables AD accounts, blocks malicious IPs, and disables file share SMB ports.
```

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **Cannot isolate VM because console interface hangs** | Hypervisor host overloaded or disk inputs blocked by encryption activity. | Hard-stop the VM or disconnect the physical network switch port supporting the host hypervisor. |
| **MFA fatigue attack continues after password change** | Attacker has registered their own MFA device to the account. | Navigate to Entra portal, open user Authentication Methods, and delete unrecognized authenticator app registrations. |

---

## Interview Questions

**Q1: What are the SANS/NIST 6 phases of Incident Response?**
> A: The 6 phases are:
> 1. **Preparation**: Setting up defenses and tools.
> 2. **Identification**: Detecting the event.
> 3. **Containment**: Restricting the damage (e.g., network isolation).
> 4. **Eradication**: Removing the threat vectors.
> 5. **Recovery**: Restoring services safely.
> 6. **Lessons Learned**: Reviewing and improving processes.

**Q2: If you discover a workstation actively encrypting files due to Ransomware, what is your immediate first step?**
> A: The immediate step is to **contain the host** by disconnecting it from the network (unplugging the network cable and disabling Wi-Fi) to prevent it from encrypting files on network file shares. The computer should *not* be powered off or rebooted immediately, as that destroys critical volatile forensic data in RAM.

**Q3: How do you terminate all active web sessions for a compromised Office 365 / Entra ID user?**
> A: You must disable the user account and run the `Revoke-MgUserSignInSession` command (or click "Revoke Sessions" in the Entra ID portal). This invalidates all refresh tokens, forcing the user to log out of all devices (Outlook, Teams, Web) within minutes.

---

## Related Notes
- [[04-Cloud-and-Security/09-Security/MFA-and-Identity-Protection]]
- [[04-Cloud-and-Security/09-Security/Microsoft-Security-Best-Practices]]
- [[04-Cloud-and-Security/09-Security/Microsoft-Sentinel-SIEM]]
