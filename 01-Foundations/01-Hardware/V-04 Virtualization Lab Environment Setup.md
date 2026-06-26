---
tags: [sysadmin, virtualization, lab-setup, home-lab]
aliases: [home-lab-setup]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-yellow]
> 🔧 **HARDWARE**

`#complete` `#intermediate` `#none`

# V-04: Virtualization Lab Environment Setup

> [!abstract] Overview
> This note outlines the architecture, hardware requirements, and network layouts for building a multi-VM sysadmin home lab. Yeh ek support engineer ko lab setup karne aur real-world scenarios practice karne ke liye zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — A customized, isolated virtualization environment hosted on a single powerful PC.
- **Why it matters** — Allows sysadmins to test configurations, AD deployments, and network routing safely without impacting production.
- **Where you see this** — Used for certification prep (RHCSA, CCNA, MSCA) or testing patches before enterprise deployment.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check VM status, ensure host has sufficient RAM/Storage. |
| **L2** | Configure VM networking, deploy OS, configure AD/DNS/DHCP inside VMs. |
| **L3** | Architecture lab network segmentation, design automated lab deployments. |

> [!tip] Seedha Simple Mein
> *Home lab ek simulated env hai jahan aap bina kisi corporate network ko impact kiye systems, active directory, aur networks configure karte hain. Hum single high-RAM host par virtual switches and isolated subnets setup karte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A Virtualization Lab** is like a **Flight Simulator** because...
>
> - You can practice emergency procedures (troubleshooting crashes, AD failures) without risking a real commercial plane (production enterprise network).
> - You can instantly reset back to safe conditions using a single button (Restore Snapshot) if you crash.

---
## 🔬 Technical Deep Dive

### 1. Recommended Host Hardware

> [!info] Key Concept
> Hardware sizing dictates how many VMs you can run concurrently without performance degradation.

- **CPU:** 6 Cores / 12 Threads minimum (Intel Core i5/i7 or AMD Ryzen 5/7). Requires virtualization support enabled in BIOS.
- **RAM:** **32 GB RAM** is the sweet spot. 16 GB RAM limits you to only 2-3 VMs; 64 GB allows a complete hybrid cloud sandbox.
- **Storage:** **1 TB NVMe SSD** minimum. Traditional mechanical HDDs are a massive bottleneck: running 3 VMs on a slow HDD will saturate the active disk controller queues, making the OS freeze.

> [!danger] Common Mistake
> Trying to run multiple Windows Server VMs on a traditional spinning HDD. Always use an NVMe SSD for your datastore.

### 2. Lab Environment Network Diagram

```text
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
    - VM-SVR-DC01 (NIC 1)         - VM-SVR-DC01 (NIC 2 - Gateway/AD)
    - VM-SRV-ROCKY (NIC 1)        - VM-CLI-WIN10 (NIC 1 - Domain Joined)
                                  - VM-SRV-ROCKY (NIC 2 - Shared intranet)
```

### 3. Recommended Lab Virtual Machine List

| 🖥️ VM Name | 🛠️ Host Role | 💿 OS Profile | 🧠 CPU | 💾 RAM | 📀 Disk | 🌐 Network Interfaces |
|---|---|---|---|---|---|---|
| **VM-SVR-DC01** | Windows Domain Controller, DNS, DHCP | Windows Server 2022 (GUI) | 2 Cores | 2 GB | 60 GB | NIC 1: NAT |
| **VM-SVR-DC02** | Secondary Domain Controller | Windows Server 2022 (Core) | 1 Core | 1.5 GB| 40 GB | NIC 1: NAT |
| **VM-SRV-ROCKY**| Linux Web Server, NFS, SSH | Rocky Linux 9 (Minimal) | 1 Core | 1 GB | 20 GB | NIC 1: NAT |
| **VM-CLI-WIN10**| Domain joined client | Windows 10 Pro | 2 Cores | 4 GB | 40 GB | NIC 1: Isolated_LAN |

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Host machine with at least 16GB RAM and 500GB SSD.
> - Virtualization enabled in BIOS.
> - Hypervisor installed (VMware Workstation, VirtualBox, or Hyper-V).

### Step 1: The Golden Rule - Snapshots

Before beginning *any* lab scenario:
1. Shut down all virtual machines completely.
2. In your hypervisor, right-click each VM and select **Take Snapshot** / **Checkpoint**.
3. Name it: `BASE_CLEAN_STATE`.

> [!success] Quick Win
> If you make a mistake, simply **Revert to Snapshot** to restore a fresh clean state in seconds.

### Step 2: Windows Server + Client Domain Lab

1. Configure `VM-SVR-DC01` with static IP `192.168.10.10/24`, Gateway `192.168.10.1`, DNS `127.0.0.1`.
2. Install **AD DS** and promote to a new forest: `company.local`.
3. Install **DHCP Server**. Create scope `192.168.10.100` to `.200`. Set DNS to `192.168.10.10`.
4. Boot `VM-CLI-WIN10` (on `Isolated_LAN`), obtain DHCP IP, and join `company.local`.

### Step 3: Linux Server with Services Lab

1. Boot `VM-SRV-ROCKY`. Configure static IP `192.168.10.20/24` using `nmcli`.
2. Open FirewallD HTTP port:
```bash
# Allow HTTP traffic permanently
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `firewall-cmd --reload` | Applies firewall changes | `firewall-cmd --reload` |
| `nmcli` | NetworkManager command line tool | `nmcli con show` |
| `gpupdate /force` | Forces Group Policy update | `gpupdate /force` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| VMs running extremely slow | Disk I/O bottleneck (running on HDD) | Migrate VMs to an NVMe SSD |
| Host freezes entirely | Over-provisioned RAM across active VMs | Reduce RAM allocation per VM or add more physical RAM |
| Client cannot ping DC | Isolated_LAN vSwitch not configured correctly | Ensure both VMs are on the same virtual switch |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Needs a Safe Testing Ground

> [!example] Ticket
> "I need to test a new Group Policy Object that restricts USB access, but I don't want to lock out real users if it fails."

**L1 Response:** Confirm the requirement and hardware availability for a test lab.
**Escalation Trigger:** If complex networking or licensing is required for the test environment.
**L2 Resolution:** Spin up the `BASE_CLEAN_STATE` VMs, replicate the AD OU structure, test the GPO, verify results, and then revert the snapshot.

---
## 🎤 Interview Questions

> [!question] Q1: Why is an SSD critical for virtualization labs compared to CPU cores?
> **Answer:** Multiple VMs perform random read/write operations constantly. An HDD's mechanical head cannot handle the parallel I/O queues, leading to massive latency, whereas an SSD handles parallel I/O natively.

==**Exam Tip:** In a lab, RAM and Storage speed are the primary bottlenecks, not CPU.==

---
## 🔗 Related Notes

- [[03-Identity-and-Core-Services/05-Windows-Server/WS-01 Windows Server 2022 Introduction|WS-01 Windows Server 2022 Introduction]] — Base host installations.
- [[02-Operating-Systems/04-Linux-RHEL/L-01 Linux Introduction and Architecture|L-01 Linux Introduction and Architecture]] — Guest Linux configurations.
- [[01-Foundations/01-Hardware/V-01 VMware Workstation Complete Guide|V-01 VMware Workstation Complete Guide]] — Managing VMs on desktop hypervisors.
- [[01-Foundations/01-Hardware/V-02 VirtualBox Complete Guide|V-02 VirtualBox Complete Guide]] — Alternative desktop VM managers.
