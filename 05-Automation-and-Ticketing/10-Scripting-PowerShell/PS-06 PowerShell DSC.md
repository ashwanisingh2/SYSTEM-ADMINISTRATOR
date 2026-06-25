---
tags: [automation, powershell, dsc, configuration, windows-server]
aliases: [powershell-dsc]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# PS-06: PowerShell Desired State Configuration (DSC)

**Verification:**
- [ ] Author a declarative DSC configuration block in PowerShell
- [ ] Compile the DSC configuration script into a Managed Object Format (`.mof`) file
- [ ] Configure the Local Configuration Manager (LCM) settings on a target node
- [ ] Apply the configuration using `Start-DscConfiguration` and verify compliance
- [ ] Test self-healing behavior by triggering configuration drift and monitoring auto-correction

> [!abstract] Overview
> PowerShell Desired State Configuration (DSC) is a management platform that enables infrastructure as code. This note covers the declarative syntax, Push vs. Pull models, the role of the Local Configuration Manager (LCM), compilation of MOF files, and drift remediation techniques.

---
## Concept Overview
- **What it is** — Desired State Configuration (DSC) is a management platform in PowerShell that enables you to manage your IT infrastructure using configuration as code. It allows you to declare how your servers should be configured, rather than scripting the step-by-step commands to configure them.
- **Why it matters for a support engineer** — In enterprise environments, manual server configuration leads to "configuration drift" (where servers slowly become inconsistent over time due to ad-hoc edits). DSC ensures all servers match a single, audited master configuration baseline, automatically correcting unauthorized changes.
- **Where you encounter this in real job** — Ensuring IIS is installed and configured on web servers, making sure security registry keys are set identically across domain members, and verifying that required folders and local administrative accounts exist.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - **L1 Resolution**: Verify if the local DSC agent is active, check server compliance using `Test-DscConfiguration`, and gather LCM logs for escalation.
  - **Escalation Trigger**: Escalate to L2 if configurations fail to apply due to access-denied errors, compilation failures, or missing DSC module dependencies on the target node.
  - **L2 Resolution**: Author standard DSC configuration scripts, compile configurations into `.mof` files, manage local LCM settings, and run local configurations using `Start-DscConfiguration`.
  - **L3 Resolution**: Deploy and maintain an enterprise-wide DSC Pull Server, write custom DSC Resource modules, and integrate DSC with cloud automation tools (such as Azure Automanage Machine Configuration).

*Seedha simple mein: PowerShell DSC hume yeh specify karne ki permission deta hai ki server kis state mein hona chahiye (jaise IIS running ho aur key registry set ho). Hume yeh nahi likhna padta ki use kaise install karna hai; DSC server par check karta hai aur agar configurations badal gayi hain, toh use automatically correct kar deta hai.*

---
## Real-World Analogy
Think of a **smart home thermostat**:
- **Imperative Scripting (Standard PowerShell)**: You monitor the room temperature and write a script saying: "If temp is 18°C, turn on heater. If temp is 25°C, turn off heater." You must continuously run and manage this script.
- **Declarative Configuration (DSC)**: You simply set the dial to `22°C` (the "Desired State"). The thermostat's internal logic (LCM) constantly monitors the room temperature. If it drifts (drops to 20°C), the thermostat automatically fires up the heater until the desired 22°C is restored.

---
## Technical Deep Dive

### 1. Declarative vs. Imperative Programming
- **Imperative (How)**: Standard scripts. You write step-by-step logic. e.g., "Check if Folder exists. If not, create it. Check if permission is correct. If not, run icacls."
- **Declarative (What)**: DSC. You define the final target state. The DSC resource contains the underlying code to handle the checks and corrections.

```powershell
# Imperative approach
if (!(Test-Path C:\WebData)) { New-Item -Path C:\WebData -ItemType Directory }

# Declarative (DSC) approach
File WebDirectory {
    Ensure          = "Present"
    DestinationPath = "C:\WebData"
    Type            = "Directory"
}
```

### 2. Core Components of DSC
- **Configurations**: The PowerShell scripts that define the resources and their properties.
- **Resources**: The built-in or custom code blocks that perform the check and change actions (e.g., `File`, `Service`, `Registry`, `WindowsFeature`).
- **MOF (Managed Object Format) File**: The compiled output of a DSC script. The local machine reads this standard document (WMI-based) to apply settings.
- **Local Configuration Manager (LCM)**: The engine running on the target node that receives, compiles, and enforces DSC configurations.

### 3. Push vs. Pull Architecture
- **Push Mode (Default)**: An administrator manually pushes MOF configuration files to target servers using `Start-DscConfiguration`. Best for small test labs.
- **Pull Mode**: Target servers check in with a central DSC Pull Server (via HTTPS) on a schedule. If a new configuration is available, they pull the MOF file and apply it locally. Best for large production scale.

### 4. Local Configuration Manager (LCM) Modes
The LCM behavior is defined by the `ConfigurationMode` setting:
- **ApplyOnly**: The configuration is applied once, and no further monitoring occurs (unless a new config is pushed).
- **ApplyAndMonitor**: The configuration is applied. The LCM monitors the system, and if drift is detected, it logs the non-compliance but does not fix it.
- **ApplyAndAutoCorrect**: The configuration is applied. The LCM actively monitors and automatically corrects configuration drift at every refresh cycle.

---
## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - Windows Server 2022 VM (`SVR-WEB01`).
> - PowerShell Console running as Administrator.
> - Execution Policy set to RemoteSigned: `Set-ExecutionPolicy RemoteSigned -Force`.

### Step 1: Write a DSC Configuration Script
We will create a configuration that ensures the **Web-Server (IIS)** feature is installed, the IIS service is running, and a custom homepage exists.

1. Open a PowerShell ISE or VS Code window.
2. Save the following script as `C:\DSCLab\WebServerConfig.ps1`:
```powershell
# Define the configuration block
Configuration WebServerConfig {
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    Node "localhost" {
        # Ensure IIS is installed
        WindowsFeature IISFeature {
            Ensure = "Present"
            Name   = "Web-Server"
        }

        # Ensure IIS Service is running
        Service IISService {
            Name      = "w3svc"
            StartupType = "Automatic"
            State     = "Running"
            DependsOn = "[WindowsFeature]IISFeature"
        }

        # Ensure custom index.html is present
        File IndexPage {
            Ensure          = "Present"
            DestinationPath = "C:\inetpub\wwwroot\index.html"
            Contents        = "<h1>Configured by PowerShell DSC</h1>"
            DependsOn       = "[Service]IISService"
        }
    }
}
```

### Step 2: Compile the Configuration into a MOF File
1. In your PowerShell window, load and execute the configuration function to compile it:
```powershell
. C:\DSCLab\WebServerConfig.ps1
WebServerConfig -OutputPath C:\DSCLab\MOF
```
**Expected Output:**
```text
    Directory: C:\DSCLab\MOF

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2026-06-25 18:10 PM           1452 localhost.mof
```

### Step 3: Configure LCM Settings (Meta-Configuration)
We will configure the Local Configuration Manager to automatically monitor and auto-correct configuration drift.

1. Save the following code as `C:\DSCLab\LCMConfig.ps1` and run it to compile and apply:
```powershell
[DSCLocalConfigurationManager()]
Configuration ConfigureLCM {
    Node "localhost" {
        Settings {
            ConfigurationMode = "ApplyAndAutoCorrect"
            RefreshFrequencyMins = 30
            ConfigurationModeFrequencyMins = 15
            RebootNodeIfNeeded = $true
        }
    }
}
ConfigureLCM -OutputPath C:\DSCLab\LCM
Set-DscLocalConfigurationManager -Path C:\DSCLab\LCM -Verbose
```
**Expected Output:** Verbose output shows LCM settings being updated on the local host.

### Step 4: Apply the Configuration
1. Start the configuration process in Push mode:
```powershell
Start-DscConfiguration -Path C:\DSCLab\MOF -Wait -Verbose -Force
```
**Expected Output:** Verbose messages showing the LCM installing IIS, starting the service, and creating the `index.html` file.

### Step 5: Test Drift and Self-Healing
1. Manually test compliance:
```powershell
Test-DscConfiguration
```
**Expected Output:** `True`

2. Force drift by deleting the homepage and stopping the IIS service:
```powershell
Stop-Service -Name w3svc
Remove-Item -Path C:\inetpub\wwwroot\index.html -Force
Test-DscConfiguration
```
**Expected Output:** `False`

3. Force the LCM to run a consistency check immediately rather than waiting for the 15-minute cycle:
```powershell
Start-DscConfiguration -UseExisting -Wait -Verbose
```
**Expected Output:** DSC detects the missing file and stopped service, restarts `w3svc`, recreates `index.html`, and restores compliance.

---
## Cheat Sheet

| Command / Setting | Purpose | Example |
|---|---|---|
| `Start-DscConfiguration` | Applies a compiled MOF configuration to target nodes | `Start-DscConfiguration -Path C:\DSCLab\MOF -Wait -Verbose` |
| `Test-DscConfiguration` | Checks whether target node is currently compliant | `Test-DscConfiguration -Detailed` |
| `Get-DscConfiguration` | Returns the current configuration settings applied to the node | `Get-DscConfiguration` |
| `Get-DscLocalConfigurationManager` | Gets LCM properties and configuration mode details | `Get-DscLocalConfigurationManager` |
| `Set-DscLocalConfigurationManager` | Applies meta-configurations to LCM | `Set-DscLocalConfigurationManager -Path C:\LCMConfig` |
| `Get-DscResource` | Lists all DSC resources available on the local machine | `Get-DscResource -Name *Service*` |
| `Update-DscConfiguration` | Forces target node in Pull mode to check pull server for new configs | `Update-DscConfiguration -Wait -Verbose` |

---
## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Error: "The WinRM client cannot process the request... connection refused." | WinRM service is stopped or not listening on the target node. | Run `winrm quickconfig` or `Enable-PSRemoting -Force` on the target machine to enable WinRM. |
| Configuration compilation fails with "Undefined DSC Resource". | The target resource (e.g. `xWebAdministration`) is not installed on the system compiling the script. | Download resource from gallery: `Install-Module -Name PSDesiredStateConfiguration -Force` or the custom modules folder. |
| Test-DscConfiguration returns `False` but no auto-correction occurs. | LCM configuration mode is set to `ApplyOnly` or `ApplyAndMonitor`. | Reconfigure LCM using a meta-configuration script with `ConfigurationMode = 'ApplyAndAutoCorrect'`. |
| MOF file name does not match target host name, causing deployment failure. | Target node name in configuration script does not match the actual hostname or domain name of the destination computer. | Ensure node names in the DSC configuration block match the exact hostnames or use `"localhost"` for local tests. |
| DSC service logs contain corruption or locked database errors. | Local WMI/CIM repository or DSC database corrupted due to unexpected shut down. | Run `Reset-DscConfiguration` to clear active jobs, or restart target WinRM and WMI services. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is configuration drift, and how does PowerShell DSC prevent it?
> **A:** Configuration drift is the gradual divergence of a server's configuration from its original design baseline (often caused by manual, undocumented edits by administrators). PowerShell DSC prevents this by checking the system state against the configured MOF definition at a regular frequency using the Local Configuration Manager (LCM). If the LCM detects that the target state is no longer compliant (drifted), it automatically re-applies the desired configuration to restore the baseline (when configured in `ApplyAndAutoCorrect` mode).

> [!question] L2 Question
> **Q:** What is the role of a `.mof` file in PowerShell DSC, and how is it generated?
> **A:** A `.mof` (Managed Object Format) file is the standardized configuration blueprint that is actually read and processed by the Local Configuration Manager (LCM) engine. It is platform-independent. In DSC, it is generated by writing a declarative configuration script in PowerShell and compiling it by executing the configuration name as a function with an output directory parameter.

> [!question] L3/Scenario Question
> **Q:** You are designing a DSC architecture for 200 virtual machines in a hybrid cloud environment. Would you recommend a Push or Pull architecture? Detail how you would implement and secure the chosen method.
> **A:** 
> - **Situation:** Provisioning DSC for a scale of 200 hybrid cloud VMs.
> - **Task:** Choose and design a scalable, secure configuration architecture.
> - **Action:** I would recommend a **Pull-based architecture**. A Push model requires the administrator's workstation to establish active WinRM connections to all 200 hosts, which fails across firewall zones and is not scalable. For the Pull architecture, I will configure a central **NPS/IIS Web DSC Pull Server** (or Azure Automation State Configuration). Target nodes will be configured via LCM meta-configurations to query this server over HTTPS (port 443).
> - **Result:** This ensures nodes check in automatically, traffic is encrypted via TLS, authentication is managed using a secure Registration Key, and configuration updates can be rolled out centrally by updating a single MOF file on the pull server.

---
## Related Notes
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-04 PowerShell Scripting|PS-04 PowerShell Scripting]] — Structuring scripts and functions.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-03 DNS Server — Install and Configure|WS-03 DNS Server — Install and Configure]] — Host name resolution for Pull servers.
- [[05-Automation-and-Ticketing/12-Ansible/ANS-01 Ansible for Windows Admins|ANS-01 Ansible for Windows Admins]] — Comparison of configuration management tools for Windows.

---
*Tags: #automation #powershell #dsc #configuration #windows-server #cert-none*
