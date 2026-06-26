---
tags: [microsoft-365, intune, endpoint-analytics, md-102]
aliases: [Endpoint Analytics]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> ☁️ **MICROSOFT 365 / INTUNE**

`#complete` `#intermediate` `#md-102`

# INT-05: Endpoint Analytics

> [!abstract] Overview
> Endpoint Analytics is a cloud-based service in Microsoft Intune that provides insights into the performance and health of devices in your organization. Yeh ek Intune service hai jo aapke company ke laptops aur devices ka health aur performance monitor karti hai, taaki hardware aur software issues proactively fix kiye ja sake. Ek support engineer ko iska knowledge hona zaroori hai taaki wo user experience (slow boot, app crashes) ko improve kar sake bina unke complain kiye.

---
## 🧠 Concept Overview

- **What it is** — Endpoint Analytics provides metrics and insights into device performance, focusing on startup performance, application reliability, and work from anywhere readiness.
- **Why it matters** — Traditional IT relies on user tickets. Endpoint Analytics allows IT to proactively identify and resolve issues before they impact productivity. *Isse ticket aane se pehle hi IT team problem solve kar sakti hai.*
- **Where you see this** — Used heavily by modern IT teams managing Windows 10/11 fleets via Intune to decide on hardware refresh cycles, identify problematic updates, and ensure smooth operations.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check device score, verify if a specific device has slow startup or app crash history when troubleshooting. |
| **L2** | Configure proactive remediations, analyze baseline metrics, investigate widespread app reliability drops, export reports. |
| **L3** | Architecture, design baseline strategies, integrate with Log Analytics, enterprise-level reporting, custom remediation script creation. |

> [!tip] Seedha Simple Mein
> *Endpoint Analytics aapke computers ka ek "Health Report Card" hai. Yeh batata hai ki kaunsa laptop slow boot ho raha hai, kis app mein sabse zyada error aa rahe hain, aur kis laptop ko nayi RAM ya SSD chahiye. Isse user complain kare usse pehle aapko problem pata chal jaati hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Car Dashboard & Telematics** is like **Endpoint Analytics** because...
>
> - Jab car mein kuch kharabi aane wali hoti hai (jaise engine oil low or tire pressure low), dashboard pe warning light aa jati hai, pehle hi. Endpoint Analytics bhi same warning deta hai IT team ko.
> - Car telematics poori fleet (bahut saari gaadiyo) ka data batata hai (kaunsi car zyada fuel le rahi hai). Waise hi yeh dashboard poori company ke laptops ka health dikhata hai.
> - Mechanic service log check karta hai problem fix karne se pehle, waise hi IT admin Analytics check karta hai laptop touch karne se pehle.

---
## 🔬 Technical Deep Dive

### 1. Key Metrics & Scores

> [!info] Key Concept
> Endpoint Analytics calculates a score out of 100 for your organization, based on several key metrics. The higher the score, the better the user experience. You can compare this score against an industry baseline.

- **Startup Performance:** Measures how long it takes for devices to boot up and become usable. *Laptop on hone mein kitna time lagta hai, aur kaunsi startup app boot process ko delay kar rahi hai.*
- **Application Reliability:** Tracks how often desktop applications crash or hang. *Kaunsa software baar baar crash ho raha hai (e.g., Teams, Excel, or custom LOB apps).*
- **Work From Anywhere:** Assesses device readiness for modern management and cloud identities, particularly Windows 11 readiness. *Check karta hai ki laptop Windows 11 ke liye ready hai ya nahi.*
- **Battery Health (preview):** Helps identify devices with degraded batteries. *Kaunse laptop ki battery jaldi drain ho rahi hai aur change karne ki zaroorat hai.*

> [!danger] Common Mistake
> Ek common mistake hai Endpoint Analytics ko enable karna but "Proactive Remediations" ko use na karna. Agar sirf data dekh rahe ho aur script se fix nahi kar rahe, toh iska full fayda nahi milta.

### 2. Proactive Remediations (Now Remediations)

Proactive remediations are PowerShell scripts that detect and fix common support issues on a user's device before they even realize there's a problem.

- **Detection Script:** Checks if a condition exists (e.g., stale group policy, missing reg key, stopped service). *Yeh script check karti hai ki background mein koi problem hai ya nahi.*
- **Remediation Script:** Runs only if the detection script finds an issue to fix it. *Agar problem milti hai, toh yeh script execute hoti hai aur usko silently fix kar deti hai.*

### 3. Baselines

A baseline allows you to track progress over time. You can compare your current organizational score to a historical baseline (e.g., 6 months ago) or an "All Organizations" median score. *Baseline se pata chalta hai ki humara IT environment pehle se improve hua hai ya degrade.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Intune Administrator or Endpoint Security Manager role.
> - Devices must be Intune enrolled or co-managed with Configuration Manager.
> - Windows 10/11 Pro, Enterprise, or Education editions.
> - Devices must have diagnostic data collection (Telemetry) turned on.

### Step 1: Onboard Devices to Endpoint Analytics

To start collecting data, you must enable the feature in the Intune console. This is done via **Intune Portal > Reports > Endpoint analytics > Settings**.

```powershell
# While onboarding is mostly done via the Intune Portal GUI, 
# ensuring diagnostic data is enabled on endpoints is critical.
# Yeh command diagnostic data level check karne ke liye use hoti hai (Require 'Optional' or 'Required' depending on OS version)
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" -Name "AllowTelemetry"
```

> [!success] Expected Output
> ```
> AllowTelemetry : 1  # 1 (Basic) or 3 (Full) depending on your config. For Endpoint analytics, usually Intune policy handles this.
> ```

### Step 2: Create a Proactive Remediation Script

Let's say we want to automatically restart the Print Spooler service if it unexpectedly stops.

**Detection Script (DetectSpooler.ps1):**
```powershell
# Check if the print spooler is running
$Service = Get-Service -Name Spooler
if ($Service.Status -eq "Running") {
    Write-Output "Spooler is running."
    exit 0 # No issue found, exit 0 means success
} else {
    Write-Warning "Spooler is NOT running."
    exit 1 # Issue detected, trigger remediation with exit 1
}
```

**Remediation Script (FixSpooler.ps1):**
```powershell
# Start the print spooler to fix the issue
Start-Service -Name Spooler
Write-Output "Spooler has been started successfully."
```

### Step 3: Deploy the Remediation Script
1. Go to **Intune Portal > Devices > Remediations**.
2. Click **Create script package**.
3. Upload `DetectSpooler.ps1` as the Detection script and `FixSpooler.ps1` as the Remediation script.
4. Assign it to a device group and set a schedule (e.g., daily).

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command / Path | 🛠️ Kya karta hai | 📝 Example |
|-------------------|-----------------|-----------|
| `Get-Service` | Services ka status check karta hai | `Get-Service Spooler` |
| `Start-Service` | Stopped service ko start karta hai | `Start-Service Spooler` |
| `Exit 0` | Detection script mein means 'All is well' | `Exit 0` |
| `Exit 1` | Detection script mein means 'Problem found' | `Exit 1` |
| Intune Portal -> Reports -> Endpoint Analytics | Endpoint Analytics dashboard open karta hai | N/A |
| `Get-ItemProperty` | Registry key read karta hai (Telemetry verify karne ke liye) | `Get-ItemProperty HKLM:\...\DataCollection` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Device not showing in Endpoint Analytics | Diagnostic data policy is blocked or not set. | Ensure Windows telemetry is set to Required/Optional via Intune Device Configuration profile. *Telemetry on karo GPO ya Intune se.* |
| "No Data" for a specific app | The app is not widely used or data hasn't aggregated yet. | Wait 24-48 hours. Endpoint analytics requires a critical mass of events to show reliable metrics. *Thoda wait karo, Cloud pe data aane mein time lagta hai.* |
| Remediation script failing | Execution policy or incorrect script logic (e.g. wrong exit codes). | Test scripts locally running as SYSTEM using Psexec. Ensure Exit 0 means Success and Exit 1 means failure. *Script ko pehle local machine pe SYSTEM context mein test karo.* |
| Battery health not showing | Feature not supported on that specific hardware or OS build. | Verify device meets hardware requirements. Virtual Machines (VMs) do not report battery health. *Check karo device VM toh nahi hai.* |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Executive Complains of Slow Laptop

> [!example] Ticket
> "My laptop takes forever to start up every morning. Please fix this immediately."

**L1 Response:** L1 checks the Endpoint Analytics -> Startup Performance tab and searches for the user's device name to verify the actual boot time.
**Escalation Trigger:** The dashboard shows a specific third-party security agent (e.g., an old Antivirus) is adding 120 seconds to the Core Boot Time.
**L2 Resolution:** L2 creates an Intune policy to update or remove the conflicting agent, or works with the security team to optimize its startup behavior. *L2 report se confirm karega ki exactly kaunsa app slow kar raha hai aur usko fix karega.*

### 🎫 Scenario 2: Widespread Teams Crashes

> [!example] Ticket
> "Several users reporting that Microsoft Teams keeps crashing randomly during calls."

**L1 Response:** L1 checks the Application Reliability report in Endpoint Analytics.
**Escalation Trigger:** L1 confirms that the crash rate for `teams.exe` has spiked in the last 3 days across 500 devices.
**L2 Resolution:** L2 checks the version of Teams that is crashing. Discovers a recent auto-update caused the issue. L2 coordinates with Microsoft Support or deploys a known stable version via Intune. *L2 app reliability chart dekh kar pattern samjhega.*

### 🎫 Scenario 3: Identifying Devices for Hardware Refresh

> [!example] Ticket
> "IT Management needs a list of devices that should be replaced next quarter due to poor performance."

**L1 Response:** This is a reporting task, escalated to L2/L3 immediately.
**Escalation Trigger:** Needs bulk data export from Endpoint Analytics.
**L2 Resolution:** L2/L3 exports the Device Performance report. They filter for devices with the lowest 'Endpoint analytics score' and devices with high 'Boot time' and degraded 'Battery health'. This data is used to justify hardware replacements. *Purane slow laptops ko identify karke replace karne ki list banai jayegi.*

### 🎫 Scenario 4: Windows 11 Upgrade Readiness

> [!example] Ticket
> "We need to plan our migration to Windows 11. How many devices are compatible?"

**L1 Response:** Escalates to L2 as this involves fleet-wide analysis.
**Escalation Trigger:** Requirement for a mass OS upgrade report.
**L2 Resolution:** L2 navigates to the 'Work from anywhere' -> 'Windows 11 readiness' report. They export the list of devices marked as "Capable" and those marked as "Not capable" (due to TPM or CPU requirements) to plan the budget. *L2 check karega kaunse devices mein naya CPU/TPM 2.0 chahiye.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the main purpose of Endpoint Analytics in Microsoft Intune?
> **Answer:** Its main purpose is to proactively monitor device performance, app reliability, and startup times to improve user experience before users open helpdesk tickets.

> [!question] Q2: In Proactive Remediations, what is the role of Exit Codes in the Detection script?
> **Answer:** The exit code determines if the remediation script will run. If the detection script returns `Exit 0`, it means the device is healthy (no issue found). If it returns `Exit 1`, an issue is detected, and the remediation script will trigger.
> ==**Exam Tip:** Exit 0 = All Good. Exit 1 = Problem Found (Run Remediation).==

> [!question] Q3: A device is enrolled in Intune but not reporting data to Endpoint Analytics. What is the most likely cause?
> **Answer:** Windows Diagnostic data (telemetry) is likely disabled or blocked by another policy (like a GPO) on the device. Devices must send diagnostic data to Microsoft.

> [!question] Q4: How does Endpoint Analytics calculate "Startup Performance"?
> **Answer:** It measures the time from power-on to the sign-in screen (Core Boot Time), and the time from sign-in to the desktop being fully usable (Core Sign-in Time).

> [!question] Q5: What is a Baseline in Endpoint Analytics and why is it useful?
> **Answer:** A baseline is a snapshot of your organization's score at a point in time or an industry median. It is useful for tracking if your IT environment is improving or degrading over time (e.g., after rolling out a new security agent).

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Intro to Intune|INT-01 Intro to Intune]] — Foundation of device management.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 App Deployment|INT-04 App Deployment]] — How apps are deployment, which relates to app reliability.
- [[04-Cloud-and-Security/07-Microsoft-365/AZ-01 Azure AD Basics|AZ-01 Azure AD Basics]] — Identity management used by Intune.
- [[01-Windows/PowerShell/PS-01 Intro to PowerShell|PS-01 Intro to PowerShell]] — Essential for writing Proactive Remediation scripts.
