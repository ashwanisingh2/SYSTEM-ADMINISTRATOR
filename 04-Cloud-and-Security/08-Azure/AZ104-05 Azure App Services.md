---
tags: [azure, app-services, cloud, compute]
aliases: [Azure Web Apps, App Service]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE / CLOUD**

`#complete` `#advanced` `#az-104`

# AZ104-05: Azure App Services

> [!abstract] Overview
> Azure App Service is a fully managed Platform as a Service (PaaS) offering that enables developers to build, deploy, and scale web apps, REST APIs, and mobile backends without managing the underlying infrastructure. *As a system administrator or cloud engineer, aapko yeh pata hona chahiye kyunki aajkal har company apne legacy servers se migrate karke PaaS solutions pe aa rahi hai. Aapka kaam in apps ki networking, scaling, security, aur troubleshooting manage karna hoga.*

---
## 🧠 Concept Overview

- **What it is** — A managed hosting service for web applications. You just bring your code (in .NET, Java, Node.js, Python, PHP) or a Docker container, and Azure takes care of patching, load balancing, and scaling the servers.
- **Why it matters** — It removes the operational overhead of managing IIS or Apache on a Linux/Windows VM. *Company ka time aur paisa bachta hai kyunki OS level patch management aur VM maintenance ka sir-dard Azure handle karta hai.*
- **Where you see this** — Hosting corporate websites, internal API backends, or any customer-facing web application that needs high availability and autoscaling.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Check app service status, review basic metrics (CPU/Memory), restart the web app, check deployment logs. |
| **L2** | Configure custom domains, SSL certificates, backup schedules, scaling rules, set up VNet Integration and Private Endpoints. |
| **L3** | Architecture, design App Service Environments (ASE), disaster recovery planning, CI/CD pipeline integration, advanced traffic routing. |

> [!tip] Seedha Simple Mein
> *Azure App Service ek aisi readymade dukaan (shop) hai jahan aapko sirf apna saamaan (code) rakhna hai. Dukaan ki safaai, security, aur bijli (infrastructure) ka dhyaan mall owner (Azure) rakhta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Living in a Managed Apartment Complex** is like **Azure App Service (PaaS)** because...
>
> - You only care about decorating the inside of your flat (writing your code).
> - The building management takes care of plumbing, security guards, and elevator maintenance (OS updates, load balancers, underlying server health).
> - Compare this to owning an independent house (IaaS VM) where you have to fix the roof and paint the walls yourself.

---
## 🔬 Technical Deep Dive

### 1. App Service Plans (ASP)

> [!info] Key Concept
> An App Service Plan defines the set of compute resources for a web app to run. Think of it as the invisible server farm behind your web app.

When you create an App Service, you must assign it to an App Service Plan. The plan determines:
- Operating System (Windows or Linux)
- Region (East US, Central India, etc.)
- Number of VM instances (Scale out)
- Size of VM instances (Small, Medium, Large)
- Pricing Tier (Free, Shared, Basic, Standard, Premium, Isolated)

*Ek App Service Plan ke andar aap multiple web apps host kar sakte ho. Agar plan ka CPU 80% pe hai, toh uske andar ke saare web apps affect honge.*

> [!danger] Common Mistake
> Placing multiple high-traffic production web apps in a single App Service Plan to save costs. If one app consumes all memory, all other apps in the same plan will experience "503 Service Unavailable" errors. *Ek app ki wajah se baaki sab down ho jayenge.*

### 2. Scaling Methods

There are two ways to scale an App Service:
- **Scale Up (Vertical Scaling):** Upgrading the underlying VM size (e.g., from 2 Cores/4GB RAM to 4 Cores/8GB RAM) or moving to a higher pricing tier to get features like Custom Domains or VNet Integration.
- **Scale Out (Horizontal Scaling):** Adding more identical instances of the VM to handle load. Azure automatically load balances traffic across them.

> [!tip] Pro Tip
> Use **Autoscale rules** based on metrics like CPU percentage (e.g., add 1 instance if CPU > 70% for 5 mins) rather than manual scaling. *System ko itna smart banao ki traffic badhne par automatically naye servers add ho jayein aur traffic kam hone par remove ho jayein.*

### 3. Deployment Slots

> [!info] Key Concept
> Deployment slots are live apps with their own hostnames. App content and configurations elements can be swapped between two deployment slots, including the production slot.

This is critical for Zero-Downtime deployments. 
*Aap ek 'staging' slot banate ho. Naya code usme dalte ho, test karte ho, aur phir ek button click se 'production' aur 'staging' ko swap kar dete ho. Agar naya code phat gaya, toh waapis swap kar do (Rollback).*

### 4. Networking

- **Inbound Traffic:** By default, it's public. You can secure it using **Access Restrictions** (allow only specific IPs) or by creating a **Private Endpoint** (gives the app a private IP in your VNet).
- **Outbound Traffic:** By default, App Service cannot talk to resources inside your Azure VNet (like an internal database VM). You need to enable **VNet Integration**. *VNet integration allows the App Service to act like it's inside your internal network for outbound calls.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Azure Subscription
> - Owner or Contributor role in a Resource Group
> - Azure CLI installed or Azure Cloud Shell access

### Step 1: Create a Resource Group and App Service Plan

```bash
# Set variables
RG_NAME="RG-AppService-Lab"
LOCATION="eastus"
PLAN_NAME="ASP-Linux-Standard"

# Create Resource Group
az group create --name $RG_NAME --location $LOCATION

# Create App Service Plan (Linux, Standard S1 tier)
az appservice plan create --name $PLAN_NAME \
  --resource-group $RG_NAME \
  --location $LOCATION \
  --sku S1 \
  --is-linux
```

> [!success] Expected Output
> ```json
> {
>   "id": "/subscriptions/.../resourceGroups/RG-AppService-Lab/providers/Microsoft.Web/serverfarms/ASP-Linux-Standard",
>   "name": "ASP-Linux-Standard",
>   "provisioningState": "Succeeded",
>   "status": "Ready"
> }
> ```

### Step 2: Create the Web App and Deploy Code

```bash
# Create a unique web app name
APP_NAME="myapp-$RANDOM"

# Create the Web App (using Node.js 18 LTS)
az webapp create --name $APP_NAME \
  --resource-group $RG_NAME \
  --plan $PLAN_NAME \
  --runtime "NODE:18-lts"

# Check the URL of the created app
echo "Your app is running at: https://$APP_NAME.azurewebsites.net"
```

*Ab aap is URL pe jayenge toh default Azure ka page dikhega. Matlab aapka infra ready hai code receive karne ke liye.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az webapp list` | Lists all web apps in the subscription. | `az webapp list --output table` |
| `az webapp stop` | Stops the running web app (saves compute cost only if plan is free/shared, otherwise you still pay for the plan). | `az webapp stop -g MyRG -n MyApp` |
| `az webapp restart` | Restarts the app service (First troubleshooting step). | `az webapp restart -g MyRG -n MyApp` |
| `az webapp log tail` | Starts live log streaming in the console. *Real-time error dekhne ke liye.* | `az webapp log tail -g MyRG -n MyApp` |
| `az webapp up` | Quick command to create plan, app, and deploy code from local folder in one shot. | `az webapp up --name myfirstapp` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| HTTP 500 Internal Server Error | Code issue, missing dependencies, or bad startup command. | Check Application Logs using Log Stream. Go to Kudu console (Advanced Tools) and check file structure. |
| HTTP 503 Service Unavailable | High CPU/Memory on the App Service Plan, or app is stopped. | Check App Service Plan metrics. Scale up the plan temporarily to stabilize, or restart the app. |
| HTTP 502 Bad Gateway / Gateway Timeout | App took too long to start or crash loop. | Check container logs (if Docker). Increase `WEBSITES_CONTAINER_START_TIME_LIMIT` app setting. |
| Cannot connect to internal Azure SQL Database | App Service is not part of the internal network. | Enable **VNet Integration** on the App Service and ensure NSGs on the SQL subnet allow traffic from the App Service subnet. |
| Custom Domain showing "Not Secure" | SSL binding is missing. | Go to TLS/SSL settings, upload a PFX certificate, and add an SNI binding for the custom domain. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Performance Degradation Alerts

> [!example] Ticket
> "Critical Alert: CPU usage for ASP-Prod-EastUS has been > 90% for the last 30 minutes. Website is responding very slowly."

**L1 Response:** 
Check the Azure Portal metrics for the App Service Plan. Verify which specific web app inside the plan is consuming the CPU. Attempt to restart the offending web app if authorized.

**Escalation Trigger:** 
If a restart doesn't drop the CPU, or if the traffic is legitimate and the server is genuinely overwhelmed.

**L2 Resolution:** 
Temporarily "Scale Out" by adding an extra instance to handle the immediate load. Investigate the Application Insights to find the specific slow database query or API call causing the spike, and pass the report to the dev team.

### 🎫 Scenario 2: Deployment Failed / Site Broke After Update

> [!example] Ticket
> "Dev team just deployed Release v2.4 to Production but now users are seeing a 500 Error. Need immediate rollback."

**L1 Response:** 
Acknowledge ticket. Navigate to the Web App -> Deployment Slots. 

**Escalation Trigger:** 
If slots were not used for deployment, or if the dev team overwrote the code directly.

**L2 Resolution:** 
If Deployment Slots were used: Simply click "Swap" to swap Production back with the old code in the Staging slot. 
If slots were NOT used: Navigate to the "Deployment Center", find the previous successful deployment commit, and trigger a redeploy of that specific older commit.

### 🎫 Scenario 3: Whitelisting External Client IP

> [!example] Ticket
> "Our B2B partner needs to access our staging API endpoint, but they are getting a 403 Forbidden error."

**L1 Response:** 
Ask the client for their public outgoing IP address or CIDR range. 

**Escalation Trigger:** 
L1 gets the IP but doesn't have permissions to modify network rules.

**L2 Resolution:** 
Go to Web App -> Networking -> Access Restrictions. Add an "Allow" rule with the client's provided IP address and set an appropriate priority. *Sirf wahi IP traffic bhej payega, baaki sab block ho jayenge.*

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between Scale Up and Scale Out in Azure App Service?
> **Answer:** Scale Up (Vertical) means increasing the CPU/RAM of the existing VM or changing the pricing tier (e.g., from Basic to Standard). Scale Out (Horizontal) means increasing the number of VM instances running the app to distribute load.

==**Exam Tip:** Scale Up requires a restart of the app and causes a brief downtime, whereas Scale Out does not cause downtime as new instances are added behind the load balancer.==

> [!question] Q2: How can an Azure App Service securely connect to an Azure SQL Database on a private IP without going over the internet?
> **Answer:** By using **Regional VNet Integration**. This gives the App Service outbound access to the VNet where the SQL Database resides. For inbound security to the SQL DB, we configure a Private Endpoint on the SQL Server.

> [!question] Q3: What happens if an App Service Plan is deleted?
> **Answer:** You cannot delete an App Service Plan if it still contains running Web Apps. You must first delete or move all the associated Web Apps to another plan before the plan itself can be deleted.

> [!question] Q4: How do Deployment Slots enable zero-downtime deployments?
> **Answer:** Deployment slots run on the same App Service Plan but have different hostnames. You deploy new code to a staging slot, warm it up, and then perform a "Swap". Azure simply switches the IP routing rules at the load balancer level between the production and staging slots instantly, without restarting the underlying worker processes.

> [!question] Q5: Your web app keeps restarting and returning 502 errors. How do you find the underlying OS-level or container-level startup errors?
> **Answer:** We can check the Application Logs, enable Log Stream, or use the Kudu console (Advanced Tools) to inspect the specific boot logs and event viewer files located in the virtual directory `/LogFiles`.

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ104-01 Introduction to Cloud|AZ104-01 Introduction to Cloud]] — Basics of PaaS vs IaaS
- [[04-Cloud-and-Security/08-Azure/AZ104-06 Azure Virtual Networks|AZ104-06 Azure Virtual Networks]] — Helps understand VNet Integration concepts
- [[04-Cloud-and-Security/08-Azure/AZ104-09 Azure Private Link|AZ104-09 Azure Private Link]] — How to secure inbound traffic to PaaS
