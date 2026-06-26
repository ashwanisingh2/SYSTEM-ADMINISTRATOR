---
tags: [automation, ansible, windows, sysadmin]
aliases: [ansible-windows-admins, ans-01]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-red]
> 🤖 **ANSIBLE**

`#complete` `#intermediate` `#none`

# ANS-01: Ansible for Windows Admins

> [!abstract] Overview
> Ansible is an open-source, agentless IT automation engine. Although the Ansible control node must run on Linux, it can manage Windows client and server operating systems using Windows Remote Management (WinRM) or SSH. This note details setting up Ansible for Windows, directory structures, inventories, and common Windows modules. *Ansible ek bahut hi powerful tool hai jo server configuration ko automate karne me madad karta hai.*

---
## 🧠 Concept Overview

- **What it is** — Ansible is a configuration management and orchestration tool that uses declarative YAML files (Playbooks) to define the desired state of target machines. *Ansible YAML files use karke servers ko configure karta hai.*
- **Why it matters** — Configuring 50 Windows servers manually is slow and error-prone. Ansible enables you to deploy features, create users, and edit registries uniformly across hundreds of machines in minutes. *Manual kaam me galtiyan ho sakti hain aur time bhi lagta hai. Ansible se ek sath saikdon servers manage kiye ja sakte hain.*
- **Where you see this** — Automating the installation of IIS web servers, setting registry policies for security compliance, or automating weekly service restarts across cluster servers. *Jaise agar aapko 100 servers par IIS install karna ho toh aap Ansible ka use kar sakte hain.*

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Executes pre-designed playbooks using `ansible-playbook`, reads command output logs, and checks target connection status. *L1 engineers bane banaye playbooks run karte hain aur logs check karte hain.* |
| **L2** | Configures WinRM listeners on Windows targets, sets up inventory hosts files, writes basic playbooks using Windows modules, and manages variables. *L2 level pe hum playbooks likhte hain aur inventory setup karte hain.* |
| **L3** | Integrates Ansible with Active Directory for domain-based execution, constructs modular Ansible Roles, and manages credential secrets using Ansible Vault. *L3 wale bade architecture, roles, aur vault secrets ko handle karte hain.* |

> [!tip] Seedha Simple Mein
> *Ansible ek control server (Linux) se saare Windows servers ko automate karne ka tool hai. Isme hume har server pe koi agent install nahi karna padta. Hum ek simple text file (Playbook) me tasks likhte hain aur woh saare target Windows servers par ek sath execute ho jate hain. Jaise hi aap command doge, automatically sab set ho jayega.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Head Chef of a Restaurant Chain** is like **Ansible** because...
>
> - Instead of visiting each of the 50 branches to teach a recipe, you write it down once in a **recipe book** (Playbook). *Chef ko har branch me jaana nahi padta, wo bas ek recipe book likhta hai.*
> - You send it to all branch managers (target servers) through **WhatsApp** (WinRM connection). *Messages WinRM ke through bheje jaate hain.*
> - The managers follow the exact steps — no variation, no mistakes. *Sab bilkul same steps follow karte hain.*
> - You never need to physically visit (install an agent on) any branch! *Bina agent install kiye kaam ban jata hai.*

---
## 🔬 Technical Deep Dive

### 1. Agentless Architecture & WinRM Setup

> [!info] Key Concept
> Unlike other tools (like Puppet or Chef), Ansible is **agentless**. It connects to Windows servers using **WinRM (Windows Remote Management)**. *Bina kisi third-party software ke, Ansible sidhe WinRM ka use karta hai.*

* WinRM operates over HTTP (Port 5985) or HTTPS (Port 5986). *In ports par WinRM communication karta hai.*
* ==HTTPS is highly recommended for security.== *Production me hamesha HTTPS hi use karein.*
* Target Windows machines require a PowerShell configuration script to enable listeners, configure firewall rules, and allow basic/certificate-based authentication. *Humein Windows target par listener configure karna hota hai taki Ansible usse connect ho sake.*
* Some environments also configure OpenSSH for Windows instead of WinRM, but WinRM is the most widely adopted standard natively built into Windows Server OS. *Kayi log OpenSSH bhi use karte hain, par WinRM default hota hai.*

### 2. Inventory Files for Windows

> [!danger] Common Mistake
> Storing plaintext passwords in the `hosts` file in a production environment. Always use **Ansible Vault** for credentials! *Kabhi bhi production inventory file me password plaintext me mat save karo. Hamesha Ansible Vault ka use karo secure rakhne ke liye.*

The inventory file (usually named `hosts`) lists the target servers. To manage Windows nodes, specific connection variables must be defined so that Ansible knows not to use SSH. *Hosts file me targets define hote hain.*

```ini
# /etc/ansible/hosts
[windows]
winserver01.company.local
winserver02.company.local

[windows:vars]
ansible_user=Administrator
ansible_password=SecurePassword123!
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

### 3. Key Ansible Modules for Windows

Here is a look at the most common Windows-specific modules used in Ansible playbooks. *Yeh kuch sabse zyada use hone wale Windows modules hain.*

| 🧩 Module | 🛠️ Purpose |
|---------|----------|
| `ansible.windows.win_service` | Manages Windows Services (starts, stops, restarts). *Windows ki services ko start ya stop karne ke liye.* |
| `ansible.windows.win_user` | Manages local user accounts. *Local users create ya delete karne ke liye.* |
| `ansible.windows.win_feature` | Installs or uninstalls Windows Roles/Features. *IIS jaise roles install karne ke liye.* |
| `ansible.windows.win_copy` | Copies files from the control node to target Windows paths. *Linux se Windows par files copy karne ke liye.* |
| `ansible.windows.win_regedit` | Adds, edits, or deletes Windows Registry keys. *Registry entries modify karne ke liye.* |
| `ansible.windows.win_shell` | Executes arbitrary PowerShell commands on target nodes. *Custom PowerShell command chalane ke liye.* |

==**Exam Tip:** Windows modules in Ansible are usually prefixed with `win_` (e.g., `win_ping`, `win_copy`). Linux uses standard modules (`ping`, `copy`).==

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A Linux Control Node (Ubuntu/RHEL) with Ansible installed.
> - A target Windows Server 2022 instance reachable over network.
> - Admin credentials on the target Windows Server.
> *Linux node aur target Windows par connectivity check kar lena pehle.*

### Step 1: Configure WinRM on target Windows Server

*Sabse pehle target server par WinRM enable karna hoga.*

```powershell
# Enable WinRM service and configure listeners
winrm quickconfig -q

# Set basic authentication and allow unencrypted transport (for HTTP testing)
Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true
Set-Item WSMan:\localhost\Service\AllowUnencrypted -Value $true
```

> [!success] Expected Output
> WinRM starts listening on Port 5985.

### Step 2: Configure Inventory on Linux Control Node

*Linux server pe `/etc/ansible/hosts` me variables set karenge.*

```bash
# Edit the inventory file
sudo vi /etc/ansible/hosts
```

```ini
[win_servers]
192.168.1.105

[win_servers:vars]
ansible_user=Administrator
ansible_password=MySecretPassword123
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

```bash
# Verify host connectivity
ansible win_servers -m ansible.windows.win_ping
```

> [!success] Expected Output
> Target returns: `"ping": "pong"`. *Agar pong aaya, matlab connectivity sahi hai.*

### Step 3: Write a Playbook to Install Web Server (IIS)

*Ab hum ek YAML playbook banayenge jisme IIS installation ke tasks honge.*

```bash
# Create playbook file
vi iis_install.yml
```

```yaml
---
- name: Configure Web Servers
  hosts: win_servers
  tasks:
    - name: Install IIS Web Server Role
      ansible.windows.win_feature:
        name: Web-Server
        state: present
        include_management_tools: yes

    - name: Ensure IIS Service is Started and Automatic
      ansible.windows.win_service:
        name: w3svc
        state: started
        start_mode: auto

    - name: Deploy Default Index Page
      ansible.windows.win_copy:
        content: "<h1>Welcome to Automated IIS Server via Ansible!</h1>"
        dest: C:\inetpub\wwwroot\index.html
```

> [!success] Expected Output
> Playbook file is successfully saved.

### Step 4: Execute Playbook and Verify

*Playbook run karenge aur result dekhenge.*

```bash
# Run the playbook
ansible-playbook iis_install.yml
```

> [!success] Expected Output
> Playbook logs display execution runs showing `changed=3` for the host. Browse to `http://192.168.1.105` to view the page. *Browser me check karo, test page dikhega.*

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `ansible [group] -m win_ping` | Ping Windows targets using WinRM. *Windows server ko ping karke check karta hai.* | `ansible win_servers -m win_ping` |
| `ansible-playbook [file.yml]` | Run an Ansible Playbook. *YAML file run karne ke liye.* | `ansible-playbook install_iis.yml` |
| `ansible-vault encrypt [file]`| Encrypt credential files. *Sensitive data encrypt karta hai.* | `ansible-vault encrypt hosts` |
| `ansible [group] -m win_service`| Restart a service (Ad-hoc). *Ad-hoc command se service restart karna.* | `ansible win_servers -m win_service -a "name=spooler state=restarted"` |
| `ansible-galaxy collection install` | Install additional modules from Galaxy. *Naye collections install karne ke liye.* | `ansible-galaxy collection install ansible.windows` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| `win_ping` fails with connection refused | WinRM service is not running or blocked by firewall. *WinRM service band hai ya firewall usko block kar rahi hai.* | Run `Get-Service winrm`. Allow port `5985/5986` in Windows Defender Firewall |
| Authentication fails with 401 Unauthorized | Credentials correct but basic auth disabled. *Password sahi hai par Basic authentication allow nahi kiya.* | Run `Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true` on Windows |
| Playbook fails due to SSL errors | Self-signed cert is not trusted by control node. *Ansible Windows ke certificate ko trust nahi kar raha.* | Set `ansible_winrm_server_cert_validation=ignore` in inventory variables |
| Missing module error | Ansible `windows` collection not installed. *Windows modules Ansible me installed nahi hain.* | Run `ansible-galaxy collection install ansible.windows` on the Linux node |
| Execution policy blocks scripts | Windows is restricting PowerShell scripts. *Windows PowerShell scripts ko block kar raha hai.* | Set `ExecutionPolicy RemoteSigned` on the target Windows server |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: IIS Deployment Playbook Fails

> [!example] Ticket
> "I am trying to run the `iis_install.yml` playbook, but it keeps failing at the gathering facts stage with a 401 Unauthorized error."

**L1 Response:** Verify that the control node can reach the Windows server over port 5985. Run `ansible win_servers -m win_ping -v` for verbose output. *Connectivity check karo aur dekho error exactly kya hai.*
**Escalation Trigger:** If ping fails with 401 even though the password is correct.
**L2 Resolution:**
1. Check the target Windows server's WinRM config.
2. Confirm that Basic Authentication is enabled in WinRM. *Basic auth enable karna hoga.*
3. Run `Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true`.
4. Run `win_ping` again.

### 🎫 Scenario 2: Playbook Freezes Indefinitely

> [!example] Ticket
> "Our monthly Windows patching playbook is stuck on one server and won't progress or fail."

**L1 Response:** Identify the exact task where it froze. Check network connectivity to the target server. *Pata lagao kis task pe ruk gaya hai.*
**Escalation Trigger:** If the server is up but Ansible hangs.
**L2 Resolution:** 
1. Log into the target server and check the CPU/Memory usage.
2. Sometimes, a Windows pop-up (like a UAC prompt or an installer GUI) is blocking the command-line execution invisibly. *Kabhi kabhi background me installer block kar deta hai.*
3. Add `become: yes` and `become_method: runas` in the playbook to properly execute tasks with elevated privileges in the background.

### 🎫 Scenario 3: Missing Ansible Collection Error

> [!example] Ticket
> "When I run my new playbook using the `ansible.windows.win_feature` module, it throws a 'module not found' error."

**L1 Response:** Check the playbook for typos in the module name. Ensure you're specifying the fully qualified collection name `ansible.windows.win_feature`. *Check karo naam sahi se likha hai ya nahi.*
**Escalation Trigger:** If the name is correct but the error persists.
**L2 Resolution:**
1. This means the required Ansible Galaxy collection for Windows is missing from the Linux control node. *Windows modules default Ansible installation me nahi aate kabhi kabhi.*
2. Run `ansible-galaxy collection install ansible.windows` on the control node.
3. Once installed, re-run the playbook.

---
## 🎤 Interview Questions

> [!question] Q1: How does Ansible manage Windows servers without installing an agent?
> **Answer:** Ansible uses **WinRM (Windows Remote Management)** (over HTTP port 5985 or HTTPS port 5986) or OpenSSH. Instead of running Python code directly on the target (as it does on Linux), Ansible constructs PowerShell commands on the control node, sends them over WinRM, and executes them natively on the Windows host. *Bina kisi extra software ke WinRM ke through PowerShell commands bheje jate hain.*

> [!question] Q2: What is the purpose of `ansible_connection=winrm` in the inventory?
> **Answer:** By default, Ansible uses SSH to connect to managed hosts. Setting `ansible_connection=winrm` tells Ansible to override the connection type and use the Windows Remote Management protocol to talk to target Windows servers. *Ye define karta hai ki Windows target se WinRM use karke connect hona hai SSH nahi.*

> [!question] Q3: How do you secure admin passwords inside an inventory host file in Ansible?
> **Answer:** You should never keep plaintext passwords in `hosts` files. You can encrypt them using **Ansible Vault** (`ansible-vault encrypt credentials.yml`) or integrate Ansible with secret storage solutions like HashiCorp Vault or Azure Key Vault. *Ansible vault ka use karte hain security ke liye taki plaintext me details na rahein.*

> [!question] Q4: Which Ansible module is used to enable Windows Server roles or features?
> **Answer:** The `ansible.windows.win_feature` module is used to install or uninstall roles and features (e.g., Web-Server, Active Directory domain services). *Role ya feature install karne ke liye win_feature use hota hai.*

==**Exam Tip:** Know the difference between `win_feature` (Windows features) and `apt`/`yum` (Linux packages).==

> [!question] Q5: Why might an Ansible playbook execution against a Windows machine freeze during task execution?
> **Answer:** Playbooks usually freeze if a command or installer running on the Windows machine prompts for user interaction (like a UAC dialog or a GUI popup) that cannot be handled invisibly. To resolve this, ensure tasks run completely unattended or use `become: yes` with `become_method: runas`. *Agar installer ko GUI ya OK button ka wait ho toh Ansible hang ho jayega.*

---
## 🔗 Related Notes

- [[03-Identity-and-Core-Services/05-Windows-Server/IIS]] — Core Web Server details
- [[05-Automation-and-Ticketing/12-Ansible/ANS-02 Ansible for Linux Admins]] — Ansible for Linux target nodes
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Automation-and-Scripting]] — Underlying scripting language for Windows
