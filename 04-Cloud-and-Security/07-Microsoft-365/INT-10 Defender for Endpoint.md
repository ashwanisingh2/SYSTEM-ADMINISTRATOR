---
tags: [microsoft-365, intune, defender, endpoint-security, advanced]
aliases: [Defender for Endpoint, MDE, ATP]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#advanced` `#md-102`

# INT-10: Defender for Endpoint

> [!abstract] Overview
> Microsoft Defender for Endpoint (MDE) is an enterprise endpoint security platform designed to help enterprise networks prevent, detect, investigate, and respond to advanced threats. *Yeh note MDE ke architecture, onboarding process, aur daily administrative tasks ko cover karta hai, taaki ek support engineer endpoints ko secure aur compliant rakh sake.* MDE provides a unified view of alerts across the entire organization, bringing together threat signals from different endpoints into a centralized dashboard.

---
## 🧠 Concept Overview

- **What it is** — An endpoint detection and response (EDR) solution that integrates deeply with Microsoft 365 Defender. It uses built-in behavioral sensors in Windows 10/11 and can also be deployed to macOS, Linux, Android, and iOS devices.
- **Why it matters** — Traditional signature-based antivirus is no longer enough to stop modern cyber attacks. EDR provides behavioral monitoring, real-time threat intelligence, and automated remediation capabilities. *Company ka data aur devices ko advanced ransomware aur zero-day attacks se bachane ke liye yeh zaroori hai.*
- **Where you see this** — In security operation centers (SOC), incident response teams, and device management (Intune) when checking device compliance or investigating a breached laptop.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — device onboarding status check karna, alert queues dekhna, aur false positives filter karna. Review automated investigation results. |
| **L2** | Configure, fix, escalate — onboarding issues fix karna, custom indicators (IoC) banana, Intune ASR policies manage karna, aur automated investigations ko review karna. |
| **L3** | Architecture, design, enterprise-level — advanced hunting queries (KQL) likhna, integration with SIEM (Microsoft Sentinel), threat modeling, and managing custom detection rules. |

> [!tip] Seedha Simple Mein
> *MDE ek smart security guard ki tarah hai jo na sirf building ke gate par khada rehta hai (Antivirus), balki har ek room mein lagatar cameras (EDR sensors) ke through monitor karta hai ki koi suspicious activity toh nahi ho rahi, aur kuch galat dikhne par automatically alarm bajata hai aur darwaze lock kar deta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Smart Enterprise Security System** is like **Microsoft Defender for Endpoint** because...
>
> - Just like a basic lock stops casual thieves, **Next-Generation Protection (Antivirus)** stops known malware using signatures.
> - Just like motion sensors and security cameras track movement inside the building, **Endpoint Detection and Response (EDR)** tracks what processes, scripts, and files are doing on the computer.
> - Just like an automated alarm system that calls the police and locks doors when an intruder is detected, **Automated Investigation and Remediation (AIR)** automatically isolates the device and removes the threat.
> - Just like an auditor checking if doors are weak or windows are broken, **Threat & Vulnerability Management (TVM)** scans for outdated software and weak configurations.

---
## 🔬 Technical Deep Dive

### 1. Key Components of MDE Architecture

> [!info] Key Concept
> MDE is composed of multiple pillars of security that work together seamlessly, rather than just being a single antivirus program.

1. **Threat & Vulnerability Management (TVM):** Discovers, prioritizes, and remediates endpoint vulnerabilities and misconfigurations in real time. It tells you which devices have outdated Chrome or unpatched Adobe Reader.
2. **Attack Surface Reduction (ASR):** Minimizes the places where an organization is vulnerable to cyber threats. Examples include blocking Office macros from running executable content, or preventing USB drives from auto-running scripts.
3. **Next-Generation Protection:** Traditional and cloud-delivered antivirus (Microsoft Defender Antivirus) that blocks threats at the front door.
4. **Endpoint Detection and Response (EDR):** Detects advanced threats that bypass the first two layers, providing deep visibility into endpoint activities. It records a timeline of every process launch, file modification, and network connection.
5. **Automated Investigation and Remediation (AIR):** Uses AI to automatically investigate alerts, group them into incidents, and resolve complex threats in minutes without human intervention.
6. **Microsoft Secure Score for Devices:** Provides visibility into the security posture of the organization's network by assessing OS configurations and application states.

### 2. Onboarding Methods and Sensor Deployment

Endpoints must be "onboarded" to MDE so their sensor data telemetry can be sent to the Microsoft 365 Defender portal.

- **Local Script:** Good for up to 10 devices (mostly used for POC or testing).
- **Group Policy (GPO):** For traditional domain-joined Windows machines in an on-premises Active Directory environment.
- **Microsoft Endpoint Configuration Manager (MECM / SCCM):** For large on-premises environments where Intune is not fully deployed.
- **Mobile Device Management (Intune):** The most common modern approach for Windows 10/11, macOS, and mobile devices, leveraging cloud-native deployment.

> [!danger] Common Mistake
> *Kai baar engineers Intune se policy push kar dete hain par device already ek third-party Antivirus (like Symantec or McAfee) run kar raha hota hai bina "Passive Mode" enable kiye. Iski wajah se performance issues, system freezing, aur file scanning conflicts aate hain.* Always ensure the transition is planned correctly.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Microsoft 365 E5 or Windows E5 / MDE Plan 1 or 2 License.
> - Access to Microsoft 365 Defender Portal (`security.microsoft.com`) with Security Administrator roles.
> - Access to Intune Admin Center (`endpoint.microsoft.com`) with Intune Administrator roles.
> - A test Windows 10/11 Virtual Machine enrolled in Intune.

### Step 1: Establish Intune to MDE Connection

1. Go to **endpoint.microsoft.com** -> Tenant administration -> Connectors and tokens -> Microsoft Defender for Endpoint.
2. Click **Open the Microsoft Defender Security Center**.
3. In MDE portal, go to Settings -> Endpoints -> Advanced features.
4. Turn on **Microsoft Intune connection**.
5. Back in Intune, toggle on **Allow Microsoft Defender for Endpoint to enforce Endpoint Security Configurations** and **Connect Windows devices**.

### Step 2: Create Intune Onboarding Policy

1. Go to **endpoint.microsoft.com** -> Endpoint security -> Endpoint detection and response.
2. Click **Create Policy** (Platform: Windows 10 and later, Profile: Endpoint detection and response).
3. Name the policy `MDE-Auto-Onboarding-Windows`.
4. In Configuration settings, set **Microsoft Defender for Endpoint client configuration package type** to `Auto from connector`. This tells Intune to automatically pull the onboarding blob from the MDE backend.
5. Assign to a test device group containing your VM.

### Step 3: Run a Detection Test on Endpoint

After onboarding completes, verify the sensor is working and reporting to the cloud by running a safe test file that simulates malicious behavior.

```cmd
# Run this on the test machine via Command Prompt to simulate a suspicious PowerShell alert
powershell.exe -NoExit -ExecutionPolicy Bypass -WindowStyle Hidden $ErrorActionPreference = 'silentlycontinue';(New-Object System.Net.WebClient).DownloadFile('http://127.0.0.1/1.exe', 'C:\\test-WDATP-test\\invoice.exe');Start-Process 'C:\\test-WDATP-test\\invoice.exe'
```

> [!success] Expected Output
> *Command Prompt band ho jayega, aur kareeb 5-10 minute mein MDE portal par ek high-severity alert generate hoga: "Suspicious PowerShell commandline". Yeh confirm karta hai ki EDR sensor properly active hai aur MDE portal ko telemetry bhej raha hai.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `sc query sense` | Checks the status of the MDE advanced sensor service. Service should be RUNNING. | `sc query sense` |
| `Get-MpComputerStatus` | Shows detailed info about Defender AV status, signature versions, and engine state. | `Get-MpComputerStatus \| Select-Object AMServiceEnabled, AntispywareEnabled` |
| `Get-MpPreference` | Displays all Defender AV preferences, active policies, and exclusion lists. | `Get-MpPreference \| Select-Object ExclusionPath, ExclusionProcess` |
| `Update-MpSignature` | Forces a manual update of Defender AV definitions from Microsoft Update. | `Update-MpSignature` |
| `MpCmdRun.exe -SignatureUpdate` | Command-line tool to update signatures (very useful in remote scripts or RMM). | `"C:\ProgramData\Microsoft\Windows Defender\Platform\<Version>\MpCmdRun.exe" -SignatureUpdate` |
| `MpCmdRun.exe -Scan -ScanType 1` | Runs a quick scan from the command line. | `MpCmdRun.exe -Scan -ScanType 1` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Device not showing in Defender Portal | Sensor service (`sense`) is not running or blocked by corporate firewall/proxy. | Check `sc query sense`. Verify URLs in proxy/firewall (*.blob.core.windows.net, *.ods.opinsights.azure.com are reachable). |
| Defender AV in Passive Mode unexpectedly | Third-party AV is installed but not properly integrated or registered in Windows Security Center. | Verify third-party AV status. If migrating to MDE, uninstall 3rd party AV completely and restart the machine. |
| High CPU usage by `MsMpEng.exe` | Defender is continuously scanning its own files, large archives, or conflicting with another heavy I/O app like SQL. | Add folder/process exclusions via Intune Policy for known safe, heavy applications. Check Task Manager. |
| Alerts not generating for test file | Device isn't fully synced or there is an internet connectivity issue to the MDE backend servers. | Force sync the device from Intune. Check event viewer logs under `Applications and Services Logs > Microsoft > Windows > Sense > Operational`. |
| "Action required" on TVM dashboard | Software is outdated or missing critical security patches. | Work with desktop team to push the required software update via Intune Apps or MECM. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Device Onboarding Failure

> [!example] Ticket
> "User PC 'LPT-FIN-001' is showing as 'Can be onboarded' in the TVM dashboard but is not reporting any sensor data. Intune shows the onboarding policy is successfully deployed."

**L1 Response:** Check basic connectivity. *Check karo ki device par internet chal raha hai aur user logged in hai.* Run `sc query sense` via remote shell or Intune to see if the EDR service is actually running.
**Escalation Trigger:** Service is running, but the portal still doesn't show any telemetry data after 24 hours of waiting.
**L2 Resolution:** Review the `Microsoft-Windows-Sense/Operational` event logs using Intune Log collection. The logs reveal that the corporate proxy is blocking the MDE telemetry URLs. Work with the network/firewall team to bypass proxy authentication for all MDE required URLs.

### 🎫 Scenario 2: False Positive Alert (Developer Machine)

> [!example] Ticket
> "I am a senior developer in the engineering team. My custom compiled executable `data_processor_v2.exe` keeps getting blocked and deleted by Microsoft Defender every time I try to run it."

**L1 Response:** Find the alert in the Defender Portal. *User ka device name aur file ka naam search karo portal ke 'Incidents & alerts' tab mein.* Identify the reason it was blocked (e.g., Heuristic or Machine Learning detection).
**Escalation Trigger:** The file is confirmed safe by the dev team and needs a global exclusion created or file submitted to Microsoft for analysis.
**L2 Resolution:** Go to **Settings -> Endpoints -> Indicators** in the Security Portal. Add the exact SHA-256 file hash of `data_processor_v2.exe` and set the action to **Allow**. Instruct the developer to wait 1-2 hours for the indicator policy to sync to their endpoint.

### 🎫 Scenario 3: Automated Investigation Isolation (Ransomware Alert)

> [!example] Ticket
> "Urgent: My laptop is connected to WiFi but I cannot access any company resources, websites, SharePoint, or Teams. I can only see the Windows desktop and nothing loads."

**L1 Response:** Search for the device name in the MDE portal. *Portal mein dekho ki device status 'Isolated' toh nahi hai.*
**Escalation Trigger:** Device is isolated due to a High severity ransomware alert or lateral movement detection.
**L2 Resolution:** Review the Automated Investigation details. The logs show the user accidentally clicked a malicious link in a phishing email. Wait for AIR to fully remediate the threat. Once remediated, perform a full Antivirus scan. When the machine is confirmed 100% clean, go to the Device page -> **Actions -> Release from isolation**.

---
## 🎤 Interview Questions

> [!question] Q1: What is the fundamental difference between Microsoft Defender Antivirus and Microsoft Defender for Endpoint (MDE)?
> **Answer:** Defender Antivirus is the Next-Generation Protection (NGP) component that handles signature-based file scanning, real-time protection, and basic malware blocking. MDE is the overarching Enterprise EDR platform that collects behavioral telemetry, provides Threat and Vulnerability Management (TVM), Automated Investigation (AIR), and central management via the Microsoft 365 Defender Portal.

> [!question] Q2: What is "Passive Mode" in Microsoft Defender Antivirus and when exactly is it used?
> **Answer:** Passive mode means Defender AV is installed and actively receives definition updates, but it does not take remediation action against threats (it doesn't block or delete files). It is primarily used when an organization is running a third-party Antivirus (like CrowdStrike or SentinelOne) as their primary protection, but still wants to onboard the device to MDE for EDR capabilities and Threat & Vulnerability Management.

> [!question] Q3: How do you exclude a specific application folder from being scanned by Defender to improve system performance?
> **Answer:** In an enterprise environment, this should never be done locally by the user. It is done by creating an **Antivirus Policy** in Microsoft Intune (Endpoint Security -> Antivirus). You add a **Folder Exclusion** (e.g., `C:\Program Files\HeavyDB\`) so Defender skips scanning files inside that directory, which significantly helps resolve high CPU or I/O performance issues.

> [!question] Q4: What happens technically when you "Isolate" a device from the MDE portal?
> **Answer:** Isolating a device immediately disconnects it from the corporate network and the internet, preventing lateral movement of malware or data exfiltration. However, it explicitly maintains a dedicated secure connection to the MDE service. This allows security admins to still run Live Response commands, fetch logs, and investigate the device remotely while the threat is contained.

> [!question] Q5: What is Attack Surface Reduction (ASR) and can you give an example of a rule?
> **Answer:** ASR rules are a set of controls that restrict behaviors commonly used by malware to infect machines. An example is the rule "Block executable content from email client and webmail." This rule prevents a user from running a `.exe` file that was directly downloaded from an Outlook email attachment, significantly reducing the chance of a successful phishing attack.

==**Exam Tip:** For the MD-102 exam, always remember that to connect Intune and MDE successfully, you must establish the service-to-service connection in **both** portals: enabling the connector in the Intune portal, and allowing Intune to enforce policies in the MDE advanced features settings. You also need to deploy the onboarding policy.==

---
## 🔗 Related Notes

- [[INT-01 Intune Architecture Overview]] — *Base Intune concepts for device management*
- [[INT-05 Compliance Policies]] — *How MDE machine risk score directly affects Intune Compliance status*
- [[SEC-01 Active Directory Security Basics]] — *General security principles and identity protection*
- [[INT-08 Windows Autopilot]] — *Provisioning new devices with MDE built-in*
