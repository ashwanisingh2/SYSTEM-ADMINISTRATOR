---
tags: [networking, wireless, wifi, 802-11, ccna]
aliases: [wireless-networking]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #ccna
---

> [!NOTE|color-yellow]
> 🌐 **NETWORKING**

`#complete` `#intermediate` `#ccna`

# N-11: Wireless Networking

> [!abstract] Overview
> This note covers the fundamentals of Wireless Local Area Networks (WLANs) governed by the IEEE 802.11 standards. It details frequency bands (2.4 GHz, 5 GHz, 6 GHz), channels, security protocols (WPA2/WPA3 Personal & Enterprise), controller-based architectures, and diagnostic operations.

---
## 🧠 Concept Overview

- **What it is** — Wireless networking (Wi-Fi) uses radio waves to connect client devices (laptops, phones) to the local network without physical copper or fiber cabling, transmitting data over the air.
- **Why it matters** — In modern workspaces, almost all user endpoints connect via Wi-Fi. A support engineer must diagnose why connections drop, why speed slows down, or why authentication fails on corporate networks.
- **Where you see this** — Troubleshooting laptops dropping off corporate Wi-Fi, setting up conference room access points, configuring guest Wi-Fi isolation, or managing Wireless LAN Controllers (WLCs).

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Assist users in joining SSIDs, verify the Wi-Fi physical switch/adapter is enabled, clear saved network profiles, and verify signal bars. |
| **L2** | Install and configure standalone Access Points, assign SSIDs to specific switchport VLANs, set up security keys (WPA2/WPA3 Personal), and select non-overlapping channels. |
| **L3** | Design central Wireless LAN Controller (WLC) architectures, integrate WPA2/WPA3 Enterprise (802.1X) with RADIUS/NPS, perform RF heat-mapping site surveys, and optimize fast-roaming thresholds (802.11r). |

> [!tip] Seedha Simple Mein
> *Wireless networking (Wi-Fi) radio waves ke zariye bina wires ke devices ko connect karta hai. Isme 2.4 GHz (dono dur tak jata hai par speed kam hoti hai) aur 5 GHz (speed tez par range kam hoti hai) bands use hote hain. Office mein central controller aur access points use kiye jate hain network manage karne ke liye.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> Think of Wireless Access Points (APs) as **FM Radio Stations**:
> 
> - **SSID** is the station name (e.g., "98.3 Radio Mirchi"). Users tune their device to this name.
> - **Frequency Bands (2.4 GHz vs. 5 GHz)**: 2.4 GHz is like **AM radio** (travels very far, passes through concrete walls easily, but has low audio quality and is crowded with static/noise). 5 GHz is like **FM radio** (high-fidelity sound, very fast speeds, but struggles to go far or pass through thick walls).
> - **Channels**: If two nearby radio stations try to broadcast on the exact same frequency, they cause scratchy static interference. Access points must use separate, non-overlapping channels to avoid jamming each other.

---
## 🔬 Technical Deep Dive

### 1. IEEE 802.11 Standards Comparison

| Standard | Release Year | Frequency Bands | Max Data Rate | Common Name |
|---|---|---|---|---|
| **802.11b** | 1999 | 2.4 GHz | 11 Mbps | Legacy |
| **802.11g** | 2003 | 2.4 GHz | 54 Mbps | Legacy |
| **802.11n** | 2009 | 2.4 GHz, 5 GHz | 600 Mbps | **Wi-Fi 4** |
| **802.11ac** | 2013 | 5 GHz | 6.93 Gbps | **Wi-Fi 5** |
| **802.11ax** | 2019 | 2.4 GHz, 5 GHz, 6 GHz | 9.6 Gbps | **Wi-Fi 6** (6 GHz is **6E**) |
| **802.11be** | 2024 | 2.4 GHz, 5 GHz, 6 GHz | 46 Gbps | **Wi-Fi 7** |

### 2. Channels and Radio Frequency (RF) Interference
- **2.4 GHz Band**: Only has **three non-overlapping channels**: **1, 6, and 11**. Each channel is 20 MHz wide. Using any other channels (like channel 3 or 4) causes **Adjacent-Channel Interference** because frequencies overlap and collide.
- **5 GHz Band**: Features up to 25+ non-overlapping 20 MHz channels. Allows channel bonding (combining channels to 40/80/160 MHz width for faster speed), but bonding reduces the number of available independent channels.
- **Interference Types**:
  - **Co-Channel Interference (CCI)**: Multiple APs operating on the same channel in the same area. They wait for each other to stop talking, slowing down the link.
  - **Non-Wi-Fi Interference**: Microwave ovens, baby monitors, and Bluetooth devices operating in the unlicensed 2.4 GHz band.

```
       Channel 1             Channel 6            Channel 11
   /---------------\     /---------------\     /---------------\
  /                 \   /                 \   /                 \
 /                   \ /                   \ /                   \
+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 |12 |13 |14 | 2.4 GHz Band
+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   \___Overlapping___/ \___Overlapping___/
```

### 3. Wi-Fi Security Protocols
- **WEP (Wired Equivalent Privacy)**: Legacy, insecure, uses RC4 cipher, easily cracked in minutes. **Never use.**
- **WPA (Wi-Fi Protected Access)**: Temporary fix, uses TKIP. Insecure.
- **WPA2**: Uses **AES (Advanced Encryption Standard)** encryption with **CCMP**. Secure, but vulnerable to offline dictionary attacks if users capture the 4-way handshake.
- **WPA3**: Replaces WPA2's pre-shared key exchange with **SAE (Simultaneous Authentication of Equals)**, which is immune to offline dictionary attacks. Requires Protected Management Frames (PMF).
- **Personal (PSK) vs. Enterprise (802.1X)**:
  - **Personal**: One password (Pre-Shared Key) for everyone. If an employee leaves, the password should be changed on all devices.
  - **Enterprise (802.1X)**: Centralized authentication. Clients enter their personal Active Directory username and password (or use a certificate). APs check with a RADIUS server (NPS/ISE) to authorize access.

### 4. WLAN Architecture
- **Autonomous APs**: Independent devices. Each must be configured individually. Not scalable.
- **Controller-Based (Lightweight APs - LAP)**: APs do not make processing decisions. They establish a secure **CAPWAP (Control and Provisioning of Wireless Access Points)** tunnel to a central **Wireless LAN Controller (WLC)**. The WLC configures all APs, manages channels, security, and client roaming dynamically.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A client machine (Windows or Linux) with a wireless network adapter.
> - Access to a local router or standalone access point management portal (e.g., standard SOHO device at `192.168.1.1` or virtual Cisco WLC environment).

### Step 1: Check Wireless Interface Status via Command Line

Identify your wireless adapter profile and connection details using native tools.

**On Windows:**
1. Open Command Prompt as Administrator and view wireless adapter details:

```cmd
netsh wlan show interfaces
```

> [!success] Expected Output
> ```text
> There is 1 interface on the system:
>     Name                   : Wi-Fi
>     Description            : Intel(R) Wi-Fi 6 AX201
>     State                  : connected
>     SSID                   : Corporate_WiFi
>     BSSID                  : 00:0a:95:9d:68:16
>     Radio type             : 802.11ax
>     Channel                : 36
>     Receive rate (Mbps)    : 1201
>     Transmit rate (Mbps)   : 1201
>     Signal                 : 99%
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `netsh wlan show interfaces` | Displays local wireless card state, signal, and SSID | `netsh wlan show interfaces` |
| `netsh wlan show profiles` | Lists all saved Wi-Fi networks on Windows | `netsh wlan show profiles` |
| `netsh wlan delete profile name="SSID"` | Deletes a saved Wi-Fi network profile | `netsh wlan delete profile name="Guest_WiFi"` |
| `netsh wlan show profile name="SSID" key=clear` | Displays the saved WPA2 pre-shared key password | `netsh wlan show profile name="HomeNetwork" key=clear` |
| `iwconfig` | Linux legacy wireless tool to check link configuration | `iwconfig wlan0` |
| `nmcli dev wifi connect "SSID" password "key"` | Connects to a Wi-Fi network via Linux CLI | `nmcli dev wifi connect "Guest_WiFi" password "12345678"` |
| **802.11r** | Fast Roaming standard; enables fast transit between APs | Enabled on WLC profile properties |
| **CAPWAP** | Protocol used by LAPs to communicate with the WLC | UDP Port 5246 (Control) & 5247 (Data) |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Client device can see the SSID but fails to obtain an IP address (stuck at "Obtaining IP..."). | VLAN mapping mismatch or DHCP scope is exhausted for that VLAN. | Verify WLC WLAN-to-VLAN interface mapping. Check DHCP server scope to ensure available IPs exist in the pool, and confirm the switchport trunk allows the VLAN. |
| Corporate Wi-Fi connection drops when walking between rooms (failed roaming). | AP overlap coverage is poor, or Fast Roaming (802.11r/k/v) is disabled on the controller. | Enable 802.11r Fast Transition on the WLC. Perform RF site survey. Ensure AP coverage overlap is around 15% to 20% at -67 dBm signal level. |
| Wireless signal shows full bars, but internet access is extremely slow and packets drop. | Co-channel interference or high channel utilization from overlapping APs. | Change channel settings. Move APs off overlapping channels (use only 1, 6, or 11 in 2.4 GHz). Reduce AP transmit power to limit overlapping coverage. |
| 802.1X Enterprise Wi-Fi rejects client connection with credential prompt looping. | Radius/NPS Server certificate expired, or client machine time is out of sync. | Verify and renew the NPS Server SSL Certificate. Ensure client system time matches Domain Controller time within 5 minutes (Kerberos constraint). |
| Modern WPA3 client cannot connect to a legacy AP. | The older access point hardware does not support WPA3 security ciphers. | Configure WPA3-Transition Mode on the AP to support both WPA2 and WPA3 clients concurrently, or upgrade AP firmware/hardware. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: WPA3-Enterprise Connection Issue

> [!example] Ticket
> "Explain how WPA3-Enterprise secures connections differently compared to WPA2-Personal, and outline the sequence of events when a client connects using 802.1X."

**L1 Response:** Verify that the client is trying to connect using valid AD credentials.
**Escalation Trigger:** If the issue involves certificate errors or deep protocol mismatches.
**L2 Resolution:** WPA3-Enterprise replaces PSK with individual user authentication via the 802.1X framework and utilizes AES-256 bit encryption (GCMP) with Protected Management Frames (PMF). The connection sequence starts with Discovery, then EAP-Request Identity, Relay to RADIUS, Authentication with NPS/AD, Accept/Derive Keys, and finally Access Granted to the switch port.

---
## 🎤 Interview Questions

> [!question] Q1: A user reports that their laptop has suddenly disconnected from the Wi-Fi and cannot see any networks. What are your initial troubleshooting steps?
> **Answer:** 1. Check if the laptop's physical Wi-Fi switch is toggled off, or if Airplane Mode is enabled. 2. Open Network Connections and verify the Wireless Network Adapter is enabled, not disabled. 3. Restart the wireless adapter. 4. Check if other nearby devices can see the Wi-Fi network to isolate whether it's a device-specific or AP outage.

==**Exam Tip:** The 2.4 GHz band operates from 2.400 GHz to 2.4835 GHz, and each standard channel requires 20 MHz of bandwidth. Always use non-overlapping channels 1, 6, and 11 to avoid Adjacent-Channel Interference.==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-06 Switching — VLANs and Trunking|N-06 Switching — VLANs and Trunking]] — Mapping SSIDs to switch VLAN interfaces.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-17 NPS and RADIUS Server|WS-17 NPS and RADIUS Server]] — Authenticating 802.1X Enterprise clients.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Dynamically assigning IPs to connected wireless clients.
