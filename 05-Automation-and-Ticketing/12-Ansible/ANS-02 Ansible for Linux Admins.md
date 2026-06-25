---
tags: [automation, ansible, linux, sysadmin, rhcsa]
aliases: [ansible-linux-admins]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

# ANS-02 Ansible for Linux Admins

> [!abstract] Overview
> Ansible is a native automation framework for Linux systems. It establishes connections over standard SSH and executes tasks by deploying Python scripts to remote target endpoints. This note details Linux-based Ansible environments, SSH configurations, common management modules, Roles, and secret management using Ansible Vault.

---

## Concept Overview
- **What it is** — Ansible automation for Linux manages infrastructure state agentlessly by utilizing target-host SSH capabilities and Python engines.
- **Why it matters for a support engineer** — Maintaining configuration drift on 100+ RHEL servers manually is impossible. Ansible ensures all servers maintain identical configurations, security policies, and package updates.
- **Where you encounter this in real job** — Patching system kernels via DNF, automating local user account creation, deploying configurations via templates, and scheduling cron jobs.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - ****L1 Resolution:**** Checks target host reachability using `ansible -m ping`, reviews playbook results, and restarts failed system services.
  - **Escalation Trigger:** Escalate to L2 if playbooks fail due to SSH permission denied issues, missing python dependencies, or playbook syntax failures.
  - ****L2 Resolution:**** Configures SSH key-based access for Ansible, writes task playbooks, sets up cron schedules via Ansible, and encrypts secrets with `ansible-vault`.
  - ****L3 Resolution:**** Writes custom ansible modules, architects highly reusable Ansible Roles, configures dynamic inventory for cloud VMs, and schedules multi-stage rollouts.

*Seedha simple mein: Linux systems me Ansible SSH key-based connectivity ke through connect karta hai. Target nodes par python installed hona chahiye. Hum control node se commands run karke package installation (yum/dnf), files copying, user accounts aur cron jobs automate kar sakte hain.*

---

## Technical Deep Dive

### 1. SSH-Based Connection Configuration
Ansible connects to remote Linux servers using standard OpenSSH.
* To achieve fully automated, passwordless execution, the controller's public SSH key must be distributed to all target servers' `authorized_keys` file.
* Variables in the inventory configuration (`/etc/ansible/hosts`):
```ini
[rhel_servers]
server1.company.local ansible_host=192.168.10.11
server2.company.local ansible_host=192.168.10.12

[rhel_servers:vars]
ansible_user=sysadmin
ansible_ssh_private_key_file=/home/sysadmin/.ssh/id_rsa
```

### 2. Common Ansible Modules for Linux
* `ansible.builtin.yum` / `ansible.builtin.dnf`: Manages packages (installs, updates, removes) on RPM systems.
* `ansible.builtin.copy`: Copies static files from the control node to target Linux hosts.
* `ansible.builtin.template`: Uses the Jinja2 template engine to generate files dynamically on targets using variables.
* `ansible.builtin.file`: Sets directory/file properties (ownership, group, permissions, symlinks).
* `ansible.builtin.service` / `ansible.builtin.systemd`: Controls background services.
* `ansible.builtin.user`: Provision local user accounts and groups.
* `ansible.builtin.cron`: Manages user and system cron entries.

### 3. Ansible Vault
Ansible Vault allows encrypting files (such as host variables containing passwords, SSH keys, or API tokens) so they can be safely stored in version control systems like Git.
* Encrypting a file: `ansible-vault encrypt credentials.yml`
* Running a playbook with encrypted files: `ansible-playbook --ask-vault-pass site.yml`

---

## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - An Ansible Control Node (Linux).
> - Two target RHEL servers (`192.168.1.51` and `192.168.1.52`).
> - SSH passwordless login configured from controller to targets.
> - Sudo privileges on target servers.

### Step 1: Distribute SSH Keys from Controller
We will generate an SSH keypair on our controller and copy it to our target nodes.

1. Generate SSH keys:
```bash
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
```
2. Copy the key to target hosts:
```bash
ssh-copy-id sysadmin@192.168.1.51
ssh-copy-id sysadmin@192.168.1.52
```
**Expected Output:** Successful key verification. Testing `ssh sysadmin@192.168.1.51` logs in without prompting for a password.

---

### Step 2: Configure Ansible Inventory and Sudo Privilege Escalation
We will define our inventory group and verify we can escalate to root.

1. Edit the inventory file `/etc/ansible/hosts`:
```ini
[linux_hosts]
192.168.1.51
192.168.1.52

[linux_hosts:vars]
ansible_user=sysadmin
ansible_become=yes
ansible_become_method=sudo
```
2. Test connection and privilege escalation:
```bash
ansible linux_hosts -m ping
```
**Expected Output:** Target nodes respond with `"ping": "success"`.

---

### Step 3: Write a Playbook to Patch Packages, Add Users, and Schedule Cron
We will write a comprehensive playbook implementing package updates, user configuration, directory creation, and a cron job.

1. Create a playbook named `system_setup.yml`:
```yaml
---
- name: System Base Setup
  hosts: linux_hosts
  become: yes
  tasks:
    - name: Update all system packages to latest version
      ansible.builtin.dnf:
        name: "*"
        state: latest

    - name: Create local admin user account
      ansible.builtin.user:
        name: backupop
        shell: /bin/bash
        groups: wheel
        append: yes

    - name: Create backups directory
      ansible.builtin.file:
        path: /var/backups/system
        state: directory
        owner: backupop
        group: backupop
        mode: '0750'

    - name: Add daily cron job to run backup script
      ansible.builtin.cron:
        name: "Run backup script"
        minute: "0"
        hour: "2"
        job: "/usr/local/bin/backup.sh > /dev/null 2>&1"
```
**Expected Output:** Playbook file is successfully saved.

---

### Step 4: Run the Playbook
We will run the playbook and verify changes.

1. Run the playbook:
```bash
ansible-playbook system_setup.yml
```
**Expected Output:** Console outputs task statuses showing package installation execution and user/directories configurations as `changed`.

---

## Common Commands / Cheat Sheet

| Command | Description | Example |
|---------|-------------|---------|
| `ansible all -m ping` | Ping all hosts in inventory | `ansible all -m ping` |
| `ansible-vault create [file]` | Create an encrypted vault file | `ansible-vault create vars.yml` |
| `ansible-vault edit [file]` | Edit an encrypted vault file | `ansible-vault edit vars.yml` |
| `ansible-playbook playbook.yml --check` | Run a dry-run of a playbook | `ansible-playbook setup.yml --check` |

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **`Permission denied (publickey)` on ping** | SSH keys not copied to targets or incorrect user set. | Run `ssh-copy-id user@target`. Ensure `ansible_user` variable matches target username. |
| **`Missing sudo password` error** | Target requires a password to escalate to root. | Run playbook with `--ask-become-pass` flag: `ansible-playbook setup.yml -K` or configure passwordless sudo on targets. |
| **DNF package task fails with lock error** | Another process is holding the DNF package database lock. | Wait for ongoing system updates to complete, or kill the locked process using `kill -9 [pid]`. |

---

## Interview Questions

**Q1: What is the significance of the `become: yes` keyword in a playbook?**
> A: The `become: yes` keyword directs Ansible to use privilege escalation (usually `sudo`) on the target server. This allows Ansible to execute administrative tasks (like package installations or service restarts) that a regular unprivileged SSH user cannot perform.

**Q2: What is the difference between the `copy` and `template` modules in Ansible?**
> A: The `copy` module transfers a static file from the control node to the target machine without modifying its content. The `template` module processes a Jinja2 template file, resolves variable placeholders dynamically (e.g. system hostnames or IP addresses), and writes the modified result to the target host.

**Q3: How do you pass the password for an encrypted Ansible Vault file when running a playbook?**
> A: You can pass it interactively by adding the `--ask-vault-pass` flag to `ansible-playbook` command, or reference a plaintext password file path using the `--vault-password-file` argument.

**Q4: How do you restrict a playbook execution to run on only one specific server from a large inventory group?**
> A: You can use the `--limit` flag. For example: `ansible-playbook site.yml --limit "192.168.1.51"`.

---

## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-09 SSH Configuration and Security]]
- [[02-Operating-Systems/04-Linux-RHEL/L-10 Software Management — YUM DNF]]
- [[05-Automation-and-Ticketing/12-Ansible/ANS-01 Ansible for Windows Admins]]
