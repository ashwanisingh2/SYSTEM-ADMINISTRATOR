---
tags: [desktop-support, devops, cicd, automation, L2]
aliases: [cicd-pipelines, dev-04]
created: 2026-06-25
status: #complete
difficulty: #intermediate
cert-relevant: #none
---

# DEV-04: CI/CD Pipelines

> [!note] Overview
> This note covers Continuous Integration and Continuous Deployment (CI/CD) pipelines, focusing on Jenkins, GitHub Actions, and Azure DevOps. It details runner architectures, trigger events, syntax configurations (YAML and Jenkinsfile), and best practices for automating application builds, tests, and deployments.

---
## Concept Overview
- **What it is** — CI/CD is a software development practice that automates the integration of code changes (build and test phases) and the subsequent deployment of the application to test, staging, or production environments.
- **Why it matters** — Manual deployments are slow, error-prone, and inconsistent. CI/CD pipelines ensure that every commit is automatically validated against automated test suites and built into deployable artifacts, lowering integration risks and speeding up releases.
- **Real job encounter** — Setting up self-hosted GitHub Actions runners, fixing failing Jenkins build jobs due to missing dependencies on build nodes, managing secrets in Azure DevOps Service Connections, and writing YAML pipeline files.
- **L1 vs L2 vs L3 responsibility split:**
  - **L1 Resolution**: Monitor pipeline execution dashboards (`SUCCESS`/`FAILED`), restart failed workflow jobs, check build logs for syntax errors, and confirm runner status.
  - **Escalation Trigger**: Escalate when runners are offline, credentials or secrets expire, or when pipeline failures are caused by network connectivity issues between runners and target environments.
  - **L2 Resolution**: Provision and maintain self-hosted runners or build agents, configure secrets and environment variables, and resolve build dependency errors on nodes.
  - **L3 Resolution**: Author complex multi-stage pipelines (YAML/Jenkinsfile) from scratch, configure parallel execution, integrate automated vulnerability scanners (SonarQube/Trivy), and manage secure deployment gates and rollbacks.

*Seedha simple mein: CI/CD pipeline ek automated factory system ki tarah hai. Jaise hi developer code commit karega, pipeline automatic code ko download karegi, compile karegi, test karegi, aur setup compile karke server pe deploy kar degi, bina kisi manual intervention ke.*

---
## Technical Deep Dive

### 1. CI/CD Architecture Components
- **Continuous Integration (CI)**: Focuses on automated build, test, and merge stages. Runs on every commit to a branch.
- **Continuous Delivery / Deployment (CD)**: Automatically deploys the tested build artifacts. *Delivery* requires manual approval to go to production; *Deployment* automatically pushes changes to production.
- **Pipelines**: The defined sequence of stages (e.g., Build -> Test -> Security Scan -> Deploy).
- **Runners / Agents**: The compute engines (virtual machines, containers, or physical servers) that download the code and execute the pipeline steps.

### 2. Jenkins (The Self-Hosted Master)
- **Controller-Agent Model**: The Jenkins Controller manages user access, schedules builds, and coordinates work, while Jenkins Agents (executors) run the actual build steps.
- **Jenkinsfile**: The configuration file placed in the repository root that defines the pipeline stages using Declarative or Scripted Groovy syntax.

### 3. GitHub Actions (The Cloud Native SaaS)
- **Workflows**: Defined in `.github/workflows/*.yml` files.
- **Jobs**: Sets of steps that run on the same runner. Jobs run in parallel by default but can be made sequential.
- **Actions**: Reusable components/plugins that perform common tasks (e.g., `actions/checkout@v4`).

### 4. Azure Pipelines (The Enterprise Orchestrator)
- **Agents**: Hosted by Microsoft or self-hosted on VMs/containers.
- **Service Connections**: Secure connectors linking Azure DevOps to external resources like cloud environments, Kubernetes clusters, or container registries without hardcoding secrets.

---
## Step-by-Step Lab

> [!warning] Pre-requisites
> - A local Linux server (Ubuntu or Rocky Linux) or local machine.
> - Git installed locally.
> - Docker installed locally (to run a Jenkins container).

### Step 1: Run Jenkins in Docker
We will run a local Jenkins instance inside a Docker container for testing.

1. Spin up Jenkins container mapping ports `8080` (UI) and `50000` (Agent communications):
```bash
docker run -d --name local-jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk17
```
2. Retrieve the initial administrator password:
```bash
docker logs local-jenkins | grep -A 5 "Please use the following password"
```
**Expected Output:** A 32-character hexadecimal password (e.g., `4a8f9c73...`).

3. Open a browser and navigate to `http://localhost:8080` to complete the initial setup (install recommended plugins and create admin user).

### Step 2: Create a Declarative Jenkins Pipeline
We will create a Jenkins Pipeline using a Jenkinsfile definition.

1. In the Jenkins Web UI, click **New Item**, name it `Test-Pipeline`, choose **Pipeline**, and click **OK**.
2. Scroll to the **Pipeline** section, set the Definition to **Pipeline script**, and paste the following Declarative Jenkinsfile:
```groovy
pipeline {
    agent any
    
    environment {
        APP_NAME = 'DevOpsApp'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git Repository...'
            }
        }
        stage('Build') {
            steps {
                echo "Compiling application: ${env.APP_NAME}"
                // Simulate build step
                sh 'echo "Build complete" > build_output.txt'
            }
        }
        stage('Test') {
            steps {
                echo 'Running Unit Tests...'
                sh 'grep "Build complete" build_output.txt'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to Staging Server...'
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Checking logs...'
        }
    }
}
```
3. Click **Save**, and then click **Build Now**.
4. Check **Console Output** to verify the stages.
**Expected Output:** All stages complete successfully with a `SUCCESS` status.

### Step 3: Write a GitHub Actions Workflow File
Here is a standard GitHub Actions YAML file template.

1. Create directory structure in a git repository:
```bash
mkdir -p .github/workflows
```
2. Create `.github/workflows/main.yml`:
```yaml
name: CI-CD-Test-Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js Runtime
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install Dependencies
        run: npm ci

      - name: Run Build
        run: npm run build --if-present

      - name: Run Automated Tests
        run: npm test --if-present
```

---
## Cheat Sheet / Quick Reference

| Tool | Concept | Syntax Example / CLI |
|---|---|---|
| **Jenkins** | Declarative Syntax | `pipeline { agent any ... stages { ... } }` |
| **Jenkins** | Scripted Syntax | `node { stage('Build') { sh 'make' } }` |
| **GitHub Actions** | Trigger on push | `on: push: branches: [ main ]` |
| **GitHub Actions** | Checkout Code | `- uses: actions/checkout@v4` |
| **GitHub Actions** | Run shell command | `- run: npm run build` |
| **Azure DevOps** | Define agent pool | `pool: vmImage: 'ubuntu-latest'` |
| **Docker** | Launch Jenkins | `docker run -d -p 8080:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts` |

---
## Troubleshooting Table

| Problem | Cause | Fix | Command |
|---|---|---|---|
| Jenkins builds fail with: `command not found` (e.g. `mvn` or `node`). | The executable binary is not installed on the Jenkins agent node or is missing from the environment PATH. | Install the package on the agent host, or specify the tool in the Jenkinsfile `tools { maven 'Maven3' }` block. | `sudo apt-get install -y maven` |
| GitHub Actions runs fail with: `Resource limit exceeded` or offline. | Self-hosted runner service is not active on the host machine. | Check service status on the runner host and start it. | `sudo ./svc.sh status` / `sudo ./svc.sh start` |
| Secrets fail to resolve in Azure DevOps pipelines. | The pipeline doesn't have permission to access the Variable Group or Library. | Open Azure DevOps Pipeline settings, navigate to Library, click the variable group, and authorize it. | N/A |
| Jenkins build hangs indefinitely in the checkout stage. | Host keys are not accepted or git credentials are misconfigured. | Use the Jenkins Credentials Provider to store SSH keys; check SSH connection manually from agent. | `ssh -T git@github.com` |
| GitHub Actions workflow fails on secrets reference. | Secret name is misspelled or not populated in Repository Secrets settings. | Check spelling in YAML: `${{ secrets.MY_SECRET }}` and verify key exists in Settings -> Secrets -> Actions. | N/A |

---
## Interview Questions

> [!question] L1 Question
> **Q:** What is the difference between Continuous Delivery and Continuous Deployment?
> **A:** In **Continuous Delivery**, the pipeline builds and tests code automatically and prepares a release-ready artifact. However, the actual deployment to production requires manual human approval. In **Continuous Deployment**, every code change that successfully passes the automated tests and pipeline stages is automatically and instantly deployed to the production environment without manual intervention.

> [!question] L2 Question
> **Q:** How do self-hosted runners communicate with GitHub/Azure DevOps servers? Does it require inbound firewall ports?
> **A:** No inbound ports are required. Self-hosted runners communicate using outbound polling over HTTPS (port 443). The runner machine initiates a persistent long-polling connection to the GitHub or Azure DevOps server to check for pending jobs. When a job is available, the runner pulls it, runs the tasks locally, and streams logs/status back via HTTPS.

> [!question] L3/Scenario Question
> **Q:** A developer commits a hotfix that bypasses build testing and deploys broken code to staging because the testing phase was too slow. How would you redesign the CI/CD pipeline to prevent this while keeping the feedback loop fast?
> **A:** 
> - **Situation:** Slow test execution leading to developers bypassing pipeline stages or hotfixing manually.
> - **Task:** Accelerate test times while enforcing mandatory quality gates.
> - **Action:** 
>   1. **Parallel Execution**: Split unit tests and integration tests. Run unit tests in parallel blocks using multiple runners.
>   2. **Pipeline Caching**: Cache dependencies (like node_modules or maven target folders) between builds using `actions/cache` or local cache volumes.
>   3. **Branch Protection Rules**: Configure branch protection in GitHub/Azure DevOps to require successful execution of the CI pipeline before merging or pulling code into production/staging branches.
>   4. **Conditional Stages**: Execute integration and security scans only on PR merges or release tags, keeping standard feature branch builds lightweight.
> - **Result:** Test execution is cut down, automated tests are structurally enforced, and deployment quality is preserved.

---
## Seedha Simple Mein
*Seedha simple mein: CI/CD pipelines software applications ki packaging aur deployments ko process automatically chalaney ka rasta hain. Developer ke code submit karte hi system automatically use test karke environment me deploy kar deta hai. Jenkins self-hosted server ke liye use hota hai, jabki GitHub Actions aur Azure DevOps cloud-based platforms hain jise YAML files se control kiya jata hai.*

---
## Related Notes
- [[05-Automation-and-Ticketing/14-DevOps-Basics/DEV-01 Docker-Fundamentals|DEV-01 Docker-Fundamentals]] — Containerizing applications for pipeline deployments.
- [[05-Automation-and-Ticketing/14-DevOps-Basics/DEV-03 Terraform-Basics|DEV-03 Terraform-Basics]] — Managing infrastructure in pipeline stages.
- [[05-Automation-and-Ticketing/12-Ansible/ANS-01 Ansible for Windows Admins|ANS-01 Ansible for Windows Admins]] — Configuration management tools.

---
*Tags: #desktop-support #devops #cicd #automation #L2*
