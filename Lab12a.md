**Lab 12 should now be rewritten to follow the simplified SonarQube workflow from the new Labs 10 and 11.**

The original Lab 12 requires students to select a controlled number of SonarQube findings, make small remediations, build/test after each change, rerun SonarQube analysis, compare before/after metrics, and determine whether the Quality Gate improved. 

# Lab 12 — Remediate SonarQube Issues and Improve the Quality Gate

## 1. What is Lab 12 about?

You already completed:

```text
LAB 10
SonarQube detects issues
        ↓
WHAT is wrong?


LAB 11
Developer + AI investigate
        ↓
WHY is it wrong?


LAB 12
Developer + AI remediate
        ↓
HOW do we fix it safely?
        ↓
BUILD
        ↓
TEST
        ↓
SONAR RESCAN
        ↓
DID QUALITY IMPROVE?
```

Lab 12 is the first Sonar lab where we deliberately **modify source code**.

The main rule is:

> **Do not ask AI to “Fix all SonarQube issues.”**

Instead:

```text
ONE confirmed issue
      ↓
ONE approved remediation
      ↓
BUILD
      ↓
TEST
      ↓
SONAR CHECK
      ↓
NEXT ISSUE
```

SonarQube for IDE provides rule explanations and, for some rules, quick-fix mechanisms, but the fix still needs engineering validation. ([SonarSource Documentation][1])

---

# 2. Learning Objectives

By the end of Lab 12, participants should be able to:

* Select a confirmed SonarQube issue from Lab 11.
* Distinguish **root-cause remediation** from simply hiding a warning.
* Ask AI for multiple remediation options.
* Review AI recommendations before changing source code.
* Implement the smallest safe change.
* Build the application after remediation.
* Run relevant automated/manual tests.
* Recheck the issue in SonarQube for IDE.
* Run a full SonarQube scan when Server/Cloud is available.
* Compare before/after quality metrics.
* Understand whether the Quality Gate passes or fails.
* Identify the exact remaining Quality Gate condition.
* Produce a SonarQube remediation report.

---

# 3. Tools Required

## Beginner path

```text
Visual Studio Code
        +
SonarQube for IDE
        +
.NET SDK
        +
Git
        +
Cursor or GitHub Copilot
```

## Full SonarQube path

Add:

```text
SonarQube Server / Community Build
OR
SonarQube Cloud

        +

SonarScanner for .NET
```

If VS Code is using **Connected Mode**, SonarQube for IDE can synchronize more closely with the bound SonarQube project, including project rules/settings and notifications about new issues or Quality Gate changes. ([SonarSource Documentation][2])

---

# 4. Starting Point

Open:

```text
C:\AI_SDLC_Labs\Lab02Api
```

You should already have:

```text
Lab02Api
│
├── Program.cs
├── OrderService.cs
├── CustomerReportService.cs
├── PerformanceService.cs
├── SecurityDemoService.cs
├── SONARQUBE_BASELINE.md
├── SONARQUBE_ROOT_CAUSE_ANALYSIS.md
└── Lab02Api.csproj
```

The most important input for Lab 12 is:

```text
SONARQUBE_ROOT_CAUSE_ANALYSIS.md
```

from Lab 11.

---

# Part A — Decide What You Are Allowed to Fix

## Step 1 — Open the Lab 11 report

Open:

```text
SONARQUBE_ROOT_CAUSE_ANALYSIS.md
```

Suppose you have:

```text
Finding 01
Classification: TRUE POSITIVE
Priority: P1

Finding 02
Classification: TRUE POSITIVE
Priority: P2

Finding 03
Classification: NEEDS MORE CONTEXT
Priority: P2
```

For Lab 12:

```text
Finding 01
        ↓
FIX CANDIDATE


Finding 02
        ↓
FIX CANDIDATE


Finding 03
        ↓
DO NOT MODIFY YET
```

Only remediate issues whose root cause you understand well enough to make a controlled change.

---

# Step 2 — Do not change False Positives just to satisfy Sonar

Suppose Lab 11 concluded:

```text
FALSE POSITIVE
```

Do not rewrite correct code merely to remove the Sonar message.

When using Connected Mode with an appropriate SonarQube Server setup and permissions, issues can be resolved as **Accepted** or **False positive** instead of changing code unnecessarily. ([SonarSource Documentation][1])

For the lab, record:

```text
Classification:
FALSE POSITIVE

Code Change:
None

Reason:
Sonar analysis does not correctly apply to
this specific code/context.

Evidence:
[Lab 11 evidence]
```

---

# Part B — Establish the BEFORE State

## Step 3 — Build first

Before changing anything:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Run:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

Why?

Because if the application was already broken:

```text
Before remediation
      ↓
BUILD FAILS
```

you cannot blame the remediation later.

---

# Step 4 — Run existing tests

If your test project already exists from the later testing exercises or you've added tests while developing:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api.Tests
dotnet test
```

Record:

```text
BEFORE

Build:
PASS

Tests:
[actual count]

Failed:
[actual count]
```

If the test project does not exist yet at this point in your classroom flow, use the manual functional tests you established in previous labs.

---

# Step 5 — Record the Sonar issue BEFORE changing it

In VS Code:

```text
Ctrl + Shift + M
```

Open:

```text
PROBLEMS
```

Select the actual Sonar issue.

Create:

```text
SONAR_REMEDIATION_01.md
```

Record:

```markdown
# Sonar Remediation 01

## BEFORE

File:
[actual]

Line:
[actual]

Rule:
[actual]

Severity:
[actual]

Software Quality:
[actual]

Message:
[actual]

Classification from Lab 11:
TRUE POSITIVE

Root Cause:
[actual Lab 11 conclusion]

Priority:
P0 / P1 / P2 / P3
```

---

# Part C — Ask AI for Remediation OPTIONS

## Step 6 — Don't let AI change code yet

Open Cursor or GitHub Copilot Chat.

Use:

```text
You are a Senior .NET Developer helping remediate
a confirmed SonarQube finding.

Do NOT modify my code yet.

SONARQUBE FINDING

File:
[FILE]

Rule:
[RULE]

Severity:
[SEVERITY]

Message:
[MESSAGE]

Affected Code:
[CODE]

LAB 11 ANALYSIS

Classification:
TRUE POSITIVE

Confirmed Root Cause:
[ROOT CAUSE]

Priority:
[PRIORITY]

Give me THREE possible remediation approaches.

For each provide:

1. Proposed change
2. How it addresses the root cause
3. Files affected
4. Expected benefit
5. Implementation effort:
   Low / Medium / High
6. Regression risk:
   Low / Medium / High
7. Tests required
8. Whether the change addresses:
   - symptom only
   - root cause

Recommend the SMALLEST SAFE remediation.

Do NOT implement anything.
Do NOT suppress the SonarQube rule.
Do NOT change unrelated code.
```

---

# 7. Why ask for three options?

Because AI might propose:

```text
OPTION A
Small refactoring


OPTION B
New architecture


OPTION C
Install another framework
```

Perhaps all three could work.

But Lab 12 teaches:

```text
Choose solution based on

Risk
+
Effort
+
Root Cause
+
Existing Architecture
```

not:

```text
Choose biggest solution
because AI sounds confident.
```

---

# Part D — Challenge the Proposed Fix

## Step 8 — Ask whether it merely hides Sonar

Use:

```text
Challenge your recommended remediation.

Could this change make the SonarQube warning
disappear WITHOUT actually improving the code?

Check specifically for:

- rule suppression
- disabling a Sonar rule
- excluding the file from analysis
- adding meaningless comments
- cosmetic restructuring
- moving bad code elsewhere
- introducing unnecessary abstractions
- changing behavior solely to satisfy the scanner

Explain:

1. Symptom being addressed
2. Root cause being addressed
3. Remaining risk
4. Regression risk

Do NOT modify code.
```

The goal is:

```text
BAD

Sonar warning
     ↓
Suppress warning
     ↓
Warning disappears
     ↓
Underlying problem remains
```

versus:

```text
GOOD

Sonar warning
     ↓
Understand root cause
     ↓
Change implementation
     ↓
Underlying quality improves
```

---

# Part E — Implement ONE Remediation

## Step 9 — Approve the smallest safe change

Once you've reviewed the options:

```text
Implement ONLY the approved remediation.

Requirements:

1. Address the confirmed root cause.
2. Preserve existing business behavior.
3. Modify only required files.
4. Do not suppress Sonar rules.
5. Do not disable Sonar analysis.
6. Do not exclude the file from SonarQube.
7. Do not introduce new packages unless essential.
8. Keep the change small enough to review.
9. Show exactly what changed.
10. Do not modify another SonarQube issue yet.
```

Let Cursor/Copilot make the controlled change.

---

# Part F — Review the Diff

## Step 10 — Don't immediately assume the AI change is correct

If using Git:

```powershell
git diff
```

Look for:

```text
Expected change
        ✓

Unrelated formatting
        ✗

Other methods rewritten
        ✗

New package
        ?

Business logic changed
        ?
```

Ask AI:

```text
Review ONLY the current Git diff.

Do NOT make further changes.

Identify:

1. Changes directly required for the Sonar remediation.
2. Unrelated changes.
3. Possible behavior changes.
4. New dependencies.
5. Regression risks.
6. Anything that should be reverted.

Do not judge success merely because the
Sonar warning may disappear.
```

---

# Part G — Build Immediately

## Step 11 — Run the build

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
dotnet build
```

Expected:

```text
Build succeeded.
```

If:

```text
Build FAILED
```

stop.

Do not move to another Sonar issue.

---

# Step 12 — If build fails

Give AI the actual error:

```text
The project failed to build immediately after
the approved SonarQube remediation.

Do NOT rewrite the application.

Build Error:

[PASTE ERROR]

Current Diff:

[PASTE RELEVANT DIFF]

Determine:

1. What caused the compilation failure.
2. Whether the remediation caused it.
3. The smallest correction required.

Do not change unrelated code.
```

Then rebuild.

---

# Part H — Test the Business Behavior

## Step 13 — Why build is not enough

This:

```text
Build succeeded
```

means:

> The compiler accepts the code.

It does **not** mean:

```text
Business rules are still correct.
```

Therefore:

```text
CODE CHANGE
     ↓
BUILD
     ↓
FUNCTIONAL TEST
```

---

# Step 14 — Run the relevant test

For example, if the change affects:

```text
CustomerReportService
```

rerun the scenarios you've already established.

If it affects:

```text
OrderService
```

rerun its pricing scenarios.

The important pattern is:

```text
BEFORE

Input
↓
Expected Output


AFTER

Same Input
↓
Same Expected Output
```

unless the purpose of the remediation is specifically to correct defective behavior.

---

# Step 15 — If xUnit tests exist

Run:

```powershell
dotnet test
```

Record:

```text
Build:
PASS

Tests:
8

Passed:
8

Failed:
0
```

Use actual values.

Do not invent the numbers.

---

# Part I — Let SonarQube for IDE Reanalyze

## Step 16 — Save the modified file

In VS Code:

```text
Ctrl + S
```

SonarQube for IDE analyzes supported files as they are edited/opened/saved and updates the local findings it reports. ([SonarSource Documentation][1])

Open:

```text
Ctrl + Shift + M
```

Check:

```text
PROBLEMS
```

Find the original issue.

---

# Step 17 — Interpret the result

You may see:

```text
BEFORE

Sonar Issue
OPEN
```

and after the local edit:

```text
AFTER

Issue no longer shown
```

This is good **local evidence**.

But be careful:

> Disappearing from SonarQube for IDE is not the same thing as proving the server-side project Quality Gate has passed.

For that we need:

```text
FULL SERVER/CLOUD ANALYSIS
```

---

# Part J — Connected Mode Path

If your classroom is using:

```text
SonarQube Server
```

or:

```text
SonarQube Cloud
```

and VS Code is connected to the project, you can now compare the local fix with the central analysis.

Connected Mode binds the local workspace with the SonarQube project so rules and project issue information can be synchronized more consistently. ([SonarSource Documentation][2])

---

# Part K — Full SonarScanner Path

## Step 18 — Check scanner

Run:

```powershell
dotnet sonarscanner --version
```

If unavailable:

```powershell
dotnet tool install --global dotnet-sonarscanner
```

SonarScanner for .NET integrates analysis into the .NET build workflow using a scanner **begin → build → end** sequence. ([SonarSource Documentation][3])

---

# Step 19 — Set the Sonar token

For a local SonarQube Server/Community Build example:

```powershell
$env:SONAR_TOKEN="YOUR_TOKEN"
```

Do not:

```text
paste token into C# code
```

Do not:

```text
commit token into Git
```

---

# Step 20 — Begin analysis

Using the actual project key from Lab 10:

```powershell
dotnet sonarscanner begin `
  /k:"lab02api" `
  /d:sonar.host.url="http://YOUR-SONAR-SERVER" `
  /d:sonar.token="$env:SONAR_TOKEN"
```

If your class is using SonarQube Cloud instead, use the configuration that was verified for that project rather than blindly copying the server command.

---

# Step 21 — Build inside the analysis

```powershell
dotnet build --no-incremental
```

---

# Step 22 — End analysis

```powershell
dotnet sonarscanner end `
  /d:sonar.token="$env:SONAR_TOKEN"
```

This submits the analysis for server-side processing. ([SonarSource Documentation][3])

---

# Part L — Open the SonarQube Dashboard

## Step 23 — Open your project

Go to the SonarQube web interface.

Select:

```text
Projects
   ↓
Lab02Api
```

Now compare:

```text
BEFORE ANALYSIS

vs

AFTER ANALYSIS
```

---

# Part M — Was the Issue Really Fixed?

## Step 24 — Find the original issue

Go to:

```text
Lab02Api
   ↓
Issues
```

Search for:

```text
Rule
File
Original issue
```

After subsequent analysis, SonarQube determines whether a previously detected issue is still present; if it no longer detects that issue, the lifecycle can recognize it as fixed. ([SonarSource Documentation][4])

You want evidence like:

```text
BEFORE

Issue:
OPEN


AFTER RESCAN

Issue:
FIXED
```

That's much stronger than:

```text
Cursor says:
"I fixed the problem."
```

---

# Part N — Compare Metrics

## Step 25 — Record actual BEFORE and AFTER values

Create a table:

| Metric            | Before |  After | Change |
| ----------------- | -----: | -----: | -----: |
| Quality Gate      | Actual | Actual |        |
| Issues            | Actual | Actual |        |
| Security          | Actual | Actual |        |
| Reliability       | Actual | Actual |        |
| Maintainability   | Actual | Actual |        |
| Security Hotspots | Actual | Actual |        |
| Duplication       | Actual | Actual |        |
| Coverage          | Actual | Actual |        |

Use **actual SonarQube values only**.

---

# Part O — Understand the Quality Gate

A Quality Gate is basically:

```text
PROJECT ANALYSIS
       ↓
QUALITY CONDITIONS
       ↓
Are requirements satisfied?
       │
    ┌──┴──┐
    ▼     ▼
   YES    NO
    │      │
   PASS   FAIL
```

SonarQube uses Quality Gates to define conditions code must meet; the built-in Sonar way gate is part of its recommended quality standards. ([SonarSource Documentation][5])

---

# Step 26 — Quality Gate PASS

If you see:

```text
Quality Gate

PASSED
```

record:

```text
Before:
FAILED

After:
PASSED
```

Then identify **which condition improved**.

Do not simply write:

```text
AI fixed SonarQube.
```

Instead:

```text
Finding XYZ was remediated.

New analysis no longer reports the issue.

Relevant metric improved.

Quality Gate changed from FAIL to PASS.
```

---

# Step 27 — Quality Gate still FAILS

This is also a valid result.

Example:

```text
Issue fixed
       ↓
Maintainability improved
       ↓
BUT
       ↓
Coverage condition still fails
       ↓
QUALITY GATE = FAIL
```

Record:

```text
Remediation:
Successful

Quality Gate:
Still Failed

Remaining Condition:
[actual failing condition]
```

This teaches students that:

> **Fixing one Sonar issue does not guarantee the whole project passes the Quality Gate.**

---

# Part P — Do NOT Weaken the Quality Gate Just to Get Green

Suppose the Quality Gate fails.

Wrong response:

```text
Gate is failing
      ↓
Reduce requirements
      ↓
Green!
```

That doesn't necessarily improve software quality.

Instead:

```text
Quality Gate failed
      ↓
Which condition failed?
      ↓
Why?
      ↓
What engineering action is required?
```

A project can be configured to use a different Quality Gate, but changing the gate is a project administration decision and should not be used merely to make a lab result appear successful. ([SonarSource Documentation][6])

---

# Part Q — Repeat One Issue at a Time

## Step 28 — Remediate Finding 02

Only after Finding 01 is:

```text
Build ✓
Test ✓
Sonar ✓
```

move to:

```text
Finding 02
```

Repeat:

```text
Lab 11 Root Cause
       ↓
AI remediation options
       ↓
Human review
       ↓
Implement ONE fix
       ↓
git diff
       ↓
dotnet build
       ↓
dotnet test
       ↓
Sonar check
```

---

# Part R — Use Git Checkpoints

After a successful remediation:

```powershell
git status
```

Then:

```powershell
git add .
```

Then:

```powershell
git commit -m "Remediate SonarQube finding 01"
```

For the next issue:

```powershell
git commit -m "Remediate SonarQube finding 02"
```

This gives you:

```text
Commit 1
Finding 01

Commit 2
Finding 02

Commit 3
Finding 03
```

instead of:

```text
One huge:
"Fix all Sonar issues"
commit
```

---

# Part S — Ask AI to Calculate Improvement

Once you have actual before/after numbers:

```text
Compare these actual SonarQube results.

BEFORE

Quality Gate:
[...]

Security:
[...]

Reliability:
[...]

Maintainability:
[...]

Issues:
[...]

Duplication:
[...]

Coverage:
[...]

AFTER

Quality Gate:
[...]

Security:
[...]

Reliability:
[...]

Maintainability:
[...]

Issues:
[...]

Duplication:
[...]

Coverage:
[...]

For each metric provide:

1. Before
2. After
3. Absolute change
4. Percentage change where mathematically meaningful
5. Engineering interpretation

Do not invent missing values.

If the Quality Gate remains failed,
identify the exact remaining failed condition.

Do NOT recommend weakening the Quality Gate
merely to obtain PASS.
```

---

# Part T — Create the Final Lab 12 Report

Create:

```text
SONARQUBE_REMEDIATION_REPORT.md
```

Use this prompt:

```text
Create SONARQUBE_REMEDIATION_REPORT.md using ONLY
actual evidence from Labs 10, 11 and 12.

# Lab 12 — SonarQube Remediation Report

## 1. Project

Lab02Api

## 2. Environment

SonarQube for IDE:
Installed / Not Installed

Connected Mode:
Yes / No

SonarQube Server / Cloud:
[Actual]

SonarScanner:
[Actual]

## 3. Baseline

Quality Gate:
[...]

Issues:
[...]

Security:
[...]

Reliability:
[...]

Maintainability:
[...]

Coverage:
[...]

Duplication:
[...]

## 4. Finding 01

File:
[...]

Rule:
[...]

Severity:
[...]

Lab 11 Classification:
[...]

Lab 11 Root Cause:
[...]

Priority:
[...]

## 5. Remediation Options

Option A:
[...]

Option B:
[...]

Option C:
[...]

## 6. Approved Remediation

Selected:
[...]

Reason:
[...]

Expected Risk:
[...]

## 7. Implementation

Files Changed:
[...]

Summary:
[...]

## 8. Build Validation

Command:
dotnet build

Result:
[...]

## 9. Functional / Automated Testing

Tests:
[...]

Result:
[...]

## 10. Sonar Validation

Local SonarQube for IDE:
[...]

Server Analysis:
[...]

Issue Status After Analysis:
[...]

## 11. Finding 02

Repeat same structure if completed.

## 12. Before vs After Metrics

Metric | Before | After | Change

## 13. Quality Gate

Before:
[...]

After:
[...]

If Still Failed:

Failed Condition:
[...]

Reason:
[...]

Recommended Next Action:
[...]

## 14. Deferred Findings

Finding
Reason
Priority
Next Action

## 15. False Positives / Accepted Findings

Document only actual examples.

## 16. Remaining Risks

[...]

## 17. Conclusion

State measurable improvements.

Do NOT claim an issue is fixed without evidence.

Do NOT invent metrics.
```

---

# Main Lab 12 AI Prompt

For students, I would give them this **single reusable master prompt**:

```text
You are a Senior .NET Developer performing
SonarQube remediation.

I am a beginner.

SONARQUBE FINDING

File:
[FILE]

Line:
[LINE]

Rule:
[RULE]

Severity:
[SEVERITY]

Message:
[MESSAGE]

Affected Code:
[CODE]

LAB 11 ANALYSIS

Classification:
TRUE POSITIVE

Confirmed Root Cause:
[ROOT CAUSE]

Impact:
[IMPACT]

Priority:
[PRIORITY]

STEP 1

Do NOT modify code.

Give THREE remediation approaches.

For every approach provide:

- proposed change
- root cause addressed
- effort
- regression risk
- tests required
- files affected

STEP 2

Recommend the smallest safe approach.

STEP 3

Challenge the recommendation.

Determine whether it actually addresses
the root cause or merely hides the Sonar warning.

Check for:

- rule suppression
- file exclusion
- disabling analysis
- cosmetic changes
- unnecessary abstractions

STEP 4

Wait until I approve an approach.

When approved:

- implement ONLY that remediation
- preserve business behavior
- modify only required files
- do not suppress Sonar
- do not modify other findings

STEP 5

After implementation, explain:

- exact files changed
- exact logic changed
- regression risk
- build required
- tests required
- Sonar validation required

Do NOT claim the issue is fixed until:

1. Project builds.
2. Required tests pass.
3. SonarQube no longer reports the issue
   in the applicable analysis.
```

---

# Lab 12 Completion Checklist

* [ ] Lab 11 root-cause report opened.
* [ ] Only confirmed findings selected.
* [ ] Original Sonar evidence recorded.
* [ ] Baseline build succeeds.
* [ ] Baseline tests recorded.
* [ ] Three remediation options generated.
* [ ] AI recommendation challenged.
* [ ] Smallest safe remediation selected.
* [ ] No Sonar rule suppressed.
* [ ] No file excluded merely to hide the issue.
* [ ] Only one finding changed at a time.
* [ ] Git diff reviewed.
* [ ] `dotnet build` passes.
* [ ] Relevant functionality tested.
* [ ] Automated tests pass where available.
* [ ] SonarQube for IDE rechecks modified code.
* [ ] Original local issue checked.
* [ ] Full SonarQube scan performed where Server/Cloud is available.
* [ ] Original server-side issue checked.
* [ ] Before/after metrics recorded.
* [ ] Quality Gate checked.
* [ ] Exact failed Quality Gate condition recorded if still failing.
* [ ] False Positives are not “fixed” by unnecessary code changes.
* [ ] Each completed remediation has a Git checkpoint.
* [ ] `SONARQUBE_REMEDIATION_REPORT.md` created.

# The Diagram to Remember

```text
                         LAB 10
                            │
                            ▼
                     SONAR DETECTS
                            │
                            ▼
                         LAB 11
                            │
                            ▼
                       WHY?
                    ROOT CAUSE
                            │
                            ▼
                         LAB 12
                            │
                            ▼
                   SELECT TRUE POSITIVE
                            │
                            ▼
                   AI GIVES 3 OPTIONS
                            │
                            ▼
                     HUMAN REVIEWS
                            │
                            ▼
                    SMALLEST SAFE FIX
                            │
                            ▼
                         git diff
                            │
                            ▼
                       dotnet build
                            │
                            ▼
                       dotnet test
                            │
                            ▼
                    SONAR LOCAL CHECK
                            │
                            ▼
                  FULL SONAR RESCAN
                     if available
                            │
                            ▼
                   ISSUE STILL THERE?
                      /           \
                    YES            NO
                     │              │
                     ▼              ▼
                 INVESTIGATE      FIXED
                                    │
                                    ▼
                              QUALITY GATE
                               /          \
                            PASS          FAIL
                             │             │
                             ▼             ▼
                         COMPLETE      FIND EXACT
                                       CONDITION
```

The three Sonar labs now become very easy for beginners to remember:

```text
LAB 10
WHAT?
Sonar detects the problem.

LAB 11
WHY?
AI helps investigate the root cause.

LAB 12
HOW?
AI proposes remediation,
human approves,
then BUILD → TEST → RESCAN → PROVE.
```

That is the version I recommend using for the course because it works for both the **easy VS Code SonarQube for IDE route** and the later **full SonarQube Server/Cloud + Quality Gate route**, without forcing every beginner to manage Docker infrastructure.

[1]: https://docs.sonarsource.com/sonarqube-for-vs-code/using/fixing-issues?utm_source=chatgpt.com "Fixing issues | VS Code | Sonar Documentation"
[2]: https://docs.sonarsource.com/sonarqube-for-vs-code/connect-your-ide/connected-mode?utm_source=chatgpt.com "Connected mode | VS Code | Sonar Documentation"
[3]: https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/scanners/dotnet/using?utm_source=chatgpt.com "Using the scanner | SonarQube Community Build | Sonar Documentation"
[4]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/solution-overview?utm_source=chatgpt.com "Issue management solution | SonarQube Community Build | Sonar Documentation"
[5]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/clean-as-you-code/about-quality-standards/?utm_source=chatgpt.com "Quality standards and new code | SonarQube Community Build | Sonar Documentation"
[6]: https://docs.sonarsource.com/sonarqube-community-build/project-administration/adjusting-analysis/changing-quality-gate-and-fudge-factor?utm_source=chatgpt.com "Changing quality gate | SonarQube Community Build | Sonar Documentation"
