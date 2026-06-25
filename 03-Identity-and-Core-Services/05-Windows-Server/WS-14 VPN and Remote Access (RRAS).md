---
tags: [sysadmin, windows-server, rras, vpn, remote-access]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# WS-14: VPN and Remote Access (RRAS)

> [!abstract] Overview
> This note covers Windows Server Routing and Remote Access Service (RRAS) and Network Policy Server (NPS) integration. It details VPN protocol profiles (PPTP, L2TP, SSTP, IKEv2), certificate authentication, and diagnostic procedures.

---
## Concept
Think of RRAS as a secure customs gate and bridge at the corporate perimeter:
- **Site-to-Site VPN** is a private, armored freight tunnel running between your headquarters and your branch office. All trucks (network traffic) drive through the tunnel automatically; the users don't even know it's there.
- **Remote Access VPN** is a personal secure ID tunnel that a remote employee (working from a cafe) spins up on-demand to connect their laptop directly to the office network.
- **PPTP** is a legacy wooden tunnel (fast but easily cracked).
- **L2TP/IPsec** is a double-walled tunnel requiring a shared security code (Pre-Shared Key).
- **SSTP** is a tunnel disguised as standard web traffic (HTTPS port 443), allowing it to pass through any cafe or airport firewall.
- **NPS (Network Policy Server)** is the security bouncer checking credentials, ensuring the client has the right access card and meets company patch compliance rules.

*Seedha simple mein: RRAS remote employees ko internet ke zariye corporate network se securely connect karne ki facility deta hai. L2TP pre-shared key use karta hai aur SSTP certificate-based connection use karta hai jo firewalls ko easily bypass kar leta hai.*

---
## Technical Deep Dive

### 1. VPN Protocol Comparison

| Protocol | Port / Transport | Encryption | Security Level | Firewall Penetration |
|---|---|---|---|---|
| **PPTP** | TCP 1723 (GRE protocol 47) | MPPE (128-bit) | Low (Vulnerable) | Poor (Blocked by many ISPs/routers) |
| **L2TP / IPsec**| UDP 500, 4500, ESP | AES (256-bit) | High | Poor (Requires IPsec pass-through) |
| **SSTP** | TCP 443 (HTTPS) | SSL / TLS | Very High | Excellent (Looks like normal HTTPS) |
| **IKEv2** | UDP 500, 4500 | AES (256-bit) | Very High | Medium (Supports MOBIKE client roaming) |

### 2. Network Policy Server (NPS) Integration
NPS is Microsoft's implementation of a **RADIUS (Remote Authentication Dial-In User Service)** server.
- **Role:** Centralizes authentication, authorization, and accounting (AAA) for VPN connections.
- **Connection Request Policies:** Define *how* requests are routed (e.g., process locally or forward to another RADIUS server).
- **Network Policies:** Define *who* is allowed access. Conditions include: group membership (e.g., user must belong to `G_VPN_Users`), time limits, and connection type.

---
## Windows Server CLI Configuration Commands

### Installing RRAS and NPS Roles
```powershell
# Install Remote Access, DirectAccess/VPN, and Network Policy Server roles
Install-WindowsFeature -Name RemoteAccess, DirectAccess-VPN, NPAS -IncludeManagementTools
```

### Checking RRAS Active Ports Status
```cmd
:: Windows Command Prompt
:: Check active routing and remote access connection ports
netstat -ano | findstr /i "1723 443 500 4500"
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Windows Server 2022 VM (`SVR-VPN01`) with two virtual NICs:
> 1. Internal NIC (`192.168.10.12` - Corporate LAN)
> 2. External NIC (`203.0.113.10` - Simulated WAN)

### Step 1: Install and Configure RRAS
1. Open Server Manager -> Add Roles and Features. Install **Remote Access** role.
2. Select **DirectAccess and VPN (RAS)**. Complete installation.
3. Open Tools -> **Routing and Remote Access**.
4. Right-click the server name and select **Configure and Enable Routing and Remote Access**.
5. Select **Virtual Private Network (VPN) access and NAT**. Click Next.
6. VPN Connection: Select the **External** interface (`203.0.113.10`). Click Next.
7. IP Address Assignment: Select **From a specified range of addresses**.
8. Address Range Assignment: Click New. Add range: `192.168.10.200` to `192.168.10.220` (these IPs are dynamically assigned to VPN clients). Click Next.
9. Managing Multiple Remote Access Servers: Select **No, use Routing and Remote Access to authenticate connection requests**. Click Next, then **Finish**.

### Step 2: Configure L2TP with a Pre-Shared Key
1. In RRAS console, right-click the server name and select **Properties**.
2. Go to the **Security** tab.
3. Check **Allow custom IPsec policy for L2TP/IKEv2 connection**.
4. Preshared Key: Enter `SecureSecret123!`.
5. Click Apply, then click OK. Click **Yes** to restart the RRAS service.

### Step 3: Grant User Dial-In Permission
1. Open **Active Directory Users and Computers**.
2. Locate the test user `jdoe`. Right-click -> **Properties**.
3. Go to the **Dial-in** tab.
4. Under Network Access Permission, select **Allow access**. Click Apply and OK.

### Step 4: Connect from Windows Client
1. On a client machine connected to the simulated WAN, open Settings -> Network & Internet -> **VPN** -> **Add a VPN connection**.
2. VPN Provider: Windows (built-in).
3. Connection Name: `Office_VPN`.
4. Server Name or IP: `203.0.113.10`.
5. VPN Type: **L2TP/IPsec with pre-shared key**.
6. Pre-shared key: `SecureSecret123!`.
7. Username/Password: `jdoe` and password.
8. Click Save. Select the connection and click **Connect**.
9. **Verify:** Once connected, open CMD on the client. Run `ipconfig`. Verify that the PPP adapter shows an IP in the `192.168.10.200-220` range. Pinging `192.168.10.10` (the DC) should succeed.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Client attempts to connect using SSTP VPN, but the connection fails, returning error: "The revocation function was unable to check revocation because the revocation server was offline (Error 0x80092013)."
- **Root Cause:** The SSTP client is verifying the SSL certificate's revocation status, but the CRL Distribution Point (CDP) HTTP path is unreachable from the client's external WAN connection.
- **Fix:**
  1. If the client is external, verify that the firewall permits HTTP (port 80) traffic to the CA server hosting the CRL.
  2. If the CRL server is down, start the IIS Web service hosting the CDP path.
  3. If a quick fix is needed to bypass revocation checks on a test machine, modify the client registry (Warning: lowers security):
     ```cmd
     reg add HKLM\System\CurrentControlSet\Services\SstpSvc\Parameters /v NoCertRevocationCheck /t REG_DWORD /d 1 /f
     ```

**Scenario 2:**
- **Problem:** Client attempts to connect using L2TP/IPsec VPN, but connection fails, returning error: "Error 789: The L2TP connection attempt failed because the security layer encountered a processing error during initial negotiations."
- **Root Cause:** IPsec service is stopped on the client, or the pre-shared keys do not match, or NAT-T (NAT Traversal) is blocked on intermediate routers.
- **Fix:**
  1. On the client machine, run `services.msc`. Verify that **IKE and AuthIP IPsec Keying Modules** and **IPsec Policy Agent** services are running.
  2. Re-verify that the pre-shared keys are identical on both RRAS and the client properties.
  3. If the client or server is behind a NAT device, enable UDP port 4500 and apply the Windows NAT-T registry fix:
     ```cmd
     reg add HKLM\SYSTEM\CurrentControlSet\Services\PolicyAgent /v AssumeUDPEncapsulationContextOnSendRule /t REG_DWORD /d 2 /f
     :: Reboot client system after setting registry.
     ```

---
## Common Mistakes
> [!warning] Avoid These
> **Using default "Control access through NPS Network Policy" in AD UC without an active NPS server:** Leaving a user's Dial-in permission set to "Control access through NPS" when no NPS policies are configured. This defaults to denying access for all connection attempts.
> **Correct approach:** Explicitly select "Allow Access" in the user's AD properties for simple setups, or set up a dedicated NPS policy to evaluate groups.

---
## Pro Tips
> [!tip] Field Experience
> When hosting a public SSTP VPN, ensure that the SSL certificate bound to IIS/SSTP on the RRAS server is issued by a publicly trusted CA (like Let's Encrypt or DigiCert). If you use a private enterprise Root CA, you must pre-install that Root CA certificate on all external personal devices before they can establish an SSTP tunnel.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | RRAS | Windows routing and VPN perimeter service managing incoming client connections. |
| 2 | SSTP | SSL/TLS VPN operating over standard TCP Port 443; easily penetrates public firewalls. |
| 3 | L2TP/IPsec | Dual-security VPN requiring UDP 500/4500 routing and a pre-shared security key. |
| 4 | NPS / RADIUS | Central authorization engine managing credentials and group checks for VPN requests. |
| 5 | Error 789 | L2TP error indicating security negotiation failure, usually caused by key mismatches. |

---
## Interview Q&A

**Q1: What are the primary operational differences between L2TP/IPsec and SSTP VPN protocols?**
A: **L2TP/IPsec** wraps Layer 2 packets in IPsec tunnels using UDP ports 500 and 4500. It requires a Pre-Shared Key (PSK) or certificates for authentication. It provides high security but struggles to penetrate public firewalls (which often block non-standard UDP ports or lack IPsec pass-through). **SSTP (Secure Socket Tunneling Protocol)** encapsulates PPP traffic inside an SSL/TLS session over TCP Port 443. It requires an SSL certificate on the server. Because it runs over standard HTTPS, it can pass through almost any firewall or proxy server, making it highly reliable for remote workers in hotels or cafes.

**Q2: A client complains that their remote employees cannot connect to the L2TP VPN. You suspect the external firewall is blocking the traffic. Describe which ports you must verify.**
A: 
- **Situation:** Users cannot connect via L2TP VPN, and firewall port block is suspected.
- **Task:** Verify and open the correct IPsec helper ports on the external firewall.
- **Action:** I will access the perimeter firewall configuration. For L2TP/IPsec to function, I must permit: 1) **UDP Port 500** for IKE key negotiations, 2) **UDP Port 4500** for NAT Traversal (if clients are behind home routers), and 3) **IP Protocol 50 (ESP - Encapsulating Security Payload)** to transport encrypted data.
- **Result:** Opening these ports and routing them to the internal RRAS IP restores remote user connections.

**Q3: Explain the role of NPS (Network Policy Server) in securing enterprise VPN access.**
A: NPS acts as a central RADIUS server that handles authentication, authorization, and accounting (AAA) for VPN connection requests. When a user connects to the RRAS server, the server forwards the credentials to the NPS. The NPS checks the request against configured connection request policies and network policies. It verifies if the user belongs to the authorized AD groups, checks if the connection meets time-of-day constraints, and can execute health checks (NAP/Network Access Protection) to verify if the client has active antivirus and security updates before granting access.

---
## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-03 DNS Server — Install and Configure|WS-03 DNS Server — Install and Configure]] — Resolving VPN server hostnames.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-06 Active Directory — Users Groups OUs|WS-06 Active Directory — Users Groups OUs]] — User Dial-in permissions.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-13 Active Directory Certificate Services|WS-13 Active Directory Certificate Services]] — Generating SSL certificates for SSTP.

