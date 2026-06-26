---
tags: [security, linux, hardening, ssh, selinux]
aliases: [Linux Security, System Hardening]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-red]
> 🛡️ **SECURITY**

`#complete` `#intermediate` `#none`

# SEC-02: Linux Security Basics

> [!abstract] Overview
> Yeh note Linux security ke basic concepts, user management, file permissions, aur system hardening ko cover karta hai. Ek support engineer ke liye yeh jaanna bahut zaroori hai kyunki kisi bhi server ko secure rakhna hamari pehli responsibility hoti hai. Bina proper security ke, servers easily compromise ho sakte hain aur data loss ya data breach ka risk badh jata hai. Is note mein hum core concepts jaise UGO permissions, SSH hardening, aur SELinux basics par focus karenge.

---
## 🧠 Concept Overview

- **What it is** — Basic security practices and principles to protect a Linux operating system from unauthorized access, vulnerabilities, and malicious attacks.
- **Why it matters** — Real job mein server compromise hone se bachane ke liye aur data integrity maintain karne ke liye. Ek secure system downtime ko reduce karta hai aur compliance standards (jaise PCI-DSS ya HIPAA) ko meet karne mein help karta hai.
- **Where you see this** — Har naye server deployment mein, routine security audits mein, aur jab bhi koi user data breach ya access denied ka ticket raise karta hai.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Password resets, checking basic file permissions (`ls -l`), monitoring system logs for failed logins (`/var/log/secure` ya `/var/log/auth.log`), aur basic access tickets handle karna. |
| **L2** | Configure, fix, escalate kab karta hai — Configuring SSH keys, setting up firewalls (firewalld/iptables/UFW), fixing SELinux contexts, aur file permission issues troubleshoot karna (SUID/SGID). |
| **L3** | Architecture, design, enterprise-level — Designing overall security policies, automated system hardening via Ansible playbooks, integrating centralized authentication (LDAP/AD/FreeIPA), aur vulnerability assessments karna. |

> [!tip] Seedha Simple Mein
> *Linux security ka matlab hai sirf right logon ko right files aur commands ka access dena, aur baaki sab malicious actors ko block karna. Jaise aap apne ghar ka lock aur chaabi kisi anjaan ko nahi dete, waise hi server ka access secure rakhna padta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **A High-Security Office Building** is like **Linux Security** because...
>
> - **Main Gate Guard (Firewall):** Check karta hai ki kon building ke andar aa sakta hai (Traffic filtering).
> - **ID Badge / Access Card (SSH Keys / Passwords):** Aapke pass entry ke liye ek unique identity proof hota hai. Sirf valid ID wale hi andar ja sakte hain.
> - **Room Keys (File Permissions - UGO):** Building ke andar ke kamron ka access depend karta hai ki aap Employee ho (Owner), specific Department/Team ke ho (Group), ya phir Visitor/Guest ho (Others).
> - **Internal Security Cameras (Audit Logs/Syslog):** Har activity monitor hoti hai ki kaun kahan gaya aur usne kya kiya.

---
## 🔬 Technical Deep Dive

### 1. File Permissions (UGO & RWX)

> [!info] Key Concept
> Linux permissions are divided into three main categories: User (Owner), Group, and Others. The permissions granted can be Read (r=4), Write (w=2), and Execute (x=1).

Linux mein har file aur directory ki ek permission set hoti hai jo exactly decide karti hai ki kaun us resource ko padh sakta hai, modify kar sakta hai, ya run kar sakta hai.

*Permissions ko hum numeric (octal) format (jaise 755 ya 644) ya symbolic format (jaise rwxr-xr-x) mein represent karte hain.*

**Detailed Breakdown:**
- **Read (r / 4):** File ke contents dekhne ke liye, ya directory ki listing (`ls`) karne ke liye.
- **Write (w / 2):** File ko edit/delete karne ke liye, ya directory ke andar nayi files banane ke liye.
- **Execute (x / 1):** Script/binary ko run karne ke liye, ya directory ke andar `cd` (change directory) karne ke liye.

> [!danger] Common Mistake
> `chmod 777` use karna ek fixing shortcut ki tarah. *Yeh kabhi mat karna production server par, iska matlab aapne internet par kisi ko bhi us file/folder ka full access de diya hai.*

### 2. SSH Hardening (Secure Shell)

> [!info] Key Concept
> Securing the SSH daemon (sshd) is critical to prevent unauthorized remote access. This is usually done by disabling root login, changing the default port, and enforcing key-based authentication over passwords.

SSH default port 22 par chalta hai. Agar isko securely configure nahi kiya, toh internet par bots lagaatar brute-force attacks try karte rehte hain (dictionary attacks).

*Hamesha direct root login ko disable karna chahiye. Ek normal user se login karke zaroorat padne par `sudo` ka istemal karna is the best practice.*

### 3. SELinux (Security-Enhanced Linux)

> [!info] Key Concept
> SELinux provides a mechanism for supporting access control security policies, including Mandatory Access Controls (MAC).

Traditional Linux security (Discretionary Access Control - DAC) ke upar SELinux ek extra strict layer hai. Yeh processes aur files ko restrict karti hai unke "context" ke hisaab se. Agar process ke context aur file ke context match nahi hote, toh root user hone ke bawajood access deny ho jayega.

*Agar permissions (777) dene ke baad bhi service start nahi ho rahi ya file read nahi ho rahi, toh 90% chance hai ki yeh SELinux ka context issue hai.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - RHEL/CentOS/Ubuntu VM (Internet connected)
> - Sudo privileges
> - Basic understanding of terminal editors like `vim` or `nano`

### Step 1: Securing SSH Configuration

```bash
# Open sshd_config file for editing
sudo vi /etc/ssh/sshd_config
```

Find and change these specific lines to enhance security:
```text
# Disable direct root login
PermitRootLogin no

# Disable password authentication (ensure SSH keys are setup first!)
PasswordAuthentication no

# Optional: Change default port to avoid simple bot scanners
Port 2222
```

```bash
# Restart SSH service to apply changes
sudo systemctl restart sshd
```

> [!success] Expected Output
> Service restarts cleanly. Uske baad direct root login try karne par "Permission denied (publickey)" aayega. 

### Step 2: Fixing and Auditing File Permissions

```bash
# Set secure permissions for a sensitive file like shadow
sudo chmod 600 /etc/shadow

# Verify the changes
ls -l /etc/shadow
```

> [!success] Expected Output
> `ls -l /etc/shadow` will strictly show `-rw-------. root root` indicating only root can read and write.

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `chmod` | Change file permissions. *File permission modify karna.* | `chmod 644 file.txt` |
| `chown` | Change file owner and group. *File ka owner badalna.* | `chown user:group file.txt` |
| `getenforce` | Check current SELinux status. *SELinux active hai ya nahi dekhna.* | `getenforce` |
| `setenforce 0` | Put SELinux in permissive mode temporarily. *Temporarily SELinux ko disable/log mode mein dalna.* | `setenforce 0` |
| `restorecon` | Restore default SELinux security contexts. *File context reset karna.* | `restorecon -Rv /var/www/html/` |
| `semanage port` | Manage SELinux port contexts. *Custom port ko bind karne ke liye allow karna.* | `semanage port -a -t http_port_t -p tcp 8080` |
| `last` | Show listing of last logged in users. *System par kon kab login hua dekhna.* | `last -n 10` |
| `faillock` | Show failed login attempts. *Galat password attempts ki history dekhna.* | `faillock --user admin` |
| `chage` | Manage user password expiry. *Password kab expire hoga check karna.* | `chage -l username` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| Web server starting but getting "403 Forbidden" or "Permission Denied" | SELinux context is incorrect. *SELinux web server process ko folder read nahi karne de raha.* | Run `restorecon -Rv /var/www/html/` to fix contexts. |
| SSH connection refused on port 22 | Firewall blocking port 22 or sshd service is down. *Ya toh firewall rule missing hai ya SSH daemon stop hai.* | `systemctl status sshd` and `firewall-cmd --add-service=ssh --permanent && firewall-cmd --reload` |
| User account locked out | Too many failed login attempts triggered PAM rules. *Kisi ne baar baar galat password dala aur account lock ho gaya.* | `faillock --user <username> --reset` to unlock. |
| Cannot `cd` into a directory | User lacks Execute (x) permission on that directory. *Folder mein enter hone ke liye hamesha execute permission chahiye hoti hai.* | `chmod +x directory_name` |
| Unable to write to a file despite being in the group | The group write permission is missing. *Group permissions mein write (w) bit set nahi hai.* | `chmod g+w filename` |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: User Account Locked Automatically

> [!example] Ticket
> "User 'john_doe' is unable to SSH into the database server. Getting account locked error continuously."

**L1 Response:** Check failed login attempts using `faillock --user john_doe`. Check if the password has naturally expired using `chage -l john_doe`. *Basic checks run karna.*
**Escalation Trigger:** If the lock is due to continuous brute-force attacks from an unknown external IP.
**L2 Resolution:** Reset the lock with `faillock --user john_doe --reset`. If it's a brute-force attack, identify the IP from `/var/log/secure` and block the IP using `iptables` or `firewall-cmd`. Configure `fail2ban` for automated blocking.

### 🎫 Scenario 2: Web Server Cannot Access New Code Directory

> [!example] Ticket
> "Developer team uploaded new application code to `/var/www/custom_app` but Nginx is throwing 403 Forbidden."

**L1 Response:** Check standard file permissions (`ls -l`). Verify the Nginx user (e.g., `nginx` or `www-data`) has read and execute access to the directories. *File permissions verify karna.*
**Escalation Trigger:** Permissions are correctly set to `755` but the server is still getting 403.
**L2 Resolution:** Check SELinux context with `ls -Z`. Change context using `chcon -R -t httpd_sys_content_t /var/www/custom_app` and make it permanent with `semanage fcontext -a -t httpd_sys_content_t "/var/www/custom_app(/.*)?"`. *SELinux context ko fix karna.*

### 🎫 Scenario 3: Custom Application Port Binding Failure

> [!example] Ticket
> "Configured Tomcat application server to run on port 9090 instead of default 8080, but it fails to start and bind to the port."

**L1 Response:** Check if the port 9090 is already in use by another application (`netstat -tulnp | grep 9090` or `ss -tulnp`). *Dekhna ki port busy toh nahi hai.*
**Escalation Trigger:** Port is free, but service logs still show "Permission denied" while binding.
**L2 Resolution:** The custom port is blocked by SELinux. Add the custom port to SELinux allowed ports for Tomcat. `semanage port -a -t tomcat_port_t -p tcp 9090`. Restart the Tomcat service.

### 🎫 Scenario 4: User Cannot Execute Script

> [!example] Ticket
> "Junior admin is trying to run a backup script `/opt/scripts/backup.sh` but getting Permission Denied."

**L1 Response:** Run `ls -l /opt/scripts/backup.sh` to check permissions. *Script par execute bit check karna.*
**Escalation Trigger:** N/A - Usually resolved at L1.
**L2 Resolution:** Grant execute permission using `chmod +x /opt/scripts/backup.sh`. Ensure the script has the correct shebang (`#!/bin/bash`) at the top.

---
## 🎤 Interview Questions

> [!question] Q1: What is the exact difference between `chmod 777` and `chmod 755`?
> **Answer:** `777` gives Read, Write, and Execute permissions to Owner, Group, and Others (Everyone). `755` gives Read, Write, Execute to the Owner, but only Read and Execute to Group and Others. *777 ka matlab koi bhi internet par aakar file delete/edit kar sakta hai, jo system ke liye extreme risk hai.*

==**Exam Tip:** Never suggest `chmod 777` as a solution in a technical interview. Always use the principle of least privilege.==

> [!question] Q2: How do you prevent direct root logins via SSH?
> **Answer:** Edit the main SSH configuration file `/etc/ssh/sshd_config`, set `PermitRootLogin no`, and then restart the sshd service. *Direct root login disable karke ek normal limited user se login karna chahiye, fir `sudo` use karna chahiye.*

> [!question] Q3: When a user creates a new file, what are the default permissions and how is it calculated?
> **Answer:** Default permissions are determined by subtracting the `umask` value from the base permissions. For files, the starting base is 666 (rw-rw-rw-), and if umask is 022, the result is 644 (rw-r--r--). For directories, the base is 777. *Umask decide karta hai ki nayi file banne par use by default kya permission milegi.*

> [!question] Q4: What are the three states of SELinux and how do you check the current state?
> **Answer:** The three states are Enforcing (blocks and logs), Permissive (only logs, does not block), and Disabled. You can check the current state using the `getenforce` or `sestatus` command. *Enforcing matlab active aur protect karega, Permissive matlab audit mode.*

> [!question] Q5: What is a SUID bit and why is it considered a security risk if misconfigured?
> **Answer:** Set User ID (SUID) allows a user to execute a file with the permissions of the file owner (usually root), e.g., the `passwd` command. If a vulnerable script has SUID set, an attacker can use it to gain unauthorized root access. *SUID normal user ko temporarily root power deta hai ek specific command ke liye.*

> [!question] Q6: How do you check which users have recently logged into the system?
> **Answer:** You can use the `last` command to view a history of logins. For a specific user, you can run `last username`. You can also review the `/var/log/secure` or `/var/log/auth.log` files for detailed authentication logs. *Log files system security audit ka primary source hote hain.*

---
## 🔗 Related Notes

- [[SEC-01 Introduction to IT Security|SEC-01 Introduction to IT Security]] — Fundamental enterprise security concepts.
- [[LIN-05 Linux File Permissions|LIN-05 Linux File Permissions]] — Deep dive into standard and advanced permissions (SUID/SGID/Sticky Bit).
- [[LIN-08 SELinux Basics|LIN-08 SELinux Basics]] — Detailed SELinux commands, policies, and advanced troubleshooting.
- [[LIN-10 SSH Key Management|LIN-10 SSH Key Management]] — How to properly generate and distribute SSH keys.
