---
tags: [desktop-support, cloud, gcp, google-cloud, L2]
aliases: [gcp-fundamentals, gcp-basics]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

# AZ9-08: GCP Fundamentals (Google Cloud Platform Basics)

> [!note] Overview
> This note covers foundational Google Cloud Platform (GCP) concepts. It details the GCP hierarchical resource model, compares key services to Azure and AWS, explains global VPC networking, and outlines gcloud CLI command operations for multi-cloud systems management.

---
## Concept Overview
- **What it is** — Google Cloud Platform (GCP) is Google's public cloud infrastructure suite. This note covers its core services, resource hierarchy organization (Organization, Folders, Projects), global VPC network topologies, and IAM structures.
- **Why it matters** — System administrators frequently manage resources across multiple cloud environments. Understanding Google Cloud's structure (such as project scoping and global networks) enables admins to manage workloads, configure cross-cloud VMs, and diagnose access controls.
- **Real job encounter** — Checking virtual machine states in Google Compute Engine, creating file storage buckets for multi-cloud backup logs, and assigning project permissions using Google Group configurations.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Verify Compute Engine VM states (Running/Terminated) in the Google Console, lookup project IDs, and search for log errors in Cloud Logging.
  - **Escalation Trigger**: Escalate if virtual servers encounter network routing failures within the global VPC, or if custom service account role mappings are required.
  - **L2 Resolution**: Provision and scale Compute Engine VM instances, write VPC firewall tag rules, set up Cloud Storage buckets, and bind IAM roles to users.
  - **L3 Resolution**: Implement Google Cloud resource organizational hierarchies, configure Inter-Cloud VPN tunnels, manage Google Kubernetes Engine (GKE) clusters, and implement Identity-Aware Proxy (IAP) access.

---
## Technical Deep Dive

### 1. The GCP Resource Hierarchy
GCP structures resources hierarchically, permitting inherited security controls and policy enforcement:
- **Organization (Root Node)**: Represents the entire enterprise (linked to a Google Workspace or Cloud Identity domain).
- **Folders**: Optional groupings used to organize projects by department (e.g., Finance) or environment (e.g., Production).
- **Projects (Core Container)**: **Every resource in GCP must belong to a project**. Billing is configured at the project level, and projects act as isolation boundaries for resources.
- **Resources**: The actual compute, storage, and database instances (e.g., GCE VMs, GCS buckets).

```
                 [ Organization ] (Root - company.com)
                        |
            +-----------+-----------+
            |                       |
            v                       v
      [ Prod Folder ]        [ Dev Folder ]
            |                       |
            v                       v
     [ Project-Prod-01 ]     [ Project-Dev-01 ] (Billing & Admin boundary)
            |
      +-----+-----+
      |           |
      v           v
  [ GCE VM ]  [ GCS Bucket ] (Resources)
```

### 2. Service Mapping: GCP vs. Azure vs. AWS
Understanding cross-platform mapping enables administrators to translate infrastructure needs:

| Service Category | Google Cloud (GCP) | Microsoft Azure | Amazon Web Services (AWS) |
|---|---|---|---|
| **Virtual Server** | **Compute Engine** (GCE) | Virtual Machines (VM) | EC2 |
| **Object Storage** | **Cloud Storage** (GCS) | Blob Storage | S3 |
| **Virtual Network** | **VPC Network** | Virtual Network (VNet) | VPC |
| **Containers** | **Google Kubernetes Engine** (GKE)| Azure Kubernetes Service (AKS)| EKS |
| **Managed SQL** | **Cloud SQL** | Azure SQL Database | RDS |
| **Access Control** | **Cloud IAM** | Microsoft Entra ID | AWS IAM |

### 3. GCP Global VPC Networking Model
GCP's networking model differs fundamentally from AWS and Azure:
- **Global Scope**: A GCP Virtual Private Cloud (VPC) network is a global resource, not regional.
- **Regional Subnets**: When you create a VPC network, subnets are regional resources.
- **Inter-Region Routing**: Because the network is global, VMs in different regions (e.g., one in Iowa and one in Belgium) can communicate directly using internal private IP addresses over Google's global fiber backbone without requiring complex VPNs, internet routing, or VPC Peering.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - An active Google Cloud Platform account (Free Trial active).
> - Google Cloud SDK (gcloud CLI) installed on your system.
> - An active GCP Project configured (e.g., ID: `my-sysadmin-lab-2026`).

### Step 1: Initialize the gcloud CLI
1. Open a terminal and run the initialization wizard:
```bash
gcloud init
```
2. The CLI will open a browser window requesting authorization to access your Google Account. Click **Allow**.
3. In the terminal, select your default project ID (`my-sysadmin-lab-2026`) and default zone (e.g., `us-central1-a`).
**Expected Output:** Console authenticated and configured to point to project.

### Step 2: Create a Compute Engine (VM) Instance
We will deploy a Rocky Linux 9 virtual machine using the CLI.

1. Execute the creation command:
```bash
gcloud compute instances create my-gcp-server --zone=us-central1-a --machine-type=e2-micro --image-family=rocky-linux-9 --image-project=rocky-linux-cloud
```
**Expected Output:** Provisioning details showing internal and external IPs:
```text
Created [https://www.googleapis.com/compute/v1/projects/...].
NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
my-gcp-server    us-central1-a  e2-micro                   10.128.0.2   34.120.45.67   RUNNING
```

### Step 3: Create a Cloud Storage Bucket and Upload a File
1. Create a storage bucket (replace `my-sysadmin-bucket-2026` with a globally unique name):
```bash
gcloud storage buckets create gs://my-sysadmin-bucket-2026 --location=us-central1
```
**Expected Output:** `Creating gs://my-sysadmin-bucket-2026/...`

2. Create a test file:
```bash
echo "Google Cloud Storage Audit File" > audit.txt
```
3. Upload the file to your bucket:
```bash
gcloud storage cp audit.txt gs://my-sysadmin-bucket-2026/audit.txt
```
**Expected Output:** `Copying file://audit.txt to gs://my-sysadmin-bucket-2026/audit.txt`

### Step 4: Clean Up Resources
1. Delete the instance and the storage bucket to avoid billing charges:
```bash
gcloud compute instances delete my-gcp-server --zone=us-central1-a --quiet
gcloud storage buckets delete gs://my-sysadmin-bucket-2026 --recursive --quiet
```
**Expected Output:** Resources deleted successfully.

---
## Cheat Sheet / Quick Reference

| GCP Command / Term | Azure Equivalent | Purpose / Example |
|---|---|---|
| **Compute Engine** | Virtual Machine | Virtual server compute node |
| **Cloud Storage** | Blob Storage | Object storage bucket (`gs://`) |
| **Project** | Subscription | Scopes billing, resources, and access control |
| `gcloud init` | `az login` | Authenticates local terminal CLI |
| `gcloud compute instances start` | `az vm start` | Powers on a stopped VM |
| `gcloud compute instances stop` | `az vm deallocate` | Powers off a running VM |
| `gcloud storage ls` | `az storage blob list` | Lists objects inside a target bucket |
| `gcloud auth login` | Authenticates account | `gcloud auth login` |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| `gcloud` command fails: "Project not specified." | No active project is associated with the local gcloud CLI configuration. | Set the active project ID: `gcloud config set project <PROJECT_ID>`. |
| Cannot connect to Compute Engine VM via SSH, times out. | The default VPC firewall rule blocking SSH (TCP Port 22) has been disabled or modified. | Check VPC firewall rules. Ensure rule `default-allow-ssh` allowing ingress TCP port 22 is active, or use Google's Identity-Aware Proxy (IAP) to connect. |
| Storage bucket creation fails with: "409 Conflict: Bucket already exists." | The requested bucket name is already used by another GCP user globally. | Rename the bucket. All Google Cloud Storage bucket names must be globally unique across all projects. |
| VM fails to start, returning: "Quota 'CPUS' exceeded." | The project has reached its CPU limit for that region. | Request a quota increase in the GCP Console under IAM & Admin -> Quotas, or delete unused VMs to free up CPU capacity. |
| Custom applications cannot access GCP resources. | The VM instance was not configured with the correct service account or API scopes. | Stop the VM, go to VM settings, and attach a service account containing the necessary IAM roles. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the root level container for billing and resource allocation in Google Cloud Platform?
> **A:** The root container is the **Project**. In GCP, every single resource (such as a VM, disk, or database) must belong to a project. Billing is configured per project, and resources in separate projects are isolated by default.

> [!question] L2 Question
> **Q:** What is a key advantage of GCP's VPC networking model compared to AWS and Azure?
> **A:** GCP's VPC network is **global**, not regional. In AWS and Azure, a virtual network is bound to a single region. In GCP, subnets are regional resources within a global VPC. This allows VMs in different continents to communicate over a secure private network natively using internal IPs, without configuring public routing or VPC Peering.

> [!question] L3/Scenario Question
> **Q:** An application hosted on a Google Compute Engine VM needs to read data from a Google Cloud Storage bucket. How do you configure this securely without storing any user passwords or key files on the VM?
> **A:** 
> - **Situation:** Securing VM access to Cloud Storage buckets.
> - **Task:** Grant programmatic permissions without hardcoded key files.
> - **Action:** I will utilize GCP **Service Accounts** and IAM role bindings:
>   1. Create a custom Service Account (e.g., `bucket-reader-sa@project.iam.gserviceaccount.com`).
>   2. Go to the Cloud Storage bucket permissions and assign the Service Account the **Storage Object Viewer** IAM role.
>   3. In the GCE VM configurations, associate this Service Account with the VM.
>   4. The application on the VM will authenticate transparently using the local metadata server, which automatically issues temporary access tokens.
> - **Result:** Access is granted securely without key management overhead on the server.

---
## Seedha Simple Mein
*Seedha simple mein: Google Cloud Platform (GCP) Google ki cloud service hai. Isme resources ko organize karne ke liye "Projects" ka use hota hai. GCP ki sabse achi baat iska global VPC networking network hai, jisse different regions (jaise Asia aur USA) ke servers directly internal IPs par bina complex routing ke baat kar sakte hain. CLI me `gcloud` command use hoti hai.*

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — Cloud design comparison.
- [[04-Cloud-and-Security/08-Azure/AZ9-07 AWS-Fundamentals|AZ9-07 AWS-Fundamentals]] — AWS cloud architecture comparison.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Azure VM configuration details.

---
*Tags: #desktop-support #cloud #gcp #google-cloud #L2*
