---
tags: [security, endpoint-security, defender, mde, md-102]
aliases: [endpoint-security-defender]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# Endpoint-Security-Defender

> [!abstract] Overview
> Microsoft Defender for Endpoint (MDE) is an enterprise endpoint security platform designed to help networks prevent, detect, investigate, and respond to advanced threats. MDE provides endpoint protection, Endpoint Detection and Response (EDR), threat and vulnerability management, and integrates natively with Microsoft Intune for configuration.

---

## Concept Overview
- **What it is** — MDE is a cloud-based security service that uses behavioral sensors embedded in Windows 10/11 and cloud-based machine learning to analyze endpoint telemetry.
- **Why it matters for a support engineer** — Supporting modern workplaces means protecting devices anywhere. Support engineers configure Defender policies, verify client onboarding, monitor security alerts, and isolate compromised systems.
- **Where you encounter this in real job** — Troubleshooting onboarding failures on a client laptop, configuring Attack Surface Reduction (ASR) rules to prevent macro exploits, and running Live Response sessions to delete malicious files.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Checks device risk score in Defender console, triggers quick malware scans, and reviews basic software vulnerability alerts.
  - **Escalation Trigger:** Escalate to L2 if a device risk score changes to High, an active malware detection cannot be cleaned automatically, or ASR rules block business applications.
  - ****L2 Resolution:**** Onboards client machines via Intune policies, configures ASR rules in Audit/Block mode, and executes Live Response forensic commands.
  - ****L3 Resolution:**** Integrates MDE with Sentinel, configures advanced device groups and RBAC permissions, and defines automated investigation and response (AIR) scopes.

*Seedha simple mein: Microsoft Defender for Endpoint (MDE) ek enterprise-grade security system hai jo normal windows defender se advance hai. Yeh cloud telemetry ka use karke process behaviors ko check karta hai aur Intune ke sath mil kar client laptops ko secure aur compliance-ready banata hai.*

---

## Technical Deep Dive

### 1. MDE Core Capabilities
* **Threat & Vulnerability Management (TVM)**: Real-time discovery of software vulnerabilities and misconfigurations (e.g., outdated Chrome browser on client machines).
* **Attack Surface Reduction (ASR)**: Set of rules to block common attack vectors, such as executing obfuscated scripts, blocking office applications from launching child processes.
* **Next-Generation Protection**: Traditional antivirus combined with cloud-based machine learning to block new malware variants.
* **Endpoint Detection and Response (EDR)**: Records process and network telemetry for analysis, threat hunting, and forensics.
* **Auto-Remediation (AIR)**: Automatically investigates alerts, runs threat diagnostics, and cleans up files.

### 2. Device Onboarding Methods
Windows 10/11 has built-in sensors but needs to be pointed to the tenant. Onboarding files (script/package) are deployed via:
* **Microsoft Intune**: Automated enrollment via Configuration Profiles (Recommended).
* **Group Policy (GPO)**: For local Active Directory-joined on-premises systems.
* **Local Script**: For testing small groups of up to 10 machines.

---

## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - A Microsoft 365 E5 license or trial.
> - Access to Microsoft Defender Portal (`security.microsoft.com`).
> - Access to Microsoft Intune admin center (`intune.microsoft.com`).
> - A Windows 11 client machine (`Client-01`) registered in Intune.

### Step 1: Onboard Windows 11 VM to Defender for Endpoint via Intune
We will configure automated onboarding for our client computers.

1. Open **Microsoft Intune admin center** -> **Endpoint security** -> **Device onboarding** under Microsoft Defender for Endpoint.
2. Ensure **Connect Windows devices to Microsoft Defender for Endpoint** is toggled to **On**.
3. Create a Configuration Profile:
   * Go to **Devices** -> **Configuration profiles** -> **Create profile**.
   * Platform: **Windows 10 and later**.
   * Profile type: **Templates** -> Select **Microsoft Defender for Endpoint (desktop devices)**.
4. Name: `MDE-Onboarding-Policy`.
5. Configuration settings: Set **Sample sharing** to **Enable** and **Telemetry reporting frequency** to **Normal**.
6. Assign to group: `All Devices` -> Save.
**Expected Output:** Within 15 minutes, `Client-01` synchronizes. Checking **Microsoft Defender Portal** -> **Device Inventory** lists `Client-01` with status `Onboarded`.

---

### Step 2: Configure Attack Surface Reduction (ASR) Rules
We will configure rules to block Excel and Word files from spawning command prompt or executing scripts.

1. In Intune, go to **Endpoint security** -> **Attack surface reduction** -> Click **Create Policy**.
2. Platform: **Windows 10 and later** -> Profile: **Attack Surface Reduction rules**.
3. Name: `ASR-Block-Office-Child-Processes`.
4. Configure rule settings:
   * Set **Block Office applications from creating child processes** to **Block**.
   * Set **Block execution of potentially obfuscated scripts** to **Block**.
5. Assign to the devices group -> Save.
**Expected Output:** Intune pushes the policy. If a user attempts to run a malicious macro that opens CMD, Windows blocks the action and displays a Toast notification.

---

### Step 3: Initiate a Live Response Session
We will connect to our client computer remote shell to retrieve running task diagnostics.

1. In **Microsoft Defender Portal** -> Go to **Device Inventory** -> Select `Client-01`.
2. Click **Initiate Live Response Session** from the top right actions.
3. Wait for connection to load the CLI prompt.
4. Run diagnostics commands in the secure terminal:
```bash
# List running processes
processes

# Retrieve file details
fileinfo "C:\Windows\System32\cmd.exe"
```
**Expected Output:** Secured cloud shell displays processes and file information from `Client-01`.

---

## Common Live Response Commands

| Command | Description | Example |
|---------|-------------|---------|
| `processes` | List active processes on target | `processes` |
| `getfile` | Download a file from target machine for analysis | `getfile "C:\Users\User\Downloads\suspicious.exe"` |
| `putfile` | Upload a helper script to target | `putfile "C:\scripts\remediate.ps1"` |
| `remediate` | Remove malware or block threat registry keys | `remediate filepath "C:\temp\malware.exe"` |

---

## Troubleshooting / Integration Matrix

```
**L1 Resolution:** Monitors compliance dashboards, checks if Defender sensor is running using `sc query sense`.
**Escalation Trigger:** Device status remains "Can be onboarded" or "Out of compliance" after Intune policy deployment.
**L2 Resolution:** Runs manual onboarding scripts, checks Event Viewer `Microsoft-Windows-Sense` logs, and runs Live Response tools.
```

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **Device not showing in Defender Portal** | Sense service failed to start or proxy blocking endpoints. | Run `sc query sense` to ensure service is active. Verify outbound connection using MDE Client Analyzer tool. |
| **ASR rules blocking legacy business applications** | Rules are too strict for custom developer environments. | Toggle specific rule to **Audit** mode instead of Block. Analyze logs to define exclusion file paths. |

---

## Interview Questions

**Q1: What is Microsoft Defender for Endpoint (MDE) and how is it different from local Windows Defender Antivirus?**
> A: Local **Windows Defender Antivirus** is a traditional next-generation endpoint protection tool focused on file-scanning and malware blocking. **MDE** is a comprehensive enterprise cloud platform that adds advanced Endpoint Detection and Response (EDR) capability, Threat & Vulnerability Management, and Attack Surface Reduction, enabling enterprise-wide security tracking, remote live shell forensics, and automated threat remediation.

**Q2: How do you onboard a corporate Windows 11 machine to MDE using Microsoft Intune?**
> A: You first establish the connection link in Intune Endpoint Security. Next, create a configuration profile using the "Microsoft Defender for Endpoint" template, define configuration telemetry settings, and assign it to the target device group. The Intune agent pushes the onboarding registry keys to the target device, which activates the built-in Windows `Sense` telemetry service.

**Q3: What are Attack Surface Reduction (ASR) rules?**
> A: ASR rules are system-hardening configurations that target specific behaviors commonly exploited by malware. Examples include preventing Office applications from spawning child processes, blocking executables from email attachments, and preventing Adobe Reader from launching child applications.

**Q4: What is a Live Response session in MDE?**
> A: Live Response is a secure forensic command shell utility that connects analysts directly to an onboarded client endpoint from the Defender cloud console. It allows administrators to run remote diagnostic commands, download suspicious files, upload remediation tools, and query active processes with minimal user interruption.

---

## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction]]
- [[04-Cloud-and-Security/07-Microsoft-365/INT-07 Endpoint Security in Intune]]
- [[04-Cloud-and-Security/09-Security/Incident-Response-Playbook]]
