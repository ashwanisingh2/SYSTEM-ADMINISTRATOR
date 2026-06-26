---
tags: [linux, rhel, troubleshooting, masterclass]
aliases: [linux-troubleshooting, linux-masterclass]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX RHEL**

`#complete` `#advanced` `#rhcsa`

# L-20: Linux Troubleshooting Masterclass

> [!abstract] Overview
> Yeh note Linux/RHEL environments mein advanced troubleshooting scenarios, tools, aur methodology ko cover karta hai. Ek support engineer ke liye yeh extremely important hai kyunki jab production systems down hote hain, toh aapko exact pata hona chahiye ki kahaan dekhna hai aur issues ko step-by-step kaise isolate karna hai. It includes boot issues, performance bottlenecks, disk full problems, aur network connectivity issues. Is guide se aapko system failures ko debug karne ki ek structured approach milegi.

---
## 🧠 Concept Overview

- **What it is** — Systematically identifying, isolating, aur resolving problems Linux servers mein using built-in command-line tools aur logs. Ek scientific approach issues ko fix karne ki.
- **Why it matters** — Real job mein server downtime sidha revenue loss karta hai. Fast aur accurate troubleshooting aapko hero banati hai. Isse aap SLA breaches ko rok sakte hain.
- **Where you see this** — Boot failures (`grub` or `fstab` errors), high CPU/memory usage, "No space left on device" errors, zombie processes, aur network routing issues.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic alerts monitor karna, initial logs (`/var/log/messages`) check karna, service restart try karna. Server ping ho raha hai ya nahi dekhna. |
| **L2** | Advanced logs analyze karna (`journalctl`), disk space / inode issues fix karna, user lockouts resolve karna, network connectivity test karna using `curl`/`nc`. |
| **L3** | Kernel panics debug karna, crash dumps (kdump) analyze karna, system architecture or performance bottlenecks resolve karna, kernel parameters (`sysctl`) tune karna. |

> [!tip] Seedha Simple Mein
> *Troubleshooting ka matlab hai system ke symptoms dekh kar actual bimari (root cause) pakadna aur uska exact ilaaj (fix) apply karna. Andar se doctor wali feeling aati hai jab aisi commands chalate ho jo system ke internals dikhati hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Linux Troubleshooting** is like **being a Detective at a crime scene** because...
>
> - Jo crime scene (server) pe evidences (logs and metrics) hain, unhe dhyan se collect aur preserve karna padta hai.
> - Aap sidhe conclusion par nahi aate; pehle suspects (CPU, RAM, Disk, Network) ko question (commands) karte hain.
> - Ek choti si detail (error message in a random file ya ek galat permission) poora case solve kar sakti hai.

---
## 🔬 Technical Deep Dive

### 1. 🥾 Boot and Startup Issues

> [!info] Key Concept
> Linux boot process (BIOS -> MBR/GPT -> GRUB -> Kernel -> Systemd) mein kisi bhi stage par failure system ko unbootable bana sakta hai. Most common issues `/etc/fstab` misconfiguration ya corrupted GRUB bootloader ki wajah se aate hain.

*Jab system boot na ho, toh sabse pehle vCenter console access ya rescue mode (Live CD) ki zaroorat padti hai, kyunki network start nahi hua hota, so SSH kaam nahi karega.*

> [!danger] Common Mistake
> Fstab (`/etc/fstab`) mein nayi disk add karne ke baad `mount -a` se verify na karna. Agar galat entry save kardi aur server reboot kiya, toh server emergency mode mein chala jayega aur remote access completely loose ho jayega.

### 2. ⚡ Performance Bottlenecks

> [!info] Key Concept
> Performance issues mainly 4 major subsystems mein aate hain: CPU, Memory, Disk I/O, aur Network. 

*Humesha `top` ya `htop` se start karein to check load average. Agar load average CPU cores se zyada hai, it means system overload hai. Agar CPU wait (wa) percentage high hai, toh matlab disk I/O slow hai aur processor disk operations complete hone ka wait kar raha hai.*

### 3. 💾 Storage and File System Issues

> [!info] Key Concept
> "No space left on device" error hamesha sirf blocks (size) full hone par nahi aata, balki inodes full hone par bhi aa sakta hai (zyada choti files ki wajah se). File system corruption pe `fsck` run karna padta hai.

*Hum `df -h` blocks ke liye aur `df -i` inodes ke liye use karte hain. Ek aur common issue hai "deleted open files" jo disk dikhate hain full par `du` mein size nahi milta.*

### 4. 🌐 Network Connectivity Drops

> [!info] Key Concept
> Network issues mein firewall rules (iptables/firewalld), routing tables, ya closed ports ka problem hota hai.

*Hamesha OSI model follow karein. Pehle Physical/Link (interface up hai ya nahi), phir Network (IP assign hua hai, ping), phir Transport (port open hai), aur aakhir mein Application (curl).*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - RHEL/CentOS/Ubuntu system with root privileges
> - Basic understanding of common Linux command line utilities aur vi editor
> - Backup hamesha pehle le lena chahiye kisi bhi config file ko edit karne se pehle

### Step 1: Fixing a "Disk Full" Issue (Block and Inodes)

Kabhi kabhi user bolta hai space nahi hai par disk khali dikhti hai.

```bash
# Check block space usage
df -h

# Check inode usage
df -i

# Agar block full hai, find largest directories under /
du -sh /* | sort -hr | head -n 5

# Agar inode full hai, find directories with most files
find / -xdev -type d -size +100k -exec ls -ld {} \;
# Ya aasan tareeka using for loop
for i in /*; do echo $i; find $i |wc -l; done
```

> [!success] Expected Output
> ```
> Filesystem      Inodes   IUsed   IFree IUse% Mounted on
> /dev/sda1      1048576 1048576       0  100% /
> ```
> *Yahan dikh raha hai inode 100% use ho gaye hain. Ab inko delete karna hoga carefully.*

### Step 2: Recovering Space from Deleted Open Files

Bahut bar aap log file delete kar dete ho par disk space free nahi hoti.

```bash
# Check if any deleted files are still held open by processes
lsof | grep deleted

# The output will show process name and PID
# Restart that specific service to free up space
systemctl restart <service_name>
```

### Step 3: Troubleshooting High Load Average

Server slow respond kar raha hai aur CPU utilization dekhni hai.

```bash
# Check system load and running processes
top
# Press '1' to see all CPU cores
# Press 'M' to sort by Memory, 'P' to sort by CPU

# Check historical load (sar -q) - if sysstat is installed
sar -q
```

### Step 4: Analyzing System Logs with Journalctl

Systemd environment mein logs efficiently filter karna aana chahiye bina `cat` ya `grep` kiye pure file ko.

```bash
# Get logs for a specific service since last boot
journalctl -u httpd.service -b

# Check all error priority logs
journalctl -p err

# Follow logs in real time (like tail -f)
journalctl -f
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `top` / `htop` | System performance, CPU/Memory usage realtime dikhata hai | `top -c` (shows absolute path of command) |
| `free -h` | Free aur used RAM human-readable format mein dikhata hai | `free -h` |
| `df -hT` | Disk space usage (blocks) check karta hai with file system type | `df -hT /var` |
| `df -i` | Inode usage check karta hai file limits janne ke liye | `df -i /var` |
| `du -sh *` | Current folder ke andar kiska size kitna hai batata hai | `du -sh /var/log/*` |
| `ss -tulnp` | Listening network ports aur unke piche konsa process run ho raha hai, woh dikhata hai | `ss -tulnp \| grep 80` |
| `dmesg -T` | Kernel ring buffer (hardware/driver issues) check karta hai with human-readable timestamps | `dmesg -T \| grep -i error` |
| `lsblk -f` | Block devices (disks/partitions) ka tree structure aur filesystem dikhata hai | `lsblk -f` |
| `journalctl -xe` | Systemd logs ke end mein jump karta hai errors trace karne ke liye | `journalctl -u sshd -xe` |
| `lsof` | List open files. Kaunsa process kis file ko use kar raha hai batata hai | `lsof -i :80` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Server is very slow, SSH taking time** | High CPU Load or out of memory (OOM killer active, system swapping heavily). | Run `top`, check top CPU/Memory consumers. Check `/var/log/messages` for OOM killer logs. Restart/kill problematic process. |
| **"No space left on device" error** | Ya toh disk block full ho gaya hai, ya inodes exhaust ho gaye hain. | `df -h` and `df -i`. Clean up old logs (`/var/log`) or temporary files (`/tmp`). Check open deleted files. |
| **Service (e.g., Apache/Nginx) fails to start** | Configuration syntax error, port pehle se in-use hai, ya permissions issue (SELinux blocking). | `systemctl status <service>`, `journalctl -xe`, test config `nginx -t`, check port `ss -tulnp \| grep 80`. |
| **User locked out or password expired** | `/etc/shadow` mein account expire ho gaya hai ya multiple fail attempts ki wajah se pam_tally2/faillock locked hai. | `faillock --user <user> --reset` ya `passwd -u <user>` to unlock. Change password if expired. |
| **System boots into Emergency Mode** | Most common cause is `/etc/fstab` mein invalid entry ya corrupted file system causing mount failure. | Login with root password. Check `journalctl -xb`. Open `/etc/fstab`, comment wrong entry, run `mount -a`, `reboot`. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Critical Web Server Down (Out of Memory)

> [!example] Ticket
> "URGENT: Corporate website is not loading. Monitoring shows the server is pingable but HTTP is timing out constantly."

**L1 Response:** Verify connectivity and basic service status. Check if Apache/Nginx service is running. Try to SSH into the server. If SSH works but very slow, escalate if basic restart doesn't help.
**Escalation Trigger:** Service restart fails and `free -h` shows 0 available memory with Swap completely full.
**L2 Resolution:** 
1. SSH into the server and run `dmesg -T | grep -i oom` aur check journalctl.
2. *Pata chala ki OOM-Killer ne Java backend process ko kill kar diya tha out of memory ki wajah se.*
3. Run `top` and check what is consuming memory right now. 
4. Check process limits. Identify a memory leak in the application code with the developer team. Increase instance memory size temporarily (if VM/Cloud).
5. Restart the web service and monitor usage.

### 🎫 Scenario 2: Can't create new files, disk full error

> [!example] Ticket
> "Database backups are failing with 'No space left on device' but `df -h` shows 30GB free space on `/data` partition."

**L1 Response:** Confirms space issue by creating a test file (`touch /data/test`). It fails. Escalate to L2 because block size shows free space but it still fails.
**Escalation Trigger:** Blocks space is free but file creation fails consistently.
**L2 Resolution:** 
1. Run `df -i /data` to check inodes. Inode usage is at 100%.
2. *Wajah yeh thi ki application roz lakho zero-byte session files create kar rahi thi aur unko delete nahi kar rahi thi.*
3. Run `find /data/sessions -type f -mtime +7 -delete` (Dhyan se chalayein!). Better to use `find /data/sessions -type f | xargs rm` if file count is too huge for simple rm.
4. Issue resolved, space and inodes freed up. Setup a cronjob to clear old sessions automatically in the future.

### 🎫 Scenario 3: Remote Server Unavailable After Reboot

> [!example] Ticket
> "After regular OS patching and reboot, server 'APP-SRV-01' is not coming up on network. Cannot SSH or ping."

**L1 Response:** Ping server. Fails. Check vCenter/Cloud console for VM state. Shows VM is powered on but stuck on a black screen with text.
**Escalation Trigger:** Server stuck during boot process. Needs console access.
**L2 Resolution:** 
1. Access server via vCenter/iLO console. Console shows system is in "Emergency Mode" and prompting for root password.
2. Provide root password to enter maintenance mode.
3. Run `journalctl -xb` or `cat /run/initramfs/rdsosreport.txt` to find the exact mount failure. 
4. *Pata chala ki kisi ne `/etc/fstab` mein ek nayi storage LUN ki entry ki thi bina `nofail` flag ke, aur SAN connectivity issues se woh disk attach nahi hui thi.*
5. Open `/etc/fstab` using `vi`, comment out the problematic line, save, and type `reboot`. Server boots successfully.

---
## 🎤 Interview Questions

> [!question] Q1: Jab aapka server slow respond kar raha hai, toh aap pehla step kya karenge?
> **Answer:** Main sabse pehle `top` command chalaunga system ka load average, CPU, aur RAM status dekhne ke liye. Agar load zyada hai, toh top consuming process isolate karunga. Phir disk I/O check karne ke liye `iostat` aur network check ke liye `ss`/`netstat` use karunga.

==**Exam Tip:** *Humesha structured approach show karein. Direct restart bolna galat hai, pehle root cause dhoondna zaroori hai troubleshooting mein.*==

> [!question] Q2: "No space left on device" error aa raha hai par `df -h` dikha raha hai 50% free. Kya wajah ho sakti hai aur kaise resolve karenge?
> **Answer:** Inodes exhaust ho gaye hain. Linux mein har file ek inode leti hai. Agar millions of small files ban jayein (jaise session files ya email queues), toh disk space blocks bache honge but inodes khatam ho jayenge. Isko `df -i` command se confirm kar sakte hain aur un choti unnecessary files ko identify karke delete karna hoga.

> [!question] Q3: `/etc/fstab` edit karte waqt kaunsi best practice follow karni chahiye?
> **Answer:** Nayi entry dalne ke baad **humesha** `mount -a` command run karna chahiye. Yeh test karega ki fstab ki entries sahi mount ho rahi hain ya nahi bina reboot kiye. Agar koi error ho, toh system reboot karne se pehle use theek kar lena chahiye taaki server emergency mode mein na jaye. Also, non-critical / external mounts ke liye `nofail` option use karna chahiye.

> [!question] Q4: Ek log file delete kar di hai, phir bhi `df -h` mein disk space free nahi hui. Yeh kyun hota hai aur ise bina server reboot kiye kaise fix karein?
> **Answer:** Yeh tab hota hai jab woh file abhi bhi kisi running process (like a web server or database) ke open file descriptor dwara held ho. Linux mein space tabhi free hoti hai jab dono file link aur open file reference close (zero) ho jayein. Ise dhoondne ke liye main `lsof | grep deleted` run karunga. Phir us particular process ko gracefully reload ya restart kar dunga, toh space turant free ho jayegi.

> [!question] Q5: Server load average `12.50, 10.20, 8.10` dikha raha hai ek 4-core CPU system pe. Iska kya matlab hai?
> **Answer:** Load average numbers 1-minute, 5-minute, aur 15-minute intervals ka data dikhate hain. 4-core system mein 4.0 load ka matlab CPU 100% utilized hai. `12.50` load on a 4-core system matlab CPU severely overloaded hai aur processes CPU time ke liye queue mein wait kar rahe hain. Is situation mein humein `top` ya `htop` use karke heavy CPU consuming process ko identify karna hoga aur zaroorat padne par use kill (`kill -9`) ya nice value dekar priority kam karni hogi.

---
## 🔗 Related Notes

- [[02-Operating-Systems/04-Linux-RHEL/L-04 Linux File System Hierarchy|L-04 Linux File System Hierarchy]] — *For understanding paths mentioned.*
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Linux Disk Management|L-08 Linux Disk Management]] — *For LVM and partitioning troubleshooting details.*
- [[02-Operating-Systems/04-Linux-RHEL/L-12 Linux Network Configuration|L-12 Linux Network Configuration]] — *For deeper network troubleshooting tools.*
- [[02-Operating-Systems/04-Linux-RHEL/L-15 Linux Process Management|L-15 Linux Process Management]] — *For managing, killing, and nice values.*
