---
tags: [devops, ci-cd, automation, pipelines]
aliases: [ci-cd-basics, continuous-integration, continuous-deployment]
created: 2026-06-26
status: #complete
difficulty: #beginner
cert-relevant: #none
---

> [!NOTE|color-purple]
> 🤖 **DEVOPS BASICS**

`#complete` `#beginner` `#none`

# DEV-05: CI-CD Basics

> [!abstract] Overview
> *Continuous Integration aur Continuous Deployment (CI/CD) software development ka wo modern tarika hai jisme code likhne se lekar server par deploy hone tak ka process automated hota hai. Ek support engineer ya sysadmin ko yeh jaanna zaroori hai kyunki aajkal har nayi application CI/CD pipelines ke through hi build aur deploy hoti hai. Agar pipeline fail hoti hai, toh naya update users tak nahi pahonch pata. Isliye troubleshooting ke liye basics aana bahut zaroori hai.*

---
## 🧠 Concept Overview

- **What it is** — CI/CD is an automated process that compiles, tests, and deploys code every time a developer makes a change. It bridges the gap between development and operations activities and teams by enforcing automation in building, testing and deployment of applications.
- **Why it matters** — Manual deployments are slow, risky, and error-prone. CI/CD ensures faster, safer, and more reliable releases. It allows organizations to ship software faster and more frequently. *Pehle code manually copy-paste hota tha server par jisme human error hone ke chances zyada the, ab sab kuch automated scripts karti hain.*
- **Where you see this** — Every time developers push code to source code repositories like GitHub, GitLab, or BitBucket, an automation server (pipeline) automatically kicks in, builds the Docker image, runs tests, and deploys it to a target environment like Kubernetes or AWS EC2.

**L1 / L2 / L3 Split:**

| 👨‍💻 Level | 📋 Responsibility |
|---------|-----------------|
| **L1** | Basic task — pipeline fail hui ya paas hui yeh check karna aur console logs dekhna. *Sirf error message dekh ke developer ya senior ko batana ki kaunsa step fail hua hai.* |
| **L2** | Configure, fix, escalate — pipeline ki configuration variables update karna, credential/token issues fix karna, workspace clean karna ya minor script errors theek karna. |
| **L3** | Architecture, design, enterprise-level — naye CI/CD tools (like Jenkins, GitLab CI) setup karna, secure runners deploy karna, aur complex deployment strategies (Blue-Green, Canary) design aur implement karna. |

> [!tip] Seedha Simple Mein
> *Jaise car manufacturing factory mein ek assembly line hoti hai jahan parts apne aap judte hain, paint hote hain, aur test hote hain, waise hi software ki assembly line ko CI/CD bolte hain. Code daalo, software nikal aayega.*

---
## 💡 Real-World Analogy

> [!info] Think of it like this...
> **Restaurant Kitchen workflow** is like a **CI/CD Pipeline** because...
>
> - **Continuous Integration (CI):** Jaise hi customer order deta hai (developer pushes code), chef turant ingredients mix karta hai, stove pe banata hai aur taste karta hai ki namak theek hai ya nahi (code is compiled and automated tests are run to verify quality).
> - **Continuous Delivery:** Dish bankar pass plate par ready rakhi hai. Waiter ka wait ho raha hai uthane ke liye. Uske bina dish table tak nahi jayegi (manual approval required for production release).
> - **Continuous Deployment:** Agar dish sahi bani hai aur restaurant mein conveyor belt system hai, toh bina kisi waiter ke sidha customer ke table par chali jati hai (code is automatically deployed to the production server with zero human touch).

---
## 🔬 Technical Deep Dive

### 1. Continuous Integration (CI)

> [!info] Key Concept
> CI is the practice of automating the integration of code changes from multiple contributors into a single software project frequently. 

CI ka main maksad bugs ko jaldi pakadna hai. *Jaise hi developer apna code master branch mein daalta hai, pipeline chalu hoti hai aur verify karti hai ki naye code ne purane chalu code ko toh break nahi kiya.*
Components involved in CI:
- **Version Control System (VCS):** Git, SVN. The source of truth where developers collaborate.
- **Build Server:** Jenkins, GitLab CI. The engine that triggers and runs tasks.
- **Build Tools:** Maven, Gradle, MSBuild, npm. Compiles source code into an executable artifact.
- **Unit Testing Frameworks:** JUnit, PyTest, Jest. Ensures code logic is sound before moving forward.

> [!danger] Common Mistake
> Developers bypassing automated tests and forcing code merges (e.g. using `git push --force` or disabling test jobs). *Bina test pass hue code merge karne se aage ka process aur production server dono down ho sakte hain.*

### 2. Continuous Delivery vs Continuous Deployment (CD)

> [!info] Key Concept
> CD refers to automating the release process. **Delivery** means it's ready to deploy (needs manual click). **Deployment** means it goes live automatically.

- **Continuous Delivery:** Code is built, tested, packaged, and pushed to a non-production environment (like staging or UAT). Uske baad ek release manager ya IT lead ka manual approval chahiye hota hai production mein dalne ke liye. *Matlab khana bankar ready hai, par manager check karega phir serve hoga.*
- **Continuous Deployment:** Agar saare automated tests pass hote hain, toh bina kisi human intervention ya button press ke code directly live production servers mein chala jata hai. *Matlab code push karte hi kuch minute mein users naya feature use kar sakte hain.*

### 3. Popular CI/CD Tools & Platforms

- **Jenkins:** Sabse purana aur popular open-source automation server. Plugins par chalta hai aur highly customizable hai.
- **GitLab CI/CD:** GitLab ke andar hi inbuilt hota hai. Ek `gitlab-ci.yml` file se chalta hai. Bohot seamless developer experience deta hai.
- **GitHub Actions:** GitHub repo ke sath directly pipelines banane ke liye use hota hai. Modern, light-weight aur open-source projects mein sabse zyada use hone laga hai.
- **Azure DevOps:** Microsoft environment aur enterprise clients ke liye best tool. Boards, Repos, Pipelines sab ek jagah integrate hote hain.
- **CircleCI / TravisCI:** Cloud-based platforms majorly used by open source projects and small startups.

---
## 🛠️ Step-by-Step Lab

> [!warning] Pre-requisites
> - A basic understanding of Linux terminal commands.
> - Git installed on your local system.
> - A free GitHub account and a basic code repository.

### Step 1: Create a basic GitHub Actions workflow file

Create a directory structure `.github/workflows` in your Git repository and create a file named `ci.yml`. *GitHub automatically `.github/workflows` folder ko check karta hai aur wahan rakhi `.yml` files ko pipeline maan kar execute karta hai.*

```yaml
# Step 1: Define workflow name
name: Simple CI Pipeline

# Step 2: Trigger on push to main branch
on:
  push:
    branches:
      - main

# Step 3: Define jobs and execution environment
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    # Step 4: Define individual steps
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Run a simple echo script
        run: echo "Hello, CI/CD Pipeline is running successfully!"
        
      - name: List files in workspace
        run: ls -la
        
      - name: Run a mock test
        run: echo "All tests passed successfully!"
```

### Step 2: Commit and Push

Push the changes to your GitHub repository.

```bash
git add .github/workflows/ci.yml
git commit -m "Add basic CI pipeline"
git push origin main
```

> [!success] Expected Output
> You can view the output in the "Actions" tab of your GitHub repository. It will show a green checkmark indicating success.
> ```text
> Hello, CI/CD Pipeline is running successfully!
> total 12
> drwxr-xr-x 3 runner docker 4096 Jun 26 10:00 .
> drwxr-xr-x 3 runner docker 4096 Jun 26 10:00 ..
> drwxr-xr-x 2 runner docker 4096 Jun 26 10:00 .github
> -rw-r--r-- 1 runner docker  235 Jun 26 10:00 README.md
> All tests passed successfully!
> ```

---
## ⌨️ Command Cheat Sheet

| ⌨️ Command | 🛠️ Kya karta hai | 📝 Example |
|-----------|-----------------|-----------|
| `git push` | Pushes local code to the remote repository which usually triggers the CI pipeline. *Code ko server par bhejta hai taaki pipeline trigger ho.* | `git push origin main` |
| `docker build` | Code aur dependencies ko ek container image mein pack karta hai. CI process ka ek bohot common step. | `docker build -t myapp:v1 .` |
| `docker push` | Banayi hui image ko Docker registry (like DockerHub ya AWS ECR) mein store karta hai. | `docker push myrepo/myapp:v1` |
| `kubectl apply` | Kubernetes cluster mein naya container deploy karta hai. CD process ka end stage jahan code live hota hai. | `kubectl apply -f deploy.yaml` |
| `npm test` | Node.js applications ke liye automated test cases run karta hai to ensure code logic. | `npm test` |
| `mvn clean install` | Java Maven project ko compile karta hai, test run karta hai aur `.jar`/`.war` build banata hai. | `mvn clean install` |

---
## 🚑 Troubleshooting Guide

| ⚠️ Problem | 🔍 Wajah (Cause) | 🛠️ Fix |
|-----------|----------------|-------|
| **Pipeline failed on 'Checkout code'** | Git repository clone nahi ho paayi. Authentication issue ya branch missing ho sakti hai. *GitHub / GitLab token expire ho gaya hoga ya branch delete ho gayi hogi.* | Check credentials and tokens in CI/CD settings. Verify if the target branch actually exists. |
| **Build failing due to missing dependencies** | `package.json` ya `pom.xml` mein naya dependency add hua hai but remote repository mein code push karte waqt sync nahi hua. | Run `npm install` locally, commit `package-lock.json` as well. Check pipeline logs to see which exact package is missing. |
| **Pipeline stuck in 'Pending' or 'Queued' state** | Jenkins node ya GitHub runner available nahi hai. *Saare server agents pehle se chalu jobs ke wajah se busy hain aur nai pipeline apni bari ka wait kar rahi hai.* | Check runner/agent status. Add more nodes/runners or cancel stuck/idle jobs to free up compute capacity. |
| **Deployment step fails with "Unauthorized"** | Pipeline service account ke paas target server (Kubernetes/AWS) pe deploy karne ke permissions nahi hain. | Grant proper RBAC roles or IAM permissions to the CI/CD service account/user being used by the pipeline. |
| **Deployment successful but application is down/crashing** | App code successfully server par chala gaya, par application run hote waqt crash ho gayi (e.g., database connection error, missing env vars). | SSH into server and check application logs (`docker logs <container-id>` or `journalctl -u app`). Revert to the previous stable build immediately. |

---
## 🎫 Real-World Ticket Scenarios

### 🎫 Scenario 1: Developer reports pipeline is failing randomly on test stage

> [!example] Ticket
> "Our Node.js build pipeline fails intermittently on the 'npm install' step. Sometimes it works fine, but sometimes it gives an ETIMEDOUT error."

**L1 Response:** Check pipeline console logs to confirm the exact error message and the time it occurs. *Developer ko batao ki yeh code ka issue nahi lag raha, balki server se download karte waqt network/timeout issue lag raha hai.*
**Escalation Trigger:** Agar har dusri pipeline fail hone lage, development ruk jaye aur L1 ke paas network fix karne ki server access na ho.
**L2 Resolution:** Check the network connectivity and DNS of the CI runner to the public npm registry. Might need to configure a private mirror (like Nexus or Artifactory) or enable registry caching to avoid internet timeout issues. *NPM registry ka cache setup karke resolve kiya gaya.*

### 🎫 Scenario 2: Deployment failed due to invalid credentials

> [!example] Ticket
> "CD pipeline failed to push the newly built Docker image to AWS ECR. The logs say: 'No basic auth credentials' or 'Access Denied'."

**L1 Response:** Identify which exact pipeline step failed. Verify if AWS credentials or IAM policies were changed recently by the security team. *Logs se error copy karke IAM/Security Admin team se pucho agar credential/password rotate hue hain pichle kuch dino mein.*
**Escalation Trigger:** Passed to L2/L3 if credentials need to be updated in the CI/CD secret manager securely and requires admin rights.
**L2 Resolution:** Generate new AWS Access Key and Secret Key for the CI/CD IAM user. Update these values securely in GitLab CI / Jenkins credentials manager or GitHub Secrets. Re-run the pipeline to verify deployment.

### 🎫 Scenario 3: Stale runners causing disk space full

> [!example] Ticket
> "Jenkins job fails immediately with 'java.io.IOException: No space left on device' error on the worker node."

**L1 Response:** SSH into the Jenkins worker node (agent) and run `df -h` to check disk space. *Worker node ka root ya workspace partition 100% full dikh raha hai jiski wajah se naya code download nahi ho pa raha.*
**Escalation Trigger:** If clearing space requires deleting old docker images, containers, or workspace directories safely without impacting currently running production jobs.
**L2 Resolution:** Run `docker system prune -af --volumes` to remove unused container images and volumes safely. Delete old build workspaces from `/var/jenkins_home/workspace`. Configure a Jenkins cron job to automatically clean up old workspaces weekly.

---
## 🎤 Interview Questions

> [!question] Q1: What is the main difference between Continuous Delivery and Continuous Deployment?
> **Answer:** In Continuous Delivery, the code is tested and ready to be deployed to production, but it requires a **manual approval** (human intervention) to push it live. In Continuous Deployment, the code goes to production **automatically** without any manual clicks if all automated tests pass.
> ==**Exam Tip:** Remember "Deployment = Fully automated end-to-end, Delivery = Manual release step at the end."==

> [!question] Q2: How do you secure credentials (like API keys, passwords) in a CI/CD pipeline?
> **Answer:** We never hardcode credentials or secrets in the plain text code repository (`git`). We store them securely using dedicated secret management tools like Jenkins Credentials Store, GitHub Secrets, GitLab CI/CD Variables (masked), or HashiCorp Vault. These are then injected into the pipeline as hidden environment variables during execution. *Password hamesha 'Secrets' section mein hide karke rakhte hain taaki logs mein na dikhe.*

> [!question] Q3: What is a 'Runner', 'Agent', or 'Worker' in the context of CI/CD?
> **Answer:** A runner/agent is a dedicated compute machine (virtual machine, Docker container, or physical server) that actually executes the commands and scripts defined in the pipeline. The main CI server (like Jenkins master) just manages the UI, triggers, and schedules the jobs, while the agents do the heavy lifting of building and testing.

> [!question] Q4: What happens if a build or test fails in the CI stage?
> **Answer:** The pipeline immediately halts and marks the build as 'Failed'. Notifications are sent out (via Email, Slack, or MS Teams) to the team/developer who triggered it. No further steps, such as deployment or publishing artifacts, will be executed. The developer must fix the bug locally, commit, and push the updated code to trigger a new fresh pipeline run. *Pehle code ka bug theek hoga, tabhi pipeline aage ka process start karegi.*

---
## 🔗 Related Notes

- [[DEV-01 Introduction to DevOps|Introduction to DevOps]] — Learn the basic DevOps culture and principles.
- [[LINUX-05 Bash Scripting Basics|Bash Scripting Basics]] — Necessary for writing custom shell scripts inside pipelines.
- [[CNT-01 Docker Basics|Docker Basics]] — Containerizing applications is a core part of modern CI pipelines.
- [[GIT-01 Git Basics|Git Basics]] — The version control system that triggers the CI/CD workflows.
