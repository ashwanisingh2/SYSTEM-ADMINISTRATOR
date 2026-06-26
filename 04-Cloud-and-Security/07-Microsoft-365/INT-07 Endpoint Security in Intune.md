---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-07-endpoint-security-in-intune, int-07]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

# INT-07: Endpoint Security in Intune

> [!abstract] Overview
> This note covers Endpoint Security in Microsoft Intune, detailing Defender AV policies, Attack Surface Reduction (ASR) rules, BitLocker management with Azure AD key escrowing, and Microsoft Security Baselines.

---

---
## Concept Overview
Think of Endpoint Security in Intune as configuring defenses for a corporate fort.
- **Antivirus Policies** are like putting guards at the entry gates with photos of known bad actors (malware signatures).
- **Firewall Policies** are like checkpoint controls determining which delivery trucks (network ports) can enter or exit.
- **Attack Surface Reduction (ASR)** is like closing off unnecessary windows and locking secondary doors (blocking scripts from launching from email attachments or Office documents).
- **BitLocker** is like locking all paper files in an encrypted safe. If someone breaks in and steals the cabinet (the physical drive), they cannot read the files without the combination (the BitLocker Recovery Key stored in Azure AD).

---

---
## Technical Deep Dive
### 1. Microsoft Defender Integration
Intune integrates with Microsoft Defender for Endpoint (MDE) to evaluate client risk status:
- Once linked, the Intune agent installs the MDE sensor on client devices.
- Devices report real-time compliance signals (e.g., malware active) back to Intune and Entra ID to restrict network access.

### 2. Core Security Policies

#### Antivirus Policies
Configures Windows Defender settings, including real-time protection, cloud-delivered protection, exclusion lists (folders, files, processes), and scheduled quick/full scans.

#### Firewall Policies
Configures stateful firewall rules, enabling/disabling Domain, Private, and Public profiles, and defining specific inbound and outbound traffic rules.

#### Attack Surface Reduction (ASR) Rules
Blocks common malware entry paths. Key ASR rules include:
- Block executable content from email client and webmail.
- Block Office applications from creating child processes.
- Block credential stealing from the Windows local security authority subsystem (lsass.exe).
- Block process creations originating from PSExec and WMI commands.

### 3. BitLocker Encryption & Recovery Escrowing
Automates encryption of the operating system drive:
- Enforces silent encryption during device enrollment without user interaction.
- Requires a TPM chip (version 2.0).
- Backs up the recovery key directly to the device's object inside Microsoft Entra ID before starting the encryption process (Recovery Key Escrowing).

### 4. Microsoft Security Baselines
Pre-configured groups of settings recommended by Microsoft security teams. Deploys standard configurations for Windows OS, Defender, and Microsoft Edge, replacing custom configurations for standard deployments.

---

## Common Mistakes
> [!warning] Avoid These
> Enabling BitLocker policies that require a startup PIN on virtual machines or remote laptops without physical keyboard access at boot, locking out users.
> Enabling all Attack Surface Reduction (ASR) rules in "Enforce" mode instantly. ASR rules block common process behaviors. Always deploy ASR rules in **Audit** mode first to identify business-critical application conflicts.
> Neglecting to back up the BitLocker recovery keys to Azure AD before initiating the encryption process, resulting in unrecoverable data if a device malfunctions.

---

## Pro Tips
> [!tip] Field Experience
> When onboarding systems to Defender for Endpoint (MDE), create a Conditional Access policy that blocks access to corporate data if the machine risk level rises above "Medium". This isolates infected devices automatically.
> Regularly audit the **Microsoft Security Baselines** dashboard in Intune. When Microsoft updates a baseline, compare the new settings with your active configuration profiles before migrating target groups.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Access to Microsoft Intune Admin Center with Endpoint Security Manager role. A licensed Windows 11 client device.

### Step 1: Configure Defender Antivirus Policy
Configure real-time scanning and cloud-delivered protection:
1. Open the **Microsoft Intune Admin Center** -> `Endpoint Security` -> `Antivirus`.
2. Click **Create Policy** -> Platform: **Windows 10, Windows 11, and Windows Server** -> Profile: **Microsoft Defender Antivirus** -> Click **Create**.
3. Name: `Corp Defender AV Policy`.
4. Under **Configuration settings**:
   - Set **Turn on real-time protection** to **Yes**.
   - Set **Turn on cloud-delivered protection** to **Yes**.
   - Set **Submit all samples** to **Send safe samples automatically**.
   - Set **Scan parameter** to **Quick scan** (run daily at 2:00 AM).
5. Click **Next** -> Assign to the `All Devices` group -> Click **Create**.

### Step 2: Configure BitLocker Encryption with Azure AD Escrowing
Automate drive encryption and escrow recovery keys:
1. Navigate to `Endpoint Security` -> `Disk encryption`.
2. Click **Create Policy** -> Platform: **Windows 10 and later** -> Profile: **BitLocker** -> Click **Create**.
3. Name: `Corp BitLocker Encryption Policy`.
4. Under **Configuration settings**:
   - Set **Require Device Encryption** to **Enabled**.
   - Under **BitLocker - Base Settings**:
     - Set **Block warning for other disk encryption** to **Yes**.
     - Set **Configure recovery folder** to **Require backup to Azure AD**.
   - Under **BitLocker - OS Drive Settings**:
     - Set **System drive redundancy** to **Require TPM**.
     - Set **Compatible TPM startup PIN** to **Blocked** (to allow silent encryption without user prompt).
5. Click **Next** -> Assign to `All Devices` -> Click **Create**.

### Step 3: Onboard Devices to Defender for Endpoint (MDE)
1. Go to `Endpoint Security` -> `Endpoint detection and response`.
2. Click **Create Policy** -> Platform: **Windows 10 and later** -> Profile: **Endpoint detection and response**.
3. Name: `Corp MDE Onboarding Policy`.
4. Set **Microsoft Defender for Endpoint client configuration package type** to **Auto-onboard** (or upload the onboarding package from the Microsoft Defender Security portal).
5. Assign and create the policy.

### Step 4: Verify BitLocker Key in Azure AD
On the client machine (after the policy applies and encryption completes):
1. In the search box, type `Manage BitLocker` to verify the C: drive displays **BitLocker on**.
2. Open the **Microsoft Entra Portal** -> `Identity` -> `Devices` -> `All Devices`.
3. Search for the client device name -> Click on it -> Click **Show Recovery Key**.
4. Verify the 48-digit BitLocker recovery key is visible in the console.

---

---
## Cheat Sheet / Quick Reference
```powershell
manage-bde -status c:                              # Displays encryption percentage and TPM details on client
manage-bde -protectors -get c:                     # Displays current recovery password ID and key on client
Get-MpComputerStatus | Select-Object RealTimeProtectionEnabled, AMServiceEnabled # Verifies local Defender status
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Real-Time Protection | Antivirus engine that continuously scans files and processes for active malware. |
| 2 | ASR Rules | Security rules that block common malware transmission vectors (e.g., Office child processes). |
| 3 | TPM 2.0 | Hardware chip that provides cryptographic security functions required by BitLocker. |
| 4 | Key Escrowing | Automatically backups BitLocker recovery keys to Azure Active Directory. |
| 5 | Security Baseline | Group of Microsoft-recommended security configurations deployed as a single profile. |

---

---
## Troubleshooting
**Scenario 1:**
- Problem: An administrator deploys the BitLocker policy, but the Intune dashboard shows the policy state as "Error" with code `0x8018001a` (TPM not found).
- Root Cause: The client device does not have a Trusted Platform Module (TPM) chip, or the TPM is disabled in the system BIOS/UEFI.
- Fix:
  1. Boot the client machine into the BIOS/UEFI configuration menu.
  2. Locate the security settings and ensure that the **TPM 2.0** or **Intel PTT / AMD fTPM** security feature is enabled.
  3. Boot back into Windows and run `tpm.msc` to confirm the TPM status shows "The TPM is ready for use."
  4. Force a device sync in the Company Portal app to apply the BitLocker policy.

**Scenario 2:**
- Problem: An administrator enables the ASR rule "Block credential stealing from the Windows local security authority subsystem," but standard users complain that a critical legacy line-of-business app crashes upon startup.
- Root Cause: The legacy application is performing direct memory reading of `lsass.exe` to retrieve authentication parameters, triggering an ASR violation block.
- Fix:
  1. Open the client machine's Event Viewer -> Navigate to `Applications and Services Logs` -> `Microsoft` -> `Windows` -> `Windows Defender` -> `Operational`.
  2. Search for **Event ID 1121** (ASR audit block) or **1122** (ASR enforcement block) to find the path of the blocked app.
  3. Open the **Intune Admin Center** -> Navigate to the active ASR profile.
  4. In the settings configuration, add the path of the application executable to the **ASR Exclusion List**.

---

---
## Interview Questions
**Q1: How would you configure a silent BitLocker deployment in Intune, and where are the recovery keys stored?**
A: To configure silent BitLocker deployment, I create a Disk Encryption policy in Endpoint Security. I enable device encryption, require TPM, set the compatible TPM startup PIN to "Blocked" (which bypasses user prompts), and set the recovery folder to require backup to Microsoft Entra ID. The recovery keys are stored securely under the device's object inside the Microsoft Entra ID portal.

**Q2: A developer complains that their local testing tool is blocked from running PowerShell scripts by an Attack Surface Reduction rule. How do you resolve this?**
A:
- **Situation**: A developer's local tool was blocked by an active ASR rule, halting their workflow.
- **Task**: I needed to identify the rule causing the block and configure an exclusion without compromising tenant security.
- **Action**: I reviewed the local Windows Defender Operational logs (Event ID 1121) on the developer's machine to identify the failing file path and the specific ASR rule ID. I updated the Intune ASR policy settings, adding the developer's project directory path to the ASR Exclusions list.
- **Result**: The developer was able to run their local testing scripts, while the ASR rule remained active for the rest of the file system.

**Q3: What is the benefit of deploying Microsoft Security Baselines over creating individual configuration profiles from scratch?**
A: Security Baselines provide pre-configured, tested groups of security settings recommended by Microsoft security experts. They ensure compliance with industry standards with minimal configuration overhead. Creating individual profiles from scratch is more time-consuming, requires manual mapping of hundreds of settings, and increases the risk of configuration errors.

---

---
## Seedha Simple Mein
*Seedha simple mein: Endpoint Security blade ke zariye aap device ke local windows defender settings, firewall rules, ASR rules, aur BitLocker encryption ko direct cloud se lock down karte hain.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Establishes identity joins and console controls.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Details compliance health checks.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details device configuration templates.
