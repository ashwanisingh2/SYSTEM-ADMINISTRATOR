---
tags: [linux, rhel, security, hardening, selinux, ssh, fail2ban, L3]
aliases: [l-14-linux-security-hardening, l-14, linux-hardening]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX RHEL**

`#complete` `#advanced` `#rhcsa`

# L-14: Linux Security Hardening

> [!abstract] Overview
> Yeh note Linux server ko surakshit (secure) banana sikhata hai — SELinux, sudoers, SSH hardening, fail2ban, auditd, umask, aur sysctl kernel tuning. Ek support engineer ke liye yeh sabse critical topic hai kyunki *ek misconfigured server = hackers ka khula darwaza*. Yahan sab kuch hai jo tujhe production server lock-down karne ke liye chahiye.

---
## 🧠 Concept Overview

- **What it is** — Linux hardening ka matlab hai system ki attack surface ko chhota karna — faaltu services band karo, permissions tight karo, access control lagao, aur monitoring on karo.
- **Why it matters** — *Real job mein ek open SSH port ya ek weak sudo rule ki wajah se poora server compromise ho sakta hai.* Enterprise mein security audit fail hone ka matlab hai project ruk jaata hai.
- **Where you see this** — New RHEL production server setup, InfoSec vulnerability scan ke baad remediation, compliance audits (PCI-DSS, SOC2), aur incident response ke dauran.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | SELinux status check (`getenforce`), password reset, sshd service monitor, basic login failure review |
| **L2** | Sudoers configuration, SSH hardening policies, auditd rule writing, fail2ban jail setup, umask tuning |
| **L3** | Enterprise-wide SELinux custom policies, kernel hardening via `sysctl`, auditd-to-SIEM integration, compliance framework implementation |

> [!tip] Seedha Simple Mein
> *Hardening ka matlab hai server ka darwaza tight karna — sirf sahi log andar aa sakein, baaki sab bahar. Jaise ghar mein tala, CCTV, aur watchman — server mein SELinux, fail2ban, aur auditd.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Ek high-security building ko secure karna** is like **Linux Security Hardening** because...
>
> - **SELinux** = Building ka strict security guard — ==even CEO (root) bhi restricted area mein bina badge ke nahi ja sakta==
> - **Sudoers** = Master key system — *har insaan ko alag rooms ki keys milti hain, kisi ko poori building ki nahi*
> - **SSH hardening** = Main entrance ko hidden biometric door se replace karna — *password ki jagah fingerprint (SSH key) lagao*
> - **fail2ban** = Automated CCTV system — *3 baar galat code try kiya? Gate band, police ko call!*
> - **umask** = Naye rooms ka default lock — *jab room banta hai tab se hi locked rehta hai, tujhe manually lock nahi karna padta*
> - **sysctl** = Building ki walls ko bulletproof banana — *kernel level pe protection, structural defense*
> - **auditd** = Visitor register + CCTV log — *kaun aaya, kya kiya, kab kiya — sab recorded*

---
## 🔬 Technical Deep Dive

### 1. SELinux (Security-Enhanced Linux)

> [!info] Key Concept
> SELinux enforces **Mandatory Access Control (MAC)** — yeh user file permissions (DAC) ko override karta hai. ==Even root bhi blocked ho sakta hai agar SELinux context match nahi karta.==

* **SELinux Modes:**
  * **Enforcing** — Policy enforce hoti hai. Unauthorized actions **blocked + logged**.
  * **Permissive** — Policy enforce nahi hoti. Actions allowed hain, but violations **logged** hain. *Troubleshooting ke liye best mode.*
  * **Disabled** — SELinux bilkul band. ==Reboot lagta hai wapas enable karne mein. Production mein kabhi Disabled mat karo!==

* **SELinux Context:** Har process, file, aur port ka ek context hota hai → `user:role:type:sensitivity`
  * **Type** sabse important hai (e.g., `httpd_sys_content_t`)
  * *Jaise har insaan ka ID card hota hai — har file ka SELinux context hai*

> [!danger] Common Mistake
> SELinux **disable** karna instead of context fix karna. Yeh sabse badi galti hai! ==Hamesha `setenforce 0` (Permissive) use karo troubleshooting ke liye, `Disabled` kabhi nahi.==

==**Exam Tip:** `getenforce` current mode dikhata hai, `sestatus` detailed info deta hai. RHCSA mein dono yaad rakho!==

---

### 2. Sudoers Privilege Escalation

> [!info] Key Concept
> Users ko root commands dene ke liye `/etc/sudoers` file use hoti hai. ==Isko directly edit karna = system lock-out ka risk!==

* **`visudo`** use karo — yeh syntax check karta hai save karne se pehle
* Individual configs `/etc/sudoers.d/` mein rakho — *modular management*
* **Syntax:** `username ALL=(run_as_user) NOPASSWD: /path/to/command`

```bash
# Example: developer ko sirf httpd restart karne do
developer ALL=(root) NOPASSWD: /usr/bin/systemctl restart httpd, /usr/bin/systemctl reload httpd
```

> [!danger] Common Mistake
> `/etc/sudoers` mein `ALL=(ALL) NOPASSWD: ALL` dena. *Yeh root access de deta hai — security ka sabse bada hole!*

==**Exam Tip:** `visudo -f /etc/sudoers.d/filename` se modular sudoers files banao. RHCSA mein yahi tarika expected hai.==

---

### 3. SSH Service Hardening

> [!info] Key Concept
> SSH server ka primary entry point hai — *agar yeh kamzor hai toh poora server kamzor hai.*

**Key hardening steps:**

| 🔒 Setting | 📝 Value | 💡 Kyun |
|-----------|---------|-------|
| `Port` | `2222` (non-standard) | *Automated brute-force scripts port 22 pe chalte hain* |
| `PermitRootLogin` | `no` | *Root direct login band — sudo use karo* |
| `PasswordAuthentication` | `no` | *Password guessing band — SSH keys mandatory* |
| `AllowUsers` | `sysadmin` | *Sirf specific users ko allow karo* |
| `MaxAuthTries` | `3` | *3 se zyada attempts band* |
| `ClientAliveInterval` | `300` | *5 min idle = disconnect* |

> [!danger] Common Mistake
> SSH port change karne ke baad **SELinux update bhool jaana** aur **firewall rule add na karna**. Dono zaroori hain warna sshd start hi nahi hoga!

==**Exam Tip:** SSH port change = 3 steps yaad rakho: (1) `sshd_config` edit, (2) `semanage port -a`, (3) `firewall-cmd --add-port`. Teeno miss kiya toh locked out!==

---

### 4. auditd — Linux Audit Daemon

> [!info] Key Concept
> auditd system calls, file changes, aur access requests track karta hai. *Server pe kya hua — sab recorded.*

* **Config file:** `/etc/audit/rules.d/audit.rules`
* **Log file:** `/var/log/audit/audit.log`
* **Key commands:**
  * `auditctl -l` — current rules dekho
  * `ausearch -k <key>` — specific events search karo
  * `aureport` — summary report generate karo

```bash
# Sudoers file ko monitor karo
sudo auditctl -w /etc/sudoers -p wa -k sudo_changes

# Password file ko monitor karo
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
```

> [!tip] Pro Tip
> *auditd rules ko permanent banane ke liye `/etc/audit/rules.d/audit.rules` mein likho, warna reboot pe rules ud jaayenge.*

---

### 5. fail2ban — Automated Intrusion Prevention

> [!info] Key Concept
> fail2ban log files parse karke repeated login failures detect karta hai aur automatically IP ban karta hai.

* **Config:** `/etc/fail2ban/jail.local` (kabhi `jail.conf` edit mat karo!)
* **Log monitored:** `/var/log/secure` (SSH failures ke liye)
* **Key parameters:**
  * `maxretry` — kitne failed attempts ke baad ban
  * `bantime` — kitni der tak IP banned rehega
  * `findtime` — kitne time window mein failures count hongi

> [!tip] Pro Tip
> *`jail.conf` ko directly edit mat karo — `jail.local` mein override likho. Updates aane pe `jail.conf` overwrite ho jaata hai, `jail.local` safe rehta hai.*

---

### 6. umask & sysctl — Default Permissions & Kernel Tuning

> [!info] Key Concept
> **umask** naye files ki default permissions decide karta hai. **sysctl** kernel parameters runtime pe configure karta hai.

**umask values:**

| 🔢 umask | 📄 File Permissions | 📁 Dir Permissions | 💡 Use Case |
|---------|-------------------|-------------------|-----------|
| `022` | `644` (rw-r--r--) | `755` (rwxr-xr-x) | Default — *sabko read access* |
| `027` | `640` (rw-r-----) | `750` (rwxr-x---) | ==Recommended for servers== |
| `077` | `600` (rw-------) | `700` (rwx------) | Maximum security — *sirf owner* |

**Critical sysctl parameters:**

```bash
# /etc/sysctl.d/99-security.conf

# Ping responses band karo (ICMP echo ignore)
net.ipv4.icmp_echo_ignore_all = 1

# IP forwarding band karo (agar router nahi hai)
net.ipv4.ip_forward = 0

# SYN flood protection enable karo
net.ipv4.tcp_syncookies = 1

# Source routing disable karo
net.ipv4.conf.all.accept_source_route = 0

# ICMP redirect band karo
net.ipv4.conf.all.accept_redirects = 0
```

==**Exam Tip:** `umask 027` production servers ke liye recommended value hai. RHCSA mein umask calculation zaroor puchha jaata hai!==

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A running RHEL 8/9 or CentOS Stream VM
> - Root or sudo access on the machine
> - Firewall daemon (`firewalld`) active
> - Internet access (fail2ban install ke liye EPEL repo chahiye)

### Step 1: SSH Server Harden Karo

```bash
# SSH configuration edit karo
sudo vi /etc/ssh/sshd_config
```

Modify or add these lines:
```text
Port 2222
PermitRootLogin no
PasswordAuthentication no
AllowUsers sysadmin
MaxAuthTries 3
ClientAliveInterval 300
```

```bash
# Firewall mein naya port allow karo
sudo firewall-cmd --add-port=2222/tcp --permanent
sudo firewall-cmd --remove-service=ssh --permanent
sudo firewall-cmd --reload

# SELinux mein naya port register karo
sudo semanage port -a -t ssh_port_t -p tcp 2222

# sshd restart karo
sudo systemctl restart sshd
```

> [!success] Expected Output
> ```
> $ systemctl status sshd
> ● sshd.service - OpenSSH server daemon
>    Active: active (running)
>    Listen: 0.0.0.0:2222
> ```

---

### Step 2: Limited Sudo Access Configure Karo

```bash
# Developer ke liye specific sudo rule banao
sudo visudo -f /etc/sudoers.d/developer
```

Write this rule:
```text
developer ALL=(root) NOPASSWD: /usr/bin/systemctl restart httpd, /usr/bin/systemctl reload httpd
```

```bash
# Verify karo
sudo -l -U developer
```

> [!success] Expected Output
> ```
> User developer may run the following commands:
>     (root) NOPASSWD: /usr/bin/systemctl restart httpd
>     (root) NOPASSWD: /usr/bin/systemctl reload httpd
> ```

---

### Step 3: Fail2ban Install aur Configure Karo

```bash
# EPEL repo aur fail2ban install karo
sudo dnf install epel-release -y
sudo dnf install fail2ban -y

# Local config banao
sudo vi /etc/fail2ban/jail.local
```

Add configuration:
```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 3

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/secure
maxretry = 3
bantime = 1h
```

```bash
# Start aur enable karo
sudo systemctl enable --now fail2ban

# Status check karo
sudo fail2ban-client status sshd
```

> [!success] Expected Output
> ```
> Status for the jail: sshd
> |- Filter
> |  |- Currently failed: 0
> |  |- Total failed:     0
> |  `- File list:        /var/log/secure
> `- Actions
>    |- Currently banned: 0
>    |- Total banned:     0
>    `- Banned IP list:
> ```

---

### Step 4: Kernel Parameters Secure Karo (sysctl)

```bash
# Security parameters add karo
sudo vi /etc/sysctl.d/99-security.conf
```

```ini
# Ping response band karo
net.ipv4.icmp_echo_ignore_all = 1

# IP forwarding band karo
net.ipv4.ip_forward = 0

# SYN flood protection
net.ipv4.tcp_syncookies = 1

# Source routing band karo
net.ipv4.conf.all.accept_source_route = 0
```

```bash
# Configuration load karo
sudo sysctl --system

# Verify karo
sysctl net.ipv4.icmp_echo_ignore_all
```

> [!success] Expected Output
> ```
> net.ipv4.icmp_echo_ignore_all = 1
> ```

---

### Step 5: auditd Rule Set Karo

```bash
# Sudoers aur passwd file monitor karo
sudo auditctl -w /etc/sudoers -p wa -k sudo_changes
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
sudo auditctl -w /etc/ssh/sshd_config -p wa -k ssh_config_changes

# Verify rules
sudo auditctl -l
```

> [!success] Expected Output
> ```
> -w /etc/sudoers -p wa -k sudo_changes
> -w /etc/passwd -p wa -k passwd_changes
> -w /etc/ssh/sshd_config -p wa -k ssh_config_changes
> ```

```bash
# Permanent banane ke liye rules file mein likho
sudo vi /etc/audit/rules.d/audit.rules
```

```text
-w /etc/sudoers -p wa -k sudo_changes
-w /etc/passwd -p wa -k passwd_changes
-w /etc/ssh/sshd_config -p wa -k ssh_config_changes
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `getenforce` | SELinux current mode check karo | `getenforce` |
| `sestatus` | SELinux detailed status dekho | `sestatus` |
| `setenforce 0` | SELinux Permissive mode (temporary) | `sudo setenforce 0` |
| `semanage port -l` | SELinux allowed ports list karo | `sudo semanage port -l \| grep ssh` |
| `semanage port -a` | Naya port SELinux mein add karo | `sudo semanage port -a -t ssh_port_t -p tcp 2222` |
| `visudo` | Sudoers file safely edit karo | `sudo visudo` |
| `visudo -f` | Specific sudoers file edit karo | `sudo visudo -f /etc/sudoers.d/dev` |
| `sudo -l -U user` | User ki sudo permissions dekho | `sudo -l -U developer` |
| `sysctl -a` | Sab kernel parameters list karo | `sysctl -a \| grep forward` |
| `sysctl --system` | Sab sysctl config files reload karo | `sudo sysctl --system` |
| `fail2ban-client status` | Fail2ban overall status dekho | `sudo fail2ban-client status` |
| `fail2ban-client status sshd` | SSH jail ka status dekho | `sudo fail2ban-client status sshd` |
| `fail2ban-client unban <IP>` | Banned IP unban karo | `sudo fail2ban-client unban 192.168.1.50` |
| `auditctl -l` | Current audit rules dekho | `sudo auditctl -l` |
| `ausearch -k <key>` | Audit events search karo | `sudo ausearch -k sudo_changes` |
| `aureport` | Audit summary report dekho | `sudo aureport --summary` |
| `umask` | Current umask value check karo | `umask` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **SSH service fails to start after port change** | SELinux ne naya port block kiya | `sudo semanage port -a -t ssh_port_t -p tcp <new_port>` |
| **Sudo command throws syntax error** | `/etc/sudoers` mein direct edit se typo | Root se login karo → `visudo` se error locate karo → fix karo |
| **Fail2ban not blocking IPs** | Log path galat ya jail enabled nahi | `jail.local` mein `logpath = /var/log/secure` check karo, `enabled = true` confirm karo |
| **User cannot SSH even with correct key** | `AllowUsers` mein username missing | `/etc/ssh/sshd_config` mein `AllowUsers` list mein user add karo → `systemctl restart sshd` |
| **SELinux denials on httpd** | File context galat hai | `sudo restorecon -Rv /var/www/html/` se default context restore karo |
| **auditd rules reboot pe lost** | Rules sirf `auditctl` se add kiye, file mein nahi likhe | `/etc/audit/rules.d/audit.rules` mein permanent rules likho |
| **fail2ban ne legitimate user ko ban kar diya** | User ne multiple baar password galat daala | `sudo fail2ban-client unban <IP>` → whitelist mein IP add karo `ignoreip = <IP>` |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: SSH Port Changed but Service Won't Start

> [!example] Ticket
> "We changed the SSH port to 2222 as per hardening policy, but now sshd service is failing to start. We are locked out of the server."

**L1 Response:** Console access le kar `systemctl status sshd` aur `journalctl -xe` check karo. Error mein "Permission denied" ya "SELinux" word dhundho.

**Escalation Trigger:** Agar `/var/log/audit/audit.log` mein SELinux denial messages dikh rahe hain.

**L2 Resolution:**
```bash
# SELinux mein naya port allow karo
sudo semanage port -a -t ssh_port_t -p tcp 2222
# Firewall rule bhi check karo
sudo firewall-cmd --add-port=2222/tcp --permanent
sudo firewall-cmd --reload
# sshd restart karo
sudo systemctl restart sshd
```

---

### 🎫 Scenario 2: Developer Locked Out After Sudoers Edit

> [!example] Ticket
> "I was editing sudoers file to add a new developer, but now no user can run sudo commands. Getting 'syntax error' message."

**L1 Response:** Verify karo ki koi bhi user `sudo` nahi chala pa raha. Console access se root login try karo.

**Escalation Trigger:** Jab multiple users affected hon aur root direct login bhi disabled ho SSH hardening ki wajah se.

**L2 Resolution:**
```bash
# Console/ILO se root login karo
# visudo se error fix karo — yeh syntax check karega
visudo
# Error line pe cursor jaayega — fix karo aur save karo
# Future mein hamesha visudo use karo, direct edit kabhi nahi!
```

> [!tip] Pro Tip
> *Hamesha `/etc/sudoers.d/` mein separate files banao. Agar ek file corrupt ho gayi toh baaki sab safe rahenge.*

---

### 🎫 Scenario 3: Fail2ban Ne Legitimate Admin Ko Ban Kar Diya

> [!example] Ticket
> "Our senior admin's IP got banned by fail2ban. He was trying to connect via SSH but kept getting 'Connection refused'. He says his password was correct."

**L1 Response:** Fail2ban banned IP list check karo: `sudo fail2ban-client status sshd`. Kya admin ka IP listed hai?

**Escalation Trigger:** Agar IP banned hai aur admin confirm karta hai ki password sahi tha — investigate kyun multiple failures log hue.

**L2 Resolution:**
```bash
# IP unban karo
sudo fail2ban-client unban 10.0.1.50

# Permanent whitelist mein daalo
sudo vi /etc/fail2ban/jail.local
# [DEFAULT] section mein add karo:
# ignoreip = 127.0.0.1/8 10.0.1.0/24

# Fail2ban restart karo
sudo systemctl restart fail2ban
```

---

### 🎫 Scenario 4: Web Application Files Not Accessible Despite Correct chmod

> [!example] Ticket
> "Apache web server is returning 403 Forbidden error. We checked file permissions and they are 755. Still not working."

**L1 Response:** `ls -lZ /var/www/html/` run karo — SELinux context check karo. Kya files ka context `httpd_sys_content_t` hai?

**Escalation Trigger:** Agar context galat hai ya `unconfined_u:object_r:default_t` dikh raha hai.

**L2 Resolution:**
```bash
# SELinux context restore karo
sudo restorecon -Rv /var/www/html/

# Verify karo
ls -lZ /var/www/html/
# Expected: system_u:object_r:httpd_sys_content_t:s0

# Apache restart karo
sudo systemctl restart httpd
```

==**Key Learning:** chmod correct hone se kaam nahi chalta agar SELinux context galat hai. DAC + MAC dono sahi hone chahiye!==

---
## 🎤 Interview Questions

> [!question] Q1: DAC aur MAC mein kya fark hai?
> **Answer:** ==DAC (Discretionary Access Control)== mein file owner decide karta hai ki kise access milega — jaise normal Linux `chmod`. ==MAC (Mandatory Access Control)== mein system-wide policy decide karti hai — SELinux enforce karta hai yeh policy, ==root bhi override nahi kar sakta.==
>
> **Real Example:** Agar root bhi `/var/www/html/` ko delete karne ki koshish kare aur SELinux mein `httpd_sys_content_t` context set hai — SELinux block kar dega.

> [!question] Q2: SELinux mode temporarily kaise change karte hain bina reboot ke?
> **Answer:** `sudo setenforce 1` → Enforcing mode. `sudo setenforce 0` → Permissive mode. ==Yeh changes reboot pe lost ho jaate hain.== Permanent change ke liye `/etc/selinux/config` edit karo aur `SELINUX=enforcing` set karo.

> [!question] Q3: `visudo` kyun use karte hain instead of directly `/etc/sudoers` edit karna?
> **Answer:** `visudo` do kaam karta hai: ==(1) File lock karta hai== taaki ek hi samay pe do log edit na karein, aur ==(2) Save karne se pehle syntax validate karta hai.== Agar typo ho toh save nahi hone deta. Direct edit mein ek galat line se ==poore system ka sudo kaam karna band ho sakta hai== — aur agar root SSH bhi disabled hai toh lockout ho jaoge.

> [!question] Q4: Kaise check karoge ki koi specific port SELinux mein allowed hai ya nahi?
> **Answer:** `sudo semanage port -l | grep <port_number>` ya service name se: `sudo semanage port -l | grep ssh_port_t`. Agar port listed nahi hai, toh `semanage port -a -t <type> -p tcp <port>` se add karo.

> [!question] Q5: fail2ban aur firewalld mein kya fark hai?
> **Answer:** ==firewalld ek static firewall hai== — tu manually rules set karta hai (allow/deny ports, IPs). ==fail2ban ek dynamic/reactive tool hai== — yeh logs monitor karta hai aur automatically offending IPs ko firewall mein ban karta hai. *Dono complementary hain — firewalld hamesha on rehna chahiye, fail2ban uske upar ek intelligent layer hai.*

> [!question] Q6: umask 027 ka matlab kya hai? Naye file aur directory ke permissions kya honge?
> **Answer:** umask 027 ka matlab:
> - ==Naye files: `640` (rw-r-----)== — Owner read+write, group read, others nothing
> - ==Naye directories: `750` (rwxr-x---)== — Owner full, group read+execute, others nothing
>
> *Calculation: Files → 666 - 027 = 640. Directories → 777 - 027 = 750.*

> [!question] Q7: Ek server pe security audit fail hua hai. Top 5 hardening steps kya honge?
> **Answer:**
> 1. ==SSH hardening== — Root login disable, key-based auth, non-standard port
> 2. ==SELinux Enforcing== mode on karo
> 3. ==fail2ban== install karo SSH protection ke liye
> 4. ==umask 027== set karo `/etc/profile` mein
> 5. ==sysctl== se IP forwarding band karo, SYN cookies enable karo
>
> *Bonus: auditd rules lagao critical files pe, unnecessary services disable karo.*

> [!question] Q8: `restorecon` command kya karta hai aur kab use karte hain?
> **Answer:** `restorecon` files ka ==SELinux context reset karta hai default policy ke according.== Jab bhi files copy/move karte ho toh context galat ho sakta hai. `sudo restorecon -Rv /path/` recursively sab files ka context fix karta hai. *Most common use case: Apache 403 errors jab files ka context `httpd_sys_content_t` nahi hota.*

==**Exam Tip:** RHCSA mein SELinux se related 2-3 questions pakka aate hain. `getenforce`, `setenforce`, `semanage`, `restorecon` — yeh 4 commands ratt lo!==

---
## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-09 SSH Configuration and Security|L-09 SSH Configuration and Security]] — SSH ki complete deep dive
- [[02-Operating-Systems/04-Linux-RHEL/L-06 File Permissions and Ownership|L-06 File Permissions and Ownership]] — DAC permissions ka foundation
- [[02-Operating-Systems/04-Linux-RHEL/L-19 SELinux-and-AppArmor|L-19 SELinux and AppArmor]] — SELinux ka advanced deep dive
- [[02-Operating-Systems/04-Linux-RHEL/L-20 Kernel-Tuning-and-sysctl|L-20 Kernel Tuning and sysctl]] — sysctl parameters ki complete guide
- [[02-Operating-Systems/04-Linux-RHEL/L-22 Firewalld|L-22 Firewalld]] — Firewall rules aur zones
- [[04-Cloud-and-Security/09-Security/CIA-Triad-and-Zero-Trust|CIA Triad and Zero Trust]] — Security ke foundational principles
