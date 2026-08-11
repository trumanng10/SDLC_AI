
The original Lab 20 is **“Integrate SonarQube Quality Gates into Jenkins.”** Its intended outcome is a Jenkins pipeline that performs SonarQube analysis, makes the Quality Gate visible/enforceable, safely demonstrates a failing condition, and documents how Jenkins, SonarQube, credentials, and the Quality Gate connect. 

# Lab 20 — Integrate SonarQube Quality Gates into Jenkins

## 1. What does Lab 20 actually mean?

First remember the previous labs:

```text
LAB 10
SonarQube
    ↓
WHAT quality problems exist?


LAB 11
AI + Developer
    ↓
WHY do the problems exist?


LAB 12
AI + Developer
    ↓
HOW do we remediate and verify them?


LAB 19
Jenkins
    ↓
Automatically
BUILD + TEST


LAB 20
Jenkins + SonarQube
    ↓
Automatically
BUILD + SCAN + QUALITY GATE
```

So Lab 20 means:

> **Every time Jenkins builds our application, Jenkins should also ask SonarQube to analyze the source code and determine whether its required quality standard is satisfied.**

---

# 2. The final workflow

Before Lab 20:

```text
Developer
    ↓
Git Repository
    ↓
Jenkins
    ↓
Build
    ↓
Test
    ↓
PASS
```

After Lab 20:

```text
Developer
    ↓
Git Repository
    ↓
Jenkins
    │
    ├── Checkout
    │
    ├── Restore
    │
    ├── Build
    │
    ├── Test
    │
    ├── SonarQube Analysis
    │         ↓
    │     SonarQube
    │         ↓
    │    Quality Gate
    │
    └── Quality Gate Result
            │
       ┌────┴────┐
       ▼         ▼
      PASS      FAIL
       │         │
       ▼         ▼
    Continue    Stop
```

The SonarQube Jenkins integration provides Pipeline steps such as `withSonarQubeEnv` for analysis configuration and `waitForQualityGate` for waiting on the resulting Quality Gate. ([SonarSource Documentation][1])

---

# 3. What tools are involved?

Students need to understand **four different Sonar things**.

| Tool                                 | Where           | Purpose                                             |
| ------------------------------------ | --------------- | --------------------------------------------------- |
| **SonarQube for IDE**                | VS Code         | Shows issues while developer codes                  |
| **SonarQube Server / Cloud**         | Central service | Stores project analysis and calculates Quality Gate |
| **SonarScanner for .NET**            | Jenkins agent   | Scans/builds the .NET project                       |
| **SonarQube Scanner Jenkins plugin** | Jenkins         | Connects Jenkins to SonarQube                       |

Think:

```text
Developer PC
────────────────────────
VS Code
     ↓
SonarQube for IDE


Jenkins
────────────────────────
SonarQube Scanner Plugin
        +
SonarScanner for .NET
           ↓

SonarQube Server
────────────────────────
Analysis
Issues
Metrics
Quality Gate
```

The Jenkins SonarQube Scanner plugin supports SonarQube Server, SonarQube Cloud and Community Build and can centrally configure scanner integration for Jenkins jobs. ([Jenkins Plugins][2])

---

# 4. Do students need Docker?

**No.**

For our revised course setup I recommend:

```text
STUDENT
────────────────────
VS Code
Git
.NET SDK


CENTRAL / INSTRUCTOR
────────────────────
Git Repository
Jenkins
SonarQube
```

Students do not need another SonarQube Docker instance on every laptop.

---

# 5. Starting point

Before Lab 20 you should already have the Lab 19 Jenkins pipeline working.

Example:

```text
Jenkins Pipeline

Checkout
    ↓
Restore
    ↓
Build
    ↓
Test
    ↓
SUCCESS
```

The original Lab 20 explicitly assumes both a working Jenkins pipeline from Lab 19 and a working SonarQube project/scan from Lab 10. 

Do not start SonarQube integration while the existing Jenkins build is already broken.

---

# PART A — Understand Quality Gate

## Step 1 — What is a Quality Gate?

A Quality Gate is effectively a decision:

```text
SonarQube scans code
        ↓
Checks configured
quality conditions
        ↓
Are conditions satisfied?
        │
     ┌──┴──┐
     ▼     ▼
    YES    NO
     │      │
    PASS   FAIL
```

It is **not the same thing as compilation**.

For example:

```text
dotnet build
     ↓
PASS
```

means:

> The program can compile.

But:

```text
Quality Gate
     ↓
FAIL
```

means:

> The analyzed code did not satisfy one or more configured quality conditions.

SonarQube Quality Gates evaluate analysis results according to configured conditions rather than simply determining whether the application compiled. ([SonarSource Documentation][1])

---

# PART B — Install Jenkins SonarQube Plugin

This section normally requires **Jenkins administrator permission**.

## Step 2 — Open Jenkins

Open your Jenkins Dashboard.

For example:

```text
Jenkins Dashboard
```

Then:

```text
Manage Jenkins
```

---

# Step 3 — Open Plugins

Depending on the Jenkins UI version, go to:

```text
Manage Jenkins
    ↓
Plugins
```

or:

```text
Manage Jenkins
    ↓
Manage Plugins
```

Search:

```text
SonarQube Scanner
```

Make sure the publisher/plugin is the SonarQube integration.

Install:

```text
SonarQube Scanner
```

SonarSource's Jenkins setup documentation instructs administrators to install the **SonarQube Scanner** plugin before configuring the SonarQube server connection. ([SonarSource Documentation][3])

---

# Step 4 — Understand what this plugin does

The plugin does **not replace SonarQube**.

It connects:

```text
JENKINS
   │
   │ plugin
   ▼
SONARQUBE
```

Without it:

```text
Jenkins       SonarQube
   ?              ?
```

With it:

```text
Jenkins
   │
   ├── knows SonarQube URL
   ├── knows which credential to use
   ├── provides Sonar environment
   └── receives Quality Gate status
            ↓
        SonarQube
```

---

# PART C — Create a SonarQube Token

## Step 5 — Open SonarQube

Log in to the instructor/shared SonarQube server.

Go to your user security/token area.

Generate an analysis/access token according to your SonarQube setup.

The official Jenkins setup stores this token in Jenkins as a **Secret Text credential**, rather than writing the token directly into the source repository or Jenkinsfile. ([SonarSource Documentation][4])

---

# Step 6 — Important security rule

Never put this into:

```groovy
SONAR_TOKEN = "abc123secret"
```

inside:

```text
Jenkinsfile
```

Also never put it in:

```text
Program.cs
appsettings.json
GitHub
```

Instead:

```text
Sonar Token
     ↓
Jenkins Credentials
     ↓
SonarQube Plugin
```

---

# PART D — Store the Token in Jenkins

## Step 7 — Open Jenkins Credentials

Go to:

```text
Jenkins Dashboard
        ↓
Credentials
        ↓
System
        ↓
Global credentials
```

Choose:

```text
Add Credentials
```

---

# Step 8 — Configure the credential

Use:

```text
Kind:
Secret text

Secret:
[PASTE SONARQUBE TOKEN]

ID:
sonarqube-token

Description:
SonarQube Training Token
```

Then:

```text
Save
```

The SonarQube Jenkins setup documentation uses a globally stored **Secret Text** credential for the SonarQube token. ([SonarSource Documentation][4])

---

# PART E — Tell Jenkins Where SonarQube Is

## Step 9 — Open Jenkins system configuration

Go to:

```text
Manage Jenkins
      ↓
System
```

Depending on the Jenkins version you may see:

```text
Configure System
```

Look for:

```text
SonarQube servers
```

---

# Step 10 — Add SonarQube

Click:

```text
Add SonarQube
```

Use something simple:

```text
Name:

SonarQube-Training
```

Then:

```text
Server URL:

[YOUR ACTUAL SONARQUBE URL]
```

Then select:

```text
Server authentication token:

sonarqube-token
```

Save.

The plugin's global setup requires a logical SonarQube installation name, server URL, and the Jenkins credential containing the token. ([SonarSource Documentation][4])

---

# Step 11 — Remember the name

This name is very important:

```text
SonarQube-Training
```

because later your Jenkinsfile will say:

```groovy
withSonarQubeEnv('SonarQube-Training')
```

If Jenkins configuration says:

```text
Sonar-Training
```

but the Jenkinsfile says:

```text
SonarQube-Training
```

the pipeline will not refer to the configured installation.

Use the **actual configured name**.

---

# PART F — Install SonarScanner for .NET in Jenkins

This is also normally instructor/admin setup.

## Step 12 — Open Tools configuration

Go to:

```text
Manage Jenkins
      ↓
Tools
```

or in older Jenkins installations:

```text
Manage Jenkins
      ↓
Global Tool Configuration
```

Look for:

```text
SonarScanner for MSBuild
```

Do not be confused by that old label.

Sonar renamed **SonarScanner for MSBuild** to **SonarScanner for .NET**, although parts of Jenkins configuration/documentation can still use the older MSBuild wording. ([SonarSource Documentation][4])

---

# Step 13 — Add scanner

Click:

```text
Add SonarScanner for MSBuild
```

Give it an easy name:

```text
SonarScanner-for-.NET
```

Choose:

```text
Install automatically
```

and use an available supported version.

Then save.

The Jenkins integration can automatically provision SonarScanner for .NET on Jenkins executors rather than requiring each participant to manually install it on the Jenkins machine. ([SonarSource Documentation][4])

---

# PART G — Understand the .NET Sonar workflow

This is very important.

A .NET Sonar scan is not simply:

```text
Build
↓
Sonar Scan
```

The scanner surrounds the build:

```text
SONAR BEGIN
     ↓
.NET BUILD
     ↓
SONAR END
     ↓
Send Analysis to SonarQube
```

Remember:

> **BEGIN → BUILD → END**

---

# PART H — Modify the Lab 19 Jenkinsfile

## Step 14 — Open your repository

Open:

```text
C:\AI_SDLC_Labs
```

or your Git repository in Cursor/VS Code.

Open:

```text
Jenkinsfile
```

---

# Step 15 — Do NOT ask AI to rewrite everything

Bad:

```text
Rewrite my Jenkinsfile and add SonarQube.
```

Better:

```text
Review my existing Lab 19 Jenkinsfile.

Do NOT modify it yet.

I need to integrate:

SonarQube installation:
SonarQube-Training

Project key:
lab02api

Project type:
.NET

Jenkins OS:
Windows

Explain:

1. Where Sonar analysis must start.
2. Which build must occur between Sonar begin/end.
3. Where Quality Gate checking should occur.
4. Which existing stages can remain unchanged.
5. What Jenkins prerequisites must already exist.

Do not invent credentials or plugin names.
```

---

# PART I — Add SonarQube Analysis Stage

A beginner-friendly Windows Jenkins Pipeline could contain this pattern:

```groovy
stage('SonarQube Analysis') {
    steps {
        script {
            def scannerHome =
                tool 'SonarScanner-for-.NET'

            withSonarQubeEnv('SonarQube-Training') {

                bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"lab02api\""

                bat "dotnet build Lab02Api\\Lab02Api.csproj --no-incremental"

                bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
            }
        }
    }
}
```

The exact executable/path depends on the Jenkins scanner installation, so use the scanner name actually configured in Jenkins. The official Jenkins/SonarQube integration provides the `withSonarQubeEnv` block and supports SonarScanner for .NET analysis from Jenkins. ([Jenkins Plugins][2])

---

# Step 16 — Understand the code

This:

```groovy
tool 'SonarScanner-for-.NET'
```

means:

```text
Jenkins:
"Give me the SonarScanner
configured with this name."
```

---

This:

```groovy
withSonarQubeEnv(
    'SonarQube-Training'
)
```

means:

```text
Jenkins
    ↓
Load configured:
SonarQube URL
Credentials
Environment
```

The plugin injects the configured SonarQube connection into the build environment for scanner execution. ([Jenkins Plugins][2])

---

This:

```groovy
begin /k:"lab02api"
```

means:

```text
Start analysis

Project Key:
lab02api
```

Then:

```groovy
dotnet build
```

builds the application while analysis is active.

Then:

```groovy
end
```

completes the scan and submits the analysis information.

---

# PART J — First Run: Analysis Only

Before adding Quality Gate enforcement, I recommend beginners first prove:

```text
Jenkins
    ↓
Sonar Analysis
    ↓
SonarQube receives it
```

Run:

```text
Build Now
```

or trigger your Jenkins Pipeline normally.

---

# Step 17 — Watch Console Output

Open:

```text
Jenkins
   ↓
Build
   ↓
Console Output
```

Look for your stage:

```text
SonarQube Analysis
```

You should see scanner activity.

Do not immediately care whether the Quality Gate passes.

First prove:

```text
Did Jenkins actually send analysis?
```

---

# Step 18 — Open SonarQube

Go to:

```text
SonarQube
    ↓
Projects
    ↓
Lab02Api
```

Verify that you see a **new analysis** corresponding to the Jenkins run.

This was also a specific objective of the original Lab 20: Jenkins should run analysis against the intended SonarQube project rather than merely producing a successful pipeline message. 

---

# PART K — Now Add the Quality Gate

Once analysis works, add another stage.

```groovy
stage('Quality Gate') {
    steps {
        timeout(
            time: 10,
            unit: 'MINUTES'
        ) {
            waitForQualityGate(
                abortPipeline: true
            )
        }
    }
}
```

`waitForQualityGate` waits for SonarQube to complete its analysis. With `abortPipeline: true`, Jenkins can stop the pipeline when the resulting Quality Gate fails. ([SonarSource Documentation][1])

---

# PART L — But One More Thing Is Required: Webhook

This is the part beginners usually don't understand.

Jenkins runs:

```text
SonarQube Scan
```

Then SonarQube processes it.

How does Jenkins know SonarQube is finished?

Through a:

```text
WEBHOOK
```

Think:

```text
Jenkins
   │
   │ "Please analyze this."
   ▼
SonarQube
   │
   │ analysis completed
   │
   │ webhook
   ▼
Jenkins
   │
   ▼
Quality Gate result
```

For `waitForQualityGate`, SonarSource documents a SonarQube webhook back to the Jenkins SonarQube endpoint as a required part of pipeline-pause integration. ([SonarSource Documentation][1])

---

# PART M — Configure the Webhook

This is normally instructor/admin setup.

## Step 19 — Identify the Jenkins address

Suppose Jenkins is:

```text
http://jenkins-training.example/
```

The SonarQube webhook is:

```text
http://jenkins-training.example/sonarqube-webhook/
```

**The final `/` matters in the documented endpoint form.** ([SonarSource Documentation][1])

Do not blindly use:

```text
localhost:8080
```

unless SonarQube can actually reach that Jenkins instance.

---

# Step 20 — Add webhook in SonarQube

In SonarQube, open the project administration/webhook configuration.

Add:

```text
Name:

Jenkins
```

URL:

```text
YOUR-JENKINS-URL/sonarqube-webhook/
```

Save.

---

# PART N — Full Jenkinsfile Concept

At this point the pipeline concept becomes:

```groovy
pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                bat 'dotnet restore Lab02Api\\Lab02Api.csproj'
            }
        }

        stage('Test') {
            steps {
                bat 'dotnet test Lab02Api.Tests\\Lab02Api.Tests.csproj'
            }
        }

        stage('SonarQube Analysis') {

            steps {

                script {

                    def scannerHome =
                        tool 'SonarScanner-for-.NET'

                    withSonarQubeEnv(
                        'SonarQube-Training'
                    ) {

                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"lab02api\""

                        bat "dotnet build Lab02Api\\Lab02Api.csproj --no-incremental"

                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }

        stage('Quality Gate') {

            steps {

                timeout(
                    time: 10,
                    unit: 'MINUTES'
                ) {

                    waitForQualityGate(
                        abortPipeline: true
                    )
                }
            }
        }
    }
}
```

Treat this as a **teaching pattern**. Paths and configured installation names must match your actual repository and Jenkins environment.

---

# PART O — Understand Pipeline Order

The workflow is now:

```text
CHECKOUT
    ↓
RESTORE
    ↓
TEST
    ↓
SONAR BEGIN
    ↓
BUILD
    ↓
SONAR END
    ↓
SonarQube processing
    ↓
QUALITY GATE
    │
 ┌──┴──┐
 ▼     ▼
PASS  FAIL
```

---

# PART P — Run the Complete Pipeline

## Step 21 — Commit Jenkinsfile

Run:

```powershell
git status
```

Then:

```powershell
git add Jenkinsfile
```

Then:

```powershell
git commit -m "Integrate SonarQube quality gate"
```

Then push to the training repository.

---

# Step 22 — Trigger Jenkins

Open the Jenkins Pipeline.

Click:

```text
Build Now
```

or trigger the build using your configured repository mechanism.

---

# Step 23 — Watch the stages

Ideally:

```text
Checkout
    ✓

Restore
    ✓

Test
    ✓

SonarQube Analysis
    ✓

Quality Gate
    ?
```

---

# PART Q — Quality Gate PASS

If SonarQube returns:

```text
Quality Gate
     ↓
PASSED
```

Jenkins continues.

Record:

```text
SonarQube Analysis:
SUCCESS

Quality Gate:
PASSED

Pipeline:
SUCCESS
```

---

# PART R — Quality Gate FAIL

Suppose:

```text
Quality Gate
     ↓
FAILED
```

With the Pipeline configured to abort on failure:

```text
Jenkins Pipeline
       ↓
STOPPED / FAILED
```

This is **not a Jenkins defect**.

It means:

```text
Jenkins works ✓

SonarQube works ✓

Quality control works ✓

Code failed required quality condition ✗
```

This distinction is extremely important.

---

# PART S — Ask AI to Explain a Failed Gate

Do not tell Copilot:

```text
Make Jenkins green.
```

Use:

```text
You are helping me investigate a Jenkins
Quality Gate failure.

Do NOT modify code or pipeline yet.

Jenkins Stage:
Quality Gate

Jenkins Result:
FAILED

SonarQube Quality Gate:
FAILED

Actual failing conditions:
[PASTE FROM SONARQUBE]

Relevant metrics:
[PASTE ACTUAL VALUES]

Explain:

1. Did Jenkins itself fail?
2. Did the build fail?
3. Did SonarQube analysis fail?
4. Or did the Quality Gate reject the analyzed code?
5. Which exact condition caused the failure?
6. What engineering action would address it?

Do not recommend weakening the Quality Gate
merely to make Jenkins green.
```

---

# PART T — Do Not Lower the Gate Just to Pass

Wrong:

```text
Quality Gate FAIL
       ↓
Lower threshold
       ↓
PASS
```

That doesn't necessarily improve the code.

Better:

```text
Quality Gate FAIL
       ↓
Identify exact condition
       ↓
Find underlying issue
       ↓
Remediate
       ↓
Build
       ↓
Test
       ↓
Rescan
       ↓
Quality Gate
```

---

# PART U — Deliberately Test a Failed Quality Gate

The original Lab 20 specifically calls for proving the pipeline responds when the configured quality criteria fail. 

Do this **only in a training branch**.

Create:

```powershell
git checkout -b lab20-quality-gate-test
```

---

# Step 24 — Look at the actual Quality Gate first

In SonarQube:

```text
Lab02Api
   ↓
Quality Gate
```

Determine:

```text
Which actual condition can
we safely demonstrate?
```

Do **not** invent a condition.

For example, the instructor may prepare code that safely triggers one rule/condition already enforced by the training Quality Gate.

---

# Step 25 — Commit the controlled test change

```powershell
git add .
git commit -m "Add controlled quality gate test issue"
```

Push the branch.

Trigger Jenkins.

Expected flow:

```text
Build
   ✓

Tests
   ✓

Sonar Scan
   ✓

Quality Gate
   ✗

Jenkins Pipeline
   ✗
```

This is a **successful lab demonstration**.

The pipeline correctly blocked unacceptable analyzed code.

---

# PART V — Revert the Test Issue

After proving the gate works:

```powershell
git revert HEAD
```

or remove the intentional issue properly.

Commit:

```powershell
git add .
git commit -m "Remove quality gate test issue"
```

Push.

Run Jenkins again.

You want:

```text
Build
   ✓

Test
   ✓

Sonar Analysis
   ✓

Quality Gate
   ✓
```

---

# PART W — Understand Build Failure vs Quality Gate Failure

Students should be able to distinguish all of these.

| Situation            | Build | Sonar                | Gate | Meaning                   |
| -------------------- | ----- | -------------------- | ---- | ------------------------- |
| Compilation error    | ❌     | Usually not complete | —    | Code doesn't build        |
| Test failure         | ✅     | Depends on pipeline  | —    | Behavior test failed      |
| Scanner/config error | ✅     | ❌                    | —    | Sonar integration problem |
| Quality Gate failure | ✅     | ✅                    | ❌    | Code fails quality policy |
| Full success         | ✅     | ✅                    | ✅    | CI quality checks passed  |

This table is one of the most important outputs of Lab 20.

---

# PART X — Common Problem: `withSonarQubeEnv` Not Found

If Jenkins says something like:

```text
No such DSL method:
withSonarQubeEnv
```

investigate whether:

```text
SonarQube Scanner plugin
```

is installed/enabled.

Do not change C# code.

---

# Common Problem: SonarQube Installation Not Found

Example:

```text
SonarQube installation
'SonarQube-Training'
does not exist
```

Compare:

```text
Jenkins configuration name

vs

Jenkinsfile name
```

They must refer to the same configured installation.

---

# Common Problem: Scanner Not Found

Example:

```text
SonarScanner-for-.NET
not found
```

Check:

```text
Manage Jenkins
      ↓
Tools
      ↓
SonarScanner
```

Again:

```text
Configured Tool Name

must equal

Jenkinsfile Tool Name
```

---

# Common Problem: Quality Gate Waits Forever

Check the webhook.

The Quality Gate wait depends on SonarQube being able to notify Jenkins through the configured `/sonarqube-webhook/` endpoint. ([SonarSource Documentation][1])

Investigate:

```text
SonarQube
      ↓
Webhook Delivery
      ↓
Can Sonar reach Jenkins?
```

Do not solve it with:

```text
sleep 60
```

---

# PART Y — Ask AI to Troubleshoot Safely

Use:

```text
You are a Jenkins + SonarQube troubleshooting expert.

Do NOT change configuration yet.

Pipeline Stage:
[STAGE]

Actual Jenkins Error:
[ERROR]

Jenkinsfile:
[RELEVANT PART]

Expected SonarQube Configuration:
[CONFIGURATION]

Generate three possible root-cause hypotheses.

For each provide:

1. Evidence
2. Confidence
3. One verification step
4. Recommended action only if confirmed

Distinguish between:

- Jenkins configuration issue
- SonarQube configuration issue
- Scanner issue
- Credential issue
- Network/webhook issue
- Source-code Quality Gate failure

Do not recommend random configuration changes.
```

This also prepares participants nicely for **Lab 21 — Troubleshoot and Optimize a Failing Jenkins Pipeline with AI**.

---

# PART Z — Create Lab 20 Report

Create:

```text
JENKINS_SONARQUBE_INTEGRATION.md
```

Use:

```text
Create JENKINS_SONARQUBE_INTEGRATION.md using
ONLY actual Lab 20 evidence.

Structure:

# Lab 20 — Jenkins + SonarQube Quality Gate

## 1. Environment

Jenkins:
[actual]

SonarQube:
[actual]

Repository:
[actual]

Project:
Lab02Api

## 2. Jenkins SonarQube Plugin

Installed:
Yes / No

## 3. SonarQube Connection

Jenkins Configuration Name:
[...]

Server:
[...]

Credential ID:
[...]

Do NOT include the secret/token.

## 4. Scanner

Scanner:
SonarScanner for .NET

Jenkins Tool Name:
[...]

## 5. Webhook

Configured:
Yes / No

Result:
[...]

Do NOT expose secrets.

## 6. Jenkins Pipeline Stages

Checkout:
Restore:
Build/Test:
SonarQube Analysis:
Quality Gate:

## 7. SonarQube Analysis Result

Project:
[...]

Analysis:
SUCCESS / FAIL

## 8. Quality Gate

Status:
PASS / FAIL

Actual Conditions:
[...]

## 9. Controlled Failure Test

Branch:
[...]

Build:
[...]

Tests:
[...]

Sonar Analysis:
[...]

Quality Gate:
[...]

Pipeline:
[...]

## 10. Recovery

Intentional test issue removed:
Yes / No

Final Quality Gate:
[...]

Final Pipeline:
[...]

## 11. Problems Encountered

Problem:
[...]

Root Cause:
[...]

Resolution:
[...]

## 12. Conclusion

Explain whether Jenkins successfully enforces
the SonarQube Quality Gate.

Do not invent results.
Do not expose credentials.
```

---

# Lab 20 Completion Checklist

Students should complete all applicable items:

* [ ] Lab 19 Jenkins pipeline works before modification.
* [ ] SonarQube project from Lab 10 exists.
* [ ] SonarQube Scanner Jenkins plugin installed.
* [ ] Sonar token stored in Jenkins Credentials.
* [ ] Token is **not** inside Jenkinsfile.
* [ ] SonarQube server configured in Jenkins.
* [ ] Configuration name recorded.
* [ ] SonarScanner for .NET configured.
* [ ] Jenkins scanner tool name recorded.
* [ ] `withSonarQubeEnv()` understood.
* [ ] Scanner **BEGIN → BUILD → END** understood.
* [ ] Jenkins sends a real analysis to SonarQube.
* [ ] New analysis appears in correct SonarQube project.
* [ ] SonarQube webhook to Jenkins configured.
* [ ] `waitForQualityGate()` added.
* [ ] Quality Gate status appears in Jenkins.
* [ ] Students understand build failure vs Quality Gate failure.
* [ ] Controlled failing-gate test performed if environment permits.
* [ ] Pipeline correctly reacts to failed gate.
* [ ] Intentional test issue reverted.
* [ ] Final pipeline rerun.
* [ ] No Quality Gate weakened merely to get green.
* [ ] No secret exposed.
* [ ] `JENKINS_SONARQUBE_INTEGRATION.md` created.

# The easiest way to remember Lab 20

```text
                       DEVELOPER
                           │
                           ▼
                      GIT PUSH
                           │
                           ▼
                        JENKINS
                           │
                ┌──────────┼──────────┐
                ▼          ▼          ▼
             BUILD       TEST       SONAR
                                      │
                                      ▼
                                 SONARQUBE
                                      │
                                      ▼
                                QUALITY GATE
                                      │
                              ┌───────┴───────┐
                              ▼               ▼
                            PASS             FAIL
                              │               │
                              ▼               ▼
                           CONTINUE          STOP
```

And the course progression is now:

```text
LAB 10
Sonar detects quality issues

        ↓

LAB 11
AI explains WHY

        ↓

LAB 12
Developer fixes + rescans

        ↓

LAB 19
Jenkins automates build/test

        ↓

LAB 20
Jenkins automatically asks:

"Does this code meet our
SonarQube Quality Gate?"

        ↓

LAB 21
When CI fails:

"WHY did the pipeline fail,
and how do we troubleshoot it?"
```

**For this course, this version is much easier than asking every participant to run SonarQube through Docker.** The infrastructure can be instructor-managed; students concentrate on the important DevOps concept: **Jenkins automatically sends code-quality analysis to SonarQube and uses the returned Quality Gate as a release-control signal.**

[1]: https://docs.sonarsource.com/sonarqube-server/2025.1/analyzing-source-code/ci-integration/jenkins-integration/pipeline-pause?utm_source=chatgpt.com "Setting up a pipeline pause | SonarQube Server 2025.1 LTA | Sonar Documentation"
[2]: https://plugins.jenkins.io/sonar/?utm_source=chatgpt.com "SonarQube Scanner | Jenkins plugin"
[3]: https://docs.sonarsource.com/sonarqube-server/analyzing-source-code/ci-integration/jenkins-integration/global-setup?utm_source=chatgpt.com "Setting up Jenkins | SonarQube Server | Sonar Documentation"
[4]: https://docs.sonarsource.com/sonarqube-server/2025.5/analyzing-source-code/ci-integration/jenkins-integration/global-setup "Setting up Jenkins | SonarQube Server 2025.5 | Sonar Documentation"
