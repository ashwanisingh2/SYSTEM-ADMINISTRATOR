---
tags: [sysadmin, windows-server, introduction, OS]
difficulty: Beginner
lab-required: Yes
read-time: 12 mins
---

# WS-01: Windows Server 2022 Introduction

> [!abstract] Overview
> This note introduces Windows Server 2022, detailing licensing editions, deployment profiles (Core vs. Desktop Experience), initial configuration baselines, and Windows Admin Center management.

---
## Concept
Think of Windows 10/11 (Client OS) as a personal family car. It is designed for one driver, has comfort features (apps, games, media players), and is great for running errands. 

Windows Server is a heavy-duty commercial cargo truck. It doesn't have fancy gaming seats, but it is built to run 24/7, haul massive payloads of data, support thousands of passengers (users) simultaneously, and act as a mobile power station (DHCP, AD, DNS) for other vehicles. 

You can configure it with a dashboard (Desktop Experience GUI) or strip it down to just the engine block to save fuel and maximize cargo capacity (Server Core CLI).

*Seedha simple mein: Windows Server enterprise networks ko manage karne ke liye design kiya gaya OS hai. Iske do versions hote hain - Desktop Experience (GUI) aur Server Core (CLI). Standard and Datacenter editions virtualization capacity ke basis par select hote hain.*

---
## Technical Deep Dive

### 1. Windows Server 2022 Editions
Microsoft licenses Windows Server 2022 based on physical CPU cores (minimum 16 core licenses per physical server).

| Edition | Virtualization Rights | Core Features | Best Use Case |
|---|---|---|---|
| **Essentials** | None (Physical install only) | Pre-configured limit: max 25 users and 50 devices. No CALs required. | Very small businesses under 25 employees |
| **Standard** | **2 Virtual Machines** (plus 1 physical host reserved for Hyper-V management) | OSE/Hyper-V containers, standard storage options. Requires Client Access Licenses (CALs). | Physical servers or low-density virtual environments |
| **Datacenter** | **Unlimited Virtual Machines** | Software-defined networking, Storage Spaces Direct (S2D), Shielded VMs. Requires CALs. | Highly virtualized hybrid cloud data centers |

### 2. Server Core vs. Desktop Experience
During installation, you must choose between two interface profiles:
- **Server Core (Recommended):**
  - **Interface:** Command line only (PowerShell and `cmd`).
  - **Advantages:** $50\%$ smaller disk footprint, lower RAM usage, reduced attack surface (fewer components for hackers to exploit), and fewer reboots because there are fewer patches.
- **Server Experience (Desktop Experience):**
  - **Interface:** Standard Windows GUI interface with Server Manager.
  - **Use Case:** Required for legacy applications that demand a graphic interface, or when administrators lack CLI experience.

### 3. Server Manager Overview
Server Manager is the centralized dashboard in the Desktop Experience. It allows administrators to:
- Monitor local and remote server statuses.
- Install and remove Roles (e.g., AD DS, DNS, IIS) and Features (e.g., BitLocker, Failover Clustering).
- View Event Viewer logs consolidated across multiple managed servers.

### 4. Windows Admin Center (WAC)
WAC is Microsoft's modern, browser-based management tool that replaces legacy MMC snap-ins.
- **Deployment:** Installed on a gateway server, allowing administrators to manage all Windows Server Core or GUI hosts from any web browser.
- **Capabilities:** Direct registry editing, file explorer, certificate management, terminal access, and performance monitoring without needing RDP.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Hyper-V host or VMware Workstation, and a Windows Server 2022 ISO file.

### Step 1: VM Provisioning
1. Open your hypervisor (e.g., Hyper-V Manager).
2. Create a new Generation 2 Virtual Machine:
   - Name: `WS2022_LAB_01`
   - RAM: 2048 MB (disable dynamic memory during setup).
   - Network: Connect to an Internal Virtual Switch.
   - Virtual Disk: 60 GB.
3. Attach the Windows Server 2022 ISO to the virtual DVD drive.

### Step 2: Operating System Installation
1. Start the VM and boot from the ISO.
2. Select Language, Time, and Keyboard layout. Click **Install Now**.
3. Select **Windows Server 2022 Standard (Desktop Experience)**. Click Next.
4. Accept license terms. Select **Custom: Install Windows only (advanced)**.
5. Select the 60 GB unallocated drive space. Click Next.
6. Once installation completes, set the built-in local administrator password (e.g., `P@ssword123!`).

### Step 3: Initial Configuration Baseline
After logging in, Server Manager launches automatically.
1. Click on **Local Server** in the left panel.
2. **Rename Host:** Click the Computer Name link. Change it to `SVR-DC01`. Click OK and defer reboot.
3. **Configure Static IP Address:**
   - Click the Ethernet link. Right-click the adapter -> Properties -> IPv4.
   - Assign static IP: `192.168.10.10`, Subnet: `255.255.255.0`, Gateway: `192.168.10.1`, DNS: `127.0.0.1` (since this host will become the DNS server). Click OK.
4. **Disable IE Enhanced Security Configuration (ESC):** Click the link and turn it **Off** for Administrators (allows internet browsing for updates download).
5. **Reboot the system** to apply hostname changes.

---
## Commands Reference
Common Server Core configuration commands:

```cmd
:: Launch Server Configuration Menu (Server Core environment tool)
sconfig

:: Windows Command Prompt
:: Check basic OS build details and license activation state
slmgr.vbs /dli

# Windows PowerShell
# Rename computer and force a restart
Rename-Computer -NewName "SVR-DC01" -Force -Restart

# Install the IIS Web Server role on the local server
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

---
## Troubleshooting Scenarios

### Scenario 1: Windows Server Core Remote Management & RDP Connectivity
**Ticket:** "Administrator cannot access the GUI or manage newly installed Windows Server Core remotely. Remote Desktop (RDP) connections fail."
**L1 Resolution:** Connect to the Server Core console. Run `sconfig` from the CLI. Select Option **7** (Remote Desktop) and type **E** to enable RDP, choosing secure network-level authentication. Select Option **4** (Configure Remote Management) and enable it.
**Escalation Trigger:** If remote management ports remain blocked by active firewall profiles or domain policies.
**L2 Resolution:** Open PowerShell as Administrator on Server Core. Run: `Set-NetFirewallRule -DisplayGroup "Remote Desktop" -Enabled True` and `Set-NetFirewallRule -DisplayGroup "Remote Administration" -Enabled True`. Target the Server Core IP from a remote Server Manager console on a management workstation to verify access.

### Scenario 2: Active Directory Role Installation Failure due to Hostname
**Ticket:** "Active Directory Domain Services role installation wizard fails with default hostname error."
**L1 Resolution:** Rename the computer to a standardized server name (e.g. `SVR-DC01`) and restart using PowerShell: `Rename-Computer -NewName "SVR-DC01" -Force -Restart`.
**Escalation Trigger:** If the computer rename fails due to existing Domain Controller registry settings, system configuration conflicts, or access restrictions.
**L2 Resolution:** Review active role installations. Clean up previous failed AD installations using `Uninstall-ADDSDomainController` if necessary, clear local AD registry traces, and execute host renaming and domain promotion after verifying local DNS settings.

---
## Common Mistakes
> [!warning] Avoid These
> **Using Datacenter licenses on low-VM servers:** Purchasing Datacenter edition for a host that will only run two virtual machines. This wastes thousands of dollars in licensing costs without any operational benefit.
> **Correct approach:** Use Standard edition for low-density hosts (2 VMs or fewer) and Datacenter edition for high-density hosts (3+ VMs).

---
## Pro Tips
> [!tip] Field Experience
> When deploy servers in enterprise datacenters, always default to **Server Core** deployment. It limits security patching cycles by up to $60\%$, meaning you do not have to reboot critical production infrastructure hosts nearly as often compared to GUI deployments.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Standard Edition | Standard server license supporting up to two virtual machines per host licensing block. |
| 2 | Datacenter Edition| Premium server license allowing unlimited VMs on a licensed physical server. |
| 3 | Server Core | Lightweight, CLI-only Windows Server installation profile with lower patch requirements. |
| 4 | sconfig | The interactive command-line tool used to configure basic parameters on Server Core. |
| 5 | Windows Admin Center| Modern, web-browser console that manages local and remote Windows Servers. |

---
## Interview Q&A

**Q1: Explain how Windows Server 2022 is licensed under standard core-based licensing rules.**
A: Windows Server 2022 Standard and Datacenter editions are licensed based on physical CPU cores. 
1. All physical cores in the server must be licensed.
2. A minimum of 16 core licenses is required for any physical server.
3. A minimum of 8 core licenses is required per physical processor (socket).
4. If a standard edition server needs to run more than 2 VMs, you must license all physical cores again (called "stacking" licenses).

**Q2: A company is upgrading its servers to Windows Server 2022. The security team insists on minimizing the attack surface of the servers. Explain how you would implement this.**
A: 
- **Situation:** The client requires maximum OS security hardening and minimal attack footprint.
- **Task:** Recommend and configure the appropriate server profile.
- **Action:** I will recommend deploying **Windows Server Core** for all infrastructural roles (AD, DNS, DHCP, File Services). Server Core lacks the GUI shell, desktop APIs, and standard media tools, which reduces the active system binaries and attack vectors. I will also enable Windows Defender and configure restricted remote management parameters.
- **Result:** Deploying Server Core meets security compliance by eliminating unused interface software and reducing the necessary monthly security patching restarts.

**Q3: What are CALs, and what is the difference between a User CAL and a Device CAL?**
A: **CALs (Client Access Licenses)** are licenses required for users or devices to connect to Windows Server services (like file sharing, printing, or Active Directory). A **User CAL** licenses an individual person, allowing them to access server resources from any number of devices (phone, laptop, desktop). It is best for companies with employees who use multiple devices. A **Device CAL** licenses a physical device, allowing any number of users to access server resources from that specific machine. It is best for shift-based companies (e.g., call centers, nurses' stations) where multiple workers share the same computer.

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — Installing directory roles on Server 2022.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-03 DNS Server — Install and Configure|WS-03 DNS Server — Install and Configure]] — Configuring base server name resolution.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-12 Hyper-V Virtualization|WS-12 Hyper-V Virtualization]] — Building and managing server VMs.

