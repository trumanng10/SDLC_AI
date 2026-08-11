# Lab 19 — Generate a Jenkinsfile with AI and Build Validation Stages

## 1. What is Lab 19?

Until now, you have been doing things manually:

```text
Open PowerShell

       ↓

dotnet restore

       ↓

dotnet build

       ↓

dotnet test

       ↓

Check PASS / FAIL
```

Lab 19 says:

> Instead of a developer manually running these commands every time, let **Jenkins run them automatically in a defined sequence**.

So:

```text
                   BEFORE LAB 19

Developer
    ↓
PowerShell
    ↓
dotnet restore
    ↓
dotnet build
    ↓
dotnet test


                   AFTER LAB 19

Developer
    ↓
Git Push
    ↓
Jenkins
    ↓
┌───────────────┐
│   Checkout    │
├───────────────┤
│   Restore     │
├───────────────┤
│   Build       │
├───────────────┤
│   Test        │
└───────────────┘
    ↓
SUCCESS / FAILURE
```

That is **Continuous Integration (CI)**.

---

# 2. How Labs 19 and 20 are different

This is important.

```text
LAB 19
───────────────────────────────
Jenkins

Checkout
   ↓
Restore
   ↓
Build
   ↓
Test

Question:
"Can my application build and pass tests
automatically?"


LAB 20
───────────────────────────────
Jenkins + SonarQube

Checkout
   ↓
Restore
   ↓
Build/Test
   ↓
SonarQube Analysis
   ↓
Quality Gate

Question:
"Does my application also meet our
quality requirements?"
```

So **do not add SonarQube to Lab 19 yet**.

Lab 19 is intentionally simpler.

---

# 3. What is Jenkins?

Jenkins is an automation server.

Think of Jenkins as a robot that can execute the same engineering workflow every time:

```text
JENKINS

"When new code arrives..."

        ↓

Get the code

        ↓

Restore dependencies

        ↓

Compile it

        ↓

Run tests

        ↓

Tell us:

PASS or FAIL
```

Jenkins Pipeline is its mechanism for representing a build/delivery process as stages and steps. Jenkins recommends keeping the Pipeline definition in a `Jenkinsfile` in source control so it can be reviewed and versioned with the application. ([Jenkins][1])

---

# 4. What is a Pipeline?

A **Pipeline** is simply the workflow Jenkins should execute.

For Lab 19:

```text
PIPELINE

Stage 1
CHECKOUT
     ↓

Stage 2
RESTORE
     ↓

Stage 3
BUILD
     ↓

Stage 4
TEST
     ↓

SUCCESS
```

Jenkins calls these large sections:

```text
STAGES
```

Inside each stage are:

```text
STEPS
```

For example:

```text
STAGE
Build

    ↓

STEP
dotnet build
```

Jenkins documents `stage` as a distinct part of the Pipeline—such as Build or Test—and a `step` as an individual action Jenkins performs. ([Jenkins][1])

---

# 5. What is a Jenkinsfile?

A:

```text
Jenkinsfile
```

is just a **text file containing Jenkins Pipeline instructions**.

It normally sits in the root of your Git repository:

```text
AI_SDLC_Labs
│
├── Jenkinsfile       ← Lab 19
│
├── Lab02Api
│   ├── Program.cs
│   ├── OrderService.cs
│   └── Lab02Api.csproj
│
└── Lab02Api.Tests
    ├── OrderServiceTests.cs
    └── Lab02Api.Tests.csproj
```

The file has **no `.txt` extension**.

Correct:

```text
Jenkinsfile
```

Wrong:

```text
Jenkinsfile.txt
```

Jenkins officially recommends storing the `Jenkinsfile` in source control with the project. ([Jenkins][2])

---

# 6. What language does Jenkinsfile use?

For beginners, use:

# **Declarative Pipeline**

Not advanced Scripted Pipeline.

A simple Jenkinsfile looks like:

```groovy
pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                echo 'Building application'

            }
        }

    }
}
```

Jenkins supports Declarative and Scripted Pipeline syntax; Declarative is deliberately more structured and opinionated and is easier to read for straightforward pipelines. ([Jenkins][3])

---

# 7. Understand the keywords first

### `pipeline`

```groovy
pipeline {
}
```

Means:

> Everything inside here defines my Jenkins Pipeline.

---

### `agent any`

```groovy
agent any
```

Means:

> Jenkins may execute this Pipeline on an available Jenkins agent capable of running the required commands.

For our beginner local installation:

```text
Jenkins computer
        =
Build computer
```

---

### `stages`

```groovy
stages {
}
```

Means:

> Here are all the major parts of my build.

---

### `stage`

```groovy
stage('Build') {
}
```

Means:

> Create a visible Jenkins stage named **Build**.

---

### `steps`

```groovy
steps {
}
```

Means:

> These are the commands Jenkins should execute inside this stage.

---

# 8. Do students need Jenkins installed?

You have two choices.

## Recommended classroom setup

Use:

```text
                    INSTRUCTOR

                  Jenkins Server
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
      Student 1     Student 2     Student 3

       Git Repo       Git Repo       Git Repo
```

Students only need:

```text
Browser
Git
.NET SDK
GitHub/Gitea access
Cursor / Copilot
```

This is the **easiest classroom arrangement**.

---

# 9. If every student must install Jenkins locally

Since your participants use Windows, I would use the **Windows Jenkins installer rather than Docker** for this beginner lab.

Current Jenkins Windows documentation describes its MSI installer as the simplest Windows installation path. A current native Jenkins Windows installation requires Java 21 or later. ([Jenkins][4])

So locally:

```text
Windows PC
│
├── Java 21+
├── Jenkins
├── Git
├── .NET SDK
└── Browser
```

Docker is **not required** for Lab 19.

---

# PART A — Verify the Existing Application First

## Step 1 — Open PowerShell

Go to:

```powershell
cd C:\AI_SDLC_Labs
```

Check your folders:

```powershell
dir
```

You should have something similar to:

```text
Lab02Api

Lab02Api.Tests

lab15-angular
```

Your actual folders are the source of truth.

---

# Step 2 — Build the .NET project manually

Run:

```powershell
dotnet restore .\Lab02Api\Lab02Api.csproj
```

Then:

```powershell
dotnet build .\Lab02Api\Lab02Api.csproj
```

You want:

```text
Build succeeded.
```

---

# Step 3 — Run the .NET tests manually

If you have:

```text
Lab02Api.Tests
```

run:

```powershell
dotnet test .\Lab02Api.Tests\Lab02Api.Tests.csproj
```

You want:

```text
Failed: 0
```

Use your actual test count.

---

# 10. Why are we doing this BEFORE Jenkins?

Because Jenkins should automate **commands you already know work**.

Bad approach:

```text
AI
 ↓
Invent Jenkins commands
 ↓
Pipeline fails
 ↓
Nobody knows whether:

Jenkins is wrong?

.NET is wrong?

Path is wrong?

AI invented something?
```

Better:

```text
STEP 1

Prove locally:

dotnet restore ✓
dotnet build   ✓
dotnet test    ✓


STEP 2

Put EXACTLY those commands
into Jenkins.
```

This is also exactly how the original Lab 19 was designed: document the verified local backend/frontend commands first, then use those commands as the source of truth for AI-generated Pipeline stages. 

---

# PART B — Record the Commands

## Step 4 — Create a build-command document

Create:

```text
BUILD_COMMANDS.md
```

Put your verified commands:

```markdown
# Verified Build Commands

## Backend Restore

dotnet restore .\Lab02Api\Lab02Api.csproj

Result:
PASS

## Backend Build

dotnet build .\Lab02Api\Lab02Api.csproj

Result:
PASS

## Backend Tests

dotnet test .\Lab02Api.Tests\Lab02Api.Tests.csproj

Result:
PASS
```

Do not write:

```text
Should work.
```

Write actual evidence:

```text
PASS
```

only after executing it.

---

# PART C — Understand Jenkins Before Using AI

## Step 5 — Start with the smallest Pipeline possible

Before building the actual project, let Jenkins do something trivial.

Create a new Pipeline job in Jenkins.

From:

```text
Jenkins Dashboard
```

click:

```text
New Item
```

Enter:

```text
Lab19-Hello-Jenkins
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

# Step 6 — Create a tiny Pipeline

Scroll to:

```text
Pipeline
```

Temporarily choose:

```text
Pipeline script
```

Enter:

```groovy
pipeline {

    agent any

    stages {

        stage('Hello') {

            steps {

                echo 'Hello from Jenkins'

            }
        }

    }
}
```

Click:

```text
Save
```

Then:

```text
Build Now
```

---

# Step 7 — View the result

Open:

```text
Build #1
    ↓
Console Output
```

You should find:

```text
Hello from Jenkins
```

and ideally:

```text
Finished: SUCCESS
```

Now students understand:

```text
Jenkins
   ↓
reads Pipeline
   ↓
executes stage
   ↓
executes step
   ↓
shows result
```

---

# PART D — Make Jenkins Run Windows Commands

Because our beginner Jenkins environment is Windows, use:

```groovy
bat 'command'
```

For example:

```groovy
stage('Check .NET') {

    steps {

        bat 'dotnet --version'

    }
}
```

Jenkins Pipeline examples use `bat` for Windows command execution and `sh` for Unix/Linux shell commands. ([Jenkins][2])

This distinction matters:

```text
WINDOWS AGENT

bat 'dotnet build'


LINUX AGENT

sh 'dotnet build'
```

For **this lab guide**, we'll assume a **Windows Jenkins agent**.

---

# PART E — Check Jenkins Can See .NET

## Step 8 — Modify the simple Pipeline

Use:

```groovy
pipeline {

    agent any

    stages {

        stage('Environment Check') {

            steps {

                bat 'dotnet --version'
                bat 'git --version'

            }
        }

    }
}
```

Run:

```text
Build Now
```

You should see actual versions.

If Jenkins says:

```text
'dotnet' is not recognized
```

then Jenkins cannot currently access the .NET SDK.

Don't generate a Jenkinsfile yet.

Fix the Jenkins environment first.

---

# PART F — Put the Project in Git

Jenkins should normally obtain source code from a repository.

Conceptually:

```text
Your PC
   ↓
Git Push
   ↓
GitHub / Gitea
   ↓
Jenkins Checkout
```

---

# Step 9 — Verify Git

On your development PC:

```powershell
cd C:\AI_SDLC_Labs
```

Run:

```powershell
git status
```

Then:

```powershell
git remote -v
```

You should know which repository Jenkins will use.

---

# Step 10 — Create the Lab 19 branch

The original lab uses a separate branch for the Pipeline exercise. 

Run:

```powershell
git checkout -b lab19-jenkinsfile-generation
```

---

# PART G — Create the Jenkinsfile with AI

Now AI finally enters the workflow.

## Step 11 — Do NOT ask AI this

Bad:

```text
Create a Jenkins pipeline for me.
```

Why?

AI may invent:

```text
Docker

Maven

npm

Azure

AWS

SonarQube

deployment

credentials
```

that your project doesn't need.

---

# Step 12 — Give AI verified commands

Use this prompt in Cursor:

```text
You are a Jenkins CI/CD Engineer.

I am a beginner learning Jenkins.

Create a minimal Declarative Jenkinsfile for my
existing Windows-based .NET project.

IMPORTANT:

Use ONLY the verified commands I provide.

Do NOT invent:

- credentials
- plugins
- Docker
- SonarQube
- deployment
- package managers
- build tools
- paths
- environment variables

VERIFIED LOCAL COMMANDS

Restore:

dotnet restore .\Lab02Api\Lab02Api.csproj

Build:

dotnet build .\Lab02Api\Lab02Api.csproj

Test:

dotnet test .\Lab02Api.Tests\Lab02Api.Tests.csproj

Requirements:

1. Use Declarative Pipeline syntax.
2. Assume a Windows Jenkins agent.
3. Use bat for Windows commands.
4. Create these stages:

   Checkout
   Environment Check
   Restore
   Build
   Test

5. Stop automatically if a command fails.
6. Do not add SonarQube yet.
7. Do not add deployment.
8. Keep the Jenkinsfile beginner-friendly.

Before generating the Jenkinsfile, explain what
each stage will do.
```

---

# PART H — Review AI's Plan Before Accepting Code

AI should propose something like:

```text
Checkout
    ↓
Obtain source repository

Environment Check
    ↓
Check dotnet version

Restore
    ↓
Restore NuGet dependencies

Build
    ↓
Compile Lab02Api

Test
    ↓
Run Lab02Api.Tests
```

Ask:

```text
Does this match our project?
```

If yes, continue.

If AI says:

```text
npm install
```

but this first Pipeline only handles your .NET application:

```text
REJECT IT
```

---

# PART I — Create the Jenkinsfile

At:

```text
C:\AI_SDLC_Labs
```

create a file:

```text
Jenkinsfile
```

A simple Windows version is:

```groovy
pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                checkout scm

            }
        }

        stage('Environment Check') {

            steps {

                bat 'dotnet --version'
                bat 'git --version'

            }
        }

        stage('Restore') {

            steps {

                bat 'dotnet restore .\\Lab02Api\\Lab02Api.csproj'

            }
        }

        stage('Build') {

            steps {

                bat 'dotnet build .\\Lab02Api\\Lab02Api.csproj --no-restore'

            }
        }

        stage('Test') {

            steps {

                bat 'dotnet test .\\Lab02Api.Tests\\Lab02Api.Tests.csproj'

            }
        }

    }
}
```

---

# 13. Understand this Jenkinsfile

The first line:

```groovy
pipeline {
```

means:

```text
Start Declarative Pipeline
```

---

This:

```groovy
agent any
```

means:

```text
Use an available Jenkins agent.
```

---

This:

```groovy
stage('Checkout')
```

means:

```text
STAGE 1
Get project code
```

---

This:

```groovy
checkout scm
```

means:

```text
Checkout the repository/revision
associated with this Jenkins Pipeline.
```

Jenkins documents `checkout scm` as a way for a Pipeline to check out the specific source revision associated with the job. ([Jenkins][2])

---

This:

```groovy
bat 'dotnet restore ...'
```

means:

```text
Windows PowerShell/cmd equivalent action:

dotnet restore
```

---

# PART J — Commit the Jenkinsfile

Run:

```powershell
git status
```

You should see:

```text
Jenkinsfile
```

Then:

```powershell
git add Jenkinsfile
```

Commit:

```powershell
git commit -m "Add Lab 19 Jenkins build pipeline"
```

Then push:

```powershell
git push -u origin lab19-jenkinsfile-generation
```

---

# PART K — Create the Real Jenkins Pipeline Job

## Step 14 — In Jenkins

Go to:

```text
Dashboard
    ↓
New Item
```

Name:

```text
Lab19-Lab02Api
```

Choose:

```text
Pipeline
```

Click:

```text
OK
```

---

# Step 15 — Configure Pipeline from Git

Scroll to:

```text
Pipeline
```

Change:

```text
Definition
```

to:

```text
Pipeline script from SCM
```

SCM:

```text
Git
```

Repository URL:

```text
[YOUR ACTUAL TRAINING REPOSITORY]
```

Credentials:

```text
[Select only if your repository requires them]
```

Branch:

```text
*/lab19-jenkinsfile-generation
```

Script Path:

```text
Jenkinsfile
```

Then:

```text
Save
```

---

# 16. Why "Pipeline script from SCM"?

Because we want:

```text
Git Repository
│
├── Source Code
├── Tests
└── Jenkinsfile
```

rather than:

```text
Jenkins UI
    ↓
Pipeline hidden inside Jenkins
```

Keeping it with the repository provides version history, reviewability and a shared source of truth. ([Jenkins][1])

---

# PART L — Run Your First Real CI Pipeline

## Step 17 — Click Build Now

Jenkins should start:

```text
Build #1
```

Watch the stages:

```text
Checkout
     ↓
Environment Check
     ↓
Restore
     ↓
Build
     ↓
Test
```

---

# Step 18 — Open Console Output

Go to:

```text
Build #1
    ↓
Console Output
```

Read what actually happened.

Do not merely look for a green icon.

You should find:

```text
dotnet restore
```

then:

```text
dotnet build
```

then:

```text
dotnet test
```

---

# PART M — Understand SUCCESS

If you see:

```text
Finished: SUCCESS
```

that means every required command completed successfully.

Conceptually:

```text
Checkout ✓
     ↓
Environment ✓
     ↓
Restore ✓
     ↓
Build ✓
     ↓
Test ✓
     ↓
PIPELINE SUCCESS
```

---

# PART N — Understand FAILURE

Suppose:

```text
Checkout ✓
Restore  ✓
Build    ✓
Test     ✗
```

Jenkins should report:

```text
FAILURE
```

That is **good CI behavior**.

Jenkins should **not continue pretending everything is okay** after a failed required build/test command. The original Lab 19 specifically requires that failed build or test commands stop the Pipeline. 

---

# PART O — Deliberately Make a Test Fail

This is one of the best demonstrations for beginners.

Temporarily modify one test.

For example:

Correct:

```csharp
Assert.Equal(160m, result);
```

Change temporarily to:

```csharp
Assert.Equal(150m, result);
```

Commit it on the training branch:

```powershell
git add .
git commit -m "Introduce controlled Lab 19 test failure"
git push
```

Run Jenkins again.

You should get:

```text
Checkout
   ✓

Restore
   ✓

Build
   ✓

Test
   ✗

Pipeline
   ✗ FAILURE
```

Now students see what Jenkins is actually protecting.

---

# PART P — Restore the Correct Test

Change:

```csharp
150m
```

back to:

```csharp
160m
```

Commit:

```powershell
git add .
git commit -m "Restore passing test after CI demonstration"
git push
```

Run Jenkins.

Now expect:

```text
SUCCESS
```

This proves:

```text
Bad code/test
      ↓
Pipeline RED

Correct code/tests
      ↓
Pipeline GREEN
```

---

# PART Q — Ask AI to Analyze a Jenkins Failure

When Jenkins fails, **do not paste “Jenkins failed, fix it.”**

Use the real evidence:

```text
You are helping me troubleshoot a beginner Jenkins
Pipeline.

Do NOT modify anything yet.

Pipeline Stage:
[ACTUAL FAILED STAGE]

Command:
[ACTUAL COMMAND]

Exit Code:
[ACTUAL EXIT CODE IF SHOWN]

First Relevant Error:
[PASTE ERROR]

Jenkinsfile:
[PASTE RELEVANT STAGE]

Determine:

1. What failed.
2. Whether the problem is:
   - Jenkins configuration
   - Git checkout
   - missing tool
   - incorrect path
   - restore failure
   - build failure
   - test failure
3. Three possible root-cause hypotheses.
4. Evidence for each.
5. The first verification step.

Do not recommend rewriting the entire Jenkinsfile.
```

---

# PART R — Don't Fix the Last Error First

Suppose Jenkins output contains:

```text
Error 1
dotnet not recognized

Error 2
Build stage failed

Error 3
Pipeline failed
```

The real starting point is:

```text
FIRST ACTIONABLE ERROR

'dotnet' is not recognized
```

not:

```text
Pipeline failed
```

This prepares students for Lab 21.

---

# PART S — Optional Angular Build

Only add frontend stages **if the Angular project is actually part of the repository and its commands have already been verified locally**.

The source lab explicitly says backend and frontend build/test stages should be included **as applicable**, not invented indiscriminately. 

For example, first verify locally:

```powershell
cd C:\AI_SDLC_Labs\lab15-angular
npm ci
npx ng build
npx ng test --no-watch
```

Only if these actually work should AI be allowed to add them.

Then your Pipeline could become:

```text
Checkout
    ↓
Backend Restore
    ↓
Backend Build
    ↓
Backend Test
    ↓
Frontend Install
    ↓
Frontend Build
    ↓
Frontend Test
```

Do **not** complicate the first exercise with Angular if the class is still learning Jenkins fundamentals.

---

# PART T — Add `post` Actions Later

Once the Pipeline works, you may add a simple:

```groovy
post {

    success {

        echo 'Pipeline completed successfully.'

    }

    failure {

        echo 'Pipeline failed. Review the first failing stage.'

    }

}
```

Result:

```text
SUCCESS
   ↓
"Pipeline completed successfully."


FAILURE
   ↓
"Pipeline failed.
 Review the first failing stage."
```

Keep it simple.

---

# PART U — Final Beginner Jenkinsfile

A reasonable Lab 19 version is:

```groovy
pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                checkout scm

            }
        }

        stage('Environment Check') {

            steps {

                bat 'dotnet --version'
                bat 'git --version'

            }
        }

        stage('Restore') {

            steps {

                bat 'dotnet restore .\\Lab02Api\\Lab02Api.csproj'

            }
        }

        stage('Build') {

            steps {

                bat 'dotnet build .\\Lab02Api\\Lab02Api.csproj --no-restore'

            }
        }

        stage('Test') {

            steps {

                bat 'dotnet test .\\Lab02Api.Tests\\Lab02Api.Tests.csproj'

            }
        }

    }

    post {

        success {

            echo 'Lab 19 Pipeline PASSED.'

        }

        failure {

            echo 'Lab 19 Pipeline FAILED. Review the failed stage.'

        }

    }
}
```

This is enough for Lab 19.

**No SonarQube yet.**

**No deployment yet.**

**No Docker required.**

---

# PART V — Ask AI to Review the Jenkinsfile

Use:

```text
Act as a Senior Jenkins Engineer.

Review this Jenkinsfile.

Do NOT modify it.

For every stage explain:

1. Purpose
2. Command executed
3. Expected result
4. What would make it fail

Then check for:

- invented tools
- wrong paths
- duplicated commands
- unnecessary plugins
- hard-coded credentials
- secrets
- commands that were not verified locally
- stages that would continue after a required failure

Classify each issue:

MUST FIX
SHOULD FIX
OPTIONAL

Do not add SonarQube or deployment.
```

---

# PART W — Create the Lab 19 Report

Create:

```text
JENKINS_PIPELINE_REPORT.md
```

Use:

```text
Create JENKINS_PIPELINE_REPORT.md using ONLY
actual Lab 19 evidence.

Structure:

# Lab 19 — Jenkins Build Validation Pipeline

## 1. Application

Lab02Api

## 2. Jenkins Environment

Jenkins:
[actual]

Agent OS:
[actual]

Repository:
[actual]

Branch:
lab19-jenkinsfile-generation

## 3. Verified Local Commands

Restore:
[...]

Result:
[...]

Build:
[...]

Result:
[...]

Test:
[...]

Result:
[...]

## 4. Jenkinsfile

Pipeline Type:
Declarative

Stages:
- Checkout
- Environment Check
- Restore
- Build
- Test

## 5. Pipeline Execution

Build Number:
[...]

Checkout:
PASS / FAIL

Environment Check:
PASS / FAIL

Restore:
PASS / FAIL

Build:
PASS / FAIL

Test:
PASS / FAIL

Overall:
SUCCESS / FAILURE

## 6. Controlled Failure Demonstration

Change:
[...]

Expected Failure:
[...]

Actual Failed Stage:
[...]

Jenkins Result:
[...]

## 7. Recovery

Change reverted:
Yes / No

Final Pipeline:
[...]

## 8. AI Review

Useful Finding:
[...]

Rejected AI Suggestion:
[...]

## 9. Problems Encountered

Problem:
[...]

Root Cause:
[...]

Resolution:
[...]

## 10. Conclusion

Explain what Jenkins now automates.

Do not invent execution results.
Do not include passwords or tokens.
```

---

# Lab 19 Completion Checklist

Before declaring Lab 19 complete:

* [ ] Student understands what Jenkins is.
* [ ] Student understands CI.
* [ ] Student understands Pipeline.
* [ ] Student understands stage.
* [ ] Student understands step.
* [ ] Student understands Jenkinsfile.
* [ ] Jenkins is available.
* [ ] Jenkins can run a simple `echo`.
* [ ] Jenkins can run `dotnet --version`.
* [ ] Git is accessible to Jenkins.
* [ ] .NET commands work locally first.
* [ ] `BUILD_COMMANDS.md` created.
* [ ] Lab 19 branch created.
* [ ] AI received only verified commands.
* [ ] AI did not invent Docker/SonarQube/deployment.
* [ ] Declarative `Jenkinsfile` created.
* [ ] Jenkinsfile committed to Git.
* [ ] Jenkins Pipeline configured from SCM.
* [ ] Checkout succeeds.
* [ ] Restore succeeds.
* [ ] Build succeeds.
* [ ] Tests succeed.
* [ ] Controlled failing test causes Jenkins to fail.
* [ ] Test restored.
* [ ] Final Jenkins Pipeline succeeds.
* [ ] No passwords/secrets in Jenkinsfile.
* [ ] `JENKINS_PIPELINE_REPORT.md` created.

# The easiest way to remember Lab 19

```text
                    DEVELOPER

                        │
                        ▼

                 VERIFIED COMMANDS

           restore → build → test

                        │
                        ▼

                       AI

         "Convert THESE commands only
              into a Jenkinsfile"

                        │
                        ▼

                   JENKINSFILE

                        │
                        ▼

                     JENKINS

                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       RESTORE        BUILD          TEST
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                 SUCCESS / FAILURE
```

And now the transition to Lab 20 becomes very clean:

```text
LAB 19

Git
 ↓
Jenkins
 ↓
Restore
 ↓
Build
 ↓
Test
 ↓
PASS


              then...


LAB 20

Git
 ↓
Jenkins
 ↓
Restore
 ↓
Build
 ↓
Test
 ↓
SonarQube
 ↓
Quality Gate
 ↓
PASS / FAIL
```

For beginners, I would teach **Lab 19 with only the .NET backend first**. Once everyone gets a green Jenkins pipeline and deliberately sees it turn red from a failing test, Jenkins makes sense. Only after that should you add Angular, SonarQube, Quality Gates, artifacts, parallel stages, or more advanced CI features.

[1]: https://www.jenkins.io/doc/book/pipeline/?utm_source=chatgpt.com "Pipeline"
[2]: https://www.jenkins.io/doc/book/pipeline/jenkinsfile/?utm_source=chatgpt.com "Using a Jenkinsfile"
[3]: https://www.jenkins.io/doc/book/pipeline/syntax/?utm_source=chatgpt.com "Pipeline Syntax"
[4]: https://www.jenkins.io/doc/book/installing/windows/?utm_source=chatgpt.com "Windows"
