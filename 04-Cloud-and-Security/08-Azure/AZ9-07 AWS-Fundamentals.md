---
tags: [desktop-support, cloud, aws, multi-cloud, L2]
aliases: [aws-fundamentals, aws-basics]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-blue]
> ☁️ **AZURE CLOUD**

`#complete` `#beginner` `#none`

# AZ9-07: AWS Fundamentals (Multi-Cloud Awareness)

> [!abstract] Overview
> Yeh note AWS (Amazon Web Services) ke foundational concepts ko cover karta hai aur inhe Azure ke equivalents se map karta hai. Ek support engineer ke liye multi-cloud environment mein cross-platform support dene aur resources manage karne ke liye yeh knowledge hona bahut zaroori hai.

---
## 🧠 Concept Overview

- **What it is** — Amazon Web Services (AWS) ek global cloud platform hai jo on-demand compute, storage, databases, aur networking resources provide karta hai.
- **Why it matters** — Modern enterprises rarely ek cloud vendor par rely karte hain. Azure (AD aur email ke liye) aur AWS (web hosting aur scaling ke liye) ka combination aam baat hai.
- **Where you see this** — Jab aap hybrid cloud networks ko support kar rahe hon, EC2 VMs ki states verify kar rahe hon, ya IAM mein user access logs check kar rahe hon.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Monitor EC2 states (Running/Stopped) via console, verify firewall rules, aur basic IAM password resets. |
| **L2** | Provision/size EC2 instances, manage security group rules, create/secure S3 buckets, aur IAM policies configure karna. |
| **L3** | Architect VPC network (VPC Peering, Direct Connect), AWS Organizations deploy karna, aur federated SSO setup karna. |

> [!tip] Seedha Simple Mein
> *Seedha simple mein: AWS duniya ka sabse bada cloud platform hai. Azure ke sath hybrid setups me iska use common hai. AWS me virtual servers ko EC2 aur storage areas ko S3 Buckets kaha jata hai. IAM se user permissions handle hoti hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **AWS aur Azure ka setup** is like **Alag-alag brands ke smartphones** because...
>
> - Android aur iOS alag OS hain par kaam dono ka same hai (calling, apps, internet).
> - Waise hi AWS EC2 aur Azure VM dono virtual machines hain, bas naam aur interface alag hain. Samajhna ek ko toh doosra aasan lagta hai.

---
## 🔬 Technical Deep Dive

### 1. AWS Global Infrastructure

> [!info] Key Concept
> **Regions** aur **Availability Zones (AZs)** AWS ke physical network ka core hain.

- **Regions**: Separate geographic areas containing multiple data centers. Every AWS region is completely isolated from others to ensure fault tolerance.
- **Availability Zones (AZs)**: One or more discrete data centers within a region, housed in separate buildings with redundant power, networking, and cooling. They are connected by low-latency fiber links.

### 2. Service Mapping: AWS vs. Azure

> [!info] Key Concept
> Multi-cloud management ke liye services ka naam aur unka maqsad samajhna zaroori hai.

| Service Category | AWS Service | Azure Service | Description |
|---|---|---|---|
| **Compute** | **EC2** | Virtual Machines (VM) | Virtualized cloud servers |
| **Object Storage** | **S3** | Blob Storage | Serverless file/data storage buckets |
| **Virtual Network** | **VPC** | Virtual Network (VNet) | Isolated virtual network environments |
| **Relational DB** | **RDS** | Azure SQL Database | Managed relational database hosts |
| **Identity / IAM** | **AWS IAM** | Microsoft Entra ID | Access control and identity engine |
| **DNS Management**| **Route 53** | Azure DNS | Public and private domain name routing |

### 3. Identity and Access Management (IAM)

> [!info] Key Concept
> IAM AWS accounts aur resources ki security ko manage karta hai.

- **Root Account**: The email address used to create the AWS account. It has absolute admin privileges. 
- **IAM Users**: Individual accounts created for employees.
- **IAM Policies**: JSON documents defining permissions.
- **IAM Roles**: Temporary credentials assumed by users, applications, or cloud services.

> [!danger] Common Mistake
> **Never use the root account for daily tasks.** Root account ko MFA se secure karein aur lock kar ke rakhein. Daily tasks ke liye Admin role wale IAM user ka use karein.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - An active AWS account (Free Tier recommended).
> - AWS Command Line Interface (AWS CLI) installed on your system.
> - An IAM user created with `AdministratorAccess` permissions and generated access keys.

### Step 1: Configure the AWS CLI

```bash
# Terminal session ko AWS API se authenticate karne ke liye
aws configure
```

> [!success] Expected Output
> ```text
> AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
> AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
> Default region name [None]: us-east-1
> Default output format [None]: json
> ```
> *Credentials saved under local profile `~/.aws/credentials`.*

### Step 2: Query EC2 Virtual Instances

```bash
# Active region mein saare virtual servers ko list karne ke liye
aws ec2 describe-instances --query "Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType,State:State.Name}" --output table
```

> [!success] Expected Output
> ```text
> ---------------------------------------------
> |             DescribeInstances             |
> +----------------------+-----------+--------+
> |          ID          |   State   |  Type  |
> +----------------------+-----------+--------+
> |  i-0a2b3c4d5e6f7g8h9 |  running  |  t2.micro|
> +----------------------+-----------+--------+
> ```

### Step 3: Create an S3 File Storage Bucket

> [!warning] Pre-requisites
> S3 bucket names must be globally unique across all AWS accounts.

```bash
# Bucket create karna (replace my-audit-bucket-ashwani)
aws s3 mb s3://my-audit-bucket-ashwani --region us-east-1

# Test file banana
echo "AWS Test File Content" > test.txt

# File S3 par upload karna
aws s3 cp test.txt s3://my-audit-bucket-ashwani/test.txt
```

> [!success] Expected Output
> ```text
> make_bucket: my-audit-bucket-ashwani
> upload: ./test.txt to s3://my-audit-bucket-ashwani/test.txt
> ```

### Step 4: Clean Up Resources

```bash
# Test object aur bucket delete karna taaki cost na aaye
aws s3 rm s3://my-audit-bucket-ashwani/test.txt
aws s3 rb s3://my-audit-bucket-ashwani --force
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `aws configure` | Local CLI sessions ko authenticate karta hai (like `az login`) | `aws configure` |
| `aws ec2 start-instances` | Stopped VM ko power on karta hai | `aws ec2 start-instances --instance-ids i-0a2b...` |
| `aws ec2 stop-instances` | Running VM ko power off karta hai | `aws ec2 stop-instances --instance-ids i-0a2b...` |
| `aws s3 ls` | Storage bucket ke andar saari files ko list karta hai | `aws s3 ls s3://my-bucket` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| API error: "UnauthorizedOperation" on DescribeInstances. | Configured API keys ke paas EC2 query karne ki permissions nahi hain. | IAM Console se user group ko `AmazonEC2ReadOnlyAccess` policy attach karein. |
| Cannot SSH to EC2 instance (connection timeout). | Security group port 22 block kar raha hai, ya VPC Route Table mein Internet Gateway nahi hai. | Security group mein TCP Port 22 allow karein. Ensure instance has public IP and route to `0.0.0.0/0`. |
| S3 bucket creation fails ("BucketAlreadyExists"). | S3 bucket namespace global hai; kisi aur ne same naam le liya hai. | Unique suffixes (e.g., company initials ya date) add kar ke naya naam use karein. |
| AWS CLI returns: "Unable to locate credentials." | `aws configure` ke values save nahi hue, ya profile incorrect hai. | `aws configure` dobara run karein, aur `~/.aws/config` check karein. |
| Capacity error for EC2 instance type. | Target AZ mein requested VM size out of stock hai. | Region ke andar doosra AZ choose karein, ya instance type change karein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: AWS EC2 Unreachable

> [!example] Ticket
> "I cannot connect to my EC2 instance i-0a2b3c4d5e6f7g8h9 via SSH. Please check."

**L1 Response:** Verify instance state is `running` and public IP is attached. Check if the security group allows inbound TCP Port 22 from the user's IP.
**Escalation Trigger:** Agar instance running hai, security group theek hai aur phir bhi connect nahi ho raha (network routing issue lag raha hai).
**L2 Resolution:** Review VPC Route Tables to ensure an Internet Gateway (IGW) is attached and routing is configured properly (`0.0.0.0/0` -> IGW). Check Network ACLs.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between an AWS Region and an Availability Zone?
> **Answer:** An AWS **Region** is a separate geographic area in the world (e.g., `us-east-1` in Virginia) containing multiple data centers. An **Availability Zone (AZ)** is a distinct, isolated cluster of data centers *within* that Region. AZs have independent power, cooling, and network links.
> ==**Exam Tip:** AZs ensure high availability within a region to prevent downtime during local disasters.==

> [!question] Q2: How do AWS Security Groups differ from Network Access Control Lists (NACLs)?
> **Answer:** **Security Groups** act as virtual firewalls for individual EC2 instances aur **stateful** hote hain (inbound allow toh outbound automatic allow). **NACLs** subnet-level firewalls hote hain aur **stateless** hote hain (inbound aur outbound separate allow/deny rules chahiye).

> [!question] Q3: How do you securely allow an EC2 instance to write to an S3 bucket without hardcoding IAM keys?
> **Answer:** Hardcoding keys is a major security risk. IAM **Roles** use karte hain. Ek role banayenge jisme `s3:PutObject` ki policy hogi, aur us role ko EC2 instance se attach karenge. SDKs metadata endpoint se automatically temporary credentials fetch kar lete hain.

---
## 🔗 Related Notes

- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — General cloud benefits comparison.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — DNS routing comparison with Route 53.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Comparing EC2 virtual servers with Azure VMs.
