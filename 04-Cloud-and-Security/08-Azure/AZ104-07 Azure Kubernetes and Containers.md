---
tags: [azure, containers, kubernetes, aks, acr, aci]
aliases: [AKS, Azure Containers, Azure Kubernetes Service]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #az-104
---

> [!NOTE|color-blue]
> ☁️ **AZURE / CLOUD**

`#complete` `#advanced` `#az-104`

# AZ104-07: Azure Kubernetes and Containers

> [!abstract] Overview
> Azure offers multiple ways to run containers, including Azure Container Instances (ACI) for simple workloads and Azure Kubernetes Service (AKS) for complex, orchestrated microservices. Yeh note cover karta hai containers ke basics, ACR (Azure Container Registry), ACI, aur AKS ke deployment, management, aur troubleshooting. Ek support engineer ke liye AKS clusters aur container issues ko handle karna kaafi crucial skill hai, especially modern cloud-native architectures mein.

---
## 🧠 Concept Overview

- **What it is** — Azure provides PaaS offerings to host and manage Docker containers. ACI is serverless containers, while AKS is a managed Kubernetes orchestration service.
- **Why it matters** — Microservices aur modern apps containers par run hoti hain. AKS removes the complexity of managing Kubernetes control plane manually. *Modern applications monolith se hatkar microservices pe ja rahi hain, isliye containers ko efficiently manage karna zaroori hai.*
- **Where you see this** — Deploying web apps, microservices architectures, CI/CD pipelines, and auto-scaling workloads based on traffic.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Check basic pod status, restart pods, verify ACR connectivity. *Dekhna ki pod chal raha hai ya crash ho gaya.* |
| **L2** | Scale node pools, manage RBAC, troubleshoot ImagePullBackOff or CrashLoopBackOff errors. *Deployment fix karna aur scaling manage karna.* |
| **L3** | Architect AKS networking (CNI), integrate with App Gateway (AGIC), design multi-region container strategies. |

> [!tip] Seedha Simple Mein
> *Container ek tiffin box jaisa hai jismein application aur uski saari dependencies pack hoti hain. ACI ek akele tiffin box ko garm karne ka microwave hai, aur AKS ek poora buffet system hai jo hazaron tiffin boxes ko manage karta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Azure Kubernetes Service (AKS)** is like a **Cruise Ship Manager** because...
>
> - The Containers are the passengers (applications).
> - The Nodes (VMs) are the different cabins where passengers stay.
> - The Kubernetes Control Plane is the Captain and Crew who decide where passengers go, ensure they have enough resources, and handle emergencies (node failures).
> - *Jaise captain ship ko manage karta hai bina passengers ko disturb kiye, waise hi AKS background mein VMs ko manage karta hai taaki apps smoothly chale.*

---
## 🔬 Technical Deep Dive

### 1. Azure Container Registry (ACR)

> [!info] Key Concept
> A managed, private Docker registry service based on the open-source Docker Registry 2.0. Used to store and manage private container images.

- ACR integrates directly with AKS, App Service, and ACI.
- Supports Geo-replication (Premium SKU).
- *Yeh ek private godown (warehouse) hai jahan hum apni custom Docker images ko secure tarike se store karte hain.*

### 2. Azure Container Instances (ACI)

> [!info] Key Concept
> PaaS service that offers the fastest and simplest way to run a container in Azure, without having to manage any virtual machines or adopt a higher-level service.

- Best for simple apps, task automation, and build jobs.
- Billed by the second for vCPU and memory.
- *Jab aapko bas ek container run karna ho bina kisi complex setup ke, tab ACI use karte hain.*

### 3. Azure Kubernetes Service (AKS)

> [!info] Key Concept
> A managed Kubernetes service that simplifies deploying, managing, and operations of Kubernetes.

**Core Components of AKS:**
1. **Control Plane (Managed by Azure):** Free of charge. Includes API Server, etcd, Scheduler, Controller Manager. *Yeh cluster ka dimag hai jo Azure khud manage karta hai.*
2. **Nodes / Node Pools (Managed by Customer):** Azure VMs that run your workloads. You pay for these. *Yeh actual servers hain jahan aapki app chalti hai.*
3. **Pods:** The smallest deployable computing unit in Kubernetes, containing one or more containers. *Yeh actual tiffin box hai.*
4. **Deployments & Services:** Manage pod replicas and provide network access (LoadBalancers, ClusterIP).

> [!danger] Common Mistake
> Exposing AKS API Server to the public internet without IP restrictions. Always use Authorized IP ranges or Private AKS clusters for enterprise environments. *API server ko public rakhna security risk hai.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Azure CLI installed (`az`)
> - `kubectl` installed
> - Contributor access to Azure Subscription

### Step 1: Create an Azure Container Registry (ACR)

```bash
# ACR create karne ki command
az acr create --resource-group myRG --name myRegistry123 --sku Basic
```

> [!success] Expected Output
> ```
> JSON output confirming ACR is created with loginServer URL.
> ```

### Step 2: Create an AKS Cluster and attach ACR

```bash
# AKS cluster create karna aur ACR ko connect karna taaki images pull ho sake
az aks create \
    --resource-group myRG \
    --name myAKSCluster \
    --node-count 2 \
    --generate-ssh-keys \
    --attach-acr myRegistry123
```

> [!success] Expected Output
> ```
> Provisioning state: Succeeded.
> ```

### Step 3: Connect to the AKS Cluster

```bash
# kubectl ko AKS ke credentials dena
az aks get-credentials --resource-group myRG --name myAKSCluster
```

> [!success] Expected Output
> ```
> Merged "myAKSCluster" as current context in ~/.kube/config
> ```

### Step 4: Deploy a Sample App

```bash
# Nginx container run karne ki command
kubectl create deployment nginx --image=nginx
# Service create karke internet pe expose karna
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `az acr login` | Log into ACR locally | `az acr login -n myRegistry123` |
| `az aks get-credentials` | Get kubeconfig for AKS | `az aks get-credentials -g rg -n aks` |
| `kubectl get pods` | List all pods | `kubectl get pods -n kube-system` |
| `kubectl describe pod` | Show detailed pod info | `kubectl describe pod nginx-123` |
| `kubectl logs` | Show container logs | `kubectl logs nginx-123` |
| `kubectl get services` | List network services | `kubectl get svc` |
| `kubectl get nodes` | Check AKS node status | `kubectl get nodes` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **ImagePullBackOff** | Pod image pull nahi kar pa raha. ACR credential issue ya image name galat hai. | Check if AKS has `AcrPull` role on ACR. Verify image tag. `kubectl describe pod <name>` |
| **CrashLoopBackOff** | Container start hota hai aur turant crash ho jata hai. App level error. | Check logs using `kubectl logs <pod-name>`. App ke andar ka error fix karo. |
| **Node NotReady** | AKS node VM hang ho gaya ya kubelet crash ho gaya. | Restart the node VM from Azure Portal or scale node pool. |
| **Pending Pods** | Cluster mein CPU/Memory resources nahi bache hain. | Add more nodes (`az aks scale`) ya pod resources check karo. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Pods stuck in ImagePullBackOff

> [!example] Ticket
> "Our new deployment is failing. The pods are showing ImagePullBackOff status in the default namespace."

**L1 Response:** Use `kubectl describe pod <pod-name>` to see the exact error. Check if the image name and tag are correct. *Check karo ki image ACR mein exist karti hai ya nahi.*
**Escalation Trigger:** If the image exists but access is denied and L1 lacks permissions to check IAM.
**L2 Resolution:** Verify that the AKS cluster's managed identity has the `acrpull` role assignment on the ACR. Re-attach ACR if needed using `az aks update -n <aks> -g <rg> --attach-acr <acr>`.

### 🎫 Scenario 2: Application is slow and scaling issues

> [!example] Ticket
> "During peak hours, our AKS application response time is very high. Looks like it's not scaling."

**L1 Response:** Check pod metrics and node metrics. See if pods are maxing out CPU/Memory. *Check karo resource limit hit ho rahi hai kya.*
**Escalation Trigger:** If manual scaling is needed or HPA (Horizontal Pod Autoscaler) is failing.
**L2 Resolution:** Check HPA configuration (`kubectl get hpa`). Ensure metrics-server is running. Verify if Cluster Autoscaler is enabled on the node pool to add more VMs when pods are pending.

### 🎫 Scenario 3: Cannot access application via Public IP

> [!example] Ticket
> "We deployed a service of type LoadBalancer, but the External IP is showing as <pending> for 30 minutes."

**L1 Response:** Check service status `kubectl get svc`. *Dekho service type sahi hai ya nahi.*
**Escalation Trigger:** If it stays pending and Azure infrastructure seems to have an issue provisioning the load balancer.
**L2/L3 Resolution:** Check Azure Portal -> Resource Group (MC_rg_aks_region) to see if the Azure Load Balancer hit a quota limit for Public IPs. Request quota increase or check NSG blocking the provisioning.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between ACI and AKS?
> **Answer:** ACI (Azure Container Instances) is serverless, meant for running single containers quickly without orchestration. AKS (Azure Kubernetes Service) is a full orchestration platform for managing complex, multi-container applications with scaling, networking, and high availability. *ACI single pod ke liye hai, AKS poore cluster ke liye.*

> [!question] Q2: How does AKS handle the Kubernetes control plane?
> **Answer:** Azure fully manages the control plane (API server, scheduler, etcd) and provides it for free. The user only manages and pays for the agent nodes (VMs) where workloads run.

> [!question] Q3: How do you securely allow AKS to pull images from ACR?
> **Answer:** By granting the AKS cluster's Managed Identity the `AcrPull` Role-Based Access Control (RBAC) role on the Azure Container Registry. We don't need to manually configure Docker secrets.

> [!question] Q4: What are the two main network plugins available in AKS?
> **Answer:**
> 1. **Kubenet:** Basic networking where pods get IPs from a logical space, and nodes NAT the traffic. Saves VNet IPs.
> 2. **Azure CNI:** Every pod gets a direct IP from the Azure VNet subnet. Better performance but requires careful subnet sizing. *CNI mein pod direct VNet ka part ban jata hai.*

==**Exam Tip:** For AZ-104, remember that Azure CNI requires a lot more IP addresses in the subnet compared to Kubenet, because every single pod gets its own IP from the VNet.==

---
## 🔗 Related Notes

- [[AZ104-04 Virtual Networks (VNet)]] — Networking concepts needed for Azure CNI
- [[AZ104-03 Azure Virtual Machines]] — Underlying infrastructure for AKS Nodes
- [[AZ104-09 Azure Load Balancer]] — Used by AKS Services to expose apps
