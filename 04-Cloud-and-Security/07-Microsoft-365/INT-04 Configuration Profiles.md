---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-04-configuration-profiles, int-04]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# INT-04: Configuration Profiles

> [!abstract] Overview
> This note covers Intune Configuration Profiles, detailing the settings catalog, device restrictions, connection configurations (Wi-Fi/VPN), policy assignments using filters, and conflict resolution rules.

---

---
## Concept Overview
Think of Configuration Profiles as the remote control settings for your organization's devices. Instead of walking up to every machine to disable USB ports, type in Wi-Fi passwords, configure the desktop background, or set up VPN tunnels, you build a configuration blueprint in the cloud. The moment a device turns on and connects to the internet, it pulls down this blueprint and configures itself.

---

---
## Technical Deep Dive
### 1. Configuration Profile Types
- **Settings Catalog**: The modern, search-first interface. Contains all available settings across platforms in one search index, replacing legacy template models.
- **Templates**: Legacy grouped profiles focusing on specific areas (e.g., Device Restrictions, VPN, Wi-Fi, Endpoint Protection).
- **Administrative Templates (ADMX)**: Ported Windows Group Policies (GPOs) available directly in the cloud.
- **Custom Profiles**: Uses OMA-URI (Open Mobile Alliance Uniform Resource Identifier) paths to configure settings not yet exposed in the standard GUI.

### 2. Core Configurations

#### Removable Storage (USB Block)
Blocks write access or completely disables USB flash drives to prevent data leakage and malware ingestion.
- OMA-URI: `./Device/Vendor/MSFT/Policy/Config/Storage/RemovableDiskDenyWriteAccess`

#### Wi-Fi & VPN Profiles
Deploys pre-configured network access parameters. Wi-Fi profiles can distribute WPA2-Enterprise credentials or certificates. VPN profiles configure connection settings (SSTP, IKEv2) and trigger auto-connect behaviors.

#### Update Rings & Windows Update settings
Controls deferral periods for quality updates (security patches) and feature updates, and defines active hours to prevent reboot interruptions.

### 3. Assignments: Groups vs. Filters
- **Groups**: Standard Entra ID security groups (User or Device). Pushing policies to large groups can be slow, and dynamic evaluation takes time.
- **Filters**: High-speed evaluation rules applied *on top* of groups. For example, assign a policy to "All Devices" but apply a filter targeting only Windows 11 machines.

### 4. Conflict Resolution Rules
When multiple profiles containing conflicting settings are assigned to the same user or device:
- **Compliance Policy vs. Configuration Profile**: Compliance settings *always* win over configuration profile settings.
- **Conflict between two Configuration Profiles**: The setting status changes to **Conflict**. The device ignores the setting, retaining its existing configuration until the administrator resolves the conflict.
- **Conflict between two different values**: The setting is marked as **Conflict**, and neither value is applied.

---

## Common Mistakes
> [!warning] Avoid These
> Mixing different settings configurations for the same feature across Templates, Administrative Templates, and the Settings Catalog. This makes troubleshooting conflicts difficult. Use the Settings Catalog as your primary configuration interface.
> Deploying VPN or Wi-Fi configuration profiles to "All Users" without assigning the required root certificates first, causing connection handshakes to fail.
> Exposing internal URLs for wallpaper or lock screen assets that remote devices cannot resolve without an active VPN connection. Use public-facing, secure URLs (`https://`).

---

## Pro Tips
> [!tip] Field Experience
> Use **Intune Filters** to refine policy targeting. Filters are evaluated in real-time before policies are evaluated, reducing enrollment delay and avoiding policy application errors.
> When converting GPOs to Intune, use the **Group Policy analytics** tool in the Intune console. Import your exported GPO XML files, and the tool will show you which settings have direct matches in the cloud and which require OMA-URI workarounds.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Microsoft Intune Admin Center with Intune Administrator permissions, and a test Windows 11 client device.

### Step 1: Create a Device Restriction Profile via Settings Catalog
Create a profile that blocks removable storage write access and customizes the lock screen:
1. Open the **Microsoft Intune Admin Center** -> `Devices` -> `Configuration`.
2. Click **Create** -> **New policy**.
3. Platform: **Windows 10 and later** -> Profile type: **Settings catalog** -> Click **Create**.
4. Name: `Corp Windows Configuration Profile`.
5. Under **Configuration settings**, click **Add settings**.
6. Search for `Removable Storage` -> Select **Administrative Templates\System\Removable Storage Access**.
7. Check **Removable Disks: Deny write access** -> Close search window.
8. Toggle the setting to **Enabled**.
9. Click **Add settings** again -> Search for `Lock Screen` -> Select **Personalization**.
10. Check **Lock Screen Image** -> Enter a public URL hosting your corporate lock screen wallpaper.

### Step 2: Configure Assignments and Filters
1. Click **Next** to proceed to Assignments.
2. Under **Included groups**, click **Add all devices**.
3. Under **Filter**, click **Filter evaluation** -> Select **Exclude** -> Select a filter targeting personal/BYOD devices (ensuring corporate configurations only apply to company hardware).
4. Click **Next** -> Click **Create**.

### Step 3: Verify Profile Status on Client
1. Log into your test Windows 11 device.
2. Insert a USB flash drive and try to copy a file onto it.
3. Windows will display an **Access Denied** message, confirming the policy is active.
4. Check the sync status in **Settings** -> **Accounts** -> **Access work or school** -> Select the account -> **Info** -> Scroll down to see applied policies.

---

---
## Cheat Sheet / Quick Reference
```powershell
# Read Intune MDM diagnostic logs in XML format on client
Get-Content -Path "C:\Users\Public\Documents\MDMDiagnostics\MDMDiagReport.xml"
# Check registry path where Intune writes applied MDM policy values
Get-ChildItem -Path "HKLM:\SOFTWARE\Microsoft\PolicyManager\current\device\"
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Settings Catalog | Searchable index containing all available Intune policy settings in one place. |
| 2 | OMA-URI | Custom path used to configure OS settings not yet exposed in the standard GUI. |
| 3 | Conflict Status | Error state triggered when two policies try to set different values for the same setting. |
| 4 | Intune Filters | High-speed evaluation rules used to target policies based on device properties. |
| 5 | GPO Analytics | Import tool that evaluates local GPOs for compatibility with cloud management. |

---

---
## Troubleshooting
**Scenario 1:**
- Problem: An administrator deploys a new Wi-Fi profile with corporate credentials, but the status dashboard in the Intune portal shows all target devices are stuck in a "Pending" state.
- Root Cause: Configuration Profiles only apply when the device checks in. If the device is offline or cannot connect to the internet, it remains pending.
- Fix:
  1. Verify the client device has an active internet connection (via ethernet or an alternative network).
  2. Force a manual sync on the device using the Company Portal app or from the Intune portal.
  3. Check the **Diagnostics** report under `Devices` -> `Configuration` -> Select the profile -> `Device status` to see if there are any specific errors.

**Scenario 2:**
- Problem: An administrator configures a lock screen wallpaper policy, but the target devices report a "Conflict" status in the Intune dashboard, and the wallpaper does not apply.
- Root Cause: Another configuration profile or an existing on-premises Group Policy (GPO) is attempting to set the lock screen wallpaper to a different value.
- Fix:
  1. Open the **Intune Admin Center** -> `Devices` -> `Configuration` -> Select the conflicting profile.
  2. Click on the **Conflict** count to see the list of conflicting policies and settings.
  3. Identify the other policy modifying the lock screen setting.
  4. Exclude the target group from one of the policies, or merge the settings into a single configuration profile.

---

---
## Interview Questions
**Q1: How does Intune resolve conflicts when two different configuration profiles apply to the same device with different values for the same setting?**
A: When two configuration profiles apply conflicting values for the same setting to a device, the status for that setting changes to "Conflict". The device does not apply either value, ignoring the setting and leaving it at its current OS state. The administrator must locate the conflicting profiles using the Device Configuration dashboard and edit the assignments to resolve the issue.

**Q2: A client wants to deploy a corporate VPN profile that automatically connects when users open their browser. How would you configure this?**
A:
- **Situation**: The client needed a VPN profile deployed to 300 Windows laptops that connected automatically when specific apps opened.
- **Task**: I needed to configure an auto-trigger VPN connection profile in Intune.
- **Action**: I created a VPN Configuration Profile using the Settings Catalog. I defined the server endpoints, selected the authentication method (Certificates), and enabled the **Always On** option. I then configured **App-triggered VPN** rules, listing the executable paths of the corporate web browsers.
- **Result**: The VPN profile deployed successfully. Whenever users launched their browser, the VPN connection initiated automatically without requiring manual user input.

**Q3: Why is the Settings Catalog preferred over legacy templates for creating new configuration profiles?**
A: The Settings Catalog is preferred because it contains all available settings across platforms in a single, searchable index. It is updated directly by Microsoft as new OS settings are exposed, making it more comprehensive than legacy templates. It also allows administrators to build custom, unified profiles rather than managing multiple legacy templates.

---

---
## Seedha Simple Mein
*Seedha simple mein: Configuration Profiles ke zariye aap devices ki OS settings remote control karte ho, jaise USB block karna, Wi-Fi profile push karna ya updates control karna.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Sets up the basic management authority and console layout.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-03 Compliance Policies|INT-03 Compliance Policies]] — Defines health states that override configuration profiles.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-05 Application Management|INT-05 Application Management]] — Details application deployment profiles.
