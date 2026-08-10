**Lab 8 is basically the security-focused version of the dependency part you saw in Lab 6.**

Your original course explicitly includes **“Dependency and Package Vulnerability Analysis,” “Analyzing Security Scan Results,” “Prioritizing Security Findings,” and “Generating Remediation Recommendations.”** 



```text
LAB 6
Are my packages old or risky?
        ↓
Basic dependency analysis

LAB 8
Is there a KNOWN SECURITY VULNERABILITY?
        ↓
Which package?
        ↓
Direct or transitive?
        ↓
How did it get into my project?
        ↓
How serious?
        ↓
How should we fix it?
        ↓
Did the vulnerability disappear?
```

# Lab 8 — Analyze Package and Dependency Vulnerabilities

## Objectives

By the end of this lab, you will be able to:

* Understand what a NuGet package dependency is.
* Distinguish **direct** and **transitive** dependencies.
* Scan a .NET project for known vulnerable packages.
* Interpret vulnerability results.
* Use AI to analyze package-security findings.
* Trace where a transitive vulnerable package came from.
* Create a remediation plan.
* Update a dependency safely when appropriate.
* Rescan after remediation.
* Produce a dependency-security report.

---

# Part 1 — Continue using the same project

Open PowerShell:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Check:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

Then check your .NET version:

```powershell
dotnet --version
```

If you're using **.NET 10**, the current command syntax is:

```powershell
dotnet package list
```

For .NET 9 or earlier, the older equivalent is:

```powershell
dotnet list package
```

The noun-first `dotnet package list` syntax was introduced with .NET 10. ([Microsoft Learn][1])

---

# Part 2 — First understand "dependency"

Suppose your application uses:

```text
Lab02Api
    │
    └── Package A
```

Package A is something you explicitly installed.

That's a:

```text
DIRECT / TOP-LEVEL DEPENDENCY
```

But Package A might itself need another package:

```text
Lab02Api
    │
    └── Package A
            │
            └── Package B
```

You didn't install B directly.

That's a:

```text
TRANSITIVE DEPENDENCY
```

Microsoft defines `--include-transitive` specifically for showing packages required by your top-level packages. ([Microsoft Learn][1])

This matters because:

```text
Your application
      ↓
Package A
      ↓
Package B ← vulnerable
```

Even if **your code never explicitly installed Package B**, it can still become part of your application's dependency graph.

---

# Part 3 — See your direct packages

Run:

```powershell
dotnet package list
```

You may see something resembling:

```text
Project 'Lab02Api' has the following package references

Top-level Package          Requested     Resolved
--------------------------------------------------
Microsoft.AspNetCore...    10.x.x        10.x.x
```

Your output will depend on your project.

**Do not worry if the list is short.**

Our Lab02Api is intentionally simple.

---

# Part 4 — Show all dependencies

Now run:

```powershell
dotnet package list --include-transitive
```

This adds the transitive packages to the output. ([Microsoft Learn][1])

Conceptually you may get:

```text
Top-level Package
-----------------
Package A
Package C


Transitive Package
-------------------
Package B
Package D
Package E
```

Now you can see the real dependency chain more clearly.

---

# Part 5 — Save the dependency baseline

Run:

```powershell
dotnet package list --include-transitive > dependency-tree.txt
```

Now you should have:

```text
Lab02Api
│
├── Program.cs
├── OrderService.cs
├── CustomerReportService.cs
├── PerformanceService.cs
├── SecurityDemoService.cs
│
├── dependency-tree.txt
│
└── Lab02Api.csproj
```

This becomes the dependency baseline.

---

# Part 6 — Scan specifically for vulnerable packages

Now run:

```powershell
dotnet package list --vulnerable --include-transitive
```

The current .NET CLI supports both `--vulnerable` and `--include-transitive`. ([Microsoft Learn][1])

This is the **main Lab 8 command**.

Think:

```text
dotnet package list
        ↓
What packages do I have?


dotnet package list --vulnerable
        ↓
Which packages have
known vulnerability information?


--include-transitive
        ↓
Check packages indirectly
brought in too
```

---

# Part 7 — You may get "No vulnerable packages"

You might see something like:

```text
The given project has no vulnerable packages
given the current sources.
```

That is completely valid.

Your Lab 8 result becomes:

```text
Known vulnerable packages detected: 0
```

Do **not** deliberately install an old vulnerable library just so the lab produces a red result.

The purpose of Lab 8 is learning the security workflow.

---

# Part 8 — Save the scan result

Run:

```powershell
dotnet package list --vulnerable --include-transitive > vulnerabilities-before.txt
```

You now have:

```text
dependency-tree.txt
vulnerabilities-before.txt
```

---

# Part 9 — Let NuGet Audit perform another check

Run:

```powershell
dotnet restore
```

Modern NuGet performs security auditing during restore. For projects targeting **.NET 10 or later, transitive dependencies are audited by default**; .NET 9 and earlier default to auditing direct dependencies unless configured otherwise. ([Microsoft Learn][2])

So `dotnet restore` is no longer just:

```text
Download packages
```

It also helps surface known package-vulnerability warnings.

Conceptually:

```text
dotnet restore
      │
      ├── Resolve packages
      ├── Download packages
      │
      └── NuGet Audit
              ↓
        Known vulnerability?
```

---

# Part 10 — If you see an NU19xx warning

You could potentially see something like:

```text
warning NU19xx:
Package 'SomePackage' ...
has a known vulnerability ...
```

Don't panic.

Do this:

```text
1. Record package name
2. Record installed version
3. Record severity
4. Record advisory information
5. Determine direct/transitive
6. Investigate remediation
```

Microsoft's NuGet guidance recommends using the advisory information to determine affected/fixed package versions rather than guessing. ([Microsoft Learn][3])

---

# Part 11 — Save the audit

In PowerShell:

```powershell
dotnet restore 2>&1 | Tee-Object restore-audit.txt
```

Now you have:

```text
dependency-tree.txt
vulnerabilities-before.txt
restore-audit.txt
```

---

# Part 12 — Ask Cursor AI to analyze the results

Now open Cursor Chat.

Use this prompt:

```text
You are performing a NuGet dependency security review.

Review:

- Lab02Api.csproj
- dependency-tree.txt
- vulnerabilities-before.txt
- restore-audit.txt

IMPORTANT:

Do not invent vulnerabilities.
Do not invent CVEs or advisory IDs.
Do not guess package versions.

Use only evidence contained in these files.

For each package security finding identify:

1. Package name
2. Installed version
3. Direct or transitive dependency
4. Vulnerability evidence
5. Severity if available
6. Potential impact
7. Recommended next investigation
8. Confidence level

Return the results as a table.

Do not modify any packages yet.
```

That last part is important:

```text
SCAN FIRST
FIX LATER
```

---

# Part 13 — If AI says "no vulnerabilities"

That's fine.

Ask:

```text
The NuGet vulnerability scan reports no known vulnerable
packages.

Explain what this result means.

Also explain what it does NOT prove.

Do not invent additional vulnerabilities.
```

A reasonable explanation is:

```text
SCAN RESULT

Known vulnerable dependencies
detected by configured sources:

0
```

But:

```text
0 findings
      ≠
Application is 100% secure
```

Because Lab 7 already showed that your own application code can still contain insecure patterns even if its packages are clean.

---

# Part 14 — If a vulnerable package IS found

Suppose your real output says:

```text
PackageX
Version 1.2.3
High vulnerability
```

**Do not copy this example as a real finding.**

Use whatever your actual scan reports.

Your next question is:

```text
Is PackageX direct
or
transitive?
```

---

# Part 15 — Direct vulnerability

A direct dependency looks like:

```text
Lab02Api
   │
   └── PackageX
```

and probably appears directly in:

```text
Lab02Api.csproj
```

Something like:

```xml
<PackageReference
    Include="PackageX"
    Version="1.2.3" />
```

Conceptually the fix is straightforward:

```text
Your Project
      ↓
vulnerable PackageX
      ↓
upgrade PackageX
```

But **don't blindly select "latest."**

You need:

```text
Safe version
    +
Compatible version
```

not merely:

```text
Newest version
```

---

# Part 16 — Transitive vulnerability

This is more interesting.

Suppose:

```text
Lab02Api
    │
    └── PackageA
            │
            └── PackageB
                    │
                    └── PackageX ← vulnerable
```

You never installed PackageX.

So:

```text
WHY THE HELL IS
PACKAGEX HERE?
```

That's where the next command helps.

---

# Part 17 — Use `dotnet nuget why`

If a real scan identifies:

```text
PackageX
```

run:

```powershell
dotnet nuget why Lab02Api.csproj PackageX
```

`dotnet nuget why` shows the dependency graph explaining **why a particular package is present in the project**. ([Microsoft Learn][4])

You might see conceptually:

```text
Lab02Api
   │
   └── PackageA
          │
          └── PackageB
                 │
                 └── PackageX
```

Now you know:

```text
PackageX wasn't directly installed.

PackageA ultimately pulled it in.
```

This is one of the most useful lessons in Lab 8.

---

# Part 18 — Ask AI to explain the dependency chain

Give Cursor:

```text
A vulnerable transitive package has been identified.

Analyze the dependency tree.

Explain:

1. Which top-level package caused it to be included.
2. The complete dependency path.
3. Whether upgrading the top-level package might resolve it.
4. Other remediation possibilities.
5. Compatibility risks that should be tested.

Do NOT modify packages.
Do NOT guess a safe version.
```

Microsoft specifically recommends using `dotnet nuget why` to trace vulnerable transitive dependencies and considering upgrading the top-level dependency that brings them in. ([Microsoft Learn][2])

---

# Part 19 — Check whether packages are outdated

Now run:

```powershell
dotnet package list --outdated
```

This tells you whether newer stable package versions are available. ([Microsoft Learn][1])

You may see:

```text
Package         Resolved     Latest

PackageA        4.1.0        4.5.0
```

But remember:

```text
OUTDATED
       ≠
VULNERABLE
```

A package can be:

```text
Old but not known vulnerable
```

or:

```text
Current but later discovered vulnerable
```

These are different concepts.

---

# Part 20 — Check deprecated packages too

Run:

```powershell
dotnet package list --deprecated
```

Now we have three separate questions:

```text
--outdated
    ↓
Is there a newer version?


--deprecated
    ↓
Is this package/version
no longer recommended?


--vulnerable
    ↓
Is there known
security vulnerability data?
```

Don't mix them together.

---

# Part 21 — Build a risk table

Ask Cursor:

```text
Using only the actual package scan results,
create a dependency risk matrix.

Columns:

Package
Installed Version
Dependency Type
Outdated?
Deprecated?
Known Vulnerability?
Severity
Recommended Action
Priority

Do not invent missing information.
```

For example, your result might be:

| Package   | Type       | Outdated | Vulnerable | Priority |
| --------- | ---------- | -------: | ---------: | -------- |
| Package A | Direct     |      Yes |         No | Low      |
| Package B | Transitive |       No |         No | Low      |
| Package X | Transitive |      Yes |       High | High     |

Again, **the names above are examples only**.

Your report must contain your actual results.

---

# Part 22 — How do we prioritize?

A simple beginner-friendly approach:

```text
CRITICAL/HIGH vulnerability
        ↓
Investigate first


MODERATE vulnerability
        ↓
Plan remediation


LOW vulnerability
        ↓
Assess exposure and schedule


OUTDATED only
        ↓
Maintenance review
```

But severity alone isn't everything.

Also ask:

```text
Is the vulnerable feature used?

Is the package exposed to untrusted input?

Is it reachable?

Is there a fixed compatible version?

Will upgrading break our application?
```

That's why security remediation still requires human judgment.

---

# Part 23 — Ask AI for a remediation plan

Use:

```text
Create a remediation plan using ONLY confirmed
dependency findings.

For each vulnerable dependency provide:

1. Package
2. Current version
3. Direct/transitive
4. Severity
5. Dependency path
6. Recommended remediation
7. Compatibility risk
8. Testing required
9. Priority

Do not modify packages.

Do not invent a fixed version unless the scan/advisory
explicitly identifies one.
```

That's much safer than:

```text
AI, update all my packages!
```

---

# Part 24 — If a real vulnerable direct package needs updating

Suppose your **actual advisory** says a certain later version is no longer affected.

Then the conceptual process is:

```text
Vulnerable Package
       ↓
Find fixed compatible version
       ↓
Update ONE package
       ↓
Restore
       ↓
Build
       ↓
Test
       ↓
Rescan
```

Microsoft recommends reviewing the vulnerability advisory to determine which versions are fixed. ([Microsoft Learn][3])

Do not simply take whatever AI remembers as the "latest version."

---

# Part 25 — After any package change: restore

Run:

```powershell
dotnet restore
```

Then:

```powershell
dotnet build
```

You want:

```text
Restore succeeded

Build succeeded
```

---

# Part 26 — Run your functional checks

Because you've changed dependencies, test the application again:

```powershell
dotnet run
```

Test the APIs created in previous labs, such as:

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

The application should still behave correctly.

---

# Part 27 — Run unit tests if you have them

Later labs will add proper unit tests.

If your project already has tests:

```powershell
dotnet test
```

You want:

```text
Passed
```

This is critical.

A security update that breaks the application isn't a complete remediation.

---

# Part 28 — RESCAN

This step is essential.

Run again:

```powershell
dotnet package list --vulnerable --include-transitive
```

Save it:

```powershell
dotnet package list --vulnerable --include-transitive > vulnerabilities-after.txt
```

Now compare:

```text
BEFORE
vulnerabilities-before.txt

        VS

AFTER
vulnerabilities-after.txt
```

---

# Part 29 — What does successful remediation look like?

Suppose the real scan initially showed:

```text
Before

Known vulnerable packages: 1
High: 1
```

After remediation:

```text
After

Known vulnerable packages: 0
```

Then:

```text
Security issue identified
        ↓
Dependency traced
        ↓
Remediation selected
        ↓
Package changed
        ↓
Application tested
        ↓
Rescan clean

        ✓
```

That's the complete security workflow.

---

# Part 30 — Ask AI to validate the remediation

Use:

```text
Compare:

- vulnerabilities-before.txt
- vulnerabilities-after.txt
- restore-audit.txt
- Lab02Api.csproj

Determine whether the confirmed dependency vulnerability
was successfully remediated.

Do not assume success merely because a package version
changed.

Explain the evidence.

Also identify any remaining dependency risks.
```

Excellent AI prompt.

You're forcing AI to use **evidence**.

---

# Part 31 — Create the final report

Create:

```text
DEPENDENCY_SECURITY_REPORT.md
```

Ask Cursor:

```text
Create DEPENDENCY_SECURITY_REPORT.md.

Use ONLY the actual outputs generated during Lab 8.

Structure:

# Lab 8 — Package and Dependency Vulnerability Analysis

## 1. Project

Lab02Api

## 2. Dependency Inventory

Summarize direct and transitive dependencies.

## 3. Vulnerability Scan

Summarize actual vulnerable-package findings.

## 4. NuGet Audit

Summarize dotnet restore audit findings.

## 5. Dependency Path Analysis

For transitive vulnerabilities, document how the package
entered the project.

## 6. Risk Assessment

Package
Version
Direct/Transitive
Severity
Risk
Priority

## 7. Remediation Plan

Describe recommended actions.

## 8. Remediation Performed

Document package changes, if any.

## 9. Validation

Document:
dotnet restore
dotnet build
dotnet test
functional testing

## 10. Rescan Result

Compare before and after.

## 11. Conclusion

Do not invent package names, vulnerabilities,
versions, CVEs, or advisories.
```

---

# Your Lab 8 project should now look like this

```text
Lab02Api
│
├── Program.cs
├── OrderService.cs
├── CustomerReportService.cs
├── PerformanceService.cs
├── SecurityDemoService.cs
│
├── dependency-tree.txt
├── vulnerabilities-before.txt
├── vulnerabilities-after.txt
├── restore-audit.txt
│
├── DEPENDENCY_SECURITY_REPORT.md
│
└── Lab02Api.csproj
```

# Lab 8 Completion Checklist

You should be able to show:

```text
[✓] Project builds

[✓] Direct dependencies listed

[✓] Transitive dependencies listed

[✓] Dependency baseline saved

[✓] Vulnerability scan executed

[✓] NuGet Audit executed

[✓] AI analyzes actual output

[✓] No vulnerabilities invented

[✓] Direct vs transitive identified

[✓] dotnet nuget why used where necessary

[✓] Dependency path understood

[✓] Risk severity assessed

[✓] Remediation plan produced

[✓] Package updated only if appropriate

[✓] Project restored

[✓] Project rebuilt

[✓] Application retested

[✓] Vulnerability scan rerun

[✓] Before vs after compared

[✓] DEPENDENCY_SECURITY_REPORT.md created
```

## The one diagram to remember

```text
                    LAB 8
                       │
                       ▼
                 Lab02Api
                       │
                       ▼
              List Dependencies
                       │
              ┌────────┴────────┐
              ▼                 ▼
           Direct          Transitive
              │                 │
              └────────┬────────┘
                       ▼
             Vulnerability Scan
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        No Finding           Vulnerability
             │                   │
             ▼                   ▼
       Record Result        Assess Severity
                                 │
                                 ▼
                       Direct or Transitive?
                         /               \
                        /                 \
                   Direct              Transitive
                     │                     │
                     │              dotnet nuget why
                     │                     │
                     └──────────┬──────────┘
                                ▼
                         Remediation Plan
                                │
                                ▼
                          Package Change
                                │
                      ┌─────────┼─────────┐
                      ▼         ▼         ▼
                   Restore    Build      Test
                      │         │         │
                      └─────────┼─────────┘
                                ▼
                              Rescan
                                │
                                ▼
                     Before vs After
                                │
                                ▼
              DEPENDENCY_SECURITY_REPORT.md
```

The easiest way to distinguish **Lab 7 and Lab 8** is:

```text
LAB 7
Look inside YOUR C# CODE
        ↓
Injection
Secrets
Error handling
Input validation
OWASP risks


LAB 8
Look inside the SOFTWARE SUPPLY CHAIN
        ↓
NuGet packages
Direct dependencies
Transitive dependencies
Known vulnerabilities
Dependency paths
Package remediation
```

That distinction is important: **your source code can be secure while a third-party dependency is vulnerable, and your packages can be clean while your own source code is insecure.** Lab 7 teaches the first side; Lab 8 teaches the second. ([Microsoft Learn][2])

[1]: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-package-list?utm_source=chatgpt.com "dotnet package list command - .NET CLI | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/dotnet/core/compatibility/sdk/10.0/nugetaudit-transitive-packages?utm_source=chatgpt.com "Breaking change: 'dotnet restore' audits transitive packages - .NET | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/nuget/reference/errors-and-warnings/nu1901-nu1904?utm_source=chatgpt.com "NuGet Warnings NU1901, NU1902, NU1903, NU1904 | Microsoft Learn"
[4]: https://learn.microsoft.com/is-is/dotnet/core/tools/dotnet-nuget-why?utm_source=chatgpt.com "dotnet nuget why command - .NET CLI | Microsoft Learn"
