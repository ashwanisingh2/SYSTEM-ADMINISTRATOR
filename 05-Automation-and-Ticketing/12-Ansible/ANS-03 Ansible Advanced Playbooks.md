---
tags: [automation, ansible, playbooks, devops, rhcsa]
aliases: [ansible-advanced]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

# ANS-03: Ansible Advanced Playbooks

**Verification:**
- [ ] Check playbook syntax and execute dry-run check mode audits
- [ ] Write a dynamic playbook using OS-specific conditionals and loops
- [ ] Implement event-driven service restarts using handlers and notify blocks
- [ ] Build robust error-handling playbooks using block-rescue-always structures
- [ ] Generate customized configuration files dynamically using Jinja2 templates

> [!abstract] Overview
> This note covers advanced Ansible playbook development patterns. It details variables, loops, conditionals, handlers, block-level exception handling, Jinja2 template compiling, dry-run testing modes, and error-handling flow controls.

---
## Concept Overview
- **What it is** — Advanced playbooks expand basic static Ansible tasks into a dynamic orchestration engine capable of loops, conditionals, template rendering, event notification (handlers), and error handling/rollbacks.
- **Why it matters for a support engineer** — Simple list-of-tasks playbooks are rigid and fail when managing diverse hybrid networks. Advanced playbooks allow your automation to adapt dynamically based on server facts, handle failures gracefully, and execute operations conditionally.
- **Where you encounter this in real job** — Deploying configuration updates across mixed OS environments (CentOS/Ubuntu), restarting services only when config files are updated, grouping tasks for fail-safe rollbacks, and compiling customized server parameters on the fly.
- **L1 vs L2 vs L3 responsibility split for this topic:**
  - **L1 Resolution**: Verify playbook syntax using `--syntax-check`, run playbooks in dry-run mode (`--check --diff`), and extract failed task descriptions from standard execution logs.
  - **Escalation Trigger**: Escalate to L2 if playbooks fail due to template rendering syntax, missing variable assignments, or looping structure logic errors.
  - **L2 Resolution**: Author playbooks utilizing variables, loops, facts, conditionals (`when`), event-driven notifications (`handlers`), and customized Jinja2 templates.
  - **L3 Resolution**: Implement block-rescue-always structures for automated error recovery, build custom filters, write inventory plugins, and optimize playbook execution speeds (pipelining/mitogen).

*Seedha simple mein: Advanced Playbooks humare automation ko smart banati hain. Iska use karke hum decision le sakte hain (jaise agar OS RHEL hai toh apache install karo, agar Ubuntu hai toh apache2), dynamic files compile kar sakte hain (Jinja2 templates ke through), aur agar playbook mein koi task fail hota hai toh backup options run kar sakte hain (block/rescue).*

---
## Real-World Analogy
Think of a **smart automated manufacturing factory**:
- **Variables**: The client specifications (car color, engine size) fed to the assembly line.
- **Conditionals (`when`)**: Sensors on the conveyor belt. If the sensor detects a sedan, it applies Red paint. If it detects a truck, it applies Blue paint.
- **Loops (`loop`)**: Feeding multiple wheels to the same robotic arm sequentially.
- **Handlers (`notify`)**: Quality control checks. The arm only recalibrates (restarts) the machine if the paint nozzles are physically replaced, not on every cycle.
- **Blocks & Rescue (`block` / `rescue`)**: An emergency protocol. If the robot drops a glass window (task fails), it immediately sweeps up the glass (rescue) and moves to the next car, preventing the whole factory line from stopping.

---
## Technical Deep Dive

### 1. Variables, Facts, and Registered State
- **Facts (`ansible_facts`)**: System metadata gathered automatically by Ansible during the setup phase (e.g., memory sizes, hostnames, IP configurations, OS types).
- **Registering variables (`register`)**: Captures the output of a task and stores it in a custom variable. You can then query this variable in subsequent tasks.

```yaml
- name: Check web service status
  ansible.builtin.command: systemctl is-active httpd
  register: web_status
  ignore_errors: yes

- name: Print status if failed
  ansible.builtin.debug:
    msg: "Apache is down!"
  when: web_status.rc != 0
```

### 2. Conditionals (`when`) and Loops (`loop`)
- **Loops**: Replace the legacy `with_items` syntax. They permit executing a task multiple times with varying arguments using the `item` variable.
- **Conditionals**: The `when` statement evaluates boolean expressions, executing the task only if the expression resolves to true.

```yaml
- name: Install packages dynamically
  ansible.builtin.dnf:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - vim
    - curl
  when: ansible_facts['os_family'] == "RedHat"
```

### 3. Handlers and Event Notifications
- **Handlers**: Special tasks that only run when triggered by a standard task using the `notify` keyword.
- **Event-Driven**: They only execute if the calling task actually made a change on the target host (e.g., updated a configuration file). If the file was already correct (no change), the handler is skipped. Handlers run at the very end of the play.

### 4. Error Handling: Block, Rescue, and Always
Blocks group multiple tasks logically. They allow for structural exception handling similar to try-catch blocks in programming:
- **`block`**: The main group of tasks you attempt to execute.
- **`rescue`**: Tasks that run only if any task inside the `block` fails. Used for rollbacks or log collection.
- **`always`**: Tasks that execute under all circumstances, regardless of whether the block succeeded or failed (e.g., closing connections, deleting temp files).

---
## Step-by-Step Lab / Configuration

> [!warning] Pre-requisites
> - An Ansible Control Node with Ansible installed.
> - Access to a target server (`192.168.10.30` - Rocky Linux).
> - SSH passwordless credentials verified.
> - Sudo rights configured.

### Step 1: Create a Jinja2 Configuration Template
We will create a dynamic configuration template for a custom application.

1. On the control node, create a template file `~/httpd_custom.conf.j2`:
```nginx
# Configured by Ansible on {{ ansible_facts['fqdn'] }}
Listen {{ custom_port | default('8080') }}
MaxClients {{ max_clients | default(150) }}
DocumentRoot /var/www/html
```

---

### Step 2: Write the Advanced Playbook
We will author a playbook that uses variables, templates, handlers, loops, and error-handling blocks.

1. Save the following playbook as `~/advanced_web.yml`:
```yaml
---
- name: Deploy Advanced Web Configuration
  hosts: all
  become: yes
  vars:
    custom_port: 8081
    max_clients: 250
    web_packages:
      - httpd
      - mod_ssl

  tasks:
    - name: Try Web Infrastructure Deployment Block
      block:
        - name: Install required web packages
          ansible.builtin.dnf:
            name: "{{ item }}"
            state: present
          loop: "{{ web_packages }}"

        - name: Deploy custom httpd config from template
          ansible.builtin.template:
            src: ~/httpd_custom.conf.j2
            dest: /etc/httpd/conf.d/custom.conf
            owner: root
            group: root
            mode: '0644'
          notify: Restart Apache Service

        - name: Ensure Apache is enabled and running
          ansible.builtin.service:
            name: httpd
            state: started
            enabled: yes

      rescue:
        - name: Log error failure status
          ansible.builtin.debug:
            msg: "Playbook encountered a failure! Reverting configurations."

        - name: Delete invalid custom config file
          ansible.builtin.file:
            path: /etc/httpd/conf.d/custom.conf
            state: absent

      always:
        - name: Clean temporary local cache
          ansible.builtin.file:
            path: /tmp/ansible_run.tmp
            state: absent

  handlers:
    - name: Restart Apache Service
      ansible.builtin.service:
        name: httpd
        state: restarted
```

---

### Step 3: Run Playbook in Syntax and Check Mode
1. Run syntax verification:
```bash
ansible-playbook ~/advanced_web.yml --syntax-check
```
**Expected Output:** `playbook: /home/sysadmin/advanced_web.yml`

2. Execute dry-run checking to inspect planned changes and template differences:
```bash
ansible-playbook ~/advanced_web.yml --check --diff
```
**Expected Output:** Displays green/yellow outputs and a diff block showing the lines of code compiled by the Jinja2 template, without applying actual changes on target.

### Step 4: Execute the Playbook
1. Run the playbook to apply changes:
```bash
ansible-playbook ~/advanced_web.yml
```
**Expected Output:** The httpd packages are installed, the template is compiled, and the handler restarts Apache (`changed=3`).

2. Run it a second time:
```bash
ansible-playbook ~/advanced_web.yml
```
**Expected Output:** Shows `changed=0`. The configuration matched the template perfectly, and the handler task is skipped.

---
## Cheat Sheet

| Command / Setting | Purpose | Example |
|---|---|---|
| `ansible-playbook --syntax-check` | Validates playbook syntax without running tasks | `ansible-playbook site.yml --syntax-check` |
| `ansible-playbook -C --diff` | Runs in dry-run mode showing file modification diffs | `ansible-playbook site.yml -C --diff` |
| `ansible-playbook --start-at-task="name"` | Begins playbook execution at a specific task name | `ansible-playbook site.yml --start-at-task="Start Apache"` |
| `ansible-playbook -t tag_name` | Filters execution to run only tasks matching a tag | `ansible-playbook site.yml -t config` |
| `ansible-playbook --skip-tags tag_name` | Skips tasks matching specified tags | `ansible-playbook site.yml --skip-tags debug` |
| `ansible-vault view secret.yml` | Displays encrypted file content without decrypting | `ansible-vault view secret.yml` |
| `failed_when: result.rc > 1` | Overrides default task failure conditions | Append under target task |
| `changed_when: false` | Suppresses task status from showing "changed" | Append under target task |

---
## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Handler does not trigger, even though it was defined in the playbook. | The task notifying the handler did not register a status change (`changed=0`). | Handlers only trigger on state changes. If testing, modify the source file or configuration to force a change. |
| Playbook fails: "AnsibleUndefinedVariable: 'var_name' is undefined". | The variable was not defined in `vars`, host files, or the command line. | Define the variable in the playbook header, host vars directory, or pass it via CLI: `--extra-vars "var_name=value"`. |
| Playbook crashes with Jinja2 parsing errors inside template compilation. | Braces `{{ }}` syntax mismatch, or incorrect spacing in the template file. | Ensure all template variable insertions are surrounded by double curly braces and technical syntax is valid. |
| Block execution fails, but the tasks inside the `rescue` section are not executed. | The failure occurred in a task situated outside the scope of the `block` block. | Move all monitored tasks inside the `block:` section. Ensure `rescue:` is aligned at the same level. |
| Tasks inside `always:` fail, preventing execution reporting. | An error occurred in the always task, masking the original block error. | Keep always tasks simple (like deleting files or printing messages). Avoid complex scripts inside always. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the purpose of the `--check` and `--diff` flags when executing an Ansible playbook?
> **A:** The `--check` flag runs the playbook in dry-run mode. It queries the target nodes and reports what changes *would* be made, without actually modifying anything on the hosts. The `--diff` flag, when used alongside `--check` or during run, displays the exact text line changes that will be made inside configuration files, helping admins audit templates before applying them.

> [!question] L2 Question
> **Q:** Explain how Handlers work in Ansible. When do they execute, and what determines if they run?
> **A:** Handlers are special tasks defined in a separate section of the playbook that are only executed when notified by another task using the `notify` keyword. They only run if the notifying task reports a status of `changed` (meaning a modification actually took place). If the state is already compliant (`ok`), the handler is skipped. Additionally, all notified handlers are queued and executed once at the very end of the play, preventing services from restarting multiple times during execution.

> [!question] L3/Scenario Question
> **Q:** You need to write a playbook that deletes a temporary configuration folder. If the deletion fails, the playbook should log the error and send a notification, but it must always clean up a lock file in `/tmp/lock` regardless of success or failure. How would you design this?
> **A:** 
> - **Situation:** Automated deletion of a directory with mandatory post-cleanup of a lock file.
> - **Task:** Design a robust playbook structure to handle task failure and cleanup.
> - **Action:** I will implement a `block`, `rescue`, and `always` structure:
>   - Inside the **`block`** section, I will define the task to delete the temporary configuration folder.
>   - Inside the **`rescue`** section, I will define a task to output a warning log using the `debug` module and trigger an email/Slack notification if the deletion task fails.
>   - Inside the **`always`** section, I will configure the file module to remove `/tmp/lock` with `state: absent`.
> - **Result:** If the folder deletion fails, the rescue tasks run, followed by the always block. If it succeeds, the rescue tasks are skipped, and the always block still runs, guaranteeing lock cleanup in both states.

---
## Related Notes
- [[05-Automation-and-Ticketing/12-Ansible/ANS-02 Ansible for Linux Admins|ANS-02 Ansible for Linux Admins]] — Configures basic SSH connectivity and inventory.
- [[02-Operating-Systems/04-Linux-RHEL/L-08 Services and Systemd|L-08 Services and Systemd]] — Background services managed by playbooks and handlers.
- [[05-Automation-and-Ticketing/10-Scripting-PowerShell/PS-04 PowerShell Scripting|PS-04 PowerShell Scripting]] — Contrast with PowerShell imperative scripting.

---
*Tags: #automation #ansible #playbooks #devops #rhcsa*
