---
tags: [sysadmin, lab, virtualization, reference]
difficulty: Advanced
lab-required: Yes
read-time: 20 mins
---

# REF-04: Lab Setup Guide

> [!abstract] Overview
> This guide details how to build and configure a complete system administrator home lab environment. It covers hardware requirements, hypervisor software installation, network topology design, virtual machine creation, and step-by-step configuration steps.

---
## Concept
Think of a sysadmin home lab as an aviation simulation hangar. You don't test new wing designs or experimental control systems on a commercial flight carrying passengers (production servers). Instead, you construct a safe, simulated hangar environment (a localized virtual network) where you can crash and rebuild planes as many times as needed until the system configurations are proven secure and flight-ready.
*Seedha simple mein: Home Lab ek safe training ground hai jahan aap local virtualization software (VirtualBox/VMware/Hyper-V) ka use karke Domain Controllers, Linux systems aur network clients ko connect karke configurations practice karte hain.*

---
## Technical Deep Dive

### 1. Lab Hardware Recommendations
To run a hybrid lab consisting of a Windows Active Directory domain (DC + Client) and several Linux server virtual machines:
- **CPU**: 6 Cores / 12 Threads minimum (Intel Core i5/i7 or AMD Ryzen 5/7).
- **RAM**: 32 GB DDR4/DDR5 is the sweet spot. 16 GB is usable but limits the count of concurrent active virtual machines.
- **Storage**: 1 TB NVMe SSD. Running virtual machines on legacy HDDs will cause severe I/O lag and sluggish performance.

### 2. Software Downloads Directory
Download official developer evaluation versions:
- **Hypervisors**: VirtualBox (Open source) or VMware Workstation Pro.
- **Windows Server 2022 Evaluation**: ISO available from the Microsoft Evaluation Center (180-day free trial).
- **Windows 10/11 Enterprise Evaluation**: ISO available from the Microsoft Evaluation Center.
- **Linux OS**: RHEL (Red Hat Developer Program - free account offers 16 developer licenses) or Rocky Linux (free enterprise-grade RHEL clone).

### 3. Lab Logical Network Topology
To simulate enterprise routing and firewall boundaries, set up two virtual switches:
- **Internal/Host-Only Network**: Isolated space where Domain Controllers, DHCP, DNS, and internal clients reside. This isolates the lab's DHCP server from broadcasting onto your home physical Wi-Fi network.
- **NAT Network**: Connects the virtual routing gateway to the internet, allowing VMs to download security updates safely.

```
                  [ Physical Home Router (WAN) ]
                                |
                               v
                     [ Physical Host PC ]
                     (Runs VMware / VirtualBox)
                                |
             +------------------+------------------+
             |                                     |
             v                                     v
       [ NAT Switch ]                     [ Host-Only Switch ]
             |                                     |
     [ VM-01: Gateway ] <----------------------> [ VM-02: DC01 ]
     (Linux Router / pfSense)             (Active Directory / DNS / DHCP)
                                                   |
                                                   +---> [ VM-03: Win Client ]
                                                   +---> [ VM-04: Linux Server ]
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Workstation running Windows 11 or macOS, 32 GB RAM, 200 GB free disk space, and VMware Workstation Pro installed.

### Step 1: Create Virtual Switches
Configure isolated networking in VMware Workstation:
1. Open VMware Workstation -> `Edit` -> `Virtual Network Editor`.
2. Click **Change Settings** (requires admin rights).
3. Select **VMnet1** -> Set type to **Host-only** -> Uncheck "Use local DHCP service to distribute IP addresses" (AD DS DC will handle DHCP). Set Subnet IP to `192.168.10.0` / Mask `255.255.255.0`.
4. Click **OK** to save.

### Step 2: Install Windows Server 2022 (Domain Controller)
1. In VMware, click **Create a New Virtual Machine** -> Select **Typical** configuration.
2. Select your Windows Server 2022 ISO file.
3. Configure VM Hardware:
   - Memory: `4096 MB` (4 GB)
   - Processors: `2` Cores
   - Hard Disk: `60 GB` NVMe/SCSI
   - Network Adapter: Change connection type to **VMnet1 (Host-only)**.
4. Power on the VM, choose **Server with Desktop Experience** installation, and name the server `DC01`.
5. Assign a static IP address to the network adapter:
   - IP: `192.168.10.10`
   - Subnet: `255.255.255.0`
   - Gateway: Leave blank (until external routing is established)
   - Preferred DNS: `127.0.0.1` (Points to itself for local DNS resolution)

### Step 3: Configure Active Directory and DNS
Install the AD DS role:
1. Open **Server Manager** -> `Manage` -> `Add Roles and Features`.
2. Select **Active Directory Domain Services** -> Click install.
3. Once completed, click the yellow warning flag -> **Promote this server to a Domain Controller**.
4. Select **Add a new forest** -> Root domain name: `lab.local`.
5. Define the Directory Services Restore Mode (DSRM) password -> Install and reboot.

### Step 4: Add Windows Client to Domain
1. Create a new Windows 11 VM in VMware. Set Network Adapter to **VMnet1**.
2. Run installation and assign client IP statically (or configure DHCP scope on `DC01` to automate distribution):
   - IP: `192.168.10.50`
   - Subnet: `255.255.255.0`
   - DNS: `192.168.10.10` (Must point to the Domain Controller IP)
3. Go to **Settings** -> **About** -> **Advanced System Settings** -> **Computer Name** -> **Change**.
4. Select **Domain** -> Type `lab.local`. Enter domain admin credentials when prompted.
5. Reboot client and verify you can log in as a domain user.

---
## Commands Reference
```powershell
# Run on Domain Controller to verify active DNS zones
Get-DnsServerZone
# Check Active Directory replication topology from DC01
repadmin /showrepl
# Force client machine to contact Domain Controller and sync policy
gpupdate /force
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- Problem: The Windows 11 client fails to join the `lab.local` domain, returning the error "An Active Directory Domain Controller (AD DC) for the domain 'lab.local' could not be contacted."
- Root Cause: The client VM's DNS settings do not point to the Domain Controller static IP address (`192.168.10.10`), or the client is connected to the default NAT switch instead of VMnet1.
- Fix:
  1. Open a command prompt on the client VM.
  2. Run `ping lab.local`. If it fails to resolve, run `nslookup lab.local` to see which DNS server is responding.
  3. Go to Network Connections -> Edit IPv4 properties -> Ensure the Preferred DNS server is set exactly to the DC's IP: `192.168.10.10`.
  4. Verify the VM settings in VMware show the network adapter connected to **VMnet1 (Host-only)**.

**Scenario 2:**
- Problem: The home network physical router flags IP address conflict warnings, and family members lose internet access.
- Root Cause: You enabled the DHCP server role on the lab VM while it was connected to the Bridged or NAT network adapter, causing it to distribute IP configurations to physical devices in your home.
- Fix:
  1. Turn off your lab virtual machines immediately.
  2. Open the Hypervisor configuration settings.
  3. Ensure that any virtual machines hosting DHCP services (like Windows Server DC or routers) are strictly assigned to an isolated internal switch network (Host-only/VMnet1) and not mapped to Bridged interfaces.

---
## Common Mistakes
> [!warning] Avoid These
> Bridging the network adapter of your Domain Controller directly to your home router. This allows your home network devices to send DNS queries to your lab DC, causing internet connection drops at home.
> Allocating all host PC RAM to virtual machines. Leave at least 8 GB of RAM for the host operating system to prevent system memory lockups.
> Neglecting to take virtual machine Snapshots before promoting servers or making major configuration changes, preventing easy rollbacks.

---
## Pro Tips
> [!tip] Field Experience
> Take a base snapshot of your virtual machines immediately after installing the OS and configuring updates, but before adding active roles. If your trial licenses expire, you can roll back to the base snapshot to reset the 180-day evaluation trial.
> Configure **Nested Virtualization** in your hypervisor CPU settings. This allows you to run hypervisors (like Hyper-V or Docker) inside your virtual machines, which is useful for testing cloud configurations.

---
## Quick Revision Table
| # | Setup Phase | Key Action | Verification Method |
|---|-------------|------------|---------------------|
| 1 | Network | Create isolated Virtual Switch | Verify VMnet status in Network Editor |
| 2 | OS Deploy | Install Windows Server (Desktop Exp) | Confirm Server Manager opens |
| 3 | Static IP | Configure IP, Subnet, DNS | Run `ipconfig /all` on DC |
| 4 | AD DS | Promote DC & Create Forest | Verify domain log on `lab.local` |
| 5 | Client Join | Point DNS to DC IP and join | Confirm client domain log in |

---
## Interview Q&A

**Q1: Why is it critical to configure a static IP address on a domain controller, and what should the preferred DNS server be set to?**
A: A Domain Controller must have a static IP address because network clients and member servers rely on it for authentication and DNS resolution. If its IP address changes via DHCP, clients will lose connection. The preferred DNS server on the DC should be set to its loopback address (`127.0.0.1` or its own static IP) because Active Directory relies on DNS to find domain resources, and the DC must manage its own local DNS database zone.

**Q2: You need to design a test lab that replicates an organization's multi-subnet network architecture. How would you configure this on a single host computer?**
A:
- **Situation**: The client needed a home lab simulating three subnets (Production, Dev, and DMZ) connected to a central firewall.
- **Task**: I needed to configure virtual switches and routing without using physical hardware.
- **Action**: I created three Host-only virtual networks in VMware (VMnet2, VMnet3, VMnet4). I deployed a virtual router machine (pfSense) configured with four network adapters: one connected to NAT (WAN) and the other three connected to the respective Host-only networks. I configured DHCP scopes on pfSense for each segment and set up firewall routing rules.
- **Result**: The virtual networks successfully isolated traffic while enabling routing between subnets, simulating a secure enterprise network architecture.

**Q3: How does a VM Snapshot differ from a Backup, and when should you use each?**
A: A snapshot captures the exact state, memory, and virtual disk contents of a VM at a point-in-time, allowing you to roll back to that state quickly (best before risky changes like software upgrades). However, snapshots degrade performance over time because writes are redirected to delta disks. A backup duplicates the VM disk data to separate storage, providing disaster recovery capability without impacting VM performance.

---
## Related Notes
- [[01-Foundations/01-Hardware/V-04 Virtualization Lab Environment Setup|V-04 Virtualization Lab Environment Setup]] — Details home lab hardware setups.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — Details DC configuration steps.
- [[00-MOC/REF-03 Troubleshooting Decision Trees|REF-03 Troubleshooting Decision Trees]] — Implements these lab setups to trace system failures.

