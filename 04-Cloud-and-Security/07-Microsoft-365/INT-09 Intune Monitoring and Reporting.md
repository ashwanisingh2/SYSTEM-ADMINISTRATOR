---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-09-intune-monitoring-and-reporting, int-09]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

# INT-09: Intune Monitoring and Reporting

> [!abstract] Overview
> This note covers monitoring, auditing, and reporting tools in Microsoft Intune, detailing compliance dashboards, App Installation states, Endpoint Analytics, Audit Logs, and Microsoft Graph API queries.

---

---
## Concept Overview
Think of Intune Monitoring and Reporting as a dashboard in an airport control tower. The dashboard displays the status of all flights (devices) in real-time.
- **Compliance Reports** show which flights are meeting safety standards (e.g., proper fuel and functional landing gear).
- **App Installation Reports** track if luggage (applications) has successfully loaded onto each aircraft.
- **Endpoint Analytics** functions like the maintenance prediction system, showing which planes are taking too long to start up (long boot times) or experiencing frequent engine issues (application crashes) before pilot complaints occur.
- **Audit Logs** act as the black box flight recorder, logging every change made by the air traffic controllers (administrators).

---

---
## Technical Deep Dive
### 1. Intune Reporting Categories
- **Operational Reports**: Fast, real-time data displaying immediate status (e.g., device sync status, active policy compliance).
- **Organizational Reports**: Summarized status aggregated across the entire tenant. Used for executive reporting and auditing.
- **Historical Reports**: Long-term trends tracked over time (e.g., device health evolution over 30 days).

### 2. Core Monitoring Dashboards

#### Device Compliance Reports
Identifies devices that fail compliance policies. It lists the specific settings that caused the failure, allowing administrators to troubleshoot remediation steps.

#### App Installation Status
Tracks installation states (Succeeded, Failed, Pending, Not Applicable) for deployed applications. It displays detailed error codes (e.g., `0x87D1041C` or `1603`) to pinpoint installation issues.

#### Endpoint Analytics
Analyzes device startup performance and software reliability:
- **Startup performance**: Measures boot and login times, separating hardware initialization, Group Policy processing, and user desktop loading times.
- **Work from anywhere**: Scores device preparedness for hybrid work.
- **Application reliability**: Tracks application crashes (app hangs, exceptions) and calculates a reliability score to help identify unstable software.

### 3. Intune Audit Logs & Log Analytics
- **Audit Logs**: Records tenant changes, including policy creations, user deletions, and device wipes. Logs specify which administrator made the change, the timestamp, and the modified settings.
- **Log Analytics Integration**: Exports Intune logs to an Azure Log Analytics Workspace, allowing complex queries using Kusto Query Language (KQL) and setting up automated alerts for critical events (e.g., alert when a device wipe is initiated).

### 4. Graph API Reporting
Intune metadata is accessible via the **Microsoft Graph API** (`https://graph.microsoft.com/v1.0/deviceManagement`). This allows administrators to run PowerShell queries to extract data for custom reporting dashboards.

---

## Common Mistakes
> [!warning] Avoid These
> Relying solely on the real-time operational dashboard for long-term auditing and compliance reporting. Operational data is purged regularly; configure diagnostic settings to export logs to Log Analytics for long-term retention.
> Interpreting an "Error" status in the app installation report as an application failure without checking if the app is already installed. If the detection rule is misconfigured, it will report an error even if the app installed successfully.
> Neglecting Endpoint Analytics alerts, resulting in users working on unstable systems with slow boot times and frequent application crashes.

---

## Pro Tips
> [!tip] Field Experience
> Integrate Intune with Azure Log Analytics and build custom dashboards. You can construct a single dashboard displaying compliance rates, active threats, and pending updates, and configure email alerts for critical failures.
> Use the **Graph Explorer** web tool (`https://developer.microsoft.com/graph/graph-explorer`) to test and build Graph API queries before automating them in PowerShell scripts.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Access to Microsoft Intune Admin Center with Intune Administrator permissions. Microsoft Graph PowerShell Module installed on an administration workstation.

### Step 1: Analyze Endpoint Analytics
Review device startup performance:
1. Open the **Microsoft Intune Admin Center** -> `Reports` -> `Endpoint analytics` -> `Startup performance`.
2. Review the **Startup score** (ranges from 0 to 100).
3. Click on the **Device startup history** tab to identify which machines have boot times longer than 2 minutes.
4. Click on **App reliability** to review which applications have the highest crash rates in the organization.

### Step 2: Query Intune Audit Logs
Filter logs to find who created a configuration profile:
1. Navigate to `Tenant administration` -> `Audit logs`.
2. Click **Filter** -> Set Activity category to **Device Configuration** -> Set Activity type to **Create**.
3. Review the results, select a log entry, and inspect the details pane to identify the administrator and settings.

### Step 3: Query Intune Device Details using Graph API and PowerShell
Run a PowerShell query to retrieve device inventory information from the Graph API:
```powershell
# Authenticate to Microsoft Graph
Connect-MgGraph -Scopes "DeviceManagementManagedDevices.Read.All"

# Query managed devices details
$devices = Get-MgDeviceManagementManagedDevice -Select "displayName,operatingSystem,complianceState,emailAddress"

# Export the device inventory to a CSV file
$devices | Select-Object DisplayName, OperatingSystem, ComplianceState, EmailAddress | Export-Csv -Path "C:\IntuneInventory.csv" -NoTypeInformation
```

### Step 4: Setup an Alert for Failed App Deployments
1. In the Intune portal, navigate to `Tenant administration` -> `Diagnostic settings`.
2. Click **Add diagnostic setting** -> Select **OperationalLogs** and **AuditLogs**.
3. Send to: Select **Log Analytics Workspace**.
4. Save the setting to enable custom alerting on failures using KQL queries in Azure Monitor.

---

---
## Cheat Sheet / Quick Reference
```powershell
# Authenticate to Microsoft Graph API with scope to read device configurations
Connect-MgGraph -Scopes "DeviceManagementConfiguration.Read.All"
# Get all active Windows Autopilot device statuses via Graph
Get-MgDeviceManagementWindowsAutopilotDeviceIdentity
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Endpoint Analytics | Dashboard displaying system performance metrics like boot times and app crash rates. |
| 2 | Audit Logs | Chronological ledger recording all administrative changes made within the tenant. |
| 3 | Graph API | API gateway used to query and manage Intune tenant settings and devices. |
| 4 | Diagnostic Settings | Configuration option to export Intune log data to Azure storage or Log Analytics. |
| 5 | Operational Report | Fast, real-time dashboard displaying immediate status information (e.g., device sync status). |

---

---
## Troubleshooting
**Scenario 1:**
- Problem: An application deployment fails on several machines, and the app installation report displays a generic "Failed" status with error code `0x87D300C9`.
- Root Cause: The installation timed out. The application installer was waiting for user interaction silently, or the download took longer than the allocated timeframe.
- Fix:
  1. Go to the Intune portal -> `Apps` -> `All apps` -> Select the failing application.
  2. Review the install command parameters and verify the silent switches are correct (e.g., `/qn`, `/S`, or `/quiet`).
  3. Ensure that the app package has not exceeded the maximum execution time limit configured in the application properties.
  4. Test the installation command locally on a test machine in the SYSTEM context using PsExec (`psexec -i -s cmd.exe`) before deploying it via Intune.

**Scenario 2:**
- Problem: A critical configuration profile is modified by an unknown user, resulting in Wi-Fi connectivity dropping for several devices.
- Root Cause: Unauthorized configuration changes were made without documentation.
- Fix:
  1. Open the **Intune Admin Center** -> `Tenant administration` -> `Audit logs`.
  2. Add a filter for the date range when the issue started and set the Activity category to **Device Configuration**.
  3. Locate the entry for the profile modification and inspect the log details.
  4. Identify the administrator account that initiated the change and the modified parameters.
  5. Revert the configuration settings and update internal change logs.

---

---
## Interview Questions
**Q1: What is Endpoint Analytics in Microsoft Intune, and how can it help administrators improve device performance?**
A: Endpoint Analytics is an analytics service that measures and scores user experience parameters, such as device startup performance and application reliability. It calculates scores based on boot times, Group Policy processing delays, and application crash frequency. This helps administrators identify performance issues (e.g., outdated hardware or unstable software) and remediate them before users report issues.

**Q2: An administrator needs to extract a list of all devices that have not synced with Intune for over 30 days. How would you automate this?**
A:
- **Situation**: The company needed to identify inactive devices in Intune to reclaim licenses.
- **Task**: I needed to generate a report of devices that had not synced for over 30 days.
- **Action**: I wrote a PowerShell script using the Microsoft Graph API module. I queried the `managedDevices` endpoint, filtering by the `lastSyncDateTime` property: `Get-MgDeviceManagementManagedDevice -Filter "lastSyncDateTime lt (Get-Date).AddDays(-30)"`. I exported the results (Display Name, OS, User Email, and Last Sync Date) to a CSV file.
- **Result**: The script generated the inactive device report, allowing the IT team to review and retire the inactive machines.

**Q3: How do you debug a failed application deployment in Intune when the console only displays a generic error code?**
A: To debug a failed application deployment, I log onto the client machine and open the Intune Management Extension logs located at `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log`. I search the log file for the application ID or installer file name to find the execution logs, exit codes, and detection rule outcomes. This reveals the actual installer failure reason, which is often masked by generic console codes.

---

---
## Seedha Simple Mein
*Seedha simple mein: Intune reporting aur monitoring tools ke zariye aap devices ki health, application install status, system performance audit aur configuration errors ko investigate karte hain.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes identity joins and MDM authority.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Details compliance settings and health checks.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details device configuration templates.
