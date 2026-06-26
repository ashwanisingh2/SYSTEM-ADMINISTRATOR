---
tags: [desktop-support, cloud, aws, multi-cloud, L2]
aliases: [aws-fundamentals, aws-basics]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #none
---

# AZ9-07: AWS Fundamentals (Multi-Cloud Awareness)

> [!note] Overview
> This note covers foundational Amazon Web Services (AWS) concepts. It maps AWS services to Microsoft Azure equivalents, outlines AWS global infrastructure, details IAM security configurations, and includes CLI management operations to build multi-cloud administrative awareness.

---
## Concept Overview
- **What it is** — Amazon Web Services (AWS) is a global cloud platform providing on-demand compute, storage, databases, and networking resources. This note introduces core AWS services, comparing them to Azure to support administrators working in hybrid and multi-cloud environments.
- **Why it matters** — Modern enterprises rarely rely on a single cloud vendor. Most combine multiple platforms (e.g., Azure for active directory and email, AWS for web hosting and app scaling). Understanding AWS terminology and service definitions is essential for cross-platform support.
- **Real job encounter** — Supporting hybrid cloud networks, verifying EC2 virtual machine states, managing file storage buckets, and checking user access logs in AWS IAM.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Monitor virtual server (EC2) states (Running/Stopped) via the AWS console, verify firewall rule port permissions, and perform basic password resets for IAM users.
  - **Escalation Trigger**: Escalate if virtual servers fail to boot due to VPC network path issues, or if IAM JSON policy modifications are required.
  - **L2 Resolution**: Provision and size EC2 instances, manage security group firewall rules, create and secure S3 storage buckets, and configure IAM user permissions.
  - **L3 Resolution**: Architect VPC network architectures (VPC Peering, Direct Connect links), deploy AWS Organizations across multi-account structures, and configure federated Single Sign-On (SSO).

---
## Technical Deep Dive

### 1. AWS Global Infrastructure
- **Regions**: Separate geographic areas containing multiple data centers. Every AWS region is completely isolated from others to ensure fault tolerance.
- **Availability Zones (AZs)**: One or more discrete data centers within a region, housed in separate buildings with redundant power, networking, and cooling. They are connected by low-latency fiber links.

### 2. Service Mapping: AWS vs. Azure
Understanding terminology mapping is the core of multi-cloud management:

| Service Category | AWS Service | Azure Service | Description |
|---|---|---|---|
| **Compute** | **EC2** (Elastic Compute Cloud) | Virtual Machines (VM) | Virtualized cloud servers |
| **Object Storage** | **S3** (Simple Storage Service) | Blob Storage | Serverless file/data storage buckets |
| **Virtual Network** | **VPC** (Virtual Private Cloud) | Virtual Network (VNet) | Isolated virtual network environments |
| **Relational DB** | **RDS** (Relational DB Service) | Azure SQL Database | Managed relational database hosts |
| **Identity / IAM** | **AWS IAM** | Microsoft Entra ID | Access control and identity engine |
| **DNS Management**| **Route 53** | Azure DNS | Public and private domain name routing |

### 3. Identity and Access Management (IAM)
AWS secures accounts using IAM:
- **Root Account**: The email address used to create the AWS account. It has absolute admin privileges. **Never use the root account for daily tasks**; secure it with MFA and lock it away.
- **IAM Users**: Individual accounts created for employees.
- **IAM Policies**: JSON documents defining permissions. E.g.:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::mybucket"
    }
  ]
}
```
- **IAM Roles**: Temporary credentials assumed by users, applications, or cloud services (e.g. allowing an EC2 server to write files directly to S3 without storing hardcoded passwords).

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - An active AWS account (Free Tier recommended).
> - AWS Command Line Interface (AWS CLI) installed on your system.
> - An IAM user created with `AdministratorAccess` permissions and generated access keys.

### Step 1: Configure the AWS CLI
Authenticate your terminal session to connect to the AWS API.

1. Open a command prompt and run the configure wizard:
```bash
aws configure
```
2. Input the following parameters when prompted:
```text
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-east-1
Default output format [None]: json
```
**Expected Output:** Credentials saved under local profile `~/.aws/credentials`.

### Step 2: Query EC2 Virtual Instances
1. Run a command to list all virtual servers in the active region:
```bash
aws ec2 describe-instances --query "Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType,State:State.Name}" --output table
```
**Expected Output:** Returns a table detailing active EC2 instances:
```text
---------------------------------------------
|             DescribeInstances             |
+----------------------+-----------+--------+
|          ID          |   State   |  Type  |
+----------------------+-----------+--------+
|  i-0a2b3c4d5e6f7g8h9 |  running  |  t2.micro|
+----------------------+-----------+--------+
```

### Step 3: Create an S3 File Storage Bucket
S3 bucket names must be globally unique across all AWS accounts.

1. Create a bucket (replace `my-audit-bucket-ashwani` with your custom name):
```bash
aws s3 mb s3://my-audit-bucket-ashwani --region us-east-1
```
**Expected Output:** `make_bucket: my-audit-bucket-ashwani`

2. Create a test text file:
```bash
echo "AWS Test File Content" > test.txt
```
3. Upload the file to S3:
```bash
aws s3 cp test.txt s3://my-audit-bucket-ashwani/test.txt
```
**Expected Output:** `upload: ./test.txt to s3://my-audit-bucket-ashwani/test.txt`

### Step 4: Clean Up Resources
1. Delete the test object and bucket to prevent maintenance costs:
```bash
aws s3 rm s3://my-audit-bucket-ashwani/test.txt
aws s3 rb s3://my-audit-bucket-ashwani --force
```

---
## Cheat Sheet / Quick Reference

| AWS Command / Term | Azure Equivalent | Purpose / Example |
|---|---|---|
| **EC2** | Virtual Machine | Virtual compute server |
| **S3** | Blob Storage | File storage bucket |
| **Security Group** | Network Security Group (NSG) | Stateful port firewall rules |
| **VPC** | Virtual Network (VNet) | Network segmentation environment |
| `aws configure` | `az login` | Authenticates local CLI sessions |
| `aws ec2 start-instances` | `az vm start` | Powers on a stopped VM |
| `aws ec2 stop-instances` | `az vm deallocate` | Powers off a running VM |
| `aws s3 ls` | `az storage blob list` | Lists all files inside a storage bucket |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| AWS CLI returns: "An error occurred (UnauthorizedOperation) when calling the DescribeInstances operation." | The API keys configured do not have permissions to query EC2 resources. | Go to AWS IAM Console. Attach an access policy (such as `AmazonEC2ReadOnlyAccess`) to the user group associated with your keys. |
| Cannot SSH to EC2 instance, connection times out. | The security group firewall is blocking port TCP 22, or the VPC Route Table lacks an Internet Gateway. | Check instance security group. Add inbound rule: Allow TCP Port 22 from your source IP. Ensure the instance has a public IP and routing points to `0.0.0.0/0` via an Internet Gateway. |
| S3 bucket creation fails with: "BucketAlreadyExists." | S3 bucket namespace is global; another user has already taken that name. | Choose a different name containing unique suffixes (e.g., adding company initials or date). |
| AWS CLI returns: "Unable to locate credentials." | The `aws configure` values were not saved, or the active profile is incorrect. | Re-run `aws configure` to re-enter keys, and verify the file exists under `~/.aws/config`. |
| EC2 instance type cannot be selected, returning capacity error. | The requested virtual server size is out of stock in the target AZ. | Choose a different AZ within the region, or change the instance family type. |

---
## Interview Questions

> [!question] Question 1 (L1)
> **Q:** What is the difference between an AWS Region and an Availability Zone?
> **A:** An AWS **Region** is a separate geographic area in the world (e.g., `us-east-1` in Virginia) containing multiple data centers. An **Availability Zone (AZ)** is a distinct, isolated cluster of data centers *within* that Region. AZs have independent power, cooling, and network links, meaning if one AZ suffers a physical disaster, the other AZs in the region remain active to prevent downtime.

> [!question] Question 2 (L2)
> **Q:** How do AWS Security Groups differ from Network Access Control Lists (NACLs)?
> **A:** **Security Groups** act as virtual firewalls for individual EC2 instances. They are **stateful** (if you allow inbound port 80, the return outbound traffic is automatically permitted). **NACLs** act as firewalls for entire subnets. They are **stateless** (you must write separate rules for both inbound and outbound traffic). Security groups only support "allow" rules, while NACLs support both "allow" and "deny" rules.

> [!question] Question 3 (L3/Scenario)
> **Q:** You are migrating a file-upload application to AWS EC2. To write files to an S3 bucket, the developer asks you to generate IAM user keys and save them in the app's config file on the server. Is this secure, and how would you resolve it?
> **A:** 
> - **Situation:** Application server requires permissions to write files to an S3 storage bucket.
> - **Task:** Secure access without hardcoding long-term API keys on the server.
> - **Action:** Saving permanent IAM access keys on the server is a major security risk; if the server is compromised, the keys are stolen. Instead, I will implement **IAM Roles**:
>   1. Create an IAM Role named `EC2-S3-Write-Role` and attach a policy allowing `s3:PutObject` access to the target bucket.
>   2. Go to the EC2 console, right-click the virtual server -> Security -> **Modify IAM Role**. Associate the newly created role with the server.
>   3. Instruct the developer to delete the hardcoded credentials. The AWS SDK on the server will automatically fetch temporary access tokens from the instance metadata endpoint.
> - **Result:** Access is authorized securely using temporary credentials, preventing key leaks.

---
## Seedha Simple Mein
*Seedha simple mein: AWS (Amazon Web Services) duniya ka sabse bada cloud platform hai. Azure ke sath hybrid ya multi-cloud setups me iska use common hai. AWS me virtual servers ko `EC2` aur storage areas ko `S3 Buckets` kaha jata hai. IAM ke zariye users aur permissions coordinate ki jati hain, jo system safety ensure karti hai.*

---
## Related Notes
- [[04-Cloud-and-Security/08-Azure/AZ9-01 Cloud Computing Fundamentals|AZ9-01 Cloud Computing Fundamentals]] — General cloud benefits comparison.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — DNS routing comparison with Route 53.
- [[04-Cloud-and-Security/08-Azure/AZ104-03 Azure Virtual Machines|AZ104-03 Azure Virtual Machines]] — Comparing EC2 virtual servers with Azure VMs.

---
*Tags: #desktop-support #cloud #aws #multi-cloud #L2*
