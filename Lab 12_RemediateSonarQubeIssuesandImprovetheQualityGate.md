**Lab 12 is where you actually change the code.** Lab 10 created the SonarQube baseline, and Lab 11 investigated *why* selected findings occurred. Lab 12 now takes the confirmed **True Positive** findings, fixes their root causes, rescans the project, and compares the **before vs. after Quality Gate and metrics**.

This matches the source course sequence of interpreting SonarQube reports, root-cause analysis, AI explanation, actionable remediation recommendations, and tracking quality improvement initiatives. 

# Lab 12 — Remediate SonarQube Issues and Improve the Quality Gate

## Objectives

By the end of this lab, you should be able to:

1. Select confirmed SonarQube issues from Lab 11.
2. Ask AI for a **minimal remediation plan**.
3. Fix the root cause rather than suppressing the Sonar rule.
4. Build and functionally test the application after each change.
5. Run SonarQube again.
6. Verify whether issues become **Fixed** after subsequent analysis.
7. Compare before/after Security, Reliability, Maintainability, duplication and Quality Gate results.
8. Understand why the Quality Gate can still fail even after several issues are fixed.
9. Create `SONARQUBE_REMEDIATION_REPORT.md`.

SonarQube determines on a subsequent analysis whether a previously detected issue is no longer present; if so, it is marked **Fixed**. ([SonarSource Documentation][1])

---

# Part 1 — Understand the workflow

Lab 12 is:

```text
LAB 10
SonarQube Scan
      ↓
Quality Baseline

      ↓

LAB 11
Root-Cause Analysis
      ↓
True Positive?
False Positive?
Needs Context?

      ↓

LAB 12
Confirmed True Positive
      ↓
AI Remediation Plan
      ↓
Human Review
      ↓
Change Code
      ↓
dotnet build
      ↓
Functional Test
      ↓
SonarQube Rescan
      ↓
Compare Before vs After
```

The key word is:

**PROVE.**

We don't simply ask Cursor:

```text
Fix all SonarQube issues.
```

and assume quality improved.

---

# Part 2 — Open your Lab 11 report

In Cursor, open:

```text
SONARQUBE_ROOT_CAUSE_ANALYSIS.md
```

Suppose your Lab 11 report contains:

```text
Finding 1
Status: TRUE POSITIVE
Priority: High

Finding 2
Status: TRUE POSITIVE
Priority: Medium

Finding 3
Status: NEEDS MORE CONTEXT
Priority: Medium
```

For Lab 12:

```text
Finding 1 → Candidate for remediation

Finding 2 → Candidate for remediation

Finding 3 → Do NOT blindly modify
```

Only remediate findings you have reasonably confirmed.

---

# Part 3 — Check SonarQube before changing anything

Start SonarQube if necessary:

```powershell
docker start sonarqube
```

Check:

```powershell
docker ps
```

Open:

```text
http://localhost:9000
```

Go to:

**Projects → Lab02Api**

Record your current Quality Gate:

```text
Quality Gate:
PASS / FAIL
```

Then record the metrics you established in Lab 10.

For example:

| Metric            |        Before |
| ----------------- | ------------: |
| Quality Gate      | Actual result |
| Security          |        Actual |
| Reliability       |        Actual |
| Maintainability   |        Actual |
| Issues            |        Actual |
| Security Hotspots |        Actual |
| Duplicated Lines  |        Actual |
| Coverage          |        Actual |

Use your actual dashboard values.

---

# Part 4 — Don't misunderstand Quality Gate

A **Quality Gate** is a set of conditions that determines whether analyzed code satisfies the project's quality policy. If one of its failing conditions is triggered, the Quality Gate fails. ([SonarSource Documentation][2])

For example:

```text
Quality Gate
      │
      ├── New issues acceptable?
      ├── Security okay?
      ├── Reliability okay?
      ├── Maintainability okay?
      ├── Coverage sufficient?
      └── Duplication acceptable?
```

Then:

```text
ALL required conditions pass
             ↓
           PASS

One required condition fails
             ↓
           FAIL
```

Therefore:

> Fixing three code smells does **not necessarily mean the Quality Gate will immediately pass**.

That's okay.

---

# Part 5 — Current Sonar way behavior matters

Sonar's current built-in **Sonar way** Quality Gate focuses primarily on **new code**. Its documented conditions include no new issues, all new Security Hotspots reviewed, new-code coverage of at least 80%, and new-code duplication of no more than 3%. ([SonarSource Documentation][2])

So your project could become:

```text
Issues reduced
Maintainability improved
Security improved

BUT

Coverage condition still fails
        ↓
Quality Gate still FAIL
```

That is not a failed lab.

It means you've identified the **remaining Quality Gate condition**.

---

# Part 6 — Choose ONE confirmed issue

Go:

**SonarQube → Lab02Api → Issues**

Pick an issue from Lab 11 that you classified:

```text
TRUE POSITIVE
```

For example, perhaps SonarQube identified something in:

```text
CustomerReportService.cs
```

or:

```text
SecurityDemoService.cs
```

Use your **actual Sonar issue**, not my example.

---

# Part 7 — Give AI the root cause, not just the Sonar message

Open Cursor Chat.

Use:

```text
You are assisting with remediation of a confirmed
SonarQube issue.

Do NOT modify the code yet.

SONARQUBE FINDING

File:
[actual file]

Rule:
[actual rule]

Severity:
[actual severity]

Message:
[actual message]

Affected code:
[paste code]

LAB 11 ROOT-CAUSE ANALYSIS

Root cause:
[paste confirmed root cause]

Classification:
TRUE POSITIVE

Propose three remediation options.

For each explain:

1. How it addresses the root cause.
2. Expected improvement.
3. Code affected.
4. Regression risk.
5. Testing required.
6. Whether it merely silences SonarQube.

Recommend the smallest safe remediation.

Do not implement anything yet.
```

This is much better than:

```text
Fix this Sonar issue.
```

---

# Part 8 — Check whether AI is only trying to silence Sonar

Ask:

```text
Challenge your recommended remediation.

Could this solution make the SonarQube issue disappear
without actually improving the underlying code?

Check for:

- rule suppression
- disabling analysis
- excluding the file
- unnecessary comments
- cosmetic changes
- hiding the issue instead of fixing it

Recommend root-cause remediation only.

Do not modify code.
```

You generally want:

```text
BAD

Problem
  ↓
Suppress warning
  ↓
Sonar issue disappears
  ↓
Bad design remains
```

versus:

```text
GOOD

Problem
  ↓
Understand root cause
  ↓
Improve implementation
  ↓
Rescan
  ↓
Finding genuinely disappears
```

---

# Part 9 — Example: maintainability issue

Suppose the confirmed problem is deep nesting.

Something resembling:

```csharp
if (customerName != null)
{
    if (isActive)
    {
        if (customerType == "VIP")
        {
            if (amount > 10000)
            {
                // ...
            }
        }
    }
}
```

The root cause might be:

```text
Multiple responsibilities
inside one large method
        ↓
Deep conditional nesting
        ↓
Poor maintainability
```

Don't fix it by suppressing the complexity rule.

Instead a reasonable remediation might involve:

```csharp
if (string.IsNullOrWhiteSpace(customerName))
{
    return "ERROR: Customer name required";
}

if (!isActive)
{
    return CreateInactiveReport(customerName);
}
```

and extracting smaller operations.

This is:

```text
Root Cause
Multiple responsibilities

       ↓

Remediation
Guard clauses
+
smaller focused methods
```

---

# Part 10 — Ask AI to implement ONE issue

After reviewing the proposal:

```text
Implement ONLY the approved remediation for this
SonarQube issue.

Requirements:

1. Preserve existing business behavior.
2. Do not modify unrelated files.
3. Do not suppress SonarQube rules.
4. Do not disable analysis.
5. Do not introduce unnecessary packages.
6. Keep the change minimal.
7. Show me exactly what changed.
```

Now allow Cursor to make the change.

---

# Part 11 — Build immediately

Run:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

If the build fails:

```text
STOP.
```

Don't continue fixing more Sonar issues.

Ask Cursor:

```text
The project failed to build after the SonarQube
remediation.

Here is the build error:

[paste error]

Identify the root cause.

Recommend the smallest correction.

Do not rewrite unrelated code.
```

---

# Part 12 — Test the affected functionality

Suppose you changed:

```text
CustomerReportService.cs
```

Run:

```powershell
dotnet run
```

Retest:

```text
/customer-report
```

using the same inputs you recorded in Lab 5.

For example:

```text
VIP + RM20,000 + Active
```

Before:

```text
High Value
```

After:

```text
High Value
```

Expected:

```text
BEFORE = AFTER
```

The code structure can change.

The expected business behavior should not accidentally change.

---

# Part 13 — Stop and create a checkpoint

After one successful remediation:

```powershell
Ctrl + C
```

If using Git:

```powershell
git add .
git commit -m "Remediate SonarQube finding 1"
```

This is useful because if Issue #2 creates problems, you can distinguish it from Issue #1.

---

# Part 14 — Remediate the second confirmed issue

Repeat:

```text
Sonar Finding
      ↓
Lab 11 Root Cause
      ↓
AI remediation options
      ↓
Challenge AI
      ↓
Human chooses
      ↓
Implement one change
      ↓
dotnet build
      ↓
Functional testing
```

Don't fix 20 issues in a single uncontrolled AI change.

---

# Part 15 — What about False Positives?

Suppose Lab 11 concluded:

```text
FALSE POSITIVE
```

You should normally **not rewrite correct code just to satisfy the scanner**.

SonarQube allows authorized users to mark an issue as **False positive** if the analysis is mistaken. It also supports **Accepted** for issues intentionally deferred. Both affect how issues are treated in quality reports and ratings. ([SonarSource Documentation][3])

For the lab, document:

```text
Finding:
[...]

Classification:
False Positive

Action:
No code modification

Reason:
[Lab 11 evidence]
```

Do not mark something false positive merely because it's inconvenient to fix.

---

# Part 16 — What about "Needs More Context"?

If Lab 11 said:

```text
NEEDS MORE CONTEXT
```

do not pretend the issue is solved.

Record:

```text
Status:
Investigation required

Missing information:
- runtime configuration
- database context
- authentication requirements
- business rules
etc.
```

That is a valid engineering result.

---

# Part 17 — Now perform the SonarQube RESCAN

Once your approved remediations compile and work, run the same .NET analysis workflow from Lab 10.

Sonar's current .NET scanner flow is:

```text
BEGIN
  ↓
BUILD
  ↓
END
```

with the build occurring between scanner begin and end. ([SonarSource Documentation][4])

Make sure your token is available:

```powershell
$env:SONAR_TOKEN
```

Then:

```powershell
dotnet sonarscanner begin `
  /k:"lab02api" `
  /d:sonar.host.url="http://localhost:9000" `
  /d:sonar.token="$env:SONAR_TOKEN"
```

Then:

```powershell
dotnet build --no-incremental
```

Finally:

```powershell
dotnet sonarscanner end `
  /d:sonar.token="$env:SONAR_TOKEN"
```

These are the documented global-tool invocation steps for SonarScanner for .NET. ([SonarSource Documentation][4])

---

# Part 18 — Return to SonarQube

Open:

```text
http://localhost:9000
```

Then:

**Projects → Lab02Api**

You now have:

```text
BEFORE SCAN
     vs
AFTER SCAN
```

Check your selected issue first.

---

# Part 19 — Did SonarQube recognize the fix?

Go:

**Issues**

Search for the original finding.

If the code no longer violates that rule, a subsequent SonarQube analysis can identify the previously open finding as **Fixed**. ([SonarSource Documentation][1])

Therefore:

```text
Before
OPEN

After remediation + rescan
FIXED
```

That's much stronger evidence than:

> "Cursor says it fixed it."

---

# Part 20 — Compare your issue count

Create:

| Metric            | Before |  After | Improvement |
| ----------------- | -----: | -----: | ----------: |
| Total Issues      | actual | actual |      actual |
| Security          | actual | actual |             |
| Reliability       | actual | actual |             |
| Maintainability   | actual | actual |             |
| Security Hotspots | actual | actual |             |
| Duplicated Lines  | actual | actual |             |
| Quality Gate      | actual | actual |             |

Don't invent numbers.

Use your actual dashboard.

---

# Part 21 — Example interpretation

Suppose yours changes like this:

```text
BEFORE

Maintainability Issues: 8
Security Issues:        1
Reliability Issues:     1

AFTER

Maintainability Issues: 5
Security Issues:        1
Reliability Issues:     0
```

Then you can conclude:

```text
Maintainability
8 → 5
Improved

Reliability
1 → 0
Improved

Security
1 → 1
Unchanged
```

Not:

```text
Everything fixed!
```

Your report should show both improvement **and remaining risk**.

---

# Part 22 — Check Quality Gate again

Now compare:

```text
BEFORE

Quality Gate:
FAILED
```

with:

```text
AFTER

Quality Gate:
?
```

There are two legitimate outcomes.

### Outcome A — PASS

Excellent:

```text
Issues remediated
        ↓
Gate conditions satisfied
        ↓
PASS
```

### Outcome B — Still FAIL

Also useful.

Click the failed Quality Gate condition.

Perhaps:

```text
Coverage insufficient
```

or:

```text
New issue remains
```

or:

```text
Security Hotspot not reviewed
```

A Quality Gate condition is tied to a particular metric and threshold, so you should identify the exact failing condition rather than guessing. ([SonarSource Documentation][2])

---

# Part 23 — Don't weaken the Quality Gate just to get green

Suppose:

```text
Coverage = 30%

Quality Gate requires = 80%
```

The wrong lesson is:

```text
Change gate requirement
80% → 20%

PASS!
```

That didn't improve your software.

You merely weakened the policy.

For the lab:

```text
Quality Gate still FAIL

Reason:
Coverage requirement not satisfied

Recommended future action:
Add meaningful automated tests
```

This naturally connects to the upcoming .NET unit-testing labs.

---

# Part 24 — Ask AI to interpret the Quality Gate

Use:

```text
Review the before and after SonarQube results.

BEFORE:
[paste actual metrics]

AFTER:
[paste actual metrics]

Quality Gate Before:
[actual]

Quality Gate After:
[actual]

If the Quality Gate still fails:

1. Identify the exact failing condition.
2. Explain what it measures.
3. Explain whether our Lab 12 remediation affected it.
4. Recommend the appropriate next action.
5. Do NOT recommend weakening the Quality Gate merely
   to obtain a PASS.
```

This is a good AI SDLC use case.

---

# Part 25 — Ask AI for quantified improvement

Now use:

```text
Calculate and explain the improvement between the
SonarQube baseline and the new scan.

Use only actual values.

For each metric show:

Before
After
Absolute Change
Percentage Change where meaningful
Interpretation

Do not invent missing metrics.
```

For example:

```text
Issues

Before = 10
After  = 6

Reduction = 4
Reduction % = 40%
```

That becomes measurable quality improvement.

---

# Part 26 — Relate this back to Lab 4

Remember Lab 4?

You manually created:

```text
CODE_QUALITY_BASELINE.md
```

Now you can compare three stages:

```text
LAB 4
AI/manual baseline

        ↓

LAB 10
SonarQube automated baseline

        ↓

LAB 12
SonarQube after remediation
```

That is a strong course narrative.

---

# Part 27 — Create the final report

Create:

```text
SONARQUBE_REMEDIATION_REPORT.md
```

Ask Cursor:

```text
Create SONARQUBE_REMEDIATION_REPORT.md using ONLY
actual results from Labs 10-12.

Use this structure:

# Lab 12 — SonarQube Remediation and Quality Improvement

## 1. Baseline

Record original:
- Quality Gate
- Security
- Reliability
- Maintainability
- Issue count
- Duplication
- Coverage

## 2. Findings Selected

For each:
- File
- Rule
- Severity
- Lab 11 classification
- Root cause

## 3. Remediation Plan

For every confirmed issue:
- Proposed remediation
- Why selected
- Regression risk

## 4. Changes Implemented

Document actual source-code changes.

## 5. Build Validation

Record dotnet build result.

## 6. Functional Validation

Record affected API/function testing.

## 7. SonarQube Rescan

Record whether each issue became:
- Fixed
- Still Open
- Accepted
- False Positive
- Needs More Work

## 8. Before vs After Metrics

Table:

Metric
Before
After
Change

## 9. Quality Gate

Before:
[...]

After:
[...]

If still failed:
Document exact failing condition.

## 10. Remaining Issues

Prioritized list.

## 11. Conclusion

State measurable improvement without claiming
unverified results.

Do not invent SonarQube results.
```

---

# Part 28 — Your final Lab 12 project

Your folder should now contain roughly:

```text
Lab02Api
│
├── Program.cs
├── OrderService.cs
├── CustomerReportService.cs
├── PerformanceService.cs
├── SecurityDemoService.cs
│
├── CODE_QUALITY_BASELINE.md
├── SONARQUBE_BASELINE.md
├── SONARQUBE_ROOT_CAUSE_ANALYSIS.md
├── SONARQUBE_REMEDIATION_REPORT.md
│
└── Lab02Api.csproj
```

---

# Lab 12 Completion Checklist

* [ ] SonarQube baseline from Lab 10 recorded.
* [ ] Lab 11 RCA report reviewed.
* [ ] Only confirmed True Positive findings selected.
* [ ] AI proposes remediation options before editing.
* [ ] AI recommendation challenged.
* [ ] Root-cause remediation chosen.
* [ ] No SonarQube rules suppressed merely to get green.
* [ ] Issue #1 remediated.
* [ ] `dotnet build` succeeds.
* [ ] Affected functionality retested.
* [ ] Issue #2 remediated if applicable.
* [ ] Application rebuilt and retested.
* [ ] SonarScanner `begin → build → end` rerun.
* [ ] SonarQube Issues checked after rescan.
* [ ] Fixed issues confirmed.
* [ ] Before/after issue counts recorded.
* [ ] Before/after Security recorded.
* [ ] Before/after Reliability recorded.
* [ ] Before/after Maintainability recorded.
* [ ] Before/after duplication recorded.
* [ ] Quality Gate checked again.
* [ ] Exact remaining failed condition documented if Gate remains failed.
* [ ] `SONARQUBE_REMEDIATION_REPORT.md` created.

The easiest way to remember **Labs 10–12** is:

```text
          LAB 10
     SonarQube SCANS
            │
            ▼
     "WHAT is wrong?"
            │
            ▼
          LAB 11
      AI investigates
            │
            ▼
      "WHY is it wrong?"
            │
            ▼
          LAB 12
      AI + Developer
           FIX
            │
            ▼
          BUILD
            │
            ▼
           TEST
            │
            ▼
         RESCAN
            │
            ▼
    "DID quality improve?"
            │
       ┌────┴────┐
       ▼         ▼
      YES        NO
       │          │
       ▼          ▼
    Record     Investigate
 improvement   remaining gate
```

**The most important result of Lab 12 is not necessarily a green Quality Gate.** The important result is that you can demonstrate with evidence: **“This SonarQube issue existed → we found its root cause → changed the code → verified the application still works → rescanned → and the issue/metric measurably improved.”** That is a proper AI-assisted SDLC remediation workflow.

[1]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/solution-overview?utm_source=chatgpt.com "Issue management solution | SonarQube Community Build | Sonar Documentation"
[2]: https://docs.sonarsource.com/sonarqube-community-build/quality-standards-administration/managing-quality-gates/introduction-to-quality-gates?utm_source=chatgpt.com "Understanding quality gates | SonarQube Community Build | Sonar Documentation"
[3]: https://docs.sonarsource.com/sonarqube-community-build/user-guide/issues/managing?utm_source=chatgpt.com "Editing issues | SonarQube Community Build | Sonar Documentation"
[4]: https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/scanners/dotnet/using?utm_source=chatgpt.com "Using the scanner | SonarQube Community Build | Sonar Documentation"
