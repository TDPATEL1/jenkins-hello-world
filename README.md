# GitHub → Jenkins → JFrog Artifactory Automation

## 1. Project Overview

This project implements an automated CI/CD pipeline using GitHub, Jenkins, Docker, ngrok, and JFrog Artifactory.

Whenever code is pushed to the GitHub repository, Jenkins detects the change through a GitHub webhook, executes the build pipeline, compiles the C program, and uploads the generated build artifact to JFrog Artifactory.

The project also includes email notifications and Docker container monitoring.

### Objectives

- Automate builds triggered by GitHub code changes.
- Run Jenkins inside Docker.
- Provide external webhook connectivity using ngrok.
- Compile C programs automatically using GCC.
- Store build artifacts in JFrog Artifactory.
- Generate timestamp-based artifact filenames.
- Organize artifacts by branch and pull request.
- Monitor Docker container availability.
- Send email notifications for pipeline and container events.

---

## 2. System Architecture

```text
             Developer
                 |
                 | Git Push
                 v
          GitHub Repository
                 |
                 | Webhook
                 v
              ngrok
                 |
                 | HTTPS Forwarding
                 v
          Jenkins (Docker)
                 |
                 | Checkout Source
                 v
          GCC Compilation
                 |
                 | Generate Artifact
                 v
          Timestamped Artifact
                 |
                 | Upload
                 v
          JFrog Artifactory
                 |
                 v
          Artifact Repository
          jenkins-artifacts

Additional Components:

Jenkins --------> Email Notifications
Docker Monitor -> Email Alerts
Artifactory ----> PostgreSQL Database
```

---

## 3. Technology Stack

| Technology | Purpose |
|---|---|
| Windows 11 | Host operating system |
| WSL2 | Linux development environment |
| Ubuntu 24.04 | Linux environment |
| Docker Desktop | Container management |
| Jenkins | CI/CD automation |
| GitHub | Source code management |
| ngrok | External webhook connectivity |
| GCC | C program compilation |
| JFrog Artifactory OSS | Artifact management |
| PostgreSQL | Artifactory database |
| PowerShell | Docker monitoring |
| Gmail SMTP | Email notifications |

---

## 4. Project Environment

### Operating System

- Host: Windows
- Linux environment: Ubuntu 24.04 through WSL2
- Linux username: `developer`
- Docker Desktop: WSL2 backend

### Project Locations

| Component | Location |
|---|---|
| JFrog installation | `D:\jfrog-artifactory\artifactory-oss-7.161.15` |
| Jenkins backup | `C:\jenkins-custom\jenkins_home_backup.tar.gz` |
| Docker monitoring | `C:\Users\admin\docker-monitor` |
| GitHub project | `TDPATEL1/jenkins-hello-world` |

---

## 5. Docker Container Configuration

The environment uses three primary Docker containers.

| Container | Image | Ports |
|---|---|---|
| Jenkins | `jenkins-gcc:latest` | 8080, 50000 |
| Artifactory | `releases-docker.jfrog.io/jfrog/artifactory-oss:7.161.15` | 8081, 8082 |
| PostgreSQL | `releases-docker.jfrog.io/postgres:17.10-alpine` | 5432 |

### Local Access URLs

| Service | URL |
|---|---|
| Jenkins | http://localhost:8080 |
| JFrog UI | http://localhost:8082 |
| JFrog API | http://localhost:8081 |

### Check Running Containers

```bash
docker ps
```

### Jenkins Docker Configuration

- Container name: `jenkins`
- Image: `jenkins-gcc:latest`
- Restart policy: `no`
- Docker networks: `bridge`, `rt_default`

### Jenkins Persistent Storage

Jenkins uses a Docker named volume.

```text
Volume Name: jenkins_home

Container Path:
/var/jenkins_home

Docker Storage Path:
/var/lib/docker/volumes/jenkins_home/_data
```

The volume preserves Jenkins jobs, plugins, configuration, credentials and workspace data independently of the container lifecycle.

---

## 6. Jenkins Configuration

Jenkins runs inside a customized Docker image based on:

```dockerfile
FROM jenkins/jenkins:lts-jdk21
```

The custom image includes GCC and the required build tools.

Installed packages:

```text
gcc
build-essential
```

### Verify GCC

```bash
docker exec jenkins gcc --version
```

### Verify Jenkins

Open:

http://localhost:8080

### Jenkins Workspace

The project workspace is:

```text
/var/jenkins_home/workspace/hello-world-c
```

The repository contains:

```text
hello-world-c/
├── hello.c
├── Jenkinsfile
└── README.md
```

### Jenkins Credentials

Credentials are managed through:

```text
Jenkins Dashboard
    → Manage Jenkins
        → Credentials
```

Main credentials file:

```text
/var/jenkins_home/credentials.xml
```

The file contains encrypted credential data. Never publish its contents or expose secret values.

---

## 7. GitHub Integration

GitHub repository:

```text
https://github.com/TDPATEL1/jenkins-hello-world
```

The repository contains the C source code and Jenkins pipeline definition.

### Webhook Automation

GitHub is configured to send webhook events to Jenkins through ngrok.

Workflow:

1. Developer pushes code to GitHub.
2. GitHub generates a push event.
3. GitHub sends an HTTP webhook request to the ngrok HTTPS endpoint.
4. ngrok forwards the request to Jenkins.
5. Jenkins detects the repository change.
6. Jenkins starts the appropriate build.

### Multibranch Pipeline

The Jenkins multibranch pipeline is configured to discover branches containing a Jenkinsfile.

Job:

```text
hello-world-c-multibranch
```

The pipeline supports branch-based builds and pull-request testing.

---

## 8. ngrok Configuration

ngrok provides an externally accessible HTTPS endpoint that forwards incoming requests to the local Jenkins server.

Local forwarding target:

```text
http://localhost:8080
```

The public HTTPS forwarding URL is generated by ngrok and may change between sessions.

### Start ngrok

```bash
ngrok http 8080
```

After starting ngrok:

1. Copy the HTTPS forwarding URL.
2. Open the GitHub repository.
3. Navigate to Settings → Webhooks.
4. Configure the webhook URL using the current ngrok address and Jenkins webhook endpoint.
5. Verify webhook delivery.

The ngrok forwarding URL must remain active for external webhook delivery.

---

## 9. JFrog Artifactory Configuration

JFrog Artifactory OSS is used as the centralized artifact repository.

Installed version:

```text
7.161.15
```

### Services

| Component | Configuration |
|---|---|
| Artifactory container | `artifactory` |
| PostgreSQL container | `postgresql` |
| Artifactory UI | Port 8082 |
| Artifactory API | Port 8081 |
| Database | PostgreSQL 17.10 |

### Repository

Repository name:

```text
jenkins-artifacts
```

Repository purpose:

- Store compiled build artifacts.
- Maintain timestamped artifact versions.
- Organize artifacts according to branch and pull-request builds.
- Provide centralized artifact access.

### Artifact Naming

Generated artifacts use a timestamp-based naming convention.

Example:

```text
a.out-20261113-124023
```

Naming pattern:

```text
a.out-YYYYMMDD-HHMMSS
```

This helps distinguish artifacts generated by different pipeline executions.

### Artifact Access

Open:

```text
http://localhost:8082
```

Navigate to:

```text
Artifacts
    → jenkins-artifacts
        → Browse
```

The repository contains the uploaded build artifacts organized according to the configured branch and pull-request structure.

---

## 10. Docker Network Configuration

Jenkins, Artifactory and PostgreSQL communicate through a shared Docker network.

Network:

```text
rt_default
```

Configuration:

| Setting | Value |
|---|---|
| Driver | bridge |
| Subnet | 172.20.0.0/16 |
| Gateway | 172.20.0.1 |

### Container IP Addresses

| Container | IP Address |
|---|---|
| Jenkins | 172.20.0.2 |
| Artifactory | 172.20.0.3 |
| PostgreSQL | 172.20.0.4 |

### Internal Communication

Jenkins connects to Artifactory using:

```text
http://artifactory:8081
```

Artifactory connects to PostgreSQL using:

```text
postgresql:5432
```

Docker container hostnames are used for internal service discovery.

### Inspect Network

```bash
docker network inspect rt_default
```

---

## 11. CI/CD Pipeline Workflow

The Jenkins pipeline automates the following operations:

### Stage 1: Checkout

Retrieve the latest source code from GitHub.

### Stage 2: Compile

Compile the C source code using GCC.

```bash
gcc -Wall -Wextra hello.c -o hello
```

### Stage 3: Execute

Run the compiled executable.

```bash
./hello
```

### Stage 4: Package

Create a compressed build package.

```bash
tar -czf hello-world-c.tar.gz hello
```

### Stage 5: Generate Timestamp

Generate a unique timestamp for the artifact filename.

Example:

```text
a.out-20261113-124023
```

### Stage 6: Upload to JFrog

Upload the generated artifact to the `jenkins-artifacts` repository using the configured Jenkins credentials.

### Stage 7: Notifications

Send pipeline status notifications through Jenkins email configuration.

---

## 12. Email Notifications

Jenkins uses the Email Extension Plugin for pipeline notifications.

Email notifications are configured to report build status.

The pipeline notification logic uses:

```groovy
emailext
```

The notification mechanism is intended to report successful and failed pipeline executions.

SMTP credentials and email configuration are maintained securely in Jenkins.

---

## 13. Docker Container Monitoring

A separate PowerShell monitoring script is configured on Windows.

Monitoring directory:

```text
C:\Users\admin\docker-monitor
```

Files:

```text
docker-monitor/
├── Monitor-Docker.ps1
├── docker-monitor.log
└── alert-state.txt
```

The monitoring system checks the Jenkins Docker container.

### Monitoring Features

- Detect Jenkins container availability.
- Identify container failure or stoppage.
- Send email alerts when Jenkins becomes unavailable.
- Send recovery notifications when Jenkins becomes available again.
- Maintain monitoring logs.
- Support automatic Jenkins restart through the monitoring script.

Windows Task Scheduler is configured with the task:

```text
Docker Container Monitor
```

The monitoring task is configured to run periodically.

---

## 14. Starting the Environment

### Step 1: Start Docker Desktop

Ensure Docker Desktop is running with the WSL2 backend.

### Step 2: Start JFrog PostgreSQL

Open Ubuntu and navigate to the installation directory.

```bash
cd /mnt/d/jfrog-artifactory/artifactory-oss-7.161.15
```

Start PostgreSQL:

```bash
docker compose -f docker-compose-postgres.yaml up -d
```

### Step 3: Start Artifactory

```bash
docker compose -f docker-compose.yaml up -d
```

### Step 4: Start Jenkins

```bash
docker start jenkins
```

### Step 5: Verify Containers

```bash
docker ps
```

### Step 6: Verify Artifactory

```bash
curl http://localhost:8081/artifactory/api/system/ping
```

Expected response:

```text
OK
```

### Step 7: Start ngrok

```bash
ngrok http 8080
```

Verify the forwarding URL and GitHub webhook delivery.

---

## 15. Troubleshooting

### Jenkins Is Not Accessible

Check the container:

```bash
docker ps -a
```

Start Jenkins:

```bash
docker start jenkins
```

Check logs:

```bash
docker logs jenkins
```

### Artifactory Is Not Accessible

Check the container:

```bash
docker ps -a
```

Check logs:

```bash
docker logs artifactory
```

### PostgreSQL Connectivity Issues

Check:

```bash
docker logs postgresql
```

Verify that PostgreSQL is running and that Artifactory uses the correct database hostname.

### GitHub Webhook Not Triggering Jenkins

Verify:

- Jenkins is running.
- ngrok is running.
- The current ngrok URL is configured in GitHub.
- The webhook endpoint is correct.
- GitHub webhook delivery reports a successful response.

### Artifact Not Visible in JFrog

Verify:

- The Jenkins build completed successfully.
- The upload stage executed.
- The `jenkins-artifacts` repository exists.
- The configured repository path is correct.
- Jenkins has valid Artifactory credentials.

### Check Container Logs

```bash
docker logs jenkins
docker logs artifactory
docker logs postgresql
```

---

## 16. Security Considerations

- Never commit passwords, API tokens or app passwords to GitHub.
- Store Jenkins credentials using the Jenkins Credentials Manager.
- Do not publish `credentials.xml`.
- Protect the ngrok authentication token.
- Use authenticated Artifactory API access.
- Avoid exposing PostgreSQL unnecessarily.
- Keep Docker and Jenkins configuration backups.
- Do not publish private configuration files or secret environment variables.

---

## 17. Project Completion Summary

The project demonstrates an automated software build and artifact management workflow.

### Implemented Components

- Docker-based Jenkins environment.
- Custom Jenkins image with GCC.
- GitHub repository integration.
- ngrok-based webhook connectivity.
- Jenkins multibranch pipeline.
- Automated C compilation and execution.
- Timestamp-based artifact generation.
- JFrog Artifactory integration.
- PostgreSQL-backed Artifactory deployment.
- Branch and pull-request artifact organization.
- Jenkins email notification configuration.
- Docker container monitoring.

### Final Workflow

```text
Developer Push
      |
      v
GitHub
      |
      v
ngrok HTTPS Webhook
      |
      v
Jenkins Pipeline
      |
      v
Source Checkout
      |
      v
GCC Compilation
      |
      v
Executable Generation
      |
      v
Timestamped Artifact
      |
      v
JFrog Artifactory
      |
      v
Centralized Artifact Storage
```

**Project Goal:** Automate the build and artifact management process so that source-code changes can trigger reproducible builds and centralized artifact storage without requiring manual compilation and upload.

---

## 18. Repository Information

GitHub Repository:

https://github.com/TDPATEL1/jenkins-hello-world

Main technologies:

- Jenkins
- Docker
- GitHub
- ngrok
- GCC
- JFrog Artifactory
- PostgreSQL
- PowerShell




# Jenkins + JFrog Artifactory + PostgreSQL
## Complete Setup Guide — From Scratch on a New Device

This repository contains the instructions to build a complete CI/CD automation environment using Docker, Jenkins, JFrog Artifactory, PostgreSQL, GitHub, and ngrok.

The objective is to automate the process of compiling C programs whenever code is pushed to GitHub and uploading the generated build artifacts to JFrog Artifactory.

---

# 1. Project Architecture

The project uses three separate Docker containers.

| Service | Container | Responsibility |
|---|---|---|
| Jenkins | `jenkins` | CI/CD automation, code compilation and build execution |
| JFrog Artifactory | `artifactory` | Build artifact storage and management |
| PostgreSQL | `postgresql` | Database for JFrog Artifactory |

### Workflow

```text
Developer
    |
    v
GitHub Repository
    |
    | Git Push / Pull Request
    v
ngrok HTTPS Tunnel
    |
    v
Jenkins Container
    |
    | 1. Checkout source code
    | 2. Compile C program
    | 3. Execute program
    | 4. Generate timestamped artifact
    |
    v
JFrog Artifactory Container
    |
    | Store artifacts by branch / PR
    |
    v
PostgreSQL Container
    |
    v
Stores JFrog internal database information
```

All three containers run on the same Docker host and communicate through a Docker network.

# 2. Prerequisites

Before starting, install the following software on the new device.

### Required software

1. Windows 10/11
2. WSL2
3. Ubuntu 24.04
4. Docker Desktop
5. Git
6. ngrok account
7. GitHub account

### Step 1: Install WSL2

Open PowerShell as Administrator.

```powershell
wsl --install -d Ubuntu-24.04
```

Restart the computer if requested.

Open Ubuntu and create a Linux username and password.

Verify:

```bash
wsl --version
```

### Step 2: Install Docker Desktop

Download Docker Desktop:

https://www.docker.com/products/docker-desktop/

During installation:

- Enable the WSL2 backend.
- Start Docker Desktop.
- Open Settings.
- Navigate to Resources → WSL Integration.
- Enable Ubuntu-24.04 integration.

Verify Docker from Ubuntu:

```bash
docker --version
docker compose version
docker ps
```

### Step 3: Install Git

```bash
sudo apt update
sudo apt install git -y
```

Verify:

```bash
git --version
```

# 3. Create Project Directory

Create a directory for the JFrog installation.

```bash
sudo mkdir -p /mnt/d/jfrog-artifactory
sudo chown -R $USER:$USER /mnt/d/jfrog-artifactory
```

Navigate:

```bash
cd /mnt/d/jfrog-artifactory
```

The final structure will look similar to:

```text
D:/
 |
 +-- jfrog-artifactory/
 |    |
 |    +-- artifactory-oss-7.161.15/
 |         |
 |         +-- docker-compose.yaml
 |         +-- docker-compose-postgres.yaml
 |         +-- .env
 |         +-- var/
 |
 +-- jenkins-docker/
      |
      +-- Dockerfile
```

# 4. Install PostgreSQL and JFrog Artifactory

We use JFrog Artifactory OSS version 7.161.15.

### Step 1: Download JFrog Artifactory OSS

Visit the official JFrog download page:

https://jfrog.com/community/download-artifactory-oss/

Download the Linux Docker Compose distribution for Artifactory OSS.

The archive used in this project is:

```text
jfrog-artifactory-oss-7.161.15-compose.tar.gz
```

Place the archive inside:

```text
/mnt/d/jfrog-artifactory/
```

Extract it:

```bash
cd /mnt/d/jfrog-artifactory

tar -xzf jfrog-artifactory-oss-7.161.15-compose.tar.gz
```

Navigate into the extracted directory:

```bash
cd artifactory-oss-7.161.15
```

### Step 2: Check Docker Compose files

```bash
ls -la
```

You should find the Compose files supplied with the JFrog distribution, including:

```text
docker-compose.yaml
docker-compose-postgres.yaml
```

The PostgreSQL configuration is supplied separately so the database can run independently.

**Important:** Keep the original JFrog-generated Compose files and their environment configuration together. Do not replace their database settings with arbitrary values.

### Step 3: Start PostgreSQL

```bash
docker compose -f docker-compose-postgres.yaml up -d
```

What this command does:

- Reads the PostgreSQL Compose configuration.
- Downloads the required PostgreSQL image if missing.
- Creates the database container.
- Starts PostgreSQL in the background.

Verify:

```bash
docker ps
```

Check logs:

```bash
docker logs postgresql
```

### Step 4: Start JFrog Artifactory

```bash
docker compose -f docker-compose.yaml up -d
```

This starts the Artifactory application container.

Verify:

```bash
docker ps
```

Check logs:

```bash
docker logs -f artifactory
```

Press `Ctrl+C` to stop viewing the logs. This does not stop the container.

### Step 5: Verify Artifactory

Open the browser:

```text
http://localhost:8082
```

The Artifactory web interface should appear.

Check the API:

```bash
curl http://localhost:8081/artifactory/api/system/ping
```

Expected response:

```text
OK
```

### Step 6: Configure Artifactory

Log in to the Artifactory UI and complete the initial setup.

Create a local Generic repository:

```text
Repository Key: jenkins-artifacts
Package Type: Generic
Repository Type: Local
```

This repository will store compiled C program artifacts.

The expected artifact structure is:

```text
jenkins-artifacts/
    |
    +-- main/
    |    |
    |    +-- a.out-20261117-120000
    |    +-- a.out-20261117-130000
    |
    +-- PR/
         |
         +-- a.out-20261117-140000
```

Create an Artifactory access token for Jenkins.

Keep the token secure. It will be configured as a Jenkins credential.

# 5. Configure Jenkins Docker Container

Jenkins will run in a separate Docker container.

We use a custom Jenkins image with GCC installed because our pipeline compiles C programs.

### Step 1: Create Jenkins directory

```bash
mkdir -p ~/jenkins-docker
cd ~/jenkins-docker
```

### Step 2: Create Dockerfile

Create a file named:

```text
Dockerfile
```

Add:

```dockerfile
FROM jenkins/jenkins:lts-jdk21

USER root

RUN apt-get update && \
    apt-get install -y \
    gcc \
    build-essential \
    curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
```

Explanation:

- `jenkins/jenkins:lts-jdk21`: Official Jenkins image with Java 21.
- `gcc`: C compiler.
- `build-essential`: Required compilation tools.
- `curl`: Used to communicate with JFrog APIs.
- `USER jenkins`: Runs Jenkins as its non-root user.

### Step 3: Build custom Jenkins image

```bash
docker build -t jenkins-gcc:latest .
```

Verify:

```bash
docker images
```

### Step 4: Start Jenkins container

```bash
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins-gcc:latest
```

Explanation:

| Configuration | Purpose |
|---|---|
| 8080 | Jenkins web interface |
| 50000 | Jenkins agent communication |
| jenkins_home | Persistent Jenkins data |
| --restart unless-stopped | Automatic restart after Docker restarts |

### Step 5: Get initial Jenkins password

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Copy the password.

Open:

```text
http://localhost:8080
```

Paste the password and complete the initial setup.

Install suggested plugins.

### Step 6: Install additional plugins

Navigate to:

```text
Manage Jenkins → Plugins
```

Install:

- Git
- GitHub
- GitHub Branch Source
- Pipeline
- Pipeline: Stage View
- Email Extension
- Credentials Binding

Restart Jenkins if requested.

# 6. Connect Jenkins and JFrog Docker Networks

Jenkins and Artifactory must communicate with each other.

The JFrog Compose project creates a Docker network.

Check:

```bash
docker network ls
```

In the existing setup, the network is:

```text
rt_default
```

Connect Jenkins:

```bash
docker network connect rt_default jenkins
```

If Jenkins is already connected, Docker will report that the endpoint already exists. Do not connect it again.

Verify:

```bash
docker inspect jenkins
```

Check that Jenkins and Artifactory are attached to the same network.

Test communication:

```bash
docker exec jenkins curl -s \
http://artifactory:8081/artifactory/api/system/ping
```

Expected:

```text
OK
```

This confirms Jenkins can reach Artifactory through Docker's internal DNS.

# 7. Configure GitHub Repository

Create or clone the GitHub repository.

Project repository used in the original setup:

https://github.com/TDPATEL1/jenkins-hello-world

Expected files:

```text
jenkins-hello-world/
    |
    +-- hello.c
    +-- Jenkinsfile
    +-- README.md
```

### Example hello.c

```c
#include <stdio.h>

int main()
{
    printf("Hello World - Jenkins Webhook Test!\n");
    return 0;
}
```

### Push code to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/TDPATEL1/jenkins-hello-world.git
git push -u origin main
```

If the repository is already cloned, simply use:

```bash
git add .
git commit -m "Update source code"
git push
```

# 8. Configure Jenkins Credentials

Navigate to:

```text
Manage Jenkins
    → Credentials
    → System
    → Global credentials
```

Add the following credentials.

### JFrog access token

```text
Kind: Secret text
Secret: YOUR_JFROG_ACCESS_TOKEN
ID: jfrog-token
```

### GitHub credentials

For a private repository, configure a GitHub personal access token.

For a public repository, anonymous checkout may be sufficient.

Never store passwords or access tokens directly inside the Jenkinsfile.

# 9. Configure Jenkins Pipeline

Create a Multibranch Pipeline job.

Navigate:

```text
Jenkins Dashboard
    → New Item
    → Multibranch Pipeline
```

Job name:

```text
hello-world-c-multibranch
```

Configure the GitHub repository as the branch source.

Repository URL:

```text
https://github.com/TDPATEL1/jenkins-hello-world.git
```

Configure branch discovery and pull request discovery according to the required workflow.

Set the Jenkinsfile script path:

```text
Jenkinsfile
```

### Jenkinsfile

Add the following pipeline to the GitHub repository.

```groovy
pipeline {
    agent any

    environment {
        JFROG_URL = 'http://artifactory:8081/artifactory'
        JFROG_REPO = 'jenkins-artifacts'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check GCC') {
            steps {
                sh 'gcc --version'
            }
        }

        stage('Compile C Program') {
            steps {
                sh 'gcc -Wall -Wextra hello.c -o a.out'
            }
        }

        stage('Run Program') {
            steps {
                sh './a.out'
            }
        }

        stage('Upload Artifact') {
            steps {
                script {
                    def timestamp = new Date().format(
                        'yyyyMMdd-HHmmss',
                        TimeZone.getTimeZone('UTC')
                    )

                    def folder = env.CHANGE_ID
                        ? 'PR'
                        : env.BRANCH_NAME.replaceAll('/', '-')

                    def artifactName = "a.out-${timestamp}"

                    withCredentials([
                        string(
                            credentialsId: 'jfrog-token',
                            variable: 'JFROG_TOKEN'
                        )
                    ]) {
                        sh """
                            curl --fail --silent --show-error \
                            -H "Authorization: Bearer \$JFROG_TOKEN" \
                            -T a.out \
                            "${JFROG_URL}/${JFROG_REPO}/${folder}/${artifactName}"
                        """
                    }

                    echo "Uploaded artifact: ${folder}/${artifactName}"
                }
            }
        }
    }

    post {
        success {
            echo 'Build completed successfully!'
        }

        failure {
            echo 'Build failed!'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
```

**Note:** This example uses the actual Multibranch Pipeline environment variables. Normal branches are stored under their branch name; pull request builds are stored under `PR`. If your required branch folder is `master` rather than `main`, adjust the branch naming logic accordingly.

Commit and push the Jenkinsfile:

```bash
git add Jenkinsfile
git commit -m "Add Jenkins JFrog pipeline"
git push
```

# 10. Configure ngrok for GitHub Webhooks

ngrok provides a public HTTPS URL that forwards requests to your locally running Jenkins server.

### Step 1: Install ngrok

Visit:

https://ngrok.com/download

Install ngrok for Windows and configure your authentication token.

### Step 2: Start Jenkins

Make sure Jenkins is running:

```bash
docker start jenkins
```

### Step 3: Start ngrok

Open PowerShell or a terminal:

```powershell
ngrok http 8080
```

ngrok will generate an HTTPS forwarding URL.

Example:

```text
https://example.ngrok-free.dev
```

Your URL will be different.

Keep the ngrok terminal running.

### Step 4: Configure GitHub webhook

Open your GitHub repository.

Navigate:

```text
Settings
    → Webhooks
    → Add webhook
```

Configure:

```text
Payload URL:
https://YOUR-NGROK-URL/github-webhook/

Content type:
application/json

Events:
Just the push event
```

If pull request events are also required, enable the pull request event.

Save the webhook.

Now GitHub can notify Jenkins whenever a new commit or pull request event occurs.

**Important:** A free ngrok URL can change when its tunnel restarts. Update the GitHub webhook whenever the forwarding URL changes.

# 11. Configure Email Notifications (Optional)

Install and configure the Jenkins Email Extension Plugin.

Navigate:

```text
Manage Jenkins
    → System
    → Extended E-mail Notification
```

Configure your SMTP server.

For Gmail:

```text
SMTP Server: smtp.gmail.com
SMTP Port: 465
SSL: Enabled
```

Create a Gmail App Password and store it securely in Jenkins credentials.

Configure pipeline notifications using `emailext` if email alerts for build success and failure are required.

# 12. Verify Complete Setup

After completing all configurations, verify each service.

### Check Docker containers

```bash
docker ps
```

Expected containers:

```text
jenkins
artifactory
postgresql
```

### Check Jenkins

Open:

```text
http://localhost:8080
```

Verify that Jenkins is accessible.

### Check Artifactory

Open:

```text
http://localhost:8082
```

Verify that the repository `jenkins-artifacts` exists.

### Check PostgreSQL

```bash
docker logs --tail 50 postgresql
```

Verify that PostgreSQL is running without startup errors.

### Check Artifactory API

```bash
curl http://localhost:8081/artifactory/api/system/ping
```

Expected:

```text
OK
```

### Check Jenkins-to-JFrog connectivity

```bash
docker exec jenkins curl -s \
http://artifactory:8081/artifactory/api/system/ping
```

Expected:

```text
OK
```

### Test the complete pipeline

1. Modify `hello.c`.
2. Push changes to GitHub.
3. GitHub sends the webhook through ngrok.
4. Jenkins automatically starts a build.
5. Jenkins checks out the source code.
6. GCC compiles the program.
7. Jenkins executes the program.
8. Jenkins generates a timestamped `a.out` artifact.
9. Jenkins uploads the artifact to JFrog.
10. Open Artifactory and verify the uploaded artifact.

# 13. Useful Docker Commands

### View all running containers

```bash
docker ps
```

### View all containers

```bash
docker ps -a
```

### Start Jenkins

```bash
docker start jenkins
```

### Start JFrog and PostgreSQL

Run from the JFrog installation directory:

```bash
docker compose -f docker-compose-postgres.yaml up -d

docker compose -f docker-compose.yaml up -d
```

### Stop Jenkins

```bash
docker stop jenkins
```

### Stop JFrog

```bash
docker compose -f docker-compose.yaml stop
```

### Stop PostgreSQL

```bash
docker compose -f docker-compose-postgres.yaml stop
```

### View Jenkins logs

```bash
docker logs -f jenkins
```

### View Artifactory logs

```bash
docker logs -f artifactory
```

### View PostgreSQL logs

```bash
docker logs -f postgresql
```

### Restart Jenkins

```bash
docker restart jenkins
```

### Restart Artifactory

```bash
docker restart artifactory
```

### Restart PostgreSQL

```bash
docker restart postgresql
```

**Warning:** Do not use `docker compose down -v` or delete Docker volumes during routine troubleshooting. Volumes contain persistent application and database data.

# 14. Persistent Storage and Backup

Docker volumes ensure that data survives container restarts and container replacement.

Important data:

| Service | Persistent data |
|---|---|
| Jenkins | `jenkins_home` |
| JFrog | Artifactory data and configuration volumes |
| PostgreSQL | PostgreSQL database volume |

List Docker volumes:

```bash
docker volume ls
```

Inspect Jenkins volume:

```bash
docker volume inspect jenkins_home
```

Back up important volumes and configuration files regularly.

For PostgreSQL, use a database-aware backup method such as `pg_dump` or a coordinated database backup. For JFrog, follow its backup and restore documentation and include the corresponding database and binary storage.

Never delete the volumes when rebuilding containers on the same device.

# 15. Troubleshooting

### Problem 1: Jenkins is not opening

Check:

```bash
docker ps -a
docker logs --tail 100 jenkins
```

Restart:

```bash
docker restart jenkins
```

### Problem 2: Artifactory is not opening

Check:

```bash
docker logs --tail 100 artifactory
```

Check PostgreSQL:

```bash
docker logs --tail 100 postgresql
```

Verify the database service is healthy before troubleshooting Artifactory connectivity.

### Problem 3: Jenkins cannot connect to JFrog

Check the Docker network:

```bash
docker network ls
```

Connect Jenkins if necessary:

```bash
docker network connect rt_default jenkins
```

Test:

```bash
docker exec jenkins curl -s \
http://artifactory:8081/artifactory/api/system/ping
```

### Problem 4: GitHub webhook is not triggering Jenkins

Check:

- Jenkins is running.
- ngrok tunnel is active.
- GitHub webhook URL is correct.
- GitHub webhook delivery status is successful.
- GitHub integration plugins are installed.

### Problem 5: JFrog artifact is not uploaded

Check:

- JFrog token is correct.
- Jenkins credential ID is `jfrog-token`.
- Repository `jenkins-artifacts` exists.
- Jenkins can communicate with Artifactory.
- Pipeline has upload permissions.

# 16. Official Documentation

- Docker: https://docs.docker.com/
- Docker Compose: https://docs.docker.com/compose/
- Jenkins: https://www.jenkins.io/doc/
- Jenkins Docker: https://github.com/jenkinsci/docker
- JFrog Artifactory: https://jfrog.com/help/
- PostgreSQL: https://www.postgresql.org/docs/
- GitHub Webhooks: https://docs.github.com/en/webhooks
- ngrok: https://ngrok.com/docs

---

# Final Expected Result

After completing the setup, the new device should have the following environment:

| Component | Expected status |
|---|---|
| Docker Desktop | Running |
| Jenkins | Running on port 8080 |
| JFrog Artifactory | Running on port 8082 |
| PostgreSQL | Running as a separate container |
| GCC | Installed in Jenkins image |
| GitHub integration | Configured |
| ngrok | Forwarding to Jenkins |
| Jenkins Pipeline | Building C programs |
| JFrog Repository | Receiving timestamped artifacts |
| Branch/PR structure | Configured |

**Project objective achieved:** A developer pushes code to GitHub, Jenkins automatically builds the C application, and JFrog Artifactory stores the generated build artifacts in an organized repository.

**Architecture:** Three independent Docker containers managed through Docker Compose and Docker networking where appropriate.

