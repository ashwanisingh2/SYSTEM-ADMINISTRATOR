---
tags: [sysadmin, windows-server, pki, certificates, adcs]
difficulty: Advanced
lab-required: Yes
read-time: 15 mins
---

# WS-13: Active Directory Certificate Services (AD CS)

> [!abstract] Overview
> This note covers Public Key Infrastructure (PKI) deployment within Windows Server. It details Root and Subordinate Certificate Authority (CA) design, Template customization, dynamic Group Policy Auto-Enrollment, and Revocation checks (CRL/OCSP).

---
## Concept
Think of a Public Key Infrastructure (PKI) as a government passport and ID department. 
- The **Root CA** is the main government office. Their word is absolute law, but they keep their main office sealed and offline to prevent break-ins (Offline Root).
- The **Subordinate CA** is the local passport office on your street. The main government signs their credentials, and they issue the actual IDs (Certificates) to citizens (Users/Computers) for daily use.
- A **Certificate** is a passport: it contains your photo, identity details, and a security hologram signed by the office. When you visit a website, you show your passport, and the browser checks the signature.
- The **CRL (Certificate Revocation List)** is the border control list of revoked/stolen passports that are no longer valid.

*Seedha simple mein: AD CS enterprise network mein digital certificates issue aur manage karne ke liye PKI setup hai. Root CA offline rakha jata hai aur Subordinate CAs dynamic certificates distribute karte hain auto-enrollment ke zariye.*

---
## Technical Deep Dive

### 1. PKI and CA Hierarchy Architecture
A standard enterprise Public Key Infrastructure (PKI) uses a two-tier CA hierarchy:
- **Tier 1: Root CA:** The trust anchor. It signs the certificate of the subordinate CA. 
  - **Best Practice:** Keep the Root CA **offline** (disconnected from the network, physically locked). This prevents external compromise. Its private key is only used to sign Subordinate CA certificates or CRLs.
- **Tier 2: Subordinate/Enterprise CA:** Connects to the network, integrates with Active Directory, and handles daily certificate requests (enrollments) for computers, users, and web servers.

### 2. Enterprise CA vs. Standalone CA
- **Enterprise CA:** Integrated with Active Directory. Can publish certificates and CRLs to AD, and supports automated enrollment based on certificate templates.
- **Standalone CA:** Independent of Active Directory. Does not use templates; all requests must be approved manually by a CA administrator. Used for offline Root CAs.

### 3. Certificate Templates
Templates define the properties, usage limits (e.g., Client Authentication, Code Signing, Encrypting File System), key length, and validity periods for certificates issued by an Enterprise CA.
- **Customizing Templates:** Standard templates cannot be modified directly. You must duplicate an existing template (e.g., duplicate the Web Server template to create a custom version with exportable private keys).

### 4. Auto-Enrollment
Auto-Enrollment is an Active Directory feature that allows client computers and users to dynamically request, enroll, and renew certificates in the background without user intervention. Configured via Group Policy.

### 5. Revocation Mechanisms
When a certificate is compromised (e.g., laptop stolen, private key leaked), it must be revoked before its expiration date.
- **CRL (Certificate Revocation List):** A signed file published periodically by the CA listing all revoked certificate serial numbers. Clients download the CRL (via HTTP or LDAP) and verify the file locally.
  - *Disadvantage:* High bandwidth usage if the list grows large.
- **OCSP (Online Certificate Status Protocol):** A real-time verification protocol. The client queries an OCSP responder targeting a specific certificate serial number. The responder checks the database and returns a small, immediate status answer: "Good", "Revoked", or "Unknown".

---
## Windows Server CLI Configuration Commands

### Installing AD CS Role Binaries
```powershell
# Install AD CS Certificate Authority role and management tools
Install-WindowsFeature -Name ADCS-Cert-Authority -IncludeManagementTools
```

### Configuring Enterprise Root CA (Post-Deployment Setup)
```powershell
# Install a new Enterprise Root CA (Run on target server)
Install-AdcsCertificationAuthority `
    -CAType EnterpriseRootCA `
    -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
    -HashAlgorithmName SHA256 `
    -KeyLength 2048 `
    -CommonName "company-RootCA" `
    -ValidityPeriod Years -ValidityPeriodUnits 10 `
    -Force
```

---
## Lab — Step by Step
> [!info] Lab Setup Needed
> A Domain Controller (`SVR-DC01` at `192.168.10.10`) running Windows Server 2022.

### Step 1: Install and Configure the CA
1. Open Server Manager -> Add Roles and Features.
2. Select **Active Directory Certificate Services**. Click Next.
3. Select **Certification Authority** role service. Click Install.
4. Once complete, click the notification flag and select **Configure Active Directory Certificate Services on the destination server**.
5. Credentials: Use Domain Admin. Click Next.
6. Role Services: Check **Certification Authority**. Click Next.
7. Setup Type: Select **Enterprise CA**. Click Next.
8. CA Type: Select **Root CA**. Click Next.
9. Private Key: Select **Create a new private key**. Click Next.
10. Cryptography: Hash: `SHA256`, Key Length: `2048`. Click Next.
11. Common Name: `company-SVR-DC01-CA`. Click Next.
12. Validity Period: Set to `5 Years`. Click Next.
13. Click **Configure**. Restart the server.

### Step 2: Configure Auto-Enrollment Template
1. Open Tools -> **Certification Authority**.
2. Right-click **Certificate Templates** and select **Manage**. The Templates console opens.
3. Locate **Workstation Authentication**. Right-click it and select **Duplicate Template**.
4. General Tab: Name it `Corp_Workstation_Auth`. Check **Publish certificate in Active Directory**.
5. Security Tab: Add **Domain Computers**. Check **Enroll** and **Autoenroll** permissions. Click Apply and OK. Close the console.
6. Go back to Certification Authority. Right-click **Certificate Templates** -> **New** -> **Certificate Template to Issue**.
7. Select `Corp_Workstation_Auth` and click OK.

### Step 3: Deploy Auto-Enrollment via GPO
1. Open **Group Policy Management** (`gpmc.msc`).
2. Edit the **Default Domain Policy** (or a custom workstation GPO).
3. Navigate to: `Computer Configuration` -> `Policies` -> `Windows Settings` -> `Security Settings` -> `Public Key Policies`.
4. Double-click **Certificate Services Client - Auto-Enrollment**.
5. Configuration:
   - Configuration Model: **Enabled**
   - Check: **Renew expired certificates, update pending certificates, and remove revoked certificates**.
   - Check: **Update certificates that use certificate templates**.
6. Click Apply and OK.
7. **Verify:** On a domain workstation, run `gpupdate /force` in CMD. Open `certlm.msc` (Local Computer Certificates). Expand Personal -> Certificates. Verify that a workstation certificate signed by `company-SVR-DC01-CA` is issued dynamically.

---
## Troubleshooting Scenarios

**Scenario 1:**
- **Problem:** Users receive SSL warnings when opening internal intranet sites: "This certificate is not trusted because it was issued by an untrusted Certificate Authority."
- **Root Cause:** The clients' operating systems do not have the Root CA's certificate installed in their "Trusted Root Certification Authorities" store.
- **Fix:**
  1. Export the Root CA's certificate as a `.cer` file from the CA server.
  2. Open Group Policy Management on the DC.
  3. Edit the Default Domain Policy. Navigate to:
     `Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies > Trusted Root Certification Authorities`.
  4. Right-click -> **Import**. Select the Root CA `.cer` file.
  5. Save and apply GPO. Workstations will import the root certificate dynamically, and SSL warnings will resolve.

**Scenario 2:**
- **Problem:** Attempting to enroll a new certificate fails with error: "The request was denied by the Active Directory Certificate Services. The certificate template could not be found."
- **Root Cause:** The target template has not been published for issuance on the CA, or the client lacks correct Read and Enroll permissions on the template security tab.
- **Fix:**
  1. Open the Certification Authority console. Expand the server node.
  2. Click **Certificate Templates**. Verify if the template is listed. If missing, right-click -> New -> Certificate Template to Issue, and add it.
  3. If listed, open the Templates Management console. Edit the target template -> **Security** tab.
  4. Verify that the requesting user/computer group has both **Read** and **Enroll** permissions checked.
  5. Re-run `certutil -pulse` on the client to retry enrollment.

---
## Common Mistakes
> [!warning] Avoid These
> **Setting Root CA validity periods shorter than Subordinate CAs:** Creating a Root CA with a 2-year validity, and attempting to issue a 3-year certificate from a subordinate CA. The subordinate CA cannot issue certificates whose lifespans exceed its own root authority expiration date.
> **Correct approach:** Root CAs should have long validity periods (e.g., 10-20 years), Subordinate CAs 5-10 years, and issued client certificates 1-2 years.

---
## Pro Tips
> [!tip] Field Experience
> When configuring CRL distribution points (CDP), always host the CRL on an external, anonymous HTTP web server rather than relying on LDAP paths. If a non-domain laptop or mobile device checks a certificate's revocation state, it cannot query the internal Active Directory LDAP directory, causing verification timeouts.

---
## Quick Revision Table
| # | Concept | One Line Summary |
|---|---------|-----------------|
| 1 | Two-Tier CA | Design utilizing an offline Root CA for core trust security, and a online Subordinate CA for issuance. |
| 2 | Auto-Enrollment | AD feature that automatically issues and renews user/computer certificates via Group Policy. |
| 3 | CRL | A published list of revoked certificate serial numbers that clients check locally. |
| 4 | OCSP | Real-time revocation checker protocol querying specific certificate status on-demand. |
| 5 | Enterprise CA | Active Directory integrated CA that uses templates and supports auto-enrollment. |

---
## Interview Q&A

**Q1: Why is it considered security best practice to keep a Root Certificate Authority (CA) offline?**
A: The Root CA is the foundation of trust for the entire corporate network. If an attacker compromises the Root CA's private key, they can generate valid certificates for any domain, user, or server, bypassing all SSL decryption controls and authenticating as administrators. Keeping the Root CA offline, disconnected from all networks, and locked in a physical vault prevents remote compromise. The subordinate CA is kept online to handle daily work, and if it is compromised, its certificate can be revoked by the offline Root CA without destroying the entire PKI root trust.

**Q2: A client's certificate auto-enrollment GPO is active, but workstations are not receiving certificates. Explain your troubleshooting process.**
A: 
- **Situation:** Auto-enrollment is enabled via GPO, but clients are not receiving certificates.
- **Task:** Diagnose the template settings and client authorization paths.
- **Action:** First, I will run `gpresult /h` on a client to verify the Public Key Policies GPO is applying. Second, I will check the certificate template permissions on the CA. The "Domain Computers" group must have explicit **Read**, **Enroll**, and **Autoenroll** permissions checked. Third, I will check the CA status to ensure the template is published under "Certificate Templates" to issue.
- **Result:** Adding the missing Autoenroll permission to the template security ACL allows clients to request and install certificates on their next GP refresh.

**Q3: Describe the difference between a certificate's Public Key and Private Key.**
A: Public and Private keys are mathematically linked under asymmetric cryptography. The **Public Key** is shared openly. Anyone can use it to encrypt data or verify a digital signature. However, data encrypted with the public key can *only* be decrypted using the corresponding private key. The **Private Key** must be kept secret by the owner. It is used to decrypt received files or generate digital signatures.

---
## Related Notes
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-02 Active Directory Domain Services|WS-02 Active Directory Domain Services]] — Storing PKI metadata in the directory.
- [[03-Identity-and-Core-Services/06-Active-Directory/WS-05 Group Policy — Complete Guide|WS-05 Group Policy — Complete Guide]] — Deploying auto-enrollment policy options.
- [[03-Identity-and-Core-Services/05-Windows-Server/WS-14 VPN and Remote Access (RRAS)|WS-14 VPN and Remote Access (RRAS)]] — Utilizing client certificates for SSTP VPN links.

