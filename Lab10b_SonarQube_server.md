ne thing to verify first: **Connected to Server** and **Bound to the correct SonarQube project** are two related but different things. 
Connected Mode works best when your local workspace is bound to the matching SonarQube project so the IDE can align rules/settings with the server. ([SonarSource Docs][1])

# Lab 10 — Kick Start from Your Current Position

## Part 1 — Open the .NET Project in VS Code

Open:

```text
C:\AI_SDLC_Labs\Lab02Api
```

In VS Code:

```text
File
  ↓
Open Folder
  ↓
C:\AI_SDLC_Labs\Lab02Api
```

You should see files such as:

```text
Lab02Api
│
├── Program.cs
├── OrderService.cs
├── CustomerReportService.cs
├── PerformanceService.cs
├── SecurityDemoService.cs
└── Lab02Api.csproj
```

---

# Part 2 — Confirm Connected Mode Is Active

On the left side of VS Code, click the:

```text
SonarQube
```

icon.

Find:

```text
SONARQUBE SETUP
      ↓
CONNECTED MODE
```

You should see your SonarQube Server connection.

Then verify the current workspace is **bound to your Lab02Api SonarQube project**.

The intended relationship is:

```text
VS Code Workspace
Lab02Api
       │
       │ Connected Mode
       ▼
SonarQube Server
Lab02Api Project
```

Connected Mode lets SonarQube for IDE synchronize relevant project rules/settings and server-side issue information with the bound workspace. ([SonarSource Docs][1])

---

# Part 3 — Run the First Local IDE Analysis

This part requires **no SonarScanner command yet**.

Open:

```text
SecurityDemoService.cs
```

or:

```text
CustomerReportService.cs
```

Then:

```text
Ctrl + S
```

SonarQube for IDE normally analyzes supported files automatically when they are opened or saved. ([SonarSource Docs][2])

So:

```text
Open C# file
     ↓
Save file
     ↓
SonarQube for IDE analyzes
     ↓
Issues appear
```

---

# Part 4 — Open the SonarQube Findings

Look at the **SONARQUBE** panel in VS Code.

You can also inspect:

```text
View
  ↓
Problems
```

or press:

```text
Ctrl + Shift + M
```

SonarQube for IDE provides its own SonarQube panel where findings can be opened and investigated, including the associated rule, severity and affected source location. ([SonarSource Docs][3])

Pick **one actual issue**.

Do not fix anything yet.

Record:

```text
File:
________________

Line:
________________

Rule:
________________

Severity:
________________

Message:
________________
```

---

# Part 5 — Open the Rule Explanation

Click the Sonar issue.

Look for the SonarQube rule description.

Read:

```text
What rule was violated?

Why is this considered a problem?

Which code triggered the rule?

Does it affect:

Security?
Reliability?
Maintainability?
```

This is your first Lab 10 exercise.

At this point:

```text
Sonar found something
       ↓
Student understands WHAT it found
```

Do **not** ask AI to fix it yet.

That comes later.

---

# Part 6 — Create Your First Lab 10 Evidence File

Create:

```text
SONARQUBE_BASELINE.md
```

Start with:

```markdown
# Lab 10 — SonarQube Baseline

## Environment

Project:
Lab02Api

IDE:
Visual Studio Code

SonarQube for IDE:
Installed

Connected Mode:
Yes

SonarQube Server:
Connected

Project Binding:
Confirmed / Not Confirmed

## Local Finding 1

File:
[actual]

Line:
[actual]

Rule:
[actual]

Severity:
[actual]

Message:
[actual]

Software Quality:
[actual if shown]

## Local Finding 2

[actual result]

## Local Finding 3

[actual result]
```

Use only actual findings.

---

# Part 7 — Now Perform the Full SonarQube Server Scan

This is the important second half of Lab 10.

Your VS Code extension performs **local IDE analysis**.

Now we want:

```text
Entire .NET Project
        ↓
SonarScanner for .NET
        ↓
SonarQube Server
        ↓
Project Dashboard
        ↓
Quality Gate
        ↓
Metrics
```

---

# Part 8 — Find Your SonarQube Project Key

Open SonarQube Server in the browser.

Open your project.

Then look for:

```text
Project Information
```

The Project Information view contains details such as the **project key**, Quality Gate and Quality Profiles. ([SonarSource Docs][4])

Record:

```text
Project Name:
________________

Project Key:
________________
```

For example only:

```text
Project Name:
Lab02Api

Project Key:
lab02api
```

Use **your actual key**.

---

# Part 9 — Open VS Code Terminal

In VS Code:

```text
Terminal
   ↓
New Terminal
```

Go to:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Check:

```powershell
dir
```

Make sure you can see:

```text
Lab02Api.csproj
```

---

# Part 10 — Verify the Project Builds BEFORE Sonar

Run:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

If it fails:

**stop here.**

Fix the normal build first.

Do not start SonarScanner until:

```text
dotnet build
      ↓
SUCCESS
```

---

# Part 11 — Check Whether SonarScanner for .NET Is Installed

Run:

```powershell
dotnet sonarscanner --version
```

If you get a version:

```text
SonarScanner for .NET ...
```

continue.

If Windows says the command doesn't exist, install it:

```powershell
dotnet tool install --global dotnet-sonarscanner
```

If it is already installed but you want the current installed tool updated:

```powershell
dotnet tool update --global dotnet-sonarscanner
```

Sonar officially supports installing SonarScanner for .NET as a .NET global tool and invoking it through `dotnet sonarscanner`. ([SonarSource Docs][5])

---

# Part 12 — Store the Token Safely in PowerShell

Do **not** put the token directly into:

```text
Program.cs
```

Do not put it into:

```text
Git
```

For the current PowerShell session:

```powershell
$env:SONAR_TOKEN="PASTE_YOUR_TOKEN_HERE"
```

Then store your server address:

```powershell
$env:SONAR_HOST_URL="YOUR_SONARQUBE_SERVER_URL"
```

For example, if the server is an instructor machine:

```text
http://192.168.x.x:9000
```

Use your actual SonarQube URL.

The current scanner requires a project key and, for SonarQube Server analysis, the server URL must be directed to your SonarQube instance; token authentication is passed with `sonar.token`. ([SonarSource Docs][6])

---

# Part 13 — Understand the Three Scanner Commands

For .NET, remember only:

```text
BEGIN
   ↓
BUILD
   ↓
END
```

### BEGIN

Tells Sonar:

> I am starting an analysis.

### BUILD

Sonar observes/analyzes the .NET build.

### END

Sonar:

> Collect the results and upload them to SonarQube Server.

Sonar documents this exact **begin → build → end** workflow for SonarScanner for .NET. ([SonarSource Docs][7])

---

# Part 14 — Run BEGIN

Replace:

```text
YOUR_PROJECT_KEY
```

with your actual project key.

Run:

```powershell
dotnet sonarscanner begin `
  /k:"YOUR_PROJECT_KEY" `
  "/d:sonar.host.url=$env:SONAR_HOST_URL" `
  "/d:sonar.token=$env:SONAR_TOKEN"
```

Example only:

```powershell
dotnet sonarscanner begin `
  /k:"lab02api" `
  "/d:sonar.host.url=$env:SONAR_HOST_URL" `
  "/d:sonar.token=$env:SONAR_TOKEN"
```

If successful, you should see scanner preparation output.

The `begin` step configures the build for analysis and retrieves analysis configuration/quality-profile information. ([SonarSource Docs][8])

---

# Part 15 — Run the BUILD

Now:

```powershell
dotnet build --no-incremental
```

Do **not** skip this.

The scanner analyzes the project through the build process. ([SonarSource Docs][7])

Conceptually:

```text
Sonar BEGIN
      ↓
dotnet build
      ↓
Sonar observes project
```

---

# Part 16 — Run END

Now:

```powershell
dotnet sonarscanner end `
  "/d:sonar.token=$env:SONAR_TOKEN"
```

The `end` step gathers the analysis information and uploads the results to SonarQube Server. ([SonarSource Docs][7])

If successful, the terminal should indicate the analysis completed/submitted successfully.

---

# Part 17 — The Complete Commands Together

Your whole first scan is:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api

dotnet build

dotnet sonarscanner --version

$env:SONAR_TOKEN="YOUR_TOKEN"
$env:SONAR_HOST_URL="YOUR_SONARQUBE_SERVER_URL"

dotnet sonarscanner begin `
  /k:"YOUR_PROJECT_KEY" `
  "/d:sonar.host.url=$env:SONAR_HOST_URL" `
  "/d:sonar.token=$env:SONAR_TOKEN"

dotnet build --no-incremental

dotnet sonarscanner end `
  "/d:sonar.token=$env:SONAR_TOKEN"
```

The sequence to remember is:

```text
NORMAL BUILD
     ↓
Scanner BEGIN
     ↓
SONAR BUILD
     ↓
Scanner END
     ↓
SONARQUBE SERVER
```

---

# Part 18 — Open SonarQube Server

Go back to the browser.

Open:

```text
SonarQube
    ↓
Projects
    ↓
Lab02Api
```

Look at:

```text
Overview
```

The project Overview shows the current analysis summary, including the Quality Gate and project measures, and the Activity area records analyses over time. ([SonarSource Docs][9])

---

# Part 19 — First Check: Did the New Analysis Arrive?

Look for:

```text
Latest Analysis
```

or the current project activity/time.

You want to prove:

```text
VS Code
   ↓
SonarScanner
   ↓
SonarQube Server

SUCCESS
```

If there is no new analysis, don't start interpreting metrics yet.

First fix the scanner/upload problem.

---

# Part 20 — Record the Quality Gate

Find:

```text
Quality Gate
```

Record:

```text
Quality Gate:

PASS / FAIL
```

A Quality Gate evaluates the analyzed code against configured quality conditions and returns pass/fail. ([SonarSource Docs][10])

**A FAIL is not a failed Lab 10.**

A failed Quality Gate can actually give you good material for Labs 11 and 12.

---

# Part 21 — Record the Main Metrics

Record the actual dashboard values available to you:

```text
Quality Gate:
________________

Security:
________________

Reliability:
________________

Maintainability:
________________

Security Hotspots:
________________

Duplications:
________________

Coverage:
________________

Issues:
________________
```

Do not invent a value that isn't shown.

---

# Important: Coverage May Be Missing or 0%

Don't panic if:

```text
Coverage:
0%
```

or there is no useful coverage number yet.

SonarQube Server **does not generate .NET test coverage itself**. A separate coverage tool must generate the coverage report and the scanner must import it. ([SonarSource Docs][11])

For this first Lab 10 scan, you can record:

```text
Coverage:
Not configured yet
```

if applicable.

Lab 14 can deal more deeply with test coverage.

---

# Part 22 — Open the Issues Page

Go to:

```text
Project
   ↓
Issues
```

Select one real issue.

Record:

```text
Finding 01

File:
________________

Line:
________________

Rule:
________________

Severity:
________________

Message:
________________
```

Do the same for up to three actual findings.

---

# Part 23 — Compare Server vs VS Code

This is a very useful exercise because you're already using Connected Mode.

From SonarQube Server:

```text
Project
   ↓
Issues
   ↓
Select an issue
```

If available, choose:

```text
Open in IDE
```

SonarQube supports opening server-side issues directly in a connected VS Code workspace, which helps students see the exact code associated with a server finding. ([SonarSource Docs][3])

Conceptually:

```text
SONARQUBE SERVER

Issue #1
     │
     │ Open in IDE
     ▼
VS CODE

SecurityDemoService.cs
       ↓
Problem line highlighted
```

That is an excellent classroom demonstration.

---

# Part 24 — Ask AI to Explain, But NOT Fix

Now use GitHub Copilot or Cursor:

```text
I am learning SonarQube.

Do NOT modify my code.

SonarQube reported this actual finding:

Project:
Lab02Api

File:
[actual]

Line:
[actual]

Rule:
[actual]

Severity:
[actual]

Message:
[actual]

Affected Code:
[paste code]

Explain in beginner-friendly language:

1. What SonarQube detected.
2. What the Sonar rule means.
3. Why this code triggered the rule.
4. Whether it mainly affects:
   - Security
   - Reliability
   - Maintainability
5. What might happen if it is ignored.

Do NOT suggest remediation yet.
Do NOT modify code.
```

This prepares directly for Lab 11.

---

# Part 25 — Complete `SONARQUBE_BASELINE.md`

Update your file:

```markdown
# Lab 10 — SonarQube Baseline

## 1. Project

Lab02Api

## 2. SonarQube Environment

VS Code SonarQube for IDE:
Installed

Connected Mode:
Yes

Workspace Bound:
Yes / No

SonarQube Server:
[actual server]

SonarScanner for .NET:
[actual version]

## 3. Build Baseline

Command:

dotnet build

Result:

PASS / FAIL

## 4. Full Sonar Analysis

Project Key:
[actual]

Scanner Begin:
PASS / FAIL

Build During Scan:
PASS / FAIL

Scanner End:
PASS / FAIL

Analysis Uploaded:
Yes / No

## 5. Quality Gate

Status:
PASS / FAIL

## 6. Metrics

Security:
[...]

Reliability:
[...]

Maintainability:
[...]

Security Hotspots:
[...]

Duplication:
[...]

Coverage:
[...]

Issues:
[...]

## 7. Finding 01

File:
[...]

Rule:
[...]

Severity:
[...]

Message:
[...]

## 8. Finding 02

[...]

## 9. Finding 03

[...]

## 10. Notes

Coverage configured:
Yes / No

Observations:
[...]

## 11. Conclusion

SonarQube successfully analyzed the project:
Yes / No
```

# Your Immediate Next Actions

Because you have already completed **connection + token**, do these now in order:

```text
1. Open Lab02Api in VS Code

2. Confirm workspace is BOUND
   to the SonarQube project

3. Open SecurityDemoService.cs

4. Ctrl + S

5. Look at SONARQUBE findings

6. Record 1–3 actual local issues

7. Open SonarQube Server

8. Copy your PROJECT KEY

9. Open VS Code Terminal

10. Run:

    dotnet build

11. Run:

    dotnet sonarscanner --version

12. Set:

    SONAR_TOKEN
    SONAR_HOST_URL

13. Run:

    BEGIN
      ↓
    BUILD
      ↓
    END

14. Open SonarQube dashboard

15. Record:

    Quality Gate
    Security
    Reliability
    Maintainability
    Hotspots
    Duplication
    Coverage
    Issues

16. Create:

    SONARQUBE_BASELINE.md
```

Once you have reached:

```text
Scanner END
     ↓
Analysis appears in SonarQube Server
     ↓
Quality Gate displayed
```

**Lab 10 has properly started.** From there, Lab 11 takes one of the actual findings and investigates **WHY** it exists; Lab 12 then remediates it and proves the improvement with a rescan.

[1]: https://docs.sonarsource.com/sonarqube-for-vs-code/connect-your-ide/connected-mode?utm_source=chatgpt.com "Connected mode | VS Code | Sonar Documentation"
[2]: https://docs.sonarsource.com/sonarqube-for-vs-code/getting-started/running-an-analysis?utm_source=chatgpt.com "Running an analysis | VS Code | Sonar Documentation"
[3]: https://docs.sonarsource.com/sonarqube-for-vs-code/using/investigating-issues?utm_source=chatgpt.com "Investigating issues | VS Code | Sonar Documentation"
[4]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/viewing-projects/viewing-project-information?utm_source=chatgpt.com "Viewing project information | SonarQube Community Build | Sonar Documentation"
[5]: https://docs.sonarsource.com/sonarqube/10.0/analyzing-source-code/scanners/sonarscanner-for-dotnet?utm_source=chatgpt.com "SonarScanner for .NET | SonarQube Server 10.0 | Sonar Documentation"
[6]: https://docs.sonarsource.com/sonarqube-server/analyzing-source-code/scanners/dotnet/using?utm_source=chatgpt.com "Using the scanner | SonarQube Server | Sonar Documentation"
[7]: https://docs.sonarsource.com/sonarqube-server/10.0/analyzing-source-code/scanners/sonarscanner-for-dotnet?utm_source=chatgpt.com "SonarScanner for .NET | SonarQube Server 10.0 | Sonar Documentation"
[8]: https://docs.sonarsource.com/sonarqube-server/10.4/analyzing-source-code/scanners/sonarscanner-for-dotnet?utm_source=chatgpt.com "SonarScanner for .NET | SonarQube Server 10.4 | Sonar Documentation"
[9]: https://docs.sonarsource.com/sonarqube-server/10.8/user-guide/viewing-projects/project-overview?utm_source=chatgpt.com "Viewing analysis summary | SonarQube Server 10.8 | Sonar Documentation"
[10]: https://docs.sonarsource.com/sonarqube-server/2025.2/quality-standards-administration/managing-quality-gates/introduction-to-quality-gates?utm_source=chatgpt.com "Introduction to quality gates | SonarQube Server 2025.2 | Sonar Documentation"
[11]: https://docs.sonarsource.com/sonarqube-server/2026.1/analyzing-source-code/dotnet-environments/dotnet-test-coverage?utm_source=chatgpt.com ".NET test coverage | SonarQube Server 2026.1 LTA | Sonar Documentation"
