---
tags: [desktop-support, cloud, gcp, google-cloud, L2]
aliases: [gcp-fundamentals, gcp-basics]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#beginner` `#none`

# AZ9-08: GCP Fundamentals (Google Cloud Platform Basics)

> [!abstract] Overview
> Yeh note Google Cloud Platform (GCP) ke foundational concepts ko cover karta hai. Ek support engineer ke liye yeh jaanna zaroori hai kyunki multi-cloud environments mein GCP resource hierarchy, global VPC network, aur IAM structures ko manage karna aana chahiye.

---
## 🧠 Concept Overview

- **What it is** — Google Cloud Platform (GCP) Google ka public cloud infrastructure suite hai. Isme resources ko Organization, Folders aur Projects ke hierarchy mein manage kiya jata hai.
- **Why it matters** — System administrators ko multi-cloud environments mein resources manage karne hote hain. GCP ka global network aur project scoping samajhna workloads aur access control ke liye zaroori hai.
- **Where you see this** — Virtual machine states check karne, multi-cloud backup logs ke liye file storage buckets create karne, aur Google Group ke through project permissions assign karne mein.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Compute Engine VM states (Running/Terminated) check karna aur Cloud Logging me errors dhoondhna. |
| **L2** | VM instances provision karna, VPC firewall rules lagana, aur IAM roles bind karna. Escalate agar VPC routing fail ho ya custom roles chahiye. |
| **L3** | Resource organizational hierarchies design karna, Inter-Cloud VPN aur GKE clusters manage karna. |

> [!tip] Seedha Simple Mein
> *Google Cloud Platform (GCP) Google ki cloud service hai. Isme resources ko organize karne ke liye "Projects" ka use hota hai. GCP ki VPC globally kaam karti hai jisse alag regions ke servers directly internal IPs par baat kar sakte hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Ek baddi Office Building** is like **GCP Resource Hierarchy** because...
>
> - **Organization** poori building (company) hai.
> - **Folders** alag-alag floors (departments) hain.
> - **Projects** specific rooms (billing boundaries) hain, aur **Resources** us room ke andar ka furniture (VMs, Buckets) hai. Har furniture ka kisi na kisi room mein hona zaroori hai.

---
## 🔬 Technical Deep Dive

### 1. The GCP Resource Hierarchy

> [!info] Key Concept
> GCP structures resources hierarchically, permitting inherited security controls and policy enforcement. **Every resource in GCP must belong to a project.**

- **Organization (Root Node)**: Represents the entire enterprise (linked to a Google Workspace or Cloud Identity domain).
- **Folders**: Optional groupings used to organize projects by department or environment.
- **Projects (Core Container)**: Billing is configured at the project level, acting as isolation boundaries.
- **Resources**: The actual compute, storage, and database instances.

> [!danger] Common Mistake
> Forgetting that billing and core isolation happen at the **Project** level. Resources cannot exist outside a project.

### 2. GCP Global VPC Networking Model

> [!info] Key Concept
> A GCP Virtual Private Cloud (VPC) network is a **global** resource, not regional like AWS/Azure.

- **Regional Subnets**: When you create a VPC network, subnets are regional resources.
- **Inter-Region Routing**: VMs in different regions can communicate directly using internal private IP addresses over Google's global fiber backbone without requiring complex VPNs.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Google Cloud Platform account (Free Trial active).
> - Google Cloud SDK (`gcloud` CLI) installed.
> - Active GCP Project (e.g., ID: `my-sysadmin-lab-2026`).

### Step 1: Initialize the gcloud CLI

```bash
# Initialize and authenticate gcloud CLI
gcloud init
```

> [!success] Expected Output
> Console authenticated and configured to point to project.

### Step 2: Create a Compute Engine (VM) Instance

```bash
# Provision a Rocky Linux 9 virtual machine
gcloud compute instances create my-gcp-server --zone=us-central1-a --machine-type=e2-micro --image-family=rocky-linux-9 --image-project=rocky-linux-cloud
```

> [!success] Expected Output
> ```text
> Created [https://www.googleapis.com/compute/v1/projects/...].
> NAME             ZONE           INTERNAL_IP  EXTERNAL_IP    STATUS
> my-gcp-server    us-central1-a  10.128.0.2   34.120.45.67   RUNNING
> ```

### Step 3: Create a Cloud Storage Bucket and Upload File

```bash
# Create a storage bucket
gcloud storage buckets create gs://my-sysadmin-bucket-2026 --location=us-central1

# Create and upload file
echo "Google Cloud Storage Audit File" > audit.txt
gcloud storage cp audit.txt gs://my-sysadmin-bucket-2026/audit.txt
```

### Step 4: Clean Up Resources

```bash
# Delete instance and bucket to avoid charges
gcloud compute instances delete my-gcp-server --zone=us-central1-a --quiet
gcloud storage buckets delete gs://my-sysadmin-bucket-2026 --recursive --quiet
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `gcloud init` | Authenticates local terminal CLI | `gcloud init` |
| `gcloud compute instances start` | Powers on a stopped VM | `gcloud compute instances start my-gcp-server` |
| `gcloud compute instances stop` | Powers off a running VM | `gcloud compute instances stop my-gcp-server` |
| `gcloud storage ls` | Lists objects inside a target bucket | `gcloud storage ls gs://my-bucket/` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `gcloud` fails: "Project not specified." | No active project is associated with CLI | Run `gcloud config set project <PROJECT_ID>` |
| Cannot connect to VM via SSH | Default VPC firewall blocking TCP 22 | Check firewall rules, enable `default-allow-ssh` or use IAP |
| Bucket creation fails: "409 Conflict" | Bucket name is already used globally | Rename the bucket. GCS names must be globally unique |
| VM fails: "Quota 'CPUS' exceeded." | Reached CPU limit for region | Request quota increase in GCP Console or delete unused VMs |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Custom App Cannot Access Resources

> [!example] Ticket
> "My application on VM cannot read from the Google Cloud Storage bucket."

**L1 Response:** Verify that the VM is running and check if the bucket exists and is accessible via console.
**Escalation Trigger:** Pass to L2 if IAM or service account bindings need modification.
**L2 Resolution:** Stop the VM, go to VM settings, and attach a service account containing the necessary IAM roles (e.g., Storage Object Viewer). Restart the VM.

---
## 🎤 Interview Questions

> [!question] Q1: What is the root level container for billing and resource allocation in GCP?
> **Answer:** The root container is the **Project**. Every single resource must belong to a project. Billing is configured per project, providing natural isolation.

==**Exam Tip:** Every resource in GCP MUST belong to a Project.==

> [!question] Q2: What is a key advantage of GCP's VPC networking model compared to AWS and Azure?
> **Answer:** GCP's VPC network is **global**, not regional. Subnets are regional within a global VPC, allowing cross-region VM communication on internal IPs without complex routing.

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — Cloud design comparison.
- [[04-Cloud-and-Security/08-Azure/AZ9-07 AWS-Fundamentals|AZ9-07 AWS-Fundamentals]] — AWS cloud architecture comparison.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Azure VM configuration details.
