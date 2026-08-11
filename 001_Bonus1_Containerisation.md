Assumes **Windows 10/11 + VS Code + GitHub Copilot**, and deliberately explains what each command is doing.

# Lab 02 — Containerise a .NET Web API and Upload It to Docker Hub

## 1. Lab Objective

In this lab, you will learn how to take a simple ASP.NET Core Web API called **Lab02API**, package it inside a Docker container, test the container locally, and upload the Docker image to Docker Hub.

You do **not** need previous Docker or .NET knowledge.

By the end of the lab, you will understand this workflow:

```text
Lab02API Source Code
        ↓
.NET Build
        ↓
Dockerfile
        ↓
docker build
        ↓
Docker Image
        ↓
docker run
        ↓
Docker Container
        ↓
docker push
        ↓
Docker Hub
```

We will use **.NET 10 LTS**. As of August 2026, .NET 10 is Microsoft's active LTS release and is supported until November 14, 2028. ([Microsoft][1])

---

# Part A — Understand the Basic Terminology

Before starting, understand these five terms.

### .NET

.NET is Microsoft's development platform.

We use the **.NET SDK** to:

```text
Create application
Compile application
Run application
Test application
Publish application
```

The `.NET SDK` includes the command-line tools needed for development. ([Microsoft Learn][2])

### Dockerfile

A Dockerfile is a set of instructions telling Docker how to package your application.

Think of it as a:

```text
Recipe for creating the Docker image.
```

### Docker Image

A Docker image is the packaged application.

For example:

```text
lab02api:1.0
```

It contains the application and the environment necessary to run it.

### Docker Container

A container is a **running instance of a Docker image**.

Conceptually:

```text
Dockerfile
    ↓
Docker Image
    ↓
Docker Container
```

### Docker Hub

Docker Hub is an online container registry where Docker images can be stored and shared.

Our final image will look something like:

```text
trumanng/lab02api:1.0
```

---

# Part B — Install the Required Software

## Step 1 — Install .NET 10 SDK

Open:

```text
Windows PowerShell
```

You can search for:

```text
PowerShell
```

from the Windows Start menu.

Run:

```powershell
winget install Microsoft.DotNet.SDK.10
```

Microsoft officially supports installing the .NET SDK on Windows using Windows Package Manager. ([Microsoft Learn][2])

After installation, close PowerShell and open it again.

Check the installation:

```powershell
dotnet --version
```

Expected result should resemble:

```text
10.0.xxx
```

You can get more information using:

```powershell
dotnet --info
```

If these commands work, your .NET environment is ready.

---

# Step 2 — Install Docker Desktop

Docker Desktop is the easiest Docker environment for this Windows lab.

Docker Desktop on Windows uses virtualization and commonly the WSL 2 backend. Current Docker requirements include a 64-bit processor, hardware virtualization and sufficient memory; Docker currently specifies 8 GB RAM for its Windows WSL 2 requirements. ([Docker Documentation][3])

First check WSL.

Open PowerShell as Administrator:

```powershell
wsl --version
```

If WSL is not installed, run:

```powershell
wsl --install
```

Restart Windows if requested.

Then update WSL:

```powershell
wsl --update
```

Download and install **Docker Desktop for Windows** from the official Docker website.

During installation, use the WSL 2 option where offered.

Start:

```text
Docker Desktop
```

Wait until Docker reports that the Docker Engine is running.

Test Docker from PowerShell:

```powershell
docker --version
```

Then:

```powershell
docker run hello-world
```

If you see a Docker welcome message, Docker is functioning correctly.

---

# Step 3 — Prepare GitHub Copilot in VS Code

Open VS Code.

Modern VS Code versions include GitHub Copilot functionality directly; sign in with your GitHub account to enable the AI features. ([Visual Studio Code][4])

Look for the **Copilot icon** in VS Code.

Select:

```text
Enable AI Features
```

or:

```text
Sign In
```

Sign in using your GitHub account.

You should then be able to open Copilot Chat.

You can also use:

```text
Ctrl + I
```

for inline Copilot editing. ([Visual Studio Code][5])

---

# Part C — Create Lab02API

## Step 4 — Create the Project

Create a folder for your labs.

Open PowerShell:

```powershell
mkdir C:\DevOpsLabs
cd C:\DevOpsLabs
```

Create the ASP.NET Core project:

```powershell
dotnet new web -n Lab02API --framework net10.0
```

Move into the project:

```powershell
cd Lab02API
```

Open the folder using VS Code:

```powershell
code .
```

Your project should resemble:

```text
Lab02API
│
├── Lab02API.csproj
├── Program.cs
├── appsettings.json
├── appsettings.Development.json
└── Properties
```

---

# Step 5 — Create the Simple API

Open:

```text
Program.cs
```

Replace its content with:

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/", () => "AI SDLC Lab 02 API");

app.MapGet("/discount", (int age) =>
{
    double discount = 0;

    if (age >= 60)
    {
        discount = 0.20;
    }

    return Results.Ok(new
    {
        age,
        discount
    });
});

app.Run();
```

Save the file.

---

# Step 6 — Test the Application Without Docker

Before Dockerising an application, make sure the application itself works.

Open the VS Code terminal:

```text
Terminal
→ New Terminal
```

Run:

```powershell
dotnet run
```

You should see output similar to:

```text
Now listening on: http://localhost:xxxx
```

Use the actual port displayed by .NET.

Open the URL in your browser.

For example:

```text
http://localhost:5000
```

Expected:

```text
AI SDLC Lab 02 API
```

Then test:

```text
http://localhost:5000/discount?age=65
```

Expected result:

```json
{
  "age": 65,
  "discount": 0.2
}
```

Test:

```text
http://localhost:5000/discount?age=30
```

Expected:

```json
{
  "age": 30,
  "discount": 0
}
```

Stop the application using:

```text
Ctrl + C
```

---

# Part D — Use GitHub Copilot to Containerise Lab02API

## Step 7 — Ask GitHub Copilot to Create the Docker Configuration

Open Copilot Chat.

Give Copilot this prompt:

```text
You are an expert DevOps Engineer specialising in Docker and ASP.NET Core.

I am a beginner and do not know Docker.

Analyse my current Lab02API ASP.NET Core .NET 10 project.

Containerise this application.

Requirements:

1. Create a Dockerfile for Lab02API.
2. Use a multi-stage Docker build.
3. Use mcr.microsoft.com/dotnet/sdk:10.0 for the build stage.
4. Use mcr.microsoft.com/dotnet/aspnet:10.0 for the runtime stage.
5. Run the ASP.NET Core Web API on container port 8080.
6. Use the non-root "app" user in the final container.
7. The entry point must run Lab02API.dll.
8. Create an appropriate .dockerignore file.
9. Do not change the existing API functionality.
10. Explain every section of the Dockerfile in simple beginner language.
11. Give me the docker build, docker run and testing commands.

Create the required files in my current VS Code project.
```

Review the proposed changes before accepting them.

---

# Step 8 — Check the Dockerfile

Your Dockerfile should be similar to:

```dockerfile
# ----------------------------
# Stage 1 - Build Application
# ----------------------------

FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build

WORKDIR /src

COPY ["Lab02API.csproj", "."]

RUN dotnet restore "./Lab02API.csproj"

COPY . .

RUN dotnet publish "./Lab02API.csproj" \
    -c Release \
    -o /app/publish \
    /p:UseAppHost=false


# ----------------------------
# Stage 2 - Runtime
# ----------------------------

FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final

WORKDIR /app

COPY --from=build /app/publish .

USER app

EXPOSE 8080

ENTRYPOINT ["dotnet", "Lab02API.dll"]
```

ASP.NET Core container images from .NET 8 onward use **port 8080 by default**, so using 8080 avoids unnecessary port configuration. Microsoft's container images also include the `app` non-root user. ([Microsoft Learn][6])

### What does the Dockerfile do?

The important process is:

```text
sdk:10.0
   ↓
Compile Lab02API
   ↓
Publish Lab02API
   ↓
aspnet:10.0
   ↓
Copy compiled application
   ↓
Run Lab02API.dll
```

The full SDK is required to build the program.

But we don't need the complete SDK simply to run the application.

Therefore the final Docker image uses:

```text
aspnet:10.0
```

instead.

This is called a:

```text
Multi-stage Docker build
```

---

# Step 9 — Check .dockerignore

Copilot should also create:

```text
.dockerignore
```

Example:

```text
bin/
obj/
.git/
.vscode/
*.user
*.suo
```

This prevents unnecessary development files from being sent into the Docker build process.

Your project should now resemble:

```text
Lab02API
│
├── Dockerfile
├── .dockerignore
├── Lab02API.csproj
├── Program.cs
├── appsettings.json
└── Properties
```

---

# Part E — Build the Docker Image

## Step 10 — Build Lab02API

Make sure Docker Desktop is running.

In the VS Code terminal, verify that you are inside:

```text
C:\DevOpsLabs\Lab02API
```

Run:

```powershell
docker build -t lab02api:1.0 .
```

Pay attention to the final:

```text
.
```

The dot means:

```text
Use the current directory as the Docker build context.
```

### What does `-t` mean?

```text
-t
```

means:

```text
tag
```

So:

```powershell
docker build -t lab02api:1.0 .
```

creates an image named:

```text
lab02api
```

with version:

```text
1.0
```

---

# Step 11 — Check the Docker Image

Run:

```powershell
docker images
```

You should see something similar to:

```text
REPOSITORY    TAG      IMAGE ID
lab02api      1.0      abc123456789
```

The application is now packaged as a Docker image.

---

# Part F — Run the Docker Container

## Step 12 — Start Lab02API

Run:

```powershell
docker run -d `
  --name lab02api-container `
  -p 8080:8080 `
  lab02api:1.0
```

Or as one line:

```powershell
docker run -d --name lab02api-container -p 8080:8080 lab02api:1.0
```

### Explanation

```text
docker run
```

starts a container.

```text
-d
```

runs it in the background.

```text
--name lab02api-container
```

gives the container a friendly name.

And:

```text
-p 8080:8080
```

means:

```text
Windows PC Port       Docker Container Port
       8080      →             8080
```

---

# Step 13 — Check the Running Container

Run:

```powershell
docker ps
```

You should see:

```text
lab02api-container
```

Check its logs:

```powershell
docker logs lab02api-container
```

---

# Step 14 — Test the Dockerised Web Service

Open your browser:

```text
http://localhost:8080
```

Expected:

```text
AI SDLC Lab 02 API
```

Now test:

```text
http://localhost:8080/discount?age=65
```

Expected:

```json
{
  "age": 65,
  "discount": 0.2
}
```

You can also test from PowerShell:

```powershell
Invoke-RestMethod http://localhost:8080/
```

Then:

```powershell
Invoke-RestMethod "http://localhost:8080/discount?age=65"
```

At this point you have successfully:

```text
Source Code
   ↓
Dockerfile
   ↓
Docker Image
   ↓
Docker Container
   ↓
Running Web API
```

---

# Part G — Upload Lab02API to Docker Hub

## Step 15 — Create a Docker Hub Account

Create/sign into your Docker Hub account.

Then create a repository called:

```text
lab02api
```

For classroom purposes, you can make it:

```text
Public
```

Docker's documented workflow is to tag the local image with the Docker Hub repository name and then push the image to that repository. ([Docker Documentation][7])

Suppose your Docker Hub username is:

```text
trumanng
```

Replace `YOUR_DOCKERHUB_USERNAME` below with your real username.

---

# Step 16 — Login to Docker Hub

In VS Code Terminal:

```powershell
docker login
```

Follow the login instructions.

Do **not** put your Docker Hub password or access token into:

```text
Dockerfile
Program.cs
GitHub Copilot prompt
Git repository
```

---

# Step 17 — Tag the Docker Image

Currently the image is:

```text
lab02api:1.0
```

Docker Hub needs the username/repository format:

```text
username/repository:tag
```

Run:

```powershell
docker tag lab02api:1.0 YOUR_DOCKERHUB_USERNAME/lab02api:1.0
```

For example:

```powershell
docker tag lab02api:1.0 trumanng/lab02api:1.0
```

Also create the `latest` tag:

```powershell
docker tag lab02api:1.0 trumanng/lab02api:latest
```

Check:

```powershell
docker images
```

You should now see something similar to:

```text
lab02api              1.0
trumanng/lab02api     1.0
trumanng/lab02api     latest
```

---

# Step 18 — Push the Image to Docker Hub

Upload version 1.0:

```powershell
docker push trumanng/lab02api:1.0
```

Upload `latest`:

```powershell
docker push trumanng/lab02api:latest
```

The `docker push` command publishes the tagged image to the registry. ([Docker Documentation][8])

Open Docker Hub.

You should now see:

```text
trumanng/lab02api
```

with tags such as:

```text
1.0
latest
```

---

# Part H — Test the Docker Hub Deployment

## Step 19 — Stop the Existing Container

Run:

```powershell
docker stop lab02api-container
```

Remove it:

```powershell
docker rm lab02api-container
```

---

# Step 20 — Pull Lab02API from Docker Hub

Imagine you are now using another computer.

Download the image:

```powershell
docker pull trumanng/lab02api:1.0
```

Run it:

```powershell
docker run -d --name lab02api-from-hub -p 8080:8080 trumanng/lab02api:1.0
```

Check:

```powershell
docker ps
```

Then open:

```text
http://localhost:8080
```

Test:

```text
http://localhost:8080/discount?age=65
```

If the API works, you have successfully completed the complete deployment workflow.

---

# Part I — Use GitHub Copilot to Verify Your Work

Give GitHub Copilot the following prompt:

```text
Act as a senior DevOps engineer and review my Lab02API Docker deployment.

Inspect:

- Program.cs
- Lab02API.csproj
- Dockerfile
- .dockerignore

Verify the following:

1. The .NET application can compile.
2. The Dockerfile uses .NET 10.
3. It uses a multi-stage Docker build.
4. The runtime image uses ASP.NET Core 10.
5. The application uses container port 8080.
6. Port 8080 is correctly exposed.
7. The application runs using Lab02API.dll.
8. The final container uses a non-root user.
9. .dockerignore is appropriate.
10. There are no passwords, tokens or secrets stored in the source code or Dockerfile.

Then provide the exact commands I should execute to:

- dotnet build
- docker build
- docker run
- test the API
- docker tag
- docker login
- docker push

Do not make unnecessary changes to my application.

Explain any problems in simple beginner language.
```

This allows Copilot to act as a basic DevOps reviewer.

---

# Part J — Useful Docker Commands

### View images

```powershell
docker images
```

### View running containers

```powershell
docker ps
```

### View all containers

```powershell
docker ps -a
```

### View logs

```powershell
docker logs lab02api-container
```

### Stop container

```powershell
docker stop lab02api-container
```

### Start container again

```powershell
docker start lab02api-container
```

### Remove container

```powershell
docker rm lab02api-container
```

### Force-remove container

```powershell
docker rm -f lab02api-container
```

### Remove image

```powershell
docker rmi lab02api:1.0
```

---

# Part K — Common Beginner Problems

## Problem 1 — `dotnet` is not recognized

Check:

```powershell
dotnet --version
```

If Windows cannot find it, reinstall the .NET SDK and restart your terminal.

---

## Problem 2 — Cannot connect to Docker

Check that:

```text
Docker Desktop
```

is running.

Then:

```powershell
docker info
```

---

## Problem 3 — Port 8080 already in use

Instead of:

```powershell
docker run -p 8080:8080 lab02api:1.0
```

use:

```powershell
docker run -p 8081:8080 lab02api:1.0
```

Then access:

```text
http://localhost:8081
```

Notice:

```text
8081 : 8080
  ↑      ↑
  |      └── Container
  └───────── Windows PC
```

---

## Problem 4 — Docker build fails

Ask Copilot:

```text
My Docker build has failed.

Analyse the terminal error together with my:

- Dockerfile
- Lab02API.csproj
- Program.cs

Identify the root cause.

Do not rewrite the whole project.

Explain the error in beginner language and make only the minimum changes necessary to fix it.

After fixing it, give me the docker build command again.
```

---

# Part L — Final Lab Validation

The student should be able to demonstrate this complete workflow:

```text
1. dotnet --version
           ↓
2. dotnet run
           ↓
3. Dockerfile
           ↓
4. docker build
           ↓
5. docker images
           ↓
6. docker run
           ↓
7. docker ps
           ↓
8. http://localhost:8080
           ↓
9. docker tag
           ↓
10. docker login
           ↓
11. docker push
           ↓
12. Docker Hub
           ↓
13. docker pull
           ↓
14. docker run
           ↓
15. Web API successfully running
```

## Lab Success Criteria

The lab is complete when all four tests succeed:

```text
Local .NET Application
        ✓

Local Docker Container
        ✓

Image uploaded to Docker Hub
        ✓

Docker Hub image pulled and executed
        ✓
```

## Final DevOps Architecture

```text
               Developer
                   │
                   ▼
              VS Code
          + GitHub Copilot
                   │
                   ▼
              Lab02API
             Program.cs
                   │
                   ▼
             .NET 10 SDK
                   │
                   ▼
              Dockerfile
                   │
             docker build
                   │
                   ▼
         ┌──────────────────┐
         │   Docker Image   │
         │  lab02api:1.0    │
         └────────┬─────────┘
                  │
           docker run
                  │
                  ▼
         ┌──────────────────┐
         │ Docker Container │
         │    Port 8080     │
         └──────────────────┘
                  │
                  ▼
       http://localhost:8080


         Docker Image
               │
          docker tag
               │
          docker push
               │
               ▼
      ┌─────────────────────┐
      │     Docker Hub      │
      │ username/lab02api   │
      │   :1.0 / :latest    │
      └─────────────────────┘
               │
          docker pull
               │
               ▼
        Other Computer /
        Server / Cloud
```

My preference for teaching this lab is to **avoid Kubernetes, Docker Compose, CI/CD and cloud deployment initially**. Get students comfortable with `dotnet run → docker build → docker run → docker push` first; that gives them the mental model they need before introducing pipelines and orchestration.

[1]: https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core?utm_source=chatgpt.com "NET and .NET Core official support policy"
[2]: https://learn.microsoft.com/en-us/dotnet/core/install/windows?utm_source=chatgpt.com "Install .NET on Windows"
[3]: https://docs.docker.com/desktop/setup/install/windows-install/?utm_source=chatgpt.com "Install Docker Desktop on Windows"
[4]: https://code.visualstudio.com/updates/v1_116?utm_source=chatgpt.com "Visual Studio Code 1.116"
[5]: https://code.visualstudio.com/docs/copilot/overview?utm_source=chatgpt.com "GitHub Copilot in VS Code"
[6]: https://learn.microsoft.com/en-us/dotnet/core/docker/introduction?utm_source=chatgpt.com "Introduction to .NET and Docker"
[7]: https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/?utm_source=chatgpt.com "Build, tag, and publish an image"
[8]: https://docs.docker.com/reference/cli/docker/image/push/?utm_source=chatgpt.com "docker image push"
