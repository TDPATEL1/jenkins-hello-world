# jenkins-hello-world

# Jenkins + GitHub Webhook Integration

## Project Overview

This project demonstrates the integration of GitHub with Jenkins to automate build execution whenever changes are pushed to a GitHub repository.

A simple C Hello World program is used to test the CI/CD workflow. Jenkins runs inside a Docker container, while ngrok exposes the locally running Jenkins server through a public HTTPS URL. A GitHub webhook sends push events to Jenkins, triggering builds automatically.

## Technologies Used

- **GitHub** – Source code management and repository hosting.
- **Jenkins** – CI/CD automation and build execution.
- **Docker** – Running Jenkins in a container.
- **ngrok** – Exposing the local Jenkins server through a public HTTPS URL.
- **C Language** – Sample application used for build testing.
- **Git Bash** – Managing Git commands and pushing changes.

## Project Structure

```text
jenkins-hello-world/
│
├── hello.c
├── Jenkinsfile
└── README.md
```

## Implementation Steps

### 1. Create a GitHub Repository

- Created a new GitHub repository named `jenkins-hello-world`.
- Initialized the repository to manage the project files.
- Configured the local Git repository with the remote GitHub repository.

### 2. Create and Push Project Files

Created two initial files:

- `hello.c` – A simple C program that prints a Hello World message.
- `Jenkinsfile` – Defines the Jenkins pipeline stages and build instructions.

Pushed both files to GitHub using Git Bash.

### 3. Install and Run Jenkins Using Docker

- Installed Docker Desktop.
- Downloaded and configured the Jenkins Docker image.
- Created and started a Jenkins container.
- Accessed the Jenkins dashboard through the local URL:

`http://localhost:8080`

- Completed the initial Jenkins setup and configuration.

### 4. Configure Jenkins

- Accessed the Jenkins dashboard.
- Installed the required plugins.
- Configured the Jenkins job to connect with the GitHub repository.
- Configured the pipeline to use the `Jenkinsfile` stored in the repository.

### 5. Expose Local Jenkins Using ngrok

Since Jenkins was running locally, ngrok was used to expose the local Jenkins server to the internet.

Started the ngrok tunnel:

```bash
ngrok http 8080
```

ngrok generated a public HTTPS URL that forwards incoming requests to the local Jenkins server.

Example:

`https://your-ngrok-domain.ngrok-free.dev`

### 6. Configure GitHub Webhook

- Opened the GitHub repository settings.
- Navigated to **Settings → Webhooks**.
- Added a new webhook.
- Configured the Payload URL using the public ngrok HTTPS URL and Jenkins webhook endpoint.
- Selected `application/json` as the content type.
- Configured the webhook to trigger on push events.
- Saved the webhook configuration.
- Verified that GitHub successfully delivered the webhook request.

### 7. Test Automatic Build Triggering

To verify the integration:

1. Modified the `hello.c` file.
2. Staged and committed the changes using Git Bash.
3. Pushed the changes to the GitHub repository.
4. GitHub sent a push event through the configured webhook.
5. ngrok forwarded the request to the local Jenkins server.
6. Jenkins automatically triggered the configured pipeline.
7. Verified the build execution and its status in Jenkins Build History.

## CI/CD Workflow

```text
Developer modifies hello.c
          |
          v
     Git Commit
          |
          v
    GitHub Push
          |
          v
   GitHub Webhook
          |
          v
    ngrok HTTPS URL
          |
          v
   Local Jenkins Server
      (Docker)
          |
          v
   Jenkins Pipeline
      (Jenkinsfile)
          |
          v
    Build Execution
          |
          v
    Build Result
```

## Learning Outcomes

- Understanding GitHub repository management.
- Working with Git commands through Git Bash.
- Running Jenkins using Docker.
- Configuring Jenkins pipelines using a Jenkinsfile.
- Exposing localhost services using ngrok.
- Configuring and testing GitHub webhooks.
- Understanding automated CI/CD build triggering.

## Conclusion

Successfully configured an automated GitHub-to-Jenkins integration using Docker and ngrok. The setup allows Jenkins to receive GitHub push events through a public HTTPS webhook and automatically execute the configured pipeline whenever code changes are pushed to the repository.

This project demonstrates the fundamentals of source code management, CI/CD automation, Docker-based Jenkins deployment, and webhook integration.
