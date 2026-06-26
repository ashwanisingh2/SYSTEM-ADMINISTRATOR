---
tags: [networking, security, firewall, vpn, ccna]
aliases: [network-security]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

# N-12: Network Security

**Verification:**
- [ ] Configure network zones (DMZ, Internal, Public) on a firewall interface
- [ ] Implement switch-level port security restricting MAC address assignments
- [ ] Configure stateful firewall rules permitting defined application ports
- [ ] Establish a secure remote access VPN tunnel using IPsec or SSL/TLS protocols
- [ ] Verify defensive features (DHCP Snooping, DAI) to prevent network spoofing

> [!abstract] Overview
> This note covers foundational network security frameworks. It details security zone designs, firewall types (Stateless, Stateful, NGFW), Intrusion Detection and Prevention Systems (IDS/IPS), IPsec/SSL VPNs, and essential Layer 2 LAN hardening parameters.

---
## Concept Overview
- **What it is** — Network Security consists of the policies, processes, and technologies implemented to protect a network infrastructure and its transmitted data from unauthorized access, misuse, modification, or exposure.
- **Why it matters for a support engineer** — Network perimeters are constantly scanned by malicious actors. As a support engineer, you must ensure that only authorized traffic enters the local network, and that internal network resources are segmented and secured against local attacks.
- **Where you encounter this in real job** — Configuring firewall rules for a new application server, restricting access to a database VLAN, setting up VPN profiles for remote employees, or troubleshooting switch ports shutting down due to security violations.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - **L1 Resolution**: Verify user VPN connection status, guide users through MFA prompts, verify local client firewall is active, and check physical cables.
  - **Escalation Trigger**: Escalate to L2 if legitimate application traffic is blocked by firewall ACLs, or if switchports are systematically disabling due to MAC limit violations.
  - **L2 Resolution**: Author stateful firewall rules (allowing specific ports), configure site-to-site IPsec VPN tunnels, deploy NAT, configure switch port security, and inspect firewall traffic logs.
  - **L3 Resolution**: Design network segmentation architectures (DMZ, Internal, Guest, IoT), implement enterprise NAC (Network Access Control) policies, deploy IDS/IPS engines, and configure anti-spoofing security (DHCP Snooping, DAI, IPSG).

*Seedha simple mein: Network Security ka matlab hai network infrastructure ko secure rakhna. Isme firewall (jo entry point par packets check karta hai), VPN (jo encrypted tunnel banata hai), aur switch security (jo check karti hai ki switchport se kaun sa system connected hai) jaise concepts aate hain.*

---
## Real-World Analogy
Think of a **bank building**:
- **Firewall** is the security gate checking visitor badges at the entrance. It only lets in people on the pre-approved guest list (Access Control Lists).
- **DMZ (Demilitarized Zone)** is the public lobby. Anyone can walk in to talk to the receptionist (Web Server), but they cannot walk into the back office vaults (Database/Application Servers).
- **IDS/IPS (Intrusion Detection/Prevention)** is the security guard walking around the building. If they see someone trying to pick a door lock, they sound the alarm (IDS) and tackle the intruder (IPS).
- **VPN (Virtual Private Network)** is an armored cash-transport truck. It drives on public roads, but its contents are locked, hidden, and protected from onlookers.

---
## Technical Deep Dive

### 1. Network Security Zoning
Network segmentation prevents an attacker from moving laterally (sideways) if they compromise one machine.
- **Intranet (Internal LAN)**: High trust zone containing internal corporate resources, Domain Controllers, and databases.
- **DMZ (Demilitarized Zone)**: Semi-trusted zone containing public-facing servers (Web, Mail, DNS). If a web server in the DMZ is hacked, firewall rules block the hacker from accessing the Intranet.
- **Extranet**: Private network segment shared with trusted external vendors or partners.
- **Internet**: Untrusted public network.

```
       [ Internet ] (Untrusted)
            |
      [ Outer Firewall ]
            |
      [ DMZ Zone ] (Web/Mail Servers - Semi-trusted)
            |
      [ Inner Firewall ]
            |
      [ Intranet Zone ] (Active Directory/DB - Trusted)
```

### 2. Firewall Technologies
Firewalls filter traffic based on security rules:
- **Stateless (Packet Filtering)**: Inspects packets individually. Checks Source IP, Destination IP, Protocol, and Port against an ACL. It does not remember past packets, making it fast but insecure (can be bypassed by spoofed return packets).
- **Stateful Inspection**: Tracks the state of active network connections (using a state table). It remembers that an internal client initiated a connection to an external site, and automatically allows the return traffic. Much more secure.
- **Next-Generation Firewall (NGFW)**: Inspects the actual application payload (Deep Packet Inspection - DPI). It goes beyond ports to identify the application (e.g., distinguishing between Skype file transfer vs. standard Skype chat) and includes built-in antivirus/IPS.

### 3. VPN Technologies: IPsec vs. SSL/TLS
- **IPsec (IP Security) VPN**:
  - Operates at the Network Layer (Layer 3).
  - Standard for Site-to-Site tunnels (connecting two branch offices).
  - Uses two main protocols: **AH (Authentication Header)** for integrity/auth, and **ESP (Encapsulating Security Payload)** for encryption.
  - Negotiates connections in two phases using **IKE (Internet Key Exchange)**.
- **SSL/TLS VPN**:
  - Operates at the Transport/Application Layer (Layer 4/7).
  - Standard for Remote Access (connecting a single remote laptop to the office).
  - Runs over standard HTTPS port 443, making it easy to pass through home firewalls and web proxies.

### 4. Layer 2 LAN Infrastructure Hardening
Attackers can connect to a wall jack and attack the local network. Layer 2 defenses block these vectors:
- **Switch Port Security**: Restricts the MAC addresses allowed to connect to a physical switch port. It can shutdown the port if an unauthorized MAC is detected (Violation).
- **DHCP Snooping**: Switch monitors DHCP traffic. It marks ports facing the real DHCP server as **Trusted** and client ports as **Untrusted**. If an attacker plugs in a rogue DHCP server on an untrusted port, the switch drops the DHCP replies.
- **Dynamic ARP Inspection (DAI)**: Prevents ARP Spoofing/Poisoning (Man-in-the-Middle attacks). The switch intercepts all ARP requests and replies on untrusted ports and verifies the MAC-to-IP binding against the database built by DHCP Snooping.

---

### Enterprise Networking & Wireless Analysis

#### 1. Wireshark Packet Capture Filtering
Use these filters during packet capture analysis to isolate issues quickly:
- **HTTP POST Request Error isolation**: `http.request.method == "POST" && http.response.code >= 400`
- **Isolate TCP Retransmissions (Packet Loss)**: `tcp.analysis.retransmission || tcp.analysis.duplicate_ack`
- **DNS Server response failures check**: `dns.flags.response == 1 && dns.flags.rcode != 0`
- **Isolate host traffic (excluding noise)**: `ip.addr == 192.168.1.50 && !arp && !dns`

#### 2. Wireless Troubleshooting (Enterprise Wi-Fi)
- **802.1X EAP Authentication Failed**: Verify certificate validation settings on RADIUS server (NPS); check client identity store configuration.
- **Roaming failure (sticky client)**: Adjust Minimum RSSI settings on wireless access points (APs) or check that both APs broadcast identical SSIDs with overlapping coverage ranges (15-20% overlap).
- **RF Interference**: Run a channel survey; move corporate APs from saturated 2.4GHz bands to clean 5GHz/6GHz channels using 20MHz or 40MHz channel widths.

#### 3. Software-Defined WAN (SD-WAN) Basics
- **What it is**: SD-WAN decouples the network control plane from the physical hardware forwarding plane. It manages WAN links (MPLS, Broadband, LTE) dynamically.
- **Why it matters**: It automatically routes business-critical traffic (like VoIP) over the lowest latency links while routing general web traffic over cheap broadband, using real-time link quality metrics (jitter, packet loss, latency).

---
## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - A Rocky Linux / RHEL 9 VM acting as a secure server gateway.
> - Access to the command line with root/sudo rights.
> - `firewalld` package installed and active: `sudo systemctl enable --now firewalld`.

### Step 1: Configure Stateful Firewall Zones on Linux Gateway
We will move the network interfaces into proper zones and write secure rules.

1. View the default active zone and assigned interfaces:
```bash
sudo firewall-cmd --get-active-zones
```
**Expected Output:**
```text
public
  interfaces: eth0 eth1
```

2. Move `eth0` (connected to WAN/Internet) to the **external** zone:
```bash
sudo firewall-cmd --zone=external --change-interface=eth0 --permanent
```
3. Move `eth1` (connected to internal LAN) to the **internal** zone:
```bash
sudo firewall-cmd --zone=internal --change-interface=eth1 --permanent
```
4. Reload the firewall configurations:
```bash
sudo firewall-cmd --reload
```

---

### Step 2: Establish Service Permissions (Port Filtering)
We will allow only secure web (HTTPS) and a custom SSH port (TCP 2222) on the external interface, blocking all other incoming requests.

1. Add HTTPS service access to the external zone:
```bash
sudo firewall-cmd --zone=external --add-service=https --permanent
```
2. Add custom SSH port 2222 access to the external zone:
```bash
sudo firewall-cmd --zone=external --add-port=2222/tcp --permanent
```
3. Remove default SSH service (port 22) from the external zone to prevent brute-force attacks:
```bash
sudo firewall-cmd --zone=external --remove-service=ssh --permanent
```
4. Reload firewall configuration and verify active rules:
```bash
sudo firewall-cmd --reload
sudo firewall-cmd --zone=external --list-all
```
**Expected Output:**
```text
external (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: https
  ports: 2222/tcp
  protocols: 
  forward-ports: 
  source-ports: 
  masquerade: yes
```

---

### Step 3: Configure Switch Port Security (Cisco IOS Concept)
Secure a physical switchport to allow only a single authorized client device.

1. Connect to the switch console and enter configuration mode:
```text
Switch# configure terminal
Switch(config)# interface gigabitethernet 0/1
```
2. Enable port security:
```text
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
```
3. Set the maximum number of allowed MAC addresses to `1`:
```text
Switch(config-if)# switchport port-security maximum 1
```
4. Configure the switch to learn the client MAC address automatically and save it to running-config (Sticky MAC):
```text
Switch(config-if)# switchport port-security mac-address sticky
```
5. Set the violation action to shut down the port if an unauthorized MAC attempts connection:
```text
Switch(config-if)# switchport port-security violation shutdown
```
6. Exit configuration and verify status:
```text
Switch(config-if)# end
Switch# show port-security interface gigabitethernet 0/1
```
**Expected Output:** `Port Security: Enabled`, `Port Status: Secure-up`, `Violation Mode: Shutdown`.

---
## Cheat Sheet

| Command / Setting | Purpose | Example |
|---|---|---|
| `firewall-cmd --state` | Checks if local firewalld service is active | `firewall-cmd --state` |
| `firewall-cmd --get-zones` | Lists all available pre-defined zones | `firewall-cmd --get-zones` |
| `firewall-cmd --reload` | Reloads firewall configurations without closing active links | `firewall-cmd --reload` |
| `firewall-cmd --panic-on` | Immediately drops all incoming/outgoing network packets (emergency) | `firewall-cmd --panic-on` |
| `firewall-cmd --panic-off` | Restores normal firewall operation | `firewall-cmd --panic-off` |
| `iptables -L -n -v` | Lists legacy iptables rules in numeric format with packet statistics | `iptables -L -n -v` |
| `show port-security` | Cisco IOS command displaying active port security features | `show port-security` |
| `show ip dhcp snooping` | Cisco IOS command verifying DHCP snooping table and port trusts | `show ip dhcp snooping` |

---
## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Client device connects to a wall jack, link light turns green for a second, then goes dark. | Switchport entered `err-disabled` state due to a port security violation (MAC limit exceeded). | Unplug rogue device. Connect original device. In switch CLI, enter the interface configurations and run `shutdown` followed by `no shutdown` to reset the port. |
| User connects to client VPN successfully, but cannot access internal fileservers. | Firewall split-tunneling routing is misconfigured, or internal ACL blocks VPN IP subnet. | Verify client routing table has route for internal subnet pointing to VPN interface. Check firewall security policies to ensure traffic from VPN zone to Internal zone is permitted. |
| DHCP clients fail to obtain IP addresses after enabling DHCP Snooping on the switch. | The uplink port facing the DHCP server was not configured as a trusted port. | Access switch config and set the port facing the DHCP server as trusted: `ip dhcp snooping trust`. |
| Linux service is configured to listen on port 8080, but external clients get connection timeouts. | Local firewalld is blocking TCP port 8080. | Run `sudo firewall-cmd --add-port=8080/tcp --permanent` and `sudo firewall-cmd --reload`. |
| Client cannot access websites, pinging 8.8.8.8 works, but nslookup times out. | Outbound UDP Port 53 (DNS) is blocked by perimeter firewall ACLs. | Add firewall rule permitting outbound UDP port 53 traffic from local subnet to DNS servers. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between a stateless firewall and a stateful firewall?
> **A:** A **stateless firewall** filters network packets individually based solely on information present in the packet header (such as source/destination IP, port, and protocol) without context of previous packets. A **stateful firewall** monitors and records the state of active network connections using a state table. It recognizes when an internal system initiates a connection to an external server and automatically permits the returning traffic, blocking any unsolicited incoming traffic that is not part of an established session.

> [!question] L2 Question
> **Q:** Why is a DMZ (Demilitarized Zone) used, and what type of servers should be placed inside it?
> **A:** A DMZ is a isolated network segment that acts as a buffer zone between the trusted internal corporate network (Intranet) and the untrusted public internet. It is used to host public-facing services (like web servers, mail gateways, and public DNS servers) that must be accessible from the internet. If an attacker compromises a server in the DMZ, the inner firewall rules prevent the attacker from moving laterally into the secure internal network where Domain Controllers and database servers reside.

> [!question] L3/Scenario Question
> **Q:** You are troubleshooting an issue where users on a network switch are experiencing packet drops and redirection. You suspect an ARP Spoofing (Man-in-the-Middle) attack. How do you mitigate this at the Layer 2 level?
> **A:** 
> - **Situation:** Suspected ARP spoofing attack on a local switch network.
> - **Task:** Implement Layer 2 security features to prevent ARP poisoning.
> - **Action:** I will enable **DHCP Snooping** and **Dynamic ARP Inspection (DAI)** on the switches. First, I will configure DHCP Snooping globally and trust the uplink ports connected to the legitimate DHCP server. The switch will dynamically build a DHCP Snooping binding database mapping MAC addresses to IP addresses. Second, I will enable DAI on the user VLAN. The switch will now intercept all ARP requests and replies on untrusted user ports and validate the sender IP and MAC against the binding database.
> - **Result:** Any spoofed ARP replies sent by the attacker will not match the binding database, causing the switch to drop the malicious packets and log the violation, effectively neutralizing the MITM attack.

---
## Related Notes
- [[01-Foundations/02-Networking/N-09 Access Control Lists|N-09 Access Control Lists]] — Deep dive into writing IP traffic access control lists.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-14 VPN and Remote Access (RRAS)|WS-14 VPN and Remote Access (RRAS)]] — Deploying Windows Server-based remote access VPNs.
- [[01-Foundations/02-Networking/N-02 Network Devices Deep Dive|N-02 Network Devices Deep Dive]] — Physical and functional details of switches and routers.

---
*Tags: #networking #security #firewall #vpn #ccna*
