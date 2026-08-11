Lab 23 is **“Design Repository Governance and Branch Protection Policies.”** It asks students to document the current branch/merge process, identify governance risks, define protections for `main`, configure them where permitted, test them with a PR, try a prohibited action, and document an exception process. 

# Lab 23 — Design Repository Governance and Branch Protection Policies

## 1. What does Lab 23 mean?

Lab 22 was:

```text
Developer creates Pull Request
        ↓
Human reviews
        ↓
AI helps review
        ↓
Approve / Request Changes
```

But there is a problem.

What if someone simply does:

```text
git push origin main
```

and bypasses the Pull Request?

Then all the good practices from Labs 19–22 can be ignored.

So Lab 23 says:

> **Make the repository enforce the rules instead of relying only on people remembering them.**

```text
                    WITHOUT GOVERNANCE

Developer
    ↓
Can push directly to main
    ↓
No PR
No review
No CI requirement
    ↓
Risk


                    WITH GOVERNANCE

Developer
    ↓
Feature branch
    ↓
Pull Request
    ↓
Review required
    ↓
CI required
    ↓
All conditions pass?
       │
   ┌───┴───┐
   ▼       ▼
  YES      NO
   │        │
 MERGE    BLOCK
```

---

# 2. What is Repository Governance?

Repository governance means:

> The rules controlling **who may change the repository, how changes enter important branches, and what checks must pass first**.

Think about five questions:

| Governance Question                 | Example          |
| ----------------------------------- | ---------------- |
| Where should developers work?       | Feature branches |
| Can they push directly to `main`?   | No               |
| Does someone need to review?        | Yes              |
| Must CI pass before merging?        | Yes              |
| Can someone rewrite `main` history? | No               |

This is exactly the scope of the original Lab 23: branch model, reviews, CI checks, force pushes, ownership, exception handling and enforcement. 

---

# 3. What is Branch Protection?

Suppose you have:

```text
main
```

This is your important branch.

Without protection:

```text
Developer
   ↓
git push origin main
   ↓
main changed
```

With branch protection:

```text
Developer
   ↓
git push origin main
   ↓
REJECTED

"You must use the approved workflow."
```

GitHub supports both classic branch protection rules and newer **rulesets**. Rulesets can require Pull Requests, required status checks, and block force pushes; multiple rulesets can also apply together. ([GitHub Docs][1])

For this beginner lab, I recommend using:

# **GitHub Rulesets**

if your training repository is on GitHub.

---

# 4. What should we protect?

For the first exercise, only protect:

```text
main
```

Don't create 20 policies.

Use a simple rule:

```text
MAIN BRANCH POLICY

1. No direct changes to main
2. Pull Request required
3. At least 1 reviewer approval
4. Important CI check must pass
5. Conversations should be resolved
6. Force push prohibited
7. Deletion prohibited
```

That is enough to understand the concept.

GitHub rulesets support requiring a Pull Request, required status checks, approval rules, resolution of review conversations and blocking force pushes. ([GitHub Docs][2])

---

# PART A — Understand the Current Problem

## Step 1 — Open your repository

Open your training GitHub repository.

For example:

```text
AI_SDLC_Labs
```

Go to:

```text
Code
```

Look at the branch:

```text
main
```

---

# Step 2 — Record the current governance

Create:

```text
REPOSITORY_GOVERNANCE_BASELINE.md
```

Start with:

```markdown
# Lab 23 — Repository Governance Baseline

## Repository

[actual repository]

## Default Branch

main

## Current Workflow

Developer Branch:
Yes / No

Pull Request Required:
Yes / No

Reviewer Approval Required:
Yes / No

CI Required:
Yes / No

Direct Push to main:
Allowed / Blocked / Unknown

Force Push:
Allowed / Blocked / Unknown

Branch Deletion:
Allowed / Blocked / Unknown

Admin Bypass:
[actual / unknown]
```

Don't guess.

Check actual repository settings.

---

# PART B — Understand the Risks

Suppose:

```text
Direct push to main:
Allowed
```

Risk:

```text
Developer
   ↓
Changes main
   ↓
No review
   ↓
Potential defect enters main
```

Suppose:

```text
CI not required
```

Risk:

```text
Pull Request created
       ↓
Tests fail
       ↓
Someone merges anyway
```

Suppose:

```text
Force push allowed
```

Risk:

```text
main history
     ↓
rewritten
     ↓
existing commits can disappear/change
```

So Lab 23 is about converting these risks into repository-enforced controls.

---

# PART C — Ask AI to Review Your Current Governance

Use Cursor/Copilot:

```text
You are a Senior DevOps and Repository Governance Engineer.

I am a beginner.

Do NOT change my repository settings.

Current repository:

Platform:
GitHub

Default Branch:
main

Current Controls:

Pull Request Required:
[YES / NO]

Reviewer Approval:
[...]

CI Required:
[...]

Direct Push:
[...]

Force Push:
[...]

Branch Deletion:
[...]

Admin Bypass:
[...]

Identify governance risks.

For each provide:

1. Risk
2. Why it matters
3. Possible consequence
4. Recommended control
5. Priority:
   HIGH / MEDIUM / LOW

Keep the recommendations appropriate for a
small training .NET project.

Do not recommend enterprise-level complexity
unless necessary.
```

---

# PART D — Define the Target Policy

For this lab, use a simple target policy:

| Control               | Lab 23 Target                   |
| --------------------- | ------------------------------- |
| Protected branch      | `main`                          |
| Pull Request required | Yes                             |
| Required approvals    | 1                               |
| Resolve conversations | Yes                             |
| Required CI           | Use only actual available check |
| Force push            | Block                           |
| Branch deletion       | Block                           |
| Bypass                | Restrict / document             |

This is a practical beginner baseline, not a universal company policy.

---

# PART E — Create a GitHub Ruleset

## Step 3 — Open repository settings

In GitHub:

```text
Repository
   ↓
Settings
```

Then:

```text
Code and automation
   ↓
Rules
   ↓
Rulesets
```

Click:

```text
New ruleset
   ↓
New branch ruleset
```

GitHub's current documented path is **Repository → Settings → Rules → Rulesets → New ruleset → New branch ruleset**. ([GitHub Docs][3])

---

# Step 4 — Give the Ruleset a name

Use:

```text
Main Branch Protection
```

Set enforcement:

```text
Active
```

An active ruleset takes effect immediately after creation. ([GitHub Docs][3])

---

# Step 5 — Target the main branch

Find:

```text
Target branches
```

Choose the option targeting your default branch or specifically:

```text
main
```

The intended result is:

```text
Ruleset:
Main Branch Protection

Target:
main
```

---

# PART F — Require a Pull Request

## Step 6 — Enable

Find:

```text
Require a pull request before merging
```

Enable it.

This means:

```text
Developer
   ↓
Directly modify main?
   ↓
NO

Developer
   ↓
Feature branch
   ↓
PR
   ↓
main
```

GitHub rulesets can require all changes to the targeted branch to be associated with a Pull Request. ([GitHub Docs][2])

---

# Step 7 — Require one approval

Inside the PR requirement, configure:

```text
Required approvals:
1
```

For this training lab, one reviewer is enough to understand the workflow.

Then:

```text
Developer
   ↓
Creates PR
   ↓
Reviewer approval required
   ↓
Merge
```

---

# Step 8 — Dismiss stale approvals

A useful protection is:

```text
Dismiss stale pull request approvals
when new commits are pushed
```

Why?

Suppose:

```text
Reviewer approves version A
        ↓
Developer pushes new version B
        ↓
Old approval remains?
```

That can mean the latest change was never reviewed.

GitHub can dismiss approvals when new reviewable commits change the Pull Request. ([GitHub Docs][2])

For a beginner lab, this is a good control to discuss even if you decide not to enable every option.

---

# PART G — Require Conversations to be Resolved

## Step 9 — Enable

Find:

```text
Require conversation resolution before merging
```

The workflow becomes:

```text
Reviewer:
"Please fix this validation."

        ↓

Developer fixes it

        ↓

Conversation resolved

        ↓

Merge allowed
```

GitHub can require Pull Request review conversations to be resolved before merging. ([GitHub Docs][2])

---

# PART H — Add Required CI Checks Carefully

This connects Lab 23 to:

```text
Lab 19 → Jenkins Build/Test
Lab 20 → SonarQube Quality Gate
```

But there is a critical rule:

> **Only require a CI status check that your GitHub repository actually receives.**

Do not invent:

```text
Jenkins-Build
```

as a required check if GitHub has never received such a status.

GitHub required status checks require the named checks/statuses to report successfully before a merge is allowed. ([GitHub Docs][2])

---

# Step 10 — Check your PR first

Open a Pull Request.

Look for:

```text
Checks
```

or the merge box.

You might see an actual check such as:

```text
build
```

or:

```text
Jenkins
```

or another CI integration.

Record the exact name.

For example:

```text
Actual Check Name:

Jenkins / Lab02Api
```

Use **your actual result**.

---

# Step 11 — Require the actual check

Back in your ruleset:

Enable:

```text
Require status checks to pass
```

Add the actual check that has reported into GitHub.

GitHub's ruleset configuration requires entering/selecting the status check name that should be required. ([GitHub Docs][3])

Now:

```text
PR
  ↓
Review approved
  ↓
CI FAILED
  ↓
MERGE BLOCKED
```

and:

```text
PR
  ↓
Review approved
  ↓
CI PASSED
  ↓
Merge can continue
```

---

# Important: What if Jenkins is NOT posting status to GitHub?

Then do **not** fake this part.

Record:

```text
Required CI Status Check:
Not configured yet

Reason:
Jenkins currently does not report a GitHub
status/check that can be selected.
```

Your policy document can still state:

```text
Target Policy:
Require Jenkins CI before merge

Implementation Status:
Pending CI-to-GitHub status integration
```

The original Lab 23 explicitly requires governance controls to align with the **actual CI capability**, not imaginary checks. 

---

# PART I — Block Force Push

## Step 12 — Make sure force push isn't allowed

A normal history looks like:

```text
A → B → C → D
```

A force push can rewrite that history:

```text
A → B → X
```

potentially replacing commits.

For `main`, generally keep:

```text
Force Push:
BLOCKED
```

GitHub's branch protection/rules mechanisms support preventing force pushes to protected branches. ([GitHub Docs][4])

---

# PART J — Protect Deletion

You also don't want someone accidentally deleting:

```text
main
```

Protected branch policies can prevent deletion of important branches. ([GitHub Docs][4])

So:

```text
Delete main:
BLOCKED
```

---

# PART K — Be Careful With Bypass Rules

Rules often need exceptions for legitimate emergency or administrative work.

But don't make:

```text
Everyone can bypass
```

because then:

```text
Branch Protection
        =
Suggestion
```

Instead:

```text
Normal Developer
       ↓
Must follow PR + CI


Authorized Emergency Role
       ↓
Bypass only when necessary
       ↓
Reason documented
       ↓
Audit trail
```

GitHub rulesets support designated bypass actors such as roles, teams and GitHub Apps. ([GitHub Docs][3])

For the classroom, keep bypass permissions minimal.

---

# PART L — Create the Ruleset

Review:

```text
Ruleset Name:
Main Branch Protection

Target:
main

Pull Request:
Required

Approvals:
1

Conversation Resolution:
Required

Status Check:
[actual, if available]

Force Push:
Blocked

Deletion:
Blocked
```

Click:

```text
Create
```

---

# PART M — Test the Protection

This is the important part.

A policy document is not evidence that the policy works.

You need to try it.

The original Lab 23 explicitly requires a test Pull Request and a prohibited action to prove the controls actually work. 

## Step 13 — Create a new branch

```powershell
cd C:\AI_SDLC_Labs

git checkout main
git pull

git checkout -b lab23-governance-test
```

Make a tiny harmless change.

For example create:

```text
LAB23_TEST.md
```

with:

```text
Lab 23 branch protection test.
```

---

# Step 14 — Commit and push

```powershell
git add LAB23_TEST.md

git commit -m "Add Lab 23 governance test"

git push -u origin lab23-governance-test
```

Create a Pull Request:

```text
lab23-governance-test
        ↓
main
```

---

# PART N — Test "No Approval"

Before anyone approves the PR, try to merge.

Expected behavior if approval is required:

```text
Pull Request
     ↓
0 approvals
     ↓
MERGE BLOCKED
```

Record:

```text
Test 1:
Merge without approval

Expected:
Blocked

Actual:
[actual]
```

---

# PART O — Approve the PR

Have another authorized reviewer perform:

```text
Review changes
     ↓
Approve
```

Now:

```text
Approval:
PASS
```

But if CI is required and hasn't passed:

```text
CI:
FAIL / PENDING

       ↓

MERGE STILL BLOCKED
```

That is exactly what governance should do.

---

# PART P — Test Failing CI

If your Jenkins/GitHub integration reports a required check, use a **safe test branch** and deliberately fail one test as you did in Lab 19.

Example:

```text
PR
 ↓
Reviewer approval ✓
 ↓
Jenkins test ✗
 ↓
Required Check ✗
 ↓
MERGE BLOCKED
```

This is an excellent Lab 23 demonstration.

Then restore the test.

---

# PART Q — Test Direct Push to Main

Now try a prohibited action.

From your local repo:

```powershell
git checkout main
```

Create a harmless file such as:

```text
DIRECT_PUSH_TEST.md
```

Commit locally:

```powershell
git add DIRECT_PUSH_TEST.md

git commit -m "Test direct push protection"
```

Then attempt:

```powershell
git push origin main
```

If your policy is correctly configured for your account/permissions, the server should reject the prohibited update.

Record the actual response.

Do not attempt this on a production repository.

---

# Important: Admin behavior can differ

If you're the repository administrator, whether you can bypass a policy depends on the ruleset/bypass configuration. GitHub lets rulesets define who may bypass them. ([GitHub Docs][3])

So if an administrator can still push, don't conclude:

```text
Branch protection is broken.
```

Instead inspect:

```text
Bypass permissions
```

and document the actual behavior.

---

# PART R — Ask AI to Review the Policy

Once your policy is configured, use:

```text
You are a Senior DevOps Governance Reviewer.

Review this repository policy.

Platform:
GitHub

Protected Branch:
main

Pull Request Required:
Yes

Required Approvals:
1

Dismiss Stale Approvals:
[Yes / No]

Conversation Resolution:
[Yes / No]

Required CI Checks:
[actual]

Force Push:
[Allowed / Blocked]

Deletion:
[Allowed / Blocked]

Bypass:
[actual]

Evaluate:

1. Quality risks
2. Security risks
3. Developer friction
4. Possible bypasses
5. Emergency-change handling
6. Whether the controls match the CI capability

Classify recommendations:

MUST HAVE
SHOULD HAVE
OPTIONAL

Keep the policy practical for a small
development team.

Do not recommend complexity without a
clear risk it addresses.
```

---

# PART S — Understand CODEOWNERS

Later, some parts of the repository may require a specialist.

Example:

```text
/src/security/
```

Maybe:

```text
Security Team must review
```

or:

```text
/Jenkinsfile
```

Maybe:

```text
DevOps Team must review
```

GitHub rulesets can require review from code owners when a Pull Request touches owned code. ([GitHub Docs][2])

Conceptually:

```text
Normal code
     ↓
General reviewer


Security code
     ↓
Security owner


CI pipeline
     ↓
DevOps owner
```

For the first Lab 23 exercise, CODEOWNERS is **optional**. Understand basic branch protection first.

---

# PART T — Emergency Change Policy

Repository governance shouldn't make emergency fixes impossible.

A simple policy could be:

```text
Production emergency
       ↓
Authorized owner approves bypass
       ↓
Emergency change
       ↓
Reason recorded
       ↓
Post-change PR/review
       ↓
Audit trail
```

The key principle is:

> Emergency exception does not mean “ignore all rules whenever inconvenient.”

The original Lab 23 specifically expects a controlled exception process with ownership and an audit trail. 

---

# PART U — If You Use Gitea Instead

The concept is almost identical.

In current Gitea documentation:

```text
Repository
   ↓
Settings
   ↓
Branches
   ↓
Add new rule
```

Then configure the protected branch pattern, push rules, approvals, force-push restrictions and status checks. ([Gitea Documentation][5])

For:

```text
Protected branch:
main
```

you can configure controls such as disabling direct pushes, required approvals, blocking merges on rejected/outstanding reviews, requiring the PR to be up to date, and requiring CI status checks. ([Gitea Documentation][5])

So the concepts learned in this lab transfer directly between GitHub and Gitea.

---

# PART V — Final Recommended Lab Policy

For your beginner course, this is enough:

| Setting                 | Recommended Lab Value                   |
| ----------------------- | --------------------------------------- |
| Branch                  | `main`                                  |
| Direct development      | Feature branches                        |
| Pull Request            | Required                                |
| Reviewer approvals      | 1                                       |
| Conversation resolution | Required                                |
| CI status               | Required **only if actually available** |
| Force push              | Block                                   |
| Branch deletion         | Block                                   |
| CODEOWNERS              | Optional                                |
| Admin bypass            | Minimize/document                       |
| Emergency exception     | Document owner + reason                 |

---

# PART W — Create the Lab 23 Report

Create:

```text
REPOSITORY_GOVERNANCE_POLICY.md
```

Use this structure:

```markdown
# Lab 23 — Repository Governance Policy

## 1. Repository

Platform:
GitHub / Gitea

Repository:
[...]

Default Branch:
main

## 2. Baseline

Direct Push:
[...]

Pull Request Required:
[...]

Approval Required:
[...]

CI Required:
[...]

Force Push:
[...]

Deletion:
[...]

## 3. Identified Risks

Risk 1:
[...]

Impact:
[...]

Control:
[...]

## 4. Target Governance Policy

Protected Branch:
main

Pull Request Required:
Yes

Required Approvals:
1

Conversation Resolution:
[...]

Required Status Checks:
[...]

Force Push:
Blocked

Deletion:
Blocked

## 5. Bypass Policy

Who Can Bypass:
[...]

When:
[...]

Approval:
[...]

Audit Evidence:
[...]

## 6. Test PR

Branch:
lab23-governance-test

PR:
[...]

Approval Requirement:
PASS / FAIL

CI Requirement:
PASS / FAIL / Not Configured

## 7. Prohibited Action Test

Action:
Direct push to main

Expected:
Blocked

Actual:
[...]

## 8. Quality Control Test

Failing CI prevents merge:
Yes / No / Not Configured

## 9. Exceptions

Emergency Procedure:
[...]

Owner:
[...]

Audit Requirement:
[...]

## 10. Remaining Gaps

[...]

## 11. Conclusion

State whether the repository now prevents
unreviewed or unvalidated changes from entering main.

Use only actual evidence.
```

# Lab 23 Completion Checklist

* [ ] Understand repository governance.
* [ ] Understand protected branches.
* [ ] Identify `main` as the protected branch.
* [ ] Record current repository controls.
* [ ] Identify governance risks.
* [ ] Define target policy.
* [ ] Create GitHub Ruleset or Gitea protection rule.
* [ ] Require Pull Request.
* [ ] Require at least one approval.
* [ ] Require conversation resolution if appropriate.
* [ ] Require only CI checks that actually exist.
* [ ] Block force push.
* [ ] Block branch deletion.
* [ ] Review bypass permissions.
* [ ] Create a test branch.
* [ ] Create test PR.
* [ ] Confirm unapproved PR is blocked.
* [ ] Confirm failing CI blocks merge if status integration exists.
* [ ] Test a prohibited direct push where safe.
* [ ] Document actual result.
* [ ] Define emergency exception procedure.
* [ ] Create `REPOSITORY_GOVERNANCE_POLICY.md`.

# The easiest way to remember Lab 23

```text
                        MAIN
                         ▲
                         │
                  PROTECTED BRANCH
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    PR REQUIRED      REVIEW REQUIRED    CI REQUIRED
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                    ALL PASSED?
                     /       \
                   YES       NO
                    │         │
                    ▼         ▼
                  MERGE      BLOCK
```

So the Labs now connect like this:

```text
LAB 22

"Should this Pull Request be approved?"

Human + AI review

        ↓

LAB 23

"Even if somebody wants to bypass the review,
will the repository ALLOW them?"

Repository governance

        ↓

LAB 24

"AI generated this code.
How do we review it consistently?"
```

The single sentence to remember is:

> **Lab 22 teaches people how to review a PR. Lab 23 makes the repository enforce that the PR, review, and available CI controls actually happen before code enters `main`.**

[1]: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets?utm_source=chatgpt.com "About rulesets - GitHub Docs"
[2]: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets?utm_source=chatgpt.com "Available rules for rulesets - GitHub Docs"
[3]: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository "Creating rulesets for a repository - GitHub Docs"
[4]: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches?ref=pantsbuild-open-source-community-blog&utm_source=chatgpt.com "About protected branches - GitHub Docs"
[5]: https://docs.gitea.com/1.25/usage/access-control/protected-branches "Protected branches | Gitea Documentation"
