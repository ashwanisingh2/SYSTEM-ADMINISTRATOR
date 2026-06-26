---
tags: [desktop-support, m365, collaboration, L2]
aliases: [int-08-windows-update-management, int-08]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #md-102
---

# INT-08: Windows Update Management

> [!abstract] Overview
> This note covers Windows Update Management using Microsoft Intune, detailing Windows Update for Business (WUfB) concepts, update rings, deployment schedules, driver updates, and update troubleshooting.

---

---
## Concept Overview
Think of Windows Update Management as water filtration at a corporate campus. You don't want to pipe raw, untreated water (newly released updates) directly to everyone's desks immediately, as it might contain contaminants (bugs). Instead, you set up filter reservoirs (Update Rings).
- **Ring 1 (IT / Testing)** gets the water first. They drink it and make sure it's safe.
- **Ring 2 (Pilot)** gets it 3 days later (Deferral Period) to test it on representative departments.
- **Ring 3 (General Production)** gets it 10 days later once it is proven clean.
- If anyone reports a stomach ache (a critical bug), you pull the emergency lever (Pause Updates) to stop the flow of water across all rings.

---

---
## Technical Deep Dive
### 1. Windows Update for Business (WUfB) Concepts
WUfB is a cloud-based service that allows administrators to manage when and how Windows devices are updated:
- **Quality Updates (Patches)**: Cumulative security updates released monthly on the second Tuesday ("Patch Tuesday"). Typically deferred by 0 to 30 days.
- **Feature Updates (Upgrades)**: Annual major upgrades (e.g., Windows 10 to Windows 11, or 23H2 to 24H2). Deferred by 0 to 365 days.
- **Diagnostics data**: Required for WUfB reporting. Devices must send basic diagnostic data to Microsoft for update status to appear in reports.

### 2. Update Ring Settings Walkthrough
Key configuration parameters for Update Rings include:
- **Deferral Period**: The number of days to wait before applying a newly released update.
- **User Experience Settings**:
  - **Active Hours**: Time periods during which reboots are blocked (e.g., 8:00 AM to 5:00 PM).
  - **Auto-reboot behavior**: Forces reboots or prompts the user with reminders.
  - **Deadline settings**: The number of days a user can delay a reboot before the system forces a reboot to apply the update (typically 3 days for quality updates, 7 days for feature updates).

### 3. Feature Update Policies & Driver Management
- **Feature Updates Policy**: Locks a device to a specific Windows OS version (e.g., Windows 11 23H2). The device will install quality patches but will not upgrade to a newer OS version until this policy is updated.
- **Driver Update Utility**: Allows administrators to review, approve, suspend, or reject device drivers (Intel, HP, Dell, etc.) released by hardware manufacturers through Windows Update.

---

## Common Mistakes
> [!warning] Avoid These
> Setting a quality update deadline to `0` days. This forces immediate reboots on client machines while users are working, resulting in data loss and disruption.
> Overlapping multiple Update Rings on the same device. If a device belongs to both a test group and a production group, conflict resolution rules may cause it to fall back to default Windows Update behavior.
> Neglecting to configure active hours, which allows updates to install and reboot machines during critical business presentations or meetings.

---

## Pro Tips
> [!tip] Field Experience
> Implement **Grace Periods** alongside deadlines. A deadline forces an update install after a set period, while the grace period gives users who have been offline (e.g., on vacation) a buffer before the reboot forces, avoiding immediate disruption upon login.
> Use **Driver Updates** policies in "Approval" mode. This allows you to test network adapter or graphics drivers on a test group before approving them for the rest of the company, preventing network dropouts.

---

---
## Step-by-Step Lab
> [!info] Lab Setup Needed
> Access to Microsoft Intune Admin Center with Intune Administrator permissions. Enrolled test Windows 11 client devices grouped into an IT testing security group and a Production security group.

### Step 1: Create the IT Testing Update Ring (Fast Ring)
1. Open the **Microsoft Intune Admin Center** -> `Devices` -> `Manage updates` -> `Windows updates` -> `Windows 10 and later update rings`.
2. Click **Create update ring**.
3. Name: `IT Testing - Fast Update Ring`.
4. Under **Update ring settings**:
   - Set **Microsoft product updates** to **Allow**.
   - Set **Windows driver updates** to **Allow**.
   - Set **Quality update deferral period (days)** to `0`.
   - Set **Feature update deferral period (days)** to `0`.
   - Under **User experience settings**:
     - Set **Automatic update behavior** to **Auto install at maintenance time**.
     - Set **Active hours start** to `8 AM` -> **Active hours end** to `5 PM`.
     - Set **Option to pause Windows updates** to **Enable**.
5. Click **Next** -> Assign the policy to the `IT Testing Devices` group -> Click **Create**.

### Step 2: Create the General Production Update Ring (Slow Ring)
1. Click **Create update ring**.
2. Name: `Production - Slow Update Ring`.
3. Under **Update ring settings**:
   - Set **Quality update deferral period (days)** to `7` (waits 7 days after release before patching).
   - Set **Feature update deferral period (days)** to `30`.
   - Under **User experience settings**:
     - Set **Automatic update behavior** to **Auto install and restart at maintenance time**.
     - Set **Quality update deadline (days)** to `3` -> Set **Grace period (days)** to `1`.
5. Click **Next** -> Assign to the `All Corporate Devices` group (ensuring you exclude the IT Testing Devices group to avoid conflicts) -> Click **Create**.

### Step 3: Configure Feature Update Locks
Ensure devices do not upgrade to Windows 11 24H2 before approval:
1. Navigate to the **Feature updates** tab.
2. Click **Create profile** -> Name: `Lock Windows 11 23H2`.
3. Feature update to deploy: Select **Windows 11, version 23H2**.
4. Assign to all devices and click **Create**.

### Step 4: Verify Local Client Registry Settings
On a client machine after policies apply, verify that Windows Update is managed by your organization:
1. Open Windows **Settings** -> Click **Windows Update**.
2. A message at the top should read: `*Some settings are managed by your organization`.
3. Open the registry and verify the update values are populated under the WUfB path.

---

---
## Cheat Sheet / Quick Reference
```powershell
# Get client local update settings registry configuration
Get-ChildItem -Path "HKLM:\SOFTWARE\Microsoft\PolicyManager\current\device\Update"
# Force local client machine to search for updates immediately
usoclient StartScan
# Force local client to initiate update installations
usoclient StartInstall
```

---
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Quality Update | Monthly cumulative security patches deployed to fix OS bugs and vulnerabilities. |
| 2 | Feature Update | Annual major OS version upgrades (e.g., upgrading from 22H2 to 23H2). |
| 3 | Deferral Period | The number of days to hold back an update release before deploying it to devices. |
| 4 | Deadline | The time limit before Intune forces a device to reboot and apply pending updates. |
| 5 | Pause | Administrative action that halts update distribution for up to 35 days during incidents. |

---

---
## Troubleshooting
**Scenario 1:**
- Problem: An administrator notice that a newly deployed quality update causes several laptops in the Production ring to crash with a Blue Screen of Death (BSOD) after restarting.
- Root Cause: A bug in the monthly Windows cumulative update is incompatible with local client hardware or software drivers.
- Fix:
  1. Open the **Intune Admin Center** -> `Devices` -> `Windows updates` -> `Windows 10 and later update rings`.
  2. Select the **Production - Slow Update Ring** profile.
  3. Click **Pause** at the top menu bar to stop the update deployment for up to 35 days, preventing other devices from installing it.
  4. Identify the target machines that installed the update and trigger a remote command or use local recovery tools to uninstall the update: `wusa /uninstall /kb:KBNumber /quiet /norestart`.

**Scenario 2:**
- Problem: Update compliance reports in the Intune console display all devices as "Not Started" or "No Status," despite devices showing they are up to date locally.
- Root Cause: Windows diagnostic data settings are disabled or blocked by local firewalls/proxies. Without diagnostic data, Microsoft cannot report update status.
- Fix:
  1. Open your active **Configuration Profile** or create a new one using the Settings Catalog.
  2. Locate the **System\Allow Telemetry** setting and set it to **Basic** or **Full**.
  3. Ensure that outbound network connections on local firewalls allow access to the required Microsoft diagnostic endpoints (e.g., `*.telemetry.microsoft.com`).

---

---
## Interview Questions
**Q1: What is Windows Update for Business, and how do you deploy a phased update roll-out in Intune?**
A: Windows Update for Business is a cloud-based service that allows administrators to control update distribution using Microsoft Intune. To configure a phased roll-out, I create multiple Update Rings with varying deferral periods. For example, an IT testing ring has a 0-day deferral, a pilot ring has a 3-day deferral, and a production ring has a 7-to-14-day deferral. This allows the testing team to identify issues before updates deploy to the rest of the organization.

**Q2: A cumulative update causes critical software to fail on several machines. How would you handle this using Intune?**
A:
- **Situation**: A monthly Microsoft quality update broke a critical line-of-business application on production machines.
- **Task**: I needed to halt the update deployment and remove it from affected systems.
- **Action**: I went to the Intune portal, selected the active Update Ring, and clicked the **Pause** option. Next, I wrote a PowerShell script to uninstall the problematic update using `wusa /uninstall /kb:KBNumber /quiet /norestart` and deployed it as a required script to the affected group.
- **Result**: The update deployment was halted, and the update was removed from the affected devices, restoring application functionality.

**Q3: Explain the difference between update deadlines and grace periods in Intune update rings.**
A: The deadline is the number of days a user has before the system forces the installation and reboot for a pending update (e.g., 3 days). The grace period is the number of days granted to a user whose device has been offline and missed the deadline (e.g., returning from vacation) before the system forces the update reboot. This prevents immediate reboots upon turning on a machine.

---

---
## Seedha Simple Mein
*Seedha simple mein: Windows Update Management ke zariye aap devices par automatic updates aur driver patches ko schedule karte hain. Deferral periods aur diagnostic checks se system crashes ko control kiya jata hai.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/INT-01 Microsoft Intune Introduction|INT-01 Microsoft Intune Introduction]] — Sets up the tenant MDM authority.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-04 Configuration Profiles|INT-04 Configuration Profiles]] — Details settings configurations for telemetry and diagnostics.
- [[04-Cloud-and-Security/07-Microsoft-365/INT-07 Endpoint Security in Intune|INT-07 Endpoint Security in Intune]] — Manages Windows Defender updates and security states.
