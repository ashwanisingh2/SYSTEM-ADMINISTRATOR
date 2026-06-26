---
tags: [ansible, playbook, roles, automation, devops]
aliases: [ansible-playbooks, ansible-roles]
created: 2026-06-26
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

> [!NOTE|color-red]
> 🤖 **ANSIBLE**

`#complete` `#intermediate` `#none`

# ANS-03: Ansible Playbooks and Roles

> [!abstract] Overview
> Ansible Playbooks are YAML files that contain a series of tasks to be executed on remote hosts, while Roles provide a way to automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. Ek support engineer ke liye playbooks aur roles likhna aur troubleshoot karna aana chahiye kyunki enterprise environment mein sab kuch code (Infrastructure as Code) ke through manage hota hai.

---
## 🧠 Concept Overview

- **What it is** — Playbooks configuration management aur deployment ka blueprint hain. Roles un playbooks ko modular, reusable, aur organized banate hain.
- **Why it matters** — Real job mein manual configuration nahi hoti. Ek playbook se hum hazaron servers ek saath configure kar sakte hain. Roles se code duplication bachta hai.
- **Where you see this** — Web server deployments, OS patching scripts, user management automation, aur complex application stacks ki deployment mein.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — Playbooks execute karna, errors dekhna aur basic syntactical checks (`ansible-playbook --syntax-check`) karna. |
| **L2** | Configure, fix, escalate kab karta hai — Playbooks likhna, variables use karna, failed tasks ko troubleshoot karna, roles import karna. |
| **L3** | Architecture, design, enterprise-level — Complex custom roles design karna, Ansible Galaxy integration, CI/CD pipelines mein Ansible ko embed karna. |

> [!tip] Seedha Simple Mein
> *Playbook ek recipe ki tarah hai jismein step-by-step likha hota hai kya karna hai. Aur Role ek pre-packaged ingredient box hai jismein specific dish banane ka saara saman aur choti recipes pehle se organized hoti hain.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Ansible Playbook and Role** is like **Building a House** because...
>
> - **Playbook** is the master blueprint that says "I need a foundation, walls, plumbing, and electricity".
> - **Roles** are the specialized contractors (Plumber, Electrician). The electrician role only knows about wires and switches, the plumber role only knows about pipes. You just call them in the master blueprint when needed.

---
## 🔬 Technical Deep Dive

### 1. The Anatomy of a Playbook

> [!info] Key Concept
> A Playbook is composed of one or more "plays". Each play maps a group of hosts (from the inventory) to a set of tasks (roles or modules).

Playbooks are written in YAML. YAML relies heavily on indentation (using spaces, NOT tabs).

*YAML syntax Ansible ke liye bahut strict hoti hai. Ek extra space ya tab poora playbook break kar sakta hai.*

**Key Components:**
- `hosts`: Kahan execute karna hai (e.g., webservers, dbservers).
- `become`: Sudo privileges (Root access) chahiye ya nahi.
- `vars`: Variables jo playbook mein use honge.
- `tasks`: Actual actions jo perform karne hain (using modules like `yum`, `apt`, `service`, `copy`).
- `handlers`: Tasks jo tabhi chalte hain jab koi doosra task change trigger kare (e.g., restart service after config change).

> [!danger] Common Mistake
> YAML mein tab `\t` use karna sabse common galti hai. *Hamesha spacebar use karein indentation ke liye, warna Ansible syntax error dega.*

### 2. Deep Dive into Roles

> [!info] Key Concept
> Roles are a framework for building fully independent, or interdependent collections of variables, tasks, files, templates, and modules.

Role Structure:
```
my_role/
├── tasks/       # Main list of tasks to be executed by the role.
├── handlers/    # Handlers, which may be used by this role or even anywhere outside this role.
├── defaults/    # Default variables for the role.
├── vars/        # Other variables for the role (higher priority than defaults).
├── files/       # Files which can be deployed via this role.
├── templates/   # Templates which can be deployed via this role.
├── meta/        # Metadata for the role, including role dependencies.
```

*Role ka fayda yeh hai ki aapko bada playbook nahi likhna padta. Aap `ansible-galaxy init role_name` command chala ke folder structure bana lete ho, aur bas specific files mein apna code daalte ho.*

### 3. Variables and Conditionals

Playbooks mein hum `when` statements (conditionals) aur loops (`loop` or `with_items`) use karte hain.
*Jaise agar OS RedHat hai toh `yum` chalao, aur agar Ubuntu hai toh `apt` chalao. Yeh `when` se hota hai.*

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - Ansible installed on the control node.
> - SSH key-based authentication configured to managed nodes.
> - A working inventory file.

### Step 1: Create a Basic Playbook for Apache

```yaml
# install_apache.yml
---
- name: Install and start Apache HTTPD
  hosts: webservers
  become: yes
  vars:
    http_port: 80
  tasks:
    - name: Install httpd package
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Ensure httpd is running and enabled
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes
```

```bash
# Yeh command syntax verify karta hai
ansible-playbook --syntax-check install_apache.yml

# Yeh command playbook execute karta hai
ansible-playbook -i inventory install_apache.yml
```

> [!success] Expected Output
> ```
> PLAY [Install and start Apache HTTPD] ****************************************
> 
> TASK [Gathering Facts] *******************************************************
> ok: [server1]
> 
> TASK [Install httpd package] *************************************************
> changed: [server1]
> 
> TASK [Ensure httpd is running and enabled] ***********************************
> changed: [server1]
> 
> PLAY RECAP *******************************************************************
> server1 : ok=3  changed=2  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
> ```

### Step 2: Convert the Playbook into a Role

```bash
# Create role directory structure
mkdir roles
cd roles
ansible-galaxy init apache_role
```

Now, move the task into `roles/apache_role/tasks/main.yml`:
```yaml
# roles/apache_role/tasks/main.yml
---
- name: Install httpd package
  ansible.builtin.yum:
    name: httpd
    state: present

- name: Ensure httpd is running and enabled
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: yes
```

Create a new playbook to use the role:
```yaml
# site.yml
---
- name: Configure Webservers
  hosts: webservers
  become: yes
  roles:
    - apache_role
```

Execute:
```bash
ansible-playbook -i inventory site.yml
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `ansible-playbook file.yml` | Playbook run karta hai | `ansible-playbook web.yml` |
| `ansible-playbook --syntax-check` | YAML syntax verify karta hai run karne se pehle | `ansible-playbook --syntax-check db.yml` |
| `ansible-playbook -C` | Dry-run (Check mode), dikhata hai kya change hoga bina actual change kiye | `ansible-playbook -C patch.yml` |
| `ansible-playbook --step` | Interactive mode, har task par puchega execute karna hai ya nahi | `ansible-playbook --step web.yml` |
| `ansible-playbook --tags "tag"` | Sirf specific tag wale tasks run karta hai | `ansible-playbook main.yml --tags "install"` |
| `ansible-galaxy init role_name` | Naya role folder structure banata hai | `ansible-galaxy init nginx_role` |
| `ansible-galaxy install user.role` | Ansible Galaxy (hub) se community role download karta hai | `ansible-galaxy install geerlingguy.mysql` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Syntax Error: We could not see the dict value** | YAML indentation galat hai (spaces/tabs issue). | YAML file mein space check karein. *Yamllint use karein format theek karne ke liye.* |
| **"Failed to connect to the host via ssh"** | SSH keys configure nahi hain, ya target server down hai. | `ssh user@host` manually test karein. SSH keys `ssh-copy-id` se copy karein. |
| **"Missing sudo password"** | Playbook mein `become: yes` hai but password nahi diya. | Command run karte time `-K` (or `--ask-become-pass`) switch add karein. |
| **Module not found** | Galat module name use kiya hai ya spelling mistake. | `ansible-doc -l` se correct module name search karein. (e.g., `yum` instead of `yum_install`). |
| **Variable is undefined** | Variable call kiya `{{ var }}` par define nahi kiya. | `vars:` section, inventory, ya `--extra-vars` mein variable pass karein. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Bulk OS Patching Failing on Few Servers

> [!example] Ticket
> "Monthly patching playbook failed on 5 out of 100 Linux servers."

**L1 Response:** Check the playbook output logs. Find the exact task where it failed. *Logs mein dekhna hota hai ki exactly kis task pe fail hua (e.g., disk space full ya package conflict).*
**Escalation Trigger:** Agar repo issue nahi hai aur disk space theek hai, but dependency conflict aa raha hai.
**L2 Resolution:** Connect directly to the failed servers. Run `yum update` manually to see the exact conflict. Resolve the conflict (e.g., remove conflicting package) and re-run the playbook specifically for those 5 servers using `ansible-playbook patch.yml --limit failed_servers`.

### 🎫 Scenario 2: Apache Config Syntax Error Breaking Production

> [!example] Ticket
> "After Ansible deployment, Apache is down on all web servers."

**L1 Response:** Check Apache status on one of the servers. Run `systemctl status httpd` or `apachectl configtest`. *Pata chalega ki config file mein typo tha jis wajah se service start nahi ho rahi.*
**Escalation Trigger:** Pata chala ki playbook ne galat template push kar diya hai.
**L2/L3 Resolution:** Fix the Jinja2 template (`.j2` file) in the Ansible role. Re-run the playbook. To prevent this in future, add a pre-check task in the handler:
```yaml
handlers:
  - name: restart apache
    ansible.builtin.service:
      name: httpd
      state: restarted
    listen: "restart apache"
```
And add a config test task before handler is called.

### 🎫 Scenario 3: Secret Credentials Exposed in Playbook

> [!example] Ticket
> "Security scan found hardcoded database passwords in Ansible playbooks repository."

**L1 Response:** Identify the exact file where the password is in plaintext. *Security breach! Plain text passwords code mein nahi hone chahiye.*
**Escalation Trigger:** Passwords need to be encrypted and code needs to be updated.
**L2 Resolution:** Use **Ansible Vault** to encrypt the sensitive data.
`ansible-vault encrypt vars/secrets.yml`.
Update playbook to use the vault file and run using `ansible-playbook site.yml --ask-vault-pass`. Change the database password immediately.

---
## 🎤 Interview Questions

> [!question] Q1: What is the difference between a Playbook and a Role in Ansible?
> **Answer:** A Playbook is a standalone YAML file containing a list of tasks. A Role is a directory structure that breaks down a complex playbook into independent, reusable components (tasks, handlers, vars, templates).

==**Exam Tip:** Roles allow sharing and code reuse (e.g., Ansible Galaxy).==

> [!question] Q2: How do you perform a "Dry Run" of an Ansible playbook?
> **Answer:** By using the `-C` or `--check` flag. It simulates the execution and shows what would be changed without actually making any modifications to the target systems.

> [!question] Q3: What is the purpose of Handlers in Ansible?
> **Answer:** Handlers are special tasks that only run when notified by another task. They are typically used to restart a service or reload a configuration only if a configuration file has actually changed.

> [!question] Q4: How do you handle sensitive data like passwords in Ansible playbooks?
> **Answer:** Using **Ansible Vault**. It encrypts files or specific variables so they can be safely stored in version control systems like Git. You use `ansible-vault create/edit/encrypt` commands.

> [!question] Q5: What happens if an Ansible task fails in the middle of a playbook?
> **Answer:** By default, Ansible stops executing the playbook on the host where the task failed, but continues on other hosts. You can change this behavior using `ignore_errors: yes` or by using `block/rescue/always` for error handling.

---
## 🔗 Related Notes

- [[ANS-01 Introduction to Ansible|Ansible Basics]] — 🤖 What is Ansible
- [[ANS-02 Ansible Inventory and Modules|Inventory and Modules]] — 📁 How to define hosts
- [[LNX-15 Linux Service Management|Systemctl & Services]] — 🐧 Managing services in Linux
