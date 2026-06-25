---
tags: [sysadmin, virtualization, lab-setup, home-lab]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# V-04: Virtualization Lab Environment Setup

> [!abstract] Overview
> This note outlines the architecture, hardware requirements, and network layouts for building a multi-VM sysadmin home lab. It details VM sizing targets, network segmentations, and scenarios for Windows, Linux, and Packet Tracer labs.

---
## Concept
Think of your virtualization lab like a flight simulator environment. 
If you want to practice emergency landings (troubleshooting crashes, recovering active directory forest failures, or testing firewall configurations), you don't do it on a real commercial plane carrying paying passengers (production enterprise network). 

Instead, you build a custom flight simulator cabinet (home lab host) and populate it with simulated cockpits (Virtual Machines) connected by virtual wires (Internal Switches). You can test any configuration, simulate failures, crash the system, and instantly reset back to safe conditions using a single button (Restore Snapshot).

*Seedha simple mein: Home lab ek simulated env hai jahan aap bina kisi corporate network ko impact kiye systems, active directory, aur networks configure karte hain. Iske liye hum single high-RAM host par virtual switches and isolated subnets setup karte hain.*

---
## Technical Deep Dive

### 1. Recommended Host Hardware for Lab Sizing
To host a standard sysadmin lab environment running multiple active VMs simultaneously:
- **CPU:** 6 Cores / 12 Threads minimum (Intel Core i5/i7 or AMD Ryzen 5/7). Requires virtualization support enabled in BIOS.
- **RAM:** **32 GB RAM** is the sweet spot. 16 GB RAM limits you to only 2-3 VMs; 64 GB allows a complete hybrid cloud sandbox.
- **Storage:** **1 TB NVMe SSD** minimum. Traditional mechanical HDDs are a massive bottleneck: running 3 VMs on a slow HDD will saturate the active disk controller queues, making the OS freeze.

### 2. Lab Environment Network Diagram
To keep the lab isolated and secure from your home network, segregate the subnets:

```
                  [Home Router / ISP]
                           |
                 (Host physical LAN: 192.168.1.0/24)
                           |
              [Host physical PC (Hypervisor)]
                           |
             +-------------+-------------+
             |                           |
    [vSwitch: NAT]             [vSwitch: Isolated_LAN]
             |                           |
   (Internet Routing)            (Internal Subnet: 192.168.10.0/24)
    - VM-SVR-DC01 (NIC 1)         - VM-SVR-DC01 (NIC 2 - Gateway/Active Directory)
    - VM-SRV-ROCKY (NIC 1)        - VM-CLI-WIN10 (NIC 1 - Domain Joined Client)
                                  - VM-SRV-ROCKY (NIC 2 - Shared intranet server)
```

### 3. Recommended Lab Virtual Machine List

| VM Name | Host role | OS Profile | CPU Cores | RAM | Disk | Network Interfaces |
|---|---|---|---|---|---|---|
| **VM-SVR-DC01** | Windows Domain Controller, DNS, DHCP | Windows Server 2022 (Desktop Experience) | 2 Cores | 2 GB | 60 GB | NIC 1: NAT |
| **VM-SVR-DC02** | Secondary Domain Controller | Windows Server 2022 (Server Core) | 1 Core | 1.5 GB| 40 GB | NIC 1: NAT |
| **VM-SRV-ROCKY**| Linux Web Server, NFS, SSH | Rocky Linux 9 (Minimal Install) | 1 Core | 1 GB | 20 GB | NIC 1: NAT |
| **VM-CLI-WIN10**| Domain joined client | Windows 10 Pro | 2 Cores | 4 GB | 40 GB | NIC 1: Isolated_LAN |

---
## Practical Lab Scenarios

### Lab Scenario 1: Windows Server + Client Domain

#### Objective
Deploy Active Directory Domain Services, configure DNS, configure a DHCP scope, join a Windows 10 client, and push GPOs.
1. **VM Configuration:**
   - Configure `VM-SVR-DC01` with static IP `192.168.10.10/24`, Gateway `192.168.10.1`, DNS `127.0.0.1`.
   - Install **AD DS** and promote to a new forest: `company.local`.
   - Install **DHCP Server**. Create scope `192.168.10.100` to `.200`. Set DNS option to `192.168.10.10`.
2. **Client Configuration:**
   - Boot `VM-CLI-WIN10`. Connect its virtual adapter to the `Isolated_LAN` switch.
   - Verify it obtains an IP in the `192.168.10.x` range via DHCP.
   - Right-click "This PC" -> Properties -> Change settings -> Domain -> Join `company.local`.
3. **Task:** Create a GPO on the DC to map a drive, run `gpupdate /force` on the client, and verify success.

---

### Lab Scenario 2: Linux Server with Services

#### Objective
Configure static IP, enable SSH keys authentication, configure a FirewallD web server, and export a shared directory via NFS.
1. **VM Configuration:**
   - Boot `VM-SRV-ROCKY`. Configure static IP `192.168.10.20/24` using `nmcli`.
   - Set up SSH keys: copy the public key from the client machine to `~/.ssh/authorized_keys`. Disable password authentication in `sshd_config`.
   - Install NGINX: `dnf install nginx -y`. Enable and start: `systemctl enable --now nginx`.
   - Open FirewallD HTTP port:
     ```bash
     firewall-cmd --permanent --add-service=http
     firewall-cmd --reload
     ```
2. **Verification:** Open the web browser on `VM-CLI-WIN10` and navigate to `http://192.168.10.20`. Confirm the NGINX welcome page displays.

---

### Lab Scenario 3: Cisco Packet Tracer Labs

#### Objective
Practice switching VLANs and routing protocols without consuming host hardware resources.
1. Download and launch Cisco Packet Tracer.
2. Build the Inter-VLAN routing (Router-on-a-Stick) lab detailed in [[01-Foundations/02-Networking/N-06 Switching — VLANs and Trunking|N-06 Switching — VLANs and Trunking]].
3. Configure the static/OSPF routing lab detailed in [[01-Foundations/02-Networking/N-07 Routing — Static and Dynamic|N-07 Routing — Static and Dynamic]].
4. Save the network topology `.pkt` file in a dedicated lab backups folder on your host machine.

---
## The Golden Rule of Lab Management: Snapshots
Before beginning *any* lab scenario:
1. Shut down all your virtual machines completely.
2. In your hypervisor (Hyper-V Manager/VMware/VirtualBox), right-click each VM and select **Take Snapshot** / **Checkpoint**.
3. Name it: `BASE_CLEAN_STATE`.
4. Run your lab scripts, practice configurations, and test commands.
5. If you make a mistake, corrupt a system registry, or break network routing, do not waste hours trying to repair it. Shut down the VMs, right-click, and select **Revert to Snapshot** to restore a fresh clean state in seconds.

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-01 Windows Server 2022 Introduction|WS-01 Windows Server 2022 Introduction]] — Base host installations.
- [[02-Operating-Systems/04-Linux-RHEL/L-01 Linux Introduction and Architecture|L-01 Linux Introduction and Architecture]] — Guest Linux configurations.
- [[01-Foundations/01-Hardware/V-01 VMware Workstation Complete Guide|V-01 VMware Workstation Complete Guide]] — Managing VMs on desktop hypervisors.
- [[01-Foundations/01-Hardware/V-02 VirtualBox Complete Guide|V-02 VirtualBox Complete Guide]] — Alternative desktop VM managers.

