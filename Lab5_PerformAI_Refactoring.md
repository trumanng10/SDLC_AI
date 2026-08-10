**Lab 5 is basically the “FIX” stage after Lab 4’s “FIND” stage.**

In Lab 4, we identified problems such as deep nesting, duplicated code, magic numbers, magic strings, and poor readability. 
Lab 5 now uses AI to **refactor those problems without changing what the program does**. That matches the original course topics of refactoring strategies, improving readability and maintainability, and generating code-improvement recommendations. 

# Lab 5 — Perform AI-Assisted Refactoring for Readability and Maintainability

## Objectives

By the end of this lab, you should be able to:

* Understand what refactoring means.
* Use AI to propose a refactoring plan before changing code.
* Reduce deeply nested `if` statements.
* Remove duplicated code.
* Replace magic values with meaningful constants.
* Improve naming and readability.
* Split a large method into smaller methods.
* Verify that behavior remains the same after refactoring.
* Compare your Lab 4 baseline against the improved code.

---

# First: What exactly is refactoring?

Suppose this code works:

```csharp
if (customerType == "VIP")
{
    if (amount > 10000)
    {
        //...
    }
}
```

You change it into cleaner code:

```csharp
bool isVip = customerType == VipCustomerType;
bool isHighValue = amount > HighValueThreshold;
```

The program should still produce the **same result**.

That is refactoring.

```text
Before
Program works
Code difficult to maintain

        ↓ Refactor

After
Program still works
Code easier to maintain
```

The most important rule is:

> **Refactoring changes the structure of the code, not the expected business behavior.**

---

# Step 1 — Continue from Lab 4

Open PowerShell:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Open the same folder in Cursor.

You should have approximately:

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

We're going to refactor:

```text
CustomerReportService.cs
```

---

# Step 2 — Look at the ugly code again

Your Lab 4 code looked approximately like this:

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

This code isn't necessarily broken.

The problem is:

```text
Too much nesting
       +
Repeated report construction
       +
Magic number 10000
       +
Magic string "VIP"
       +
Long method
       +
Poor readability
```

---

# Step 3 — Build before touching anything

Run:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

This establishes:

```text
BEFORE REFACTORING
Build = PASS
```

Do not refactor code that doesn't currently compile unless fixing compilation is specifically part of the task.

---

# Step 4 — Make the code testable through the browser

We should be able to prove that the behavior stays the same.

Open:

```text
Program.cs
```

Add this **before**:

```csharp
app.Run();
```

Insert:

```csharp
var reportService = new CustomerReportService();

app.MapGet("/customer-report", (
    string customerName,
    string customerType,
    decimal amount,
    bool isActive) =>
{
    var report = reportService.GenerateReport(
        customerName,
        customerType,
        amount,
        isActive);

    return Results.Ok(new
    {
        report
    });
});
```

So near the bottom you should have:

```csharp
var reportService = new CustomerReportService();

app.MapGet("/customer-report", (
    string customerName,
    string customerType,
    decimal amount,
    bool isActive) =>
{
    var report = reportService.GenerateReport(
        customerName,
        customerType,
        amount,
        isActive);

    return Results.Ok(new
    {
        report
    });
});

app.Run();
```

Save.

---

# Step 5 — Run the API

Execute:

```powershell
dotnet run
```

Suppose you see:

```text
Now listening on: http://localhost:5187
```

Use your actual port number.

---

# Step 6 — Record the BEFORE results

Test this first:

```text
http://localhost:5187/customer-report?customerName=John&customerType=VIP&amount=20000&isActive=true
```

You should get approximately:

```json
{
  "report": "Customer: John, Type: VIP, Amount: RM20000, Status: Active, Category: High Value"
}
```

Write that down.

Now test:

```text
http://localhost:5187/customer-report?customerName=Mary&customerType=VIP&amount=5000&isActive=true
```

Expected:

```text
Customer: Mary,
Type: VIP,
Amount: RM5000,
Status: Active,
Category: Normal
```

Test:

```text
http://localhost:5187/customer-report?customerName=David&customerType=Regular&amount=20000&isActive=true
```

Expected category:

```text
High Value
```

Test:

```text
http://localhost:5187/customer-report?customerName=Susan&customerType=Regular&amount=5000&isActive=true
```

Expected category:

```text
Normal
```

Finally:

```text
http://localhost:5187/customer-report?customerName=Peter&customerType=VIP&amount=20000&isActive=false
```

Expected:

```text
Customer: Peter, Status: Inactive
```

These are your **before-refactoring results**.

---

# Step 7 — Why are we doing this?

Because later AI might say:

> "I've successfully refactored your code."

That doesn't prove anything.

You must test:

```text
BEFORE output
      =
AFTER output
```

For all important scenarios.

That is called **behavior preservation**.

---

# Step 8 — Stop the API

Press:

```text
Ctrl + C
```

Now we're ready to refactor.

---

# Step 9 — Ask AI to analyze before changing anything

Open Cursor Chat.

Enter:

```text
Review CustomerReportService.cs.

We are performing a refactoring exercise.

IMPORTANT:
Do NOT modify the code yet.

The business behavior must remain unchanged.

Identify opportunities to improve:

1. Readability
2. Maintainability
3. Deep nesting
4. Duplicate code
5. Magic numbers
6. Magic strings
7. Method size
8. Naming
9. String construction

Give me a prioritized refactoring plan.

For every proposed change explain:
- What is wrong
- What you recommend
- Why it improves maintainability
- Risk of changing it
```

Notice that we're telling AI:

```text
PLAN FIRST
CODE LATER
```

That's a better AI development practice than:

```text
"Make this code better."
```

---

# Step 10 — AI should identify the problems

AI should find things resembling:

```text
1. Replace deep nesting with guard clauses.

2. Extract customer category logic.

3. Extract report construction.

4. Replace 10000 with a named constant.

5. Replace "VIP" with a named constant.

6. Replace repeated concatenation with string interpolation.

7. Simplify isActive == true.

8. Break GenerateReport into smaller methods.
```

Now you review the plan before asking AI to modify anything.

---

# Step 11 — Refactor ONE concern at a time

Don't tell AI:

```text
Fix everything.
```

Instead we'll perform controlled changes.

Start with this prompt:

```text
Implement only these changes:

1. Replace the magic number 10000 with a meaningful constant.
2. Replace the repeated "VIP" business value with a meaningful constant.

Do not perform any other refactoring.

Show me the changes.
```

AI may create:

```csharp
private const decimal HighValueThreshold = 10000m;
private const string VipCustomerType = "VIP";
```

Then replace:

```csharp
amount > 10000
```

with:

```csharp
amount > HighValueThreshold
```

and:

```csharp
customerType == "VIP"
```

with:

```csharp
customerType == VipCustomerType
```

---

# Step 12 — Why is that better?

Compare:

```csharp
if (amount > 10000)
```

with:

```csharp
if (amount > HighValueThreshold)
```

The second one explains itself.

A developer immediately understands:

```text
10000
↓
What the hell is this?

HighValueThreshold
↓
Ah — this determines whether
the customer is high value.
```

That's maintainability.

---

# Step 13 — Build after the small change

Run:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

If it fails, stop there.

Don't continue adding refactorings on top of broken code.

---

# Step 14 — Ask AI to remove deep nesting

Now use:

```text
CustomerReportService.cs still has deep nested if statements.

Refactor the control flow to reduce nesting.

Requirements:

1. Preserve existing behavior.
2. Prefer guard clauses where appropriate.
3. Do not change the public GenerateReport method signature.
4. Do not add external libraries.
5. Do not change business rules.
6. Keep the solution understandable for junior developers.

Show me your proposed change first.
```

The most obvious improvement is handling invalid or inactive customers early.

For example:

```csharp
if (string.IsNullOrWhiteSpace(customerName))
{
    return "ERROR: Customer name required";
}

if (!isActive)
{
    return $"Customer: {customerName}, Status: Inactive";
}
```

Instead of:

```text
IF invalid
ELSE
   IF active
      IF VIP
         IF amount
            ...
```

Now the flow becomes:

```text
Invalid?
  ↓ Yes
Return error

  ↓ No

Inactive?
  ↓ Yes
Return inactive

  ↓ No

Process active customer
```

Much flatter.

---

# Step 15 — Understand a Guard Clause

A guard clause handles exceptional/simple cases early.

Before:

```csharp
if (customerName == null || customerName == "")
{
    result = "ERROR...";
}
else
{
    // another huge block
}
```

After:

```csharp
if (string.IsNullOrWhiteSpace(customerName))
{
    return "ERROR: Customer name required";
}
```

Once invalid input is handled, the rest of the method doesn't need to sit inside an `else`.

That's why it reduces nesting.

---

# Step 16 — Build again

```powershell
dotnet build
```

Again:

```text
Build succeeded.
```

---

# Step 17 — Ask AI to remove duplicated report construction

Now prompt:

```text
Review the remaining duplicated report-building logic
in CustomerReportService.cs.

Refactor ONLY the duplicated report construction.

Requirements:

- Preserve identical output.
- Do not change business rules.
- Prefer named variables or a small helper method.
- Keep it beginner friendly.
```

Currently we're doing things like:

```csharp
"Customer: " + customerName +
", Type: VIP" +
", Amount: RM" + amount +
", Status: Active" +
", Category: High Value";
```

and again:

```csharp
"Customer: " + customerName +
", Type: VIP" +
", Amount: RM" + amount +
", Status: Active" +
", Category: Normal";
```

Most of it is repeated.

A better design is:

```csharp
string category =
    amount > HighValueThreshold
        ? "High Value"
        : "Normal";
```

Then:

```csharp
return $"Customer: {customerName}, " +
       $"Type: {customerType}, " +
       $"Amount: RM{amount}, " +
       $"Status: Active, " +
       $"Category: {category}";
```

Now the only thing that changes is:

```text
category
```

instead of repeating the entire report.

---

# Step 18 — Ask AI to extract business rules

Now go one step further:

```text
Refactor CustomerReportService.cs so that
GenerateReport is easier to read.

Extract clearly named private methods where they improve
readability.

Do not over-engineer the solution.

Requirements:

- Preserve output.
- Preserve GenerateReport's public signature.
- Do not introduce interfaces or design patterns.
- Do not add packages.
- Keep it suitable for beginners.
```

A reasonable result could look like this:

```csharp
public class CustomerReportService
{
    private const decimal HighValueThreshold = 10000m;
    private const string VipCustomerType = "VIP";

    public string GenerateReport(
        string customerName,
        string customerType,
        decimal amount,
        bool isActive)
    {
        if (string.IsNullOrWhiteSpace(customerName))
        {
            return "ERROR: Customer name required";
        }

        if (!isActive)
        {
            return CreateInactiveReport(customerName);
        }

        string normalizedCustomerType =
            GetCustomerType(customerType);

        string category =
            GetCustomerCategory(amount);

        return CreateActiveReport(
            customerName,
            normalizedCustomerType,
            amount,
            category);
    }

    private static string GetCustomerType(string customerType)
    {
        return customerType == VipCustomerType
            ? VipCustomerType
            : "Regular";
    }

    private static string GetCustomerCategory(decimal amount)
    {
        return amount > HighValueThreshold
            ? "High Value"
            : "Normal";
    }

    private static string CreateInactiveReport(
        string customerName)
    {
        return $"Customer: {customerName}, Status: Inactive";
    }

    private static string CreateActiveReport(
        string customerName,
        string customerType,
        decimal amount,
        string category)
    {
        return
            $"Customer: {customerName}, " +
            $"Type: {customerType}, " +
            $"Amount: RM{amount}, " +
            $"Status: Active, " +
            $"Category: {category}";
    }
}
```

Now look at `GenerateReport`.

It reads almost like English:

```text
Validate customer name

If inactive
    create inactive report

Determine customer type

Determine category

Create active report
```

That's much easier to maintain.

---

# Step 19 — Compare BEFORE and AFTER

### BEFORE

```text
GenerateReport
    ↓
if name
    ↓
else
    ↓
if active
    ↓
if VIP
    ↓
if amount
    ↓
construct report
```

### AFTER

```text
GenerateReport
      │
      ├── Validate Name
      │
      ├── CreateInactiveReport()
      │
      ├── GetCustomerType()
      │
      ├── GetCustomerCategory()
      │
      └── CreateActiveReport()
```

The second design separates responsibilities.

---

# Step 20 — But don't trust it yet

Run:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

Then:

```powershell
dotnet run
```

Now repeat the URLs you tested before.

### VIP / High Value

```text
http://localhost:xxxx/customer-report?customerName=John&customerType=VIP&amount=20000&isActive=true
```

Expected:

```text
Customer: John, Type: VIP, Amount: RM20000, Status: Active, Category: High Value
```

### VIP / Normal Value

```text
http://localhost:xxxx/customer-report?customerName=Mary&customerType=VIP&amount=5000&isActive=true
```

Expected category:

```text
Normal
```

### Regular / High Value

```text
http://localhost:xxxx/customer-report?customerName=David&customerType=Regular&amount=20000&isActive=true
```

Expected:

```text
High Value
```

### Regular / Normal Value

```text
http://localhost:xxxx/customer-report?customerName=Susan&customerType=Regular&amount=5000&isActive=true
```

Expected:

```text
Normal
```

### Inactive

```text
http://localhost:xxxx/customer-report?customerName=Peter&customerType=VIP&amount=20000&isActive=false
```

Expected:

```text
Customer: Peter, Status: Inactive
```

---

# Step 21 — Compare behavior

Create a simple table:

| Scenario         | Before     | After      | Result |
| ---------------- | ---------- | ---------- | ------ |
| VIP RM20k Active | High Value | High Value | PASS   |
| VIP RM5k Active  | Normal     | Normal     | PASS   |
| Regular RM20k    | High Value | High Value | PASS   |
| Regular RM5k     | Normal     | Normal     | PASS   |
| Inactive         | Inactive   | Inactive   | PASS   |

The important column is:

```text
Before = After
```

That proves the refactoring preserved the tested behavior.

---

# Step 22 — Now ask AI to review the refactoring

Use:

```text
Act as a senior C# code reviewer.

Compare the current CustomerReportService.cs
against the original Lab 4 version.

Do NOT change anything.

Explain which code smells were removed or reduced.

Compare:

- Maximum nesting depth
- Duplicate code
- Magic numbers
- Magic strings
- Method readability
- Method responsibilities
- String construction
- Maintainability

Present Before vs After.
```

---

# Step 23 — Update the Lab 4 baseline

Your Lab 4 baseline might have looked like:

| Metric              |  Before |
| ------------------- | ------: |
| Deep nesting        |    High |
| Long methods        |       1 |
| Duplicate blocks    |       4 |
| Magic numbers       |       1 |
| `"VIP"` repetitions | Several |
| Readability         |    Poor |

After Lab 5:

| Metric                  |   Before |          After |
| ----------------------- | -------: | -------------: |
| Deep nesting            |       4+ |              1 |
| Large methods           |        1 |              0 |
| Duplicate report blocks | Multiple |              0 |
| Magic number            |  `10000` | Named constant |
| VIP magic strings       | Multiple |    Centralized |
| String construction     | Repeated |    Centralized |
| Readability             |     Poor |       Improved |

**That is what makes Lab 5 measurable.**

---

# Step 24 — Ask AI whether it over-refactored

This is another useful exercise.

Ask:

```text
Review your refactoring critically.

Did you over-engineer anything?

Could any extracted method make the code harder rather than
easier to understand?

Recommend simplifications if appropriate.

Do NOT change the code.
```

This matters because:

```text
More methods
≠
Automatically better code
```

Refactoring should make things clearer, not create unnecessary architecture.

---

# Step 25 — Optional Git workflow

Before refactoring:

```powershell
git add .
git commit -m "Baseline before readability refactoring"
```

Create a branch:

```powershell
git switch -c lab05-refactoring
```

After your changes:

```powershell
git add .
git commit -m "Refactor customer reporting for maintainability"
```

Then:

```powershell
git diff HEAD~1 HEAD
```

This lets you inspect exactly what AI changed.

That's extremely useful when working with AI-generated modifications.

---

# Step 26 — The most important AI rule in this lab

Don't use:

```text
Please make my code better.
```

That's too broad.

Instead:

```text
Analyze only
        ↓
Propose plan
        ↓
Human reviews
        ↓
Change one concern
        ↓
Build
        ↓
Change next concern
        ↓
Build
        ↓
Run tests
        ↓
Human review
```

This gives you much more control.

---

# Step 27 — What exactly did AI do?

In this lab, AI was acting as a:

```text
AI Code Reviewer
       ↓
Identified maintainability problems

AI Refactoring Assistant
       ↓
Suggested improved structure

AI Developer
       ↓
Generated refactored code

AI Reviewer
       ↓
Reviewed its own changes
```

But **you** remained responsible for:

```text
Accepting changes
Checking business requirements
Building code
Testing outputs
Verifying behavior
```

---

# Step 28 — Lab 4 vs Lab 5

This is the easiest way to remember the difference:

```text
LAB 4
────────────────────────
What's wrong with
the QUALITY of this code?

        ↓

Detect code smells
Create baseline


LAB 5
────────────────────────
How can we improve
the code WITHOUT
breaking behavior?

        ↓

AI-assisted refactoring
Verify results
```

---

# Step 29 — Lab 5 final deliverables

At completion, you should have:

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

And you should be able to demonstrate:

```text
BEFORE
CustomerReportService.cs
↓
Deep nesting
Duplication
Magic values
Poor readability

         ↓ AI Refactoring

AFTER
CustomerReportService.cs
↓
Guard clauses
Smaller methods
Named constants
Less duplication
Clear business logic

         ↓

dotnet build
PASS

         ↓

Functional Testing
PASS

         ↓

Before output = After output
```

## Lab 5 completion checklist

1. `CustomerReportService.cs` from Lab 4 is working before refactoring.
2. Baseline outputs are captured.
3. AI analyses the source without immediately modifying it.
4. AI generates a refactoring plan.
5. Magic values are replaced with meaningful constants.
6. Deep nesting is reduced.
7. Duplicate report construction is removed.
8. Large logic is divided into meaningful responsibilities.
9. String construction is simplified.
10. `dotnet build` succeeds after the changes.
11. All five baseline scenarios are retested.
12. Before and after outputs match.
13. AI performs a final maintainability review.
14. Lab 4 baseline metrics are compared with Lab 5 results.

The key lesson is:

```text
              LAB 4
          Identify Problems
                 │
                 ▼
        CODE QUALITY BASELINE
                 │
                 ▼
              LAB 5
                 │
          AI Refactor Plan
                 │
                 ▼
        Small Controlled Changes
                 │
                 ▼
            dotnet build
                 │
                 ▼
          Functional Testing
                 │
                 ▼
       BEFORE = AFTER ?
            /          \
          NO            YES
          ↓              ↓
       Reject/Fix     Accept
                         │
                         ▼
              Maintainable Code
```

For your training course, **I would make Lab 5 directly continue from Lab 4 exactly like this**. That makes the learning progression very clear: **Lab 4 discovers the code smells; Lab 5 removes them while proving that the application's behavior has not changed.**
