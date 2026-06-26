---
tags: [desktop-support, windows-os, deployment, L2]
aliases: [wds-mdt, windows-deployment]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

# Windows Deployment Services (WDS) & Microsoft Deployment Toolkit (MDT)

> [!note] Overview
> This note covers the integration of Windows Deployment Services (WDS) and Microsoft Deployment Toolkit (MDT) to automate enterprise OS deployment. It details the PXE boot process, Lite-Touch Installation (LTI) setup, task sequence logic, driver injection, and troubleshooting network imaging.

---
## Concept Overview
- **What it is** — WDS is a Windows Server role that enables PXE network booting to install operating systems over a local network. MDT is a Microsoft solution accelerator that automates deployment tasks, including driver management, application installations, and OS configurations using Task Sequences.
- **Why it matters** — Individually installing Windows using USB drives on hundreds of company laptops is slow, inconsistent, and prone to errors. WDS & MDT centralize this, allowing support teams to deploy a standardized corporate OS image to dozens of machines simultaneously over the network.
- **Real job encounter** — Staging new hardware batches for employee onboarding, upgrading operating systems from Windows 10 to Windows 11, or rebuilding corrupted client computers.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Connect client systems to the staging network, boot via network card (PXE boot by pressing F12/Fn+F12), select the appropriate task sequence, and monitor progress until completion.
  - **Escalation Trigger**: Escalate if the client fails to obtain an IP via PXE, if the boot image hangs, or if the task sequence errors out during execution.
  - **L2 Resolution**: Import updated drivers and OS patches, create and test deployment task sequences, modify configurations in `CustomSettings.ini`, and rebuild boot images.
  - **L3 Resolution**: Design network integration (DHCP options, IP Helpers, PXE response times), configure multicast sessions to optimize bandwidth, and integrate deployment databases.

---
## Technical Deep Dive

### 1. The PXE Network Boot Process
Preboot Execution Environment (PXE) uses DHCP and TFTP to load a bootable environment over the network:
1. **DHCP Discover**: Client broadcasts a discover request containing PXE client identifiers.
2. **DHCP Offer**: DHCP server provides an IP configuration along with **Option 66** (TFTP boot server IP) and **Option 67** (boot file path, e.g., `boot\x64\wdsnbp.com`).
3. **TFTP Transfer**: Client contacts the WDS server via TFTP and downloads the boot file.
4. **Boot Execution**: The client runs the boot loader, downloads the Windows PE boot image (`LiteTouchPE_x64.wim`), and boots into memory.

```
+--------+                    +-------------+                     +------------+
| Client |                    | DHCP Server |                     | WDS Server |
+--------+                    +-------------+                     +------------+
    |                               |                                   |
    |---- 1. DHCP Discover -------->|                                   |
    |<--- 2. DHCP Offer ------------|                                   |
    |      (IP, Option 66 & 67)     |                                   |
    |                                                                   |
    |---- 3. TFTP Request (wdsnbp.com) -------------------------------->|
    |<--- 4. TFTP Send (Bootloader files) ------------------------------|
    |                                                                   |
    |---- 5. TFTP Request (LiteTouchPE_x64.wim) ------------------------>|
    |<--- 6. TFTP Send (WinPE Boot Image) ------------------------------|
    |                                                                   |
    |==== 7. Boots into Windows PE and starts MDT Wizard ===============|
```

### 2. WDS vs. MDT: Split of Work
- **WDS (The Transport Layer)**: Acts strictly as the PXE server, hosting the network bootloader and broadcasting WIM images using unicast or multicast.
- **MDT (The Logic Engine)**: Handles file organization. It runs the Lite-Touch script framework, stores the operating system files, apps, and drivers, and guides the client through installation steps using **Task Sequences**.

### 3. Key Deployment Settings: Rules and Bootstrap
MDT behavior is defined in two key INI configuration files inside the deployment share:
- **`Bootstrap.ini`**: Controls the initial environment load before connection. Typically configures the path to the deployment share and default access credentials.
- **`CustomSettings.ini`**: Defines wizard screen skips, computer naming conventions, domain join details, administrative accounts, and application selections.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server 2022 VM (`SVR-WDS01`) joined to `corp.local`.
> - AD DS and a DHCP server configured with scope options.
> - Windows ADK (Assessment and Deployment Kit) and WinPE Add-on installed on the server.
> - Microsoft Deployment Toolkit (MDT) installed.

### Step 1: Install the Windows Deployment Services Role
1. Log in to `SVR-WDS01`. Open PowerShell as Administrator and execute:
```powershell
# Install WDS and administrative consoles
Install-WindowsFeature -Name WDS -IncludeManagementTools
```
**Expected Output:** `Success : True` and `RestartNeeded : No`.

### Step 2: Initialize WDS Server Configuration
1. Open the WDS console (`wdsmgmt.msc`).
2. Right-click the server name and click **Configure Server**.
3. Choose **Active Directory Integrated**.
4. Set the Remote Installation folder path to `E:\RemoteInstall` (avoid installing on the system drive `C:`).
5. PXE Server Initial Settings: Select **Respond to all client computers (known and unknown)**.
6. Under DHCP Option 60: Check both boxes if DHCP is installed on the same server, allowing WDS to share ports. Click **Finish**.

### Step 3: Create and Configure MDT Deployment Share
1. Open Deployment Workbench (`DeploymentWorkbench.msc`).
2. Right-click **Deployment Shares** and select **New Deployment Share**.
3. Set path to `C:\DeploymentShare`. Keep default share name `DeploymentShare$`.
4. Clear all wizard defaults to enforce silent installation rules later. Click **Finish**.
5. Right-click the new deployment share, select **Properties**, and click the **Rules** tab.
6. Edit the configurations as follows:
```ini
[Settings]
Priority=Default
Properties=MyCustomVar

[Default]
OSInstall=Y
SkipCapture=YES
SkipAdminPassword=YES
AdminPassword=SecurePassword123!
SkipProductKey=YES
SkipComputerName=NO
SkipDomainMembership=YES
JoinDomain=corp.local
DomainAdmin=Administrator
DomainAdminDomain=corp.local
DomainAdminPassword=DomainSecurePassword123!
```
7. Click the **Edit Bootstrap.ini** button and define credentials:
```ini
[Settings]
Priority=Default

[Default]
DeployRoot=\\SVR-WDS01\DeploymentShare$
UserDomain=corp.local
UserID=Administrator
UserPassword=DomainSecurePassword123!
SkipBDDWelcome=YES
```
8. Save files and click **Apply**.

### Step 4: Import Operating System and Create Task Sequence
1. In Deployment Workbench, right-click **Operating Systems** -> **Import Operating System**.
2. Select **Full set of source files**, navigate to your mounted Windows 11 ISO drive, and complete import.
3. Right-click **Task Sequences** -> **New Task Sequence**.
4. Define settings:
   - ID: `WIN11-PROD`
   - Name: `Deploy Windows 11 Enterprise`
   - Template: **Standard Client Task Sequence**
   - OS: Select the imported Windows 11 image.
5. Complete the wizard.

### Step 5: Update Deployment Share and Link WDS
1. Right-click the MDT deployment share and select **Update Deployment Share**. Select **Optimize the boot image updating process**.
2. This generates the Boot WIM files (`LiteTouchPE_x64.wim`) inside `C:\DeploymentShare\Boot\`.
3. Open WDS console. Right-click **Boot Images** -> **Add Boot Image**.
4. Browse to `C:\DeploymentShare\Boot\LiteTouchPE_x64.wim`. Name it `MDT LiteTouch x64`. Click **Finish**.
5. Connect a target virtual machine to the same network interface, turn it on, boot via PXE, and verify that the MDT selection menu loads automatically.

---
## Cheat Sheet / Quick Reference

| Command / Setting | Purpose | Example |
|---|---|---|
| `wdsutil /get-Server` | Queries configuration parameters of the local WDS server | `wdsutil /get-Server /Show:All` |
| `wdsutil /start-server` | Starts the WDS service | `wdsutil /start-server` |
| `wdsutil /stop-server` | Stops the WDS service | `wdsutil /stop-server` |
| `dism /Get-WimInfo` | Inspects index configurations inside WIM files | `dism /Get-WimInfo /WimFile:C:\image.wim` |
| `dism /Mount-Image` | Mounts a WIM file to a local folder for offline servicing | `dism /Mount-Image /WimFile:C:\boot.wim /Index:1 /MountDir:C:\mount` |
| `dism /Unmount-Image` | Unmounts a image saving all modified changes | `dism /Unmount-Image /MountDir:C:\mount /Commit` |
| **DHCP Option 66** | DHCP option containing WDS Boot Server Hostname/IP | `192.168.10.15` |
| **DHCP Option 67** | DHCP option containing the bootloader file path | `boot\x64\wdsnbp.com` |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Client fails to boot PXE: "PXE-E32: TFTP open timeout." | Port UDP 69 (TFTP) is blocked by firewall, or WDS service is stopped. | Run `wdsutil /start-server`. Verify that the Windows Defender Firewall allows inbound WDS/TFTP traffic. |
| Client PXE-boots, but returns: "PXE-E53: No boot filename received." | DHCP options 66 and 67 are missing or misconfigured. | Open the DHCP server console. Set Option 66 to the WDS server's IP, and set Option 67 to `boot\x64\wdsnbp.com` (for legacy BIOS) or `boot\x64\wdsmgmt.efi` (for UEFI). |
| MDT deployment starts, but fails with error: "A connection to the deployment share could not be made." | WinPE environment lacks correct network drivers for the client NIC, or SMB access credentials are incorrect. | Check network card interface link. Re-verify credentials in `Bootstrap.ini`. Import the client's network driver into MDT's "Out-of-box Drivers" folder and rebuild the boot image. |
| MDT wizard hangs on loading: "No storage device detected." | WinPE lacks the RAID/SATA controllers drivers for the client motherboard. | Import matching storage driver (Intel RST, etc.) into the boot selection profile and update deployment share. |
| Task sequence fails at the end: "Unable to join domain corp.local." | DNS IP was not assigned via DHCP, or domain-join credentials are incorrect. | Ensure DHCP is distributing the local Domain Controller's IP as DNS server option 006. Verify access credentials in `CustomSettings.ini`. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What keyboard key do you press on a target client machine to boot into a network installation, and what is this boot protocol called?
> **A:** On most enterprise computers, you press the **F12** key (or **Fn+F12**) during POST to access the network boot interface. This network booting protocol is called **PXE** (Preboot Execution Environment), which enables client computers to fetch boot files over the network without a local OS or storage medium.

> [!question] L2 Question
> **Q:** What is the difference between `Bootstrap.ini` and `CustomSettings.ini` in Microsoft Deployment Toolkit (MDT)?
> **A:** `Bootstrap.ini` is processed first, before the client connects to the deployment share. It contains settings required to establish the connection, such as the UNC path to the deployment share and the network credentials. `CustomSettings.ini` is processed after the deployment share is mapped. It controls the behavior of the deployment wizard screens (skipping pages, setting computer names, defining domain join configurations, and scripting app installations).

> [!question] L3/Scenario Question
> **Q:** You are configuring a deployment lab to image 100 laptops simultaneously. When you launch the deployment, the network speed drops, and TFTP timeouts occur. How do you resolve this?
> **A:** 
> - **Situation:** Simultaneous imaging of 100 laptops degrades network bandwidth and crashes TFTP transfers.
> - **Task:** Optimize network infrastructure to handle high-volume image streaming.
> - **Action:** 
>   1. **Implement Multicast**: Instead of sending 100 separate unicast streams (which multiplies traffic), I will open WDS and configure a **Multicast Transmission** namespace. This allows the WDS server to broadcast the image file once, and all client devices listen to that single broadcast stream.
>   2. **Configure IGMP Snooping**: Enable IGMP Snooping on the network switches to ensure multicast traffic only goes to the ports where PXE clients are connected, preventing network saturation.
>   3. **Adjust TFTP Window Size**: Modify the WDS Server properties under the TFTP tab. Increase the **Maximum Block Size** and enable **Variable Window Extension** to optimize file transfer speeds.
> - **Result:** Multicast streaming reduces network overhead by up to 90%, preventing timeouts and allowing stable imaging of all 100 machines.

---
## Seedha Simple Mein
*Seedha simple mein: WDS aur MDT ka integration corporate systems mein automatically Windows install karne ke liye kiya jata hai. WDS computer ko network ke zariye boot (PXE boot) karne me madad karta hai, aur MDT Windows installation settings (drivers, configuration, apps) ko manage karta hai, jisse support engineers ko manual process nahi karna padta.*

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-04 DHCP Server — Install and Configure|WS-04 DHCP Server — Install and Configure]] — Configuring PXE scope options.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — Computer domain registration processes.
- [[05-Automation-and-Ticketing/12-Ansible/ANS-01 Ansible for Windows Admins|ANS-01 Ansible for Windows Admins]] — Post-deployment configuration options.

---
*Tags: #desktop-support #windows-os #deployment #L2 #cert-md-102*
