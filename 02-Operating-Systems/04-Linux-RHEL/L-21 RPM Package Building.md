---
tags: [linux, rhel, rpm, package-building, rpmbuild]
aliases: [RPM Build, RPM Packaging]
created: 2026-06-26
status: #complete
difficulty: #advanced
cert-relevant: #rhcsa
---

> [!NOTE|color-green]
> 🐧 **LINUX / RHEL**

`#complete` `#advanced` `#rhcsa`

# L-21: RPM Package Building

> [!abstract] Overview
> Jab humein enterprise environment mein custom software ya scripts ko multiple servers par deploy karna hota hai, toh raw files copy karne ke bajaye hum uska RPM package banate hain. Yeh note sikhata hai ki source code se apna khud ka `.rpm` package kaise build karein using `rpmbuild` and `.spec` files. Ek support engineer (especially L3) ke liye yeh bahut zaroori hai kyunki badi companies apne custom tools ko RPM format mein hi distribute karti hain via Satellite or custom YUM repositories. Package building standardizes software distribution.

---
## 🧠 Concept Overview

- **What it is** — RPM building is the process of taking raw source code (tarballs) and a recipe (SPEC file) to compile and package them into a standard `.rpm` file. 
- **Why it matters** — Manual installation (make, make install) tracking and uninstallation mushkil hoti hai. RPM ensures proper tracking, dependency resolution, version control, and clean uninstallation across the enterprise.
- **Where you see this** — Custom in-house applications, modified open-source tools with enterprise patches, or configuration packages that need to be distributed across 1000s of servers efficiently.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Identify if a package is custom built or from standard repo. Use `rpm -qi` to check vendor details. |
| **L2** | Troubleshoot installation failures of custom RPMs, collect logs, check missing dependencies. |
| **L3** | Write the `.spec` file, compile source code, build the `.rpm` package, and publish it to the enterprise YUM repository. |

> [!tip] Seedha Simple Mein
> *Source code ek kachha saaman (raw material) hai, SPEC file ek recipe hai, aur `rpmbuild` command ek factory hai jo in sabko use karke ek final packed product (RPM file) banati hai jise kahin bhi easily install kiya ja sakta hai.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Building an RPM** is like **creating an IKEA furniture box** because...
>
> - The **Source Code** is the raw wood and screws.
> - The **SPEC File** is the step-by-step instruction manual (recipe) that tells the factory how to assemble everything.
> - The **`rpmbuild` command** is the automated factory machine packing everything properly into the box.
> - The final **RPM file** is the neatly packed IKEA box that the end user can easily buy and assemble (install) using `yum install` without needing to cut the wood themselves.

---
## 🔬 Technical Deep Dive

### 1. The Build Environment (rpmbuild directories)

Jab hum RPM build karte hain, toh default `~/rpmbuild` directory use hoti hai. Isme 5 main folders hote hain:

> [!info] Key Concept
> - **BUILD**: Temporary directory jahan source code extract, compile aur build hota hai.
> - **RPMS**: Yahan final compiled binary RPM files (`.rpm`) save hoti hain, architecture ke subfolders mein (e.g., x86_64, noarch).
> - **SOURCES**: Original source code tarballs (`.tar.gz`) aur patch files yahan rakhi jaati hain.
> - **SPECS**: Isme `.spec` files hoti hain jo RPM banane ke poore instructions hold karti hain.
> - **SRPMS**: Yahan Source RPM files (`.src.rpm`) save hoti hain (isme source aur spec dono hote hain, but compiled binary nahi).

> [!danger] Common Mistake
> Kabhi bhi RPM ko `root` user se build mat karo! Hamesha ek normal standard user use karna chahiye RPM building ke liye. Agar script mein koi galti hui (jaise wrong paths in install section), toh `root` user poore OS ko corrupt kar sakta hai (like `rm -rf /`).

### 2. The SPEC File Breakdown

The `.spec` file is the heart of RPM building. Isme kai sections hote hain jo phases define karte hain:

- **Preamble**: Basic info like Name, Version, Release, Summary, License, Source0.
- `%description`: Package ka detailed description jo `rpm -qi` run karne par dikhta hai.
- `%prep`: Source code ko unpack/extract karta hai. (Usually uses `%setup -q` macro).
- `%build`: Code ko compile karta hai (Usually runs `./configure` and `make`).
- `%install`: Compiled files ko dummy `BUILDROOT` directory mein copy karta hai jahan se package banega (Usually runs `make install`).
- `%files`: Un sabhi files aur directories ki list jo RPM package ke andar aayengi. (Agar file yahan list nahi ki, toh package mein nahi jayegi aur error aayega).
- `%changelog`: Changes and version updates ka technical record.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - RHEL/CentOS/Rocky Linux machine
> - Standard user account (e.g., `student` ya `admin`, NEVER use root)
> - Internet connection for installing development tools

### Step 1: Install Required Packages

First, humein rpm building tools install karne honge.

```bash
# As root or using sudo
sudo yum install rpm-build rpmdevtools gcc make
```

> [!success] Expected Output
> `rpm-build`, `rpmdevtools` aur unke dependencies successfully install ho jayenge.

### Step 2: Create the Build Tree

Normal user se login karke, directory structure create karein.

```bash
# As standard user
rpmdev-setuptree

# Verify directories
ls ~/rpmbuild
```

> [!success] Expected Output
> ```
> BUILD  RPMS  SOURCES  SPECS  SRPMS
> ```

### Step 3: Download Source Code

Ek sample source code tarball download karke `SOURCES` mein rakhein. (Example: GNU `hello-2.10`)

```bash
cd ~/rpmbuild/SOURCES
wget https://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz
```

### Step 4: Create the SPEC File

`SPECS` directory mein jayen aur ek spec file banayein. `rpmdev-newspec` command se acha starting template mil jaata hai.

```bash
cd ~/rpmbuild/SPECS
rpmdev-newspec hello
vi hello.spec
```

*Edit the spec file to look exactly like this:*

```spec
Name:           hello
Version:        2.10
Release:        1%{?dist}
Summary:        The GNU Hello program
License:        GPLv3+
URL:            https://www.gnu.org/software/hello/
Source0:        https://ftp.gnu.org/gnu/hello/%{name}-%{version}.tar.gz

BuildRequires:  gcc, make

%description
The GNU Hello program produces a familiar, friendly greeting.
This is used as an example to learn RPM packaging.

%prep
%setup -q

%build
%configure
make %{?_smp_mflags}

%install
rm -rf $RPM_BUILD_ROOT
%make_install

%files
/usr/bin/hello
/usr/share/info/hello.info.gz
/usr/share/man/man1/hello.1.gz
%license COPYING
%doc AUTHORS ChangeLog NEWS README

%changelog
* Fri Jun 26 2026 Admin <admin@example.com> - 2.10-1
- Initial RPM release
```

### Step 5: Build the RPM Package

Ab hum `rpmbuild` command chalayenge.

```bash
rpmbuild -ba ~/rpmbuild/SPECS/hello.spec
```

> [!success] Expected Output
> Pura compile process screen par aayega, aur end mein file creation ki confirmation milegi:
> ```
> Wrote: /home/user/rpmbuild/SRPMS/hello-2.10-1.el9.src.rpm
> Wrote: /home/user/rpmbuild/RPMS/x86_64/hello-2.10-1.el9.x86_64.rpm
> Wrote: /home/user/rpmbuild/RPMS/x86_64/hello-debuginfo-2.10-1.el9.x86_64.rpm
> Wrote: /home/user/rpmbuild/RPMS/x86_64/hello-debugsource-2.10-1.el9.x86_64.rpm
> Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.xxxx
> ```

### Step 6: Verify and Install

Ab check karein ki RPM theek se ban gaya hai, aur usko install test karein.

```bash
# Check the package details (info)
rpm -qip ~/rpmbuild/RPMS/x86_64/hello-2.10-1.el9.x86_64.rpm

# Check the package contents (list files)
rpm -qlp ~/rpmbuild/RPMS/x86_64/hello-2.10-1.el9.x86_64.rpm

# Install it via yum
sudo yum install ~/rpmbuild/RPMS/x86_64/hello-2.10-1.el9.x86_64.rpm

# Run the command to test
hello
```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `rpmdev-setuptree` | `~/rpmbuild` ke 5 standard folders create karta hai | `rpmdev-setuptree` |
| `rpmdev-newspec pkg` | `.spec` file ka basic boilerplate template banata hai | `rpmdev-newspec nginx` |
| `rpmbuild -ba file.spec`| Binary (`.rpm`) and Source (`.src.rpm`) dono build karta hai | `rpmbuild -ba myapp.spec` |
| `rpmbuild -bb file.spec`| Sirf Binary RPM build karta hai | `rpmbuild -bb myapp.spec` |
| `rpmbuild -bs file.spec`| Sirf Source RPM build karta hai | `rpmbuild -bs myapp.spec` |
| `rpmlint file.spec` | Spec file ya rpm mein syntax/standard errors check karta hai | `rpmlint myapp.spec` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Installed (but unpackaged) file(s) found** | `%install` phase ne kuch files banayi hain jo `%files` section mein list nahi hain. | `.spec` ke `%files` section mein missing file paths add karo, ya unwanted files delete kardo. |
| **File not found by glob** | `%files` mein jo path diya hai, wo buildroot mein exist nahi karti. | Check `%install` section, ho sakta hai path galat copy ho raha ho ya `make install` fail ho gaya. |
| **Bad exit status from /var/tmp/rpm-tmp...** | `%prep`, `%build`, ya `%install` mein script fail ho gayi (jaise `./configure` error). | Logs ko carefully padho. Usually compiler ya header files (BuildRequires) missing hoti hain. |
| **No such file or directory: SOURCES/xxx.tar.gz** | Source code tarball file `SOURCES` directory mein present nahi hai ya naam match nahi kar raha. | `Source0:` mein jo tarball ka naam likha hai, exactly wahi file `SOURCES` folder mein download/copy karo. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Deploying a Custom Script to All Servers

> [!example] Ticket
> "Security team wants our custom Python compliance script deployed on 500 RHEL servers. They provided `check-sec.py`. Package it as an RPM so Satellite can distribute it."

**L1 Response:** Confirms the ticket details and forwards the script to the packaging/deployment team (L3).
**Escalation Trigger:** L1 cannot package applications, only deploy them once packaged.
**L2/L3 Resolution:**
1. Create a `check-sec` directory, copy script, and create `check-sec-1.0.tar.gz` in `SOURCES`.
2. Write a `.spec` file that copies `check-sec.py` to `/usr/local/bin/` during `%install`.
3. Add `%files` entry for `/usr/local/bin/check-sec.py`.
4. Run `rpmbuild -ba check-sec.spec`.
5. Upload the resulting RPM to the Satellite Server's Custom repository for mass deployment.

### 🎫 Scenario 2: Build Fails Due to Missing BuildRequires

> [!example] Ticket
> "Developer trying to build an RPM for their C++ tool on a new Jenkins build agent. The `rpmbuild` command fails with 'gcc-c++: command not found'."

**L1 Response:** Asks the user for the exact error log snippet.
**Escalation Trigger:** Needs to install compiler packages on the CI/CD build agent server.
**L2 Resolution:**
- Developer didn't realize the new build agent lacked standard compiler tools.
- Install the required tools: `sudo yum install gcc-c++`.
- *Pro-Tip:* Advise developer to add `BuildRequires: gcc-c++` in their `.spec` file so `rpmbuild` warns about this properly next time instead of failing mid-compile.

### 🎫 Scenario 3: Missing Shared Libraries on Target Machine

> [!example] Ticket
> "Our custom RPM installs fine, but when running the app, it throws 'error while loading shared libraries: libpq.so.5'."

**L1 Response:** Tries to reinstall the app, issue persists. Collects `ldd` output and escalates to L2/L3.
**Escalation Trigger:** The package installed successfully, but the application is broken due to undeclared runtime dependencies.
**L3 Resolution:**
- The RPM was built without defining runtime dependencies.
- Update the `.spec` file and add `Requires: postgresql-libs`.
- Rebuild the RPM and increment the `Release` version.
- Re-deploy. Now when `yum install custom-app` is run, YUM will automatically pull `postgresql-libs`.

---
## 🎤 Interview Questions

> [!question] Q1: What is the purpose of a `.spec` file in RPM building?
> **Answer:** A `.spec` file is the master recipe that contains all instructions for `rpmbuild`. It defines package metadata (name, version), how to unpack the source code (`%prep`), how to compile it (`%build`), how to install the compiled files into a temporary build root (`%install`), and which files should be included in the final RPM (`%files`).

> [!question] Q2: Why should you NEVER build an RPM as the `root` user?
> **Answer:** During the `%install` or `%clean` phases, if there is a typo in the directory paths (like `rm -rf / $RPM_BUILD_ROOT` instead of `$RPM_BUILD_ROOT/`), running as root would delete critical OS files and destroy the system. A standard user doesn't have privileges to break the OS outside of their home directory.

> [!question] Q3: What is the difference between `rpmbuild -ba` and `rpmbuild -bb`?
> **Answer:** `rpmbuild -ba` builds BOTH the binary package (`.rpm`) and the source package (`.src.rpm`). `rpmbuild -bb` builds ONLY the binary package.
> ==**Exam Tip:** Remember it simply: `-ba` means "Build All", `-bb` means "Build Binary".==

> [!question] Q4: How do you check for errors or policy violations in a newly built RPM or SPEC file?
> **Answer:** Use the `rpmlint` command. Running `rpmlint package.spec` or `rpmlint package.rpm` checks for common packaging errors, missing tags, or incorrect file permissions before you distribute the package to production.

> [!question] Q5: What is the difference between `BuildRequires` and `Requires` in a SPEC file?
> **Answer:** `BuildRequires` specifies packages needed ONLY during the compilation/build phase (like `gcc`, `make-devel`, `cmake`). `Requires` specifies packages needed to actually run the installed application on the end-user's machine (like `java`, `python3`, `httpd`).

---
## 🔗 Related Notes

- [[L-20 YUM and RPM Package Management]] — Managing pre-built packages.
- [[L-15 Software Installation and Make]] — Compiling from source without creating an RPM.
- [[A-05 Automation with Ansible]] — How to push these custom RPMs to multiple servers efficiently.
