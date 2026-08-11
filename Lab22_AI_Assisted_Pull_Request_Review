**Lab 22 should start from zero**, because students first need to understand what a **Pull Request (PR)** is before using AI to review one.

The original Lab 22 is **“Perform an AI-Assisted Pull Request Review.”** Its intended outcome is a structured PR review where the student checks the change manually, asks AI to inspect the diff for correctness/security/maintainability/test gaps, validates the AI findings, and leaves actionable review comments. 

# Lab 22 — Perform an AI-Assisted Pull Request Review

## 1. What is a Pull Request?

A Pull Request means:

> “I changed some code in my branch. Please review it before merging it into the main branch.”

Simple flow:

```text
Developer
   ↓
Creates branch
   ↓
Changes code
   ↓
Commits
   ↓
Pushes branch
   ↓
Creates Pull Request
   ↓
Reviewer checks changes
   ↓
Approve / Request Changes
   ↓
Merge
```

So a PR is basically:

```text
"Please review my code before it enters main."
```

---

# 2. What does Lab 22 teach?

Lab 22 teaches:

```text
PR created
   ↓
Human reviews first
   ↓
AI reviews the same diff
   ↓
Compare findings
   ↓
Verify AI findings
   ↓
Classify comments
   ↓
Approve / Request Changes
```

The important part is:

> AI helps review. AI does not replace the reviewer.

---

# 3. Tools Needed

You need:

```text
Git
GitHub or Gitea
Browser
VS Code or Cursor
GitHub Copilot / AI Assistant
Existing Lab02Api repository
```

No new major software is required.

---

# 4. Understand Branches First

Suppose your main branch is:

```text
main
```

You create a separate branch:

```text
lab22-pr-review
```

Think:

```text
main
 │
 ├──── lab22-pr-review
 │
 │     changes happen here
 │
 │
 └──── PR asks:
       "Can I merge these changes back into main?"
```

---

# PART A — Create a Small Change for Review

## Step 1 — Open PowerShell

```powershell
cd C:\AI_SDLC_Labs
```

Check status:

```powershell
git status
```

Make sure you are starting from a clean working tree.

---

# Step 2 — Switch to main

```powershell
git checkout main
```

Pull latest changes if your class repository requires it:

```powershell
git pull
```

---

# Step 3 — Create a Lab 22 branch

```powershell
git checkout -b lab22-pr-review
```

Now:

```text
main
  ↓
lab22-pr-review
```

---

# Step 4 — Make a SMALL controlled change

For beginners, keep the change manageable.

For example, modify:

```text
OrderService.cs
```

or:

```text
CustomerReportService.cs
```

You can make a simple improvement such as:

```text
- clearer validation
- small refactoring
- one additional test
- one small behavior change
```

Do not create 20 files of changes.

The original Lab 22 recommends a manageable PR, ideally around **100–400 changed lines**, so reviewers can understand the change properly. 

For a beginner class, even smaller is better.

---

# PART B — Build and Test Before Creating PR

## Step 5 — Run the application build

For the .NET project:

```powershell
dotnet build .\Lab02Api\Lab02Api.csproj
```

Expected:

```text
Build succeeded.
```

---

# Step 6 — Run tests

```powershell
dotnet test .\Lab02Api.Tests\Lab02Api.Tests.csproj
```

Use actual results.

Do not create a PR if your branch already fails basic build/tests unless the failure is intentional and documented.

---

# PART C — Inspect Your Own Change

## Step 7 — View Git diff

Run:

```powershell
git diff
```

This shows:

```text
BEFORE
vs
AFTER
```

That difference is called a:

```text
DIFF
```

A PR reviewer normally reviews the diff.

---

# 8. What is a diff?

Example:

```diff
- if (customerType == "VIP")
+ if (customerType.Equals("VIP", StringComparison.OrdinalIgnoreCase))
```

The:

```text
-
```

means removed.

The:

```text
+
```

means added.

So:

```text
PR Review
   ↓
Review changed lines
   ↓
Understand impact
```

---

# PART D — Commit and Push

## Step 9 — Commit the change

```powershell
git status
```

Then:

```powershell
git add .
```

Commit:

```powershell
git commit -m "Add Lab 22 PR review change"
```

Push:

```powershell
git push -u origin lab22-pr-review
```

---

# PART E — Create the Pull Request

## Step 10 — Open GitHub or Gitea

Open your repository.

You may see:

```text
Compare & pull request
```

Click it.

Or manually choose:

```text
Pull Requests
   ↓
New Pull Request
```

---

# Step 11 — Select branches

Use:

```text
Base:
main

Compare:
lab22-pr-review
```

Meaning:

```text
Take changes from:

lab22-pr-review

and propose merging into:

main
```

---

# Step 12 — Write a good PR title

Example:

```text
Improve OrderService validation
```

Avoid:

```text
Update
```

or:

```text
Changes
```

---

# Step 13 — Write the PR description

Use something like:

```text
## What changed

- Updated OrderService validation.
- Added/updated related unit tests.

## Why

Improve validation behavior while preserving
existing pricing rules.

## Validation

- dotnet build: PASS
- dotnet test: PASS

## Risk

Low / Medium / High

## Known Issues

None / [actual]
```

Now the reviewer knows:

```text
WHAT changed

WHY

HOW tested

WHAT risk exists
```

---

# PART F — Before Using AI, Review the PR Yourself

This is extremely important.

The original Lab 22 explicitly says to review the PR manually before using AI, then compare the AI findings against the human review. 

## Step 14 — Open Files Changed

In the PR:

```text
Files changed
```

Look at every changed file.

Ask:

```text
Does this change match the PR description?

Does the logic look correct?

Could existing behavior break?

Are tests present?

Are there security concerns?

Is there unnecessary code?
```

---

# PART G — Use a Human Review Checklist

For each changed file, check:

## Correctness

```text
Does the code do what the requirement says?
```

## Maintainability

```text
Is the code understandable?

Is unnecessary complexity introduced?
```

## Security

```text
New secrets?

Unsafe input handling?

Authorization issue?
```

## Tests

```text
Does the risky behavior have tests?
```

## Compatibility

```text
Could existing callers break?
```

---

# PART H — Record Human Findings First

Create:

```text
PR_REVIEW_NOTES.md
```

Start:

```markdown
# Lab 22 — Manual PR Review Notes

## PR

Title:
[...]

Branch:
lab22-pr-review

Base:
main

## Human Review

### Finding H1

File:
[...]

Line:
[...]

Category:
Correctness / Security / Maintainability / Test Gap

Observation:
[...]

Impact:
[...]

Suggested Action:
[...]

### Finding H2

[...]
```

If you found nothing, write:

```text
No confirmed issue identified during initial manual review.
```

Do not invent problems.

---

# PART I — Now Ask AI to Review the PR

This is the AI part.

## Step 15 — Give AI the actual diff

In Cursor, GitHub Copilot, or another approved AI assistant, use:

```text
You are a Senior .NET Pull Request Reviewer.

I am learning how to review a Pull Request.

Do NOT modify any code.

PR PURPOSE

[PASTE PR DESCRIPTION]

REQUIREMENT / EXPECTED BEHAVIOR

[PASTE ACTUAL REQUIREMENT]

GIT DIFF

[PASTE DIFF OR SELECT CHANGED FILES]

Review ONLY the changes in this Pull Request.

Check:

1. Functional correctness
2. Business-rule regressions
3. Security risks
4. Input validation
5. Error handling
6. Maintainability
7. Duplicated logic
8. Unnecessary complexity
9. Backward compatibility
10. Missing or weak tests

For every finding provide:

- File
- Relevant changed line/hunk
- Category
- Problem
- Why it matters
- Evidence
- Suggested direction

Classify each finding:

MUST FIX
SHOULD FIX
QUESTION
NIT

Separate confirmed problems from questions.

Do NOT invent code outside the diff.
Do NOT modify anything.
```

---

# 16. Why classify findings?

Use:

```text
MUST FIX
```

Means:

> Cannot safely merge without addressing it.

Example:

```text
Wrong business calculation
```

---

```text
SHOULD FIX
```

Means:

> Important improvement, but not necessarily merge-blocking.

---

```text
QUESTION
```

Means:

> Reviewer needs clarification before deciding.

---

```text
NIT
```

Means:

> Minor style/readability suggestion.

The source lab uses this same practical classification model for review findings. 

---

# PART J — Never Trust AI Findings Automatically

Suppose AI says:

```text
MUST FIX:

NullReferenceException possible on line 45.
```

Do not immediately post it.

Check:

```text
Can this object actually be null?
```

Maybe the constructor guarantees it.

Then AI could be wrong.

So:

```text
AI Finding
   ↓
Inspect actual code
   ↓
Check surrounding context
   ↓
Confirm / Reject
```

---

# PART K — Validate Each AI Finding

Create a table:

| AI Finding | Valid?        | Evidence                    | Action       |
| ---------- | ------------- | --------------------------- | ------------ |
| A1         | Confirmed     | Actual code supports it     | Post comment |
| A2         | Rejected      | AI ignored prior validation | Do not post  |
| A3         | Needs context | Requirement unclear         | Ask question |

This is very important for Lab 22.

---

# PART L — Ask AI to Challenge Its Own Review

Use:

```text
Challenge your previous Pull Request review.

For every finding:

1. State evidence supporting it.
2. State evidence that could disprove it.
3. Identify assumptions.
4. Determine whether surrounding code is needed.
5. Reclassify confidence:
   HIGH / MEDIUM / LOW.

Then classify each finding:

CONFIRMED
LIKELY
QUESTION
REJECT

Do not modify code.
```

This reduces false positives.

---

# PART M — Compare Human vs AI Review

Now compare:

```text
HUMAN
  ↓
What did I detect?


AI
  ↓
What did AI detect?
```

Create:

| Finding               | Human | AI | Final     |
| --------------------- | ----- | -- | --------- |
| Missing test          | ✓     | ✓  | Confirmed |
| Possible null bug     |       | ✓  | Rejected  |
| Duplicated validation | ✓     |    | Confirmed |

This is one of the most useful exercises in Lab 22.

---

# PART N — Turn a Confirmed Finding into a Good PR Comment

A bad review comment:

```text
This is bad.
```

Another bad comment:

```text
Fix this.
```

A good review comment contains:

```text
WHAT

WHY

IMPACT

OPTIONAL DIRECTION
```

Example:

```text
This branch changes the VIP discount path, but there is
no regression test covering VIP customers with coupons.
Because this behavior previously contained a defect,
could you add a regression test confirming the expected
final amount before merging?
```

That is actionable.

---

# PART O — Ask AI to Improve the Review Comment

Use:

```text
Rewrite this confirmed PR finding as a professional,
concise code-review comment.

Finding:
[...]

Evidence:
[...]

Impact:
[...]

Requirements:

- State the issue clearly.
- Explain why it matters.
- Avoid aggressive wording.
- Suggest direction only when useful.
- Do not claim certainty beyond the evidence.
```

Then manually review before posting.

---

# PART P — Review Tests Specifically

Tests should not be treated as an afterthought.

Ask:

```text
Which business behavior changed?
       ↓
Is there a test?
```

For example:

```text
Production Code Changed
        ↓
OrderService discount logic
        ↓
Question:
Which test protects this behavior?
```

---

# Step 17 — Ask AI for test-gap analysis

```text
Review ONLY the changes in this PR and the existing tests.

Do NOT generate tests yet.

For every changed business behavior:

1. Identify the requirement.
2. Identify existing test coverage.
3. Identify missing regression coverage.
4. Rank test gap:
   HIGH / MEDIUM / LOW

Do not recommend tests for private implementation
details unless behavior risk justifies them.
```

---

# PART Q — Check Build Evidence

Before approving:

```text
Build:
PASS?

Tests:
PASS?

Jenkins:
PASS?

SonarQube:
PASS?
```

If your Labs 19–20 environment is available, a PR should ideally include CI evidence.

Conceptually:

```text
PR
 │
 ├── Human Review
 ├── AI Review
 ├── Unit Tests
 ├── Jenkins
 └── SonarQube
        ↓
MERGE DECISION
```

---

# PART R — Make the Final Review Decision

Use one of:

```text
APPROVE
```

Meaning:

> No merge-blocking problem found.

---

```text
REQUEST CHANGES
```

Meaning:

> At least one confirmed issue needs correction.

---

```text
COMMENT / QUESTION
```

Meaning:

> Need clarification, but not necessarily blocking yet.

The exact buttons differ slightly between GitHub and Gitea, but the engineering decision is the same.

---

# PART S — Ask AI for the Final Review Summary

Use:

```text
You are helping me prepare the final Pull Request review.

Use ONLY these confirmed findings:

[PASTE CONFIRMED FINDINGS]

Rejected AI findings:

[PASTE REJECTED FINDINGS]

Build/Test Evidence:

[PASTE ACTUAL RESULTS]

Create a concise PR review summary containing:

1. Overall assessment
2. MUST FIX findings
3. SHOULD FIX findings
4. Test coverage assessment
5. Security assessment
6. Final recommendation:

   APPROVE
   REQUEST CHANGES
   NEEDS CLARIFICATION

Explain the recommendation.

Do not reintroduce rejected findings.
```

---

# PART T — Example Review Summary

If everything is okay:

```text
Overall assessment:

The change is focused and aligns with the stated
requirement. Existing behavior remains covered and the
build/tests pass.

Must Fix:
None.

Should Fix:
None confirmed.

Test Coverage:
Relevant regression tests are present.

Security:
No new confirmed security issue identified.

Recommendation:
APPROVE.
```

If there is a real issue:

```text
Overall assessment:

The main implementation is understandable, but one
business-critical regression path is not currently
protected by a test.

Must Fix:
Add regression coverage for VIP coupon behavior.

Recommendation:
REQUEST CHANGES.
```

Use actual findings.

---

# PART U — Optional Controlled Defect Exercise

If the class needs a clear PR-review demonstration, introduce a safe bug in the training branch.

For example, change:

```csharp
if (hasCoupon && customerType != "VIP")
```

to:

```csharp
if (hasCoupon)
```

This could reintroduce the old VIP coupon behavior.

The reviewer should notice:

```text
VIP coupon logic changed
      ↓
Potential business regression
      ↓
Existing regression test?
      ↓
Jenkins test should catch it
```

This connects Lab 22 back to Labs 3 and 13.

After the exercise, revert the defect.

---

# PART V — Create the Lab 22 Report

Create:

```text
AI_ASSISTED_PR_REVIEW.md
```

Use:

```text
Create AI_ASSISTED_PR_REVIEW.md using ONLY
actual Lab 22 evidence.

# Lab 22 — AI-Assisted Pull Request Review

## 1. Pull Request

Title:
[...]

Repository:
[...]

Base Branch:
main

Feature Branch:
lab22-pr-review

## 2. Purpose of Change

[...]

## 3. Files Changed

[...]

## 4. Build and Test Evidence

Build:
PASS / FAIL

Tests:
[...]

Jenkins:
[...]

SonarQube:
[...]

## 5. Initial Human Review

### H1

File:
[...]

Category:
[...]

Finding:
[...]

## 6. AI Review

### A1

File:
[...]

Classification:
MUST FIX / SHOULD FIX / QUESTION / NIT

Finding:
[...]

Confidence:
[...]

## 7. AI Validation

Finding | Result | Evidence

Confirmed / Rejected / Needs Context

## 8. Human vs AI Comparison

Finding
Human
AI
Final Decision

## 9. Test Coverage Review

Changed Behavior:
[...]

Existing Test:
[...]

Gap:
[...]

## 10. Final Review Comments

[...]

## 11. Final Decision

APPROVE /
REQUEST CHANGES /
NEEDS CLARIFICATION

Reason:
[...]

## 12. AI Calibration

Useful AI Finding:
[...]

Rejected AI Finding:
[...]

Why Rejected:
[...]

## 13. Conclusion

Explain where AI helped and where human validation
was necessary.

Do not invent findings or test results.
```

---

# Lab 22 Completion Checklist

* [ ] Student understands what a Pull Request is.
* [ ] Student understands source branch vs target branch.
* [ ] Lab 22 feature branch created.
* [ ] Small controlled change made.
* [ ] Project builds.
* [ ] Tests pass before PR.
* [ ] Change committed.
* [ ] Branch pushed.
* [ ] Pull Request created.
* [ ] PR description explains what/why/testing/risk.
* [ ] Files Changed reviewed manually.
* [ ] Human findings recorded before AI review.
* [ ] AI reviewed actual diff.
* [ ] AI checked correctness.
* [ ] AI checked security.
* [ ] AI checked maintainability.
* [ ] AI checked tests.
* [ ] AI checked backward compatibility.
* [ ] Findings classified MUST FIX / SHOULD FIX / QUESTION / NIT.
* [ ] Each AI finding manually validated.
* [ ] At least one AI finding accepted or documented.
* [ ] At least one AI finding rejected if applicable.
* [ ] Human vs AI comparison completed.
* [ ] Confirmed findings converted into actionable comments.
* [ ] Build/test/CI evidence considered.
* [ ] Final decision made.
* [ ] `AI_ASSISTED_PR_REVIEW.md` created.

# The easiest way to remember Lab 22

```text
                   DEVELOPER
                       │
                       ▼
                 FEATURE BRANCH
                       │
                       ▼
                 CHANGE + TEST
                       │
                       ▼
                    GIT PUSH
                       │
                       ▼
                 PULL REQUEST
                       │
              ┌────────┴────────┐
              ▼                 ▼
        HUMAN REVIEW         AI REVIEW
              │                 │
              └────────┬────────┘
                       ▼
                VERIFY FINDINGS
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       MUST FIX    SHOULD FIX    QUESTION
          │
          └────────────┼────────────┘
                       ▼
               BUILD / TEST / CI
                       │
                       ▼
                 FINAL DECISION
                  /           \
              APPROVE      REQUEST CHANGES
```

The course progression is now:

```text
LAB 19
Jenkins automatically builds/tests

        ↓

LAB 20
Jenkins checks SonarQube Quality Gate

        ↓

LAB 21
Troubleshoot failed CI

        ↓

LAB 22
Review the CODE CHANGE before merge

        ↓

LAB 23
Protect the repository so reviews/checks
cannot easily be bypassed
```

The single sentence to remember is:

> **Lab 22 = “Before this branch enters `main`, review the actual diff manually, ask AI for a second opinion, verify every AI claim, check test/CI evidence, then approve or request changes.”**
