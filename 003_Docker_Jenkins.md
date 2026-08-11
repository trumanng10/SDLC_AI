**Build → Test → Docker Build → Docker Scan → Push to Docker Hub**. Scanning before `docker push` prevents Jenkins from publishing an image that fails the security gate.

Assumption your Jenkins server is **Ubuntu Linux** and Jenkins is reachable at `http://12.12.12.12:8080`. 
Jenkins’ standard Linux installation runs as a `jenkins` service account and normally listens on port 8080 unless you changed it. ([Jenkins][1])

# Bonus Lab 3 — Build a Simple Jenkins CI Pipeline for Lab02API

## 1. Lab Objective

Continue from the previous labs:

```text
Lab 02
.NET API
   ↓
Docker
   ↓
Docker Hub

Lab 03
Docker Hub
   ↓
Kubernetes

Lab 04
GitHub
   ↓
Jenkins CI Pipeline
```

In this lab Jenkins will automatically perform:

```text
GitHub Repository
       │
       ▼
     Jenkins
       │
       ▼
① Checkout Source Code
       │
       ▼
② Restore .NET
       │
       ▼
③ Build .NET
       │
       ▼
④ Build Docker Image
       │
       ▼
⑤ Test Container
       │
       ▼
⑥ Scan Docker Image
       │
       ▼
⑦ Push Docker Image
       │
       ▼
    Docker Hub
```

The main file controlling this process is:

```text
Jenkinsfile
```

A Jenkinsfile stores the Pipeline definition as code in the source-code repository. Jenkins supports credentials being injected into Pipeline steps instead of storing passwords directly in the Jenkinsfile. ([Jenkins][2])

---

# Part A — What You Need

You already have:

```text
Jenkins Server

IP:
12.12.12.12
```

For this lab we also need the Jenkins server to have:

```text
Git
.NET 10 SDK
Docker
Trivy
curl
```

Trivy will be our Docker vulnerability scanner. It can scan container images for known vulnerabilities and can return a non-zero exit code so Jenkins can stop a Pipeline when security problems meet our chosen threshold. ([trivy.dev][3])

---

# Part B — Connect to Jenkins

## Step 1 — Open Jenkins

From your Windows browser open:

```text
http://12.12.12.12:8080
```

If your Jenkins administrator configured another port, use that port instead.

Login with your Jenkins account.

You should see:

```text
Jenkins Dashboard
```

---

# Part C — Prepare the Jenkins Server

## Step 2 — SSH Into the Jenkins Server

From Windows PowerShell:

```powershell
ssh YOUR_USERNAME@12.12.12.12
```

For example:

```powershell
ssh ubuntu@12.12.12.12
```

You should now be inside the Linux Jenkins server.

---

# Step 3 — Check Existing Software

Run:

```bash
git --version
```

Then:

```bash
dotnet --version
```

Then:

```bash
docker --version
```

Then:

```bash
trivy --version
```

Then:

```bash
curl --version
```

If they already work, do not reinstall them.

---

# Part D — Install .NET 10

## Step 4 — Install the .NET SDK

For supported Ubuntu versions such as Ubuntu 24.04 and Ubuntu 26.04, .NET 10 is available through the Ubuntu package system. ([Microsoft Learn][4])

Run:

```bash
sudo apt-get update
```

Then:

```bash
sudo apt-get install -y dotnet-sdk-10.0
```

Check:

```bash
dotnet --version
```

Expected:

```text
10.0.xxx
```

The SDK is required because Jenkins will run commands such as:

```text
dotnet restore
dotnet build
```

---

# Part E — Install Docker

## Step 5 — Install Docker Engine

If:

```bash
docker --version
```

already works, skip this section.

Docker's current Ubuntu installation uses Docker's official APT repository. ([Docker Documentation][5])

Run:

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

Create the key directory:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Download Docker's signing key:

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
-o /etc/apt/keyrings/docker.asc
```

Set permissions:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the Docker repository:

```bash
echo \
"Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc" \
| sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null
```

Update:

```bash
sudo apt update
```

Install Docker:

```bash
sudo apt install -y \
docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin
```

Check:

```bash
sudo systemctl status docker
```

Test:

```bash
sudo docker run hello-world
```

Docker documents `hello-world` as a basic installation verification test. ([Docker Documentation][5])

---

# Part F — Allow Jenkins to Use Docker

This step is very important.

Jenkins normally runs using the Linux account:

```text
jenkins
```

But Docker's Unix socket normally requires elevated privileges or membership in the `docker` group. Docker warns that membership in this group effectively grants root-level privileges, so this approach is suitable for this controlled training environment but should be reviewed carefully for production infrastructure. ([Docker Documentation][6])

## Step 6 — Add Jenkins to Docker Group

Run:

```bash
sudo usermod -aG docker jenkins
```

Restart Jenkins:

```bash
sudo systemctl restart jenkins
```

Test Docker as Jenkins:

```bash
sudo -u jenkins docker version
```

Then:

```bash
sudo -u jenkins docker ps
```

If these commands work, Jenkins can use Docker.

---

# Part G — Install Trivy

## Step 7 — Install the Docker Security Scanner

Install the prerequisites:

```bash
sudo apt-get install -y wget gnupg
```

Add the Trivy key:

```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
| gpg --dearmor \
| sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
```

Add its repository:

```bash
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" \
| sudo tee /etc/apt/sources.list.d/trivy.list
```

Update:

```bash
sudo apt-get update
```

Install:

```bash
sudo apt-get install -y trivy
```

These are the current official Debian/Ubuntu installation steps published by Trivy. ([Trivy][7])

Check:

```bash
trivy --version
```

---

# Step 8 — Test Trivy

Run:

```bash
trivy image hello-world
```

Trivy automatically obtains and maintains the vulnerability databases it needs for normal scanning. ([Trivy][8])

---

# Part H — Install Jenkins Plugins

Go back to:

```text
http://12.12.12.12:8080
```

Open:

```text
Manage Jenkins
     ↓
Plugins
```

Make sure the following are installed:

```text
Pipeline

Git

Credentials Binding
```

The Git plugin provides Pipeline-compatible Git checkout operations, while Credentials Binding lets Pipeline steps securely receive stored Jenkins credentials. ([Jenkins][9])

Restart Jenkins if requested.

---

# Part I — Create a Docker Hub Access Token

Do not place your Docker Hub password directly inside:

```text
Jenkinsfile
```

and do not put it into:

```text
GitHub
VS Code
Program.cs
Dockerfile
```

Docker recommends Personal Access Tokens for automated systems and CI/CD pipelines instead of exposing the main Docker Hub password. ([Docker Documentation][10])

Create a Docker Hub Personal Access Token with permission to push images.

You will need:

```text
Docker Hub Username

Docker Hub Access Token
```

---

# Part J — Store Docker Credentials in Jenkins

## Step 9 — Add Jenkins Credential

From Jenkins:

```text
Manage Jenkins
     ↓
Credentials
     ↓
System
     ↓
Global credentials
     ↓
Add Credentials
```

Select:

```text
Kind:
Username with password
```

Enter:

```text
Username:
YOUR_DOCKERHUB_USERNAME
```

For password use:

```text
YOUR_DOCKERHUB_ACCESS_TOKEN
```

Set:

```text
ID:

dockerhub-creds
```

Description:

```text
Docker Hub Credentials
```

Click:

```text
Create
```

Jenkins provides a central Credentials system specifically so secrets can be referenced by Pipelines without hard-coding them in source files. ([Jenkins][11])

---

# Part K — Prepare the Lab02API GitHub Repository

Your GitHub repository should eventually resemble:

```text
Lab02API
│
├── Program.cs
├── Lab02API.csproj
├── Dockerfile
├── .dockerignore
├── Jenkinsfile
│
├── k8s
│   ├── deployment.yaml
│   └── service.yaml
│
└── ...
```

Notice the new file:

```text
Jenkinsfile
```

---

# Part L — Use GitHub Copilot to Generate the Pipeline

## Step 10 — Open Lab02API in VS Code

Open:

```text
Lab02API
```

in VS Code.

Open GitHub Copilot Chat.

Give Copilot this prompt:

```text
You are an expert DevOps Engineer specialising in Jenkins,
Docker, .NET 10 and container security.

I am a beginner.

My project is Lab02API.

It is an ASP.NET Core .NET 10 application.

The project already contains a working Dockerfile.

The Docker container listens on port 8080.

I have an Ubuntu Jenkins server.

Create a simple declarative Jenkinsfile in the project root.

The Jenkins Pipeline must perform these stages:

1. Checkout source code.
2. Restore .NET dependencies.
3. Build the .NET application.
4. Build a Docker image.
5. Start the Docker container temporarily and perform
   an HTTP smoke test against port 8080.
6. Remove the test container.
7. Scan the Docker image using Trivy.
8. Report HIGH and CRITICAL vulnerabilities.
9. Fail the Pipeline if an unfixed CRITICAL vulnerability exists.
10. Login to Docker Hub using Jenkins credential ID:
    dockerhub-creds
11. Tag the image using the Jenkins BUILD_NUMBER.
12. Also tag it as latest.
13. Push both tags to Docker Hub.
14. Logout from Docker Hub.
15. Never hard-code passwords or access tokens.

Docker image repository:

YOUR_DOCKERHUB_USERNAME/lab02api

Use Linux sh commands.

Keep the Jenkinsfile simple and beginner-friendly.

Explain each stage after creating the file.
```

Replace:

```text
YOUR_DOCKERHUB_USERNAME
```

with your real Docker Hub username.

---

# Part M — Create the Jenkinsfile

## Step 11 — Create

In VS Code create:

```text
Jenkinsfile
```

There is:

```text
NO extension
```

Not:

```text
Jenkinsfile.txt
```

---

# Step 12 — Use This Jenkinsfile

Replace:

```text
YOUR_DOCKERHUB_USERNAME
```

with your actual Docker Hub username.

```groovy
pipeline {

    agent any

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    environment {
        DOCKER_IMAGE = 'YOUR_DOCKERHUB_USERNAME/lab02api'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Getting source code from GitHub...'
                checkout scm
            }
        }


        stage('Restore') {
            steps {
                echo 'Restoring .NET dependencies...'

                sh '''
                    dotnet restore Lab02API.csproj
                '''
            }
        }


        stage('Build') {
            steps {
                echo 'Building Lab02API...'

                sh '''
                    dotnet build Lab02API.csproj \
                    --configuration Release \
                    --no-restore
                '''
            }
        }


        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                '''
            }
        }


        stage('Test') {
            steps {
                echo 'Testing the Docker container...'

                sh '''
                    docker rm -f lab02api-test \
                    >/dev/null 2>&1 || true

                    docker run -d \
                    --name lab02api-test \
                    -p 18080:8080 \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}

                    echo "Waiting for Lab02API..."

                    SUCCESS=0

                    for i in 1 2 3 4 5 6 7 8 9 10
                    do
                        if curl -fsS http://127.0.0.1:18080/
                        then
                            SUCCESS=1
                            break
                        fi

                        sleep 2
                    done

                    if [ "$SUCCESS" -ne 1 ]
                    then
                        echo "API test failed."

                        docker logs lab02api-test || true

                        exit 1
                    fi

                    echo "API test successful."
                '''
            }

            post {
                always {
                    sh '''
                        docker rm -f lab02api-test \
                        >/dev/null 2>&1 || true
                    '''
                }
            }
        }


        stage('Docker Security Scan') {
            steps {
                echo 'Scanning Docker image using Trivy...'

                sh '''
                    echo "HIGH and CRITICAL vulnerability report:"

                    trivy image \
                    --ignore-unfixed \
                    --severity HIGH,CRITICAL \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}

                    echo "Checking security gate..."

                    trivy image \
                    --ignore-unfixed \
                    --severity CRITICAL \
                    --exit-code 1 \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }


        stage('Push to Docker Hub') {

            steps {

                echo 'Uploading Docker image to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_TOKEN" | \
                        docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin

                        docker tag \
                        ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        ${DOCKER_IMAGE}:latest

                        docker push \
                        ${DOCKER_IMAGE}:${IMAGE_TAG}

                        docker push \
                        ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }
    }


    post {

        success {
            echo '================================='
            echo 'PIPELINE SUCCESSFUL'
            echo '================================='
            echo "Docker Image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
        }

        failure {
            echo '================================='
            echo 'PIPELINE FAILED'
            echo 'Check the failed stage above.'
            echo '================================='
        }

        always {

            sh '''
                docker rm -f lab02api-test \
                >/dev/null 2>&1 || true

                docker logout \
                >/dev/null 2>&1 || true
            '''
        }
    }
}
```

---

# Part N — Understand the Pipeline

The Pipeline now contains:

```text
Stage 1
Checkout
   ↓
GitHub source code downloaded


Stage 2
Restore
   ↓
.NET dependencies restored


Stage 3
Build
   ↓
Lab02API compiled


Stage 4
Docker Build
   ↓
Docker image created


Stage 5
Test
   ↓
Container started
   ↓
HTTP request sent
   ↓
API must respond


Stage 6
Security Scan
   ↓
Trivy scans image
   ↓
Critical vulnerabilities?
   │
   ├── YES → Pipeline FAILED
   │
   └── NO → Continue


Stage 7
Docker Hub
   ↓
Login securely
   ↓
Push Build Number tag
   ↓
Push latest
```

---

# Part O — Understand the Build Number

Jenkins automatically provides:

```text
BUILD_NUMBER
```

Imagine this is Jenkins build:

```text
Build #15
```

Our image becomes:

```text
YOUR_DOCKERHUB_USERNAME/lab02api:15
```

We also create:

```text
YOUR_DOCKERHUB_USERNAME/lab02api:latest
```

Therefore Docker Hub might contain:

```text
lab02api

Tags:

1
2
3
4
5
...
15

latest
```

This is much better than overwriting only:

```text
1.0
```

because we can identify exactly which Jenkins build produced an image.

---

# Part P — Understand the Test Stage

For this first Jenkins lab, we deliberately use a:

```text
Smoke Test
```

rather than building a complicated unit-test framework.

Jenkins temporarily runs:

```text
Lab02API container
```

using:

```text
Jenkins Server Port
18080
       │
       ▼
Container Port
8080
```

Then Jenkins executes:

```bash
curl http://127.0.0.1:18080/
```

If Lab02API responds:

```text
AI SDLC Lab 02 API
```

the test succeeds.

If it cannot respond:

```text
Pipeline FAILED
```

Later, you can add:

```text
xUnit
MSTest
Integration Tests
Code Coverage
```

as a separate lab.

---

# Part Q — Understand the Docker Security Scan

The first Trivy command:

```bash
trivy image \
--ignore-unfixed \
--severity HIGH,CRITICAL \
IMAGE
```

shows:

```text
HIGH vulnerabilities

CRITICAL vulnerabilities
```

Trivy supports severity filtering directly on container-image scans. ([Trivy][3])

Then we run:

```bash
trivy image \
--ignore-unfixed \
--severity CRITICAL \
--exit-code 1 \
IMAGE
```

This means:

```text
No qualifying CRITICAL vulnerability
                │
                ▼
             Continue


CRITICAL vulnerability found
                │
                ▼
            exit code 1
                │
                ▼
          Jenkins FAILED
                │
                X
        DO NOT Docker Push
```

Trivy normally exits with code `0` even when vulnerabilities are detected; `--exit-code 1` is what turns the scan into a CI security gate. ([Trivy][12])

For a beginner lab, this policy is a reasonable starting point:

```text
HIGH
→ Display and investigate

CRITICAL + fix available
→ Stop deployment
```

---

# Part R — Understand Docker Hub Security

Our Jenkinsfile does NOT contain:

```text
Docker Password

Docker Access Token
```

Instead it contains:

```groovy
credentialsId: 'dockerhub-creds'
```

Jenkins retrieves the secret from:

```text
Jenkins Credentials Store
```

at runtime. Jenkins' Credentials Binding mechanism creates environment variables only within the relevant Pipeline scope. ([Jenkins][13])

For login we use:

```bash
echo "$DOCKER_TOKEN" |
docker login \
-u "$DOCKER_USER" \
--password-stdin
```

Docker recommends `--password-stdin` for non-interactive login because it avoids putting the password/token directly into command history or log files. ([Docker Documentation][14])

---

# Part S — Commit the Jenkinsfile to GitHub

## Step 13 — Ask Copilot to Review Everything

In VS Code Copilot Chat:

```text
Act as a senior Jenkins DevOps Engineer.

Review my:

- Jenkinsfile
- Dockerfile
- Lab02API.csproj
- Program.cs

Check that the Pipeline correctly performs:

1. Checkout
2. .NET restore
3. .NET build
4. Docker build
5. Container smoke test
6. Docker cleanup
7. Trivy vulnerability scan
8. Security gate
9. Secure Docker Hub login
10. Docker tag
11. Docker push
12. Docker logout

Check for:

- Jenkins Groovy syntax errors
- wrong ports
- wrong filenames
- hard-coded passwords
- Docker command errors
- shell command errors

Do not redesign the Pipeline.

Make only the minimum corrections required.

Explain all changes in beginner-friendly language.
```

---

# Step 14 — Push to GitHub

From VS Code terminal:

```bash
git status
```

Then:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add Jenkins CI pipeline"
```

Push:

```bash
git push
```

Your GitHub repository now contains:

```text
Lab02API
│
├── Jenkinsfile
├── Dockerfile
├── Program.cs
└── Lab02API.csproj
```

---

# Part T — Create the Jenkins Pipeline Job

## Step 15 — Create New Jenkins Job

Open:

```text
Jenkins Dashboard
```

Click:

```text
New Item
```

Enter:

```text
Lab02API-Pipeline
```

Select:

```text
Pipeline
```

Click:

```text
OK
```

---

# Step 16 — Configure Pipeline From GitHub

Scroll to:

```text
Pipeline
```

For:

```text
Definition
```

select:

```text
Pipeline script from SCM
```

For SCM select:

```text
Git
```

Enter your GitHub repository address.

For example conceptually:

```text
GitHub
YOUR_ACCOUNT/Lab02API
```

If the repository is public:

```text
Credentials:
None
```

If it is private, use a Jenkins Git credential rather than writing the GitHub secret into the Jenkinsfile. Jenkins' Git plugin supports credential-backed checkout operations. ([Jenkins][9])

For branch:

```text
*/main
```

Set:

```text
Script Path:

Jenkinsfile
```

Click:

```text
Save
```

---

# Part U — Run the Pipeline

## Step 17 — Start Your First Build

Click:

```text
Build Now
```

You should see:

```text
Build #1
```

Click it.

Then:

```text
Console Output
```

---

# Step 18 — Watch the Pipeline

You should see stages similar to:

```text
Checkout
    ✓

Restore
    ✓

Build
    ✓

Docker Build
    ✓

Test
    ✓

Docker Security Scan
    ✓

Push to Docker Hub
    ✓
```

Final result:

```text
SUCCESS
```

---

# Part V — Check Docker Hub

Open your Docker Hub repository.

You should see:

```text
lab02api
```

Tags should include something similar to:

```text
1

latest
```

Run Jenkins again.

You should then get:

```text
2

latest
```

Run again:

```text
3

latest
```

The complete process is now automatic.

---

# Part W — What Happens if Build Fails?

For example:

```text
Program.cs
contains compilation error
```

Pipeline:

```text
Checkout
   ✓

Restore
   ✓

Build
   ✕

Docker Build
   NOT RUN

Test
   NOT RUN

Security Scan
   NOT RUN

Docker Push
   NOT RUN
```

This is exactly what we want.

---

# Part X — What Happens if the API Does Not Start?

Pipeline:

```text
Docker Build
   ✓

Test
   ↓

docker run
   ↓

curl localhost:18080
   ↓

No response
   ↓

FAILED
```

Jenkins then displays the container logs.

No image is pushed.

---

# Part Y — What Happens if Trivy Finds a Critical Vulnerability?

```text
Build
  ✓

Test
  ✓

Docker Image
  ✓

Trivy
  ↓

CRITICAL vulnerability
  ↓

exit code 1
  ↓

Jenkins Pipeline
  FAILED
  ↓

Docker Push
  NOT RUN
```

That is your first:

```text
DevSecOps Security Gate
```

---

# Part Z — Common Problems

## Problem 1 — Permission Denied on docker.sock

You may see:

```text
permission denied while trying to connect
to the Docker daemon socket
```

Check:

```bash
groups jenkins
```

Look for:

```text
docker
```

If missing:

```bash
sudo usermod -aG docker jenkins
```

Then:

```bash
sudo systemctl restart jenkins
```

Remember that Docker documents the `docker` group as providing root-level access to the daemon. ([Docker Documentation][6])

---

## Problem 2 — dotnet Not Found

Check:

```bash
sudo -u jenkins dotnet --version
```

If not found:

```bash
sudo apt-get update
sudo apt-get install -y dotnet-sdk-10.0
```

Microsoft currently documents `dotnet-sdk-10.0` for supported Ubuntu releases. ([Microsoft Learn][4])

---

## Problem 3 — Trivy Not Found

Check:

```bash
sudo -u jenkins trivy --version
```

If the command is unavailable, reinstall Trivy using the official repository procedure above. ([Trivy][7])

---

## Problem 4 — Docker Login Failed

Check Jenkins:

```text
Manage Jenkins
→ Credentials
```

Confirm:

```text
ID:

dockerhub-creds
```

The ID must exactly match:

```groovy
credentialsId: 'dockerhub-creds'
```

Also verify that the stored username is your Docker ID and the password field contains a valid Docker Hub PAT.

Docker recommends PATs for CI/CD authentication. ([Docker Documentation][10])

---

## Problem 5 — Docker Push Denied

Example:

```text
requested access to the resource is denied
```

Check:

```groovy
DOCKER_IMAGE =
'YOUR_DOCKERHUB_USERNAME/lab02api'
```

The Docker Hub username must match the account represented by:

```text
dockerhub-creds
```

---

## Problem 6 — Smoke Test Failed

Look at the Jenkins Console Output.

You can also ask GitHub Copilot:

```text
My Jenkins Lab02API smoke test has failed.

Analyse this Jenkins console error.

Also inspect:

- Jenkinsfile
- Dockerfile
- Program.cs

My ASP.NET Core container should listen on port 8080.

Jenkins maps:

18080:8080

Find the root cause.

Do not redesign the Pipeline.

Make only the minimum fix necessary and explain it
in beginner-friendly language.
```

---

# Essential Jenkins Server Checks

Run these before troubleshooting the Pipeline:

```bash
sudo -u jenkins git --version
```

```bash
sudo -u jenkins dotnet --version
```

```bash
sudo -u jenkins docker version
```

```bash
sudo -u jenkins trivy --version
```

```bash
sudo -u jenkins curl --version
```

All five should work.

---

# Final Architecture

```text
               Developer
                   │
                   ▼
               VS Code
                   │
            GitHub Copilot
                   │
                   ▼
              Source Code
                   │
                   ▼
                 GitHub
                   │
                   │ Checkout
                   ▼
         ┌────────────────────┐
         │      Jenkins       │
         │    12.12.12.12     │
         └─────────┬──────────┘
                   │
                   ▼
            .NET Restore
                   │
                   ▼
             .NET Build
                   │
                   ▼
             Docker Build
                   │
                   ▼
          ┌────────────────┐
          │ Docker Image   │
          │ lab02api:#     │
          └───────┬────────┘
                  │
                  ▼
             Smoke Test
                  │
                  ▼
              Passed?
            ┌─────┴─────┐
           NO          YES
           │             │
           ▼             ▼
        FAILED          Trivy
                         │
                         ▼
                   Security Scan
                         │
                   CRITICAL?
                    ┌────┴────┐
                   YES       NO
                    │         │
                    ▼         ▼
                 FAILED   Docker Login
                              │
                              ▼
                         Docker Push
                              │
                              ▼
                       ┌──────────────┐
                       │  Docker Hub  │
                       │ lab02api:#   │
                       │   latest     │
                       └──────────────┘
```

---

# What You Have Built

Before Jenkins:

```text
Developer manually:

dotnet build

docker build

docker run

test

trivy scan

docker login

docker push
```

After Jenkins:

```text
Developer
   │
   ▼
git push
   │
   ▼
Jenkins
   │
   ├── Build
   ├── Test
   ├── Docker Build
   ├── Security Scan
   └── Docker Push
```

For this first CI lab, press:

```text
Build Now
```

manually.

Do not add:

```text
GitHub Webhook
Kubernetes Deployment
SonarQube
Email Notifications
Parallel Builds
Helm
Argo CD
```

yet.

Those are best added only after this basic Pipeline works reliably.

The important outcome is that students can clearly see **where CI stops when something goes wrong**: a compilation error stops before Docker, a failed API test stops before scanning, and a critical Trivy finding stops before Docker Hub. That makes this a much better first Jenkins exercise than packing deployment and Kubernetes into the same pipeline.

[1]: https://www.jenkins.io/doc/book/installing/linux/?utm_source=chatgpt.com "Linux"
[2]: https://www.jenkins.io/doc/book/pipeline/jenkinsfile/ "Using a Jenkinsfile"
[3]: https://trivy.dev/docs/latest/references/configuration/cli/trivy_image/?utm_source=chatgpt.com "Image"
[4]: https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install "Install .NET on Ubuntu - .NET | Microsoft Learn"
[5]: https://docs.docker.com/engine/install/ubuntu/?utm_source=chatgpt.com "Install Docker Engine on Ubuntu"
[6]: https://docs.docker.com/engine/install/linux-postinstall/?utm_source=chatgpt.com "Linux post-installation steps for Docker Engine"
[7]: https://trivy.dev/docs/latest/getting-started/installation/ "Installation - Trivy"
[8]: https://trivy.dev/docs/latest/configuration/db/?utm_source=chatgpt.com "Trivy - Databases"
[9]: https://www.jenkins.io/doc/pipeline/steps/git/?utm_source=chatgpt.com "Git plugin"
[10]: https://docs.docker.com/security/access-tokens/?utm_source=chatgpt.com "Personal access tokens"
[11]: https://www.jenkins.io/doc/book/using/using-credentials/?utm_source=chatgpt.com "Using credentials"
[12]: https://trivy.dev/docs/latest/guide/configuration/others/?utm_source=chatgpt.com "Others"
[13]: https://www.jenkins.io/doc/pipeline/steps/credentials-binding/ "Credentials Binding Plugin"
[14]: https://docs.docker.com/reference/cli/docker/login/?utm_source=chatgpt.com "docker login"
