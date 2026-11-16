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

