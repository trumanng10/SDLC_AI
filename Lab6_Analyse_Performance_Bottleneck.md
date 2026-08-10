Truman, **Lab 6 has two completely separate exercises**:

```text
PART A — PERFORMANCE
"Why is my code slow?"

PART B — DEPENDENCY RISK
"Are the external packages my program uses outdated,
deprecated, or vulnerable?"
```

It continues directly from Labs 4–5. Your original course specifically places **Performance Optimization Techniques** and **Dependency Analysis and Risk Identification** inside the same code-quality module, so this is exactly what Lab 6 is intended to teach. 

# Lab 6 — Analyze Performance Bottlenecks and Dependency Risk with AI

## Objectives

By the end of this lab, you will be able to:

* Create deliberately slow C# code.
* Measure how long the code takes.
* Use AI to identify the likely performance bottleneck.
* Ask AI for an optimization plan before modifying code.
* Optimize the code.
* compare **before vs. after execution time**.
* List NuGet dependencies used by a .NET project.
* Identify outdated, deprecated, and vulnerable packages.
* Use AI to explain dependency risks.
* Produce a `PERFORMANCE_DEPENDENCY_REPORT.md`.

---

# Part A — Performance Bottleneck

## Step 1 — Open your existing project

Continue using your Lab 2–5 project:

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

---

# Step 2 — What is a performance bottleneck?

Imagine:

```text
User requests API
      ↓
API receives request
      ↓
Some code takes 5 seconds  ← BOTTLENECK
      ↓
Other code takes 20 ms
      ↓
Response
```

A **bottleneck** is a part of the system that significantly slows down the overall operation.

Examples could include:

```text
Slow loop
Database query
Repeated file access
Repeated API calls
Unnecessary calculations
Large memory allocations
Inefficient collection lookup
```

For this beginner lab, we'll deliberately create an inefficient collection lookup.

---

# Step 3 — Create `PerformanceService.cs`

In Cursor:

**Right-click project → New File**

Create:

```text
PerformanceService.cs
```

Paste:

```csharp
public class PerformanceService
{
    public int CountMatches(
        List<int> source,
        List<int> lookup)
    {
        int matches = 0;

        foreach (int number in source)
        {
            if (lookup.Contains(number))
            {
                matches++;
            }
        }

        return matches;
    }
}
```

Save it.

---

# Step 4 — What does this code do?

Suppose:

```text
source
1, 2, 3, 4, 5

lookup
4, 5, 6, 7
```

The method checks:

```text
Is 1 inside lookup? NO
Is 2 inside lookup? NO
Is 3 inside lookup? NO
Is 4 inside lookup? YES
Is 5 inside lookup? YES
```

Result:

```text
2 matches
```

Functionally, the code is fine.

The potential problem is this:

```csharp
foreach (int number in source)
{
    if (lookup.Contains(number))
```

For every number in `source`, the program searches through `lookup`.

With large collections, this can become inefficient.

**Don't optimize it yet.**

---

# Step 5 — Create a performance testing API

Open:

```text
Program.cs
```

Before:

```csharp
app.Run();
```

add:

```csharp
app.MapGet("/performance-test", (int size) =>
{
    var performanceService = new PerformanceService();

    var source =
        Enumerable.Range(1, size).ToList();

    var lookup =
        Enumerable.Range(size / 2, size).ToList();

    var stopwatch =
        System.Diagnostics.Stopwatch.StartNew();

    int matches =
        performanceService.CountMatches(
            source,
            lookup);

    stopwatch.Stop();

    return Results.Ok(new
    {
        size,
        matches,
        elapsedMilliseconds =
            stopwatch.ElapsedMilliseconds
    });
});
```

Microsoft's `Stopwatch` is designed for measuring elapsed time; `ElapsedMilliseconds` reports the elapsed duration in milliseconds. ([Microsoft Learn][1])

---

# Step 6 — Understand the test

We are generating two lists.

For:

```text
size = 10000
```

you'll roughly have:

```text
source:

1
2
3
...
10000
```

and:

```text
lookup:

5000
5001
5002
...
14999
```

Then we ask:

> How many numbers from `source` also appear in `lookup`?

More importantly:

> How long does the calculation take?

---

# Step 7 — Build

Run:

```powershell
dotnet build
```

Expected:

```text
Build succeeded.
```

---

# Step 8 — Run

```powershell
dotnet run
```

Suppose your API shows:

```text
Now listening on: http://localhost:5187
```

Open:

```text
http://localhost:5187/performance-test?size=10000
```

You may get something similar to:

```json
{
  "size": 10000,
  "matches": 5001,
  "elapsedMilliseconds": 47
}
```

**Your elapsed time will be different.**

CPU speed, .NET version, background processes, and other factors affect timing.

That's okay.

Record whatever your machine shows.

---

# Step 9 — Increase the workload

Try:

```text
http://localhost:5187/performance-test?size=20000
```

Then:

```text
http://localhost:5187/performance-test?size=30000
```

If `30000` takes too long, stop and use a smaller number.

Don't make your computer suffer just to prove the point.

Create a simple table:

|   Size | Matches |   Time |
| -----: | ------: | -----: |
|  5,000 |  record | record |
| 10,000 |  record | record |
| 20,000 |  record | record |

For example:

```text
PERFORMANCE BASELINE

5,000      12 ms
10,000     43 ms
20,000    168 ms
```

Again, **use your actual results**.

---

# Step 10 — Why does increasing input make it much slower?

Conceptually, the current code behaves like:

```text
Take item 1
    ↓
Search lookup list

Take item 2
    ↓
Search lookup list

Take item 3
    ↓
Search lookup list

...

Take item 20,000
    ↓
Search lookup list
```

Potentially:

```text
Many source items
       ×
Many lookup comparisons
       =
Lots of work
```

This is the performance problem we want AI to discover.

---

# Step 11 — Ask AI to analyze it

Open Cursor Chat and enter:

```text
Review PerformanceService.cs.

Do NOT modify the code.

I measured increasingly slow execution as the list size
increased.

Analyze CountMatches() for possible performance bottlenecks.

Explain in beginner-friendly language:

1. What operation is potentially expensive?
2. Why does performance worsen as the lists grow?
3. What is the approximate time complexity?
4. What data structure would be more appropriate?
5. What optimization would you recommend?

Do not change any code yet.
```

AI will probably identify:

```csharp
lookup.Contains(number)
```

as the main issue.

---

# Step 12 — Understanding the AI's "O(n²)" explanation

AI may start talking about:

```text
O(n²)
O(n)
```

Don't let the terminology confuse you.

For this lab, think of it approximately like this:

### Inefficient approach

```text
10 students
×
search 10 records

≈ lots of comparisons
```

When the data becomes:

```text
10,000
×
10,000
```

the number of potential comparisons becomes huge.

---

# Step 13 — Ask AI for alternatives

Enter:

```text
Give me three possible ways to optimize CountMatches().

Compare each solution based on:

- speed
- memory usage
- code complexity
- maintainability

Recommend the simplest solution suitable for this lab.

Do NOT modify the code.
```

AI should probably recommend something involving:

```csharp
HashSet<int>
```

---

# Step 14 — What is a `HashSet`?

Your current `List` behaves conceptually like:

```text
Looking for customer "Truman"

List:
Adam
Bob
Carol
David
Edward
...
Truman

Search one-by-one
```

A `HashSet` is designed for much faster membership lookup:

```text
Do we have Truman?

HashSet
     ↓
direct lookup
     ↓
YES
```

For this exercise, that's what we need.

---

# Step 15 — Ask AI to optimize the method

Now enter:

```text
Optimize PerformanceService.CountMatches().

Requirements:

1. Preserve the method signature.
2. Preserve the result.
3. Use HashSet if appropriate.
4. Do not add external packages.
5. Keep the implementation beginner-friendly.
6. Modify only the performance-related code.

Explain the change before implementing it.
```

A reasonable result would be:

```csharp
public class PerformanceService
{
    public int CountMatches(
        List<int> source,
        List<int> lookup)
    {
        var lookupSet =
            new HashSet<int>(lookup);

        int matches = 0;

        foreach (int number in source)
        {
            if (lookupSet.Contains(number))
            {
                matches++;
            }
        }

        return matches;
    }
}
```

---

# Step 16 — Understand what changed

### Before

```csharp
if (lookup.Contains(number))
```

### After

```csharp
var lookupSet =
    new HashSet<int>(lookup);
```

then:

```csharp
if (lookupSet.Contains(number))
```

Conceptually:

```text
BEFORE

source item
    ↓
search List
one-by-one
    ↓
repeat
    ↓
repeat


AFTER

Build lookup structure once
        ↓
    HashSet
        ↓
Fast membership check
        ↓
repeat
```

---

# Step 17 — Build again

Stop your API:

```text
Ctrl + C
```

Then:

```powershell
dotnet build
```

Expected:

```text
Build succeeded.
```

Run:

```powershell
dotnet run
```

---

# Step 18 — Repeat exactly the same performance tests

Test:

```text
http://localhost:5187/performance-test?size=5000
```

Then:

```text
http://localhost:5187/performance-test?size=10000
```

Then:

```text
http://localhost:5187/performance-test?size=20000
```

Record the results.

For example, your report might eventually look like:

|   Size |      Before |       After | Matches Same? |
| -----: | ----------: | ----------: | ------------- |
|  5,000 | your result | your result | Yes           |
| 10,000 | your result | your result | Yes           |
| 20,000 | your result | your result | Yes           |

**Do not use made-up timing numbers in your final lab report. Use what your computer actually measures.**

---

# Step 19 — The important result

You need two things:

```text
1. PERFORMANCE improves

AND

2. FUNCTIONAL RESULT stays the same
```

For example:

```text
BEFORE
matches = 5001

AFTER
matches = 5001
```

If AI made the code fast but changed:

```text
5001 → 4382
```

then the optimization is wrong.

---

# Step 20 — Ask AI to review its optimization

Use:

```text
Review the optimized PerformanceService.cs.

Do NOT modify it.

Compare the old List.Contains approach with the current
HashSet approach.

Explain:

1. Why the new implementation should scale better.
2. Additional memory it may consume.
3. Whether behavior should remain equivalent.
4. Edge cases that should be tested.
5. Situations where this optimization might not be worthwhile.
```

Notice we're teaching:

```text
Performance optimization
       ≠
Make everything fast at any cost
```

There are trade-offs.

---

# Part B — Dependency Risk

Now we move to the second half.

# Step 21 — What is a dependency?

Your own code might be:

```text
Program.cs
OrderService.cs
CustomerReportService.cs
PerformanceService.cs
```

But real software also relies on packages written by other people.

For example:

```text
Your Application
      │
      ├── NuGet Package A
      ├── NuGet Package B
      └── NuGet Package C
```

Those packages are called **dependencies**.

---

# Step 22 — Why can dependencies be risky?

Imagine:

```text
Your Code
   ↓
Package A
   ↓
Package B
   ↓
Package C
```

Suppose Package C has a known security vulnerability.

Your own code may be perfectly written, but your application can still inherit risk through that dependency.

So we examine:

```text
Outdated?
Deprecated?
Known vulnerability?
Direct dependency?
Transitive dependency?
```

---

# Step 23 — Check your .NET version

Run:

```powershell
dotnet --version
```

For example:

```text
10.0.xxx
```

This matters because Microsoft changed the preferred command syntax in .NET 10. With .NET 10+, use `dotnet package list`; .NET 9 and earlier use the older `dotnet list package` form. ([Microsoft Learn][2])

---

# Step 24 — List your packages

### If you have .NET 10

Run:

```powershell
dotnet package list
```

Microsoft documents this command as listing the NuGet package references of the project or solution. ([Microsoft Learn][2])

You might see:

```text
Project 'Lab02Api' has the following package references

Top-level Package
-----------------
...
```

Don't worry if there aren't many packages.

This is a small project.

---

# Step 25 — Direct vs transitive dependency

Imagine:

```text
YOU install Package A
       ↓
Package A needs Package B
       ↓
Package B needs Package C
```

Then:

```text
Package A = Direct dependency

Package B = Transitive dependency

Package C = Transitive dependency
```

You didn't explicitly choose B and C, but your application still relies on them.

---

# Step 26 — Show transitive dependencies

For .NET 10:

```powershell
dotnet package list --include-transitive
```

Microsoft's current CLI supports `--include-transitive` for inspecting the broader dependency graph. ([Microsoft Learn][2])

Now you'll potentially see more packages.

---

# Step 27 — Check for outdated packages

Run:

```powershell
dotnet package list --outdated
```

This asks:

> Are newer package versions available?

Microsoft supports `--outdated` on the package-list command. ([Microsoft Learn][2])

Important:

```text
OUTDATED
≠
automatically insecure
```

It simply means a newer version exists.

---

# Step 28 — Check for deprecated packages

Run:

```powershell
dotnet package list --deprecated
```

A package may be deprecated because its publisher no longer maintains it or recommends users move to another package. ([Microsoft Learn][3])

Think:

```text
Outdated:
There's a newer version.

Deprecated:
The publisher says this package/version
should no longer be the preferred choice.
```

---

# Step 29 — Check known vulnerabilities

Run:

```powershell
dotnet package list --vulnerable --include-transitive
```

Microsoft documents `--vulnerable` for listing packages with known vulnerabilities; the command can also include transitive dependencies. ([Microsoft Learn][2])

You may receive:

```text
The given project has no vulnerable packages
```

That's completely fine.

**Do not intentionally install an old vulnerable package merely to make this lab interesting.**

The purpose is learning the analysis process.

---

# Step 30 — Save dependency results

Let's give AI something concrete to analyse instead of asking it to guess.

Run:

```powershell
dotnet package list > packages.txt
```

Then:

```powershell
dotnet package list --include-transitive > dependencies.txt
```

Then:

```powershell
dotnet package list --outdated > outdated.txt
```

Then:

```powershell
dotnet package list --deprecated > deprecated.txt
```

Then:

```powershell
dotnet package list --vulnerable --include-transitive > vulnerabilities.txt
```

Your project now contains:

```text
Lab02Api
│
├── Program.cs
├── OrderService.cs
├── CustomerReportService.cs
├── PerformanceService.cs
│
├── packages.txt
├── dependencies.txt
├── outdated.txt
├── deprecated.txt
├── vulnerabilities.txt
│
└── Lab02Api.csproj
```

---

# Step 31 — Ask AI to analyze the dependency results

Now Cursor can examine those files.

Prompt:

```text
Perform a dependency-risk review of this .NET project.

Review:

- Lab02Api.csproj
- packages.txt
- dependencies.txt
- outdated.txt
- deprecated.txt
- vulnerabilities.txt

IMPORTANT:
Do not invent vulnerabilities or CVEs.

Use only findings actually present in these files.

For each dependency identify:

1. Package name
2. Installed version
3. Direct or transitive dependency
4. Outdated status
5. Deprecated status
6. Known vulnerability status
7. Risk level: Low / Medium / High
8. Recommended action

Present the results as a table.

Do not modify any packages.
```

That phrase is very important:

```text
Do not invent vulnerabilities or CVEs.
```

AI should analyze evidence rather than hallucinate package risks.

---

# Step 32 — If no vulnerabilities are detected

Suppose you get:

```text
No vulnerable packages found.
```

Ask AI:

```text
No known vulnerable NuGet packages were detected.

Explain why this does NOT mean that the application has
zero security risk.

Keep the answer focused on dependency-management risk.
```

The lesson is:

```text
No known vulnerabilities today
        ≠
Package can never have vulnerabilities
```

Dependency management is an ongoing activity.

---

# Step 33 — Don't automatically update everything

Suppose AI notices:

```text
Package X
Installed: 8.x
Latest: 10.x
```

Don't immediately execute:

```text
UPDATE EVERYTHING!
```

Instead ask:

```text
For each outdated package, classify the proposed upgrade as:

- patch
- minor
- major

Explain possible compatibility risk.

Do not update anything.
```

Major-version updates in particular can require application changes, so dependency upgrades should be reviewed and tested rather than applied blindly.

---

# Step 34 — Create the final report

Create:

```text
PERFORMANCE_DEPENDENCY_REPORT.md
```

Ask Cursor:

```text
Create PERFORMANCE_DEPENDENCY_REPORT.md
using the actual results from this lab.

Include:

# Lab 6 Performance and Dependency Risk Report

## 1. Performance Baseline

Table:
Input Size
Before Optimization
After Optimization
Improvement

## 2. Performance Bottleneck

Explain the List.Contains bottleneck.

## 3. AI Recommendation

Explain why HashSet was recommended.

## 4. Optimization Validation

Confirm whether match counts remained identical.

## 5. Dependencies

List direct and transitive dependencies.

## 6. Dependency Risk

Summarize:
- outdated packages
- deprecated packages
- known vulnerable packages

Do not invent package findings.

## 7. Recommended Actions

Prioritized recommendations.

## 8. Conclusion
```

---

# Step 35 — Your final Lab 6 project

You should now have approximately:

```text
Lab02Api
│
├── Program.cs
│
├── OrderService.cs
├── CustomerReportService.cs
├── PerformanceService.cs
│
├── CODE_QUALITY_BASELINE.md
├── PERFORMANCE_DEPENDENCY_REPORT.md
│
├── packages.txt
├── dependencies.txt
├── outdated.txt
├── deprecated.txt
├── vulnerabilities.txt
│
└── Lab02Api.csproj
```

---

# Lab 6 checklist

You should be able to demonstrate:

```text
[✓] Existing .NET application builds

[✓] PerformanceService.cs created

[✓] Slow implementation created

[✓] /performance-test API created

[✓] Stopwatch measures execution time

[✓] Performance baseline captured

[✓] AI analyzes bottleneck

[✓] AI proposes optimization

[✓] HashSet optimization implemented

[✓] Before vs After performance measured

[✓] Functional result remains identical

[✓] NuGet packages listed

[✓] Transitive dependencies inspected

[✓] Outdated dependencies checked

[✓] Deprecated dependencies checked

[✓] Known vulnerabilities checked

[✓] AI analyzes actual dependency output

[✓] PERFORMANCE_DEPENDENCY_REPORT.md created
```

# The whole Lab 6 in one picture

```text
                       LAB 6
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
       PERFORMANCE              DEPENDENCIES
             │                       │
             ▼                       ▼
      Create slow code          List packages
             │                       │
             ▼                 ┌─────┼─────┐
       Measure runtime         ▼     ▼     ▼
             │             Outdated Deprecated Vulnerable
             ▼                 │     │     │
        AI analyzes             └─────┼─────┘
        bottleneck                   ▼
             │                  AI Risk Review
             ▼                       │
       AI optimization               ▼
             │               Recommended Actions
             ▼
       Measure again
             │
             ▼
       Before vs After
             │
             └───────────┬───────────┘
                         ▼
          PERFORMANCE_DEPENDENCY_REPORT.md
```

The easiest way to remember Labs 4–6 is:

```text
LAB 4
"What is wrong with my CODE QUALITY?"
        ↓
Detect smells


LAB 5
"Can I make the CODE easier to maintain?"
        ↓
Refactor


LAB 6
"Can I make the CODE faster,
and are my PACKAGES risky?"
        ↓
Performance + Dependency Analysis
```

For your course, I would **definitely use this concrete version of Lab 6** instead of simply telling participants to “profile the application and assess dependency risk.” This version gives them a deliberately measurable bottleneck, a real optimization, and actual NuGet commands they can run.

[1]: https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch.elapsedmilliseconds?view=net-9.0&utm_source=chatgpt.com "Stopwatch.ElapsedMilliseconds Property (System.Diagnostics) | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-package-list?utm_source=chatgpt.com "dotnet package list command - .NET CLI | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/nuget/nuget-org/deprecate-packages?utm_source=chatgpt.com "Deprecating packages on nuget.org | Microsoft Learn"
