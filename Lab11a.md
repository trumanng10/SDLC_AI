 **beginner-friendly Lab 10**.

The original Lab 11 objective remains the same: interpret SonarQube findings, group related findings into underlying causes, use AI to explain them, verify the explanation against real code, and create a remediation backlog. 

# Lab 11 — Use AI for Root-Cause Analysis of SonarQube Findings

## 1. What is Lab 11 about?

In Lab 10 you learned:

```text
SOURCE CODE
     ↓
SonarQube for IDE
     ↓
Finds an issue
     ↓
"This code may have a problem"
```

Lab 11 asks the next question:

```text
WHY?
```

So:

```text
LAB 10
WHAT is wrong?
     ↓
Sonar finds issue

LAB 11
WHY is it wrong?
     ↓
AI + Developer investigate

LAB 12
HOW should we fix it?
     ↓
Remediation
```

**Do not fix the code in Lab 11.**

The goal is investigation.

---

# 2. Learning Objectives

By the end of Lab 11, participants should be able to:

* Open SonarQube findings in VS Code.
* Understand the Sonar rule that triggered the issue.
* Identify the affected source code.
* Distinguish **symptom** from **root cause**.
* Give Sonar evidence to GitHub Copilot/Cursor.
* Ask AI to generate root-cause hypotheses.
* Validate AI reasoning against the actual source.
* Perform a simple Five Whys analysis.
* Decide whether a finding is:

  * Confirmed / True Positive
  * False Positive
  * Needs More Context
* Determine the likely impact.
* Prioritize findings.
* Produce a root-cause analysis report.
* Prepare findings for remediation in Lab 12.

---

# 3. Tools Required

You should already have these from Lab 10:

```text
Required
────────────────────────
Visual Studio Code
SonarQube for IDE extension
.NET SDK
Lab02Api project
GitHub Copilot OR Cursor


Optional
────────────────────────
SonarQube Server / Community Build
OR
SonarQube Cloud
Connected Mode
```

You **do not need Docker specifically for Lab 11** if you are doing the beginner/local-IDE path.

SonarQube for IDE automatically analyzes supported files when they are opened or saved and reports findings in the VS Code **Problems** panel. ([SonarSource Documentation][1])

---

# 4. Starting Project

Open:

```text
C:\AI_SDLC_Labs\Lab02Api
```

You may already have files such as:

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

For the beginner exercise, pick **one actual Sonar issue** from any of these files.

Do not manufacture a finding if Sonar already gives you one.

---

# 5. First understand: What is a Sonar issue?

Sonar analyzes your source code using coding rules.

Conceptually:

```text
SOURCE CODE
     ↓
SONAR RULES
     ↓
Does code violate a rule?
     │
   ┌─┴─┐
   │   │
  NO  YES
       │
       ▼
     ISSUE
```

Sonar defines an issue as a problem raised when code breaks one of its coding rules. An issue can carry software-quality impact and severity information. ([SonarSource Documentation][2])

---

# Part A — Find Your First Issue in VS Code

## Step 1 — Open Visual Studio Code

Open:

```text
Visual Studio Code
```

Then:

**File → Open Folder**

Select:

```text
C:\AI_SDLC_Labs\Lab02Api
```

---

# Step 2 — Check SonarQube for IDE

Click:

```text
Extensions
```

or:

```text
Ctrl + Shift + X
```

Confirm:

```text
SonarQube for IDE
Publisher: SonarSource
```

is installed.

---

# Step 3 — Open a C# file

Try:

```text
CustomerReportService.cs
```

or:

```text
SecurityDemoService.cs
```

Save the file:

```text
Ctrl + S
```

SonarQube for IDE normally triggers analysis when supported files are opened or saved. ([SonarSource Documentation][1])

---

# Step 4 — Open the Problems panel

Press:

```text
Ctrl + Shift + M
```

or:

**View → Problems**

You may see:

```text
PROBLEMS

Lab02Api

CustomerReportService.cs
  SonarQube: ...

SecurityDemoService.cs
  SonarQube: ...
```

Your actual findings will be different.

**Use actual results.**

---

# Step 5 — Pick ONE issue

Do not analyze 20 findings at once.

Choose:

```text
Issue 01
```

For example:

```text
File:
CustomerReportService.cs

Line:
[actual]

Sonar Message:
[actual]

Severity:
[actual]

Rule:
[actual]
```

Start small.

---

# Part B — Understand the Sonar Rule

## Step 6 — Open the issue details

In the **Problems** panel:

Right-click the Sonar finding.

Choose the SonarQube option to show the issue/rule details.

Current SonarQube for VS Code documentation describes the action as:

```text
SonarQube: Show issue details for ...
```

The resulting rule view can show the rule explanation, examples, software-quality classification and severity. ([SonarSource Documentation][3])

---

# Step 7 — Ask these five questions

Before using AI, answer:

```text
1. Which file?

2. Which line?

3. What rule was violated?

4. What does Sonar say is wrong?

5. Why does Sonar consider it a problem?
```

Do not ask:

```text
How do I make the warning disappear?
```

yet.

---

# Step 8 — Create an evidence file

In VS Code create:

```text
SONAR_ISSUE_01.md
```

Put:

```markdown
# SonarQube Issue 01

## File

[Actual filename]

## Line

[Actual line]

## Rule

[Actual Sonar rule]

## Severity

[Actual severity]

## Software Quality

[Security / Reliability / Maintainability
if shown]

## SonarQube Message

[Paste actual message]

## Affected Code

[Paste only relevant code]

## Why Sonar Raised It

[Summarize the Sonar rule explanation]
```

Do not invent anything.

---

# Part C — Symptom vs Root Cause

This is the most important concept in Lab 11.

Suppose Sonar reports:

```text
This method is too complex.
```

That is a:

```text
SYMPTOM
```

Why is the method complex?

Perhaps:

```text
Many nested IF statements
       ↓
Multiple business rules
       ↓
Validation + classification + formatting
all mixed together
       ↓
Method has too many responsibilities
```

Then:

```text
Sonar Finding
"Method complexity is high"

          ↓

Immediate Cause
"Many nested decisions"

          ↓

Root Cause
"Several responsibilities are implemented
inside one method"
```

So:

> **Finding = What Sonar detected.**

> **Root cause = Why the code ended up in that condition.**

---

# Part D — Let AI Explain the Finding

## Step 9 — Open GitHub Copilot Chat or Cursor

Give AI the **evidence**, not just:

```text
Why Sonar complain?
```

Use this prompt:

```text
You are a Senior .NET Developer performing
root-cause analysis.

I am a beginner learning SonarQube.

Do NOT modify my code.

SonarQube reported:

File:
[FILE]

Line:
[LINE]

Rule:
[RULE]

Severity:
[SEVERITY]

Software Quality:
[SECURITY / RELIABILITY / MAINTAINABILITY]

Message:
[MESSAGE]

Affected code:

[PASTE CODE]

SonarQube rule explanation:

[PASTE OR SUMMARIZE RULE DESCRIPTION]

Explain:

1. What SonarQube detected.
2. What this rule means in beginner-friendly language.
3. Why this specific code triggered the rule.
4. What could happen if it is ignored.
5. Whether the problem appears local to this line or
   may originate from the surrounding method/design.

Do NOT suggest a fix yet.
Do NOT modify code.
Do NOT invent missing information.
```

---

# Step 10 — Check the AI explanation

Do not automatically believe it.

Compare:

```text
AI says
   ↓
"The method has four responsibilities"

Actual source
   ↓
Can we actually identify those responsibilities?
```

If AI says something that is **not in the code**, reject it.

Example:

```text
AI:
"This code writes to a SQL Server database."
```

But your code does not connect to SQL Server.

Then:

```text
REJECT THE ASSUMPTION
```

The original Lab 11 specifically requires verifying AI explanations against the real codebase and correcting assumptions about architecture or business rules. 

---

# Part E — Generate Root-Cause Hypotheses

## Step 11 — Ask AI for three possibilities

Now use:

```text
Do NOT modify the code.

Generate three possible root-cause hypotheses for
this SonarQube finding.

For each provide:

1. Hypothesis
2. Evidence supporting it
3. Evidence against it
4. Relevant source-code location
5. Confidence:
   HIGH / MEDIUM / LOW
6. Additional evidence needed

Rank them from most likely to least likely.

Do not assume the first explanation is correct.
```

You might get:

```text
HYPOTHESIS 1
Deep nested conditional flow
Confidence: HIGH


HYPOTHESIS 2
Method contains multiple responsibilities
Confidence: HIGH


HYPOTHESIS 3
Business rules were added incrementally without
structural refactoring
Confidence: LOW
```

Why is #3 lower confidence?

Because the **code** may show the structure, but it doesn't necessarily tell us its development history.

Good AI analysis should distinguish:

```text
EVIDENCE
vs
ASSUMPTION
```

---

# Part F — Review the Whole Method

## Step 12 — Do not analyze only one line

Suppose Sonar marks:

```text
Line 35
```

That doesn't necessarily mean:

```text
ROOT CAUSE = Line 35
```

Ask:

```text
Review the complete method containing the
SonarQube finding.

Do NOT modify anything.

Determine whether the issue is caused by:

- this single line
- surrounding control flow
- duplicated logic
- excessive responsibilities
- another method
- dependency design
- broader class design

Use exact source-code evidence.

Revise your root-cause hypotheses if necessary.
```

---

# Part G — Primary and Secondary Locations

Sonar issues can contain:

```text
Primary Location
       +
Secondary Locations
```

and some issue types can provide execution flows from one part of the code toward another. ([SonarSource Documentation][2])

For example:

```text
SOURCE
   ↓
Step 1
   ↓
Step 2
   ↓
PROBLEM LOCATION
```

If Sonar displays additional locations, include them in the AI prompt.

Use:

```text
SonarQube shows these related locations:

Primary:
[...]

Secondary:
[...]

Execution flow if shown:
[...]

Explain how these code locations are connected.

Identify which location is:

- symptom
- contributing cause
- most likely root cause

Do not change code.
```

---

# Part H — Five Whys Root-Cause Analysis

## Step 13 — Understand Five Whys

Example Sonar issue:

```text
Method complexity too high
```

Ask:

```text
WHY?
```

Because:

```text
There are many nested branches.
```

Ask again:

```text
WHY?
```

Because:

```text
Several business decisions occur in one method.
```

Again:

```text
WHY?
```

Because:

```text
Validation, discount classification and output
generation are combined.
```

We now have something closer to the root cause.

---

# Step 14 — Ask AI to perform Five Whys

Prompt:

```text
Perform a Five Whys analysis for this SonarQube finding.

Start with the exact Sonar finding.

For every "Why":

1. Give the explanation.
2. Show supporting source-code evidence.
3. Stop if the next answer would require speculation.

Do not invent organizational history.

Finish with:

Most Likely Root Cause:
[...]

Confidence:
HIGH / MEDIUM / LOW

Do NOT modify code.
```

The phrase:

```text
Stop if the next answer would require speculation.
```

is very important.

---

# Part I — Challenge the AI

## Step 15 — Make AI attack its own answer

Use:

```text
Challenge your own root-cause analysis.

Assume your first conclusion could be wrong.

For every proposed root cause provide:

1. Supporting evidence
2. Contradicting evidence
3. Alternative explanation
4. Missing information
5. Revised confidence

Then identify the single most likely root cause.

Do NOT modify code.
```

This prevents the workflow from becoming:

```text
Sonar says something
       ↓
AI says something
       ↓
Human believes everything
```

Instead:

```text
Sonar evidence
       ↓
AI hypothesis
       ↓
Challenge
       ↓
Human verification
```

---

# Part J — Decide Whether the Finding Is Real

## Step 16 — Use three classroom classifications

For Lab 11 use:

```text
TRUE POSITIVE
FALSE POSITIVE
NEEDS MORE CONTEXT
```

### True Positive

Sonar identified a genuine engineering problem.

```text
Sonar finding
     ↓
Source evidence confirms it
     ↓
TRUE POSITIVE
```

---

### False Positive

Sonar reports a problem but the rule does not correctly apply to this specific situation.

```text
Sonar finding
     ↓
Developer investigates
     ↓
Rule does not apply
     ↓
FALSE POSITIVE
```

SonarQube itself supports marking an issue **False positive** when authorized users determine the analysis is mistaken. ([SonarSource Documentation][4])

---

### Needs More Context

Example:

```text
AI:
"This value is insecure."

Developer:
"We need to know whether this is production data,
a test value or configuration supplied at runtime."
```

Then:

```text
NEEDS MORE CONTEXT
```

Do not force a Yes/No answer.

---

# Step 17 — Ask AI to classify it

Use:

```text
Using ONLY the evidence collected so far,
classify this SonarQube finding as:

TRUE POSITIVE
FALSE POSITIVE
NEEDS MORE CONTEXT

Consider:

- Sonar rule
- affected source
- surrounding method
- secondary locations if any
- business rules we actually know

Provide:

Classification:
[...]

Evidence:
[...]

Missing Information:
[...]

Confidence:
HIGH / MEDIUM / LOW

Do NOT modify code.
```

Important:

**True Positive** is our lab-review term. Sonar's actual issue lifecycle uses statuses such as **Open, Accepted, False positive and Fixed**. ([SonarSource Documentation][5])

---

# Part K — Optional: SonarQube Server / Cloud Path

If Lab 10 connected VS Code to:

```text
SonarQube Server
```

or:

```text
SonarQube Cloud
```

you can perform an extra investigation.

Open the project dashboard.

Go to:

```text
Project
   ↓
Issues
```

Find the same issue if it is available in the server-side analysis.

Compare:

```text
VS CODE
Local Sonar finding

        vs

SONARQUBE DASHBOARD
Server/project finding
```

Connected Mode allows SonarQube for IDE to use settings from a SonarQube Server, Community Build or Cloud project, so what students see locally can more closely align with project analysis. ([SonarSource Documentation][6])

---

# Part L — Determine Impact

## Step 18 — Ask what the issue affects

Use:

```text
Assuming this finding is a TRUE POSITIVE,
perform an impact assessment.

Rate:

Security:
NONE / LOW / MEDIUM / HIGH

Reliability:
NONE / LOW / MEDIUM / HIGH

Maintainability:
NONE / LOW / MEDIUM / HIGH

Also assess:

User Impact
Developer Impact
Operational Impact

For every rating provide source-code evidence.

Do not exaggerate the risk.
```

Sonar findings carry software-quality and severity information derived from their rules; the IDE can show these classifications in the issue/rule details. ([SonarSource Documentation][3])

---

# Part M — Prioritize the Finding

## Step 19 — Use a simple priority system

Use:

```text
P0 — Critical
Immediate action

P1 — High
Fix soon

P2 — Medium
Planned remediation

P3 — Low
Backlog / improvement
```

Ask:

```text
Prioritize this confirmed SonarQube finding.

Use:

P0
P1
P2
P3

Consider:

- Security impact
- Reliability impact
- Maintainability impact
- Likelihood
- Business impact
- Frequency of the code path
- Cost of future change

Explain the priority.

Do not classify everything as P0.
```

---

# Part N — Do NOT Remediate Yet

At this point students will be tempted to click:

```text
Quick Fix
```

or ask:

```text
Copilot, fix it.
```

Don't.

Lab 11 ends here:

```text
Finding
   ↓
Understand
   ↓
Root Cause
   ↓
Validate
   ↓
Classify
   ↓
Prioritize
```

Lab 12 is:

```text
Remediate
   ↓
Build
   ↓
Test
   ↓
Rescan
```

Sonar's IDE provides issue information and rule descriptions that can help with remediation, but Lab 11 deliberately stops before implementing the fix. ([SonarSource Documentation][7])

---

# Part O — Analyze 3 Actual Findings

Once Issue 01 is understood, repeat for:

```text
ISSUE 01
Prefer Maintainability


ISSUE 02
Prefer Reliability
if available


ISSUE 03
Prefer Security
if available
```

But:

> **Only use findings that actually exist.**

If your project has:

```text
3 Maintainability findings
0 Security findings
```

do not invent a security vulnerability just for the exercise.

---

# Part P — Group Related Findings

This brings Lab 11 back to the original course design.

Suppose Sonar reports:

```text
Finding A
Deeply nested conditions

Finding B
High cognitive complexity

Finding C
Duplicated branches
```

They might all have one underlying root cause:

```text
Business logic concentrated
inside one large method
```

So:

```text
Finding A ─┐
Finding B ─┼──→ COMMON ROOT CAUSE
Finding C ─┘
```

The original Lab 11 explicitly asks participants to group findings by file, rule or recurring pattern and identify common underlying engineering causes rather than treating every line as a separate problem. 

---

# Step 20 — Ask AI to cluster the findings

Prompt:

```text
These are actual SonarQube findings from my project.

Finding 1:
[...]

Finding 2:
[...]

Finding 3:
[...]

Do NOT modify code.

Determine whether any findings share the same
engineering root cause.

For each cluster provide:

Cluster Name:
[...]

Findings Included:
[...]

Common Root Cause:
[...]

Supporting Code Evidence:
[...]

Confidence:
HIGH / MEDIUM / LOW

Findings that require separate treatment:
[...]

Do not group findings merely because they are
in the same file.
```

---

# Part Q — Create the Lab 11 Report

Create:

```text
SONARQUBE_ROOT_CAUSE_ANALYSIS.md
```

Use Cursor/Copilot:

```text
Create SONARQUBE_ROOT_CAUSE_ANALYSIS.md using ONLY
the actual SonarQube findings investigated in Lab 11.

Use this structure:

# Lab 11 — SonarQube Root-Cause Analysis

## 1. Project

Lab02Api

## 2. Environment

SonarQube for IDE:
Installed

Connected Mode:
Yes / No

SonarQube Server/Cloud:
[Actual setup or Not Used]

## 3. Analysis Method

Sonar finding
→ rule explanation
→ source inspection
→ AI hypothesis
→ Five Whys
→ challenge AI
→ human validation

## 4. Finding 1

### Sonar Evidence

File:
Line:
Rule:
Severity:
Software Quality:
Message:

### Affected Code

[...]

### What Sonar Detected

[...]

### Root-Cause Hypotheses

1.
2.
3.

### Five Whys

[...]

### Most Likely Root Cause

[...]

### Classification

TRUE POSITIVE /
FALSE POSITIVE /
NEEDS MORE CONTEXT

### Confidence

HIGH / MEDIUM / LOW

### Impact

Security:
Reliability:
Maintainability:

### Priority

P0 / P1 / P2 / P3

## 5. Finding 2

Same structure.

## 6. Finding 3

Same structure if available.

## 7. Root-Cause Clusters

Cluster
Related Findings
Common Root Cause
Evidence

## 8. Prioritized Remediation Backlog

Finding
Root Cause
Priority
Recommended Investigation/Remediation Direction
Validation Needed

## 9. Remaining Unknowns

[...]

## 10. Conclusion

Explain what was learned.

Do NOT claim the issues have been fixed.
Do NOT invent findings.
```

---

# Part R — Main AI Prompt for Lab 11

For classroom use, I would give students **one reusable master prompt**:

```text
You are a Senior .NET Developer and
SonarQube Root-Cause Analyst.

I am a beginner.

Do NOT modify source code.

SONARQUBE FINDING

File:
[FILE]

Line:
[LINE]

Rule:
[RULE]

Severity:
[SEVERITY]

Software Quality:
[QUALITY]

Message:
[MESSAGE]

Sonar Rule Explanation:
[EXPLANATION]

Affected Code:
[CODE]

TASK

1. Explain the finding in beginner-friendly language.

2. Separate:
   - Symptom
   - Immediate cause
   - Possible root cause

3. Generate three root-cause hypotheses.

4. For each hypothesis give:
   - supporting evidence
   - contradicting evidence
   - confidence
   - missing information

5. Perform a Five Whys analysis.
   Stop if further answers require speculation.

6. Challenge your own analysis.

7. Determine the most likely root cause.

8. Classify:
   - TRUE POSITIVE
   - FALSE POSITIVE
   - NEEDS MORE CONTEXT

9. Assess impact:
   - Security
   - Reliability
   - Maintainability

10. Assign priority:
    P0 / P1 / P2 / P3

11. Explain what evidence would be needed
    before remediation.

Do NOT:

- change code
- suppress Sonar rules
- invent architecture
- invent requirements
- invent vulnerabilities
- claim the issue is fixed
```

---

# Lab 11 Completion Checklist

Students should finish Lab 11 only when they can tick:

* [ ] VS Code opens `Lab02Api`.
* [ ] SonarQube for IDE is enabled.
* [ ] Problems panel opens.
* [ ] At least one actual Sonar issue selected.
* [ ] File recorded.
* [ ] Line recorded.
* [ ] Sonar rule recorded.
* [ ] Severity recorded.
* [ ] Software quality recorded if available.
* [ ] Sonar issue details reviewed.
* [ ] Affected source code reviewed.
* [ ] AI explains the issue.
* [ ] Symptom distinguished from root cause.
* [ ] Three root-cause hypotheses generated.
* [ ] Evidence checked against source.
* [ ] Five Whys completed.
* [ ] AI challenged its own conclusion.
* [ ] Finding classified as True Positive / False Positive / Needs More Context.
* [ ] Impact assessed.
* [ ] Priority assigned.
* [ ] Two or three findings analyzed if available.
* [ ] Related issues grouped where justified.
* [ ] Root-cause backlog created.
* [ ] `SONARQUBE_ROOT_CAUSE_ANALYSIS.md` created.
* [ ] **No remediation performed yet.**

# The easiest diagram to remember

```text
                     LAB 10
                        │
                        ▼
                SONARQUBE FINDS
                    A PROBLEM
                        │
                        ▼
                     LAB 11
                        │
                        ▼
                  OPEN ISSUE
                        │
                        ▼
                  READ RULE
                        │
                        ▼
                 INSPECT CODE
                        │
                        ▼
                  ASK AI WHY
                        │
                        ▼
            ROOT-CAUSE HYPOTHESES
                        │
                        ▼
                   FIVE WHYS
                        │
                        ▼
                 CHALLENGE AI
                        │
                        ▼
                HUMAN VALIDATION
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
           TRUE       FALSE      NEEDS
         POSITIVE    POSITIVE    CONTEXT
             │
             ▼
       IMPACT + PRIORITY
             │
             ▼
      REMEDIATION BACKLOG
             │
             ▼
                   LAB 12
                        │
                        ▼
                      FIX
```

So the one sentence students should remember is:

> **Lab 10: SonarQube tells me WHAT may be wrong. Lab 11: AI helps me investigate WHY, but I must verify the explanation against the actual code. Lab 12: only then do I fix it.**

[1]: https://docs.sonarsource.com/sonarqube-for-vs-code/getting-started/running-an-analysis?utm_source=chatgpt.com "Running an analysis | VS Code | Sonar Documentation"
[2]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/introduction?utm_source=chatgpt.com "Introduction | SonarQube Community Build | Sonar Documentation"
[3]: https://docs.sonarsource.com/sonarqube-for-vs-code/using/rules?utm_source=chatgpt.com "Rules and languages | VS Code | Sonar Documentation"
[4]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/managing?utm_source=chatgpt.com "Editing issues | SonarQube Community Build | Sonar Documentation"
[5]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/solution-overview?utm_source=chatgpt.com "Issue management solution | SonarQube Community Build | Sonar Documentation"
[6]: https://docs.sonarsource.com/sonarqube-for-vs-code/using/investigating-issues?utm_source=chatgpt.com "Investigating issues | VS Code | Sonar Documentation"
[7]: https://docs.sonarsource.com/sonarqube-for-vs-code/using/fixing-issues?utm_source=chatgpt.com "Fixing issues | VS Code | Sonar Documentation"
