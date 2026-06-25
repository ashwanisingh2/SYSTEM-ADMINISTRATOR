---
tags: [sysadmin, linux, security, ssh]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# L-09: SSH Configuration and Security

> [!abstract] Overview
> This note covers Secure Shell (SSH) remote management, server hardening parameters, key-based authentication setups, port forwarding tunnels, and brute-force protection (fail2ban).

---
## Concept
Think of SSH as a highly secure, digital diplomatic courier tunnel. 
By default, the gate is open on Port 22. 
- **Password Authentication** is like entry via a password key-phrase: if someone guesses your phrase, they get in (brute-force vulnerability).
- **Key-Based Authentication** is like a custom physical vault lock system. You generate a padlock (Public Key) and its matching key (Private Key). You put the padlock on the server's door. Anyone can see the lock, but only you, holding the physical private key on your laptop, can unlock it and enter.
- **Fail2ban** is the armed security guard hiding in the bushes. If someone tries to pick the lock 5 times rapidly with bad keys, the guard locks them in a jail (Firewall block) for 10 minutes.

*Seedha simple mein: SSH remote servers ko securely manage karne ka standard tool hai. Security best practices ke liye hum default port 22 change karte hain, direct root login block karte hain, aur password logins ko disable karke key-based authentication configure karte hain.*

---
## Technical Deep Dive

### 1. SSH Server Configuration Hardening (/etc/ssh/sshd_config)
The SSH daemon configuration controls authentication and transport rules. Important parameters to secure:
- **`Port 2222`** — Changes default port from 22 to mitigate automated bot scans.
- **`PermitRootLogin no`** — Blocks the root user from logging in directly over SSH. Administrators must log in as standard users and elevate via `sudo`.
- **`PasswordAuthentication no`** — Disables password-based logins, requiring all users to authenticate using SSH keys.
- **`PubkeyAuthentication yes`** — Enables SSH key-based authentication.
- **`X11Forwarding no`** — Disables graphical interface forwarding, reducing overhead and security risks.
- **Access Restrictions:**
  - `AllowUsers sysadmin devuser` — Limits logins exclusively to listed users.
  - `DenyUsers root guest` — Explicit blocks.

### 2. Key-Based Authentication Mechanics
SSH keys utilize asymmetric cryptography (RSA, ECDSA, Ed25519).
- **Public Key (`id_ed25519.pub`):** Stored on the remote server inside the user's home folder: `~/.ssh/authorized_keys`. Does not need to be kept secret.
- **Private Key (`id_ed25519`):** Stored securely on the client machine. **MUST NEVER** be shared. Protected with a passphrase.
- **How it works:**
  1. Client sends a login request with its public key identifier.
  2. Server generates a random challenge message, encrypts it using the client's public key, and sends it back.
  3. Client decrypts the challenge using its local private key and returns the decrypted signature.
  4. Server verifies the signature; login is granted. The password never travels across the network.

### 3. SSH Port Forwarding (Tunneling)
SSH can encapsulate other TCP traffic inside the encrypted SSH stream.
- **Local Port Forwarding (`-L`):** Maps a port on the client machine to a port on the remote server.
  - *Command:* `ssh -L 8080:localhost:80 web-srv01` (Browsing to `localhost:8080` locally connects securely to port 80 on the remote web server).
- **Remote Port Forwarding (`-R`):** Maps a port on the remote server to a port on the client machine (used to bypass NAT firewalls).

### 4. Fail2ban Brute-Force Protection
Fail2ban scans system logs (e.g., `/var/log/secure`) for repeated authentication failures. If a threshold is reached, it dynamically writes firewalld/iptables rules to block the attacker's IP address.
- **Jail Configuration (`/etc/fail2ban/jail.local`):**
  - `bantime = 10m` — Duration of IP block.
  - `findtime = 10m` — Window to track failures.
  - `maxretry = 5` — Failed attempts allowed before block.

---
## Windows/Linux SSH Configuration Commands

### Generating and Copying Keys
```bash
# Generate a modern, secure Ed25519 SSH key pair on client
ssh-keygen -t ed25519 -C "admin@company.com"

# Copy public key to remote server (automates auth keys creation)
ssh-copy-id -i ~/.ssh/id_ed25519.pub sysadmin@192.168.10.20
```

### SSH Connection Test
```bash
# Log in specifying a custom port
ssh -p 2222 sysadmin@192.168.10.20
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> Two Linux VMs: Client VM (`Rocky-Client`) and Server VM (`Rocky-Server` at `192.168.10.20`).

### Step 1: Generate Key and Establish Connection
1. On `Rocky-Client`, generate the key pair. Press Enter to accept default paths and set a passphrase:
   ```bash
   ssh-keygen -t ed25519
   ```
2. Copy the public key to the server user `sysadmin`:
   ```bash
   ssh-copy-id sysadmin@192.168.10.20
   ```
3. Test key login:
   ```bash
   ssh sysadmin@192.168.10.20
   ```
   **Verify:** The login should succeed without prompting for the user's account password (it will only prompt for the private key passphrase if configured).

### Step 2: Harden the SSH Server Configuration
1. On `Rocky-Server`, log in as root and edit the configuration:
   ```bash
   vim /etc/ssh/sshd_config
   ```
2. Modify or add the following parameters (uncomment where necessary):
   ```text
   Port 2222
   PermitRootLogin no
   PasswordAuthentication no
   AllowUsers sysadmin
   ```
3. Save and close (`:wq`).
4. **CRITICAL:** If SELinux is active, you must authorize the new port before restarting SSH:
   ```bash
   semanage port -a -t ssh_port_t -p tcp 2222
   ```
5. Open the firewall port:
   ```bash
   firewall-cmd --permanent --add-port=2222/tcp
   firewall-cmd --permanent --remove-service=ssh
   firewall-cmd --reload
   ```
6. Restart the SSH service:
   ```bash
   systemctl restart sshd
   ```

### Step 3: Verify Hardening Settings
1. Open a new terminal on `Rocky-Client`. Attempt standard login:
   ```bash
   ssh sysadmin@192.168.10.20
   ```
   **Verify:** Connection fails because port 22 is blocked.
2. Attempt login on custom port:
   ```bash
   ssh -p 2222 sysadmin@192.168.10.20
   ```
   **Verify:** Connection succeeds.
3. Attempt root login: `ssh -p 2222 root@192.168.10.20`.
   **Verify:** Connection is rejected by the server immediately.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Copying the SSH key fails, or SSH key authentication is ignored. The server still prompts for password login, despite public keys being copied to `authorized_keys`.
- **Root Cause:** Secure directory permissions violation. SSH strictly rejects key logins if the client or server `~/.ssh` directory or `authorized_keys` file has write access granted to Group or Others.
- **Fix:**
  1. Log into the remote server as the target user.
  2. Enforce secure permissions on the SSH directory and keys:
     ```bash
     chmod 700 ~/.ssh
     chmod 600 ~/.ssh/authorized_keys
     ```
  3. Re-test SSH key authentication.

**Scenario 2:**
- **Problem:** After changing the SSH port to `2222` and restarting the daemon, the SSH service fails to start, logging error: `sshd.service: main process exited, code=exited, status=255/EXCEPTION`.
- **Root Cause:** SELinux is active in Enforcing mode and blocks the SSH process from binding to a non-standard port (port 2222 is blocked by default policy).
- **Fix:**
  1. Temporarily set SELinux to permissive to verify:
     ```bash
     setenforce 0
     ```
  2. Start the service: `systemctl start sshd`. If it starts, SELinux is the culprit.
  3. Add the port rule to the SELinux policy store:
     ```bash
     semanage port -a -t ssh_port_t -p tcp 2222
     ```
  4. Enable SELinux enforcing: `setenforce 1`.
  5. Restart SSH to confirm stability.

---
## Common Mistakes
> [!warning] Avoid These
> **Disabling password authentication before verifying key login:** Modifying `PasswordAuthentication no` and restarting SSH without having an active terminal session open to test the keys. If the keys fail, you are locked out of the remote server.
> **Correct approach:** Always keep your current SSH connection active. Open a *second* terminal window and test the key login. If it fails, you can correct the configuration in your active session.

---
## Pro Tips
> [!tip] Field Experience
> When generating SSH keys, use **Ed25519** (`ssh-keygen -t ed25519`) instead of legacy RSA. Ed25519 keys are physically shorter (easier to copy), computationally faster, and offer significantly higher security than standard 2048-bit or 4096-bit RSA keys.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Port 22 | Default SSH port; should be changed to a custom port to mitigate brute-force bots scans. |
| 2 | ssh-copy-id | Script that securely copies your public key to the remote server's `authorized_keys` file. |
| 3 | Key Permissions | The `~/.ssh` directory must be set to `700` and `authorized_keys` to `600` or key login fails. |
| 4 | Visudo / Sudoers| Elevation configs; visudo should be used to edit rights before restricting direct root login. |
| 5 | fail2ban | Security tool that dynamically blocks IP addresses exhibiting repeated login failures. |

---
## Interview Q&A

**Q1: Explain how SSH key-based authentication works using asymmetric encryption.**
A: SSH key-based authentication uses public-private key cryptography. The client has a private key, and the server holds the corresponding public key inside `~/.ssh/authorized_keys`. 
1. The client requests a connection.
2. The server generates a random string (challenge) and encrypts it using the client's public key.
3. The server sends the encrypted challenge to the client.
4. The client decrypts the challenge using its private key, signs it, and returns it to the server.
5. The server uses the public key to verify the signature. If matches, the identity is authenticated, and login is granted.

**Q2: A server's SSH port was changed to 2222. An administrator cannot connect. Describe your step-by-step diagnostic workflow.**
A: 
- **Situation:** SSH connection fails on a custom port 2222.
- **Task:** Diagnose the connection path, port status, and firewalls.
- **Action:** First, I will verify network connectivity by pinging the server. Second, I will check if port 2222 is open on the server from the client using `nc -zv [Server_IP] 2222`. Third, if blocked, I will log in locally and verify if the SSH service is running (`systemctl status sshd`) and listening on 2222 (`ss -tulnp`). Fourth, I will check the local firewall rules (`firewall-cmd --list-all`) and SELinux port policy.
- **Result:** Resolving firewalld port mappings or SELinux policies allows connection traffic to reach the port.

**Q3: What is the difference between Local Port Forwarding and Remote Port Forwarding in SSH?**
A: **Local Port Forwarding** (`ssh -L`) redirects traffic from a port on the client (local) machine to a port on the remote server. It allows you to access a service running on the server (like a database or web dashboard) that is not exposed to the public network. **Remote Port Forwarding** (`ssh -R`) redirects traffic from a port on the remote server to a port on the client machine. It allows a remote server on the internet to access a local service running on your local computer, bypassing local NAT firewalls.

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-02 Command Line Basics|L-02 Command Line Basics]] — Standard terminal network diagnostics.
- [[02-Operating-Systems/04-Linux-RHEL/L-05 User and Group Management|L-05 User and Group Management]] — Wheel group configurations and visudo controls.
- [[02-Operating-Systems/04-Linux-RHEL/L-12 Network Configuration in Linux|L-12 Network Configuration in Linux]] — Configuration of firewalld ports.

