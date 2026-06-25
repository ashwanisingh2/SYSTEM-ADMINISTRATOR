---
tags: [automation, ansible, windows, sysadmin]
aliases: [ansible-windows-admins]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# ANS-01 Ansible for Windows Admins

> [!abstract] Overview
> Ansible is an open-source, agentless IT automation engine. Although the Ansible control node must run on Linux, it can manage Windows client and server operating systems using Windows Remote Management (WinRM) or SSH. This note details setting up Ansible for Windows, directory structures, inventories, and common Windows modules.

---

## Concept Overview
- **What it is** — Ansible is a configuration management and orchestration tool that uses declarative YAML files (Playbooks) to define the desired state of target machines.
- **Why it matters for a support engineer** — Configuring 50 Windows servers manually is slow and error-prone. Ansible enables you to deploy features, create users, and edit registries uniformly across hundreds of machines in minutes.
- **Where you encounter this in real job** — Automating the installation of IIS web servers, setting registry policies for security compliance, or automating weekly service restarts across cluster servers.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Executes pre-designed playbooks using `ansible-playbook`, reads command output logs, and checks target connection status.
  - **Escalation Trigger:** Escalate to L2 if playbook execution fails due to WinRM authentication failures, syntax errors, or registry key permission issues.
  - ****L2 Resolution:**** Configures WinRM listeners on Windows targets, sets up inventory hosts files, writes basic playbooks using Windows modules, and manages variables.
  - ****L3 Resolution:**** Integrates Ansible with Active Directory for domain-based execution, constructs modular Ansible Roles, and manages credential secrets using Ansible Vault.

*Seedha simple mein: Ansible ek control server (Linux) se saare Windows servers ko automate karne ka tool hai. Isme hume har server pe koi agent install nahi karna padta. Hum ek simple text file (Playbook) me tasks likhte hain aur woh saare target Windows servers par ek sath execute ho jate hain.*

---

## Technical Deep Dive

### 1. Agentless Architecture & WinRM Setup
Unlike other tools, Ansible is **agentless**. It connects to Windows servers using **WinRM (Windows Remote Management)**.
* WinRM operates over HTTP (Port 5985) or HTTPS (Port 5986). HTTPS is highly recommended for security.
* Target Windows machines require a PowerShell configuration script (`Configure-SMWinRM.ps1`) to enable listeners, configure firewall rules, and allow basic/certificate-based authentication.

### 2. Inventory Files for Windows
The inventory file (usually named `hosts`) lists the target servers. To manage Windows nodes, specific connection variables must be defined:
```ini
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
Ansible uses target modules to perform tasks on Windows:
* `ansible.windows.win_service`: Manages Windows Services (starts, stops, restarts).
* `ansible.windows.win_user`: Manages local user accounts.
* `ansible.windows.win_feature`: Installs or uninstalls Windows Roles/Features.
* `ansible.windows.win_copy`: Copies files from the control node to target Windows paths.
* `ansible.windows.win_regedit`: Adds, edits, or deletes Windows Registry keys.

---

## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - A Linux Control Node (Ubuntu/RHEL) with Ansible installed.
> - A target Windows Server 2022 instance reachable over network.
> - Admin credentials on the target Windows Server.

### Step 1: Configure WinRM on target Windows Server
We must prepare the Windows host to accept connections from our Linux control node.

1. Log in to the target Windows Server.
2. Open PowerShell as Administrator and run the WinRM setup commands:
```powershell
# Enable WinRM service and configure listeners
winrm quickconfig -q

# Set basic authentication and allow unencrypted transport (for HTTP testing, use HTTPS in prod)
Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true
Set-Item WSMan:\localhost\Service\AllowUnencrypted -Value $true
```
**Expected Output:** WinRM starts listening on Port 5985.

---

### Step 2: Configure Inventory on Linux Control Node
We will define our target Windows group and connection parameters.

1. On the Linux control node, create or edit the inventory file:
```bash
sudo vi /etc/ansible/hosts
```
2. Insert the target server details:
```ini
[win_servers]
192.168.1.105

[win_servers:vars]
ansible_user=Administrator
ansible_password=MySecretPassword123
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```
**Expected Output:** Run connection ping to verify host connectivity:
```bash
ansible win_servers -m ansible.windows.win_ping
```
Target returns: `"ping": "pong"`.

---

### Step 3: Write a Playbook to Install Web Server (IIS) and Configure Service
We will write a playbook to install IIS and ensure the World Wide Web Publishing service is running.

1. Create a playbook file named `iis_install.yml`:
```bash
vi iis_install.yml
```
2. Write the playbook structure in YAML:
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
**Expected Output:** Playbook file is successfully saved.

---

### Step 4: Execute Playbook and Verify
We will run the playbook and verify target web server status.

1. Execute the playbook:
```bash
ansible-playbook iis_install.yml
```
**Expected Output:** Playbook logs display execution runs showing `changed=3` for the host.
2. Test by browsing to target Windows IP `http://192.168.1.105` to view the page.

---

## Common Commands / Cheat Sheet

| Command | Description | Example |
|---------|-------------|---------|
| `ansible [group] -m win_ping` | Ping Windows targets using WinRM | `ansible win_servers -m win_ping` |
| `ansible-playbook [playbook.yml]` | Run an Ansible Playbook | `ansible-playbook install_iis.yml` |
| `ansible-vault encrypt [file]` | Encrypt credential files | `ansible-vault encrypt hosts` |
| `ansible [group] -m win_service -a "name=spooler state=restarted"` | Restart a service (Ad-hoc) | `ansible win_servers -m win_service -a "name=spooler state=restarted"` |

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **`win_ping` fails with connection refused** | WinRM service is not running on target or blocked by firewall. | Run `Get-Service winrm` on Windows. Allow port `5985/5986` in Windows Defender Firewall. |
| **Authentication fails with 401 Unauthorized** | The user credentials are correct but basic authentication is disabled. | Run `Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true` on target Windows system. |
| **Playbook fails due to SSL errors** | Self-signed certificate is not trusted by control node. | Set `ansible_winrm_server_cert_validation=ignore` in inventory variables to bypass verification. |

---

## Interview Questions

**Q1: How does Ansible manage Windows servers without installing an agent?**
> A: Ansible uses **WinRM (Windows Remote Management)** (over HTTP port 5985 or HTTPS port 5986) or OpenSSH. Instead of running python code directly on the target (as it does on Linux), Ansible constructs PowerShell commands on the control node, sends them over WinRM, and executes them natively on the Windows host.

**Q2: What is the purpose of `ansible_connection=winrm` in the inventory?**
> A: By default, Ansible uses SSH to connect to managed hosts. Setting `ansible_connection=winrm` tells Ansible to override the connection type and use the Windows Remote Management protocol to talk to target Windows servers.

**Q3: How do you secure admin passwords inside an inventory host file in Ansible?**
> A: You should never keep plaintext passwords in hosts files. You can encrypt them using **Ansible Vault** (`ansible-vault encrypt credentials.yml`) or integrate Ansible with secret storage solutions like HashiCorp Vault or Azure Key Vault.

**Q4: Which Ansible module is used to enable Windows Server roles or features?**
> A: The `ansible.windows.win_feature` module is used to install or uninstall roles and features (e.g., Web-Server, Active Directory domain services).

---

## Related Notes
- [[03-Identity-and-Core-Services/05-Windows-Server/IIS]]
- [[05-Automation-and-Ticketing/12-Ansible/ANS-02 Ansible for Linux Admins]]
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PowerShell-Automation-and-Scripting]]
