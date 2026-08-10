**Lab 11 does NOT mean “fix all SonarQube problems.”** Lab 10 gave you the SonarQube dashboard and quality baseline. Lab 11 teaches you to take **one SonarQube finding at a time** and use AI to answer:

> **Why did SonarQube report this? What is the underlying cause? Is it really a problem?**

This follows your original course directly: **Interpreting SonarQube Reports → Root Cause Analysis of Findings → Using AI to Explain SonarQube Results → Generating Actionable Remediation Recommendations.** 

# Lab 11 — Use AI for Root-Cause Analysis of SonarQube Findings

## Objectives

By the end of this lab, you will be able to:

* Select a real SonarQube issue.
* Understand the rule that generated it.
* Find the exact affected C# code.
* Distinguish **symptom** from **root cause**.
* Give SonarQube evidence to Cursor/Copilot.
* Use AI to explain the finding.
* Ask AI for several possible root causes.
* Challenge the AI's explanation.
* Decide whether the finding is a **True Positive, False Positive, or Needs More Context**.
* Prioritize the issue.
* Create a root-cause analysis report.
* **Not automatically fix the code yet.**

---

# Part 1 — Understand the difference

This is the most important concept.

Suppose SonarQube reports:

```text
Method is too complex.
```

That is the:

```text
SYMPTOM
```

The root cause might actually be:

```text
Too many business rules
       ↓
all placed inside
one method
       ↓
nested if statements
       ↓
high complexity
```

So:

```text
SONARQUBE FINDING
       ↓
"What happened?"

ROOT CAUSE ANALYSIS
       ↓
"Why did it happen?"
```

---

# Part 2 — Lab 10 vs Lab 11

Think of them like this:

```text
LAB 10
────────────────────────

Scan entire project
        ↓
SonarQube Dashboard
        ↓
7 issues
3 code smells
1 security issue
etc.
        ↓
Record baseline


LAB 11
────────────────────────

Take ONE issue
        ↓
Understand rule
        ↓
Inspect affected code
        ↓
Ask AI WHY
        ↓
Find root cause
        ↓
Validate AI explanation
```

**Lab 10 = Find.**

**Lab 11 = Understand.**

---

# Part 3 — Make sure SonarQube is running

Open PowerShell:

```powershell
docker ps
```

You should see:

```text
sonarqube
```

If you previously stopped it:

```powershell
docker start sonarqube
```

Open:

```text
http://localhost:9000
```

Login.

---

# Part 4 — Open your project

In SonarQube:

**Projects → Lab02Api**

Then click:

**Issues**

SonarQube generates an issue when source code breaks one of the active coding rules. The issue records the affected file, rule, status, impacted software quality/type, line information, and other context. ([SonarSource Documentation][1])

You may see something like:

```text
Lab02Api

Issues

□ Program.cs
□ SecurityDemoService.cs
□ CustomerReportService.cs
□ PerformanceService.cs
```

Your actual results will be different.

---

# Part 5 — Pick ONE issue

Do not start with all 20 issues.

Choose one that you can understand.

Prefer something under:

```text
Maintainability
```

or, if your SonarQube is using Standard Experience:

```text
Code Smell
```

SonarQube Community Build can display findings either in **MQR mode**, using Security/Reliability/Maintainability, or in **Standard Experience**, using Vulnerabilities/Bugs/Code Smells. ([SonarSource Documentation][2])

For your first exercise:

```text
ONE issue only.
```

---

# Part 6 — Open the issue

Click the issue.

SonarQube should show information such as:

```text
Rule

Severity

File

Line

Issue description

Affected code
```

Sonar's issue-detail view includes **Where is the issue?**, which identifies the relevant code location. Some issues also show secondary locations or execution flows to help trace how the problem develops through the code. ([SonarSource Documentation][3])

---

# Part 7 — Read "Why is this an issue?"

Inside the SonarQube issue, find:

**Why is this an issue?**

Click it.

SonarQube provides this explanation specifically to help you understand why the underlying rule exists. You can also open the actual rule definition from the issue. ([SonarSource Documentation][3])

Do **not** go straight to:

```text
"How do I fix it?"
```

First understand:

```text
WHY
```

---

# Part 8 — Record the evidence

Create a new file in Cursor:

```text
SONAR_ISSUE_01.md
```

Record:

```markdown
# SonarQube Issue 01

## File

[actual filename]

## Line

[actual line]

## Rule

[actual SonarQube rule]

## Severity

[actual severity]

## SonarQube Message

[actual issue message]

## Affected Code

[paste affected code]

## SonarQube Explanation

Summarize "Why is this an issue?"
```

Do not invent anything.

Use what you see in SonarQube.

---

# Part 9 — Give the issue to AI

Open Cursor Chat.

Use this prompt:

```text
I am performing root-cause analysis of an actual
SonarQube finding.

Do NOT modify my code.

SonarQube reported:

Rule:
[paste actual rule]

Severity:
[paste actual severity]

Message:
[paste actual message]

Affected file:
[paste filename]

Affected code:
[paste affected code]

First explain in beginner-friendly language:

1. What SonarQube detected.
2. What this rule means.
3. Why SonarQube considers this a problem.
4. Which software quality is affected:
   Security, Reliability or Maintainability.
5. What could happen if we ignore it.

Do not suggest a fix yet.
```

That final instruction is important:

```text
DO NOT SUGGEST A FIX YET.
```

Lab 11 is about diagnosis.

---

# Part 10 — Ask AI for possible root causes

Now prompt:

```text
Do NOT modify the code.

For this SonarQube finding, provide three possible
root-cause hypotheses.

For each hypothesis give:

1. Possible root cause
2. Evidence in the code
3. Why it could produce the SonarQube finding
4. Confidence:
   High / Medium / Low
5. Additional evidence needed to confirm it

Rank the hypotheses from most likely to least likely.
```

Now AI has to **reason from evidence** instead of immediately rewriting your code.

---

# Part 11 — Example: symptom vs root cause

Suppose, for illustration, SonarQube complains about complexity.

AI might return:

```text
SONARQUBE FINDING

High cognitive complexity
        ↓

Possible Root Cause 1

Deep nested if statements
Confidence: HIGH


Possible Root Cause 2

Multiple business rules in one method
Confidence: HIGH


Possible Root Cause 3

Method has too many responsibilities
Confidence: MEDIUM
```

Then your RCA becomes:

```text
Symptom:
High complexity

        ↓

Immediate Cause:
Nested control flow

        ↓

Deeper Cause:
Several responsibilities are
implemented inside one method
```

That's root-cause analysis.

---

# Part 12 — Ask AI to examine more context

One line often isn't enough.

Suppose SonarQube reports something at:

```text
CustomerReportService.cs
Line 25
```

Don't give AI only line 25.

Ask Cursor:

```text
Review the complete method containing this SonarQube
finding.

Do NOT modify the source.

Determine whether the issue is caused by:

- this single line
- surrounding control flow
- duplicated logic
- method design
- another method
- a broader architectural decision

Use exact code evidence.

Revise your root-cause hypotheses if necessary.
```

This is important because:

```text
Issue LOCATION
     ≠
necessarily
Root Cause LOCATION
```

---

# Part 13 — Use SonarQube's secondary locations

For some findings, SonarQube may display:

```text
Primary Location
       ↓
Secondary Location
       ↓
Secondary Location
```

or an execution flow:

```text
Source
  ↓
Step
  ↓
Step
  ↓
Sink
```

SonarQube explicitly supports secondary locations and execution flows for issues where the problem originates elsewhere in the code. ([SonarSource Documentation][1])

If these appear, give them to AI too:

```text
SonarQube shows the following execution flow:

Step 1:
[...]

Step 2:
[...]

Step 3:
[...]

Explain how these locations are related and identify
the most likely root cause.

Do not change the code.
```

---

# Part 14 — Ask AI for a Five Whys analysis

Now use a classic root-cause technique.

Prompt:

```text
Perform a Five Whys analysis for this SonarQube finding.

Start with the SonarQube issue and repeatedly ask
"Why did this happen?"

Stop when you reach a root cause that a development team
could realistically act on.

Do not invent organizational causes without evidence.

Do not modify code.
```

You may get:

```text
Problem:
Method has excessive complexity.

WHY 1
Why?
Because there are many nested conditions.

        ↓

WHY 2
Why are there many nested conditions?
Because several business decisions are handled
inside one method.

        ↓

WHY 3
Why are several decisions handled there?
Because validation, classification and formatting
are mixed together.

        ↓

ROOT CAUSE

Method has too many responsibilities.
```

Now you're doing genuine RCA rather than:

```text
Sonar says bad.
AI says fix.
```

---

# Part 15 — Challenge the AI

This is essential.

Ask:

```text
Now challenge your own root-cause analysis.

Do NOT assume your first explanation is correct.

For each proposed root cause identify:

1. Evidence supporting it.
2. Evidence against it.
3. Alternative explanations.
4. Information you are missing.
5. Confidence after reconsideration.

Then identify the SINGLE most likely root cause.
```

This reduces blind acceptance of AI reasoning.

---

# Part 16 — Determine whether SonarQube is correct

Now ask:

```text
Based on:

- the SonarQube rule
- affected code
- surrounding method
- SonarQube explanation
- our root-cause analysis

classify this finding as:

TRUE POSITIVE
FALSE POSITIVE
NEEDS MORE CONTEXT

Explain the decision.

Do NOT modify code.
```

This is important because SonarQube findings still require developer judgment.

SonarQube allows authorized users to mark an issue **Accepted** when it will be fixed later, or **False positive** when the analysis is considered mistaken. ([SonarSource Documentation][4])

---

# Part 17 — What do those classifications mean?

### True Positive

Sonar identified a real issue.

```text
Sonar says:
Problem here.

Human reviews:
Yes, genuine problem.

→ TRUE POSITIVE
```

### False Positive

Sonar flags something but the rule doesn't correctly apply to this situation.

```text
Sonar says:
Problem here.

Human investigates:
Not actually a problem.

→ FALSE POSITIVE
```

### Needs More Context

You can't responsibly decide yet.

```text
Sonar says:
Problem here.

Human:
Need database/config/runtime/business-rule
information.

→ NEEDS MORE CONTEXT
```

This is often a better answer than guessing.

---

# Part 18 — Determine impact

Now ask:

```text
Assuming this is a true positive, perform an impact
assessment.

Evaluate:

1. Security impact
2. Reliability impact
3. Maintainability impact
4. User impact
5. Developer impact
6. Operational impact

Rate each:
None / Low / Medium / High

Explain using evidence from the project.
```

SonarQube's current MQR model organizes issues around **Security, Reliability and Maintainability**, with severity indicating the magnitude of impact. ([SonarSource Documentation][5])

---

# Part 19 — Do NOT let AI override Sonar severity blindly

Suppose Sonar says:

```text
Medium
```

and AI says:

```text
Critical
```

Ask:

```text
SonarQube assigned this severity:

[actual severity]

You assessed the impact differently.

Explain the difference.

Do not override the SonarQube severity automatically.

Separate:

1. SonarQube rule severity
2. Project-specific business impact
```

This teaches an important distinction:

```text
Sonar Rule Severity
        ≠
Your business context
```

For example, the same maintainability issue might be minor in a disposable prototype but more important in a core payment-processing module.

---

# Part 20 — Now ask for remediation OPTIONS — but don't implement

Only after RCA is complete:

```text
The root-cause analysis is now complete.

Do NOT modify the source code.

Give me three remediation options.

For each provide:

1. Proposed change
2. Which root cause it addresses
3. Expected benefit
4. Implementation effort
5. Regression risk
6. Testing required

Recommend the smallest change that addresses the
root cause rather than merely hiding the SonarQube warning.
```

The crucial sentence is:

> **Address the root cause rather than merely hiding the warning.**

---

# Part 21 — Bad remediation example

Suppose Sonar reports:

```text
Method is too complex.
```

AI could "fix" it by suppressing the warning.

Something conceptually like:

```text
Suppress Sonar rule
```

Dashboard becomes:

```text
0 issues!
```

But the method is still:

```text
huge
nested
hard to maintain
```

That's not root-cause remediation.

You want:

```text
Issue
 ↓
Find root cause
 ↓
Correct design/code
 ↓
Rescan
 ↓
Issue genuinely disappears
```

not:

```text
Issue
 ↓
Hide warning
 ↓
Pretend quality improved
```

---

# Part 22 — Ask AI whether the proposed fix merely silences Sonar

Use this excellent prompt:

```text
Review your remediation recommendation.

Could this recommendation merely make the SonarQube
warning disappear without improving the underlying code?

Explain:

1. Whether it addresses the symptom.
2. Whether it addresses the root cause.
3. Whether it introduces technical debt.
4. Whether rule suppression is involved.

Prefer root-cause remediation.
```

---

# Part 23 — Create an RCA table

Your analysis should eventually look something like:

| Field              | Finding                      |
| ------------------ | ---------------------------- |
| Sonar issue        | Actual issue                 |
| File               | Actual file                  |
| Rule               | Actual rule                  |
| Severity           | Actual severity              |
| Symptom            | What Sonar detected          |
| Root cause         | Confirmed underlying cause   |
| Classification     | True Positive / FP / Context |
| AI confidence      | High/Medium/Low              |
| Impact             | Actual assessment            |
| Recommended action | Proposed only                |
| Status             | Not remediated yet           |

---

# Part 24 — Analyze 3 SonarQube findings

After completing Issue #1, repeat the same process for:

```text
ISSUE 01
Maintainability / Code Smell

ISSUE 02
Reliability / Bug

ISSUE 03
Security / Vulnerability
```

**only if those categories actually exist in your scan.**

If your SonarQube scan only has maintainability findings, that's fine.

Do not create fake security issues just to fill the lab.

SonarQube's rule engine analyzes source against active rules, and the findings present depend on your actual code and configured quality profile. ([SonarSource Documentation][6])

---

# Part 25 — Use one standard AI prompt for each issue

For teaching, I recommend giving students this reusable prompt:

```text
You are a Senior .NET Developer and Application
Code Quality Reviewer.

Perform root-cause analysis of this SonarQube finding.

SONARQUBE EVIDENCE

Project:
Lab02Api

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

Affected code:
[CODE]

SonarQube explanation:
[WHY IS THIS AN ISSUE]

Do NOT modify the code.

Perform:

1. Explain the issue in beginner-friendly language.
2. Identify symptom vs root cause.
3. Generate three root-cause hypotheses.
4. Rank each hypothesis by confidence.
5. Identify evidence supporting each hypothesis.
6. Identify missing context.
7. Perform Five Whys analysis.
8. Determine the most likely root cause.
9. Classify:
   - True Positive
   - False Positive
   - Needs More Context
10. Assess:
    - Security impact
    - Reliability impact
    - Maintainability impact
11. Give three remediation OPTIONS.
12. Recommend the smallest remediation that addresses
    the root cause.

Do NOT implement any remediation.
Do NOT suppress the SonarQube rule.
Do NOT invent information not supported by the code.
```

That's the **main Lab 11 AI prompt**.

---

# Part 26 — Create the final report

Create:

```text
SONARQUBE_ROOT_CAUSE_ANALYSIS.md
```

Ask Cursor:

```text
Create SONARQUBE_ROOT_CAUSE_ANALYSIS.md using ONLY the
actual SonarQube issues investigated during Lab 11.

Structure:

# Lab 11 — SonarQube Root-Cause Analysis

## 1. Project

Lab02Api

## 2. Analysis Method

Explain:
SonarQube finding
→ AI explanation
→ root-cause hypotheses
→ human validation

## 3. Finding 1

### SonarQube Evidence
- File
- Line
- Rule
- Severity
- Message

### Symptom

### Root-Cause Hypotheses

### Five Whys

### Confirmed/Most Likely Root Cause

### Classification
True Positive / False Positive / Needs Context

### Impact

### Remediation Options

### Recommended Action

## 4. Finding 2

Same structure.

## 5. Finding 3

Same structure if applicable.

## 6. Findings Summary

Table:
Issue
Rule
Severity
Root Cause
Classification
Priority

## 7. Lessons Learned

Explain why SonarQube identifies the issue while AI helps
investigate and explain its cause.

Do NOT claim remediation has been completed.
```

---

# Part 27 — Don't rescan yet

At the end of Lab 11:

```text
Code should generally remain unchanged.
```

Why?

Because this lab is:

```text
INVESTIGATION
```

not yet:

```text
REMEDIATION
```

Your flow should be:

```text
              LAB 10
           SONARQUBE SCAN
                 │
                 ▼
             FINDINGS
                 │
                 ▼
              LAB 11
                 │
         Select One Finding
                 │
                 ▼
       Read Sonar Rule/Context
                 │
                 ▼
         Give Evidence to AI
                 │
                 ▼
        AI Explains Finding
                 │
                 ▼
      Root-Cause Hypotheses
                 │
                 ▼
           Five Whys
                 │
                 ▼
         Challenge the AI
                 │
                 ▼
         Human Validation
                 │
        ┌────────┼─────────┐
        ▼        ▼         ▼
       True     False     Needs
     Positive  Positive   Context
        │
        ▼
   Impact Assessment
        │
        ▼
   Remediation OPTIONS
        │
        ▼
SONARQUBE_ROOT_CAUSE_ANALYSIS.md
```

# Lab 11 completion checklist

* [ ] SonarQube is running.
* [ ] `Lab02Api` dashboard opens.
* [ ] Actual SonarQube Issues page reviewed.
* [ ] One real issue selected.
* [ ] File and line captured.
* [ ] SonarQube rule captured.
* [ ] Severity captured.
* [ ] **Why is this an issue?** reviewed.
* [ ] Affected source code captured.
* [ ] AI explains the finding.
* [ ] AI generates three root-cause hypotheses.
* [ ] Evidence is provided for each hypothesis.
* [ ] Five Whys analysis completed.
* [ ] AI challenges its first conclusion.
* [ ] Most likely root cause identified.
* [ ] Finding classified as True Positive / False Positive / Needs Context.
* [ ] Impact assessed.
* [ ] Remediation options proposed.
* [ ] **No automatic fix performed.**
* [ ] Analysis repeated for 2–3 actual issues if available.
* [ ] `SONARQUBE_ROOT_CAUSE_ANALYSIS.md` created.

The easiest way to remember **Labs 10 and 11** is:

```text
LAB 10
SonarQube says:

"THERE IS A PROBLEM HERE."
            ↓
       DETECTION


LAB 11
You + AI ask:

"WHY IS THIS HAPPENING?"
            ↓
       ROOT CAUSE


NEXT LAB
You will ask:

"HOW SHOULD WE FIX IT
AND PROVE IT IS FIXED?"
            ↓
       REMEDIATION
```

The key principle, Truman, is **SonarQube should provide the evidence; AI should help investigate and explain it; the human should decide whether the explanation is correct.** SonarQube itself supports this review-oriented model by exposing the rule, code location, explanatory context, status, secondary locations/flows where applicable, and mechanisms for handling genuine versus false-positive findings. ([SonarSource Documentation][3])

[1]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/introduction?utm_source=chatgpt.com "Introduction | SonarQube Community Build | Sonar Documentation"
[2]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/code-metrics/changing-modes?utm_source=chatgpt.com "Changing instance modes | SonarQube Community Build | Sonar Documentation"
[3]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/reviewing?utm_source=chatgpt.com "Reviewing issues | SonarQube Community Build | Sonar Documentation"
[4]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/managing?utm_source=chatgpt.com "Editing issues | SonarQube Community Build | Sonar Documentation"
[5]: https://docs.sonarsource.com/sonarqube-community-build/quality-standards-administration/managing-rules/software-qualities?utm_source=chatgpt.com "Software qualities | SonarQube Community Build | Sonar Documentation"
[6]: https://docs.sonarsource.com/sonarqube-community-build/quality-standards-administration/managing-rules/rules?utm_source=chatgpt.com "SonarQube rules | SonarQube Community Build | Sonar Documentation"
