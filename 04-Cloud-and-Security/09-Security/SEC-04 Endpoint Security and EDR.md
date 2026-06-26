---
tags: [security, edr, endpoint, crowdstrike, defender, xdr, mde]
aliases: [Endpoint Security, EDR, XDR, Microsoft Defender]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #none
---

> [!NOTE|color-red]
> 🛡️ **SECURITY**

`#complete` `#advanced` `#none`

# SEC-04: Endpoint Security and EDR

> [!abstract] Overview
> Endpoint Detection and Response (EDR) system endpoints (laptops, servers, mobile devices) ko advanced threats (jaise ransomware, zero-day attacks, fileless malware) se bachata hai. Traditional antivirus sirf signatures par kaam karta tha, lekin EDR behavior aur heuristics par focus karta hai. Ek system admin ya SOC analyst ko EDR alerts samajhna aur unhe remediate karna aana chahiye kyunki ek compromised endpoint poore network ko infect kar sakta hai. Is note mein hum EDR ke core concepts, architecture, troubleshooting, aur real-world scenarios cover karenge jo production environments mein commonly dekhe jaate hain.

---
## 🧠 Concept Overview

- **What it is** — EDR is an advanced security solution that continuously monitors endpoints, detects suspicious behaviors, and provides automated or manual response capabilities to isolate and remediate threats.
- **Why it matters** — Traditional AV (Antivirus) fails against modern attacks. *Aaj kal ke hackers fileless malware use karte hain jo memory mein run hota hai, jise EDR hi pakad sakta hai.*
- **Where you see this** — Deploying CrowdStrike Falcon, Microsoft Defender for Endpoint (MDE), or SentinelOne on enterprise servers and user workstations. *Jab koi user phishing link pe click karke malicious file download karta hai, EDR use automatically block aur isolate kar deta hai.*

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Monitor alerts, check device status, run basic scans, inform users. *Alerts dekhna, user se baat karna, aur basic quarantine action lena.* |
| **L2** | Deep dive into alerts, isolate machines, investigate process trees, unblock false positives. *Device ko network se isolate karna, malware ka source dhoondhna aur rules adjust karna.* |
| **L3** | Create custom detection rules, tune policies, architecture deployment, integrate with SIEM/SOAR. *Enterprise-wide policies design karna, deployment strategies banana, aur advanced threat hunting karna.* |

> [!tip] Seedha Simple Mein
> *Traditional Antivirus ek security guard ki tarah hai jo sirf known criminals ki photo (signature) dekh ke rokta hai. EDR CCTV camera + SWAT team ki tarah hai, jo har activity record karta hai aur agar koi ajeeb harkat (behavior) kare toh turant action leta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **EDR (Endpoint Detection and Response)** is like **a highly advanced immune system with a central nervous system** because...
>
> - **Antivirus (White Blood Cells)** sirf known infections ko attack karta hai.
> - **EDR (Nervous System)** har activity ko brain (Central Console) tak bhejta hai. Agar koi safe dikhne wala process achanak se system files delete karne lage (jaise ransomware), toh EDR turant us body part (endpoint) ko paralyze (isolate) kar deta hai taaki infection na phaile.
> - **Threat Hunting (Doctors)** un patterns ko analyze karte hain jo shayad abhi tak detect nahi hue.

---
## 🔬 Technical Deep Dive

### 1. Traditional AV vs EDR vs XDR

> [!info] Key Concept
> Understanding the evolution of endpoint protection is critical for architecture and troubleshooting.

- **Legacy AV (EPP - Endpoint Protection Platform):** Uses hash signatures. If the hash matches a known virus, it blocks it. *Bahut basic hai, naye threats nahi pakad pata.*
- **EDR (Endpoint Detection & Response):** Monitors process execution, registry changes, network connections. It records the "flight data" of the endpoint. *Har choti activity record hoti hai cloud console mein, taaki investigation ki ja sake.*
- **XDR (Extended Detection and Response):** Combines EDR with network, email, cloud identity logs, and more to give a complete picture. *Isme endpoint ke alawa baaki saari telemetry bhi jud jaati hai.*

### 2. How EDR Works (The Sensor/Agent)

EDR software runs as a lightweight agent (sensor) at the kernel level (Ring 0). 
*Kyunki agent kernel level par chalta hai, malware ise easily bypass ya disable nahi kar sakta.*

- **Telemetry Collection:** It sends process executions (Event ID 4688), network connections, and DNS requests to the cloud.
- **Behavioral Analysis:** Uses Machine Learning to identify patterns (e.g., PowerShell downloading an executable and running it).
- **Automated Response:** Kills malicious processes, quarantines files, and isolates the host from the network automatically based on policy.

### 3. Key Features of an Enterprise EDR

- **Network Isolation:** Temporarily cuts off an endpoint from the corporate network while preserving connection to the EDR cloud portal.
- **Live Response / Remote Shell:** Allows admins to connect to an endpoint via a secure remote shell to run scripts, kill processes, or fetch logs without RDP.
- **Threat Hunting:** Using KQL (Kusto Query Language) or similar syntax to proactively search endpoint logs for indicators of compromise (IoCs).

> [!danger] Common Mistake
> Installing two EDR/AV solutions on the same endpoint without proper configuration. *Ek hi machine pe Defender aur CrowdStrike dono active rakhna system crash (BSOD) ya extreme performance lag ka kaaran ban sakta hai. Ek ko "Passive Mode" mein rakhna padta hai.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows 10/11 VM or Windows Server.
> - Microsoft Defender for Endpoint (MDE) or any EDR trial (like SentinelOne).
> - PowerShell with Administrator privileges.

### Step 1: Onboarding a Windows Device to Defender for Endpoint

To onboard a device, you need the onboarding script from the MDE Security Center.

```powershell
# Step 1: Run the onboarding script provided by MDE Portal
# Yeh script device ko cloud console se connect karti hai
.\WindowsDefenderATPLocalOnboardingScript.cmd
```

> [!success] Expected Output
> ```
> Starting Windows Defender ATP machine onboarding.
> ...
> Successfully onboarded machine to Microsoft Defender for Endpoint.
> ```

### Step 2: Testing EDR Detection (Simulating an Attack)

We will use a benign test file to trigger an alert without actually infecting the system.

```powershell
# Step 2: Run a benign simulated attack (DoIt.ps1) or create a fake malicious process
# Yeh command sirf EDR ko trigger karne ke liye hai (Safe test)
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "Write-Host 'Simulating malicious behavior...'; Start-Sleep -s 2"
```

*Real-world lab mein aap EICAR test file download karke check kar sakte hain ki EDR usko kitni jaldi block karta hai.*

### Step 3: Verifying Agent Status Locally

```cmd
# Step 3: Check if the Defender ATP service is running
sc query sense
```

> [!success] Expected Output
> ```
> SERVICE_NAME: sense
>         TYPE               : 10  WIN32_OWN_PROCESS
>         STATE              : 4  RUNNING
> ```

### Step 4: Using Live Response (Simulated)

Most EDRs have a "Live Response" feature.
- Go to the EDR Portal.
- Select the Machine.
- Click **Initiate Live Response Session**.
- *Yeh ek remote terminal khol dega jahan aap bina RDP kiye command run kar sakte hain.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `sc query sense` | Checks Defender for Endpoint agent status (Sense service). | `sc query sense` |
| `sc query windefend` | Checks core Windows Defender AV service status. | `sc query windefend` |
| `Get-MpComputerStatus` | Shows detailed Defender settings, engine versions, and definition dates. | `Get-MpComputerStatus` |
| `Update-MpSignature` | Forces an immediate update of AV definitions. | `Update-MpSignature` |
| `Get-MpPreference` | Displays all exclusion paths, policies, and AV configurations. | `Get-MpPreference` |
| `Set-MpPreference -DisableRealtimeMonitoring $true` | Disables real-time protection (usually blocked by Tamper Protection). | `Set-MpPreference ...` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Device not showing in EDR Console** | Onboarding script failed, device offline, or blocked by firewall. | Run `Test-NetConnection` to EDR cloud URLs (e.g., `*.dm.microsoft.com`). Re-run onboarding script. Check proxy. |
| **High CPU usage by EDR Agent** | EDR is scanning a huge file, looping on archives, or conflicting with another AV. | Check Task Manager. Add exclusion for heavy I/O directories (like database/log folders) if safe. Ensure no 3rd party AV conflict. |
| **Legitimate app blocked (False Positive)** | App behaves suspiciously (e.g., executing scripts from Temp folder, unsigned binaries). | *Console mein jaakar us file hash, path, ya signer certificate ko "Allow/Exclude" list mein daalna padega.* |
| **Cannot uninstall EDR Agent / Service cannot be stopped** | Tamper Protection is enabled. | *Admin console se pehle Tamper Protection disable karein ya uninstall password generate karein, tabhi agent uninstall hoga.* |
| **Logs not syncing to SIEM** | API key expired or integration broken. | Regenerate API keys in EDR portal and update the SIEM connector. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Complains that a Custom In-house App is Blocked

> [!example] Ticket
> "Hi IT, I am trying to run our HR application (hrapp.exe), but CrowdStrike/Defender keeps deleting it automatically. It says 'Malicious Behavior Detected'. Please fix ASAP as payroll is blocked."

**L1 Response:** Check the EDR console for the alert. *Pehle check karein ki kaunsa rule trigger hua hai. File ka hash VirusTotal pe verify karein.*
**Escalation Trigger:** The file is clean on VirusTotal and developed internally, but keeps getting blocked. *Jab sure ho jayein ki yeh false positive hai, L2/SecOps ko pass karein.*
**L2 Resolution:** Add the file hash or certificate to the EDR Global Exclusions / Allow Indicator list. Restore the file on the user's machine from quarantine via the console. Inform the developer to sign their application.

### 🎫 Scenario 2: High Severity Ransomware Alert on a Domain Controller

> [!example] Ticket
> "[SEV 1] Automated Alert: Suspicious encryption activity detected on Server DC-01. Process: vssadmin.exe (Deleting Volume Shadow Copies)."

**L1 Response:** Immediately verify the alert. *Dekhein ki kya sach mein encryption chalu hai aur kon sa user account use ho raha hai.*
**Escalation Trigger:** High severity alerts on Tier 0 assets (Domain Controllers) are immediate L2/L3 or Incident Response escalations.
**L2 Resolution:** Instantly trigger "Network Isolation" from the EDR console. *Server ko network se kaat dein taaki ransomware doosre servers pe na phaile.* Investigate the process tree, dump memory if needed, rotate compromised credentials, and initiate disaster recovery protocols.

### 🎫 Scenario 3: Device Shows as "Inactive" in EDR Portal

> [!example] Ticket
> "Security compliance report shows laptop LT-SALES-55 is inactive in Microsoft Defender for Endpoint for the last 15 days."

**L1 Response:** Contact the user to check if the laptop is turned on and connected to the internet. *Shayad user chhutti par ho ya laptop band pada ho.*
**Escalation Trigger:** The laptop is online, user is working, but it's still not reporting to the console.
**L2 Resolution:** Remote into the laptop. Check if the `sense` service is running. Check Windows Event Logs (Microsoft-Windows-Sense/Operational). Verify proxy settings or firewall rules that might be dropping telemetry traffic to Microsoft endpoints.

### 🎫 Scenario 4: Malware Outbreak Detected Across Multiple Hosts

> [!example] Ticket
> "Multiple alerts originating from 10+ different endpoints about 'Suspicious PowerShell Download' within a 5-minute window."

**L1 Response:** Notice the trend and escalate immediately as a potential widespread outbreak or active intrusion. *Agar ek saath bohot machines pe same alert aaye, toh yeh serious breach ho sakta hai.*
**Escalation Trigger:** Multi-host coordinated attacks require immediate SOC lead / L3 intervention.
**L3 Resolution:** Use EDR Advanced Hunting to query the entire environment for the malicious hash or IP. Isolate all affected machines simultaneously. Block the C2 (Command & Control) IP address at the perimeter firewall.

---
## 🎤 Interview Questions

> [!question] Q1: What is the main difference between traditional Antivirus and EDR?
> **Answer:** Traditional AV relies primarily on signature-based detection (known hashes of malware). EDR focuses on behavioral analysis, continuous telemetry collection, and anomaly detection. This allows EDR to catch zero-day attacks and fileless malware by monitoring process behavior rather than just looking at file signatures.

==**Exam Tip:** Always emphasize "Behavioral Analysis", "Telemetry", and "Response Actions" when defining EDR.==

> [!question] Q2: If an EDR blocks a legitimate business application, how do you fix it permanently?
> **Answer:** First, investigate the alert to confirm it's a false positive (check VirusTotal, speak to developers). Then, gather the file hash, path, or signing certificate. Finally, add an exclusion (Allow Indicator) in the EDR console for that specific hash or certificate to prevent future blocking.

> [!question] Q3: What does "Isolating a host" mean in an EDR context, and why is it useful?
> **Answer:** Network isolation cuts off all inbound and outbound network communication from the infected endpoint at the OS/Hypervisor level, EXCEPT for the connection to the EDR cloud console. This prevents malware from spreading laterally (worming) or communicating with its Command & Control server, while allowing admins to investigate remotely.

> [!question] Q4: A user clicked a phishing link and malware executed. How would you use EDR to investigate the root cause?
> **Answer:** I would review the Process Tree (Incident Graph) in the EDR console. I'd trace the execution backward from the payload. For example: `outlook.exe` -> `chrome.exe` (downloaded payload) -> `powershell.exe` (execution). I would look for the exact commands run, files dropped, and external IP addresses communicated with to understand the blast radius.

> [!question] Q5: Why is "Tamper Protection" important in EDR tools?
> **Answer:** Advanced malware (like ransomware) tries to disable, pause, or uninstall security software before encrypting files. Tamper Protection locks down the EDR agent at the kernel level, preventing local administrators, scripts, or malware from stopping the service, modifying critical registry keys, or uninstalling it without authorized cloud approval.

---
## 🔗 Related Notes

- [[SEC-01 Introduction to Cyber Security|SEC-01 Introduction to Cyber Security]] — Basic security concepts.
- [[SEC-02 Firewalls and Network Security|SEC-02 Firewalls and Network Security]] — Network perimeter defense and blocking IPs.
- [[WIN-05 Active Directory Fundamentals|WIN-05 Active Directory Fundamentals]] — Securing the central identity system against lateral movement.
- [[SEC-05 SIEM and SOC Operations|SEC-05 SIEM and SOC Operations]] — Where EDR alerts are sent for centralized monitoring and correlation.
