Truman, **Lab 10 sounds complicated because it introduces a new tool: SonarQube.** But the workflow is actually straightforward:

```text
Your C# Source Code
        ↓
SonarScanner for .NET
        ↓
SonarQube
        ↓
Automatic Code Analysis
        ↓
Dashboard
        ↓
Quality Metrics
```

Your original course specifically requires SonarQube metrics, Quality Gates, technical debt, code smells, security hotspots, vulnerabilities, report interpretation, and remediation. 

# Lab 10 — Run a SonarQube Scan and Establish Quality Metrics

## Objectives

By the end of this lab, you should be able to:

* Install and start SonarQube Community Build locally.
* Understand what SonarQube and SonarScanner do.
* Create a SonarQube project.
* Generate an analysis token.
* Install SonarScanner for .NET.
* Scan your existing `Lab02Api`.
* Read SonarQube findings.
* Understand Quality Gate status.
* Record security, reliability, maintainability, duplication, complexity, and other baseline metrics.
* Create `SONARQUBE_BASELINE.md`.

SonarQube Community Build is Sonar's self-managed automated code-review/static-analysis product. Its metrics cover security, reliability, maintainability, complexity, duplication, coverage, issues, and Quality Gate status. ([SonarSource Documentation][1])

---

# Part 1 — What does SonarQube actually do?

Until now you have been doing this:

```text
LAB 4

CustomerReportService.cs
        ↓
Cursor AI
        ↓
"These look like code smells."
```

Lab 10 introduces:

```text
CustomerReportService.cs
PerformanceService.cs
SecurityDemoService.cs
Program.cs
        ↓
    SonarScanner
        ↓
     SonarQube
        ↓
Rules automatically applied
        ↓
Objective dashboard
```

So we can compare:

```text
AI Review
"AI thinks this may be problematic."

              vs

SonarQube Rule
"This line violates rule Sxxxx."
```

They complement one another.

---

# Part 2 — Understand SonarQube vs SonarScanner

There are **two pieces**.

### SonarQube

SonarQube is the **server + dashboard**.

You open it in your browser:

```text
http://localhost:9000
```

### SonarScanner

SonarScanner is the program that analyzes your `.NET` build and sends the analysis to SonarQube. Sonar recommends **SonarScanner for .NET** for projects built with `dotnet` or MSBuild. ([SonarSource Documentation][2])

Think:

```text
Lab02Api
    │
    ▼
SonarScanner
    │
    │ sends analysis
    ▼
SonarQube Server
    │
    ▼
Browser Dashboard
```

---

# Part 3 — Check Docker

For this beginner lab, I recommend running **SonarQube Community Build in Docker** rather than doing a manual server installation.

Open PowerShell:

```powershell
docker --version
```

If Docker is installed, you should see something similar to:

```text
Docker version xx.x.x
```

Also:

```powershell
docker info
```

If Docker Desktop isn't running, `docker info` will fail.

Docker Desktop's current Windows setup normally uses the WSL 2 backend; Docker documents WSL 2 as the normal per-user setup for most Windows users. ([Docker Documentation][3])

If Docker is not installed, install Docker Desktop first and start it before continuing.

---

# Part 4 — Start SonarQube

First pull the official Community Build image:

```powershell
docker pull sonarqube:community
```

Then run SonarQube:

```powershell
docker run -d `
  --name sonarqube `
  -p 9000:9000 `
  -v sonarqube_data:/opt/sonarqube/data `
  -v sonarqube_extensions:/opt/sonarqube/extensions `
  -v sonarqube_logs:/opt/sonarqube/logs `
  sonarqube:community
```

The official SonarQube Docker image supports the `community` tag and exposes SonarQube on port `9000`. ([Docker Hub][4])

For this **classroom lab**, the default embedded database is fine. Sonar explicitly states that its embedded H2 database is **not intended for production**, so don't use this exact setup for a production SonarQube deployment. ([Docker Hub][4])

---

# Part 5 — Check whether SonarQube is running

Run:

```powershell
docker ps
```

You should see something resembling:

```text
CONTAINER ID   IMAGE                  PORTS
abc123         sonarqube:community    0.0.0.0:9000->9000/tcp
```

You can inspect the startup log:

```powershell
docker logs sonarqube --tail 50
```

Then open your browser:

```text
http://localhost:9000
```

SonarQube Community Build uses port `9000` by default. ([SonarSource Documentation][5])

---

# Part 6 — Login to SonarQube

The initial administrative credentials are:

```text
Username: admin
Password: admin
```

Sonar's official documentation confirms the default administrator account is `admin/admin`. ([SonarSource Documentation][6])

It should ask you to change the password.

Choose a lab password you can remember.

---

# Part 7 — Your first look at SonarQube

You'll initially see something like:

```text
SONARQUBE

Projects
Issues
Rules
Quality Profiles
Quality Gates
```

Don't worry about all of these yet.

For Lab 10 we care mainly about:

```text
Projects
   ↓
Lab02Api
   ↓
Overview
Issues
Measures
```

---

# Part 8 — Create the SonarQube project

Click:

**Projects → Create Project → Local Project**

This is the official flow for a project that isn't being imported directly from a DevOps platform. ([SonarSource Documentation][7])

Enter:

```text
Project display name:
Lab02Api
```

Project key:

```text
lab02api
```

Use exactly that key for this lab because we'll need it later.

Conceptually:

```text
SonarQube Project Name
Lab02Api

SonarQube Project Key
lab02api
```

The **key** is the unique identifier the scanner uses.

---

# Part 9 — What is a SonarQube token?

Your scanner needs permission to upload analysis results.

Instead of giving the scanner:

```text
Username + Password
```

we give it:

```text
Analysis Token
```

This works like:

```text
SonarScanner
     │
     │ token
     ▼
SonarQube
     │
     ▼
"Yes, you may upload this analysis."
```

Sonar recommends tokens for analysis authentication, and project-analysis tokens can be scoped to a particular project. ([SonarSource Documentation][8])

---

# Part 10 — Generate the token

In SonarQube:

**Top-right user menu → My Account → Security**

Sonar documents this as the location for generating user/project analysis tokens. ([SonarSource Documentation][8])

Create something like:

```text
Token name:
lab10-analysis
```

If available, choose:

```text
Project Analysis Token
```

for:

```text
Lab02Api
```

Click:

**Generate**

You will get something resembling:

```text
sqp_xxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Important

Copy it immediately.

Sonar does not show the token value again after you dismiss it. ([SonarSource Documentation][8])

And **don't paste the token into Cursor or Git**.

---

# Part 11 — Put the token in a PowerShell variable

Instead of putting your token directly into every command, use:

```powershell
$env:SONAR_TOKEN="PASTE-YOUR-TOKEN-HERE"
```

Example:

```text
Do not use this example:
sqp_abc123...
```

Use your own token.

Check that the variable exists:

```powershell
$env:SONAR_TOKEN
```

Sonar supports the `SONAR_TOKEN` environment variable for analysis authentication. ([SonarSource Documentation][8])

---

# Part 12 — Go back to your .NET project

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Check:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

We want a working application before introducing SonarQube.

---

# Part 13 — Install SonarScanner for .NET

Run:

```powershell
dotnet tool install --global dotnet-sonarscanner
```

Sonar documents the .NET global tool as the simplest scanner installation for modern .NET projects. ([SonarSource Documentation][9])

If it says the tool is already installed, use:

```powershell
dotnet tool update --global dotnet-sonarscanner
```

Recent SonarScanner versions support automatic JRE provisioning with current SonarQube Community Build, so a modern setup usually doesn't require you to separately install Java just for the scanner. ([SonarSource Documentation][9])

---

# Part 14 — Understand the magic three-step Sonar process

For `.NET`, remember only:

```text
BEGIN
  ↓
BUILD
  ↓
END
```

### BEGIN

Tell SonarScanner:

> "I'm starting analysis."

### BUILD

Compile the C# application while Sonar analyzes it.

### END

Finish analysis and upload results to SonarQube.

Sonar's .NET scanner is specifically designed to integrate analysis into the build process. ([SonarSource Documentation][2])

---

# Part 15 — BEGIN the scan

From:

```text
C:\AI_SDLC_Labs\Lab02Api
```

run:

```powershell
dotnet sonarscanner begin `
  /k:"lab02api" `
  /d:sonar.host.url="http://localhost:9000" `
  /d:sonar.token="$env:SONAR_TOKEN"
```

Let's decode this.

```text
/k:"lab02api"

means:

Which SonarQube project?
        ↓
lab02api
```

```text
sonar.host.url

means:

Where is SonarQube?
        ↓
http://localhost:9000
```

```text
sonar.token

means:

Am I authorized?
        ↓
Use my token
```

These are the current parameters shown in Sonar's Community Build documentation. ([SonarSource Documentation][10])

You should eventually see:

```text
Pre-processing succeeded.
```

---

# Part 16 — BUILD the application

Now run:

```powershell
dotnet build --no-incremental
```

Sonar recommends a non-incremental build between the `begin` and `end` steps so the analysis observes a complete build. ([SonarSource Documentation][10])

You should see:

```text
Build succeeded.
```

You may also see Sonar analysis warnings.

**Don't immediately fix them.**

Lab 10 is about establishing the baseline.

---

# Part 17 — END the scan

Run:

```powershell
dotnet sonarscanner end `
  /d:sonar.token="$env:SONAR_TOKEN"
```

The scanner collects the analysis generated during the build and sends it to SonarQube in the `end` phase. ([SonarSource Documentation][10])

Look for something resembling:

```text
Analysis succeeded.
```

At this point:

```text
Lab02Api
    ↓
SCANNED
    ↓
Results uploaded
    ↓
SonarQube
```

---

# Part 18 — Open the SonarQube dashboard

Return to:

```text
http://localhost:9000
```

Click:

**Projects → Lab02Api**

You should now have your first real SonarQube dashboard.

SonarQube's project overview shows the Quality Gate and project measures; its Measures page lets you drill into the detailed metrics. ([SonarSource Documentation][11])

---

# Part 19 — What metrics should you look at?

Depending on your SonarQube mode, the terminology may differ.

In **Standard Experience**, you'll see familiar categories such as:

```text
Bugs
Vulnerabilities
Code Smells
```

In **MQR Mode**, Sonar groups issues around:

```text
Reliability
Security
Maintainability
```

Current SonarQube Community Build supports both views, so don't worry if your screen doesn't use exactly the same labels as mine. ([SonarSource Documentation][12])

For our course, record:

```text
Quality Gate

Reliability / Bugs

Security / Vulnerabilities

Maintainability / Code Smells

Security Hotspots

Duplicated Lines

Complexity

Lines of Code

Coverage (if available)
```

---

# Part 20 — Don't expect my numbers to match yours

For example, your SonarQube might report:

```text
Issues              7
Maintainability     A
Reliability         A
Security            A
Duplicated Lines    2.5%
Coverage            0%
```

Or completely different numbers.

**Do not copy these numbers.**

Record the actual values on your dashboard.

Your results depend on your exact code, SonarQube version, active quality profile, and rules.

---

# Part 21 — Understand Quality Gate

Look near the top of the dashboard.

You might see:

```text
QUALITY GATE

PASSED
```

or:

```text
FAILED
```

A Quality Gate is basically:

> **Does this project meet our minimum quality policy?**

Sonar defines a Quality Gate as a set of conditions applied to analysis metrics. If one of the failing conditions is triggered, the Quality Gate fails. ([SonarSource Documentation][13])

Conceptually:

```text
             SONARQUBE RESULTS
                    │
                    ▼
               QUALITY GATE
                    │
              ┌─────┴─────┐
              ▼           ▼
            PASS         FAIL
              │           │
          acceptable   investigate
```

---

# Part 22 — What is "Sonar way"?

Your installation normally has the built-in:

```text
Sonar way
```

Quality Gate.

Sonar currently describes its recommended built-in gate as focusing on **new code**, with conditions around new issues, reviewed security hotspots, test coverage, and duplication. ([SonarSource Documentation][13])

Don't customize it yet.

Lab 10 should establish the baseline using the default configuration.

---

# Part 23 — Click Issues

Click:

**Lab02Api → Issues**

You may see something like:

```text
Program.cs
    → Issue

CustomerReportService.cs
    → Issue

SecurityDemoService.cs
    → Issue
```

Click an issue.

Sonar normally shows:

```text
File
Line
Rule
Description
Severity
Why this is an issue
```

This is much more useful than merely seeing:

```text
"3 code smells"
```

because you can find the exact affected code.

---

# Part 24 — Compare SonarQube with Lab 4

Remember Lab 4?

You manually identified:

```text
Deep nesting
Duplicate code
Magic numbers
Magic strings
Readability problems
```

Now ask Cursor:

```text
Review my CODE_QUALITY_BASELINE.md from Lab 4.

I have now run SonarQube.

I will provide the actual SonarQube findings.

Compare:

1. Issues AI identified in Lab 4
2. Issues SonarQube detected
3. Findings found by both
4. Findings found only by AI
5. Findings found only by SonarQube

Do not invent SonarQube findings.
```

This creates a very useful exercise:

```text
AI Review
    ↓
        ↘
          Compare
        ↗
SonarQube
```

---

# Part 25 — Look at Measures

Click:

**Lab02Api → Measures**

SonarQube defines metrics for security, reliability, maintainability, test coverage, complexity, duplication, size, issues, and Quality Gate status. ([SonarSource Documentation][1])

For Lab 10, record at least:

| Metric             | Your Result |
| ------------------ | ----------- |
| Quality Gate       |             |
| Reliability        |             |
| Security           |             |
| Maintainability    |             |
| Total Issues       |             |
| Security Hotspots  |             |
| Duplicated Lines % |             |
| Lines of Code      |             |
| Complexity         |             |
| Coverage           |             |

Fill these with **your real SonarQube numbers**.

---

# Part 26 — Why might Coverage be missing or 0?

Don't panic if you haven't created unit tests yet.

SonarQube **does not itself generate .NET coverage reports**. A coverage tool such as Coverlet or `dotnet-coverage` needs to generate a report, and then SonarScanner imports it. ([SonarSource Documentation][14])

Therefore, for Lab 10 you can write:

```text
Coverage:
Not yet established / no coverage report imported
```

We will properly handle coverage in the later unit-testing labs.

---

# Part 27 — Ask AI to interpret SonarQube

Now you can use AI intelligently.

Don't ask:

```text
Fix SonarQube.
```

Instead, copy **one actual SonarQube issue** into Cursor and use:

```text
You are reviewing an actual SonarQube finding.

SonarQube reported:

[paste actual issue]

Affected code:

[paste code]

Do NOT modify the code.

Explain:

1. What SonarQube detected
2. Why the rule exists
3. Whether it is relevant in this project
4. Potential impact
5. Whether it appears to be:
   - True Positive
   - False Positive
   - Needs More Context
6. Recommended next action
```

Now:

```text
SonarQube
    ↓
automatically detects
    ↓
AI
    ↓
explains
    ↓
Human
    ↓
decides
```

This is exactly the kind of **AI + traditional static analysis** workflow your course should teach.

---

# Part 28 — Don't automatically fix every SonarQube issue

SonarQube findings still require human review.

Sonar allows authorized users to mark findings as **Accepted** when deliberately deferred or **False positive** when the analysis doesn't apply correctly. ([SonarSource Documentation][15])

So:

```text
SonarQube says issue
        ↓
DO NOT automatically assume
        ↓
"Must change code"
```

Instead:

```text
Sonar Finding
      ↓
Understand Rule
      ↓
Examine Context
      ↓
True Positive?
   /            \
 YES             NO
 ↓               ↓
Fix          False Positive
```

---

# Part 29 — Create your baseline file

In Cursor create:

```text
SONARQUBE_BASELINE.md
```

Put:

```markdown
# Lab 10 — SonarQube Quality Baseline

## Project

Lab02Api

## Analysis Environment

SonarQube Community Build
SonarScanner for .NET

## Build Result

Build: PASS

## Quality Gate

Status: [ENTER ACTUAL RESULT]

## Quality Metrics

| Metric | Result |
|---|---|
| Reliability | [actual] |
| Security | [actual] |
| Maintainability | [actual] |
| Total Issues | [actual] |
| Security Hotspots | [actual] |
| Duplicated Lines | [actual] |
| Lines of Code | [actual] |
| Complexity | [actual] |
| Coverage | [actual / not available] |

## Highest Priority Findings

1. [Actual finding]
2. [Actual finding]
3. [Actual finding]

## Initial Assessment

[Your observations]

## Baseline Purpose

This baseline records the project's quality condition
before SonarQube remediation work is performed.
```

**Do not refactor all the issues yet.**

Lab 10's purpose is:

```text
SCAN
   ↓
MEASURE
   ↓
BASELINE
```

---

# Part 30 — Your completed Lab 10 architecture

You now have:

```text
                  DEVELOPMENT MACHINE

                    Lab02Api
                        │
                        │
                SonarScanner for .NET
                        │
                 BEGIN → BUILD → END
                        │
                        ▼
              ┌───────────────────┐
              │     SonarQube     │
              │ Community Build   │
              └─────────┬─────────┘
                        │
             Static Code Analysis
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
   Reliability       Security      Maintainability
        │               │               │
        └───────────────┼───────────────┘
                        │
              Complexity/Duplication
                        │
                        ▼
                  QUALITY GATE
                        │
                  ┌─────┴─────┐
                  ▼           ▼
                PASS         FAIL
                        │
                        ▼
              SONARQUBE_BASELINE.md
```

## The commands to remember

There are really only **five important commands**.

Start SonarQube:

```powershell
docker run -d --name sonarqube -p 9000:9000 sonarqube:community
```

Install scanner:

```powershell
dotnet tool install --global dotnet-sonarscanner
```

Start analysis:

```powershell
dotnet sonarscanner begin `
  /k:"lab02api" `
  /d:sonar.host.url="http://localhost:9000" `
  /d:sonar.token="$env:SONAR_TOKEN"
```

Build:

```powershell
dotnet build --no-incremental
```

Finish analysis:

```powershell
dotnet sonarscanner end `
  /d:sonar.token="$env:SONAR_TOKEN"
```

## Lab 10 completion checklist

* [ ] Docker works.
* [ ] SonarQube Community Build container runs.
* [ ] `http://localhost:9000` opens.
* [ ] SonarQube login completed.
* [ ] `Lab02Api` project created.
* [ ] Analysis token created.
* [ ] `SONAR_TOKEN` environment variable configured.
* [ ] SonarScanner for .NET installed.
* [ ] `begin` command succeeds.
* [ ] `dotnet build --no-incremental` succeeds.
* [ ] `end` command succeeds.
* [ ] SonarQube dashboard contains analysis results.
* [ ] Quality Gate recorded.
* [ ] Reliability recorded.
* [ ] Security recorded.
* [ ] Maintainability recorded.
* [ ] Issues inspected.
* [ ] Duplication recorded.
* [ ] Complexity recorded.
* [ ] Coverage recorded or marked not yet available.
* [ ] `SONARQUBE_BASELINE.md` created.
* [ ] **No mass remediation performed yet.**

The easiest way to remember the progression is:

```text
LAB 7
AI looks for security problems

LAB 8
NuGet looks for package vulnerabilities

LAB 9
Challenge AI's security fixes

LAB 10
SonarQube independently scans the source code
        ↓
Establish objective quality metrics
```

**Lab 10 should end with a dashboard and baseline—not with all the code fixed.** The next SonarQube labs can then use this baseline to investigate findings, perform root-cause analysis, remediate them, rescan, and prove quantitatively that quality improved.

[1]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/code-metrics/metrics-definition?utm_source=chatgpt.com "Understanding measures and metrics | SonarQube Community Build | Sonar Documentation"
[2]: https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/scanners/dotnet/introduction?utm_source=chatgpt.com "Introduction | SonarQube Community Build | Sonar Documentation"
[3]: https://docs.docker.com/desktop/setup/install/windows-install/?utm_source=chatgpt.com "Install Docker Desktop on Windows | Docker Docs"
[4]: https://hub.docker.com/_/sonarqube/?utm_source=chatgpt.com "sonarqube - Official Image | Docker Hub"
[5]: https://docs.sonarsource.com/sonarqube-community-build/server-installation/from-docker-image/installation-overview?utm_source=chatgpt.com "Installation overview | SonarQube Community Build | Sonar Documentation"
[6]: https://docs.sonarsource.com/sonarqube-community-build/instance-administration/user-management/introduction?utm_source=chatgpt.com "Introduction | SonarQube Community Build | Sonar Documentation"
[7]: https://docs.sonarsource.com/sonarqube-community-build/project-administration/creating-project/creating-manually?utm_source=chatgpt.com "Creating your project manually | SonarQube Community Build | Sonar Documentation"
[8]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/managing-tokens?utm_source=chatgpt.com "Managing your tokens | SonarQube Community Build | Sonar Documentation"
[9]: https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/dotnet-environments/getting-started-with-net?utm_source=chatgpt.com "Getting started with .NET | SonarQube Community Build | Sonar Documentation"
[10]: https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/scanners/dotnet/using?utm_source=chatgpt.com "Using the scanner | SonarQube Community Build | Sonar Documentation"
[11]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/viewing-projects/project-overview?utm_source=chatgpt.com "Viewing analysis summary | SonarQube Community Build | Sonar Documentation"
[12]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/code-metrics/changing-modes?utm_source=chatgpt.com "Changing instance modes | SonarQube Community Build | Sonar Documentation"
[13]: https://docs.sonarsource.com/sonarqube-community-build/quality-standards-administration/managing-quality-gates/introduction-to-quality-gates?utm_source=chatgpt.com "Understanding quality gates | SonarQube Community Build | Sonar Documentation"
[14]: https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/test-coverage/dotnet-test-coverage?utm_source=chatgpt.com ".NET test coverage | SonarQube Community Build | Sonar Documentation"
[15]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/managing?utm_source=chatgpt.com "Editing issues | SonarQube Community Build | Sonar Documentation"
