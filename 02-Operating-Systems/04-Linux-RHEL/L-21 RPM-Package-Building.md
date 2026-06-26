---
tags: [desktop-support, linux-rhel, package-management, rpmbuild, L2]
aliases: [rpm-package-building, custom-rpm]
created: 2026-06-25
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

# L-21: RPM Package Building

> [!note] Overview
> This note covers Red Hat Package Manager (RPM) development and package building for enterprise RHEL/Rocky environments. It details workspace configuration, SPEC file authoring, dependency definitions, packaging compilation commands, and repository creation.

---
## Concept Overview
- **What it is** — RPM Package Building is the process of packaging application binaries, scripts, dependencies, configuration files, and installation metadata into a single, standardized `.rpm` installer. The process utilizes the `rpmbuild` engine guided by a `.spec` definition file.
- **Why it matters** — Installing applications manually from source code (`make install`) across 500+ production servers leads to untracked configurations, dependency conflicts, and uninstaller failures. Custom RPMs allow you to cleanly install, update, and audit applications using the YUM/DNF packaging tools.
- **Real job encounter** — Distributing a custom local security auditing script to all corporate RHEL hosts, updating internal applications, and creating signed package patches for offline servers.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Install internal RPM packages using `dnf install`, query installed files using `rpm -ql`, and verify package validation logs.
  - **Escalation Trigger**: Escalate if package installation fails due to dependency errors, or if custom RPM signatures fail local security validations.
  - **L2 Resolution**: Configure the build directory tree, write functional `.spec` files, define requirements, compile packages using `rpmbuild`, and sign packages with GPG.
  - **L3 Resolution**: Setup central local YUM/DNF software repositories using `createrepo`, configure GPG key rings, and automate packaging builds using CI/CD pipelines (Jenkins/GitLab CI).

---
## Technical Deep Dive

### 1. The Build Directory Structure (`~/rpmbuild/`)
The tool `rpmbuild` requires a strict folder layout under the user's home directory:
- **`SOURCES`**: Contains the raw application source files (tarballs, patches, configuration scripts).
- **`SPECS`**: Contains the `.spec` file, which is the recipe describing how the package is compiled and installed.
- **`BUILD`**: The temporary workspace directory where the tarballs are extracted and compiled.
- **`BUILDROOT`**: A mock target installation filesystem mimicking the target server root `/` (e.g. `/usr/bin`, `/etc`).
- **`RPMS`**: The destination directory where the completed `.rpm` files are saved.
- **`SRPMS`**: The destination directory containing source RPMs (`.src.rpm`), which are used to rebuild the package.

```
                  +--------------------------------+
                  |           ~/rpmbuild           |
                  +--------------------------------+
                                  |
         +-------+-------+--------+-------+-------+
         |       |       |                |       |
         v       v       v                v       v
     [SOURCES] [SPECS] [BUILD]        [RPMS]   [SRPMS]
         |       |       |                |
     Source     SPEC    Tmp Workspace  Finished
     Tarballs   File    (Compilation)  RPMs
```

### 2. Anatomy of a SPEC File
The SPEC file is divided into metadata blocks and scripts:
- **Preamble**: Defines package metadata:
  - `Name`: The package filename identifier (e.g., `sys-audit`).
  - `Version`: The software release version (e.g., `1.0.0`).
  - `Release`: The package build iteration (e.g., `1`).
  - `Requires`: Runtime software dependencies required on the target machine.
- **`%prep`**: Contains shell commands to clean up build paths and extract tarball sources.
- **`%build`**: Contains build commands (like running `./configure` and `make`) to compile binaries.
- **`%install`**: Copies compiled binaries from `%build` to the mock `BUILDROOT` directory structure.
- **`%files`**: Lists all files and permissions that must be included inside the final RPM file.
- **`%changelog`**: Version histories.

---

### Enterprise RHEL Service & Network Configurations

#### 1. Custom Systemd Service Creation
Create a custom systemd service configuration file `/etc/systemd/system/myapp.service`:
```ini
[Unit]
Description=My Custom Enterprise Application
After=network.target

[Service]
Type=simple
User=sysadmin
ExecStart=/usr/bin/python3 /opt/myapp/server.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
```

#### 2. Log Rotation & Rsyslog Configuration
Configure log rotation rule in `/etc/logrotate.d/myapp` for automatic log cleaning:
```text
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0660 sysadmin sysadmin
}
```
Define Rsyslog rule in `/etc/rsyslog.d/50-myapp.conf` to redirect application logs to a dedicated file:
```text
if $programname == 'myapp' then /var/log/myapp/syslog.log
& stop
```
Restart Rsyslog service:
```bash
sudo systemctl restart rsyslog
```

#### 3. Network Bonding & Teaming (LACP Link Aggregation)
Create a network team interface config file `/etc/sysconfig/network-scripts/ifcfg-team0` (RHEL standard):
```text
DEVICE=team0
DEVICETYPE=Team
BOOTPROTO=none
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
ONBOOT=yes
TEAM_CONFIG='{"runner": {"name": "lacp"}}'
```
Bind slave physical interfaces (e.g., `eth1`) to the team interface:
```text
# /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
ONBOOT=yes
TEAM_MASTER=team0
DEVICETYPE=TeamPort
```

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A Rocky Linux / RHEL 9 VM.
> - Root or sudo access.
> - Outbound internet connectivity to download build utilities.

### Step 1: Install Development Tools and Create Workspace
1. Install RPM creation utilities:
```bash
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y rpm-build rpmdevtools
```
2. Generate the standard RPM build directories:
```bash
rpmdev-setuptree
```
3. Verify directory creation:
```bash
ls -F ~/rpmbuild
```
**Expected Output:** `BUILD/  BUILDROOT/  RPMS/  SOURCES/  SPECS/  SRPMS/`

### Step 2: Create Source Script and Archive
We will create a simple system audit script to package.

1. Create script folder:
```bash
mkdir -p ~/rpmbuild/SOURCES/sysaudit-1.0
```
2. Write script content:
```bash
cat << 'EOF' > ~/rpmbuild/SOURCES/sysaudit-1.0/sysaudit.sh
#!/bin/bash
echo "=== SYSTEM PERFORMANCE AUDIT ==="
echo "Date: $(date)"
echo "Uptime: $(uptime -p)"
echo "Disk Usage:"
df -h /
EOF
```
3. Make script executable and compress it:
```bash
chmod +x ~/rpmbuild/SOURCES/sysaudit-1.0/sysaudit.sh
tar -czf ~/rpmbuild/SOURCES/sysaudit-1.0.tar.gz -C ~/rpmbuild/SOURCES sysaudit-1.0
```

### Step 3: Author the SPEC File
1. Create and edit a spec file under `~/rpmbuild/SPECS/`:
```bash
vi ~/rpmbuild/SPECS/sysaudit.spec
```
2. Add the following SPEC layout:
```text
Name:           sysaudit
Version:        1.0
Release:        1%{?dist}
Summary:        A simple local system audit tool
License:        GPL
Source0:        sysaudit-1.0.tar.gz
BuildArch:      noarch
Requires:       bash

%description
This package installs a simple system performance audit script.

%prep
%setup -q

%build
# No compilation required for bash script

%install
rm -rf $RPM_BUILD_ROOT
mkdir -p $RPM_BUILD_ROOT/usr/local/bin
cp sysaudit.sh $RPM_BUILD_ROOT/usr/local/bin/sysaudit

%files
/usr/local/bin/sysaudit

%changelog
* Thu Jun 25 2026 Ashwani Singh <ashwani@corp.local> - 1.0-1
- Initial release of sysaudit script.
```
3. Save the file and exit.

### Step 4: Compile and Build the RPM
1. Run the build command:
```bash
rpmbuild -ba ~/rpmbuild/SPECS/sysaudit.spec
```
**Expected Output:** Builds binary and source packages. Output ends with:
`Wrote: /home/sysadmin/rpmbuild/RPMS/noarch/sysaudit-1.0-1.el9.noarch.rpm`
`Exit status: 0`

### Step 5: Test Custom Package Installation
1. Install your compiled RPM:
```bash
sudo dnf install -y ~/rpmbuild/RPMS/noarch/sysaudit-1.0-1.el9.noarch.rpm
```
**Expected Output:** Package installed successfully.
2. Run the installed tool:
```bash
sysaudit
```
**Expected Output:** `=== SYSTEM PERFORMANCE AUDIT ===` output.

3. Verify package tracking inside the DNF database:
```bash
rpm -qi sysaudit
```

---
## Cheat Sheet / Quick Reference

| Command / Setting | Purpose | Example |
|---|---|---|
| `rpmdev-setuptree` | Initializes standard build directory structure | `rpmdev-setuptree` |
| `rpmbuild -ba <spec>` | Compiles both binary (.rpm) and source (.src.rpm) files | `rpmbuild -ba sysaudit.spec` |
| `rpmbuild -bb <spec>` | Compiles binary RPM package only | `rpmbuild -bb sysaudit.spec` |
| `rpm -qpi <file>.rpm` | Queries metadata inside an uninstalled RPM file | `rpm -qpi sysaudit.rpm` |
| `rpm -qpl <file>.rpm` | Lists target install file paths in uninstalled RPM | `rpm -qpl sysaudit.rpm` |
| `rpm -ql <name>` | Lists all files created by an installed package | `rpm -ql sysaudit` |
| `rpm -qf <file_path>` | Identifies which package installed a specific file | `rpm -qf /usr/local/bin/sysaudit` |
| `createrepo <dir>` | Creates metadata databases to host custom repositories | `createrepo /var/www/html/repo/` |

---
## Troubleshooting Table

| Problem | Cause | Fix |
|---|---|---|
| Compilation fails: "BuildDependency: missing..." | The package compiling the SPEC requires helper tools that are not installed. | Install the missing dependencies listed under the `BuildRequires` section of the SPEC file. |
| Build fails at `%install` step: "No such file or directory." | The files were not copied correctly from the compilation folder to the mock `BUILDROOT` path. | Check `%install` paths. Ensure all target folders are created in `$RPM_BUILD_ROOT` (using `mkdir -p`) before executing `cp` or `install` commands. |
| Build fails: "Installed (but unpackaged) file(s) found." | Files were copied into `$RPM_BUILD_ROOT` during `%install`, but not listed under `%files` block. | Add the exact path of the unlisted files to the `%files` section of the SPEC file. |
| Build fails when running as Root. | Writing files as root can overwrite system binaries if paths are misconfigured. | **Never build RPMs as root**. Set up the `rpmdev-setuptree` workspace and run compilation processes under a standard non-privileged user account. |
| Yum/Dnf fails to install custom package: "Package signature failed." | System is configured to block unsigned RPM files. | Sign the package with a local GPG key: `rpm --addsign package.rpm`, or install using the `--nogpgcheck` switch (temporary testing only). |

---
## Interview Questions

> [!question] L1 Question
> **Q:** How do you find out which RPM package installed a specific config file (like `/etc/httpd/conf/httpd.conf`) on a RHEL server?
> **A:** I can use the rpm query file command: `rpm -qf /etc/httpd/conf/httpd.conf`. This checks the local packaging database and returns the exact package name and version that installed the file.

> [!question] L2 Question
> **Q:** What is the purpose of the `%files` section inside an RPM SPEC file, and what happens if you forget to include a file there?
> **A:** The `%files` section lists every file and directory that must be packed into the final `.rpm` archive. If a file is copied to the `$RPM_BUILD_ROOT` during the `%install` phase but is not listed in the `%files` section, the `rpmbuild` tool will abort the compilation process and return an "Installed (but unpackaged) file(s) found" error.

> [!question] L3/Scenario Question
> **Q:** You need to distribute a proprietary internal application package to 100 Linux servers securely. How do you set up this packaging and distribution pipeline using native tools?
> **A:** 
> - **Situation:** Securing software package distribution to 100 production nodes.
> - **Task:** Design a secure packaging, signing, and repository hosting workflow.
> - **Action:** 
>   1. **Set Up Build**: Configure `rpmbuild` on a staging node. Author a SPEC file containing dependencies, compile the binary RPM, and create a GPG key: `gpg --gen-key`.
>   2. **Sign Package**: Configure `~/.rpmmacros` to point to the GPG key, and sign the package: `rpm --addsign myapp-1.0.rpm`.
>   3. **Host Repository**: Copy the signed RPM to a secure internal web server directory (e.g. `/var/www/html/repo/`) and generate repository metadata database: `createrepo /var/www/html/repo/`. Export public GPG key to the root of the web directory.
>   4. **Target Node Configuration**: Deploy a custom repository file `/etc/yum.repos.d/internal.repo` to all target hosts:
>      ```ini
>      [internal-repo]
>      name=Internal Application Repo
>      baseurl=http://repo.corp.local/repo/
>      gpgcheck=1
>      gpgkey=http://repo.corp.local/RPM-GPG-KEY-internal
>      ```
> - **Result:** Target nodes fetch, verify signature, and install the package securely via normal `dnf install` processes.

---
## Seedha Simple Mein
*Seedha simple mein: RPM building ka matlab hai custom softwares, scripts ya config files ko ek single executable installer package (.rpm file) me convert karna. `rpmbuild` command aur `.spec` file ka use karke hum application ke prerequisites aur installation paths compile karte hain taaki applications ko standard YUM/DNF repositories ke zariye distribute kiya ja sake.*

---
## Related Notes
- [[02-Operating-Systems/04-Linux-RHEL/L-10 Software Management — YUM DNF|L-10 Software Management — YUM DNF]] — Installing and managing packages.
- [[02-Operating-Systems/04-Linux-RHEL/L-03 File System Management|L-03 File System Management]] — Standard system directory structures.
- [[05-Automation-and-Ticketing/12-Ansible/ANS-02 Ansible for Linux Admins|ANS-02 Ansible for Linux Admins]] — Automating package configuration files distribution.

---
*Tags: #desktop-support #linux-rhel #package-management #rpmbuild #L2 #cert-rhcsa*
