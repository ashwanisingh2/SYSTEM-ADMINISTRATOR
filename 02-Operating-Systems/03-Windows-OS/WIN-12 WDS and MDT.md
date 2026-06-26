---
tags: [desktop-support, windows-os, deployment, L2]
aliases: [wds-mdt, windows-deployment]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #md-102
---

> [!NOTE|color-blue]
> 🏢 **WINDOWS**

`#complete` `#advanced` `#md-102`

# WIN-12: Windows Deployment Services (WDS) & Microsoft Deployment Toolkit (MDT)

> [!abstract] Overview
> Yeh note Windows Deployment Services (WDS) aur Microsoft Deployment Toolkit (MDT) ke integration ko cover karta hai jo enterprise OS deployment ko automate karta hai. Ek support engineer ko yeh aana chahiye kyunki manually Windows install karna slow aur inconsistent hai, jabki WDS & MDT se network ke zariye bulk imaging ki ja sakti hai.

---
## 🧠 Concept Overview

- **What it is** — WDS ek Windows Server role hai jo PXE network boot enable karta hai. MDT ek logic engine hai jo OS installation, drivers, aur apps ko Task Sequences ke through automate karta hai.
- **Why it matters** — Individually USB drive se Windows install karna hundreds of laptops pe time-consuming hai. WDS & MDT centralized standardized corporate OS image deploy karne mein madad karte hain.
- **Where you see this** — Naye employees ke liye bulk hardware staging, Windows 10 se 11 OS upgrade, ya corrupted client computers ko rebuild karne mein.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Client systems ko network par connect karna, PXE boot (F12) karna, aur task sequence select karke monitor karna. |
| **L2** | Drivers aur OS patches import karna, task sequences banana, `CustomSettings.ini` modify karna, aur boot images rebuild karna. |
| **L3** | Network integration design karna (DHCP options, IP Helpers), multicast sessions configure karna, aur deployment databases integrate karna. |

> [!tip] Seedha Simple Mein
> *WDS aur MDT ka integration corporate systems mein automatically Windows install karne ke liye kiya jata hai. WDS network se boot karwata hai, aur MDT installation manage karta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Restaurant Kitchen** is like **WDS & MDT Deployment** because...
>
> - **WDS** is the Waiter (Transport Layer) who brings the menu (bootloader) to your table.
> - **MDT** is the Chef (Logic Engine) who follows the recipe (Task Sequence) to prepare your customized meal (OS, drivers, apps).

---
## 🔬 Technical Deep Dive

### 1. The PXE Network Boot Process

> [!info] Key Concept
> Preboot Execution Environment (PXE) DHCP aur TFTP use karke network par bootable environment load karta hai.

1. **DHCP Discover**: Client broadcast karta hai PXE request.
2. **DHCP Offer**: DHCP server IP deta hai aur **Option 66** (TFTP boot server IP) & **Option 67** (boot file path e.g. `boot\x64\wdsnbp.com`).
3. **TFTP Transfer**: Client WDS server se TFTP se boot file download karta hai.
4. **Boot Execution**: Boot loader run hota hai aur WinPE image (`LiteTouchPE_x64.wim`) memory mein load hoti hai.

### 2. WDS vs. MDT: Split of Work

- **WDS (The Transport Layer)**: Sirf PXE server ki tarah kaam karta hai, network bootloader aur WIM images (unicast/multicast) host karta hai.
- **MDT (The Logic Engine)**: Lite-Touch script framework run karta hai, files, apps, drivers store karta hai aur **Task Sequences** execute karta hai.

### 3. Key Deployment Settings: Rules and Bootstrap

> [!info] Configuration Files
> MDT behavior do INI files mein define hota hai:

- **`Bootstrap.ini`**: Initial connection setup karta hai (e.g., path to deployment share, access credentials).
- **`CustomSettings.ini`**: Wizard screen skips, computer naming, domain join details, etc. define karta hai.

> [!danger] Common Mistake
> Agar `Bootstrap.ini` mein credentials galat hain, toh client deployment share connect nahi payega aur WinPE mein error dega.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Windows Server 2022 VM (`SVR-WDS01`) joined to `corp.local`.
> - AD DS and a DHCP server configured with scope options.
> - Windows ADK and WinPE Add-on installed.
> - MDT installed.

### Step 1: Install the WDS Role

```powershell
# Install WDS and administrative consoles
Install-WindowsFeature -Name WDS -IncludeManagementTools
```

> [!success] Expected Output
> `Success : True` and `RestartNeeded : No`.

### Step 2: Initialize WDS Server Configuration

1. Open **WDS console** (`wdsmgmt.msc`).
2. Right-click server -> **Configure Server** -> **Active Directory Integrated**.
3. Set Remote Installation folder path to `E:\RemoteInstall`.
4. PXE Server Initial Settings: **Respond to all client computers**.
5. Check DHCP Option 60 boxes if DHCP is on same server. Click **Finish**.

### Step 3: Create MDT Deployment Share

1. Open **Deployment Workbench** (`DeploymentWorkbench.msc`).
2. New Deployment Share -> `C:\DeploymentShare`.
3. Right-click share -> **Properties** -> **Rules** tab.
4. Update `CustomSettings.ini`:

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

5. Update `Bootstrap.ini`:

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

### Step 4: Import OS and Create Task Sequence

1. Import OS: **Operating Systems** -> **Import Operating System** -> Select full source files (Win 11 ISO).
2. Create Task Sequence: **Task Sequences** -> **New Task Sequence**.
   - ID: `WIN11-PROD`
   - Template: **Standard Client Task Sequence**
   - OS: Select Windows 11 image.

### Step 5: Update Deployment Share and Link WDS

1. Update MDT share: Right-click -> **Update Deployment Share**. This generates `LiteTouchPE_x64.wim`.
2. Open WDS console: **Boot Images** -> **Add Boot Image** -> Browse to `LiteTouchPE_x64.wim`.
3. Target VM ko PXE se boot karein aur verify karein.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `wdsutil /get-Server` | Local WDS server ki configuration query karta hai | `wdsutil /get-Server /Show:All` |
| `wdsutil /start-server` | WDS service ko start karta hai | `wdsutil /start-server` |
| `wdsutil /stop-server` | WDS service ko stop karta hai | `wdsutil /stop-server` |
| `dism /Get-WimInfo` | WIM files ke andar image indexes check karta hai | `dism /Get-WimInfo /WimFile:C:\image.wim` |
| `dism /Mount-Image` | WIM file ko mount karta hai offline servicing ke liye | `dism /Mount-Image /WimFile:C:\boot.wim /Index:1 /MountDir:C:\mount` |
| `dism /Unmount-Image` | Modified changes save karke image unmount karta hai | `dism /Unmount-Image /MountDir:C:\mount /Commit` |
| **DHCP Option 66** | WDS Boot Server ka IP address batata hai | `192.168.10.15` |
| **DHCP Option 67** | Bootloader file path define karta hai | `boot\x64\wdsnbp.com` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **"PXE-E32: TFTP open timeout"** | UDP 69 (TFTP) blocked ya WDS stopped hai. | `wdsutil /start-server` run karein. Firewall rules check karein. |
| **"PXE-E53: No boot filename received"** | DHCP Options 66 & 67 missing hain. | DHCP mein Option 66 (WDS IP) aur 67 (`boot\x64\wdsnbp.com` or `wdsmgmt.efi`) configure karein. |
| **"A connection to the deployment share could not be made"** | WinPE mein network driver nahi hai, ya credentials galat hain. | `Bootstrap.ini` credentials check karein. MDT mein NIC driver add karke boot image update karein. |
| **"No storage device detected"** | WinPE mein SATA/RAID controller drivers nahi hain. | MDT mein Intel RST jaise storage drivers import karein aur boot WIM rebuild karein. |
| **"Unable to join domain corp.local"** | DHCP se DNS IP nahi mila, ya AD credentials galat hain. | DHCP Option 006 (DNS) verify karein. `CustomSettings.ini` mein DomainAdmin credentials check karein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Bulk Laptops Imaging TFTP Timeout

> [!example] Ticket
> "Deploying OS to 100 new laptops simultaneously. Imaging is failing with TFTP timeouts and network is extremely slow."

**L1 Response:** Check network cables and PXE boot status. Reboot one device to see if it works individually.
**Escalation Trigger:** Agar single device kaam kar raha hai par bulk mein sab fail ho rahe hain, L2/L3 ko pass karein.
**L2/L3 Resolution:** WDS server pe **Multicast Transmission** configure karein. Switches pe **IGMP Snooping** enable karein aur TFTP block size badhayen taki network saturate na ho.

---
## 🎤 Interview Questions

> [!question] Q1: What key initiates PXE boot and what does PXE do?
> **Answer:** F12 (or Fn+F12) initiates PXE (Preboot Execution Environment) boot. It allows computers to boot and fetch files over the network without a local OS or hard drive.
> 
> ==**Exam Tip:** PXE relies entirely on DHCP (for IP and Options 66/67) and TFTP (for file transfer).==

> [!question] Q2: What is the difference between `Bootstrap.ini` and `CustomSettings.ini`?
> **Answer:** `Bootstrap.ini` is processed first to connect to the deployment share (contains UNC path, credentials). `CustomSettings.ini` is processed after connection to control the wizard rules (skip screens, join domain, computer name).

> [!question] Q3: How do you resolve a "connection to deployment share could not be made" error in WinPE?
> **Answer:** Usually, it's either incorrect credentials in `Bootstrap.ini` or the WinPE boot image is missing the network interface driver (NIC) for that specific client model.

---
## 🔗 Related Notes

- [[03-Identity-and-Core-Services/05-Windows-Server/WS-04 DHCP Server — Install and Configure|WS-04 DHCP Server — Install and Configure]] — Configuring PXE scope options.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — Computer domain registration processes.
- [[05-Automation-and-Ticketing/12-Ansible/ANS-01 Ansible for Windows Admins|ANS-01 Ansible for Windows Admins]] — Post-deployment configuration options.
