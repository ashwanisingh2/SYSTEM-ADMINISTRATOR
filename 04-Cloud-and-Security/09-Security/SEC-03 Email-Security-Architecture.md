---
tags: [desktop-support, security, email-security, architecture, L2]
aliases: [email-security-architecture, sec-03]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #none
---

# SEC-03: Email Security Architecture (SPF, DKIM, DMARC, and SEG)

> [!note] Overview
> This note covers secure enterprise email routing architectures. It details Secure Email Gateways (SEG) integration, MX record management, transport-level security (Opportunistic vs. Forced TLS, MTA-STS, DANE), SMTP relay mechanisms, and domain authentication protocols.

---
## Concept Overview
- **What it is** — Email Security Architecture is the design of defensive systems protecting the organization's email pipeline. It includes perimeter protection (Secure Email Gateways), DNS-based domain authentication (SPF, DKIM, DMARC), and secure transport protocols (TLS, MTA-STS) protecting mail in transit.
- **Why it matters** — Over 90% of cyberattacks begin with a phishing email. A weak email security architecture exposes employees to malware links and credential theft, and allows attackers to spoof the corporate domain to defraud partners and clients.
- **Real job encounter** — Configuring MX records for spam gateways (e.g. Proofpoint), setting up secure SMTP relays for office copy-scanners, and enforcing TLS-encrypted email connections with corporate partners.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Analyze email headers to trace routing hops, retrieve false-positive messages from quarantine queues, and investigate spam notifications.
  - **Escalation Trigger**: Escalate if emails to key partners fail delivery due to TLS handshake failures, or if email domains are blacklisted globally.
  - **L2 Resolution**: Author and update SPF/DKIM/DMARC DNS records, configure inbound/outbound mail flow rules (transport rules), and configure spam filter policies.
  - **L3 Resolution**: Design hybrid mail routing architectures, configure Secure Email Gateway (SEG) MX routes, implement MTA-STS policies, and manage secure SMTP relay servers.

---
## Technical Deep Dive

### 1. Inbound Mail Flow with Secure Email Gateways (SEG)
A Secure Email Gateway (SEG) is an email proxy server situated at the network perimeter.
- To implement a SEG (e.g., Proofpoint, Cisco IronPort, Microsoft Defender for Office 365), you must modify your public **MX (Mail Exchanger) record** to point to the SEG.
- All external mail servers are forced to route mail to the SEG first.
- The SEG decrypts the TLS connection, performs spam filtering, antivirus scanning, URL rewriting, and reputation checks, and forwards clean messages to the target mail server.

```
       [ External Sender ]
                |
          Routs via MX Record
                |
                v
  +---------------------------+
  | Secure Email Gateway (SEG)| (Scans for spam, virus, bad links)
  +---------------------------+
                |
        Forwards Clean Mail
                |
                v
  +---------------------------+
  |    Mail Server Host       | (Exchange Online / On-Premises)
  +---------------------------+
```

### 2. Transport-Level Encryption (SMTP Security)
- **Opportunistic TLS**: By default, SMTP attempts to encrypt connections using TLS. If the receiving mail server does not support TLS, the sender automatically falls back to unencrypted cleartext SMTP, exposing traffic to eavesdropping.
- **Forced TLS (Secure Connector)**: Bypasses fallback. If the target server does not establish a valid TLS handshake, the mail is blocked and queued, preventing plaintext leaks.
- **MTA-STS (Mail Transfer Agent Strict Transport Security)**:
  - A DNS and HTTPS standard that allows domain owners to declare that their mail servers require TLS encryption.
  - Senders check the policy file via HTTPS and refuse to send mail if TLS cannot be established, preventing MitM downgrade attacks.
- **DANE (DNS-based Authentication of Named Entities)**: Uses DNS Security Extensions (DNSSEC) to bind TLS certificates directly to the domain's DNS, verifying server authenticity.

### 3. SMTP Relays
An **SMTP Relay** is a server that forwards email on behalf of client devices.
- In enterprises, network copiers, scanners, and applications (like ERP systems) need to send automated emails but cannot perform complex authentication.
- Administrators set up an internal SMTP relay. Devices send mail anonymously to the relay over port 25 (restricted to internal IP subnets), and the relay authenticates and forwards the emails to external recipients.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Rocky Linux 9 server (`SVR-RELAY01` with IP `192.168.10.40`).
> - A public DNS registrar console (to query/configure records).
> - Root or sudo access on the server.

### Step 1: Query MX Records using nslookup
1. Check the active mail servers for your domain:
```cmd
nslookup -type=mx corp.com
```
**Expected Output:**
```text
corp.com   MX preference = 10, mail exchanger = corp-com.mail.protection.outlook.com
```

### Step 2: Install and Configure an Internal SMTP Relay using Postfix
We will configure Postfix on Rocky Linux to act as a secure internal SMTP relay.

1. Install Postfix:
```bash
sudo dnf install -y postfix
```
2. Open the main configuration file:
```bash
sudo vi /etc/postfix/main.cf
```
3. Edit the following parameters:
```text
# Bind to all interfaces to listen to local clients
inet_interfaces = all

# Restrict relay access to local subnet clients only
mynetworks = 127.0.0.0/8, 192.168.10.0/24

# Set the smart host (external mail server) to route mail
relayhost = [corp-com.mail.protection.outlook.com]:25
```
4. Start and enable Postfix service:
```bash
sudo systemctl enable --now postfix
```
5. Test sending an email from the relay using `mailx`:
```bash
sudo dnf install -y mailx
echo "Test Alert from Server Relay" | mail -s "System Alert" user@corp.com
```
**Expected Output:** The mail is accepted by Postfix and relayed to Microsoft 365. Verify the logs in `/var/log/maillog`.

---

### Step 3: Deploy MTA-STS Policy
MTA-STS requires publishing a DNS record and hosting a policy file over HTTPS.

1. Create a public DNS TXT record declaring MTA-STS availability:
   - Hostname: `_mta-sts.corp.com`
   - Value: `v=STSv1; id=2026062501;`
2. Host the policy file on a web server accessible at: `https://mta-sts.corp.com/.well-known/mta-sts.txt`.
3. Configure the `mta-sts.txt` policy content:
```text
version: STSv1
mode: enforce
mx: *.mail.protection.outlook.com
max_age: 604800
```
*Note: This enforces TLS for all incoming mail sent to `corp.com` by external MTA-STS compliant senders.*

---
## Cheat Sheet / Quick Reference

| Record / Setting | Purpose | Configuration Example |
|---|---|---|
| **MX Record** | Directs domain email traffic to specific gateway servers | `10 company-com.mail.protection.outlook.com` |
| **MTA-STS (TXT)** | DNS record announcing strict TLS transport support | `v=STSv1; id=2026062501` |
| **SMTP Port 25** | Default unencrypted port used for server-to-server mail transit | Restricted at firewalls |
| **SMTP Port 587** | Default secure submission port (requires TLS/Auth) | Used by email client configurations |
| **SMTP Port 465** | Legacy SSL submission port | Avoid in modern setups |
| `/etc/postfix/main.cf` | Primary configuration file for Postfix relays | Configuration file |
| `postconf -n` | Lists all active non-default Postfix configurations | `postconf -n` |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Outbound emails fail with: "550 5.7.1 Service unavailable; Client host blocked." | The SMTP relay server's public IP has been blacklisted due to spam propagation (open relay). | Verify Postfix settings. Ensure `mynetworks` restricts access to internal subnets only. Run tests to confirm it is not an **Open Relay**. Request removal from blacklist. |
| Inbound email delivery fails with: "Loops back to myself." | The mail server's internal DNS zone routes MX records back to the local relay rather than the mail host. | Configure the split-brain DNS zones to ensure internal MX records point directly to the mail server IP or the local SEG. |
| Inbound emails to the organization bypass the Secure Email Gateway (SEG). | The firewall allows direct SMTP (port 25) connections from any internet IP directly to the internal mail servers. | Configure the mail server's firewall to allow inbound TCP Port 25 traffic **only** from the IP addresses of the SEG. |
| Secure emails fail to deliver to partner domain, logs show "TLS Negotiation failed." | The partner organization's TLS certificate has expired, or cipher suites do not match. | Temporarily bypass Forced TLS setting for that specific partner domain in the connector properties, and notify their IT department to renew the certificate. |
| Postfix relay logs return: "Host connection refused." | Local firewalls block outbound UDP/TCP port 25 on the relay server. | Open outbound port TCP 25 on your perimeter and local firewalls for the SMTP relay host. |

---
## Interview Questions

> [!question] L1 Question
> **Q:** How do you determine the routing path an email took by looking at its headers?
> **A:** I will open the email headers and analyze the **`Received:`** lines. These headers are appended by every mail server (MTA) that handles the message, starting from the bottom (the original sender) up to the top (the final receiving mailbox server). I can inspect timestamps and IP addresses to identify delays or routing hops.

> [!question] L2 Question
> **Q:** What is an "Open Relay", and why is it dangerous?
> **A:** An Open Relay is an SMTP server configured to accept and forward mail from any source IP address to any destination domain without authentication. This is highly dangerous because spammers can discover and abuse the relay to send millions of unsolicited emails, causing the server's public IP address to be blacklisted globally, blocking legitimate corporate emails.

> [!question] L3/Scenario Question
> **Q:** Explain the difference between Opportunistic TLS and Forced TLS. How does MTA-STS improve email transport security?
> **A:** 
> - **Situation:** Securing email data in transit over SMTP.
> - **Task:** Compare transit encryption models and define MTA-STS security value.
> - **Action:** 
>   - **Comparison**: **Opportunistic TLS** attempts to encrypt the connection but falls back to plaintext if the receiver fails the handshake, exposing mail to interception. **Forced TLS** requires encryption and drops the connection if TLS fails, but must be configured manually for specific domains.
>   - **MTA-STS Role**: **MTA-STS** automates forced TLS at scale. It publishes a policy via HTTPS and DNS. When an external mail server attempts to send an email to our domain, it checks our MTA-STS policy. If the policy is set to `enforce`, the sending server is forbidden from falling back to plaintext, preventing downgrade attacks.
> - **Result:** Transport-level encryption is guaranteed for all incoming email connections.

---
## Seedha Simple Mein
*Seedha simple mein: Email Security Architecture ka matlab hai email pipeline ko secure banana. Isme hum spam filters (Secure Email Gateways), DNS records (SPF, DKIM, DMARC), aur transport security (MTA-STS, TLS) ka use karte hain taaki spam/phishing emails block ho sakein aur mail transfer encrypted ho.*

---
## Related Notes
- [[04-Cloud-and-Security/07-Microsoft-365/M365-06 Email-Security-Deep-Dive|M365-06 Email-Security-Deep-Dive]] — M365-specific SPF/DKIM/DMARC setups.
- [[01-Foundations/02-Networking/N-08 IP Services — DHCP DNS NAT|N-08 IP Services — DHCP DNS NAT]] — DNS record details.
- [[04-Cloud-and-Security/09-Security/Incident-Response-Playbook|Incident-Response-Playbook]] — Remediating phishing incidents.

---
*Tags: #desktop-support #security #email-security #architecture #L2*
