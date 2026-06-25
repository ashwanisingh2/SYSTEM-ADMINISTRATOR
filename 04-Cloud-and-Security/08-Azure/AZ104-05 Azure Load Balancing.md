---
tags: [sysadmin, azure, az-104, load-balancer, high-availability]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# AZ104-05: Azure Load Balancing and Hybrid Connectivity

> [!abstract] Overview
> This note covers Azure traffic routing and hybrid networks. It details Layer 4 Load Balancers, Layer 7 Application Gateways, Global routers (Front Door/Traffic Manager), VPN Gateways, and ExpressRoute circuit routing.

---
## Concept
Think of Azure Load Balancing like managing customer queues in a massive bank branch:
- **Azure Load Balancer (Layer 4)** is a simple queue helper at the entrance. They look at your ticket number (IP address/Port) and quickly route you to the next available teller (backend VM). They don't open your paperwork or care what service you need.
- **Application Gateway (Layer 7)** is a specialized receptionist. They open your file, read the request (URL path, e.g., `/images` vs `/payment`), and route you to the specific department office handling that task. They also check you for weapons (Web Application Firewall - WAF).
- **Azure Front Door** is a global routing coordinator: they direct users to the nearest regional bank branch (East US vs West Europe) based on speed and availability.
- **VPN Gateway** is a secure, private armored car transit link connecting your home branch to corporate headquarters.

*Seedha simple mein: AZ-104 load balancing L4 (Load Balancer) aur L7 (Application Gateway) options offer karta hai. Global routing ke liye Front Door aur DNS-based Traffic Manager use hote hain. Hybrid connections VPN Gateway ya ExpressRoute ke zariye established kiye jaate hain.*

---
## Technical Deep Dive

### 1. Azure Load Balancer (Layer 4)
Operates at the transport layer (TCP/UDP). It distributes incoming traffic across healthy VM instances in a backend pool.
- **Components:**
  - **Frontend IP Configuration:** The incoming public IP or private IP.
  - **Backend Pool:** The group of target VMs configured to receive the traffic.
  - **Health Probe:** Periodically checks VM health (e.g., checks port 80 response every 5 seconds). If a VM fails the probe, the load balancer stops routing traffic to it.
  - **Load Balancing Rules:** Maps the frontend IP and port to the backend pool and port.
- **Classifications:**
  - **Public Load Balancer:** Map public internet traffic to internal VM private IPs.
  - **Internal (Private) Load Balancer:** Distributes traffic within a VNet (e.g., routing traffic from web VMs to database VMs).

### 2. Azure Application Gateway (Layer 7)
A web traffic load balancer operating at the application layer.
- **Capabilities:**
  - **URL-Path Based Routing:** Routes requests for `example.com/video/*` to one backend pool of VMs, and `example.com/images/*` to another.
  - **SSL Termination:** Decrypts HTTPS traffic at the gateway, reducing CPU overhead on the backend web servers.
  - **WAF (Web Application Firewall):** Protects web apps against common vulnerabilities (SQL injection, cross-site scripting) using OWASP core rule sets.

### 3. Global Load Balancing: Front Door vs. Traffic Manager
- **Azure Front Door:** A global, scalable web entry point that uses Microsoft's private global edge network. It combines L7 load balancing, SSL offloading, and CDN capabilities. Best for HTTP/HTTPS web apps.
- **Azure Traffic Manager:** A DNS-based traffic load balancer. It resolves DNS queries to the closest regional endpoint based on routing methods (Performance, Priority, Geographic). Traffic does not flow *through* Traffic Manager; it only points the client to the best IP.

---

### Load Balancer Decision Flowchart
```
                  [ Is it HTTP/HTTPS Web Traffic? ]
                     /                         \
                   Yes                          No
                   /                             \
     [ Is it Global/Multi-Region? ]        [ Is it Global/Multi-Region? ]
       /                     \                /                     \
     Yes                      No            Yes                      No
     /                         \            /                         \
[ Azure Front Door ]    [ App Gateway ]  [ Traffic Manager ]    [ L4 Load Balancer ]
```

---

### 4. Hybrid Connectivity: VPN Gateway & ExpressRoute
- **VPN Gateway (VNet Gateway):**
  - **Site-to-Site (S2S):** An IPsec VPN tunnel connecting on-prem firewall routers to the Azure VNet Gateway over the public internet. Uses BGP (Border Gateway Protocol) for dynamic routing.
  - **Point-to-Site (P2S):** Connects individual client laptops to the VNet using SSTP or IKEv2. Supports certificate or Microsoft Entra ID authentication.
- **ExpressRoute:**
  - Provides a dedicated private connection via a connectivity provider.
  - **Peering Types:**
    - *Private Peering:* Connects to your private VNets (VMs).
    - *Microsoft Peering:* Connects to public Azure services (Storage, SQL) and Microsoft 365.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Access to the Azure Portal, two web VMs (`vm-web-01` and `vm-web-02`) running in the same availability set on `vnet-hub`.

### Step 1: Create a Public Load Balancer
1. Log into the Azure Portal. Search for and click **Load balancers**. Click **+ Create**.
2. Project Details:
   - Resource Group: `rg-lab-infrastructure`.
   - Name: `lb-web-prod`. Region: **East US**.
   - Type: **Public** | SKU: **Standard**.
3. Frontend IP Configuration: Click **Add a frontend IP**.
   - Name: `lb-frontend-ip`.
   - Public IP address: Create new named `pip-lb-web`. Click Add.
4. Click **Review + create**, then click **Create**.

### Step 2: Configure Backend Pools and Probes
1. Open the new load balancer `lb-web-prod`.
2. Under Settings, click **Backend pools** -> **+ Add**.
3. Name: `web-backend-pool`.
4. Virtual Network: Select `vnet-hub`.
5. Virtual Machines: Click **Add** and select `vm-web-01` and `vm-web-02`. Click Save.
6. Click **Health probes** -> **+ Add**.
7. Name: `http-health-probe`. Protocol: **TCP** | Port: **80** | Interval: **5 seconds**. Click Add.

### Step 3: Create Load Balancing Rule
1. Under Settings, click **Load balancing rules** -> **+ Add**.
2. Configuration:
   - Name: `http-routing-rule`.
   - Frontend IP: Select `lb-frontend-ip`.
   - Backend pool: Select `web-backend-pool`.
   - Health probe: Select `http-health-probe`.
   - Protocol: **TCP** | Port: **80** | Backend port: **80**.
   - Session persistence: **None** (Allows round-robin load distribution).
3. Click **Add**.
4. **Verify:** Open a web browser on your host machine. Navigate to the Load Balancer's public IP address (`pip-lb-web`). Refresh repeatedly. Confirm that traffic alternates between the responses from `vm-web-01` and `vm-web-02`, showing load balancing is active.

---
## Related Notes
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Understanding DNS resolution and NAT port forwarding.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Creating VMs and scale sets.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — VNet design and Gateway subnets.

