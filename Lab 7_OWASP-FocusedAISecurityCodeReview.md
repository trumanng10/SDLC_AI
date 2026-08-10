Include **OWASP security risks, AI-assisted security code reviews, identifying weaknesses, prioritizing findings, and remediation recommendations**. 

For this lab, we will deliberately create **three simple security problems** inside the same .NET project and ask Cursor/Copilot to find them.

One update from the earlier lab guide: the current OWASP Top 10 release is **OWASP Top 10:2025**, not 2021. It includes categories such as Broken Access Control, Security Misconfiguration, Software Supply Chain Failures, Cryptographic Failures, Injection, and Authentication Failures. ([OWASP Foundation][1])

# Lab 7 — Conduct an OWASP-Focused AI Security Code Review

## Objectives

By the end of Lab 7, you should be able to:

* Understand what OWASP means.
* Understand what a security code review is.
* Recognize several insecure coding patterns.
* Use AI to inspect C# code for security weaknesses.
* Map findings to OWASP categories.
* Rank findings by severity.
* Ask AI for remediation recommendations.
* Validate AI findings instead of blindly accepting them.
* Create a `SECURITY_REVIEW_REPORT.md`.

---

# Part 1 — What is OWASP?

OWASP stands for:

```text
Open Worldwide Application Security Project
```

The **OWASP Top 10** is an awareness document describing major categories of web-application security risk. The current 2025 list contains these categories: ([OWASP Foundation][1])

```text
A01 — Broken Access Control
A02 — Security Misconfiguration
A03 — Software Supply Chain Failures
A04 — Cryptographic Failures
A05 — Injection
A06 — Insecure Design
A07 — Authentication Failures
A08 — Software or Data Integrity Failures
A09 — Security Logging and Alerting Failures
A10 — Mishandling of Exceptional Conditions
```

**You do not need to memorize all ten for Lab 7.**

We will concentrate on three easy ones:

```text
A05 — Injection

A04 — Cryptographic Failures /
       sensitive secret handling

A10 — Mishandling Exceptional Conditions
```

We'll also discuss security misconfiguration where appropriate.

---

# Part 2 — What is a security code review?

In previous labs we asked:

```text
LAB 4
Is this code maintainable?

LAB 5
Can we refactor it?

LAB 6
Is it slow?
```

Now the question becomes:

```text
LAB 7
Can somebody misuse this code
to do something they shouldn't?
```

For example:

```csharp
string query =
    "SELECT * FROM Customers WHERE Name = '" +
    customerName +
    "'";
```

It may compile.

It may even work.

But because user input is being inserted directly into a database command, it represents the sort of unsafe query construction associated with **Injection**. OWASP specifically identifies dynamic or non-parameterized queries involving untrusted input as an injection risk. ([OWASP Foundation][2])

---

# Part 3 — Continue from Lab 6

Open PowerShell:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Build:

```powershell
dotnet build
```

Expected:

```text
Build succeeded.
```

Open the project in Cursor.

You should now have approximately:

```text
Lab02Api
│
├── Program.cs
├── OrderService.cs
├── CustomerReportService.cs
├── PerformanceService.cs
├── CODE_QUALITY_BASELINE.md
├── PERFORMANCE_DEPENDENCY_REPORT.md
└── Lab02Api.csproj
```

---

# Part 4 — Create deliberately insecure code

Right-click project:

**New File**

Create:

```text
SecurityDemoService.cs
```

Paste this code:

```csharp
public class SecurityDemoService
{
    private const string ApiKey =
        "DEMO-SECRET-12345";

    public string BuildCustomerQuery(
        string customerName)
    {
        return
            "SELECT * FROM Customers " +
            "WHERE Name = '" +
            customerName +
            "'";
    }

    public string GetApiKey()
    {
        return ApiKey;
    }

    public string GetDetailedError(
        Exception exception)
    {
        return exception.ToString();
    }
}
```

Save:

```text
Ctrl + S
```

## Important

`DEMO-SECRET-12345` is deliberately fake.

**Never paste a real API key, password, database password, token, or production credential into this lab.**

---

# Part 5 — What problems did I deliberately create?

Do **not** tell Cursor yet.

You should discover them through AI review.

There are three main problems hidden here:

```text
SecurityDemoService
        │
        ├── Hard-coded fake secret
        │
        ├── Unsafe SQL construction
        │
        └── Detailed exception disclosure
```

---

# Part 6 — Build first

Run:

```powershell
dotnet build
```

You will probably get:

```text
Build succeeded.
```

Here is another important lesson:

```text
Build succeeded
       ≠
Secure application
```

Exactly like:

```text
Build succeeded
       ≠
Good code quality
```

The compiler does not automatically catch every application-security problem.

---

# Part 7 — First ask AI to explain the code

Open Cursor Chat.

Enter:

```text
Review SecurityDemoService.cs.

Do NOT modify the source code.

I am learning application security.

First explain what each method does in beginner-friendly
language.

Explain:

1. What BuildCustomerQuery does
2. What GetApiKey does
3. What GetDetailedError does
4. What data enters each method
5. What data leaves each method

Do not perform remediation yet.
```

This keeps the workflow disciplined:

```text
Understand
   ↓
Review
   ↓
Fix
```

rather than:

```text
AI, fix everything!
```

---

# Part 8 — Ask AI for an OWASP review

Now give Cursor:

```text
Conduct an OWASP-focused security code review of
SecurityDemoService.cs.

Use OWASP Top 10:2025 terminology where applicable.

Do NOT modify any code.

For every potential security finding provide:

1. Finding name
2. Exact affected code
3. OWASP category
4. Explanation
5. Possible security impact
6. Severity: Low / Medium / High / Critical
7. Confidence: Low / Medium / High
8. Recommended remediation

Do not invent vulnerabilities that are not supported by
the code.
```

This is the core of Lab 7.

---

# Part 9 — Finding #1: Unsafe query construction

AI should notice:

```csharp
return
    "SELECT * FROM Customers " +
    "WHERE Name = '" +
    customerName +
    "'";
```

The problem is:

```text
Application SQL
      +
User-controlled value
      ↓
combined directly
```

Conceptually:

```text
customerName
     ↓
placed directly into SQL
     ↓
database interpreter receives it
```

This maps naturally to:

```text
OWASP A05:2025 — Injection
```

OWASP's Injection guidance warns about untrusted input being used in dynamic/non-parameterized queries. ([OWASP Foundation][2])

---

# Part 10 — You can demonstrate the coding problem safely

We don't need a database.

Suppose:

```text
customerName = John
```

The generated text becomes:

```sql
SELECT * FROM Customers
WHERE Name = 'John'
```

Now suppose the legitimate customer's surname contains an apostrophe:

```text
O'Brien
```

The resulting SQL text becomes:

```sql
SELECT * FROM Customers
WHERE Name = 'O'Brien'
```

Look carefully:

```text
'O'Brien'
```

The quotes no longer behave as intended.

This already demonstrates why **manually concatenating SQL strings is fragile**.

We don't need to attack a real database to understand the security issue.

---

# Part 11 — Ask AI how it should be fixed

Now prompt:

```text
For the unsafe SQL construction finding only:

Explain how a production .NET application should avoid
building SQL statements by concatenating user input.

Discuss parameterized queries.

Do not implement a real database connection.

Show a conceptual safe example only.
```

AI should explain parameterization.

Conceptually:

```text
BAD

SQL + user text
      ↓
mixed together


BETTER

SQL command
      +
parameter value
      ↓
kept separate
```

For example conceptually:

```csharp
command.CommandText =
    "SELECT * FROM Customers WHERE Name = @name";

command.Parameters.AddWithValue(
    "@name",
    customerName);
```

The precise database API would depend on which database provider your application actually uses, so we are not introducing one merely for this lab.

---

# Part 12 — Finding #2: Hard-coded secret

AI should also notice:

```csharp
private const string ApiKey =
    "DEMO-SECRET-12345";
```

Ask:

```text
What is wrong with storing a secret directly in source code?

Explain:

1. Git repository risk
2. Source code sharing risk
3. Log/screenshot risk
4. Secret rotation difficulty
5. Better approaches in .NET

Do not modify the code yet.
```

OWASP's cryptographic-failures material has historically included hard-coded credentials among relevant weaknesses; in the current 2025 Top 10, Cryptographic Failures is A04. ([OWASP Foundation][1])

---

# Part 13 — Why hard-coded secrets are bad

Imagine:

```text
Program.cs
SecurityDemoService.cs
appsettings.json
     ↓
Git Commit
     ↓
GitHub / Gitea
     ↓
20 developers clone repository
```

If this exists:

```csharp
string password = "MyRealPassword123";
```

then everybody who can read the repository may potentially see it.

Even if you later delete it:

```text
Git history
```

may still contain previous versions.

So:

```text
SOURCE CODE
   ≠
SECRET STORAGE
```

---

# Part 14 — Where should secrets go?

For local .NET development, developers commonly separate configuration from source code and can use mechanisms such as environment variables or development secret storage.

Conceptually:

```text
Source Code
     │
     │ asks for
     ▼
Configuration
     │
     ├── Environment Variable
     ├── Secret Manager
     └── External Secret Store
```

Then source code does not contain:

```text
actual password
actual API key
actual token
```

For this beginner lab, understanding the pattern is enough.

---

# Part 15 — Finding #3: Returning full exception details

Now look at:

```csharp
public string GetDetailedError(
    Exception exception)
{
    return exception.ToString();
}
```

Ask Cursor:

```text
Analyze GetDetailedError from a security perspective.

Explain why returning exception.ToString() directly to an
external API caller may be dangerous.

Give examples of the TYPES of internal information that
could unintentionally be revealed.

Do not modify the code.
```

AI may mention:

```text
Stack traces
Internal class names
File paths
Framework information
Database details
Implementation details
```

The current OWASP Top 10:2025 includes **A10 Mishandling of Exceptional Conditions**, which is directly relevant to thinking about how application failures and exception handling are managed. ([OWASP Foundation][1])

---

# Part 16 — Internal error vs external error

Bad concept:

```text
Something fails
      ↓
Exception.ToString()
      ↓
send everything to customer
```

Better concept:

```text
Something fails
      │
      ├───────────────► Internal log
      │                  detailed error
      │
      ▼
External response
"An unexpected error occurred."
```

So:

```text
DEVELOPERS
may need detailed diagnostics

USERS
normally don't need internal stack traces
```

---

# Part 17 — Ask AI to prioritize the findings

Now enter:

```text
Rank the security findings in SecurityDemoService.cs.

Use this table:

Priority
Finding
OWASP Category
Severity
Likelihood
Potential Impact
Confidence
Recommended Action

Explain why finding #1 should be addressed before
finding #2, etc.

Do not modify anything.
```

A reasonable result might resemble:

| Priority | Finding                       | OWASP                      | Severity |
| -------- | ----------------------------- | -------------------------- | -------- |
| 1        | Unsafe SQL construction       | A05 Injection              | High     |
| 2        | Hard-coded secret             | A04 Cryptographic Failures | High     |
| 3        | Detailed exception disclosure | A10 Exceptional Conditions | Medium   |

The exact severity is contextual, so the important part is **AI's reasoning**, not merely the label.

---

# Part 18 — Very important: challenge the AI

AI security reviews can produce false positives.

Ask:

```text
Now challenge your own security review.

For each finding classify it as:

CONFIRMED
LIKELY
NEEDS MORE CONTEXT

Explain what additional evidence would be required
to confirm the issue.

Do not modify code.
```

This is an excellent security-review habit.

For example:

### Query construction

```text
The pattern is clearly unsafe-looking.

But is the returned SQL ever actually executed?

Need more context.
```

In our demo:

```text
It isn't connected to a database.
```

Therefore the code contains a **dangerous pattern**, but this training version does not create a functioning SQL-injection path against a real database.

This distinction matters.

---

# Part 19 — Search the entire project

Now tell Cursor:

```text
Review the entire .NET project for OWASP-related security
risks.

Focus on:

- Program.cs
- OrderService.cs
- CustomerReportService.cs
- PerformanceService.cs
- SecurityDemoService.cs
- Lab02Api.csproj
- appsettings.json
- appsettings.Development.json

Do NOT modify anything.

Look for:

- access-control concerns
- hard-coded secrets
- injection risks
- security misconfiguration
- unsafe exception handling
- vulnerable dependency indicators
- missing input validation
- sensitive information exposure

For every finding include the exact file and code evidence.

Do not invent findings.
```

Now we're moving from:

```text
one-file review
```

to:

```text
repository security review
```

---

# Part 20 — Ask AI about input validation

Your endpoints accept things like:

```text
amount
customerType
customerName
size
```

Ask:

```text
Review all API parameters in Program.cs.

Identify which parameters need validation.

For each parameter specify:

- expected datatype
- reasonable constraints
- invalid examples
- possible impact of missing validation

Do not modify the code.
```

You might receive observations such as:

```text
size
↓
Should probably not accept arbitrarily huge values

subtotal
↓
Should not be negative

customerName
↓
Should have sensible length limits

customerType
↓
Should only accept recognized values
```

This is security-related because uncontrolled or unexpected input can contribute to application abuse and insecure behavior.

---

# Part 21 — Connect Lab 6 with Lab 7

Remember your Lab 6 endpoint:

```text
/performance-test?size=...
```

Suppose someone calls:

```text
size = extremely large number
```

Your program might consume excessive CPU or memory.

Ask Cursor:

```text
Review the /performance-test endpoint from a security
perspective.

Could unrestricted size input create an availability or
resource-consumption problem?

Recommend a reasonable validation strategy.

Do not modify the code yet.
```

Notice how SDLC topics connect:

```text
LAB 6
Performance problem

        ↓

LAB 7
Could the same behavior become
a security / availability risk?
```

That's a valuable lesson.

---

# Part 22 — Create a security remediation plan

Before fixing anything:

```text
Based on the confirmed findings, create a security
remediation plan.

For each finding provide:

1. Finding
2. OWASP category
3. Current risk
4. Proposed remediation
5. Files affected
6. Testing required
7. Risk of the change
8. Priority

Do NOT change any code.
```

Your workflow is now:

```text
Security Scan
      ↓
AI Findings
      ↓
Human Validation
      ↓
Prioritization
      ↓
Remediation Plan
```

Still no code changes.

---

# Part 23 — Why don't we fix everything in Lab 7?

Because the title is:

> **Conduct an OWASP-Focused AI Security Code Review**

The purpose is primarily:

```text
FIND
UNDERSTAND
CLASSIFY
PRIORITIZE
RECOMMEND
```

not:

```text
Automatically rewrite everything
```

That makes this lab different from a remediation exercise.

---

# Part 24 — Create the security report

Create:

```text
SECURITY_REVIEW_REPORT.md
```

Ask Cursor:

```text
Create SECURITY_REVIEW_REPORT.md from the actual findings
in this project.

Use this structure:

# AI-Assisted OWASP Security Review

## 1. Scope

Files reviewed.

## 2. Review Method

Explain that this was an AI-assisted static source-code
review with human validation.

## 3. Findings Summary

Table:

ID
Finding
File
OWASP Category
Severity
Confidence
Status

## 4. Detailed Findings

For every finding include:

- Evidence
- Why it matters
- Possible impact
- Recommended remediation

## 5. Dependency Security

Summarize the vulnerability findings from Lab 6.

Do not invent package vulnerabilities.

## 6. Prioritized Remediation Plan

Critical
High
Medium
Low

## 7. Limitations

Explain what cannot be proven through source-code review
alone.

## 8. Conclusion

Do not modify application source code.
```

---

# Part 25 — Example security baseline

Your report might contain something like:

```text
SECURITY BASELINE
────────────────────────────────────────

Finding                          Risk
────────────────────────────────────────
Unsafe SQL construction          High

Hard-coded demo secret           High

Verbose exception disclosure     Medium

Input validation gaps            Medium

Known vulnerable packages        Based on
                                 Lab 6 result
```

Again:

**Do not invent vulnerabilities just to make the report look impressive.**

---

# Part 26 — Static review does not prove security

This is one of the most important lessons in Lab 7.

AI is looking at:

```text
SOURCE CODE
```

That's essentially a form of static review.

It can detect suspicious patterns.

But it cannot automatically prove:

```text
Application is 100% secure.
```

Because real security also depends on:

```text
Runtime configuration
Authentication
Authorization
Database configuration
Network configuration
Cloud configuration
Secrets
Dependencies
Deployment
User behavior
Infrastructure
```

OWASP itself positions the Top 10 as an **awareness document**, not as proof that an application is secure. ([OWASP Foundation][1])

---

# Part 27 — AI should be treated as reviewer, not judge

Your workflow should be:

```text
              SOURCE CODE
                   │
                   ▼
               AI REVIEW
                   │
         ┌─────────┼─────────┐
         ▼         ▼         ▼
      Finding    Finding    Finding
         │         │         │
         └─────────┼─────────┘
                   ▼
              HUMAN REVIEW
                   │
         ┌─────────┼─────────┐
         ▼         ▼         ▼
     Confirmed   False     Need More
                Positive    Context
                   │
                   ▼
             Prioritization
                   │
                   ▼
            Remediation Plan
```

That's much closer to a professional security-review workflow.

---

# Part 28 — Optional Git workflow

Before adding the security demo:

```powershell
git add .
git commit -m "Baseline before security review lab"
```

Create branch:

```powershell
git switch -c lab07-security-review
```

Add the security-review artifacts:

```powershell
git add .
git commit -m "Add OWASP security review exercise"
```

Then:

```powershell
git status
```

---

# Part 29 — What Lab 7 actually teaches

It is **not**:

```text
Learn hacking
```

It is:

```text
Look at ordinary developer code
             ↓
Ask security questions
             ↓
Map risks to OWASP
             ↓
Challenge AI findings
             ↓
Prioritize genuine problems
             ↓
Recommend safer implementation
```

That's exactly the mindset developers and QA engineers need.

---

# Lab 7 Completion Checklist

You should be able to demonstrate:

```text
[✓] Existing Lab02Api builds

[✓] SecurityDemoService.cs created

[✓] Fake secret used — no real credential

[✓] Unsafe SQL construction identified

[✓] Hard-coded secret identified

[✓] Exception disclosure identified

[✓] AI performs OWASP-focused review

[✓] Findings mapped to OWASP Top 10:2025

[✓] AI assigns severity

[✓] AI assigns confidence

[✓] Human challenges AI findings

[✓] False-positive / context analysis performed

[✓] Entire project reviewed

[✓] Input validation reviewed

[✓] Lab 6 dependency findings incorporated

[✓] Remediation plan created

[✓] SECURITY_REVIEW_REPORT.md created

[✓] No real attack against a database/system required
```

## The easiest way to remember Labs 4–7

```text
LAB 4
"What smells bad?"
       ↓
CODE QUALITY


LAB 5
"Can I clean it up?"
       ↓
REFACTORING


LAB 6
"Is it slow or dependent on risky packages?"
       ↓
PERFORMANCE + DEPENDENCIES


LAB 7
"Could somebody misuse this code?"
       ↓
SECURITY
       ↓
OWASP REVIEW
```

And for **Lab 7 itself**:

```text
                 LAB 7
                   │
                   ▼
             SOURCE CODE
                   │
                   ▼
           AI SECURITY REVIEW
                   │
       ┌───────────┼────────────┐
       ▼           ▼            ▼
    Injection   Secrets     Exceptions
       │           │            │
       ▼           ▼            ▼
     OWASP       OWASP         OWASP
       │           │            │
       └───────────┼────────────┘
                   ▼
             Severity + Risk
                   │
                   ▼
             Human Validation
                   │
                   ▼
            Remediation Plan
                   │
                   ▼
      SECURITY_REVIEW_REPORT.md
```

**I would teach Lab 7 exactly this way before moving into automated security scanners.** First, participants should understand *why* a line of code is risky. Later, when SonarQube or another scanner flags Injection, secrets, misconfiguration, or dependency issues, the report will actually mean something to them.

[1]: https://owasp.org/Top10/?utm_source=chatgpt.com "OWASP Top 10:2025"
[2]: https://owasp.org/Top10/2021/A03_2021-Injection/?utm_source=chatgpt.com "A03 Injection - OWASP Top 10:2021"
