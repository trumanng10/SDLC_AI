> **SonarScanner is not a VS Code plugin.**
> The VS Code extension is called **SonarQube for IDE**. SonarScanner for .NET is a command-line build/scanning tool. ([SonarSource Documentation][1])

The original Lab 10 genuinely expects a SonarQube scan, metrics, findings and a Quality Gate, so the IDE extension alone does **not fully replace** the SonarQube platform. 

## Which is easier?

For your course, I rank the options like this:

| Approach                               | Beginner Difficulty | Full Metrics? |    Quality Gate? | My Recommendation                   |
| -------------------------------------- | ------------------: | ------------: | ---------------: | ----------------------------------- |
| **SonarQube for IDE in VS Code**       |         ⭐ Very easy |     ❌ Limited | ❌ Not standalone | ✅ Start here                        |
| **SonarQube Cloud + VS Code**          |             ⭐⭐ Easy |             ✅ |                ✅ | ⭐ Best if internet/GitHub available |
| **Central SonarQube server + VS Code** |                  ⭐⭐ |             ✅ |                ✅ | ⭐ Best classroom setup              |
| **Each student runs SonarQube Docker** |                ⭐⭐⭐⭐ |             ✅ |                ✅ | ❌ Too much for beginners            |
| Native SonarQube server install        |               ⭐⭐⭐⭐⭐ |             ✅ |                ✅ | ❌ Avoid                             |

### My recommended design for your class

I would change Lab 10 to:

```text
PART A — Beginner
Install SonarQube for IDE in VS Code
        ↓
See code issues immediately
        ↓
Understand Rule / Severity / Why


PART B — Full SonarQube
Connect to SonarQube Cloud
OR
Instructor-hosted SonarQube
        ↓
Run full project analysis
        ↓
Dashboard
        ↓
Metrics
        ↓
Quality Gate
```

That is much easier pedagogically.

---

# Revised Lab 10

# Run a SonarQube Scan and Establish Quality Metrics

## Part 1 — First understand the three Sonar tools

Before installing anything, participants need this explanation.

### 1. SonarQube for IDE

This is a **VS Code extension**.

Think of it like a spell checker:

```text
Write C# code
     ↓
SonarQube for IDE
     ↓
"This line may have a problem."
     ↓
VS Code Problems panel
```

Sonar describes it as a free IDE extension that analyzes code while you work and highlights maintainability, reliability and security issues. ([SonarSource Documentation][2])

### 2. SonarScanner for .NET

This is **not a VS Code extension**.

It is a command-line tool:

```text
.NET project
     ↓
SonarScanner for .NET
     ↓
Build project
     ↓
Send analysis
     ↓
SonarQube
```

Sonar recommends SonarScanner for .NET for projects built using `dotnet` or MSBuild. ([SonarSource Documentation][3])

### 3. SonarQube Server / Community Build / Cloud

This gives you the proper dashboard:

```text
               SonarQube
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
    Security   Reliability  Maintainability
       │           │           │
       └───────────┼───────────┘
                   ▼
               Metrics
                   ↓
             Quality Gate
                   ↓
               PASS/FAIL
```

Quality Gates belong to the SonarQube platform and evaluate analysis results against defined conditions. ([SonarSource Documentation][4])

---

# Part 2 — Start with VS Code only

This should be the **first exercise**.

No Docker.

No server.

No token.

No scanner.

Just VS Code.

## Step 1 — Open VS Code

Open:

```text
Visual Studio Code
```

Then click:

```text
Extensions
```

or press:

```text
Ctrl + Shift + X
```

---

# Step 2 — Search for SonarQube

Search:

```text
SonarQube for IDE
```

Use the extension published by:

```text
SonarSource
```

Then click:

```text
Install
```

Official Sonar instructions use exactly this process: Extensions → search **SonarQube for IDE** → Install. ([SonarSource Documentation][1])

---

# Step 3 — Reload VS Code if requested

If you see:

```text
Reload Required
```

click it.

---

# Step 4 — Open our existing project

In VS Code:

**File → Open Folder**

Open:

```text
C:\AI_SDLC_Labs\Lab02Api
```

You should see:

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

# Step 5 — Open a C# file

For example:

```text
CustomerReportService.cs
```

or:

```text
SecurityDemoService.cs
```

SonarQube for IDE automatically performs analysis when supported files are opened or saved. ([SonarSource Documentation][5])

Students don't need to run a special command.

Think:

```text
Open .cs file
      ↓
Sonar analyzes automatically
      ↓
Problem detected
      ↓
Squiggly line appears
```

---

# Step 6 — Open the Problems panel

Press:

```text
Ctrl + Shift + M
```

or:

**View → Problems**

You may see something like:

```text
PROBLEMS

SecurityDemoService.cs

SonarQube:
Remove this hard-coded credential.

CustomerReportService.cs

SonarQube:
Refactor this method to reduce complexity.
```

The exact results depend on your actual code and active rules.

---

# Step 7 — Click a Sonar issue

Click one actual finding.

Students should examine:

```text
Rule
Severity
Message
Affected line
Explanation
```

Ask:

> What is Sonar complaining about?

Then:

> Why?

Not:

> How do I remove the warning?

---

# Step 8 — Show "Why is this an issue?"

Click the issue/rule information.

Sonar provides detailed rule descriptions explaining why code triggers a finding. ([SonarSource Documentation][5])

Now students can learn:

```text
SOURCE CODE

    ↓

SONAR RULE

    ↓

WHY THIS IS AN ISSUE

    ↓

Developer understands risk
```

---

# Part 3 — Let AI explain the Sonar finding

Now use GitHub Copilot/Cursor.

Example prompt:

```text
I am a beginner learning SonarQube.

SonarQube for IDE reported this issue:

[paste issue]

Affected code:

[paste code]

Do NOT modify anything.

Explain:

1. What SonarQube detected.
2. What the rule means.
3. Why the code triggered it.
4. Whether it affects:
   - Maintainability
   - Reliability
   - Security
5. Explain using beginner-friendly language.
```

This makes the workflow very clear:

```text
Sonar
   ↓
Finds issue

AI
   ↓
Explains issue

Human
   ↓
Understands / verifies
```

---

# Part 4 — Important limitation of VS Code extension

This needs to be explicitly written in Lab 10.

### SonarQube for IDE alone can show local code issues.

But it does **not replace the full SonarQube project dashboard**.

With only the extension, Lab 10 can teach:

```text
✓ Code issue
✓ Rule
✓ Severity
✓ Why it matters
✓ Suggested remediation
```

But the full Lab 10 also wants:

```text
Quality Gate

Project-wide metrics

Duplication

Overall Security

Overall Reliability

Overall Maintainability

Project analysis history
```

For the full Sonar solution, the extension uses **Connected Mode** with SonarQube Server, SonarQube Cloud, or Community Build. ([SonarSource Documentation][6])

---

# Therefore I recommend NOT installing Docker on every student's PC

Instead use this classroom architecture:

```text
            INSTRUCTOR / CENTRAL SERVER

                  SonarQube
                     │
             Project: Lab02Api
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼

       Student 1  Student 2  Student 3

       VS Code    VS Code    VS Code
          │          │          │
    SonarQube   SonarQube   SonarQube
      for IDE     for IDE     for IDE
```

This is **much better** than:

```text
Student 1
Docker
SonarQube
Scanner
Token
Ports
Volumes

Student 2
Docker
SonarQube
Scanner
Token
Ports
Volumes

Student 3
Docker
SonarQube
Scanner
Token
Ports
Volumes
```

The second approach creates too many places for beginners to fail before they even learn what SonarQube is.

---

# Even easier: SonarQube Cloud

If your training repositories are on GitHub and internet access is reliable, **SonarQube Cloud may actually be easier than Docker for the full Lab 10**.

SonarQube Cloud can connect to GitHub repositories, and eligible GitHub projects can use automatic analysis without configuring a local SonarQube server. ([SonarSource Documentation][7])

Then the architecture becomes:

```text
GitHub Repository
       │
       ▼
SonarQube Cloud
       │
       ├── Quality Gate
       ├── Security
       ├── Reliability
       ├── Maintainability
       └── Issues
              │
              ▼
        SonarQube for IDE
              │
              ▼
           VS Code
```

No local SonarQube Docker container.

---

# Option A — Simplest Lab 10

If you want **zero infrastructure headache**, rename Lab 10 slightly:

## Lab 10 — Analyze Code Quality with SonarQube for IDE

Tools:

```text
VS Code
+
SonarQube for IDE
+
Lab02Api
```

Students learn:

```text
1. Install extension
2. Open project
3. Open source file
4. Observe Sonar findings
5. Open Problems panel
6. Understand rules
7. Give finding to AI
8. Validate AI explanation
9. Record findings
```

### Advantage

Extremely easy.

### Disadvantage

You cannot properly teach:

```text
Quality Gate
Project dashboard
Project-wide metrics
Jenkins Quality Gate integration
```

So it doesn't fully satisfy the existing Lab 10 objective.

---

# Option B — Best for your complete 30-Lab course

This is the option I recommend.

### Student installs

```text
VS Code
      +
SonarQube for IDE
      +
.NET SDK
      +
SonarScanner for .NET
```

### Instructor provides

```text
ONE SonarQube Community Build server
```

or:

```text
SonarQube Cloud
```

Then students learn in stages.

### Stage 1

```text
VS Code
   ↓
SonarQube for IDE
   ↓
Local findings
```

### Stage 2

```text
SonarQube for IDE
   ↓
Connected Mode
   ↓
Central SonarQube
```

SonarQube for VS Code has a Connected Mode wizard where users can add either a SonarQube Server or SonarQube Cloud connection. ([SonarSource Documentation][8])

### Stage 3

```text
SonarScanner for .NET
        ↓
Full project scan
        ↓
SonarQube Dashboard
```

### Stage 4

```text
Dashboard
   ↓
Quality Gate
   ↓
Metrics
```

That prepares students naturally for:

```text
Lab 11
Root Cause Analysis

Lab 12
Remediation

Lab 20
Jenkins + SonarQube Quality Gate
```

---

# How to install SonarScanner for .NET

This part is actually easy.

Once `.NET` is installed:

```powershell
dotnet tool install --global dotnet-sonarscanner
```

Verify:

```powershell
dotnet sonarscanner --version
```

SonarScanner for .NET is intended to run in the same environment where the .NET application is built. ([SonarSource Documentation][9])

So students do **not** need to think of it as another application with a UI.

It's simply:

```text
PowerShell command
```

---

# Connected Mode in VS Code

Once the instructor's SonarQube project exists:

In VS Code, click the:

```text
SonarQube
```

icon.

Look for:

```text
SONARQUBE SETUP

CONNECTED MODE
```

Choose:

```text
Add SonarQube Server Connection
```

or:

```text
Add SonarQube Cloud Connection
```

Then bind:

```text
Local Lab02Api
       ↓
SonarQube Lab02Api Project
```

Sonar documents this wizard as the normal Connected Mode setup. ([SonarSource Documentation][8])

Once connected:

```text
SERVER RULES
     ↓
VS CODE

Quality profile
Issue status
Project configuration
Quality Gate notifications
```

are more closely synchronized. ([SonarSource Documentation][6])

---

# I would change your Lab 10 sequence to this

```text
LAB 10
Run a SonarQube Scan and Establish Quality Metrics


PART A
What is SonarQube?
     ↓

PART B
Understand:
SonarQube for IDE
vs
SonarScanner
vs
SonarQube Server/Cloud
     ↓

PART C
Install SonarQube for IDE
in VS Code
     ↓

PART D
Open Lab02Api
     ↓

PART E
See first Sonar issue
in VS Code
     ↓

PART F
Ask AI to explain issue
     ↓

PART G
Connect VS Code to
central SonarQube / Cloud
     ↓

PART H
Install SonarScanner for .NET
     ↓

PART I
Run complete project scan
     ↓

PART J
Open SonarQube Dashboard
     ↓

PART K
Understand:
Security
Reliability
Maintainability
Duplication
Coverage
     ↓

PART L
Understand Quality Gate
     ↓

PART M
Create
SONARQUBE_BASELINE.md
```

## My recommendation for your course

**Do not make every participant install SonarQube through Docker in Lab 10.**

Use:

```text
STUDENTS
────────────────────────────
VS Code
SonarQube for IDE
SonarScanner for .NET


INSTRUCTOR / SHARED
────────────────────────────
SonarQube Community Build

OR

SonarQube Cloud
```

This keeps the important enterprise concept—the **SonarQube dashboard and Quality Gate**—while removing most of the infrastructure complexity from the student.

And if you want the absolute easiest first exercise, **start Lab 10 with SonarQube for IDE only**. Students can literally install the extension, open `SecurityDemoService.cs`, and start seeing Sonar findings. Only after they understand what a Sonar finding is should you introduce **server → scanner → project metrics → Quality Gate**.

That sequence will make Labs 10, 11, 12 and 20 much easier to follow.

[1]: https://docs.sonarsource.com/sonarqube-for-vs-code/getting-started/installation?utm_source=chatgpt.com "Installation | VS Code | Sonar Documentation"
[2]: https://docs.sonarsource.com/sonarqube-for-vs-code?utm_source=chatgpt.com "Homepage | VS Code | Sonar Documentation"
[3]: https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/scanners/dotnet/introduction?utm_source=chatgpt.com "Introduction | SonarQube Community Build | Sonar Documentation"
[4]: https://docs.sonarsource.com/sonarqube-community-build/quality-standards-administration/managing-quality-gates/introduction-to-quality-gates?utm_source=chatgpt.com "Understanding quality gates | SonarQube Community Build | Sonar Documentation"
[5]: https://docs.sonarsource.com/sonarqube-for-vs-code/getting-started/running-an-analysis?utm_source=chatgpt.com "Running an analysis | VS Code | Sonar Documentation"
[6]: https://docs.sonarsource.com/sonarqube-for-vs-code/connect-your-ide/connected-mode?utm_source=chatgpt.com "Connected mode | VS Code | Sonar Documentation"
[7]: https://docs.sonarsource.com/sonarqube-cloud/getting-started/github?utm_source=chatgpt.com "Getting started with GitHub | SonarQube Cloud | Sonar Documentation"
[8]: https://docs.sonarsource.com/sonarqube-for-vs-code/connect-your-ide/setup?utm_source=chatgpt.com "Connected mode setup | VS Code | Sonar Documentation"
[9]: https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/dotnet-environments/getting-started-with-net?utm_source=chatgpt.com "Getting started with .NET | SonarQube Community Build | Sonar Documentation"
