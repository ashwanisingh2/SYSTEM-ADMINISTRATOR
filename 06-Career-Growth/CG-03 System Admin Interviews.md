---
tags: [career-growth, interviews, beginner]
aliases: [sysadmin-interviews, interview-prep]
created: 2026-06-26
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-yellow]
> 📈 **CAREER GROWTH**

`#complete` `#beginner` `#none`

# CG-03: System Admin Interviews

> [!abstract] Overview
> System Administrator interviews technical aur behavioral (HR) rounds ka mix hote hain. Yeh note aapko step-by-step batayega ki interview ko kaise approach karna hai, kya pre-requisites hain, aur kis tarah ke questions aate hain. Ek L1/L2 engineer ko technical concepts ke saath-saath apni problem-solving approach ko show karna bahut zaroori hota hai.

---
## 🧠 Concept Overview

- **What it is** — It's a formal assessment process to evaluate your technical skills, problem-solving mindset, and cultural fit within an IT team.
- **Why it matters** — Your interview performance directly impacts your job placement and salary package. *Aap kitna bhi technical jante ho, agar usko interview mein sahi se present nahi kar paye toh selection mushkil hai.*
- **Where you see this** — Applying for new jobs, internal promotions, or project transitions.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Focus on basics, troubleshooting steps, basic commands, and ticket handling. |
| **L2** | Focus on architecture, scenario-based issues, server setups, and root cause analysis (RCA). |
| **L3** | Focus on enterprise design, automation, disaster recovery, and leadership skills. |

> [!tip] Seedha Simple Mein
> *Interview ek exam nahi, ek discussion hai jahan aapko dikhana hai ki aap company ki problems kaise solve kar sakte ho. Confidence aur basic concepts clear rakhna sabse badi key hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **System Admin Interview** is like **Doctor diagnosing a patient** because...
>
> - As a doctor asks questions to understand symptoms before prescribing medicine, an interviewer gives you a scenario to see how you reach the root cause before fixing it.
> - Prescribing the wrong medicine without diagnosis is dangerous. Similarly, guessing random commands in an interview without logic is a red flag.

---
## 🔬 Technical Deep Dive

### 1. Resume & Preparation Strategy

> [!info] Key Concept
> A well-structured resume is your first impression. The interview starts from what you have written in your resume.

- **Tailor Your Resume:** Match keywords from the job description (e.g., AD, DNS, Linux, RHEL, Hyper-V).
- **Projects over Duties:** Mention what you achieved, not just your daily tasks.
- **Lab Practice:** Create your own virtual labs using VirtualBox or VMware.

> [!danger] Common Mistake
> *Apne resume mein aisi technologies likhna jo aapko nahi aati. Interviewer hamesha wahi se poochega jo aapne likha hai, so always be honest.*

### 2. Behavioral & Communication Skills

- **STAR Method:** Answer scenario questions using Situation, Task, Action, Result.
- **Honesty:** If you don't know an answer, say "I don't know the exact answer, but here is how I would find out or troubleshoot it."

### 3. Core Technologies to Master

- **Operating Systems (Linux/Windows):** Process management, memory management, disk partitions (LVM, RAID), and service management.
- **Networking Fundamentals:** OSI Model (specifically Layer 2, 3, 4, 7), Subnetting, VLANs, Routing, DNS, DHCP, VPN, and Firewalls.
- **Virtualization & Cloud:** VMware ESXi, Hyper-V, Docker basics, and cloud basics (AWS/Azure).
- **Automation & Scripting:** Bash or PowerShell scripting is becoming mandatory. You must know how to automate repetitive tasks.

---
## 🛠️ Step-by-Step Lab (Mock Interview Setup)

> [!warning] Pre-requisites
> - A printed copy of your resume.
> - A quiet room and good internet connection (for online interviews).

### Step 1: Self-Introduction (Elevator Pitch)

```text
# Example Introduction Structure
"Hi, I am [Name]. I have X years of experience in managing Windows/Linux environments. 
My daily tasks include handling Active Directory, troubleshooting network issues, and resolving L1/L2 tickets. 
In my current role, I recently optimized the server patching process..."
```

> [!success] Expected Output
> An impressive start that directs the interviewer towards your strong areas.

### Step 2: Practical Scenarios (Lab Test)

Many interviews include a practical test. Set up a local lab to practice.

```bash
# Practice basic server health checks continuously
top
df -h
free -m
systemctl status <service_name>
```

---
## ⌨️ Command Cheat Sheet (Interview Essentials)

Interviewers often ask for specific commands to test if you actually have hands-on experience or just theoretical knowledge.

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `ping` | Network reachability test karta hai. | `ping 8.8.8.8` |
| `nslookup` / `dig` | DNS resolution check karta hai. | `nslookup google.com` |
| `tracert` / `traceroute` | Network path trace karta hai. | `traceroute 1.1.1.1` |
| `netstat` / `ss` | Active network connections aur open ports dikhata hai. | `netstat -tulnp` |
| `top` / `htop` | Real-time CPU aur memory usage dikhata hai. | `top` |
| `grep` | Logs mein error dhoondhne ke kaam aata hai. | `cat /var/log/messages \| grep error` |
| `systemctl` | Services ko start/stop/restart/enable karta hai. | `systemctl restart httpd` |
| `chmod` / `chown` | File permissions aur ownership change karta hai. | `chmod 755 file.sh` |

---
## 🚑 Troubleshooting Guide (Interview Mindset)

Interviewers love to give broken scenarios. This is how you should approach answering them.

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix (Interview Answer Approach) |
|-----------|----------------|-------|
| Server is unreachable over SSH/RDP. | Network issue, Firewall, Service down. | *"Sabse pehle main ping karunga network check karne ke liye, phir port check karunga, aur aakhir mein firewall rules dekhunga."* |
| Disk is 100% full. | Large log files, old backups. | *"Main `df -h` se disk check karunga, phir `du -sh *` se large files dhoondhunga, aur unnecessary logs ko delete/compress karunga."* |
| Application is slow. | High CPU/RAM, Database issue, Network latency. | *"Main `top` command run karunga server resources check karne ke liye. Phir logs dekhunga aur network latency test karunga."* |
| User cannot login to domain. | Password expired, Account locked, Network issue, DNS down. | *"Main user se error message poochunga, AD mein account status check karunga, aur ensure karunga PC domain controller se communicate kar pa raha hai."* |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: The "Website is Down" Ticket

> [!example] Ticket
> "Users are reporting that the internal portal `portal.company.local` is not loading."

**L1 Response:** 
*Check karunga ki portal srif ek user ke liye down hai ya sabke liye. Ping test karunga aur basic DNS resolution check karunga.*

**Escalation Trigger:** 
*Agar portal server tak ping ja raha hai lekin page load nahi ho raha, aur service restart karne se bhi fix nahi hua.*

**L2 Resolution:** 
*Server par login karke web server (IIS/Apache) status check karna, error logs dekhna, aur firewall/port status verify karna.*

### 🎫 Scenario 2: High Memory Alert

> [!example] Ticket
> "Monitoring tool triggered critical alert: Server-DB-01 memory utilization is at 99%."

**L1 Response:** 
*Server par login karke check karunga konsa process sabse zyada RAM use kar raha hai using `top` or Task Manager.*

**Escalation Trigger:** 
*Agar process business critical hai (like SQL server) aur usko kill/restart nahi kar sakte production hours mein.*

**L2 Resolution:** 
*DB admin se coordinate karna query optimization ke liye, aur agar zarurat ho toh hardware upgrade (scale up) plan karna.*

### 🎫 Scenario 3: User Password Locking Continuously

> [!example] Ticket
> "User complains their Active Directory account gets locked every 10 minutes even after unlocking."

**L1 Response:** 
*User ka account unlock karunga aur unko naya password set karne bolunga. Check karunga ki kya unka account mobile device ya kisi purane system pe login hai.*

**Escalation Trigger:** 
*Agar password reset ke baad bhi issue persist karta hai.*

**L2 Resolution:** 
*Domain Controller ke security event logs check karna (Event ID 4740) to find the source machine jahan se wrong password attempts aa rahe hain. Phir us device par cached credentials clear karna.*

### 🎫 Scenario 4: "Cannot access Shared Drive"

> [!example] Ticket
> "User is unable to access the \\FS01\Finance shared folder, getting 'Access Denied'."

**L1 Response:** 
*Check karunga ki user Finance group ka member hai ya nahi Active Directory mein.*

**Escalation Trigger:** 
*User group me hai but phir bhi access denied aa raha hai, ya folder permissions corrupt lag rahi hain.*

**L2 Resolution:** 
*Share permissions aur NTFS permissions dono verify karna. Effective access check karna ki koi 'Deny' rule toh nahi laga hua.*

---
## 🎤 Interview Questions

> [!question] Q1: How does the DNS resolution process work?
> **Answer:** Step-by-step bataiye. First, local host file check hoti hai, phir local cache, uske baad DNS server (Recursive resolver), phir Root server, TLD server, aur finally Authoritative server IP address return karta hai.
> ==**Exam Tip:** "Host file -> DNS Cache -> Recursive -> Root -> TLD -> Authoritative" yeh sequence yaad rakhiye.==

> [!question] Q2: Explain the boot process of Linux/Windows?
> **Answer:** 
> - **Linux:** BIOS/UEFI -> MBR/GPT -> GRUB -> Kernel -> Init/Systemd -> Runlevel/Target.
> - **Windows:** BIOS/UEFI -> MBR/GPT -> Bootmgr -> Winload.exe -> Ntoskrnl.exe -> Smss.exe -> Winlogon.exe.
> ==**Exam Tip:** Interviwer aapse specifically pooch sakta hai ki agar MBR corrupt ho jaye toh kya error aayega.==

> [!question] Q3: What is the difference between TCP and UDP? Give examples.
> **Answer:** TCP connection-oriented aur reliable hai (3-way handshake, e.g., HTTP, SSH). UDP connection-less aur fast hai, no guarantee of delivery (e.g., DNS, Video Streaming).
> ==**Exam Tip:** TCP ka 3-way handshake (SYN, SYN-ACK, ACK) hamesha explain karein jab yeh question pucha jaye.==

> [!question] Q4: How would you troubleshoot a server that is completely unresponsive?
> **Answer:** 
> 1. Console level access (iLO/iDRAC/vSphere console) se server check karunga.
> 2. Agar kernel panic ya BSOD screen hai, toh server ko reboot karunga aur baad me logs/dump file analyze karunga.
> 3. Agar server power off hai, toh physical hardware, power supply, aur PDU check karunga.
> ==**Exam Tip:** Hamesha "Out-of-Band management" (iLO/iDRAC) ka mention karein, yeh dikhata hai ki aapne actual physical/virtual data center me kaam kiya hai.==

> [!question] Q5: What is RAID? Explain RAID 0, 1, and 5.
> **Answer:** RAID (Redundant Array of Independent Disks) multiple disks ko combine karke performance ya redundancy badhata hai.
> - **RAID 0:** Striping (High speed, no redundancy).
> - **RAID 1:** Mirroring (Redundancy, exact copy on two disks).
> - **RAID 5:** Striping with Parity (Good speed and redundancy, minimum 3 disks required).
> ==**Exam Tip:** Agar database server banana ho toh log RAID 10 use karte hain (Mirroring + Striping).==

> [!question] Q6: What is a reverse proxy?
> **Answer:** Ek server jo internet (clients) se requests receive karta hai aur unhe internal backend servers tak forward karta hai. Yeh security, load balancing, aur caching provide karta hai (e.g., Nginx, HAProxy).
> ==**Exam Tip:** Reverse proxy backend servers ko hide karta hai aur client-facing point banta hai.==

---
## 🔗 Related Notes

- [[01-Linux-RHEL/LNX-01 Basic Commands|LNX-01 Basic Commands]] — For brushing up Linux skills before interview.
- [[02-Windows-Server/WIN-02 Active Directory Basics|WIN-02 Active Directory Basics]] — Essential for Windows AD rounds.
- [[04-Networking/NET-01 Troubleshooting|NET-01 Troubleshooting]] — Important network concepts asked in interviews.
