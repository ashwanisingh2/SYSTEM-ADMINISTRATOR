---
tags: [sysadmin, linux, security, user-management]
difficulty: Intermediate
lab-required: Yes
read-time: 15 mins
---

# L-05: User and Group Management

> [!abstract] Overview
> This note covers Linux identity administration, including file formats (`/etc/passwd`, `/etc/shadow`, `/etc/group`), user/group lifecycle commands, privilege elevation models (`su` vs. `sudo`), and password aging configurations.

---
## Concept
Think of user management in Linux like organizing security permissions in a apartment complex:
- **`/etc/passwd`** is the public directory board in the lobby. Anyone can read it to see who lives in which room number (UID) and where their mailbox is located (Home Directory).
- **`/etc/shadow`** is the locked security safe behind the manager's desk. Only the building manager (root) can open it to verify the biometric key hashes (passwords) and check if rent is overdue (password expiration).
- **`/etc/group`** is the list of club memberships.
- **`su`** is climbing into the manager's office and changing into their uniform completely (unsecure, hard to track).
- **`sudo`** is having the manager sign a guest pass card allowing you to execute a single task with their authority, logged at the front desk (secure, fully audited).

*Seedha simple mein: Linux multi-user OS hai jahan users aur groups ki information text files (/etc/passwd, /etc/shadow) mein store hoti hai. Privilege management ke liye hum user ko sudo group mein add karte hain visudo tool ka use karke.*

---
## Technical Deep Dive

### 1. Identity File Formats

#### Parsing `/etc/passwd` (Readable by Everyone)
Each line represents a user, split by colons into seven fields:
```
sysadmin:x:1000:1000:Lead Administrator:/home/sysadmin:/bin/bash
   1     2   3    4           5                6           7
```
1. **Username:** Login identifier.
2. **Password Placeholder (`x`):** Indicates the encrypted password is stored in `/etc/shadow`.
3. **User ID (UID):** Unique integer. Root UID is always `0`. System users are typically `1-999`. Standard users start at `1000+`.
4. **Group ID (GID):** Primary group ID.
5. **GECOS (Comment):** User description (real name, phone).
6. **Home Directory:** Absolute path to user home folder (where shell starts).
7. **Default Shell:** Shell program launched on login (e.g., `/bin/bash` or `/sbin/nologin` to block login).

#### Parsing `/etc/shadow` (Readable by Root Only)
Contains encrypted password hashes and aging limits:
```
sysadmin:$6$salt$hash:19500:0:90:7:5:20000:
   1          2         3   4 5  6 7   8    9
```
1. **Username.**
2. **Password Hash:** Pre-fixed by algorithm identifier (e.g., `$6$` is SHA-512, `$y$` is Yescrypt). `*` or `!` indicates account is locked.
3. **Last Password Change:** Number of days since Jan 1, 1970 (Epoch time) that password was modified.
4. **Minimum Age:** Days user must wait before they can change password again.
5. **Maximum Age:** Days the password remains valid.
6. **Warning Period:** Days warning before password expires.
7. **Inactivity Period:** Days after expiration before account is disabled.
8. **Expiration Date:** Date account is disabled (in days since Epoch).
9. **Reserved Field.**

#### Parsing `/etc/group`
```
wheel:x:10:sysadmin,jdoe
  1   2  3      4
```
1. **Group Name.**
2. **Group Password Placeholder.**
3. **Group ID (GID).**
4. **Group Members:** Comma-separated list of users in this supplementary group.

### 2. User Lifecycle Management
- **`useradd` Options:**
  - `useradd -u 1500 -g developers -G git,wheel -c "John Doe" -m -s /bin/bash jdoe`
    - `-u` Sets UID.
    - `-g` Sets primary group (user must belong to exactly one).
    - `-G` Sets supplementary groups.
    - `-c` Adds GECOS comment.
    - `-m` Forces creation of home directory (`/home/jdoe`).
    - `-s` Sets login shell.
- **`usermod` (Modify User):**
  - `usermod -aG sudoers jdoe` — Append (`-a`) user to supplementary group (`-G`). **WARNING:** Always use `-a` alongside `-G`; neglecting `-a` removes the user from all other supplementary groups.
  - `usermod -L jdoe` — Lock account password.
  - `usermod -U jdoe` — Unlock account password.
- **`userdel` (Delete User):**
  - `userdel -r jdoe` — Delete user and recursively remove (`-r`) their home directory and mail spool.

### 3. Primary vs. Supplementary Groups
- **Primary Group:** Assigned during creation. Every file the user creates is owned by this group by default.
- **Supplementary Groups:** Additional groups assigned to grant secondary access (e.g., adding user to `wheel` or `sudo` to grant admin rights).

### 4. Privilege Elevation: su vs. sudo
- **`su` (Switch User):**
  - `su -` switches shell to the root user environment. Requires the **root password**.
  - *Risk:* Forces sharing the root password with multiple administrators. No audit logs tracking which administrator executed which command.
- **`sudo` (Superuser Do):**
  - Executes a single command with root privileges. Requires the **user's own password**.
  - *Audit:* Logs all commands to `/var/log/secure` (Red Hat) or `/var/log/auth.log` (Ubuntu), linking actions to the specific user account.

### 5. Configuring Sudoers (/etc/sudoers)
- **CRITICAL RULE:** Never edit `/etc/sudoers` with standard editors. Always use **`visudo`**. Visudo locks the file and performs syntax validation before saving, preventing configurations that could lock everyone out of root access.
- **Sudo rule format:**
  `sysadmin   ALL=(ALL)   ALL`
  - *Fields:* User host=(run_as_user:run_as_group) commands_allowed.
  - `%wheel  ALL=(ALL)   ALL` — Allows all members of the `wheel` group to run any command as root.
  - `%wheel  ALL=(ALL)   NOPASSWD: ALL` — Run all commands without prompting for password.

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A running Linux VM and root terminal access.

### Step 1: Create Group and User
1. Log in as root.
2. Create a new group named `security`:
   ```bash
   groupadd security
   ```
3. Create user `jdoe` with primary group `security` and supplementary group `wheel` (RHEL administrator group):
   ```bash
   useradd -g security -G wheel -c "John Doe" -m jdoe
   ```
4. Set password for `jdoe`:
   ```bash
   passwd jdoe
   ```

### Step 2: Configure Password Expiration Aging
1. Configure password policy for `jdoe` using `chage`:
   - Password expires after 90 days.
   - User must wait 2 days before changing it.
   - System warns user 7 days before expiration.
   ```bash
   chage -M 90 -m 2 -W 7 jdoe
   ```
2. Verify policy settings:
   ```bash
   chage -l jdoe
   ```

### Step 3: Configure Custom Sudo Rules via Visudo
1. Open sudoers using visudo:
   ```bash
   visudo
   ```
2. Scroll down. Append a rule allowing members of the `security` group to restart the systemd-journald service without password prompts:
   ```text
   %security  ALL=(root)  NOPASSWD: /usr/bin/systemctl restart systemd-journald
   ```
3. Save and close visudo (`:wq`).
4. Switch user to `jdoe`: `su - jdoe`.
5. Test the sudo execution:
   ```bash
   sudo systemctl restart systemd-journald
   ```
   **Verify:** The service restarts successfully without asking for root or user passwords. Attempting any other sudo command (e.g., `sudo systemctl stop firewalld`) should prompt for password and return access denied.

---
## Commands Reference
```bash
# Display active password aging policy for user jdoe
chage -l jdoe

# Lock local user account jdoe instantly
sudo usermod -L jdoe

# Check group memberships for user jdoe
groups jdoe
```

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** When attempting to run `sudo`, a standard user receives: `sysadmin is not in the sudoers file. This incident will be reported.`
- **Root Cause:** The user account is not a member of the administrative group (`wheel` on RHEL, `sudo` on Debian), or has no explicit rule in `/etc/sudoers`.
- **Fix:**
  1. Log in as root (or a user with existing admin privileges).
  2. Add the user to the default admin group:
     - On RHEL/CentOS:
       ```bash
       usermod -aG wheel sysadmin
       ```
     - On Ubuntu/Debian:
       ```bash
       usermod -aG sudo sysadmin
       ```
  3. Log the user out and log back in to refresh group security tokens.

**Scenario 2:**
- **Problem:** An administrator made a syntax error in the `/etc/sudoers` file manually, and now running `sudo` returns syntax errors and fails, locking them out of root access (they do not know the direct root password).
- **Root Cause:** Manual edit of sudoers file without syntax checking.
- **Fix:**
  1. If existing shell sessions are open as root, use them to run `visudo` and fix the error.
  2. If locked out, reboot the server. Intercept GRUB bootloader.
  3. Edit boot line, append `init=/bin/bash` or `systemd.unit=rescue.target` to bypass standard boot initialization, loading a direct root shell.
  4. Remount root filesystem as writeable:
     ```bash
     mount -o remount,rw /
     ```
  5. Run `visudo` to repair the syntax error. Save, remount read-only, and reboot.

---
## Common Mistakes
> [!warning] Avoid These
> **Running usermod without the append flag:** Executing `usermod -G devs jdoe` to add user `jdoe` to the `devs` group. Failing to specify the `-a` (append) flag will remove `jdoe` from all other supplementary groups (including `wheel` or `sudo`), stripping their admin privileges.
> **Correct approach:** Always specify the append flag together with groups: `usermod -aG devs jdoe`.

---
## Pro Tips
> [!tip] Field Experience
> When deleting old employee accounts, never use `userdel` immediately. First, lock the account (`usermod -L [user]`) and change their login shell to `/sbin/nologin`. Keep the account deactivated for 30 days. This allows you to verify if any running production cron scripts, databases, or API integrations depend on that user ID before deleting their home directory files.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | /etc/passwd | Text file storing user accounts, UID, home directory paths, and default shell configurations. |
| 2 | /etc/shadow | Root-only file storing encrypted password hashes and password expiration thresholds. |
| 3 | visudo | Safe editor tool that locks and validates the sudoers configuration file before saving. |
| 4 | su vs sudo | su switches environment to target user (root); sudo executes single commands under user audit. |
| 5 | chage | Password aging utility; configures warning times and maximum password validity days. |

---
## Interview Q&A

**Q1: Explain what happens when you run the command `sudo -i`. How does it differ from `su`?**
A: **`su`** (Switch User) prompts for the target user's (usually root) password and starts a new shell. **`sudo -i`** prompts for the *current user's* password, validates their sudo rights against `/etc/sudoers`, and if permitted, simulates a login shell for the root user. It imports root's environment variables (`PATH`, home directory `/root`, shell). It is preferred because it logs the escalation under the administrator's own account, avoiding the need to share the raw root password.

**Q2: A security auditor demands that no standard users should be allowed to reuse their last 5 passwords. How do you implement this in RHEL?**
A: 
- **Situation:** Enforce password history limits for security compliance.
- **Task:** Configure PAM (Pluggable Authentication Modules) parameters to remember password history.
- **Action:** I will edit the PAM configuration file `/etc/pam.d/system-auth` (and `/etc/pam.d/password-auth` on RHEL). I will locate the line referencing the `pam_pwhistory.so` or `pam_unix.so` module and append the `remember=5` argument.
  - *Example line:* `password sufficient pam_unix.so sha512 shadow remember=5`
- **Result:** The system will store hashes of the last 5 passwords in `/etc/security/opasswd` and block users from reusing them.

**Q3: What is the significance of UID 0 in Linux?**
A: In Linux, the operating system kernel does not care about the text name "root." Instead, it checks the User Identifier (UID). Any account assigned **UID 0** is treated by the kernel as the superuser (root), granting it complete bypass of all filesystem permissions, access to all memory blocks, and rights to execute any system calls. If a hacker modifies a standard user account in `/etc/passwd` to have GID/UID 0, that user becomes a root administrator.

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — Owner metadata and directory lists.
- [[02-Operating-Systems/04-Linux-RHEL/L-06 File Permissions and Ownership|L-06 File Permissions and Ownership]] — Controlling file access based on UID/GID.
- [[02-Operating-Systems/04-Linux-RHEL/L-09 SSH Configuration and Security|L-09 SSH Configuration and Security]] — Hardening remote admin logins.

