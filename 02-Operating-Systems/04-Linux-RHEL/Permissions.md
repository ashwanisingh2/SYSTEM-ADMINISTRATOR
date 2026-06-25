---
tags: [desktop-support, linux, permissions, security, L1]
aliases: [linux-permissions, chmod-guide, acl-guide]
created: 2026-06-25
status: #complete
difficulty: #beginner
cert-relevant: #rhcsa
---

# Linux File Permissions and Security

---

## Concept Overview
- **What it is**: Linux File Permissions govern access to files and directories by defining read (r), write (w), and execute (x) privileges for three ownership scopes: User/Owner (u), Group (g), and Others (o).
- **Why it matters for a support engineer**: Linux enforces strict permission security boundaries. Support engineers resolve "Permission Denied" errors, configure shared group directories, secure configuration files (like private SSH keys), and implement granular access control lists (ACLs).
- **Where you encounter this in real job**: Enabling script execution privileges, fixing web server permission blocks (Nginx/Apache), creating secure group folders, and auditing root-owned system files.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Resolves basic file access issues, explains owner/group concepts, and changes standard permissions (`chmod`, `chown`).
  - **L2**: Manages special permissions (SUID, SGID, Sticky Bit), configures user default permissions (`umask`), and sets up Access Control Lists (ACLs).
  - **L3**: Establishes global security permission policies, audits systems for unauthorized SUID binaries, designs encrypted file structures, and configures SELinux context overlays.

---

## Technical Deep Dive

### 1. Standard Permission Representation
Permissions are displayed as a string of 10 characters in the command `ls -l`:

```
d rwx r-x r--
|  |   |   |
|  |   |   +-- Others (Read-only)
|  |   +------ Group (Read & Execute)
|  +---------- User/Owner (Read, Write & Execute)
+------------- File Type (d: Directory, -: Regular File, l: Symbolic Link)
```

#### Octal vs. Symbolic Value Conversions:
Permissions can be defined using octal numeric weights:
- **Read (r)** = `4`
- **Write (w)** = `2`
- **Execute (x)** = `1`
- **No Permission (-)** = `0`

By adding these weights together, we define access levels for Owner, Group, and Others:
- **`755`** (`rwxr-xr-x`): Owner has full access (`4+2+1=7`); Group and Others have Read & Execute access (`4+1=5`).
- **`644`** (`rw-r----`): Owner has Read & Write access (`4+2=6`); Group and Others have Read-only access (`4`).
- **`700`** (`rwx------`): Only the Owner has access; Group and Others are blocked.

### 2. Special Permissions
Special permissions extend standard security boundaries:
1. **SUID (Set User ID - Weight `4`)**: 
   - *Behavior*: Applied to executable files. The program runs with the privileges of the file **Owner** (usually root) rather than the user executing it.
   - *Example*: `/usr/bin/passwd` has SUID enabled (`-rwsr-xr-x`). This allows standard users to run it to change their passwords, which updates the root-owned `/etc/shadow` file.
2. **SGID (Set Group ID - Weight `2`)**:
   - *Behavior*: Applied to directories. Any new file or subfolder created inside inherits the **Group ownership** of the parent directory automatically, rather than the primary group of the user who created it. Excellent for collaborative directories.
3. **Sticky Bit (Weight `1`)**:
   - *Behavior*: Applied to directories. Restricts file deletion; only the **Owner** of the file, the owner of the parent directory, or the root user can delete or rename files inside, regardless of write permissions.
   - *Example*: The `/tmp` directory has the Sticky Bit active (`drwxrwxrwt`).

### 3. Default Permissions & Umask
The **umask** (User Mask) determines default permissions assigned to newly created files and directories:
- **Default Base**: Directories start at `777`, Files start at `666` (no default execute).
- **Calculation**: Default permissions are calculated by subtracting the umask value from the base.
  - *If umask is `022`*: Directories are created with `755` (`777 - 022 = 755`); Files are created with `644` (`666 - 022 = 644`).
  - *If umask is `077`*: Directories are created with `700` (private); Files are created with `600`.

### 4. Access Control Lists (ACLs)
Standard permissions only allow defining one owner and one group. **ACLs** allow assigning unique permissions to multiple specific users or groups on a single file:
- **`getfacl`**: Displays the active ACL details of a file.
- **`setfacl`**: Modifies the file's ACL permissions.

---

## Commands & Syntax

### Bash
```bash
# Change file owner to 'john' and group to 'developers'
chown john:developers /var/www/index.html

# Grant read, write, and execute permissions to Owner, and read/execute to Group/Others (Octal)
chmod 755 /usr/local/bin/deploy-script.sh

# Add execute permission to user/owner only (Symbolic)
chmod u+x /usr/local/bin/deploy-script.sh

# Enable SGID on a collaborative directory (Symbolic 'g+s' or Octal prefix '2')
chmod g+s /opt/shared/
chmod 2770 /opt/shared/

# Enable the Sticky Bit on a directory (Symbolic 'o+t' or Octal prefix '1')
chmod +t /opt/tmp_folder/
chmod 1777 /opt/tmp_folder/

# Check the current shell's umask setting
umask

# View Access Control Lists (ACL) on a file
getfacl /etc/secure_config.conf

# Grant read/write ACL permissions to a specific user 'jane' on a file
setfacl -m u:jane:rw /etc/secure_config.conf

# Remove all custom ACLs from a file
setfacl -b /etc/secure_config.conf
```

---

## Real-World Scenarios

### Scenario 1: Developer Blocked from Executing Custom Script
**User Complaint:** A junior developer reports: *"I wrote a bash script named 'cleanup.sh' in my home directory. When I try to run it using './cleanup.sh', the terminal returns 'bash: ./cleanup.sh: Permission denied', even though I am the owner of the file."*
**Your First 3 Checks:**
1. Check the file permissions of `cleanup.sh` using `ls -l`.
2. Verify who owns the file.
3. Verify if the execution bit (x) is enabled.
**Diagnosis Steps:**
1. SSH to the developer's environment and locate the file. Run:
   `ls -l cleanup.sh`
   - Output: `-rw-r--r--. 1 devuser developers 240 Jun 25 12:00 cleanup.sh`
2. The user `devuser` is indeed the owner.
3. *Identify the permissions*: The permissions are `-rw-` for the owner. There is no `x` (execute) character. In Linux, files are created without the execution bit by default (for security).
**Root Cause:** The script file is missing the execute permission bit.
**Fix:**
1. Add the execute permission for the owner:
   `chmod u+x cleanup.sh`
2. Run `ls -l` to verify:
   - Output: `-rwxr--r--. 1 devuser developers ...`
3. Instruct the developer to run the script. It executes successfully.
**Prevention:** Educate developers to run `chmod +x` on scripts intended to be executed directly, or to run them via the interpreter: `bash cleanup.sh`.
**Ticket Close Note:** "Added execute permission to cleanup.sh using chmod u+x. Script runs successfully. Closed."

### Scenario 2: Collaborative Team Directory Overwriting Group Ownership
**User Complaint:** A team lead complains: *"We have a shared directory '/opt/marketing' where members edit assets. But when 'Jane' creates a file, 'John' cannot edit it. John gets a permission denied error, even though they both belong to the 'marketing' group."*
**Your First 3 Checks:**
1. Check the group ownership of the `/opt/marketing` directory.
2. Check the default primary groups of Jane and John.
3. Check the default permissions of new files created inside the directory.
**Diagnosis Steps:**
1. View the shared directory's permissions:
   `ls -ld /opt/marketing`
   - Output: `drwxrwxr-x. 2 root marketing ...` (Group owner is marketing).
2. Jane creates a file `campaign.txt` inside `/opt/marketing`. Check the file's properties:
   `ls -l /opt/marketing/campaign.txt`
   - Output: `-rw-r--r--. 1 jane jane ...`
3. *Why did this happen?* When Jane creates a file, the file's group defaults to Jane's primary group (`jane`), not the directory's group (`marketing`). Because the group permission is read-only (`r--`), John cannot write to it.
**Root Cause:** The directory lacked the SGID special permission, causing new files to inherit the creator's primary group instead of the directory's group.
**Fix:**
1. Enable SGID on the shared directory so new files inherit the `marketing` group:
   `chmod g+s /opt/marketing`
2. Change the permissions to allow group write access:
   `chmod 2770 /opt/marketing` (Octal `2` sets SGID, `770` grants full access to Owner and Group, and blocks others).
3. Correct existing files group ownership:
   `chown -R :marketing /opt/marketing`
   `chmod -R g+w /opt/marketing`
4. Test: Jane creates a new file. It displays group owner `marketing`, allowing John to edit it.
**Prevention:** Standardize all collaborative folders by applying the SGID bit during directory creation.
**Ticket Close Note:** "Configured SGID bit (chmod g+s) on the marketing directory. Corrected existing group ownerships. Group members can now edit shared files. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never run `chmod 777` on files or directories to resolve a permission block.
> - Setting `777` grants full read, write, and execute access to **every user on the system**, including guest accounts and automated system processes. This is a massive security vulnerability. Identify the specific user or group that needs access and apply targeted permissions instead (e.g., using `chmod 770` or ACLs).

> [!warning] Common Trap
> - Forgetting that directory write permissions control file deletion.
> - In Linux, write permission on a *directory* allows a user to delete files inside that directory, even if they have no read or write permissions on the *file* itself. To prevent users from deleting each other's files in a shared folder, you must enable the **Sticky Bit**.

> [!tip] Senior Engineer Tip
> - When troubleshooting permissions, check if **SELinux** is running in Enforcing mode. Even if `ls -l` shows `777` permissions, SELinux policies can still block access. If permissions look correct but access is blocked, check the SELinux logs (`/var/log/audit/audit.log`) or temporarily set it to Permissive: `setenforce 0`.

> [!success] Verification Steps
> - Run `ls -l` to verify the permission string matches the targeted structure.
> - Run `getfacl [filename]` to verify active Access Control Lists are assigned to the target users.

> [!question] Interview Alert
> - "What is the SGID bit on a directory and when do you configure it?"
> - Answer: "The SGID (Set Group ID) bit on a directory forces all newly created files and subdirectories inside it to inherit the group ownership of the parent directory, rather than the primary group of the user who created them. It is configured on collaborative department folders to ensure team members can read and edit each other's files."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Running `chmod 777` to fix errors | Quick, easy workaround | Target permissions specifically using chown and chmod, or use getfacl/setfacl for custom users. |
| Forgetting to use the `-R` recursive flag | Changing parent directory only | Use `chmod -R` or `chown -R` to apply changes recursively to all subfolders and files. |
| Neglecting the umask value | New files are created too open/closed | Configure default umask values in `/etc/login.defs` or `/etc/profile` to enforce company security baselines. |

---

## Lab Exercise

**Objective:** Create a collaborative directory, assign group ownership, configure SGID and the Sticky Bit, set a custom user ACL via `setfacl`, and verify the permissions.
**Time Required:** 30 minutes
**Environment Needed:** A Linux VM or sandbox environment.
**Pre-requisites:** Root or sudo access.

**Steps:**
1. Open a terminal. Create two test users and a security group:
   ```bash
   groupadd marketing
   useradd -g marketing john
   useradd -g marketing jane
   ```
2. Create the shared directory:
   ```bash
   mkdir -p /opt/marketing_docs
   ```
3. Set group ownership of the directory to `marketing`:
   ```bash
   chown root:marketing /opt/marketing_docs
   ```
4. Set permissions: grant owner and group full access, deny others, and enable **SGID** and the **Sticky Bit**:
   ```bash
   chmod 3770 /opt/marketing_docs
   ```
   - *Note: Octal '3' sets both SGID (2) and Sticky Bit (1).*
5. Verify directory permissions:
   ```bash
   ls -ld /opt/marketing_docs
   ```
   - Expected Output: `drwxrwxrwt.` (where `t` indicates the Sticky Bit is active).
6. Assign a specific user 'auditor' read-only Access Control List (ACL) access:
   ```bash
   useradd auditor
   setfacl -m u:auditor:rx /opt/marketing_docs
   ```
7. Verification: View the applied ACL rules:
   ```bash
   getfacl /opt/marketing_docs
   ```
   - Confirm user `auditor` is listed with `r-x` permissions.

**Success Criteria:** The directory is created, group ownership set, SGID/Sticky bit active, and custom user ACL verified.
**Common Failures:** The `setfacl` command fails if the hosting filesystem does not support or have ACL options enabled in mount parameters.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: What do the permissions `644` and `755` mean on a Linux file?**
A: `644` means the file owner has read and write permissions (6), while the group and others have read-only permissions (4). `755` means the owner has read, write, and execute permissions (7), while the group and others have read and execute permissions (5).

**Q: Which command do you use to change the owner of a file from 'root' to 'admin'?**
A: I run the command: `chown admin filename`. To change both owner and group at the same time, I run: `chown admin:groupname filename`.

### Intermediate (L2 Level)
**Q: What is the Sticky Bit and in what scenario is it used?**
A: The Sticky Bit is a special permission applied to directories that restricts file deletion. When active, only the owner of a file, the owner of the parent directory, or the root user can delete or rename files inside that directory, even if other users have write access. It is used on public directories like `/tmp` to prevent users from deleting each other's files.

**Q: How does the umask value `0022` affect the creation of new files and directories?**
A: Umask subtracts permissions from a base state. The base is `777` for directories and `666` for files. A umask of `0022` subtracts group and others write access, resulting in new directories having permissions of `755` (777 - 022) and new files having `644` (666 - 022).

### Advanced (L3/Senior Level)
**Q: A web application running under the user 'nginx' is unable to write logs to `/var/log/myapp/error.log`. Standard group permissions are locked for system reasons. How do you grant write access to Nginx specifically without breaking standard groups?**
A:
- **Situation**: Application user blocked from writing logs due to locked group permissions.
- **Task**: Grant Nginx write access specifically, complying with least privilege.
- **Action**: I implement Access Control Lists (ACLs). I check the current permissions using `getfacl /var/log/myapp/error.log`. To grant write access to the `nginx` user specifically, I run: `setfacl -m u:nginx:rw /var/log/myapp/error.log`. To ensure newly generated files in that log directory also inherit this permission, I set a default ACL on the parent directory: `setfacl -d -m u:nginx:rw /var/log/myapp/`.
- **Result**: Nginx writes logs successfully, while the standard owner and group permissions remain untouched.

### HR / Behavioral
**Q: Tell me about a time you identified a security vulnerability in your organization and how you corrected it.**
A: While auditing a web server, I found a developer had set permissions on a configuration folder containing database passwords to `777` because the app was throwing permission errors. This allowed anyone on the system to read the password file. I immediately revoked the permissions and set them to `600`. I found the web app ran under the user `apache`, so I changed the file owner to `apache`. I explained to the developer why `777` is dangerous and set up a training session on Linux permissions for the team.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The security model defining read, write, and execute privileges on Linux files and folders.
> **Why**: Secures system configuration files and isolates user directory structures.
> **How**: Use `chmod` to set permissions, `chown` to assign owners, enable special bits (SUID/SGID/Sticky), and configure ACLs for granular access.
> **Command**: `chmod 755` / `chown owner:group` / `setfacl -m`
> **Interview Answer Starter**: "To manage file security in Linux, I assign standard Owner-Group-Others permissions, utilizing special bits like SGID for shared spaces, and ACLs for custom access..."

**Key Numbers to Remember:**
- SUID value: `4`
- SGID value: `2`
- Sticky Bit value: `1`
- Directory base permission: `777` / File base: `666`

**3 Things Interviewer Wants to Hear:**
- Never use `chmod 777` to resolve permission issues
- SGID on a folder ensures new files inherit the group owner
- The difference between `chmod` (permissions) and `chown` (ownership)

---

## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/Filesystem|Linux Filesystem]] — Explains the directory hierarchy where permissions apply.
- [[04-Cloud-and-Security/09-Security/Access-Management|Access Management]] — Discusses privilege design principles.
- [[02-Operating-Systems/04-Linux-RHEL/Systemctl|Systemctl]] — Services managed by systemd require correct user execution rights.

---

## Tags
#desktop-support #linux #permissions #security #L1 #interview-topic #lab-complete #daily-use

