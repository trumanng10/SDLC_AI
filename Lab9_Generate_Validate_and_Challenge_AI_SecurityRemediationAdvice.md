Lab 7 taught you how AI can **find security problems**. Lab 8 taught you how AI can **analyze vulnerable dependencies**. Lab 9 now teaches you to ask AI for a fix, then **challenge that fix before accepting it**.

This directly follows your course requirements around generating remediation recommendations and **validating/verifying AI security recommendations**. 

# Lab 9 — Generate, Validate and Challenge AI Security Remediation Advice

## Objectives

By the end of this lab, you will be able to:

* Ask AI to propose security remediation.
* Separate **finding** from **recommended fix**.
* Review whether AI's fix actually addresses the root cause.
* Detect over-engineered or unsafe AI recommendations.
* Challenge assumptions made by AI.
* Build and test security fixes.
* Compare before and after behavior.
* Produce a validated remediation report.

---

# Part 1 — Understand the Lab 9 workflow

The most important concept is:

```text
Security Finding
       ↓
AI Recommendation
       ↓
DO NOT ACCEPT YET
       ↓
Challenge AI
       ↓
Validate Fix
       ↓
Build
       ↓
Test
       ↓
Review Again
       ↓
Accept / Reject
```

Lab 9 teaches:

> **AI security advice is a recommendation, not a security guarantee.**

---

# Part 2 — Continue from Lab 7

Use the same project:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Build it:

```powershell
dotnet build
```

Expected:

```text
Build succeeded.
```

Open:

```text
SecurityDemoService.cs
```

Your Lab 7 code should look approximately like this:

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

Remember, these are intentionally insecure examples.

---

# Part 3 — Our three security findings

From Lab 7:

```text
Finding 1
Unsafe SQL string construction

Finding 2
Hard-coded secret

Finding 3
Detailed exception disclosure
```

Lab 9 asks:

```text
"What should we DO about them?"
```

But more importantly:

```text
"Is AI's proposed solution actually safe?"
```

---

# Part 4 — Ask AI for remediation advice

Open Cursor Chat.

Use this prompt:

```text
Review SecurityDemoService.cs.

We previously identified these potential security issues:

1. Unsafe SQL query construction
2. Hard-coded API key
3. Returning exception.ToString()

Do NOT modify the code yet.

For each finding provide:

1. Root cause
2. Recommended remediation
3. Alternative remediation
4. Security benefit
5. Implementation complexity
6. Compatibility risk
7. Testing required
8. Residual risk after remediation

Clearly identify any assumptions you are making.
```

That last line is important:

```text
Clearly identify assumptions.
```

---

# Part 5 — Why ask for assumptions?

AI may say:

> "Use Entity Framework Core."

But your project doesn't use Entity Framework.

Or:

> "Store the secret in Azure Key Vault."

But your environment may not use Azure.

Or:

> "Use authentication middleware."

But maybe the endpoint isn't supposed to be public in the first place.

So always ask:

```text
What are you assuming?
```

---

# Part 6 — Challenge Recommendation #1: SQL Injection

AI will probably recommend:

```text
Use parameterized queries
```

That's generally sensible.

But don't stop there.

Ask:

```text
Challenge your recommendation for the SQL injection finding.

You recommended parameterized queries.

Answer:

1. Does SecurityDemoService currently execute SQL?
2. Or does it only construct a string?
3. What additional code/context would you need before implementing
   a real fix?
4. Which database provider would matter?
5. Should we introduce a database library purely for this lab?

Do not modify the project.
```

Now AI should recognize:

```text
Current demo
    ↓
Builds SQL text

But
    ↓
does NOT actually execute it
```

Therefore:

```text
Finding:
Unsafe pattern

But remediation implementation:
Needs database context
```

That's exactly the type of nuance Lab 9 is designed to teach.

---

# Part 7 — Wrong AI advice example

Suppose AI suggests:

```csharp
customerName =
    customerName.Replace("'", "''");
```

and says:

> "This fixes SQL injection."

You should challenge it.

Ask:

```text
Is manually escaping apostrophes a reliable substitute
for parameterized SQL queries?

Explain why or why not.

Do not change the code.
```

This is important because a simplistic AI fix can create **false confidence**.

The safer design principle is:

```text
User Input
     ↓
Parameter

NOT

User Input
     ↓
Manually manipulate text
     ↓
Concatenate into SQL
```

---

# Part 8 — Challenge Recommendation #2: Hard-coded secret

AI may recommend:

```text
Move the API key to appsettings.json
```

Looks good?

Not necessarily.

Ask:

```text
You recommended moving the API key to appsettings.json.

Challenge this advice.

Explain:

1. Is appsettings.json automatically a secure secret store?
2. Could appsettings.json be committed to Git?
3. What is appropriate for local development?
4. What is appropriate for production?
5. What should never be committed?
```

Now you are **challenging the remediation**, not just the original vulnerability.

---

# Part 9 — Understand configuration vs secret storage

This:

```json
{
  "ApiKey": "REAL-PRODUCTION-SECRET"
}
```

inside a Git-tracked:

```text
appsettings.json
```

may still expose the secret.

So:

```text
Move secret from C# file
        ↓
appsettings.json
```

does **not automatically mean secure**.

Better conceptually:

```text
Development
    ↓
User Secrets / environment variable

Production
    ↓
Managed secret store /
environment-specific secure configuration
```

The exact production system depends on the organization's infrastructure.

---

# Part 10 — Ask AI for a beginner-safe secret remediation

Prompt:

```text
For this beginner lab, recommend the simplest remediation
for the hard-coded fake API key.

Constraints:

- No cloud account
- No Azure Key Vault
- No AWS Secrets Manager
- No external package
- Local .NET development
- Do not put the secret into source control

Explain the approach before modifying anything.
```

For a beginner .NET lab, AI may recommend:

```text
.NET User Secrets
```

or an environment variable.

That's much more appropriate than adding cloud infrastructure to a simple lab.

---

# Part 11 — Remove the hard-coded secret

Instead of:

```csharp
private const string ApiKey =
    "DEMO-SECRET-12345";
```

we can retrieve it through configuration.

One simple approach is to change the service to receive configuration.

For example:

```csharp
public class SecurityDemoService
{
    private readonly IConfiguration _configuration;

    public SecurityDemoService(
        IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GetApiKey()
    {
        return _configuration["DemoApiKey"]
            ?? "Not configured";
    }
}
```

But before blindly applying this, ask Cursor:

```text
Review this proposed IConfiguration solution.

Explain:

1. Whether it removes the secret from source code.
2. Whether it automatically secures the secret.
3. Where DemoApiKey should come from in local development.
4. What additional setup Program.cs requires.

Do not modify anything yet.
```

---

# Part 12 — Configure .NET User Secrets

From your project directory:

```powershell
dotnet user-secrets init
```

Then add a **fake training secret**:

```powershell
dotnet user-secrets set "DemoApiKey" "LAB-DEMO-KEY-67890"
```

Check:

```powershell
dotnet user-secrets list
```

You should see something similar to:

```text
DemoApiKey = LAB-DEMO-KEY-67890
```

Again, this is a **fake lab key**.

Never use real production secrets in classroom exercises.

---

# Part 13 — Register the service correctly

If `SecurityDemoService` now requires `IConfiguration`, you should not create it manually using:

```csharp
new SecurityDemoService()
```

Instead add before:

```csharp
var app = builder.Build();
```

something like:

```csharp
builder.Services.AddSingleton<SecurityDemoService>();
```

Then use dependency injection.

Ask Cursor:

```text
Update the application to register SecurityDemoService
through ASP.NET Core dependency injection.

Requirements:

- Keep the implementation minimal.
- Do not introduce external packages.
- Explain each change first.
```

---

# Part 14 — Build after remediation

Run:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

If it fails, ask AI:

```text
The build failed after the security remediation.

Do not rewrite the entire project.

Explain the root cause of the build error and suggest
the smallest correction.
```

---

# Part 15 — Challenge Recommendation #3: Exception handling

The current insecure code is:

```csharp
public string GetDetailedError(
    Exception exception)
{
    return exception.ToString();
}
```

AI might recommend:

```csharp
return "Something went wrong.";
```

That's better for the external user.

But now challenge it:

```text
You recommended replacing exception.ToString()
with a generic message.

Challenge this recommendation.

If we only return a generic message:

1. How will developers troubleshoot the problem?
2. Where should detailed information go?
3. What should be logged?
4. What should NOT be exposed to the API caller?
5. Could logging itself leak secrets?
```

Excellent security question.

---

# Part 16 — Better error-handling model

Think of it as two outputs:

```text
                EXCEPTION
                    │
           ┌────────┴────────┐
           ▼                 ▼
    INTERNAL SYSTEM      EXTERNAL USER
           │                 │
    Detailed logging      Generic message
           │                 │
    Stack trace etc.      No internals
```

But even internal logs must avoid sensitive values such as:

```text
Passwords
Tokens
API keys
Personal data
```

So:

```text
Log everything
```

is also poor advice.

---

# Part 17 — Ask AI to produce a safer implementation

Use:

```text
Refactor GetDetailedError so that external callers receive
a generic message.

Do NOT implement a complex logging system yet.

Explain how a production system should log diagnostic details
without returning them to the client.

Keep this lab implementation minimal.
```

A beginner version could be:

```csharp
public string GetPublicErrorMessage()
{
    return "An unexpected error occurred.";
}
```

The point is not the exact method.

The point is separating:

```text
Internal diagnostic information

from

External error information
```

---

# Part 18 — Now challenge ALL remediation advice

This is the most important prompt in Lab 9.

Use:

```text
Act as an independent application security reviewer.

Review the security remediation recommendations proposed
so far.

Do NOT assume the previous AI advice is correct.

For each recommendation identify:

1. What it fixes
2. What it does NOT fix
3. New risks it could introduce
4. Assumptions it depends on
5. Whether it is under-engineered
6. Whether it is over-engineered
7. What evidence is required before approval

Classify each recommendation:

ACCEPT
ACCEPT WITH CHANGES
REJECT
NEEDS MORE CONTEXT
```

This teaches AI-versus-AI review.

---

# Part 19 — AI Developer vs AI Security Reviewer

The workflow becomes:

```text
AI Developer
     ↓
"I recommend this fix."

        ↓

AI Security Reviewer
     ↓
"Is that actually safe?"

        ↓

Human
     ↓
decides
```

This is much stronger than:

```text
Ask AI once
    ↓
Copy code
    ↓
Done
```

---

# Part 20 — Check for over-engineering

Ask:

```text
Review all proposed security fixes specifically for
over-engineering.

This is a beginner .NET Web API lab.

Reject solutions that unnecessarily introduce:

- cloud infrastructure
- external databases
- identity platforms
- unnecessary NuGet packages
- complicated design patterns

Prefer the smallest secure improvement that teaches
the principle correctly.
```

This is useful because AI often produces technically sophisticated solutions that are unsuitable for a classroom exercise.

---

# Part 21 — Check for under-engineering

Now the opposite:

```text
Review the proposed fixes for under-engineering.

Identify any solution that appears simple but provides
false security.

Examples to consider:

- manual SQL escaping
- hiding a key in another source file
- putting secrets in Git-tracked config
- swallowing exceptions
- removing all error logging

Explain why.
```

Now you are actively testing whether AI security advice is superficial.

---

# Part 22 — Build a remediation decision table

Ask Cursor:

```text
Create a security remediation decision table.

Columns:

Finding
Original AI Recommendation
Challenge
Decision
Reason
Implementation Action
Validation Required

Use only the findings from SecurityDemoService.cs.
```

Your table might resemble:

| Finding            | Recommendation         | Decision                                   |
| ------------------ | ---------------------- | ------------------------------------------ |
| SQL construction   | Parameterized query    | Needs DB context                           |
| Hard-coded secret  | User Secrets           | Accept for local lab                       |
| Exception exposure | Generic external error | Accept with internal logging consideration |

That is much more professional than just listing "fixes."

---

# Part 23 — Validate source code after remediation

Ask Cursor:

```text
Review the current SecurityDemoService.cs after remediation.

Determine whether:

1. The fake secret remains hard-coded anywhere.
2. Unsafe SQL concatenation still exists.
3. Detailed exceptions are returned externally.
4. Any new security problems were introduced.

Provide exact file/code evidence.

Do not modify anything.
```

---

# Part 24 — Search the project for the old fake secret

In PowerShell:

```powershell
Select-String -Path *.cs -Pattern "DEMO-SECRET-12345"
```

Ideally:

```text
No matches
```

You can also search the whole current folder:

```powershell
Get-ChildItem -Recurse -File |
    Select-String "DEMO-SECRET-12345"
```

If it appears in an old report file, that's different from executable source, but document it.

---

# Part 25 — Build the project again

Run:

```powershell
dotnet build
```

Expected:

```text
Build succeeded.
```

---

# Part 26 — Run the application

```powershell
dotnet run
```

Retest your existing APIs:

```text
/status
```

```text
/price
```

```text
/customer-report
```

```text
/performance-test
```

The security changes should not unexpectedly break unrelated behavior.

---

# Part 27 — Rescan dependencies too

Because Lab 8 focused on package risk, run:

```powershell
dotnet package list --vulnerable --include-transitive
```

Why?

Because remediation should consider both:

```text
YOUR CODE
+
YOUR DEPENDENCIES
```

---

# Part 28 — Ask AI for residual risk

This is another excellent prompt:

```text
After the current remediations, identify residual security risk.

For each original finding explain:

- Fully resolved
- Partially resolved
- Not resolved
- Needs runtime validation

Explain why.

Do not assume source-code changes prove complete security.
```

For example:

```text
SQL Injection pattern
↓
Could still require validation against actual database code.

Secrets
↓
Removed from source, but production secret management
still requires infrastructure decisions.

Exception handling
↓
External disclosure reduced, but production logging
still needs design.
```

That's realistic.

---

# Part 29 — Create `SECURITY_REMEDIATION_VALIDATION.md`

Create a new file:

```text
SECURITY_REMEDIATION_VALIDATION.md
```

Ask Cursor:

```text
Create SECURITY_REMEDIATION_VALIDATION.md.

Use the actual findings and remediation work completed
during Lab 9.

Structure:

# Lab 9 — AI Security Remediation Validation

## 1. Original Findings

List confirmed Lab 7 findings.

## 2. AI Recommendations

Record the proposed fixes.

## 3. Challenge Review

For each recommendation document:
- assumptions
- weaknesses
- alternatives

## 4. Decision

Classify each:
- ACCEPT
- ACCEPT WITH CHANGES
- REJECT
- NEEDS MORE CONTEXT

## 5. Remediation Implemented

Describe actual changes.

## 6. Validation

Record:
- dotnet build
- functional testing
- source-code review
- dependency scan

## 7. Residual Risk

Document what remains unresolved.

## 8. Lessons Learned

Explain why AI-generated security advice requires
human validation.

Do not invent results.
```

---

# Part 30 — What Lab 9 actually teaches

This lab is **not primarily about writing more C#**.

It teaches this:

```text
             SECURITY FINDING
                    │
                    ▼
             AI RECOMMENDATION
                    │
                    ▼
            "Sounds reasonable"
                    │
                    ▼
                 STOP
                    │
                    ▼
             CHALLENGE THE AI
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   Assumptions   New Risks   Alternatives
       │            │            │
       └────────────┼────────────┘
                    ▼
             HUMAN DECISION
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
      ACCEPT                 REJECT
         │
         ▼
     Implement
         │
         ▼
       Build
         │
         ▼
        Test
         │
         ▼
       Rescan
         │
         ▼
    Residual Risk
```

# Lab 7 vs Lab 8 vs Lab 9

The easiest way to remember:

```text
LAB 7
"What security problems are in MY CODE?"
        ↓
OWASP review


LAB 8
"What security problems are in MY PACKAGES?"
        ↓
Dependency vulnerability review


LAB 9
"Is the AI's proposed SECURITY FIX actually correct?"
        ↓
Challenge + Validate + Approve
```

## Lab 9 Completion Checklist

You should be able to demonstrate:

```text
[✓] Lab 7 findings reused

[✓] AI generated remediation advice

[✓] AI assumptions identified

[✓] SQL remediation challenged

[✓] Secret remediation challenged

[✓] Error-handling remediation challenged

[✓] Over-engineering checked

[✓] Under-engineering checked

[✓] Recommendations classified:
    Accept / Reject / Needs Context

[✓] Appropriate fixes implemented

[✓] Fake secret removed from source

[✓] dotnet build succeeds

[✓] Existing APIs retested

[✓] Dependency vulnerability scan rerun

[✓] Residual risk identified

[✓] SECURITY_REMEDIATION_VALIDATION.md created
```

The most important takeaway from **Lab 9** is this:

```text
AI Finding
≠ Guaranteed vulnerability

AI Recommendation
≠ Guaranteed solution

AI-generated secure code
≠ Proven secure application

AI + Evidence + Testing + Human Review
= Much better security decision
```

For your AI SDLC course, **Lab 9 is actually one of the most important labs**, because it prevents participants from developing the dangerous habit of copying AI-generated security fixes without understanding whether those fixes actually address the risk.
