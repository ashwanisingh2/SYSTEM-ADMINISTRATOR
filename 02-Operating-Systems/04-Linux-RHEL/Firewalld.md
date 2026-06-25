---
tags: [desktop-support, linux, firewall, network-security, L2]
aliases: [firewalld-guide, firewall-cmd, rich-rules]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #rhcsa
---

# Linux Firewall Management (Firewalld)

---

## Concept Overview
- **What it is**: Firewalld is a dynamic firewall manager daemon standard on Red Hat Enterprise Linux (RHEL) systems. It provides a stateful packet filtering firewall, using **Zones** to classify network interface trust levels, and supports rule updates without dropping active network connections.
- **Why it matters for a support engineer**: A server's local firewall is its primary defense. Support engineers open network ports for newly deployed services, restrict administrative access to trusted management IP subnets, and block malicious scanning traffic in real-time.
- **Where you encounter this in real job**: Opening port `80/443` for web servers, blocking brute-force SSH attacks, checking active firewall rules, and assigning network interfaces to zones.
- **L1 vs L2 vs L3 responsibility split**:
  - **L1**: Checks firewall status (`systemctl status firewalld`), lists active rules (`firewall-cmd --list-all`), and verifies port availability.
  - **L2**: Configures zone bindings, opens permanent TCP/UDP ports, permits pre-configured system services, and reloads firewall daemons.
  - **L3**: Designs complex packet routing schemes, writes rich rules for custom network logging/rate-limiting, configures IP masquerading (NAT), and troubleshoots panic mode lockouts.

---

## Technical Deep Dive

### 1. The Concept of Firewall Zones
Firewalld organizes network traffic filtering using **Zones** based on the level of trust assigned to network interfaces:
- **`public`**: (Default) For untrusted networks. Allows only selected inbound connections (e.g., SSH).
- **`external`**: For external routers. Used when masquerading (NAT) is enabled to protect internal networks.
- **`dmz`**: For isolated demilitarized zone servers with limited access to internal networks.
- **`work` / `home` / `internal`**: For trusted local networks, permitting more standard services.
- **`trusted`**: **WARNING**: All network packets are accepted. Use only for internal loopbacks or isolated testing switches.

### 2. Runtime vs. Permanent Configuration
A crucial concept in firewalld operation:
- **Runtime Configuration**: Changes take effect immediately but **are lost** when the firewalld service is reloaded, restarted, or the server reboots. Great for temporary rule testing.
- **Permanent Configuration**: Changes are written to XML configuration files on disk. They **do not apply** immediately, but survive restarts. To activate permanent rules, you must run `firewall-cmd --reload`.

```
[firewall-cmd Command] --+---> (No --permanent flag) ---> [Runtime Memory] (Lost on reload)
                         |
                         +---> (--permanent flag) ------> [XML Config Files] --[--reload]--> [Runtime Memory] (Persistent)
```

### 3. Rich Rules
Rich Rules provide fine-grained, advanced access controls. While standard rules open a port to everyone, Rich Rules allow you to define combinations of source IP addresses, destination ports, logging, and actions:
- *Example Syntax*: `rule family="ipv4" source address="192.168.1.100" port port="22" protocol="tcp" accept`

---

## Commands & Syntax

### Bash
```bash
# Check if the firewalld service is active and running
firewall-cmd --state

# List all active configurations for the default zone
firewall-cmd --list-all

# List active configurations for a specific zone (e.g., work)
firewall-cmd --zone=work --list-all

# Check the default zone assigned to new network interfaces
firewall-cmd --get-default-zone

# Change the default zone to 'work'
firewall-cmd --set-default-zone=work

# Add a service (HTTP) temporarily (Runtime only)
firewall-cmd --add-service=http

# Add a port (TCP 8080) permanently
firewall-cmd --permanent --add-port=8080/tcp

# Remove a port rule permanently
firewall-cmd --permanent --remove-port=8080/tcp

# Reload the firewall to apply all permanent configurations
firewall-cmd --reload

# Add a Rich Rule to allow SSH access ONLY from a specific host IP permanently
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.50" port port="22" protocol="tcp" accept'

# Add a Rich Rule to reject all traffic from a malicious subnet permanently
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="198.51.100.0/24" reject'
```

### Configuration Directory Locations:
- User-defined configurations (Permanent settings): `/etc/firewalld/zones/`
- Default system templates: `/usr/lib/firewalld/zones/`

---

## Real-World Scenarios

### Scenario 1: New Web Service on Port 8080 is Unreachable Externally
**User Complaint:** A developer deploys a custom Java web application running on port 8080 on a RHEL server. They report: *"The application runs locally, and I can curl it from the server console. However, when I try to open 'http://server-ip:8080' from my laptop, the connection times out."*
**Your First 3 Checks:**
1. Verify if the web application is actively listening on port 8080 using `ss` or `netstat`.
2. Check the active default zone in firewalld.
3. Check the open ports list in the default zone using `firewall-cmd`.
**Diagnosis Steps:**
1. Verify application port status:
   `ss -tunlp | grep 8080`
   - Output: `tcp LISTEN 0 128 *:8080` (Listening on all interfaces).
2. Query active firewalld rules:
   `firewall-cmd --list-all`
   - Default Zone: `public`
   - Services: `cockpit dhcpv6-client ssh`
   - Ports: (Empty)
3. *Identify the block*: The application is listening, but firewalld only permits SSH and DHCP inbound. Port 8080 is blocked, causing remote connection requests to time out.
**Root Cause:** Firewalld rules did not permit inbound TCP traffic on custom port 8080.
**Fix:**
1. Add the port rule permanently to the default public zone:
   `firewall-cmd --permanent --add-port=8080/tcp`
2. Reload the firewall to apply the change:
   `firewall-cmd --reload`
3. Verify that the port is now listed:
   `firewall-cmd --list-all`
   - Ports: `8080/tcp`
4. Test the web application link from the developer's laptop. The site opens successfully.
**Prevention:** Integrate port configuration verification into standard application deployment runbooks.
**Ticket Close Note:** "Opened port 8080/tcp permanently in firewalld and reloaded. Verified connection from developer workstation. Closed."

### Scenario 2: Blocking a Brute-Force SSH Attack on a Public Linux Server
**User Complaint:** A security alert triggers: *"A Linux server hosted in our cloud environment is experiencing a high volume of failed SSH login attempts from IP address 198.51.100.42, risking account compromises."*
**Your First 3 Checks:**
1. Check the failed login logs in `/var/log/secure` to confirm the attack IP.
2. Verify if the attacker's IP is already blocked by other security groups.
3. Apply a blocking rich rule in firewalld to drop the traffic.
**Diagnosis Steps:**
1. Audit the secure log:
   `tail -n 20 /var/log/secure | grep "Failed password"`
   - Output: `Failed password for root from 198.51.100.42 port 49120 ssh2` repeated dozens of times.
2. The IP `198.51.100.42` is sending brute-force connection requests.
3. *Decision*: To block this IP immediately without disrupting legitimate SSH users on other IPs, we must write a rich rule to reject traffic from this source address.
**Root Cause:** Unauthorized brute-force access attempts from a public IP address.
**Fix:**
1. Add a rich rule to drop all traffic from the malicious IP address immediately (Runtime first to cut connection, then permanent):
   `firewall-cmd --add-rich-rule='rule family="ipv4" source address="198.51.100.42" drop'`
   `firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="198.51.100.42" drop'`
2. Verify the rule is active in the running configuration:
   `firewall-cmd --list-all`
   - Rich Rules: `rule family="ipv4" source address="198.51.100.42" drop`
3. Check `/var/log/secure` again. The SSH connections from the target IP have stopped, as packets are dropped before reaching the SSH service port listener.
**Prevention:** Install fail2ban to automate temporary IP blocking for failed logins.
**Ticket Close Note:** "Added rich rule to drop all inbound packets from 198.51.100.42. Checked logs and verified brute-force attempts have stopped. Closed."

---

## Critical Points

> [!danger] Never Do This
> - Never run `systemctl stop firewalld` on a public-facing production server to troubleshoot network issues.
> - Stopping the local firewall exposes all unpatched services, internal databases, and management ports on that server to the entire internet. Use Network utility tests (`telnet`, `nc`, `nmap`) or temporary runtime rules to troubleshoot instead.

> [!warning] Common Trap
> - Forgetting to add the `--permanent` flag when configuring firewall rules.
> - Running `firewall-cmd --add-port=80/tcp` works immediately and passes tests. However, the next time the server reboots or the firewall is reloaded, the rule is deleted from memory, causing a sudden service outage. Always verify with `firewall-cmd --reload` after writing permanent rules.

> [!tip] Senior Engineer Tip
> - If you lock yourself out of a remote server after writing a bad firewall rule, use the **Panic Mode** switch if you have console access: `firewall-cmd --panic-on`. This cuts off all inbound and outbound traffic completely, stopping active attacks. Run `firewall-cmd --panic-off` to restore standard rules.

> [!success] Verification Steps
> - Run `firewall-cmd --list-ports` and `firewall-cmd --list-services` to verify allowed protocols.
> - Test connectivity from an external machine using: `nc -zv [Server-IP] [Port]` to check port availability.

> [!question] Interview Alert
> - "How do you verify if a port rule will persist across a firewalld reload?"
> - Answer: "I check if the rule was written with the `--permanent` flag. I run `firewall-cmd --permanent --list-all` to inspect the rules stored on disk. If the port is listed there, it will persist after running `firewall-cmd --reload`."

---

## Common Mistakes & Fixes

| Mistake | Why It Happens | Correct Approach |
|---------|----------------|------------------|
| Forgetting to run `--reload` after a permanent change | Assuming rules apply immediately | Always run `firewall-cmd --reload` to copy disk configuration changes into active memory. |
| Assigning interfaces to the `trusted` zone | Trying to bypass firewall issues quickly | Isolate network ports using public/work zones, opening only required ports. |
| Writing overlapping rich rules | Lack of syntax planning | Keep rich rules simple, and use system security groups for complex IP routing tasks. |

---

## Lab Exercise

**Objective:** Inspect active zones, create a custom zone, allow a service and a custom TCP port permanently, reload firewalld, write a rich rule to permit a specific host, and verify active configurations.
**Time Required:** 30 minutes
**Environment Needed:** A RHEL VM.
**Pre-requisites:** Root or sudo access.

**Steps:**
1. Open a terminal. View active zones and configuration:
   ```bash
   firewall-cmd --get-active-zones
   ```
2. Create a new custom zone named `secureapp`:
   ```bash
   firewall-cmd --permanent --new-zone=secureapp
   firewall-cmd --reload
   ```
3. Add the `http` service and TCP port `9000` permanently to the new zone:
   ```bash
   firewall-cmd --permanent --zone=secureapp --add-service=http
   firewall-cmd --permanent --zone=secureapp --add-port=9000/tcp
   ```
4. Add a rich rule to the `secureapp` zone allowing SSH ONLY from a lab IP (`192.168.50.10`) permanently:
   ```bash
   firewall-cmd --permanent --zone=secureapp --add-rich-rule='rule family="ipv4" source address="192.168.50.10" service name="ssh" accept'
   ```
5. Reload the firewall to apply changes:
   ```bash
   firewall-cmd --reload
   ```
6. Verification: List the configuration of the custom zone:
   ```bash
   firewall-cmd --zone=secureapp --list-all
   ```
   - Confirm `http`, `9000/tcp`, and the rich rule are listed.

**Success Criteria:** The custom zone is created, service and port added, rich rule configured, and all configurations verified after reload.
**Common Failures:** The zone creation fails if the zone name contains capital letters or special characters, which are blocked by firewalld naming rules.

---

## Interview Questions & Answers

### Basic (L1 Level)
**Q: How do you check if the local RHEL firewall 'firewalld' is currently active and running?**
A: I run the command: `systemctl status firewalld` or the firewall-specific command: `firewall-cmd --state`.

**Q: What is the difference between running a firewall-cmd command with and without the '--permanent' flag?**
A: Without the `--permanent` flag, the change is applied immediately to the runtime memory but is deleted when the firewall is reloaded or the server reboots. With the `--permanent` flag, the change is written to configuration files on disk; it survives reboots but does not apply until you run `firewall-cmd --reload`.

### Intermediate (L2 Level)
**Q: How do you open TCP port 443 permanently in the default public zone?**
A: I run two commands:
1. `firewall-cmd --permanent --add-port=443/tcp` (writes the rule to disk)
2. `firewall-cmd --reload` (loads the rule into runtime memory)
I then verify it works by running `firewall-cmd --list-ports`.

**Q: What is a Firewall Zone in RHEL and how is it used?**
A: A zone is a feature that defines the trust level of your network connections and interfaces. You assign network cards to specific zones (e.g., public for internet-facing interfaces, work for internal subnets). The zone contains the rules that filter traffic coming into that specific interface.

### Advanced (L3/Senior Level)
**Q: A database server needs to accept connections on port 3306 (MySQL) but only from our application server (IP: 10.0.1.15). How do you configure this?**
A:
- **Situation**: MySQL database port must be restricted to a single application IP.
- **Task**: Implement a host-restricted firewall rule.
- **Action**: I implement a Rich Rule in the default zone. I run:
  `firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.1.15" port port="3306" protocol="tcp" accept'`
  I reload the firewall: `firewall-cmd --reload`. To verify the rule, I run `firewall-cmd --list-all` and check the "rich rules" section.
- **Result**: Inbound traffic on port 3306 is allowed from 10.0.1.15, while all other source IP attempts to reach the database port are rejected.

### HR / Behavioral
**Q: Describe a time you had to resolve a conflict with a colleague from a different department to fix an IT issue.**
A: A developer complained that our new firewall rules blocked their app, and they demanded we turn off the firewall on our servers. I explained that turning off the firewall would violate our security baseline. I sat down with them, and we reviewed their app call logs. We identified the exact ports their app needed (TCP 5000 and 5001). I wrote a permanent rule allowing those ports, reloaded the firewall, and verified the app worked. The developer was satisfied, and we kept the server secure.

---

## Quick Revision Sheet
> [!info] 60-Second Summary
> **What**: The dynamic firewall management service standard for RHEL Linux configurations.
> **Why**: Secures server interfaces by classifying trust levels via zones and filtering TCP/UDP ports.
> **How**: Use `firewall-cmd` to manage rules, differentiate between runtime and permanent states, and apply rich rules for host filtering.
> **Command**: `firewall-cmd --reload` / `firewall-cmd --list-all` / `firewall-cmd --permanent --add-port`
> **Interview Answer Starter**: "To manage host security in RHEL, I utilize firewalld to bind interfaces to appropriate zones, writing permanent port rules and applying rich rules for source-IP restrictions..."

**Key Numbers to Remember:**
- Default firewall zone: `public`
- Exit status code for a healthy state check: `0`
- User configurations path: `/etc/firewalld/`
- Default SSH port: `22`

**3 Things Interviewer Wants to Hear:**
- Never disable firewalld in production to troubleshoot
- Permanent rules do not take effect until `firewall-cmd --reload` is executed
- Rich rules permit filtering access based on source IP and target port combinations

---

## Related Notes
- [[01-Foundations/02-Networking/TCP-IP-and-Ports|TCP/IP and Ports]] — Underpins the port protocols filtered by firewalld.
- [[02-Operating-Systems/04-Linux-RHEL/Systemctl|Systemctl]] — The service manager that controls the firewalld daemon.
- [[04-Cloud-and-Security/09-Security/Zero-Trust|Zero Trust]] — The identity security model utilizing local host firewalls.

---

## Tags
#desktop-support #linux #firewall #network-security #L2 #interview-topic #lab-complete #daily-use

