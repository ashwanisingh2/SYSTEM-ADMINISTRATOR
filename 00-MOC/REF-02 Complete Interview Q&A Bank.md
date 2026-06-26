---
tags: [sysadmin, interview, careers, reference]
difficulty: Advanced
lab-required: No
read-time: 45 mins
---

# REF-02: Complete Interview Q&A Bank

> [!abstract] Overview
> This reference note contains a comprehensive interview preparation bank containing 175 detailed questions and answers. It covers PC Hardware, CCNA Networking, Windows Server administration, Linux RHCSA commands, Microsoft Azure (AZ-900/104), M365/Intune operations, WSUS, Ansible, and Incident Response.

---
## Concept
Think of this interview bank as a flight simulator. A pilot doesn't learn how to recover from engine failure by reading theory during a flight; they practice in a simulator. This document simulates technical interviews, preparing you to answer questions confidently under pressure.
*Seedha simple mein: Yeh ek master interview question bank hai jo hardware, networks, active directory, linux, azure aur intune ke advanced technical aur scenario-based questions ka complete collection hai.*

---
## Technical Deep Dive: 175 Interview Questions & Answers

### Part 1: Hardware & Storage (Q1 - Q25)

**Q1: What is the difference between UEFI and BIOS, and why is UEFI preferred today?**
A: BIOS (Basic Input/Output System) is legacy firmware that initializes hardware using 16-bit real mode and supports disks up to 2.2 TB (MBR). UEFI (Unified Extensible Firmware Interface) runs in 32-bit or 64-bit mode, supports disks up to 9.4 ZB (GPT), features Secure Boot to prevent unsigned malware from loading at boot, and integrates with TPM 2.0.

**Q2: How does TPM 2.0 function, and why is it a requirement for Windows 11?**
A: Trusted Platform Module (TPM) 2.0 is a secure cryptographic hardware chip that generates, stores, and protects encryption keys, certificates, and passwords. Windows 11 requires it to secure features like BitLocker drive encryption, Windows Hello, and Credential Guard.

**Q3: Explain the difference between MBR and GPT partition tables.**
A: MBR (Master Boot Record) supports up to 4 primary partitions and disk sizes up to 2.2 TB. GPT (GUID Partition Table) supports up to 128 partitions per drive, disk sizes up to 9.4 ZB, and stores duplicate partition layouts at the end of the disk for redundancy.

**Q4: Compare NVMe SSDs and SATA SSDs regarding performance and interface lanes.**
A: SATA SSDs use the legacy AHCI protocol designed for HDDs, peaking at 600 MB/s over SATA III. NVMe SSDs communicate directly with the CPU using PCIe lanes, utilizing the NVMe protocol to reach speeds over 7,000 MB/s (PCIe Gen 4/5) with lower latency.

**Q5: What is RAID 10, and how does it compare to RAID 5?**
A: RAID 10 (1+0) is a stripe of mirrors, requiring a minimum of 4 disks. It provides high write performance and can survive multiple disk failures if they are in different mirrored sets. RAID 5 stripes data and parity across a minimum of 3 disks. It is more storage-efficient but has slower write speeds due to parity calculation overhead.

**Q6: What does the 80 Plus rating on an SMPS indicate?**
A: It certifies that the power supply is at least 80% energy efficient at 20%, 50%, and 100% loads. The tiers (Bronze, Silver, Gold, Platinum, Titanium) represent increasing efficiency levels, which reduces heat output and energy costs.

**Q7: Explain the concept of thermal throttling in modern CPUs.**
A: When a CPU exceeds its safe operating temperature threshold (usually 95°C–100°C), it automatically reduces its clock speed and core voltage to prevent thermal damage, which reduces system performance until temperatures normalize.

**Q8: What is the difference between ECC and non-ECC RAM?**
A: ECC (Error-Correcting Code) RAM detects and corrects single-bit memory errors in real-time, preventing system crashes and data corruption. Non-ECC RAM does not have this capability and is used in standard consumer devices.

**Q9: What steps would you take if a PC turns on, fans spin, but there is no display (no POST)?**
A: I would verify the monitor and cable connections, check for POST beep codes or motherboard debug LEDs, isolate the RAM by testing one stick at a time, clear the CMOS battery to reset BIOS defaults, and test the power supply voltages.

**Q10: What is the difference between PCIe lanes x4, x8, and x16?**
A: The multipliers represent physical and electrical bandwidth lanes. An x16 slot has 16 individual data transmission pathways, providing double the bandwidth of an x8 slot, and is typically used for discrete graphics cards.

**Q11: Explain how an M.2 NVMe slot shares bandwidth with SATA ports.**
A: On many motherboards, the M.2 slot shares PCIe lanes with specific SATA ports. Installing an NVMe drive may disable those SATA ports. Refer to the motherboard manual to plan drive connections.

**Q12: What is the difference between single-channel and dual-channel RAM configurations?**
A: Single-channel RAM uses a single 64-bit data path. Dual-channel RAM utilizes two independent 64-bit paths (128-bit total), doubling the memory bandwidth and improving CPU performance.

**Q13: How does a modular PSU differ from a non-modular PSU?**
A: A modular PSU allows you to connect only the power cables needed for your specific build. This improves cable management, clean airflow, and reduces clutter compared to non-modular PSUs with fixed cable bundles.

**Q14: Explain the POST (Power-On Self-Test) process.**
A: The POST is a diagnostic test run by the system BIOS/UEFI immediately after power-on. It verifies the integrity of essential components (CPU, RAM, storage controllers, GPU) before loading the operating system bootloader.

**Q15: What is the function of the CMOS battery?**
A: The CMOS battery (usually a CR2032 button cell) provides continuous power to the motherboard's volatile RTC (Real-Time Clock) chip and stores custom BIOS settings when the PC is powered off.

**Q16: How do you identify bad sectors on a hard drive?**
A: I monitor SMART attributes using diagnostic tools like CrystalDiskInfo. Specifically, I look at the Reallocated Sectors Count, Current Pending Sector Count, and Uncorrectable Sector Count to assess drive health.

**Q17: What is XMP/DOCP, and why must it be enabled in the BIOS?**
A: XMP (Intel) and DOCP (AMD) are pre-configured manufacturer memory profiles. If disabled, high-speed RAM will run at standard default JEDEC speeds (e.g., 2133 MHz instead of 3600 MHz).

**Q18: What is the difference between LGA and PGA CPU sockets?**
A: LGA (Land Grid Array) has pins on the motherboard socket, with pads on the CPU (used by Intel and modern AMD). PGA (Pin Grid Array) has pins on the CPU itself, which insert into a motherboard socket.

**Q19: What is thermal paste, and why is it applied between the CPU and cooler?**
A: Thermal paste is a conductive compound that fills microscopic air gaps between the CPU integrated heat spreader (IHS) and the cooler base, maximizing heat transfer.

**Q20: Why do laptops use SODIMM RAM instead of standard UDIMM RAM?**
A: SODIMM (Small Outline Dual In-line Memory Module) is physically smaller than UDIMM, allowing it to fit within the space-constrained chassis of laptops and small-form-factor PCs.

**Q21: Explain the difference between active and passive cooling.**
A: Active cooling uses power to move air or liquid (e.g., fans, pumps) to dissipate heat. Passive cooling uses heatsinks with no moving parts, relying on natural convection to cool components silently.

**Q22: How does a liquid CPU cooler work?**
A: A pump circulates liquid coolant through a water block mounted on the CPU. The liquid absorbs heat and travels to a radiator, where fans cool it before it returns to the water block.

**Q23: What are the symptoms of a failing SMPS?**
A: Symptoms include random system reboots under load, failure to power on, coil whine, electrical burning smells, USB device disconnects, or BSODs related to power instability.

**Q24: How do you perform a paperclip test on a PSU?**
A: I disconnect the 24-pin ATX power cable from the motherboard, insert a paperclip into the Green wire pin (PS_ON) and any Black wire pin (COM/Ground), and plug in the power cord. If the PSU fan spins, the unit can power on.

**Q25: Explain the function of PCIe bifurcation.**
A: PCIe bifurcation allows a single PCIe slot to split its lanes across multiple devices, such as splitting an x16 slot into four x4 connections to support a multi-drive M.2 expansion card.

---

### Part 2: Networking & Routing (Q26 - Q50)

**Q26: Explain the difference between the OSI model and the TCP/IP model.**
A: The OSI model is a 7-layer reference model (Physical, Data Link, Network, Transport, Session, Presentation, Application). The TCP/IP model is a 4-layer practical suite (Network Access, Internet, Transport, Application) that maps directly to internet protocols.

**Q27: How does ARP (Address Resolution Protocol) work?**
A: ARP maps a known 32-bit IPv4 address to a 48-bit physical MAC address. A device broadcasts an "ARP request" containing the target IP. The owner of that IP replies with its MAC address, which the sender caches in its ARP table.

**Q28: Explain the difference between a collision domain and a broadcast domain.**
A: A collision domain is a network segment where packets can collide (e.g., ports on a hub). A switch divides collision domains on each port. A broadcast domain is a segment where a broadcast packet reaches all hosts. A router separates broadcast domains at its layer-3 interfaces.

**Q29: How does a Layer 2 switch build its MAC address table?**
A: A switch inspects the source MAC address of incoming frames on each port. It records the MAC address and incoming port number in its MAC table (CAM table) to forward future frames directly to that port.

**Q30: What is the DORA process in DHCP?**
A: It is the 4-step IP allocation sequence:
- **Discover**: Client broadcasts a request for an IP address.
- **Offer**: DHCP server offers an IP configuration.
- **Request**: Client requests to lease the offered IP.
- **Acknowledge**: Server confirms the lease and sends configuration details.

**Q31: What is the difference between public and private IP addresses, and what RFC defines private ranges?**
A: Public IPs are routable on the global internet. Private IPs are restricted to internal networks to conserve IP address space. RFC 1918 defines the private IP ranges: `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`.

**Q32: Explain the purpose of a subnet mask.**
A: A subnet mask determines which part of an IP address represents the network address and which part represents the host address. It uses binary 1s for the network portion and 0s for the host portion.

**Q33: What is CIDR notation? Give an example.**
A: Classless Inter-Domain Routing (CIDR) represents a subnet mask using a slash followed by the count of active network bits. For example, `/24` represents the subnet mask `255.255.255.0`.

**Q34: How does IPv6 NDP (Neighbor Discovery Protocol) replace ARP?**
A: IPv6 does not use broadcast packets. NDP uses multicast ICMPv6 packets (Neighbor Solicitation and Neighbor Advertisement) to discover MAC addresses on the local link.

**Q35: What is the difference between a standard ACL and an extended ACL in Cisco routing?**
A: Standard ACLs (numbered 1–99) filter traffic based on source IP address only and should be placed close to the destination. Extended ACLs (numbered 100–199) filter based on source/destination IPs, protocols, and port numbers, and should be placed close to the source.

**Q36: Explain the purpose of Administrative Distance (AD) in routing.**
A: AD determines the trustworthiness of a routing protocol. If a router receives paths to the same destination from multiple protocols, it selects the path with the lowest AD value (e.g., Connected: 0, Static: 1, OSPF: 110).

**Q37: How does OSPF (Open Shortest Path First) build its routing database?**
A: OSPF routers exchange Link-State Advertisements (LSAs) to build a link-state database (LSDB). They then run the Dijkstra Shortest Path First (SPF) algorithm to calculate the loop-free path to each network.

**Q38: List the 7 OSPF neighbor states in order.**
A: The states are: Down, Init, Two-Way, ExStart, Exchange, Loading, and Full.

**Q39: What is the difference between NAT and PAT?**
A: NAT (Network Address Translation) maps one private IP to one public IP (1:1). PAT (Port Address Translation / NAT Overload) maps multiple private IPs to a single public IP using unique source port numbers.

**Q40: Explain the DNS resolution process.**
A: When a client queries a domain, it first checks local cache and the hosts file. If not found, it queries the local DNS resolver, which contacts the Root name servers, the Top-Level Domain (TLD) servers (e.g., `.com`), and the Authoritative name servers to retrieve the IP address.

**Q41: What is a wildcard mask, and how does it differ from a subnet mask?**
A: A wildcard mask is the inverse of a subnet mask, used in ACLs. Binary 0s indicate that the corresponding address bits must match, while 1s indicate they can be ignored.

**Q42: What is the difference between TCP and UDP?**
A: TCP is a connection-oriented, reliable protocol that uses a 3-way handshake, flow control, and retransmissions. UDP is a connectionless, unreliable protocol that sends packets without verification, providing lower latency.

**Q43: What is the purpose of the Spanning Tree Protocol (STP)?**
A: STP prevents network loops on Layer 2 switch topologies by placing redundant links into a blocking state, maintaining a single loop-free path.

**Q44: What is a VLAN, and why is it used?**
A: A Virtual Local Area Network (VLAN) partitions a physical switch into multiple logical networks, separating broadcast domains and improving security.

**Q45: Explain the difference between an access port and a trunk port.**
A: An access port carries traffic for a single assigned VLAN, typically connected to end devices. A trunk port carries traffic for multiple VLANs simultaneously, using 802.1Q encapsulation tags to identify the VLAN for each frame.

**Q46: What is Inter-VLAN routing, and how is "Router-on-a-Stick" configured?**
A: Inter-VLAN routing forwards traffic between different VLANs. In a "Router-on-a-Stick" configuration, a single physical router interface is split into multiple logical sub-interfaces, each assigned an IP address on a different VLAN and configured for 802.1Q encapsulation.

**Q47: Explain the function of the Default Gateway.**
A: The default gateway is the router interface that local hosts send traffic to when the destination IP address is outside the local subnet.

**Q48: What is the purpose of a Loopback address?**
A: A loopback address (e.g., `127.0.0.1` in IPv4, `::1` in IPv6) is a virtual interface used by a device to send network traffic to itself, useful for diagnostic testing.

**Q49: What is the function of the TTL (Time-To-Live) field in an IP header?**
A: TTL prevents packets from circulating indefinitely in routing loops. Every router that forwards the packet decrements the TTL value by 1. If TTL reaches 0, the packet is discarded, and an ICMP "Time Exceeded" message is sent back to the source.

**Q50: How does the traceroute command work using TTL?**
A: Traceroute sends packets with increasing TTL values starting at 1. The first router decrements the TTL to 0, drops the packet, and returns an ICMP Time Exceeded message, revealing its IP address. This repeats until the packet reaches the destination.

---

### Part 3: Windows Server & Active Directory (Q51 - Q75)

**Q51: What is the purpose of Active Directory Domain Services (AD DS)?**
A: AD DS is a directory service that stores information about network objects (users, computers, groups, printers) and manages authentication and authorization across a domain environment.

**Q52: Explain the logical components of Active Directory.**
A: The logical components are:
- **Organizational Units (OUs)**: Containers used to organize objects and delegate administrative permissions.
- **Domains**: The core security boundary sharing a common directory database.
- **Trees**: A collection of domains sharing a contiguous namespace.
- **Forests**: The outermost security boundary containing one or more trees that share a common schema and configuration directory.

**Q53: What is the Active Directory database file, and where is it stored?**
A: The database file is `ntds.dit`, stored by default in the `%systemroot%\NTDS` folder on Domain Controllers.

**Q54: What is the SYSVOL folder, and why is it critical?**
A: SYSVOL is a shared directory on all Domain Controllers that stores public files, software installation packages, and Group Policy templates. It must be replicated across all Domain Controllers.

**Q55: Explain the processing order of Group Policy Objects (GPOs).**
A: GPOs are processed in **LSDOU** order: Local, Site, Domain, and Organizational Unit (OU). The policy applied last overrides policies applied earlier in the sequence.

**Q56: What is the difference between Group Policy Enforcement and Block Inheritance?**
A: **Block Inheritance** prevents GPOs linked to higher-level containers from applying to child OUs. **Enforced** overrides Block Inheritance, forcing the GPO settings to apply to all child containers.

**Q57: List the 5 FSMO (Flexible Single Master Operation) roles.**
A: The 5 FSMO roles are: Schema Master, Domain Naming Master, PDC Emulator, RID Master, and Infrastructure Master.

**Q58: What happens if the RID Master fails?**
A: The RID Master allocates pools of Relative IDs to Domain Controllers. If it fails, Domain Controllers can continue to create new security objects (users, computers) until their active RID pool is exhausted.

**Q59: Explain the difference between standard security groups and distribution groups.**
A: Security groups control access permissions to shared resources (files, folders) and can also be used as distribution lists. Distribution groups are used only for email distribution and cannot be assigned access permissions.

**Q60: What is the AGDLP rule in Active Directory?**
A: It is a group nesting best practice: **A**ccounts are placed into **G**lobal groups, which are nested inside **D**omain **L**ocal groups, which are assigned **P**ermissions to resources.

**Q61: Explain the difference between NTFS permissions and Share permissions.**
A: Share permissions apply when accessing files over the network. NTFS permissions apply both locally and over the network. When combined, the most restrictive permission set applies.

**Q62: What is the Active Directory Recycle Bin, and how do you enable it?**
A: The Active Directory Recycle Bin allows administrators to restore deleted AD objects without restoring from backup. It is enabled using the Active Directory Administrative Center (ADAC) or PowerShell. Once enabled, it cannot be disabled.

**Q63: What is a Read-Only Domain Controller (RODC), and when would you deploy one?**
A: An RODC holds a read-only copy of the AD DS database. It is deployed in locations with limited physical security (e.g., branch offices), as it does not cache user passwords by default except as permitted by a Password Replication Policy.

**Q64: Explain the difference between DNS forwarders and conditional forwarders.**
A: A DNS forwarder sends all queries it cannot resolve to an external DNS server (e.g., ISP or public DNS). A conditional forwarder sends queries for a specific domain name (e.g., `partner.com`) to a designated DNS server.

**Q65: What is DNS scavenging, and why is it configured?**
A: DNS scavenging automatically deletes stale, old DNS records from the zone database, preventing database bloat and IP conflicts.

**Q66: How does DHCP Failover in Windows Server function?**
A: DHCP Failover provides high availability for DHCP scopes. It runs in **Hot Standby** mode (active/passive, where one server handles requests and the other takes over if the primary fails) or **Load Balance** mode (both servers actively distribute IPs).

**Q67: Explain the purpose of the PDC Emulator role.**
A: The PDC Emulator manages time synchronization across the domain, acts as the primary target for password updates and resets, and processes GPO changes.

**Q68: How do you transfer FSMO roles, and how does it differ from seizing them?**
A: **Transferring** FSMO roles is a graceful migration performed when both Domain Controllers are online. **Seizing** is an emergency action performed when the role holder DC is permanently offline and cannot be recovered. Seized DCs must never be brought back online.

**Q69: What is DFS (Distributed File System) Namespace?**
A: DFS Namespace groups shared folders located on different servers into a single logical folder tree, providing users with a unified access path (e.g., `\\domain.local\dfsroot`).

**Q70: What is File Server Resource Manager (FSRM)?**
A: FSRM is a Windows Server role service that allows administrators to manage storage quotas, screen files (e.g., block saving `.mp3` or `.exe` files), and generate storage reports.

**Q71: Explain the difference between a basic disk and a dynamic disk in Windows.**
A: Basic disks use standard MBR or GPT partition tables. Dynamic disks support software RAID configurations (Spanned, Striped, Mirrored, RAID 5 volumes) across multiple drives.

**Q72: What is Hyper-V replication?**
A: Hyper-V replication copies virtual machines from one Hyper-V host to another over a network link, providing disaster recovery capability.

**Q73: What is the difference between Generation 1 and Generation 2 virtual machines in Hyper-V?**
A: Generation 1 VMs emulate legacy hardware configurations and boot using BIOS. Generation 2 VMs use UEFI, support Secure Boot, utilize virtual SCSI controllers, and provide faster boot times.

**Q74: Explain the difference between External, Internal, and Private virtual switches in Hyper-V.**
A:
- **External**: Binds to the physical network adapter, allowing VMs to access the external network.
- **Internal**: Allows communication between VMs and the host operating system.
- **Private**: Allows communication only between VMs on the same host, isolated from the host OS.

**Q75: What is Active Directory Certificate Services (AD CS)?**
A: AD CS is a server role that acts as a Public Key Infrastructure (PKI), issuing and managing digital certificates for secure authentication, encryption, and signatures across the organization.

---

### Part 4: Linux Systems (RHCSA) (Q76 - Q100)

**Q76: What is the function of the Linux Kernel?**
A: The Kernel is the core of the operating system. It manages hardware resources, memory allocation, process scheduling, device drivers, and system calls between hardware and applications.

**Q77: Explain the Linux filesystem hierarchy (FHS).**
A: The FHS defines the directory layout:
- `/etc`: Configuration files.
- `/var`: Volatile files (logs, databases).
- `/home`: User home directories.
- `/bin` & `/sbin`: System binary commands.
- `/dev`: Device files.

**Q78: What is the difference between a hard link and a soft link?**
A: A hard link points directly to the inode of a file, sharing the same data block; it cannot cross filesystems or point to directories. A soft link (symlink) is a pointer shortcut to the file path; it can cross filesystems and point to directories, but becomes broken if the target file is deleted.

**Q79: Explain the permissions string `-rwxr-xr-x` on a Linux file.**
A: The string represents:
- `-`: Regular file.
- `rwx`: Owner has Read, Write, and Execute permissions (7).
- `r-x`: Group members have Read and Execute permissions (5).
- `r-x`: Others have Read and Execute permissions (5).

**Q80: What is the purpose of the umask value?**
A: The umask (User Mask) determines the default permissions assigned to newly created files and directories. For example, a umask of `022` results in default permissions of `644` for files (666 - 022) and `755` for directories (777 - 022).

**Q81: How do SUID, SGID, and the Sticky Bit work?**
A:
- **SUID (Set User ID)**: Executes the file with the privileges of the file owner (e.g., `/usr/bin/passwd` runs as root).
- **SGID (Set Group ID)**: Executes the file with the privileges of the file group, or forces new files in a directory to inherit the parent directory's group ownership.
- **Sticky Bit**: Restricts file deletion inside a directory to the file owner or root user (e.g., `/tmp`).

**Q82: What is the difference between the su and sudo commands?**
A: `su` switches the session to the root user account, requiring the root password. `sudo` executes a command with elevated privileges using the current user's credentials, logged in `/etc/sudoers` for auditing.

**Q83: Explain the fields of the `/etc/passwd` file.**
A: The fields are: Username, Password placeholder (x), User ID (UID), Group ID (GID), User Info (GECOS), Home Directory, and Default Shell.

**Q84: What is the difference between process states R, S, D, and Z?**
A:
- **R (Running/Runnable)**: Active in CPU queue.
- **S (Interruptible Sleep)**: Waiting for an event or resource.
- **D (Uninterruptible Sleep)**: Waiting for I/O operations.
- **Z (Zombie)**: Process has terminated but its exit status has not been read by its parent.

**Q85: How do you change a process priority in Linux?**
A: I use `nice` to set the priority of a command upon launch (value range -20 to 19), or `renice` to change the priority of an already running process using its PID. Lower values indicate higher priority.

**Q86: What are the differences between SIGTERM (15) and SIGKILL (9) signals?**
A: `SIGTERM` requests a process to terminate gracefully, allowing it to save data and clean up resources. `SIGKILL` forces the kernel to terminate the process immediately, risking data corruption.

**Q87: What is systemd, and why did it replace SysV init?**
A: `systemd` is the modern system initialization daemon. It parallelizes service startup, uses target units instead of runlevels, simplifies dependency tracking, and logs output directly to a centralized journal (`journalctl`).

**Q88: How do you configure a service to start automatically at boot?**
A: I run `systemctl enable <service-name>`. To start the service immediately as well, I run `systemctl enable --now <service-name>`.

**Q89: How do you restrict root login over SSH?**
A: I edit `/etc/ssh/sshd_config`, change the parameter `PermitRootLogin` to `no`, and run `systemctl restart sshd`.

**Q90: What is the difference between yum and dnf package managers?**
A: `dnf` is the successor to `yum` in RHEL 8/9. It has better dependency resolution, uses less memory, and is backward-compatible with `yum` syntax.

**Q91: Explain the structure of the `/etc/fstab` file.**
A: It contains six columns: Device identifier (UUID/path), Mount point, Filesystem type (ext4, xfs), Mount options (defaults), Dump backup flag, and fsck file check priority order.

**Q92: What are PV, VG, and LV in LVM?**
A:
- **Physical Volume (PV)**: The physical hard drive or partition.
- **Volume Group (VG)**: A storage pool created by combining multiple PVs.
- **Logical Volume (LV)**: A partition carved from the VG, which can be formatted with a filesystem and mounted.

**Q93: How do you extend an LVM volume and resize its filesystem online?**
A: I run `lvextend -L +5G /dev/vg_name/lv_name` to expand the logical volume. Then, I resize the filesystem using `resize2fs` (for ext4) or `xfs_growfs` (for xfs).

**Q94: What is SELinux, and what are its three modes?**
A: Security-Enhanced Linux (SELinux) is a kernel security module enforcing mandatory access control (MAC). The modes are:
- **Enforcing**: Blocks actions that violate policies and logs them.
- **Permissive**: Allows actions but logs policy violations.
- **Disabled**: Turns off SELinux enforcement and logging.

**Q95: How do you configure a static IP address in RHEL using the CLI?**
A: I use `nmcli` (NetworkManager CLI) commands to configure connection settings:
`nmcli connection modify eth0 ipv4.addresses 192.168.1.50/24 ipv4.gateway 192.168.1.1 ipv4.method manual`
I then restart the connection: `nmcli connection up eth0`.

**Q96: Explain the difference between public and private SSH keys.**
A: The **private key** is kept secure on the client machine and must not be shared. The **public key** is uploaded to the target server's `~/.ssh/authorized_keys` file. The server uses the public key to encrypt a challenge that only the holder of the matching private key can decrypt.

**Q97: How do you view active network connections and listening ports in Linux?**
A: I run `ss -tulnp` to list TCP and UDP listening sockets along with their process names and PIDs.

**Q98: What is the function of the `/etc/resolv.conf` file?**
A: It defines the DNS nameservers and search domains used by the system resolver to perform name resolution queries.

**Q99: How do you view log entries in real-time in Linux?**
A: I use `tail -f /var/log/messages` or query systemd logs using `journalctl -f`.

**Q100: Explain what a shebang (`#!/bin/bash`) does in a script.**
A: The shebang is the first line of a script. It instructs the kernel loader to execute the script file using the designated shell interpreter path (e.g., `/bin/bash` or `/usr/bin/python3`).

---

### Part 5: Azure Administration (AZ-900 / AZ-104) (Q101 - Q125)

**Q101: Explain the cloud deployment models: IaaS, PaaS, and SaaS.**
A:
- **IaaS (Infrastructure as a Service)**: Azure manages the physical infrastructure, and you manage the OS, middleware, and applications (e.g., Azure VMs).
- **PaaS (Platform as a Service)**: Azure manages the OS and runtime environments, and you manage only the application code (e.g., Azure App Services).
- **SaaS (Software as a Service)**: Azure manages the entire application stack, and you use the software directly (e.g., Microsoft 365).

**Q102: What is the difference between Azure Availability Sets and Availability Zones?**
A: **Availability Sets** protect applications from hardware failures within a single data center by spreading VMs across multiple fault and update domains. **Availability Zones** protect applications from entire data center failures by spreading VMs across physically separate data center facilities within an Azure region.

**Q103: Explain the structure of Azure Resource Groups.**
A: A Resource Group is a logical container used to group and manage related Azure resources (VMs, storage accounts, VNets). All resources must belong to a resource group, but resources can interact across groups.

**Q104: What is the shared responsibility model in cloud computing?**
A: It defines security ownership. Azure is responsible for the security *of* the cloud (physical hosts, datacenter facilities). The customer is responsible for the security of data *in* the cloud (operating systems, network configurations, identity access).

**Q105: Explain the difference between Azure AD (Microsoft Entra ID) and Active Directory Domain Services (AD DS).**
A: AD DS is a traditional, on-premises LDAP directory service that uses Kerberos and NTLM authentication. Microsoft Entra ID is a cloud-based identity and access management service that uses REST APIs and modern protocols like OAuth 2.0, SAML, and OpenID Connect.

**Q106: What are Azure Resource Locks, and what are the two types?**
A: Resource Locks prevent accidental deletion or modification of critical resources. The types are:
- **CanNotDelete**: Users can read and modify the resource, but cannot delete it.
- **ReadOnly**: Users can read the resource, but cannot delete or modify it.

**Q107: Explain how Azure Policy differs from RBAC.**
A: RBAC (Role-Based Access Control) manages *who* has access to resources (e.g., Owner, Contributor, Reader). Azure Policy manages *what* resource properties can be deployed (e.g., enforcing that only specific VM sizes can be created).

**Q108: What are the different redundancy options for Azure Storage?**
A:
- **LRS (Locally Redundant Storage)**: Replicates data three times within a single data center.
- **ZRS (Zone Redundant Storage)**: Replicates data across three Availability Zones within the region.
- **GRS (Geo-Redundant Storage)**: Replicates data to a secondary region located hundreds of miles away.
- **GZRS (Geo-Zone-Redundant Storage)**: Replicates data across local zones and to a secondary region.

**Q109: What is Azure File Sync?**
A: Azure File Sync caches files from an Azure file share on on-premises Windows Servers, combining local performance with cloud scalability.

**Q110: Explain the purpose of Shared Access Signatures (SAS) in Azure Storage.**
A: A SAS provides delegated access to resources in a storage account without exposing the account access keys. It grants restricted access permissions with defined start and expiration times.

**Q111: What is Azure Virtual Network (VNet) Peering?**
A: VNet Peering connects two Azure Virtual Networks, allowing resources in both networks to communicate securely using Microsoft's private network backbone.

**Q112: Explain the purpose of a Network Security Group (NSG).**
A: An NSG filters inbound and outbound network traffic to Azure resources (VMs, subnets). It contains security rules evaluated by priority (lowest number has highest priority).

**Q113: What is Azure Bastion, and why is it used?**
A: Azure Bastion provides secure RDP and SSH access to VMs directly through the Azure Portal over SSL (HTML5 browser), eliminating the need to expose public IP addresses on VMs.

**Q114: Explain the difference between Azure Load Balancer and Application Gateway.**
A: Azure Load Balancer operates at Layer 4 (Transport layer), routing traffic based on IP addresses and ports. Application Gateway operates at Layer 7 (Application layer), supporting routing based on HTTP headers, URL paths, and features a Web Application Firewall (WAF).

**Q115: What is Azure ExpressRoute?**
A: ExpressRoute provides a dedicated, private physical connection between on-premises infrastructure and Azure data centers, bypassing the public internet.

**Q116: How does Azure Monitor collect and structure data?**
A: Azure Monitor collects data into two formats:
- **Metrics**: Numerical values representing system performance parameters (e.g., CPU percentage) collected at regular intervals.
- **Logs**: Records containing structured events, performance counters, and logs queryable using KQL.

**Q117: What is a Log Analytics Workspace?**
A: A Log Analytics Workspace is a storage container used to collect, index, and analyze log data from Azure resources, on-premises systems, and other clouds.

**Q118: Write a simple KQL query to find VM heartbeat failures.**
A:
```kusto
Heartbeat
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| where LastHeartbeat < ago(5m)
```

**Q119: What is Azure Recovery Services Vault?**
A: It is a storage entity in Azure used to manage and store backup copies of VMs, databases, and files, and manage Azure Site Recovery replication.

**Q120: How does Azure Site Recovery (ASR) assist in disaster recovery?**
A: ASR replicates workloads running on physical or virtual machines from a primary site to a secondary location (Azure cloud or alternate data center). If a disaster occurs, workloads failover to the secondary site to maintain availability.

**Q121: What is an ARM (Azure Resource Manager) Template?**
A: An ARM template is a JSON file that defines the infrastructure and configuration details for an Azure deployment, enabling Infrastructure as Code (IaC) automation.

**Q122: What are Azure Availability Zones SLA targets?**
A: Azure provides a 99.99% SLA for virtual machines deployed across two or more Availability Zones within the same region.

**Q123: Explain the concept of serverless computing in Azure.**
A: Serverless computing (e.g., Azure Functions) allows running code on-demand without provisioning or managing underlying servers. You pay only for the compute resources consumed during execution.

**Q124: What is Azure Advisor?**
A: Azure Advisor analyzes resource configurations and usage history to provide recommendations across five categories: Cost, Security, Reliability, Performance, and Operational Excellence.

**Q125: What is the purpose of Microsoft Entra ID Conditional Access?**
A: Conditional Access evaluates signals (such as user group, location, device status) during login to enforce access controls, such as requiring MFA or a compliant corporate device.

---

### Part 6: Microsoft 365 & Intune (Q126 - Q150)

**Q126: What is the difference between MDM and MAM in Microsoft Intune?**
A: MDM (Mobile Device Management) manages the entire device configuration, requiring device enrollment. MAM (Mobile Application Management) manages only corporate applications and data, typically used to secure work data on personal devices (BYOD) without enrolling the hardware.

**Q127: Explain the purpose of Microsoft Teams Guest Access versus External Access.**
A: Guest Access allows adding external users to teams, granting them access to channels, files, and chat histories. External Access (Federation) allows users to search, chat, and call users in other M365 domains, but does not grant them access to local teams or files.

**Q128: How does the OneDrive Sync Client handle files using "Files On-Demand"?**
A: Files On-Demand downloads file metadata (name, size) to the local machine, displaying files as cloud-only place-holders (blue cloud icon). The file contents download to local storage only when a user opens the file.

**Q129: What is Microsoft Autopilot, and what are its key requirements?**
A: Autopilot is a cloud-based service used to customize the device setup experience (OOBE). Requirements include: Windows 10/11 Pro/Enterprise, an active Intune license, Entra ID integration, and the device's hardware hash registered in the Autopilot database.

**Q130: What is the Enrollment Status Page (ESP) in Autopilot?**
A: The ESP displays configuration progress during device setup. It can be configured to block access to the Windows desktop until specified critical applications and configuration profiles have fully installed.

**Q131: Explain the purpose of Intune Compliance Policies.**
A: Compliance Policies check if a device meets defined security requirements (e.g., BitLocker enabled, minimum OS version, password set). If it fails, the device is marked "Not Compliant", which can trigger a Conditional Access block.

**Q132: What are Intune Configuration Profiles, and how do they differ from Compliance Policies?**
A: Configuration Profiles deploy settings to devices (e.g., configuring Wi-Fi, VPNs, lock screens, and disabling USB ports). Compliance Policies only evaluate device health and security status.

**Q133: How does Intune resolve settings conflicts?**
A: If two configuration profiles apply conflicting values for the same setting, the status changes to **Conflict**, and neither setting applies. If a conflict occurs between a configuration profile and a compliance policy, the compliance policy setting always wins.

**Q134: How do you package an application for Win32 deployment in Intune?**
A: I use the **Microsoft Win32 Content Prep Tool** (`IntuneWinAppUtil.exe`) to package the installation files into an encrypted `.intunewin` file, then configure install and uninstall commands and detection rules in the Intune console.

**Q135: What are detection rules in Intune Win32 app deployment?**
A: Detection rules verify that an application installed successfully on the client device. Intune supports checking for a specific file path, checking a registry key, or running a custom PowerShell detection script.

**Q136: Explain the difference between Required, Available, and Uninstall app assignments.**
A:
- **Required**: The app installs automatically in the background on target devices.
- **Available**: The app appears in the Company Portal for manual installation by users.
- **Uninstall**: Automatically removes the app if it is detected on target devices.

**Q137: What is the purpose of App Protection Policies?**
A: App Protection Policies protect corporate data within applications. They prevent users from copying data from managed apps (like Outlook) and pasting it into unmanaged apps (like personal mail or social media).

**Q138: How does Windows Update for Business (WUfB) function in Intune?**
A: WUfB uses Update Rings in Intune to manage the installation of quality and feature updates. Administrators configure deferral periods, active hours, and deadlines to control reboots.

**Q139: What is the difference between Retire and Wipe remote actions in Intune?**
A: **Retire** removes corporate data, applications, and configurations from the device, leaving personal files intact (best for BYOD). **Wipe** performs a full factory reset, erasing all data and partitions on the device.

**Q140: How does Autopilot Reset differ from a standard Wipe?**
A: Autopilot Reset removes local user files, settings, and apps, but retains the device's Entra ID join status and enrollment configurations, preparing the device for the next user.

**Q141: What is Endpoint Analytics in Intune?**
A: Endpoint Analytics measures and scores user experience parameters, such as device startup performance and application reliability, helping administrators identify and resolve performance issues.

**Q142: How do you retrieve client-side logs for Intune troubleshooting on a Windows device?**
A: I run the **Collect Diagnostics** action in the Intune portal to upload system logs, or locate them on the client machine at `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log`.

**Q143: Explain how to configure a dynamic device group for Autopilot devices.**
A: In the Entra Portal, I create a dynamic device group with the query: `(device.devicePhysicalIDs -any _ -contains "[ZTDID]")`. This matches the zero-touch deployment ID assigned to imported Autopilot devices.

**Q144: What are Attack Surface Reduction (ASR) rules?**
A: ASR rules block common malware transmission paths, such as preventing Office applications from creating child processes or blocking credential stealing from `lsass.exe`.

**Q145: What is Microsoft Defender for Office 365 Plan 1 versus Plan 2?**
A: Plan 1 focuses on prevention features like Safe Links, Safe Attachments, and Anti-Phishing protection. Plan 2 adds threat investigation tools, automated investigation and response (AIR), and attack simulation training.

**Q146: What are SharePoint site collections, and why is a flat structure recommended?**
A: Site collections are top-level containers that act as security and management boundaries. A flat structure (making each site its own site collection) is recommended to simplify permissions, migration, and restructure actions compared to nested subsite hierarchies.

**Q147: Explain the DMARC protocol and how it works.**
A: DMARC uses SPF and DKIM checks to verify email authenticity. The domain owner publishes a DMARC record instructing receiving servers on how to handle failures (none, quarantine, or reject) and configure reports.

**Q148: What is a Shared Mailbox in Exchange Online, and does it require a license?**
A: A Shared Mailbox allows multiple users to read and send email from a common address. It does not require a license if it is under 50 GB and does not use features like an archive mailbox.

**Q149: Explain the difference between Litigation Hold and Purview Retention Policies.**
A: **Litigation Hold** preserves all mailbox content indefinitely or for a set duration for legal compliance. **Retention Policies** manage the lifecycle of data across multiple M365 services, retaining or deleting data based on policy rules.

**Q150: What is the Microsoft Purview Compliance Center?**
A: The compliance center is a management portal used to govern data compliance, configure DLP rules, publish retention labels, manage sensitivity classifications, and run eDiscovery investigations.

**Q151: What is SELinux and how does it differ from traditional Linux permissions?**
> A: Traditional Linux permissions (owner, group, others) use Discretionary Access Control (DAC). SELinux enforces Mandatory Access Control (MAC). Under MAC, every process, file, and system resource has a security context (label). A process can only access a file if its context is explicitly allowed by the central policy, even if the process runs as root.
> *Simple words: Traditional permissions (chmod) me file owner permissions set karta hai, jabki SELinux me central kernel policy tay karti hai ki kaunsa process kis file ya port ko access kar sakta hai (chahe user root hi kyun na ho).*

**Q152: How do you check if SELinux is blocking a process, and how do you find the logs?**
> A: I check the current SELinux state using `getenforce`. If it is in `Enforcing` mode, I look for audit messages in `/var/log/audit/audit.log` or `/var/log/messages`. I filter using `grep sealert` or `ausearch -m AVC` (Access Vector Cache) to find violations.
> *Simple words: `/var/log/audit/audit.log` me SELinux violation events save hote hain. Hum `ausearch -m AVC` command run karke blocked events ko view kar sakte hain.*

**Q153: How do you temporarily switch SELinux to permissive mode for troubleshooting?**
> A: I run `sudo setenforce 0` to temporarily switch to Permissive mode. This lets the system allow operations that would be blocked by policies, but logs them. To make it permanent or turn it back on, I modify `/etc/selinux/config` and set `SELINUX=permissive` or `SELINUX=enforcing`.
> *Simple words: `setenforce 0` se temporary permissive (warnings only) and `setenforce 1` se enforcing set hota hai. Permanent change ke liye `/etc/selinux/config` file edit karte hain.*

**Q154: What is the purpose of the `visudo` command?**
> A: `visudo` is used to safely edit the `/etc/sudoers` file. It locks the file to prevent concurrent edits and parses the configuration for syntax errors before saving. If there is a syntax error, it warns the administrator and refuses to save, preventing lockout situations.
> *Simple words: Sudoers file ko manually vi se edit nahi karna chahiye. `visudo` command edit save karne se pehle syntax validity check karta hai taaki root lockouts na ho.*

**Q155: How do you configure a user to run only one specific command (e.g., systemctl restart nginx) using sudo without password?**
> A: In `/etc/sudoers` or a file in `/etc/sudoers.d/`, I add the rule: `username ALL=(root) NOPASSWD: /usr/bin/systemctl restart nginx`. I must use the absolute path of the command.
> *Simple words: User ko specific privilege dene ke liye command ka complete path `/etc/sudoers` configuration rule me `NOPASSWD:` tag ke sath set kiya jata hai.*

**Q156: How do you disable root login and enforce key-based authentication in SSH?**
> A: I edit `/etc/ssh/sshd_config` and set `PermitRootLogin no` and `PasswordAuthentication no`. Then, I append the user's public key to `~/.ssh/authorized_keys` and restart the service: `sudo systemctl restart sshd`.
> *Simple words: `/etc/ssh/sshd_config` me root login and password login ko disable karke key-based login set kiya jata hai aur service reload ki jati hai.*

**Q157: What is auditd in Linux, and how do you track modifications to a critical file like `/etc/passwd`?**
> A: `auditd` is the Linux Audit Daemon. To track file modifications, I write a watch rule: `auditctl -w /etc/passwd -p wa -k passwd_changes` (where `-w` is file path, `-p wa` watches write and attribute changes, and `-k` is a tracking tag).
> *Simple words: `auditd` daemon ke through files change trace hote hain. `auditctl -w` command path use karke hum monitor aur key tags generate kar sakte hain.*

**Q158: What is fail2ban and how does it prevent brute force attacks?**
> A: `fail2ban` reads log files (like `/var/log/secure`) looking for patterns of failure. If an IP fails authentication too many times within a set window, fail2ban automatically updates firewall rules (IPtables/Firewalld) to block that IP for a set duration.
> *Simple words: Fail2ban failure logs monitor karta hai. Limit exceed hone par target IP ko automatically firewall rules insert karke temporary block kar deta hai.*

**Q159: What does a umask of 027 mean when creating files and directories?**
> A: A umask defines the permissions to subtract from default permissions (files defaults: 666, directories: 777). A umask of 027 means:
> * New files get: 666 - 027 = 640 (Owner: rw, Group: r, Others: no access).
> * New directories get: 777 - 027 = 750 (Owner: rwx, Group: r-x, Others: no access).
> *Simple words: Umask default permissions ko override karta hai. `027` umask set karne se system users and groups ke alawa third-party groups (Others) ke liye files aur folders read/write select hone se rokta hai.*

**Q160: How do you disable ping responses (ICMP requests) on a Linux server using sysctl?**
> A: I add the parameter `net.ipv4.icmp_echo_ignore_all = 1` inside `/etc/sysctl.d/99-security.conf` and apply the changes immediately by running `sudo sysctl --system`.
> *Simple words: `/etc/sysctl.d/` folder me config build karke kernel parameter `net.ipv4.icmp_echo_ignore_all = 1` set karte hain taaki ping ignore ho.*

**Q161: What is WSUS and what role does it play in enterprise patch management?**
> A: Windows Server Update Services (WSUS) is a role in Windows Server that allows administrators to download, test, and approve Microsoft product updates centrally before deploying them to client machines, saving internet bandwidth and testing updates for stability.
> *Simple words: WSUS client computer updates centralize control karta hai. Iski help se bandwidth bachti hai aur updates ko production me distribute karne se pehle test aur approve kiya ja sakta hai.*

**Q162: What is the difference between client-side targeting and server-side targeting in WSUS?**
> A: In **server-side targeting**, administrators manually move computers into groups in the WSUS console. In **client-side targeting**, Group Policy Objects (GPO) or Registry settings on the client machines automatically assign them to specific groups (e.g., Test Group or Production Group).
> *Simple words: Server-side me computers ko WSUS console se manually group me daala jata hai. Client-side me GPO ke through client automatically apne target group me sync ho jata hai.*

**Q163: How do you configure client computers to report to a WSUS server using Group Policy (GPO)?**
> A: I configure the GPO path: `Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Windows Update`. I enable **Specify intranet Microsoft update service location** and set the URL to my WSUS server (e.g., `http://wsus-server.company.local:8530`).
> *Simple words: Domain Group Policy me Windows Update section ke under 'Specify intranet Microsoft update service location' set karke WSUS Server IP aur Port define kiya jata hai.*

**Q164: What is the default port used by WSUS for HTTP and HTTPS client communication?**
> A: Since Windows Server 2012, the default ports are **8530** for HTTP and **8531** for HTTPS.
> *Simple words: WSUS update downloads aur reporting ke liye target port `8530` (HTTP) ya `8531` (HTTPS) default use karta hai.*

**Q165: A Windows client is not showing up in the WSUS Console. How do you troubleshoot this?**
> A: I check client network connectivity to the WSUS server port using `test-netconnection`. If connected, I reset the client's WSUS authorization configuration by running:
> ```powershell
> Stop-Service wuauserv
> Remove-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate" -Name "AccountDomainSid", "PingID", "SusClientId" -ErrorAction SilentlyContinue
> Start-Service wuauserv
> wuauclt /detectnow /reportnow
> ```
> *Simple words: WSUS ID mismatch hone par, Registry se updates settings (`SusClientId`) delete karke windows update service update cycle force launch kiya jata hai.*

**Q166: What is a WSUS downstream server and how do replica vs autonomous modes differ?**
> A: A downstream server fetches updates from an upstream WSUS server.
> * **Replica Mode**: Downstream server inherits approved updates and target groups directly from the upstream server.
> * **Autonomous Mode**: Downstream server downloads update files but allows local admins to approve updates and configure group memberships independently.
> *Simple words: Replica mode upstream server ke approvals ko exact copy karta hai, jabki Autonomous mode updates downstream local admin ko approvals validate karne ki autonomy deta hai.*

**Q167: How do you clean up disk space on a WSUS server when the update directory is full?**
> A: I run the **WSUS Server Cleanup Wizard** from the WSUS console or via PowerShell: `Get-WsusServer | Invoke-WsusServerCleanup -CleanupObsoleteComputers -CleanupObsoleteUpdates -CleanupUnneededReplacedUpdates -DeclineExpiredUpdates -DeclineSupersededUpdates`.
> *Simple words: Disk full hone par WSUS update clean operations, obsolete logs delete, aur superseded (outdated) patches ko cleanup command ke through flush karte hain.*

**Q168: What is Dual Scan in Windows Update and why is it problematic for WSUS environments?**
> A: Dual Scan occurs when a client is configured to get updates from both WSUS and Windows Update online. When enabled, client machines bypass WSUS approvals and pull patches directly from Microsoft servers, creating configuration drift.
> *Simple words: Dual Scan configuration me client WSUS approval instructions chor kar directly Microsoft cloud update check karne lagta hai, jisse unapproved updates download ho jate hain.*

**Q169: What is Patch Tuesday and Out-of-Band (OOB) updates?**
> A: **Patch Tuesday** is the second Tuesday of every month when Microsoft regularly releases security and quality patches. **Out-of-Band (OOB) updates** are critical patches released out of schedule to resolve zero-day vulnerabilities or major bugs.
> *Simple words: Patch Tuesday har month ke second Tuesday ko standard patches release hone ki tradition hai, jabki OOB patches emergency critical security fixes hote hain.*

**Q170: How do you force a Windows client to immediately check for updates and report status?**
> A: On the client, I open PowerShell as administrator and run:
> ```powershell
> usoclient StartScan
> # Or using the newer cmdlets:
> Get-WindowsUpdate -Install -AcceptAll
> ```
> *Simple words: Client side se immediately scan status check force karne ke liye `usoclient StartScan` command execute kiya jata hai.*

**Q171: What is a Log Analytics Workspace in Azure?**
> A: A Log Analytics Workspace is a logical storage container in Azure where log data from virtual machines, containers, databases, and application resources is consolidated, structured, and queried using KQL.
> *Simple words: Log Analytics Workspace cloud telemetry aur system event logs ko store karne ka main database repository hai, jise KQL queries se access kiya jata hai.*

**Q172: What is Kusto Query Language (KQL) and how is it used in Azure Monitor?**
> A: KQL is a structured query language used to analyze, filter, and aggregate log data. In Azure Monitor, it is used to search logs, build dashboard charts, and configure Alert rules based on log thresholds.
> *Simple words: KQL data querying and filtering language hai jo log tables se values pull karne, metrics analyze karne aur alerting set karne ke kaam aata hai.*

**Q173: How does the Azure Monitor Agent (AMA) differ from the legacy Log Analytics agent (MMA/OMS)?**
> A: The legacy agent had separate settings for metrics and logs. The modern **Azure Monitor Agent (AMA)** uses Data Collection Rules (DCR) to centrally define what logs to collect from virtual machines (Windows/Linux), supports scoping targets, and scales better.
> *Simple words: Legacy MMA agent out-of-date ho chuka hai. New AMA agent Data Collection Rules (DCR) use karta hai jo centralized system logs collection controls define karte hain.*

**Q174: What is a Data Collection Rule (DCR) in Azure Monitor?**
> A: A DCR is an Azure resource that defines what data should be collected (e.g., Windows Event Logs, Syslogs, Performance Metrics), how it should be transformed, and where it should be sent (e.g., Log Analytics Workspace).
> *Simple words: DCR ek rule set hai jo monitor agents ko batata hai ki VMs se kaunse specific events collect karne hain aur kis target workspace me transfer karne hain.*

**Q175: Write a KQL query to find all Virtual Machines in a Log Analytics Workspace that have experienced a CPU usage greater than 90% in the last 24 hours.**
> A:
> ```kql
> Perf
> | where TimeGenerated > ago(24h)
> | where CounterName == "% Processor Time"
> | summarize MaxCPU = max(CounterValue) by Computer
> | where MaxCPU > 90
> ```
> *Simple words: Query last 24 hours me `% Processor Time` counter inspect karti hai aur un virtual machines (Computer) ke names return karti hai jinka CPU usage > 90% raha ho.*

**Q176: What is the difference between Azure Monitor Metrics and Azure Monitor Logs?**
> A: **Metrics** are numerical values representing system performance at a point in time (e.g., CPU %, Disk write rates). They are stored in a time-series database and are near-real-time. **Logs** are records containing detailed event data, traces, and performance strings, optimized for complex analytics queries.
> *Simple words: Metrics numerical value streams hote hain jo speed/graphs ke kaam aate hain. Logs multi-column record tables hote hain jisme detailed event details store hoti hain.*

**Q177: How do you configure an alert in Azure Monitor when a specific log event (e.g., Event ID 4625 - Failed Logon) occurs?**
> A: I create an Alert Rule in Azure Monitor, select my Log Analytics Workspace as target, set the condition query to look for Event ID 4625:
> ```kql
> Event | where EventID == 4625
> ```
> I define the threshold (e.g., Count > 5), assign a search frequency, and link an **Action Group** to send email alerts or trigger logic apps.
> *Simple words: Log monitor alert set karne ke liye query likhte hain jo target error ID check kare. Failures count trigger limit match hone par Action Group notification system launch karta hai.*

**Q178: What is an Action Group in Azure Monitor?**
> A: An Action Group is a collection of notification preferences (email, SMS, push alerts) and automation triggers (runbooks, webhooks, Azure Functions, Logic Apps) defined by the administrator to run when an alert fires.
> *Simple words: Action Group ek template hai jo batata hai ki alert hone par system ko kya karna hai (jaise admin ko mail bhejna ya vm restart playbook launch karna).*

**Q179: How do you collect Syslog from on-premises Linux servers into Azure Log Analytics Workspace?**
> A: I install the Azure Monitor Agent (AMA) on the Linux server, configure a Data Collection Rule (DCR) targeting Linux Syslog facilities (e.g., auth, daemon, syslog), and bind the rule to the Log Analytics Workspace.
> *Simple words: Linux servers par AMA agent install karke DCR rules set karte hain, jo local syslog files ko secure HTTPS protocol ke through Azure Workspace me load karta hai.*

**Q180: What is Azure Monitor Workbooks?**
> A: Workbooks are interactive report documents in the Azure Portal that combine text, analytics queries, metrics graphs, and logs data into a single visual dashboard format for analysis.
> *Simple words: Workbooks interactive charts aur graphical dashboards hote hain jo KQL query calculations ko visual format me dynamically represent karte hain.*

**Q181: Explain the concept of Idempotency in Ansible.**
> A: Idempotency means that executing a playbook multiple times on a target machine will always yield the same result, and changes are only applied if the target's current state differs from the playbook configuration, avoiding repetitive work or corrupted files.
> *Simple words: Idempotency ka matlab hai playbook ko bar-bar run karne par target server ka status change nahi hoga agar desirable state pehle se active ho.*

**Q182: How does Ansible's agentless design operate differently from tools like Puppet or Chef?**
> A: Puppet and Chef require a client agent software constantly running on client machines to fetch changes. Ansible is agentless; it uses SSH (Linux) or WinRM/SSH (Windows) to connect, push transient script commands, and clean them up afterward.
> *Simple words: Puppet/Chef me nodes par agent runs active rehte hain. Ansible agentless hai aur targets par standard SSH/WinRM protocol ke through directly interact karta hai.*

**Q183: What is the default port and configuration required for Ansible to connect to a Windows Server node?**
> A: Ansible uses WinRM. The default ports are **5985** (HTTP - requires basic auth configurations) or **5986** (HTTPS - encrypted, certificate authenticated).
> *Simple words: Windows automation ke liye WinRM protocol port `5985` (unencrypted) ya `5986` (HTTPS encrypted) default use karta hai.*

**Q184: How do you store and use sensitive credentials (passwords, private keys) securely in Ansible?**
> A: I use **Ansible Vault** to encrypt sensitive files: `ansible-vault encrypt vars.yml`. During execution, I provide the vault password: `ansible-playbook -i hosts playbook.yml --ask-vault-pass`.
> *Simple words: Secrets aur credentials files ko secure rakhne ke liye `ansible-vault` tool se variable files ko encrypt kar diya jata hai.*

**Q185: Explain the structure and purpose of an Ansible Playbook.**
> A: Playbooks are configuration files written in YAML. They define a list of plays. Each play targets specific host groups and lists tasks. Each task calls a specific module (e.g., `apt`, `service`, `win_user`) to verify or modify target state.
> *Simple words: Playbooks YAML files hoti hain jisme target hosts aur key modules configuration details define ki jati hain.*

**Q186: What are Ansible Roles and how do they help in scaling automation?**
> A: Roles are structured directory templates used to organize playbooks. Instead of putting all tasks in one file, roles split them logically into `tasks`, `vars`, `handlers`, `templates`, and `defaults` subdirectories, making playbooks reusable and maintainable.
> *Simple words: Roles playbooks ko modular parts (tasks, variables, templates) me divide karne ke standard folders structure rules hain jisse large code clean rehta hai.*

**Q187: How does the `lineinfile` module differ from the `template` module in Ansible?**
> A: `lineinfile` is used to add, edit, or delete a single specific line in an existing configuration file on the target host. The `template` module replaces the entire file with a dynamically parsed file generated from a Jinja2 template file.
> *Simple words: `lineinfile` targeted files me sirf single line update/delete karta hai. `template` pure file config ko dynamic variables inputs se generate karke overwrite karta hai.*

**Q188: What is an Ansible Inventory file?**
> A: An inventory file lists the IP addresses or domain names of the target nodes. It groups nodes logically (e.g., `[web_servers]`, `[db_servers]`) and defines host-specific variable parameters.
> *Simple words: Inventory target hosts ki list details store karne wali host database settings file hai.*

**Q189: How do you verify the syntax of an Ansible Playbook without executing it on the targets?**
> A: I run `ansible-playbook playbook.yml --syntax-check` to validate code structure, and `ansible-playbook playbook.yml --check` (Dry Run) to see what changes would be made.
> *Simple words: Bina command run kiye syntax test karne ke liye `--syntax-check` run karte hain. Dry run ke liye `--check` flag use kiya jata hai.*

**Q190: What is the purpose of Handlers in Ansible playbooks?**
> A: Handlers are special tasks that only run when triggered by a `notify` directive from another task. They are typically used to restart a service (e.g., restart Nginx) only if a configuration file change actually occurs.
> *Simple words: Handlers auxiliary tasks hote hain jo notify trigger hone par hi run karte hain, jaise config change hone par system check ya service restart command chalana.*

**Q191: An alert indicates that a production Windows File Server's local file share is rapidly renaming and encrypting files. What is your immediate technical course of action?**
> A:
> 1. **Identify and Contain**: I run a quick command to identify active SMB sessions accessing the file server shares. I identify the source computer ID.
> 2. **Network Isolation**: I immediately disable the network adapter (NIC) of that specific client workstation via command/Intune, or disable its network port at the switch to stop the spread.
> 3. **Service Halt**: I stop the Server service (`LanmanServer`) on the file server immediately to halt SMB activity.
> 4. **Eradication**: Identify the ransomware binary on the patient zero workstation, scan VM folders, and restore file backups once the threat is eradicated.
> *Simple words: Sabse pehle file share lock commands run karke and client session verify karke source user network connection disconnect kiya jata hai. File server on-going file lock hone se bachane ke liye Server service stop kar di jati hai.*

**Q192: A critical alert informs you that an Administrator account in Entra ID has signed in successfully from an unusual country, and MFA was bypassed. Walk me through the containment procedure.**
> A:
> 1. **Account Lock**: I immediately disable the administrator user account using Entra portal or PowerShell `Set-MgUser -UserId [id] -AccountEnabled $false`.
> 2. **Token Revocation**: I revoke all active OAuth session tokens using `Revoke-MgUserSignInSession` to force disconnect the attacker from all sessions (browser, Outlook, etc.).
> 3. **Vulnerability Scan**: Check Entra audit logs for any newly created user accounts, global admin role assignments, or app registrations done by the attacker during the session.
> 4. **MFA Audit**: Delete any newly registered MFA devices or authentication methods set during the compromise.
> *Simple words: Admin account ko block kar dete hain. `Revoke-MgUserSignInSession` command run karke all active logins clear karte hain aur audit logs check karke attacker ke add-on config changes delete karte hain.*

**Q193: Your company receives an email from an external security researcher claiming that a database backup file is exposed publicly on an Azure Storage Blob. How do you verify and secure the data?**
> A:
> 1. **Investigate URL**: I check the storage account blob endpoint details.
> 2. **Revoke Access**: In the Azure Portal or via PowerShell, I change the Storage Account Blob access level from **Public** or **Blob** access to **Private (no anonymous access)** immediately.
> 3. **Assess Scope**: Rotate all Shared Access Signature (SAS) tokens and connection strings linked to that storage container in case they were compromised.
> 4. **Audit Logs**: Query the Azure Storage Diagnostic logs to analyze what IPs downloaded files from the public blob to evaluate exfiltration impact.
> *Simple words: Storage blob configurations open access settings check karke instantly Private set karte hain. SAS tokens change kiye jate hain aur audit logs read karke data leak range analyze karte hain.*

**Q194: You notice an employee's computer is generating abnormal DNS requests to known command-and-control (C2) servers. How do you respond?**
> A:
> 1. **Host Isolation**: I isolate the host from the corporate network immediately using the Microsoft Defender for Endpoint console or Microsoft Intune console to prevent lateral movement.
> 2. **Log Review**: Audit DNS cache logs and running processes using `Get-Process` or MDE telemetry to locate the source process causing the DNS queries.
> 3. **Remediation**: Kill the process, locate the persistent registry startup entries, delete the executable, and run a complete offline Next-Generation Antivirus scan.
> *Simple words: Laptop network link disconnect (isolate) karte hain. Trace logs run karke process identity target karte hain jo malware process C2 traffic generate kar raha ho aur execute delete clean up karte hain.*

**Q195: An attacker compromise has been confirmed inside the local Active Directory domain. What is the importance of resetting the KRBTGT account password, and how should it be executed?**
> A:
> 1. **Significance**: The `krbtgt` account encrypts all Kerberos ticket-granting tickets (TGT) in the domain. If an attacker gains domain admin rights, they can dump the krbtgt hash and create a "Golden Ticket" for persistent, permanent access, even if all administrator passwords are changed.
> 2. **Execution**: The password must be reset **twice** to invalidate old golden tickets, because AD stores the last two passwords for replication sync stability. I must wait for replication between the resets to avoid breaking authentication.
> *Simple words: KRBTGT account Kerberos tickets signatures control karta hai. Attackers domain access control maintain rakhne ke liye iska misuse (Golden Ticket) kar sakte hain. Is security leak ko fix karne ke liye krbtgt password reset tool run karke system ko update kiya jata hai.*

---

## Lab — Step by Step
> [!info] Lab Setup Needed
> A web browser and access to a quiz platform (e.g., Anki, Quizlet) or a Markdown reader to self-test using these questions.

### Step 1: Create a Self-Study Flashcard Deck
Copy the questions of interest into flashcard files or import this markdown file directly into your study planner tool.

### Step 2: Use the "Active Recall" Study Method
1. Read a question from the list.
2. Formulate your answer without looking at the text.
3. Compare your response with the provided answer, focusing on technical terminology (e.g., checking if you mentioned "Escrowing" for BitLocker or "CAM table" for switches).

### Step 3: Verify with Systems
Verify technical answers on active systems (e.g., run `dsregcmd /status` on a Windows client to confirm your understanding of Entra ID join states).

---
## Commands Reference
```powershell
# Commands to verify AD system status during study
Get-ADDomainController                                # Verifies local DC parameters
Get-Host                                              # Verifies current PowerShell engine version
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- Problem: An interviewer asks a scenario question regarding an AD replication failure, but you experience "brain freeze" and cannot recall the exact commands or steps.
- Root Cause: Interview stress blocking recall.
- Fix:
  1. Take a breath and use the **STAR method** to structure your response.
  2. Start with a high-level troubleshooting strategy: "To troubleshoot replication issues, I start by verifying network connectivity between Domain Controllers using ping and test-port commands."
  3. Mention diagnostic tools: "I then run `repadmin /showrepl` and check the Event Viewer Directory Services log to identify the failing partition."
  4. Explain resolution: "Once identified, I force replication or correct DNS resolve errors."

**Scenario 2:**
- Problem: The interviewer asks a deep technical question about an OS kernel feature you have not worked with directly.
- Root Cause: Knowledge gap.
- Fix:
  1. Do not guess or make up an answer.
  2. Frame your response around how you would find the information: "I have not configured that specific kernel parameter directly, but I know it relates to resource allocation. In a production scenario, I would consult the official documentation and test configurations in a lab environment before deploying to production."

---
## Common Mistakes
> [!warning] Avoid These
> Providing brief, one-word answers during technical interviews. Express your thought process and mention the specific technical terms (e.g., talk about "inodes" when explaining links).
> Guessing technical answers when you are unsure. Interviewers can identify incorrect information quickly. Admit knowledge gaps and explain how you would research the solution.
> Neglecting scenario-based questions (STAR format). Technical skills are important, but resolving issues under pressure is a critical sysadmin skill.

---
## Pro Tips
> [!tip] Field Experience
> When answering troubleshooting questions, always start by checking the logs. Interviewers look for systematic diagnostic skills rather than immediate fixes.
> Relate your answers to business impact. For example, when explaining why you configure BitLocker, mention that it protects proprietary client data in the event of hardware loss.

---
## Quick Revision Table
| # | Category | Core Concept | Key Diagnostic Tool |
|---|----------|--------------|---------------------|
| 1 | Hardware | POST Failure | Motherboard Debug LEDs / Beep codes |
| 2 | Networking | ARP Resolution | `arp -a` / NDP Multicast |
| 3 | Win Server | AD Replication | `repadmin /showrepl` / `dcdiag` |
| 4 | Linux | Service Failures | `systemctl status` / `journalctl` |
| 5 | Cloud / MDM| Policy Sync | Company Portal Sync / Graph API |

---
## Related Notes
- [[00-MOC/REF-01 Complete Command Cheat Sheet|REF-01 Complete Command Cheat Sheet]] — Lists the command syntax referenced in these answers.
- [[00-MOC/REF-03 Troubleshooting Decision Trees|REF-03 Troubleshooting Decision Trees]] — Visualizes the diagnostic flows discussed in the scenarios.
- [[00-MOC/REF-04 Lab Setup Guide|REF-04 Lab Setup Guide]] — Details how to configure the test environments used to verify these concepts.
