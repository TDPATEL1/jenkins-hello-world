# GitHub → Jenkins → JFrog Artifactory Automation
## Complete Setup Guide – Building the Entire Project from Scratch on a New Laptop

**Project:** Automated CI/CD Pipeline for C Programs  
**Source Repository:** https://github.com/TDPATEL1/jenkins-hello-world  
**Jenkins Job:** `hello-world-c-multibranch`  
**JFrog Repository:** `jenkins-artifacts`

---

# 1. Project Overview

This project implements an automated CI/CD pipeline using GitHub, Jenkins, Docker, ngrok, and JFrog Artifactory.

Whenever a developer pushes code to GitHub, Jenkins automatically detects the change through a GitHub webhook and executes the pipeline.

Jenkins checks out the source code, compiles the C program using GCC, executes the generated executable, prepares a ZIP package containing the executable and build metadata, and uploads the ZIP package to JFrog Artifactory.

The project also includes Jenkins email notifications and Docker container monitoring.

## Project Objectives

1. Automate builds whenever code is pushed to GitHub.
2. Run Jenkins inside Docker.
3. Provide external webhook connectivity using ngrok.
4. Compile C programs automatically using GCC.
5. Generate uniquely named build artifacts.
6. Generate a `manifest.json` containing build information.
7. Package the executable and manifest into a ZIP file.
8. Upload ZIP artifacts to JFrog Artifactory.
9. Organize artifacts according to branch and pull request.
10. Send Jenkins build notifications through email.
11. Monitor Docker container availability.
12. Send email alerts when Jenkins stops or recovers.

# 2. Final Project Architecture

```text
                    Developer
                        |
                        |
                     Git Push
                        |
                        v
                GitHub Repository
                        |
                        |
                     Webhook
                        |
                        v
                      ngrok
                        |
                        |
                  HTTPS Forwarding
                        |
                        v
                 Jenkins (Docker)
                        |
                        |
                  Checkout Source
                        |
                        v
                  GCC Compilation
                        |
                        |
                   Execute Program
                        |
                        v
                 Generate Executable
                        |
                        v
                  Build Metadata
                        |
                        v
                  manifest.json
                        |
                        v
                   Create ZIP
                        |
                        |
                        v
               JFrog Artifactory
                        |
                        v
                jenkins-artifacts
```

Additional components:

```text
Jenkins ----------------> Email Notifications

Docker Monitor ----------> Email Alerts

Artifactory -------------> PostgreSQL Database
```

# 3. Technology Stack

| Technology | Purpose |
|---|---|
| Windows 11 | Host operating system |
| WSL2 | Linux environment |
| Ubuntu 24.04 | Development environment |
| Docker Desktop | Container management |
| Docker Compose | Container deployment |
| Jenkins | CI/CD automation |
| GitHub | Source code management |
| ngrok | External webhook connectivity |
| GCC | C program compilation |
| JFrog Artifactory OSS | Artifact management |
| PostgreSQL | Artifactory database |
| PowerShell | Docker monitoring |
| Gmail SMTP | Email notifications |

# 4. Prerequisites

Install the following software on the new laptop:

1. Windows 10/11
2. WSL2
3. Ubuntu 24.04
4. Docker Desktop
5. Git
6. ngrok
7. A GitHub account
8. An internet connection

The laptop should have sufficient memory and disk space to run Jenkins, Artifactory, PostgreSQL and Docker simultaneously.

Recommended: At least 16 GB RAM.

# 5. Install WSL2 and Ubuntu 24.04

## Step 1: Open PowerShell as Administrator

Run:

```powershell
wsl --install -d Ubuntu-24.04
```

This installs WSL2 and Ubuntu.

Restart the laptop if Windows requests it.

## Step 2: Verify WSL

```powershell
wsl --version
```

Check installed distributions:

```powershell
wsl --list --verbose
```

Expected distribution:

```text
NAME            STATE       VERSION
Ubuntu-24.04    Running     2
```

If Ubuntu is not using WSL2:

```powershell
wsl --set-version Ubuntu-24.04 2
```

## Step 3: Open Ubuntu

Open Ubuntu 24.04 from the Windows Start menu.

Create a Linux username and password.

In this project, the Linux username is:

```text
developer
```

Update Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
```

Install basic utilities:

```bash
sudo apt install -y \
    git \
    curl \
    wget \
    tar \
    gzip \
    unzip \
    nano \
    ca-certificates \
    gnupg
```

Verify:

```bash
git --version
curl --version
```

# 6. Install Docker Desktop

## Step 1: Download Docker Desktop

Official download:

https://www.docker.com/products/docker-desktop/

Install Docker Desktop for Windows.

## Step 2: Configure Docker Desktop

Open Docker Desktop.

Navigate to:

```text
Settings
    → General
```

Enable:

```text
Use the WSL 2 based engine
```

Navigate to:

```text
Settings
    → Resources
        → WSL Integration
```

Enable:

```text
Ubuntu-24.04
```

Apply the changes and restart Docker Desktop.

## Step 3: Verify Docker from Ubuntu

Open Ubuntu and execute:

```bash
docker --version
```

```bash
docker compose version
```

```bash
docker ps
```

Check Docker information:

```bash
docker info
```

Docker should be accessible directly from the Ubuntu terminal.

# 7. Install Git

Git is used to manage the source code and push changes to GitHub.

Run:

```bash
sudo apt update
sudo apt install git -y
```

Verify:

```bash
git --version
```

Configure Git:

```bash
git config --global user.name "YOUR_GITHUB_USERNAME"
```

```bash
git config --global user.email "YOUR_GITHUB_EMAIL"
```

Check configuration:

```bash
git config --global --list
```

Replace the placeholders with the actual GitHub account information.

# 8. Install JFrog Artifactory OSS

We use JFrog Artifactory OSS version:

```text
7.161.15
```

Artifactory will run inside Docker.

PostgreSQL will run as a separate Docker container.

## Step 1: Download JFrog Artifactory

Open the official download page:

https://jfrog.com/community/download-artifactory-oss/

Download the Linux Docker Compose distribution.

Archive used in this project:

```text
jfrog-artifactory-oss-7.161.15-compose.tar.gz
```

Place the downloaded archive in the Windows D drive under:

```text
D:\jfrog-artifactory
```

From Ubuntu, the Windows D drive is available at:

```text
/mnt/d
```

## Step 2: Create the installation directory

```bash
sudo mkdir -p /mnt/d/jfrog-artifactory
```

Give the current Linux user access:

```bash
sudo chown -R $USER:$USER /mnt/d/jfrog-artifactory
```

Navigate:

```bash
cd /mnt/d/jfrog-artifactory
```

Check the downloaded archive:

```bash
ls -lh
```

## Step 3: Extract the archive

```bash
tar -xzf jfrog-artifactory-oss-7.161.15-compose.tar.gz
```

Navigate:

```bash
cd artifactory-oss-7.161.15
```

Check the files:

```bash
ls -la
```

The extracted directory should contain the JFrog Docker Compose configuration, including:

```text
docker-compose.yaml
docker-compose-postgres.yaml
```

It may also contain environment configuration files and other supporting files.

Keep all supplied configuration files together.

Do not replace the generated database configuration with arbitrary values.

## Step 4: Start PostgreSQL

Run:

```bash
docker compose -f docker-compose-postgres.yaml up -d
```

This command:

- Reads the PostgreSQL Compose configuration.
- Downloads the required PostgreSQL image if necessary.
- Creates the database container.
- Starts PostgreSQL in the background.

Verify:

```bash
docker ps
```

Check PostgreSQL logs:

```bash
docker logs --tail 50 postgresql
```

## Step 5: Start Artifactory

Run:

```bash
docker compose -f docker-compose.yaml up -d
```

This starts JFrog Artifactory.

Check containers:

```bash
docker ps
```

Check Artifactory logs:

```bash
docker logs --tail 100 artifactory
```

Artifactory may take several minutes to initialize.

Do not assume it is ready immediately after the container starts.

## Step 6: Verify Artifactory

Open:

```text
http://localhost:8082
```

The JFrog Artifactory web interface should appear.

Check the API:

```bash
curl -i http://localhost:8081/artifactory/api/system/ping
```

Expected successful response:

```text
HTTP/1.1 200
OK
```

If it initially returns an error, inspect the Artifactory and PostgreSQL logs and wait for initialization.

# 9. Configure JFrog Artifactory

## Step 1: Log in

Open:

```text
http://localhost:8082
```

Complete the initial setup and log in using the configured administrator account.

## Step 2: Create a Local Generic Repository

Navigate to:

```text
Administration
    → Repositories
        → Local
            → Create a Local Repository
```

Select:

```text
Package Type: Generic
```

Configure:

| Setting | Value |
|---|---|
| Repository Key | jenkins-artifacts |
| Package Type | Generic |
| Repository Type | Local |

Save the repository.

## Step 3: Create an Artifactory Access Token

Create an access token for Jenkins.

Keep the following information:

```text
Artifactory URL
Artifactory username
Artifactory access token
```

Never commit the token to GitHub.

The token will be stored in Jenkins Credentials Manager.

## Step 4: Expected Repository Structure

```text
jenkins-artifacts/
    |
    +-- master/
    |      |
    |      +-- 1.0.0.3_master_2026.11.17_18.31_767566c.zip
    |      |
    |      +-- 1.0.0.3_master_2026.11.17_19.20_abcdef1.zip
    |
    +-- PR/
           |
           +-- 1.0.0.3_PR-123_2026.11.17_19.30_abcdef1.zip
```

The filenames above are examples.

# 10. Create the Custom Jenkins Docker Image

Jenkins requires GCC to compile C programs.

We create a custom Jenkins image based on the official Jenkins image.

The image already contains JDK 21, which provides the `jar` utility used to create ZIP packages.

## Step 1: Create the Jenkins directory

From Ubuntu:

```bash
mkdir -p ~/jenkins-docker
```

Navigate:

```bash
cd ~/jenkins-docker
```

## Step 2: Create the Dockerfile

Run:

```bash
nano Dockerfile
```

Paste:

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

Save the file.

Explanation:

| Package | Purpose |
|---|---|
| jenkins/jenkins:lts-jdk21 | Jenkins with Java 21 |
| gcc | C compiler |
| build-essential | C development tools |
| curl | HTTP communication with JFrog |
| jar | ZIP packaging utility supplied by the JDK |

## Step 3: Build the Docker image

```bash
docker build -t jenkins-gcc:latest .
```

Verify:

```bash
docker images
```

Expected image:

```text
jenkins-gcc
```

# 11. Start the Jenkins Container

## Step 1: Create a persistent Docker volume

```bash
docker volume create jenkins_home
```

This volume stores:

- Jenkins jobs
- Plugins
- Credentials
- Jenkins configuration
- Build history
- Other Jenkins data

## Step 2: Start Jenkins

Run:

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
| jenkins_home | Persistent Jenkins storage |
| --restart unless-stopped | Restart Jenkins automatically unless manually stopped |

## Step 3: Verify Jenkins

```bash
docker ps
```

Check logs:

```bash
docker logs jenkins
```

## Step 4: Get the initial administrator password

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Copy the generated password.

Open:

```text
http://localhost:8080
```

Paste the password and complete the Jenkins installation.

Select:

```text
Install suggested plugins
```

Create the Jenkins administrator account.

# 12. Install Jenkins Plugins

Navigate to:

```text
Manage Jenkins
    → Plugins
        → Available Plugins
```

Install the following plugins:

1. Git
2. GitHub
3. GitHub Branch Source
4. Pipeline
5. Pipeline: Stage View
6. Email Extension
7. Credentials Binding

Install the plugins and restart Jenkins if requested.

# 13. Verify Jenkins Build Tools

The Jenkins Docker image must contain GCC, Java and the JAR utility.

Run from Ubuntu:

```bash
docker exec jenkins bash -c "which gcc && gcc --version"
```

Check Java:

```bash
docker exec jenkins java -version
```

Check the JAR utility:

```bash
docker exec jenkins bash -c "which jar && jar --version"
```

Check curl:

```bash
docker exec jenkins curl --version
```

These commands should execute successfully.

# 14. Connect Jenkins and Artifactory Through Docker Networking

Jenkins and Artifactory run in separate containers.

They must communicate through a common Docker network.

The JFrog Compose project creates a network named:

```text
rt_default
```

## Step 1: Check Docker networks

```bash
docker network ls
```

## Step 2: Inspect the JFrog network

```bash
docker network inspect rt_default
```

## Step 3: Connect Jenkins

```bash
docker network connect rt_default jenkins
```

If Jenkins is already connected, Docker may report that the endpoint already exists. This is normal.

## Step 4: Verify the connection

```bash
docker inspect jenkins
```

Check that `rt_default` appears under the container's network settings.

## Step 5: Test Artifactory connectivity from Jenkins

Use the Artifactory internal service port:

```bash
docker exec jenkins curl -i \
http://artifactory:8081/artifactory/api/system/ping
```

Expected successful response:

```text
OK
```

The internal Docker hostname is:

```text
artifactory
```

The direct Artifactory service port is:

```text
8081
```

Use the same endpoint in the Jenkins pipeline after verifying it works.

# 15. Configure GitHub Repository

GitHub repository:

```text
https://github.com/TDPATEL1/jenkins-hello-world
```

Repository name:

```text
jenkins-hello-world
```

The repository contains:

```text
jenkins-hello-world/
    |
    +-- hello.c
    |
    +-- Jenkinsfile
    |
    +-- README.md
```

## Step 1: Clone the repository

Open Ubuntu:

```bash
cd ~
```

Clone:

```bash
git clone https://github.com/TDPATEL1/jenkins-hello-world.git
```

Navigate:

```bash
cd jenkins-hello-world
```

Check files:

```bash
ls -la
```

## Step 2: Example hello.c

The C program is compiled by Jenkins.

Example:

```c
#include <stdio.h>

int main()
{
    printf("Hello World - Jenkins Webhook Test!\n");
    printf("This is a C language program.\n");

    return 0;
}
```

## Step 3: Configure Git

```bash
git config user.name "YOUR_GITHUB_USERNAME"
```

```bash
git config user.email "YOUR_GITHUB_EMAIL"
```

## Step 4: Push changes

```bash
git add .
```

```bash
git commit -m "Initial project setup"
```

```bash
git push origin main
```

For an existing repository, use the same procedure whenever you modify the source code or Jenkinsfile.

# 16. Configure Jenkins Credentials

Navigate:

```text
Manage Jenkins
    → Credentials
        → System
            → Global credentials
                → Add Credentials
```

Two sets of credentials may be required.

## A. GitHub Credentials

For a private repository or authenticated GitHub API access, create a GitHub Personal Access Token.

Configure the appropriate repository and API permissions.

For a fine-grained token, check the permissions needed by the installed GitHub Branch Source plugin, including repository contents and commit status access if Jenkins is expected to publish build statuses.

Add the credential:

| Setting | Value |
|---|---|
| Kind | Username with password |
| Username | GitHub username |
| Password | GitHub PAT |
| ID | github-pat |

For a public repository, anonymous Git checkout may be sufficient, but authenticated API access is useful for branch and PR discovery.

## B. Artifactory Credentials

Add another credential:

| Setting | Value |
|---|---|
| Kind | Username with password |
| Username | Artifactory username |
| Password | Artifactory access token |
| ID | artifactory-creds |

The Jenkinsfile uses:

```groovy
credentialsId: 'artifactory-creds'
```

The credential ID must match exactly.

Never place the actual token or password inside the Jenkinsfile.

# 17. Configure Jenkins Multibranch Pipeline

Open Jenkins:

```text
http://localhost:8080
```

Navigate:

```text
Jenkins Dashboard
    → New Item
```

Enter:

```text
hello-world-c-multibranch
```

Select:

```text
Multibranch Pipeline
```

Click OK.

## Step 1: Configure Branch Source

Add:

```text
GitHub
```

Configure:

| Setting | Value |
|---|---|
| Credentials | github-pat |
| Repository Owner | TDPATEL1 |
| Repository | jenkins-hello-world |

Alternatively, use the full repository URL if the plugin configuration requests it.

## Step 2: Configure Branch Discovery

Enable branch discovery.

The pipeline should discover:

```text
main
```

## Step 3: Configure Pull Request Discovery

Enable the required pull request discovery strategy.

Configure the desired PR build behavior.

The pipeline uses:

```groovy
env.CHANGE_ID
```

to identify pull request builds.

## Step 4: Configure the Jenkinsfile path

Set:

```text
Jenkinsfile
```

Save the configuration.

## Step 5: Scan Repository

Click:

```text
Scan Multibranch Pipeline Now
```

Jenkins should discover the main branch and any eligible pull requests.

# 18. Configure ngrok for GitHub Webhooks

ngrok provides a public HTTPS endpoint that forwards external requests to the local Jenkins server.

## Step 1: Create an ngrok account

Open:

https://ngrok.com/

Create an account.

Download ngrok for Windows.

## Step 2: Configure the authentication token

Follow the current ngrok dashboard instructions to add your authentication token.

The command generally looks like:

```powershell
ngrok config add-authtoken YOUR_NGROK_AUTHTOKEN
```

Do not publish the token.

## Step 3: Start Jenkins

From Ubuntu:

```bash
docker start jenkins
```

Verify:

```bash
docker ps
```

Open:

```text
http://localhost:8080
```

## Step 4: Start ngrok

Open Windows PowerShell.

Run:

```powershell
ngrok http 8080
```

ngrok generates an HTTPS forwarding URL.

Example:

```text
https://example.ngrok-free.dev
```

The actual URL will be different.

Keep the ngrok terminal running.

## Step 5: Configure GitHub Webhook

Open:

```text
https://github.com/TDPATEL1/jenkins-hello-world
```

Navigate:

```text
Settings
    → Webhooks
        → Add webhook
```

Configure:

Payload URL:

```text
https://YOUR-NGROK-URL/github-webhook/
```

Content type:

```text
application/json
```

Select the required events:

```text
Push events
Pull request events
```

Save the webhook.

## Step 6: Verify webhook delivery

Navigate:

```text
GitHub Repository
    → Settings
        → Webhooks
            → Recent Deliveries
```

Verify that the webhook delivery succeeds.

Important:

- ngrok must remain running.
- Jenkins must be accessible on port 8080.
- The webhook URL must use the current ngrok HTTPS address.
- A free ngrok URL may change after restarting the tunnel.

Update the GitHub webhook whenever the forwarding URL changes.

# 19. Configure the Jenkinsfile

The Jenkinsfile defines the complete CI/CD workflow.

The current requirements are:

1. Detect whether the build is from the main branch or a PR.
2. Generate an IST timestamp.
3. Get the complete Git commit hash.
4. Use the first seven characters in the artifact filename.
5. Compile the C program.
6. Execute the program.
7. Rename the executable.
8. Generate `manifest.json`.
9. Package both files into a ZIP.
10. Upload the ZIP to JFrog Artifactory.

## Artifact Naming Convention

For main builds:

```text
1.0.0.3_master_YYYY.MM.DD_HH.MM_<firstSevenCharOfCommit>.zip
```

For pull request builds:

```text
1.0.0.3_PR-<XYZ>_YYYY.MM.DD_HH.MM_<firstSevenCharOfCommit>.zip
```

## Expected ZIP Structure

Example master build:

```text
1.0.0.3_master_2026.11.17_18.31_767566c.zip
    |
    +-- 1.0.0.3_master_2026.11.17_18.31_767566c
    |
    +-- manifest.json
```

The executable does not have an extension.

## Expected manifest.json

```json
{
    "timestamp": "2026.11.17_18.31",
    "Jenkins-hello-world": {
        "commit": "767566c931d254506f4afcf6c638b68e64cac1f2"
    }
}
```

The manifest contains:

| Field | Description |
|---|---|
| timestamp | Build date and time in IST |
| Jenkins-hello-world | Git repository name |
| commit | Full Git commit hash |

The ZIP filename uses only the first seven characters of the commit hash.

The manifest stores the complete commit hash.

## Complete Jenkinsfile

Create or update the Jenkinsfile in the GitHub repository.

Use:

```groovy
pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'http://artifactory:8081/artifactory'
        REPO = 'jenkins-artifacts'
        VERSION = '1.0.0.3'
        GIT_REPO_NAME = 'Jenkins-hello-world'
    }

    stages {

        stage('Build Metadata') {
            steps {
                script {
                    // Generate build timestamp in IST
                    env.BUILD_TIMESTAMP = sh(
                        script: "TZ=Asia/Kolkata date '+%Y.%m.%d_%H.%M'",
                        returnStdout: true
                    ).trim()

                    // Get full commit hash
                    env.COMMIT_FULL = sh(
                        script: 'git rev-parse HEAD',
                        returnStdout: true
                    ).trim()

                    // Get first seven characters
                    env.COMMIT_SHORT = env.COMMIT_FULL.take(7)

                    // Identify branch or Pull Request
                    if (env.CHANGE_ID) {
                        env.ARTIFACT_FOLDER = 'PR'
                        env.BUILD_TYPE = "PR-${env.CHANGE_ID}"
                    } else if (env.BRANCH_NAME == 'main') {
                        env.ARTIFACT_FOLDER = 'master'
                        env.BUILD_TYPE = 'master'
                    } else {
                        error("Unsupported branch: ${env.BRANCH_NAME}")
                    }

                    // Generate artifact names
                    env.ARTIFACT_NAME =
                        "${env.VERSION}_${env.BUILD_TYPE}_${env.BUILD_TIMESTAMP}_${env.COMMIT_SHORT}"

                    env.ZIP_NAME = "${env.ARTIFACT_NAME}.zip"

                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Build Type: ${env.BUILD_TYPE}"
                    echo "Full Commit: ${env.COMMIT_FULL}"
                    echo "Short Commit: ${env.COMMIT_SHORT}"
                    echo "Timestamp: ${env.BUILD_TIMESTAMP}"
                    echo "Artifact: ${env.ARTIFACT_NAME}"
                    echo "ZIP: ${env.ZIP_NAME}"
                    echo "Folder: ${env.ARTIFACT_FOLDER}"
                }
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

        stage('Run Hello World') {
            steps {
                sh './a.out'
            }
        }

        stage('Prepare Artifact Package') {
            steps {
                sh '''
                    set -eu

                    # Create a clean packaging directory
                    rm -rf package
                    mkdir -p package

                    # Rename executable
                    cp a.out "package/$ARTIFACT_NAME"

                    # Generate manifest.json
                    cat > package/manifest.json <<EOF
{
    "timestamp": "$BUILD_TIMESTAMP",
    "$GIT_REPO_NAME": {
        "commit": "$COMMIT_FULL"
    }
}
EOF

                    # Verify manifest exists and is not empty
                    test -s package/manifest.json

                    # Create ZIP containing exactly two files
                    cd package
                    jar --create --file "../$ZIP_NAME" "$ARTIFACT_NAME" manifest.json
                    cd ..

                    echo "ZIP package created: $ZIP_NAME"

                    # Display ZIP contents
                    jar --list --file "$ZIP_NAME"
                '''
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'artifactory-creds',
                        usernameVariable: 'ART_USER',
                        passwordVariable: 'ART_PASS'
                    )
                ]) {
                    sh '''
                        set -eu

                        curl --fail --silent --show-error \
                          -u "$ART_USER:$ART_PASS" \
                          -T "$ZIP_NAME" \
                          "$ARTIFACTORY_URL/$REPO/$ARTIFACT_FOLDER/$ZIP_NAME"

                        echo "ZIP uploaded successfully!"
                    '''
                }
            }
        }

        stage('Success') {
            steps {
                echo "Build completed successfully!"
                echo "Uploaded ZIP: ${env.ARTIFACT_FOLDER}/${env.ZIP_NAME}"
                echo "Executable: ${env.ARTIFACT_NAME}"
                echo "Manifest: manifest.json"
                echo "Full Commit: ${env.COMMIT_FULL}"
            }
        }
    }

    post {
        always {
            echo "Jenkins build finished: ${currentBuild.currentResult}"
        }
    }
}
```

**Important configuration notes:**

- The Artifactory endpoint above uses internal port 8081. Verify it using the Jenkins-to-Artifactory connectivity test before using it.
- The repository must contain `hello.c`.
- Jenkins must have the credential ID `artifactory-creds`.
- The pipeline requires the `jar` command, supplied by the JDK.
- Python is not required by this version of the Jenkinsfile.
- The main GitHub branch is `main`, but the Artifactory folder and artifact identifier are `master`.
- PR builds require the GitHub Branch Source plugin and correct PR discovery configuration.

# 20. Push Jenkinsfile to GitHub

From the repository directory:

```bash
cd ~/jenkins-hello-world
```

Check changes:

```bash
git status
```

Add the Jenkinsfile:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add ZIP packaging and manifest generation"
```

Push:

```bash
git push origin main
```

Jenkins should detect the push through the webhook.

Open:

```text
http://localhost:8080
```

Navigate:

```text
hello-world-c-multibranch
    → main
        → Build History
            → Latest Build
                → Console Output
```

Check the build stages.

Expected stages:

```text
Build Metadata
Check GCC
Compile C Program
Run Hello World
Prepare Artifact Package
Upload to Artifactory
Success
```

# 21. Verify the ZIP Package

After a successful build, the Console Output should show:

```text
ZIP package created:
1.0.0.3_master_2026.11.17_18.31_767566c.zip
```

The archive listing should show exactly two files:

```text
1.0.0.3_master_2026.11.17_18.31_767566c
manifest.json
```

The package can also be downloaded from JFrog and extracted.

Verify that:

1. The executable is named according to the required convention.
2. `manifest.json` exists.
3. The manifest contains the full commit hash.
4. The ZIP contains exactly two files at its root.

# 22. Verify Artifacts in JFrog

Open:

```text
http://localhost:8082
```

Navigate:

```text
Artifacts
    → jenkins-artifacts
        → Browse
```

Open:

```text
master
```

For a main branch build, the ZIP should be visible.

For a pull request build, open:

```text
PR
```

The ZIP should be visible there.

Expected structure:

```text
jenkins-artifacts/
    |
    +-- master/
    |      |
    |      +-- 1.0.0.3_master_2026.11.17_18.31_767566c.zip
    |
    +-- PR/
           |
           +-- 1.0.0.3_PR-123_2026.11.17_19.30_abcdef1.zip
```

The filenames are examples.

# 23. Configure Jenkins Email Notifications

Jenkins uses the Email Extension Plugin for build notifications.

## Step 1: Install the plugin

Navigate:

```text
Manage Jenkins
    → Plugins
        → Available Plugins
```

Install:

```text
Email Extension Plugin
```

## Step 2: Configure SMTP

Navigate:

```text
Manage Jenkins
    → System
        → Extended E-mail Notification
```

For Gmail, configure:

```text
SMTP Server: smtp.gmail.com
SMTP Port: 465
SSL: Enabled
```

Create a Gmail App Password through the Google Account security settings.

Store the password securely in Jenkins.

Do not put the password directly into the Jenkinsfile.

## Step 3: Configure email settings

Configure:

- SMTP authentication
- Default sender address
- Recipient address
- Email format
- SMTP test connection

Send a test email.

## Step 4: Configure pipeline notifications

The Email Extension Plugin provides:

```groovy
emailext
```

Use the pipeline notification configuration to send build success and failure messages.

The email notification should include:

- Job name
- Build number
- Build status
- Build URL
- Build result

Note: The Jenkinsfile in Section 19 currently performs packaging and uploading but does not include an `emailext` notification stage. Add the notification logic after SMTP configuration if build notifications are required.

# 24. Configure Docker Container Monitoring

A separate PowerShell script monitors the Jenkins Docker container.

Monitoring directory:

```text
C:\Users\admin\docker-monitor
```

Expected structure:

```text
docker-monitor/
    |
    +-- Monitor-Docker.ps1
    |
    +-- docker-monitor.log
    |
    +-- alert-state.txt
```

## Monitoring Features

1. Check whether the Jenkins container is running.
2. Detect when Jenkins stops.
3. Send an email when Jenkins becomes unavailable.
4. Detect when Jenkins recovers.
5. Send a recovery email.
6. Maintain a monitoring log.
7. Support automatic Jenkins restart.

## Step 1: Verify Docker from PowerShell

Open PowerShell:

```powershell
docker ps
```

Verify Jenkins:

```powershell
docker inspect jenkins
```

Check status:

```powershell
docker inspect -f '{{.State.Status}}' jenkins
```

## Step 2: Create the monitoring directory

```powershell
New-Item -ItemType Directory -Force -Path "C:\Users\admin\docker-monitor"
```

## Step 3: Create monitoring files

```powershell
New-Item -ItemType File -Force -Path "C:\Users\admin\docker-monitor\Monitor-Docker.ps1"
```

```powershell
New-Item -ItemType File -Force -Path "C:\Users\admin\docker-monitor\docker-monitor.log"
```

```powershell
New-Item -ItemType File -Force -Path "C:\Users\admin\docker-monitor\alert-state.txt"
```

The monitoring script must contain the Docker status-checking, alert-state, email notification and recovery logic.

Configure the SMTP credentials securely rather than writing an app password directly into the script.

Use the tested monitoring script from the original project as the reference when transferring the setup to a new laptop.

## Step 4: Test the monitor

From PowerShell:

```powershell
docker stop jenkins
```

Run the monitor script:

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\Users\admin\docker-monitor\Monitor-Docker.ps1"
```

Verify that the stopped-container alert is generated.

Start Jenkins again:

```powershell
docker start jenkins
```

Run the monitor again.

Verify that the recovery notification is generated.

## Step 5: Configure Windows Task Scheduler

Open:

```text
Task Scheduler
```

Navigate:

```text
Create Task
```

Task name:

```text
Docker Container Monitor
```

Configure a trigger to run periodically, such as every one minute.

Configure the action:

Program:

```text
powershell.exe
```

Arguments:

```text
-NoProfile -ExecutionPolicy Bypass -File "C:\Users\admin\docker-monitor\Monitor-Docker.ps1"
```

Set the task to run whether the user is logged in or not, if the Windows account and permissions permit it.

Test the scheduled task.

Important: Docker Desktop must be running for container monitoring and automatic container recovery to work.

The PowerShell script and its email configuration must be transferred securely from the original setup or recreated and tested on the new laptop.

# 25. Complete Environment Startup Procedure

After the initial setup, use this procedure whenever the laptop restarts.

## Step 1: Start Docker Desktop

Open Docker Desktop in Windows.

Wait until Docker is running.

## Step 2: Open Ubuntu

Open Ubuntu 24.04.

## Step 3: Navigate to JFrog directory

```bash
cd /mnt/d/jfrog-artifactory/artifactory-oss-7.161.15
```

## Step 4: Start PostgreSQL

```bash
docker compose -f docker-compose-postgres.yaml up -d
```

## Step 5: Start Artifactory

```bash
docker compose -f docker-compose.yaml up -d
```

Wait for Artifactory initialization.

## Step 6: Start Jenkins

```bash
docker start jenkins
```

## Step 7: Verify containers

```bash
docker ps
```

Expected containers:

```text
jenkins
artifactory
postgresql
```

## Step 8: Verify Artifactory API

```bash
curl http://localhost:8081/artifactory/api/system/ping
```

Expected:

```text
OK
```

## Step 9: Verify Jenkins

Open:

```text
http://localhost:8080
```

## Step 10: Verify JFrog

Open:

```text
http://localhost:8082
```

## Step 11: Start ngrok

Open Windows PowerShell:

```powershell
ngrok http 8080
```

Verify the HTTPS forwarding URL.

If the URL has changed, update the GitHub webhook.

# 26. Useful Docker Commands

## Container Management

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

Start Jenkins:

```bash
docker start jenkins
```

Stop Jenkins:

```bash
docker stop jenkins
```

Restart Jenkins:

```bash
docker restart jenkins
```

Start Artifactory:

```bash
docker start artifactory
```

Restart Artifactory:

```bash
docker restart artifactory
```

Start PostgreSQL:

```bash
docker start postgresql
```

Restart PostgreSQL:

```bash
docker restart postgresql
```

## View Logs

Jenkins:

```bash
docker logs -f jenkins
```

Artifactory:

```bash
docker logs -f artifactory
```

PostgreSQL:

```bash
docker logs -f postgresql
```

Display only the last 100 lines:

```bash
docker logs --tail 100 jenkins
```

## Docker Network

List networks:

```bash
docker network ls
```

Inspect JFrog network:

```bash
docker network inspect rt_default
```

Inspect Jenkins:

```bash
docker inspect jenkins
```

## Docker Volumes

List volumes:

```bash
docker volume ls
```

Inspect Jenkins volume:

```bash
docker volume inspect jenkins_home
```

**Warning:** Never use `docker compose down -v` or delete Docker volumes during routine troubleshooting. These volumes may contain Jenkins configuration, build history, Artifactory data and PostgreSQL database information.

# 27. Troubleshooting

## Problem 1: Jenkins is not opening

Check:

```bash
docker ps -a
```

Check logs:

```bash
docker logs --tail 100 jenkins
```

Start:

```bash
docker start jenkins
```

## Problem 2: Artifactory is not opening

Check:

```bash
docker ps -a
```

Check logs:

```bash
docker logs --tail 100 artifactory
```

Verify PostgreSQL:

```bash
docker logs --tail 100 postgresql
```

Wait for Artifactory initialization.

## Problem 3: Jenkins cannot connect to Artifactory

Check network:

```bash
docker network ls
```

Connect Jenkins if necessary:

```bash
docker network connect rt_default jenkins
```

Test:

```bash
docker exec jenkins curl -i \
http://artifactory:8081/artifactory/api/system/ping
```

Expected:

```text
OK
```

## Problem 4: Jenkins reports `jar: not found`

Check:

```bash
docker exec jenkins bash -c "which jar && jar --version"
```

The custom image must use a JDK-based Jenkins image.

Rebuild the custom image if necessary.

## Problem 5: Jenkins reports `gcc: not found`

Check:

```bash
docker exec jenkins gcc --version
```

If GCC is missing, rebuild the custom Jenkins image with GCC installed.

## Problem 6: GitHub webhook is not triggering Jenkins

Check:

1. Jenkins is running.
2. ngrok is running.
3. The current ngrok URL is configured in GitHub.
4. The webhook endpoint is `/github-webhook/`.
5. GitHub webhook delivery succeeded.
6. GitHub plugins are installed.
7. The multibranch pipeline is configured correctly.

## Problem 7: GitHub returns a 403 commit-status error

Example:

```text
Resource not accessible by personal access token
```

This indicates that Jenkins cannot publish commit status using the configured GitHub token.

Review the token's repository permissions and the GitHub Branch Source plugin configuration.

This issue is separate from the C compilation and JFrog upload stages.

## Problem 8: Artifactory upload fails

Check:

1. The `jenkins-artifacts` repository exists.
2. Jenkins has the correct Artifactory credentials.
3. The credential ID is `artifactory-creds`.
4. Jenkins can connect to Artifactory.
5. The access token has deployment permissions.
6. The upload URL is correct.

## Problem 9: ZIP package creation fails

Check:

```bash
docker exec jenkins jar --version
```

Check that the JDK is available.

Also verify that the package directory contains the renamed executable and `manifest.json`.

## Problem 10: manifest.json is missing

Check the Jenkins console output.

Verify that the `Prepare Artifact Package` stage completed.

The manifest must be generated before the ZIP creation command.

## Problem 11: Docker monitor emails are not arriving

Check:

- Windows Task Scheduler task status.
- PowerShell script execution.
- Gmail SMTP configuration.
- Gmail App Password.
- Docker Desktop availability.
- Monitoring logs.

Inspect:

```powershell
Get-Content "C:\Users\admin\docker-monitor\docker-monitor.log" -Tail 50
```

# 28. Security Considerations

1. Never commit passwords or access tokens to GitHub.
2. Store GitHub credentials in Jenkins Credentials Manager.
3. Store Artifactory credentials in Jenkins Credentials Manager.
4. Never publish `credentials.xml`.
5. Keep the ngrok authentication token private.
6. Do not expose PostgreSQL unnecessarily.
7. Protect Gmail App Passwords.
8. Keep Docker volumes backed up.
9. Do not share private Jenkins configuration files.
10. Use authenticated Artifactory access.
11. Use secure backups for PostgreSQL and Artifactory.
12. Avoid publishing environment variables containing secrets.

# 29. Backup and Persistent Storage

Jenkins uses a Docker named volume:

```text
jenkins_home
```

Inspect it:

```bash
docker volume inspect jenkins_home
```

JFrog and PostgreSQL also use persistent Docker storage.

List volumes:

```bash
docker volume ls
```

Back up Jenkins data before making major changes.

For PostgreSQL, use a database-aware backup method such as `pg_dump`.

For JFrog, back up the database and binary storage consistently using the appropriate JFrog backup procedure.

Do not delete volumes when rebuilding Docker containers.

# 30. Final Project Verification Checklist

Use this checklist before declaring the project operational.

## Environment

- [ ] Windows and WSL2 installed.
- [ ] Ubuntu 24.04 configured.
- [ ] Docker Desktop installed.
- [ ] Docker Compose working.
- [ ] Jenkins Docker image built.
- [ ] Jenkins container running.
- [ ] PostgreSQL container running.
- [ ] Artifactory container running.

## Jenkins

- [ ] Jenkins dashboard accessible.
- [ ] GCC installed.
- [ ] Java 21 available.
- [ ] JAR utility available.
- [ ] GitHub plugins installed.
- [ ] Jenkins credentials configured.
- [ ] Multibranch pipeline created.

## GitHub and ngrok

- [ ] GitHub repository connected.
- [ ] GitHub credentials configured.
- [ ] Branch discovery configured.
- [ ] Pull request discovery configured.
- [ ] ngrok authentication configured.
- [ ] ngrok tunnel running.
- [ ] GitHub webhook configured.
- [ ] Webhook delivery verified.

## JFrog

- [ ] Artifactory accessible.
- [ ] PostgreSQL connected.
- [ ] `jenkins-artifacts` repository created.
- [ ] Artifactory credentials configured in Jenkins.
- [ ] Jenkins-to-Artifactory connectivity verified.

## Artifact Packaging

- [ ] C program compiles successfully.
- [ ] C program executes successfully.
- [ ] IST timestamp generated.
- [ ] Full commit hash captured.
- [ ] Seven-character commit hash generated.
- [ ] Correct branch or PR detected.
- [ ] Executable renamed correctly.
- [ ] `manifest.json` generated.
- [ ] ZIP package created.
- [ ] ZIP contains exactly two files.
- [ ] ZIP uploaded successfully to JFrog.
- [ ] Main branch artifacts stored under `master`.
- [ ] PR artifacts stored under `PR`.

## Monitoring and Notifications

- [ ] Jenkins SMTP configured.
- [ ] Jenkins email test successful.
- [ ] Pipeline notifications configured.
- [ ] PowerShell Docker monitor configured.
- [ ] Docker monitoring email tested.
- [ ] Recovery email tested.
- [ ] Windows Task Scheduler configured.

# 31. Final Expected Result

After completing all the setup steps, the new laptop should have the following environment:

| Component | Expected Result |
|---|---|
| Docker Desktop | Running |
| WSL2 | Configured |
| Ubuntu 24.04 | Available |
| Jenkins | Running on port 8080 |
| Artifactory | Running on port 8082 |
| PostgreSQL | Running separately |
| GCC | Installed inside Jenkins |
| Java 21 | Available inside Jenkins |
| GitHub | Connected |
| ngrok | Forwarding to Jenkins |
| Multibranch Pipeline | Configured |
| JFrog Repository | Receiving ZIP artifacts |
| manifest.json | Generated |
| Email Notifications | Configured |
| Docker Monitoring | Configured |

## Final Artifact Structure

For a master build:

```text
jenkins-artifacts/
    |
    +-- master/
           |
           +-- 1.0.0.3_master_2026.11.17_18.31_767566c.zip
                    |
                    +-- 1.0.0.3_master_2026.11.17_18.31_767566c
                    |
                    +-- manifest.json
```

For a pull request build:

```text
jenkins-artifacts/
    |
    +-- PR/
           |
           +-- 1.0.0.3_PR-123_2026.11.17_19.30_abcdef1.zip
                    |
                    +-- 1.0.0.3_PR-123_2026.11.17_19.30_abcdef1
                    |
                    +-- manifest.json
```

# 32. Final Workflow

```text
              Developer
                  |
                  v
             GitHub Push
                  |
                  v
            GitHub Webhook
                  |
                  v
                ngrok
                  |
                  v
             Jenkins Docker
                  |
                  v
             Source Checkout
                  |
                  v
             Build Metadata
                  |
                  v
              GCC Compile
                  |
                  v
             Execute Program
                  |
                  v
           Generate Executable
                  |
                  v
             Create Manifest
                  |
                  v
              Create ZIP
                  |
                  v
          Upload ZIP to JFrog
                  |
                  v
         jenkins-artifacts Repo
                  |
           +------+------+
           |             |
           v             v
         master          PR
           |             |
           v             v
       ZIP Package    ZIP Package
```

Additional services:

```text
Jenkins Pipeline -------> Gmail Notifications

Docker Monitor ---------> Gmail Alerts

Artifactory ------------> PostgreSQL
```

**Project Goal**

Automate the entire build and artifact management process so that developers only need to push source-code changes to GitHub.

Jenkins automatically compiles and executes the C program, generates the timestamped executable and `manifest.json`, packages both into a ZIP file, and uploads the final artifact to JFrog Artifactory.

This provides a reproducible CI/CD workflow with centralized artifact storage, branch and pull-request organization, email notifications, and Docker container monitoring.

---

**GitHub Repository**

https://github.com/TDPATEL1/jenkins-hello-world

**Project Technologies**

- Jenkins
- Docker
- Docker Compose
- GitHub
- ngrok
- GCC
- JFrog Artifactory OSS
- PostgreSQL
- PowerShell
- Gmail SMTP

**End of Setup Guide**
