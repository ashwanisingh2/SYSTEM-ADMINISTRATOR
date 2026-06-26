---
tags: [desktop-support, azure, cloud, L2]
aliases: [az104-05-azure-load-balancing, az104-05]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#advanced` `#az-104`

# AZ104-05: Azure Load Balancing and Hybrid Connectivity

> [!abstract] Overview
> Yeh note Azure ke load balancing solutions aur hybrid connectivity options cover karta hai. L4 Load Balancer, L7 Application Gateway, Azure Front Door, VPN Gateways aur ExpressRoute ka comparison aur use cases samjhaye gaye hain.

---
## 🧠 Concept Overview

- **What it is** — Traffic ko multiple servers par distribute karne ke tools aur on-prem network ko Azure se securely connect karne ke tareeqe.
- **Why it matters** — High availability, fault tolerance aur secure cross-premises connectivity ensure karne ke liye.
- **Where you see this** — Jab traffic zyada ho aur application ko scale karna ho, ya head office ko cloud VNet se jodna ho.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Load balancer frontend IPs check karna aur basic health probe status dekhna. |
| **L2** | Backend pool modify karna, App Gateway rules update karna, VPN tunnel status check karna. |
| **L3** | Multi-region load balancing design karna, ExpressRoute architecture banana. |

> [!tip] Seedha Simple Mein
> *L4 Load balancer ek simple traffic police hai jo seedha rasta batata hai, jabki App Gateway (L7) ek smart receptionist hai jo aapka kaam samajh ke specific department mein bhejta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Load Balancing** is like managing customer queues in a **Massive Bank Branch** because...
>
> - **Azure Load Balancer (Layer 4)** is a queue helper at the entrance. They route you to the next available teller based on your ticket, without reading your documents.
> - **Application Gateway (Layer 7)** is a specialized receptionist. They open your file, read the request (e.g., `/loans`), route you accordingly, and check for weapons (WAF).
> - **Azure Front Door** directs users globally to the nearest regional branch.
> - **VPN Gateway** is a secure armored car transit link connecting your home branch to HQ.

---
## 🔬 Technical Deep Dive

### 1. Azure Load Balancer (Layer 4)

> [!info] Key Concept
> Operates at the transport layer (TCP/UDP). It distributes traffic across healthy VM instances.

- **Components:** Frontend IP, Backend Pool, Health Probe, Load Balancing Rules.
- **Classifications:** Public (internet to VNet) aur Internal (VNet to VNet).

> [!danger] Common Mistake
> Agar health probe fail ho raha hai toh Load Balancer uss VM par traffic nahi bhejega. Hamesha ensure karein ki OS firewall probe port allow kar raha hai.

### 2. Azure Application Gateway (Layer 7)

- **URL-Path Based Routing:** `/video/*` ko ek pool aur `/images/*` ko dusre pool mein bhejna.
- **SSL Termination:** Decrypts HTTPS at gateway.
- **WAF (Web App Firewall):** Protects against SQL injection, XSS.

### 3. Global Load Balancing: Front Door vs Traffic Manager

- **Azure Front Door:** Global L7 LB with SSL offloading & CDN for HTTP/HTTPS. Uses MS private edge network.
- **Traffic Manager:** DNS-based L4 load balancer. Directs traffic based on DNS resolution (Performance, Priority, Geo).

### 4. Hybrid Connectivity

- **VPN Gateway (Site-to-Site & Point-to-Site):** Encrypted tunnels over the public internet.
- **ExpressRoute:** Dedicated private physical connection (Microsoft Peering & Private Peering).

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Two web VMs (`vm-web-01`, `vm-web-02`) running in same availability set.

### Step 1: Create a Public Load Balancer

```bash
az network lb create -g rg-lab-infrastructure -n lb-web-prod --sku Standard
```

1. Portal > Load balancers > **+ Create**.
2. Type: **Public**, SKU: **Standard**.
3. Create new Frontend IP named `pip-lb-web`.

### Step 2: Configure Backend Pools and Probes

1. Open `lb-web-prod` > **Backend pools** > **+ Add**. Add VMs `vm-web-01` and `vm-web-02`.
2. **Health probes** > **+ Add**. Protocol: TCP, Port: 80, Interval: 5s.

### Step 3: Create Load Balancing Rule

1. **Load balancing rules** > **+ Add**.
2. Frontend IP: `lb-frontend-ip`, Backend pool: `web-backend-pool`, Probe: `http-health-probe`.
3. Protocol TCP, Port 80, Backend Port 80.

> [!success] Expected Output
> Accessing the Public IP in browser will alternate traffic between `vm-web-01` and `vm-web-02`.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az network lb list` | List load balancers | `az network lb list -g rg-lab-infrastructure` |
| `az network vpn-gateway show` | Check VPN Gateway status | `az network vpn-gateway show -g rg -n myVpnGW` |
| `Resolve-DnsName` | Check Traffic Manager DNS resolution | `Resolve-DnsName mytm.trafficmanager.net` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| LB not routing traffic to VM | Health probe failing | Check if web service is running and OS firewall allows probe port. |
| App Gateway 502 Bad Gateway | Backend servers unreachable or invalid SSL | Check NSG on backend subnet and verify VM health in App Gateway pool. |
| S2S VPN disconnected | Pre-shared key mismatch or routing issue | Verify PSK matches on both on-prem firewall and Azure VPN Gateway. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Website Down behind App Gateway

> [!example] Ticket
> "Users are getting 502 Bad Gateway when accessing the company portal."

**L1 Response:** Verify App Gateway frontend IP is active and basic ping/DNS is working.
**Escalation Trigger:** If backend health shows unhealthy.
**L2 Resolution:** Check Backend Health metrics. SSH into backend VM and ensure Nginx/IIS is running. Verify NSG allows traffic from App Gateway subnet.

---
## 🎤 Interview Questions

> [!question] Q1: What is the main difference between Azure Load Balancer and Application Gateway?
> **Answer:** Azure Load Balancer is Layer 4 (TCP/UDP) and routes purely on IP/Port. Application Gateway is Layer 7 (HTTP/HTTPS), capable of URL-path routing, SSL termination, and WAF.

> [!question] Q2: When would you use Traffic Manager instead of Front Door?
> **Answer:** Traffic Manager is a DNS-level router supporting non-HTTP traffic (like UDP/TCP), while Front Door is strictly for global HTTP/HTTPS traffic with CDN caching.

==**Exam Tip:** Standard Load Balancer supports backend pools across availability zones, whereas Basic SKU does not.==

---
## 🔗 Related Notes

- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — Understanding DNS resolution and NAT port forwarding.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Creating VMs and scale sets.
- [[04-Cloud-and-Security/08-Azure/AZ104-04 Azure Virtual Networking|AZ104-04 Azure Virtual Networking]] — VNet design and Gateway subnets.
