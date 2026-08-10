**Lab 4 is much easier than it sounds**. You are not installing SonarQube yet. The purpose is to take the project from Labs 2–3, measure its **current code-quality condition**, deliberately examine some imperfect code, and use AI to identify **code smells** and technical debt.

This fits Module 2 , which specifically covers maintainable/scalable code, code smells, technical debt, AI-assisted source-code analysis, refactoring, readability, and improvement recommendations. 

# Lab 4 — Build a Code Quality Baseline and Detect Code Smells

## Objectives

By the end of this lab, you will be able to:

* Explain what a **code quality baseline** is.
* Explain what a **code smell** is.
* Capture the current condition of a .NET project before improvements.
* Use AI to review C# source code.
* Identify common code smells.
* Rank findings by severity.
* Produce a simple code-quality report.
* **Not refactor yet** — Lab 4 is mainly detection and assessment.

---

# Part 1 — What does "Code Quality Baseline" mean?

A baseline simply means:

> **What does the code look like before we improve it?**

For example:

```text
Before Improvement
────────────────────────────
Build errors             0
Build warnings           2
Large methods            1
Duplicated logic         3
Magic numbers            5
Hard-coded strings       4
Deep nesting             2
Poor naming              3
```

This becomes your **baseline**.

Later, after refactoring, you can compare:

```text
                   BEFORE    AFTER
───────────────────────────────────
Build warnings        2        0
Large methods         1        0
Duplicate logic       3        0
Magic numbers         5        0
Deep nesting          2        0
```

That's all "baseline" means.

---

# Part 2 — What is a Code Smell?

A code smell does **not necessarily mean the program is broken**.

For example:

```csharp
if (customerType == "VIP")
```

This works.

But `"VIP"` appears directly in the code.

If it is repeated everywhere:

```csharp
if (customerType == "VIP")
...
if (customerType == "VIP")
...
if (customerType == "VIP")
```

that's harder to maintain.

This is a potential **code smell**.

Think of it this way:

```text
BUG
↓
Program behaves incorrectly.

CODE SMELL
↓
Program may work,
but code is difficult to maintain,
understand, test, or modify.
```

---

# Part 3 — Continue from Lab 3

Use the project from Lab 3:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Check the files:

```powershell
dir
```

You should have something like:

```text
Lab02Api
│
├── Program.cs
├── OrderService.cs
├── Lab02Api.csproj
├── appsettings.json
└── Properties
```

---

# Part 4 — Make sure the project currently works

Run:

```powershell
dotnet build
```

Look for:

```text
Build succeeded.
```

Then run:

```powershell
dotnet run
```

Test:

```text
http://localhost:xxxx/status
```

and:

```text
http://localhost:xxxx/price?subtotal=200&customerType=VIP&hasCoupon=true
```

Your price should still return:

```json
{
  "subtotal": 200,
  "customerType": "VIP",
  "hasCoupon": true,
  "total": 160
}
```

Stop the program:

```text
Ctrl + C
```

So our starting application is working.

---

# Part 5 — Create intentionally "smelly" code

We'll add a new file so you have something useful to analyse.

In Cursor:

**Right-click project → New File**

Name it:

```text
CustomerReportService.cs
```

Paste this:

```csharp
public class CustomerReportService
{
    public string GenerateReport(
        string customerName,
        string customerType,
        decimal amount,
        bool isActive)
    {
        string result = "";

        if (customerName == null || customerName == "")
        {
            result = "ERROR: Customer name required";
        }
        else
        {
            if (isActive == true)
            {
                if (customerType == "VIP")
                {
                    if (amount > 10000)
                    {
                        result =
                            "Customer: " + customerName +
                            ", Type: VIP" +
                            ", Amount: RM" + amount +
                            ", Status: Active" +
                            ", Category: High Value";
                    }
                    else
                    {
                        result =
                            "Customer: " + customerName +
                            ", Type: VIP" +
                            ", Amount: RM" + amount +
                            ", Status: Active" +
                            ", Category: Normal";
                    }
                }
                else
                {
                    if (amount > 10000)
                    {
                        result =
                            "Customer: " + customerName +
                            ", Type: Regular" +
                            ", Amount: RM" + amount +
                            ", Status: Active" +
                            ", Category: High Value";
                    }
                    else
                    {
                        result =
                            "Customer: " + customerName +
                            ", Type: Regular" +
                            ", Amount: RM" + amount +
                            ", Status: Active" +
                            ", Category: Normal";
                    }
                }
            }
            else
            {
                result =
                    "Customer: " + customerName +
                    ", Status: Inactive";
            }
        }

        return result;
    }
}
```

Save:

```text
Ctrl + S
```

---

# Part 6 — Don't improve the code!

This is important.

At this stage, **do not ask Cursor to fix it**.

We're doing:

```text
Lab 4
DETECT

not

FIX
```

The workflow is:

```text
Code
 ↓
Measure
 ↓
Analyse
 ↓
Detect smells
 ↓
Document findings
```

Refactoring comes afterwards.

---

# Part 7 — Verify that bad-quality code can still compile

Run:

```powershell
dotnet build
```

You'll probably get:

```text
Build succeeded.
```

This teaches an important lesson:

> **Build succeeded does not mean code quality is good.**

Our code compiles.

But the code is still difficult to maintain.

---

# Part 8 — First baseline measurement: build status

Record:

| Metric            |        Result |
| ----------------- | ------------: |
| Build successful  |           Yes |
| Compiler errors   |             0 |
| Compiler warnings | Record result |

For example:

```text
Code Quality Baseline

Build Status: PASS
Errors:       0
Warnings:     0
```

That's your first baseline measurement.

---

# Part 9 — Ask AI to explain the code

Open **Cursor Chat**.

Use:

```text
Review CustomerReportService.cs.

Do NOT modify the source code.

I am learning code quality analysis.

First explain what GenerateReport() does in beginner-friendly language.

Identify:

1. Inputs
2. Output
3. Main decisions
4. Business rules
5. Error handling
6. Overall complexity

Do not refactor anything yet.
```

AI should describe things like:

```text
Input
 ├── customerName
 ├── customerType
 ├── amount
 └── isActive
       ↓
Several nested decisions
       ↓
Generate report string
```

---

# Part 10 — Ask AI specifically for code smells

Now enter:

```text
Perform a code smell review of CustomerReportService.cs.

Do NOT modify the code.

Look specifically for:

- Long method
- Deep nesting
- Duplicate code
- Magic numbers
- Magic strings
- Poor naming
- Unnecessary conditions
- String construction problems
- Repeated business logic
- Maintainability problems

For every finding provide:

1. Code smell name
2. Exact code/example
3. Why it is a problem
4. Severity: Low, Medium or High
5. Recommended improvement

Return the findings as a table.
```

This is the core exercise.

---

# Part 11 — What should AI discover?

You should see findings similar to these.

### Smell 1 — Deep Nesting

For example:

```csharp
if (customerName == null || customerName == "")
{
}
else
{
    if (isActive == true)
    {
        if (customerType == "VIP")
        {
            if (amount > 10000)
```

That's:

```text
IF
 └── ELSE
      └── IF
           └── IF
                └── IF
```

Too many nested decisions make code difficult to follow.

---

# Part 12 — Smell 2: Duplicate Code

Look at:

```csharp
result =
    "Customer: " + customerName +
    ", Type: VIP" +
    ", Amount: RM" + amount +
    ", Status: Active" +
    ", Category: High Value";
```

Then:

```csharp
result =
    "Customer: " + customerName +
    ", Type: VIP" +
    ", Amount: RM" + amount +
    ", Status: Active" +
    ", Category: Normal";
```

Most of the code is identical.

Only this changes:

```text
High Value
Normal
```

That's duplicate logic.

---

# Part 13 — Smell 3: Magic Number

Look at:

```csharp
if (amount > 10000)
```

Question:

> What does `10000` mean?

A developer reading this months later doesn't immediately know.

It might mean:

```text
High Value Customer Threshold
```

A clearer solution eventually might be:

```csharp
const decimal HighValueThreshold = 10000m;
```

But **don't change it yet**.

For Lab 4 we simply record:

```text
Magic Number: 10000
Purpose: High-value threshold
```

---

# Part 14 — Smell 4: Magic String

Look at:

```csharp
customerType == "VIP"
```

`"VIP"` is embedded directly in the business logic.

That's a **magic string**.

If another developer types:

```csharp
"Vip"
```

or:

```csharp
"vip"
```

the behavior may change.

Record:

```text
Finding:
Hard-coded customer type = "VIP"
```

---

# Part 15 — Smell 5: Unnecessary Boolean Comparison

Look at:

```csharp
if (isActive == true)
```

Usually this can simply be:

```csharp
if (isActive)
```

The first version isn't necessarily wrong.

It's just unnecessarily verbose.

Severity:

```text
Low
```

---

# Part 16 — Smell 6: Poor Empty-String Check

Look at:

```csharp
if (customerName == null || customerName == "")
```

This works in many cases.

But it's verbose and does not handle whitespace such as:

```text
"    "
```

Later you could use:

```csharp
string.IsNullOrWhiteSpace(customerName)
```

Again, **don't refactor yet**.

---

# Part 17 — Smell 7: String Concatenation

The code repeatedly does:

```csharp
"Customer: " + customerName +
", Type: " + customerType +
", Amount: RM" + amount
```

A cleaner approach eventually would be interpolation:

```csharp
$"Customer: {customerName}, Type: {customerType}"
```

For now record it as:

```text
Readability issue
```

---

# Part 18 — Ask AI to rank everything

Now ask Cursor:

```text
Based on your previous review, rank all code-quality findings from highest priority to lowest priority.

Use this format:

Priority
Finding
Severity
Impact
Recommended Action

Do not change any code.
```

You might get:

| Priority | Finding                       | Severity |
| -------- | ----------------------------- | -------- |
| 1        | Deep nesting                  | High     |
| 2        | Duplicate report construction | High     |
| 3        | Repeated business rules       | Medium   |
| 4        | Magic number 10000            | Medium   |
| 5        | Magic string VIP              | Medium   |
| 6        | Verbose string concatenation  | Low      |
| 7        | `isActive == true`            | Low      |

That's now useful information instead of:

> "The code looks bad."

---

# Part 19 — Ask AI for a quantitative baseline

Use this prompt:

```text
Create a simple quantitative code-quality baseline for
CustomerReportService.cs.

Count or estimate:

- Number of methods
- Method length
- Maximum nesting depth
- Number of duplicated report-building blocks
- Number of magic numbers
- Number of important magic strings
- Number of code smell findings
- High severity findings
- Medium severity findings
- Low severity findings

Do NOT modify the code.

Present it as a table.
```

You may get something approximately like:

| Metric                  | Baseline |
| ----------------------- | -------: |
| Methods                 |        1 |
| Long methods            |        1 |
| Maximum nesting         |      4–5 |
| Duplicate blocks        |        4 |
| Magic numbers           |        1 |
| Important magic strings |  Several |
| High findings           |        2 |
| Medium findings         |        3 |
| Low findings            |        2 |

The exact numbers are less important than having a **repeatable baseline**.

---

# Part 20 — Create `CODE_QUALITY_BASELINE.md`

Now create:

```text
CODE_QUALITY_BASELINE.md
```

Put something like:

```markdown
# Code Quality Baseline

## Project

Lab02Api

## File Reviewed

CustomerReportService.cs

## Build Status

Build: PASS
Compiler Errors: 0
Compiler Warnings: 0

## Code Quality Findings

| Finding | Severity |
|---|---|
| Deep nesting | High |
| Duplicate report logic | High |
| Magic number 10000 | Medium |
| Hard-coded VIP value | Medium |
| Repeated business rules | Medium |
| Verbose boolean comparison | Low |
| Repetitive string concatenation | Low |

## Baseline Metrics

Maximum nesting depth: 4+
Long methods: 1
Duplicate blocks: Multiple
Magic number count: 1
Critical build errors: 0

## Conclusion

The code compiles successfully but contains several
maintainability issues. The highest priorities are reducing
nested conditional logic and eliminating duplicated report
construction.
```

Now you have a physical **baseline document**.

---

# Part 21 — Ask AI for improvement recommendations

Now ask:

```text
Based on CODE_QUALITY_BASELINE.md and
CustomerReportService.cs, create a refactoring plan.

Do NOT change the code.

For each recommendation provide:

1. Problem
2. Proposed change
3. Expected benefit
4. Risk of making the change
5. How we should verify behavior afterwards

Order recommendations from highest value to lowest value.
```

Notice again:

```text
AI Recommendation
      ≠
AI automatically changes everything
```

A responsible workflow is:

```text
Detect
 ↓
Explain
 ↓
Prioritize
 ↓
Plan
 ↓
Human decides
```

---

# Part 22 — Compare Bug vs Code Smell

Your `CustomerReportService.cs` probably works.

For example:

```text
John
VIP
RM20,000
Active
```

might correctly return:

```text
Customer: John,
Type: VIP,
Amount: RM20000,
Status: Active,
Category: High Value
```

So:

```text
FUNCTIONALLY
✓ Works

CODE QUALITY
✗ Poor
```

That's the key lesson of Lab 4.

---

# Part 23 — Why don't we use SonarQube yet?

Because your original course puts **general code-quality analysis before the dedicated SonarQube module**. Module 2 introduces code smells and technical debt first; SonarQube report analysis appears later as its own module.  

So I recommend this learning sequence:

```text
LAB 4
Human + AI understand
what code smells are
        ↓
LAB 5
Refactor and optimize
        ↓
Later labs
SonarQube automatically
detects/measures issues
```

That's much easier for beginners.

---

# Part 24 — Your Lab 4 deliverables

At the end of Lab 4 you should have:

```text
Lab02Api
│
├── Program.cs
├── OrderService.cs
├── CustomerReportService.cs
├── CODE_QUALITY_BASELINE.md
├── Lab02Api.csproj
└── ...
```

And your checklist:

```text
[✓] Existing .NET project builds

[✓] CustomerReportService.cs created

[✓] Intentionally smelly code analysed

[✓] AI explained the code

[✓] AI identified code smells

[✓] Deep nesting identified

[✓] Duplication identified

[✓] Magic numbers identified

[✓] Magic strings identified

[✓] Findings ranked by severity

[✓] Baseline metrics recorded

[✓] CODE_QUALITY_BASELINE.md created

[✓] Refactoring recommendations generated

[✓] Source code NOT refactored yet
```

## The one diagram to remember

```text
             SOURCE CODE
                  │
                  ▼
           Does it compile?
                  │
                  ▼
             dotnet build
                  │
                  ▼
          Build Baseline
                  │
                  ▼
             AI Review
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
   Duplication  Nesting   Magic Values
       │          │          │
       └──────────┼──────────┘
                  ▼
           Rank Severity
                  │
                  ▼
       CODE_QUALITY_BASELINE.md
                  │
                  ▼
       Improvement Recommendations
```

**For Lab 4, don't worry about SonarQube, Jenkins, complex metrics, or special tools yet.** The learning objective is simply: **"This program works, but how do I systematically identify why its code quality is poor?"** Once that clicks, the later SonarQube labs will make much more sense.
