## Lab 3 — Refactor and Debug Legacy Code with AI Assistance

### What are we learning?

The workflow is:

```text
Messy / Legacy Code
        ↓
Reproduce Bug
        ↓
Ask AI to Explain Code
        ↓
Ask AI to Find Root Cause
        ↓
Fix ONLY the Bug
        ↓
Test Again
        ↓
Ask AI to Refactor
        ↓
Test Again
        ↓
Review AI Changes
```

This directly supports the original course topics of **code explanation, code refactoring/modernization, and debugging/troubleshooting**. 

---

# Step 1 — Open your Lab 2 project

Open **PowerShell**.

Go to the project you created in Lab 2:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Check that it still builds:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

Then open the folder using Cursor:

```text
File → Open Folder
```

Select:

```text
C:\AI_SDLC_Labs\Lab02Api
```

---

# Step 2 — Create the "legacy" code

In Cursor, right-click the project folder:

```text
Lab02Api
```

Select:

```text
New File
```

Name it:

```text
OrderService.cs
```

Paste this code exactly:

```csharp
public class OrderService
{
    public decimal CalculateFinalAmount(
        decimal subtotal,
        string customerType,
        bool hasCoupon)
    {
        if (subtotal < 0)
        {
            throw new ArgumentException("Subtotal cannot be negative.");
        }

        decimal total = subtotal;

        if (customerType == "VIP")
        {
            if (subtotal >= 100)
            {
                total = subtotal - (subtotal * 0.20m);
            }
            else
            {
                total = subtotal - (subtotal * 0.10m);
            }
        }
        else
        {
            if (subtotal >= 100)
            {
                total = subtotal - (subtotal * 0.05m);
            }
            else
            {
                total = subtotal;
            }
        }

        if (hasCoupon)
        {
            if (customerType == "VIP")
            {
                total = total - (total * 0.10m);
            }
            else
            {
                total = total - (total * 0.10m);
            }
        }

        if (total < 0)
        {
            total = 0;
        }

        return Math.Round(total, 2);
    }
}
```

Don't improve it yet.

I've deliberately made it:

* unnecessarily nested;
* repetitive;
* harder to understand;
* and **buggy**.

That is our fake "legacy code."

---

# Step 3 — Understand the business requirement

Pretend the customer gave us these requirements.

```text
VIP Customer
------------------------------
Purchase ≥ RM100 → 20% discount
Purchase < RM100 → 10% discount

Normal Customer
------------------------------
Purchase ≥ RM100 → 5% discount
Purchase < RM100 → No discount

Coupon
------------------------------
Normal customer → Extra 10%
VIP customer → Coupon NOT allowed

Important:
VIP discounts cannot be combined with coupons.
```

Therefore:

```text
Customer: VIP
Subtotal: RM200
Coupon: Yes

Expected calculation:

RM200 × 80%
= RM160
```

The correct answer should be:

```text
RM160
```

But our legacy code contains a bug.

---

# Step 4 — Create an API to call OrderService

Now open:

```text
Program.cs
```

Find:

```csharp
app.Run();
```

Immediately **before** `app.Run();`, insert:

```csharp
var orderService = new OrderService();

app.MapGet("/price", (
    decimal subtotal,
    string customerType,
    bool hasCoupon) =>
{
    var total = orderService.CalculateFinalAmount(
        subtotal,
        customerType,
        hasCoupon);

    return Results.Ok(new
    {
        subtotal,
        customerType,
        hasCoupon,
        total
    });
});
```

The bottom of `Program.cs` should therefore contain something like:

```csharp
var orderService = new OrderService();

app.MapGet("/price", (
    decimal subtotal,
    string customerType,
    bool hasCoupon) =>
{
    var total = orderService.CalculateFinalAmount(
        subtotal,
        customerType,
        hasCoupon);

    return Results.Ok(new
    {
        subtotal,
        customerType,
        hasCoupon,
        total
    });
});

app.Run();
```

Save:

```text
Ctrl + S
```

---

# Step 5 — Build the application

Run:

```powershell
dotnet build
```

You should get:

```text
Build succeeded.
```

If you don't, give the error message to Cursor:

```text
My dotnet build failed.

Here is the error:

[paste error]

Explain the error to me as a beginner and tell me the
smallest change needed to fix it.
```

---

# Step 6 — Run the API

Run:

```powershell
dotnet run
```

You should see something like:

```text
Now listening on: http://localhost:5187
```

Your port may be different.

For example, if yours is:

```text
5187
```

open:

```text
http://localhost:5187/price?subtotal=200&customerType=VIP&hasCoupon=true
```

---

# Step 7 — Look at the bug

You should get approximately:

```json
{
    "subtotal": 200,
    "customerType": "VIP",
    "hasCoupon": true,
    "total": 144
}
```

But our requirement says:

```text
Expected = RM160
Actual   = RM144
```

### Why RM144?

The code first gives the VIP discount:

```text
RM200 - 20%
= RM160
```

Then incorrectly applies another 10% coupon:

```text
RM160 - 10%
= RM144
```

There is our bug.

**We have now reproduced the bug.**

This is important in software engineering.

Don't start changing code until you can demonstrate the problem.

---

# Step 8 — Now use AI to understand the legacy code

Don't tell AI where the bug is yet.

Open **Cursor Chat** and enter:

```text
I am learning how to debug legacy C# code.

Please examine OrderService.cs.

Do NOT rewrite or modify the code yet.

Explain the CalculateFinalAmount method in beginner-friendly
language.

Identify:

1. Inputs
2. Output
3. Validation
4. Decision points
5. Side effects
6. Business rules that appear to be implemented
7. Areas that are difficult to maintain

Explain the execution flow step by step.
```

This is the first important skill in Lab 3:

```text
AI ≠ immediately generate code

AI → understand existing code first
```

---

# Step 9 — Ask AI to investigate the bug

Now tell Cursor the expected behavior:

```text
We have reproduced this problem:

Input:
subtotal = 200
customerType = "VIP"
hasCoupon = true

Expected:
160

Actual:
144

Business rule:
VIP customers cannot combine their VIP discount with coupons.

Do NOT change the code yet.

Give me three possible root-cause hypotheses.

Rank them from most likely to least likely.

For each hypothesis:
- identify the relevant code
- explain why it could produce 144
- show the calculation
```

AI should identify this area:

```csharp
if (hasCoupon)
{
    if (customerType == "VIP")
    {
        total = total - (total * 0.10m);
    }
```

That is suspicious because the requirement says:

```text
VIP customer → coupon NOT allowed
```

---

# Step 10 — Ask AI for the smallest bug fix

Now give Cursor this instruction:

```text
Fix ONLY the confirmed bug.

Requirement:

VIP customers must not receive the coupon discount.

Normal customers can receive the coupon discount.

Do not refactor the rest of the method yet.

Show me exactly what you intend to change before changing it.
```

The simplest fix is approximately:

```csharp
if (hasCoupon && customerType != "VIP")
{
    total = total - (total * 0.10m);
}
```

Notice what we're doing:

```text
BUG FIX first
     ↓
REFACTOR later
```

Don't combine everything into one huge AI-generated change.

That's an important professional practice.

---

# Step 11 — Test the fix

Save the changes.

Stop the application:

```text
Ctrl + C
```

Build again:

```powershell
dotnet build
```

Then:

```powershell
dotnet run
```

Test again:

```text
http://localhost:5187/price?subtotal=200&customerType=VIP&hasCoupon=true
```

Now you should get:

```json
{
    "subtotal": 200,
    "customerType": "VIP",
    "hasCoupon": true,
    "total": 160
}
```

Excellent.

```text
Expected = RM160
Actual   = RM160

PASS
```

---

# Step 12 — Test another customer

We shouldn't test only one scenario.

Try:

```text
http://localhost:5187/price?subtotal=200&customerType=Regular&hasCoupon=true
```

Calculation:

```text
RM200

5% normal customer discount
RM200 × 95%
= RM190

10% coupon
RM190 × 90%
= RM171
```

Expected:

```json
{
    "subtotal": 200,
    "customerType": "Regular",
    "hasCoupon": true,
    "total": 171
}
```

Now we know the bug fix didn't break the normal customer case.

---

# Step 13 — Now ask AI to identify bad code

We fixed the **defect**.

But this code is still ugly:

```csharp
if (customerType == "VIP")
{
    if (subtotal >= 100)
    {
        ...
    }
    else
    {
        ...
    }
}
else
{
    if (subtotal >= 100)
    {
        ...
    }
...
```

Ask Cursor:

```text
The bug is now fixed.

Do NOT change the code yet.

Review CalculateFinalAmount for maintainability problems.

Look specifically for:

- duplicated code
- nested if statements
- long method
- unclear responsibilities
- readability problems

Give me a refactoring plan.

The refactoring MUST preserve the current behavior.
```

You might get recommendations such as:

```text
Extract customer discount logic

Extract coupon logic

Reduce nested conditionals

Use clearer method names
```

Now we're entering the **refactoring** portion of Lab 3.

---

# Step 14 — Ask AI to refactor it

Tell Cursor:

```text
Refactor OrderService.cs according to the plan.

Requirements:

1. Preserve existing behavior.
2. Do not introduce additional libraries.
3. Do not change the public CalculateFinalAmount method signature.
4. Reduce nested if statements.
5. Extract clearly named private methods where appropriate.
6. Keep the code simple enough for a junior developer.
```

A good final version could look like this:

```csharp
public class OrderService
{
    public decimal CalculateFinalAmount(
        decimal subtotal,
        string customerType,
        bool hasCoupon)
    {
        ValidateSubtotal(subtotal);

        decimal total =
            ApplyCustomerDiscount(subtotal, customerType);

        if (hasCoupon && customerType != "VIP")
        {
            total = ApplyCoupon(total);
        }

        return Math.Round(total, 2);
    }

    private static void ValidateSubtotal(decimal subtotal)
    {
        if (subtotal < 0)
        {
            throw new ArgumentException(
                "Subtotal cannot be negative.");
        }
    }

    private static decimal ApplyCustomerDiscount(
        decimal subtotal,
        string customerType)
    {
        if (customerType == "VIP")
        {
            return subtotal >= 100
                ? subtotal * 0.80m
                : subtotal * 0.90m;
        }

        return subtotal >= 100
            ? subtotal * 0.95m
            : subtotal;
    }

    private static decimal ApplyCoupon(decimal total)
    {
        return total * 0.90m;
    }
}
```

Compare this with the original.

### Before

```text
CalculateFinalAmount
       ↓
Validation
       ↓
VIP checking
       ↓
Nested amount checking
       ↓
Normal checking
       ↓
Nested amount checking
       ↓
Coupon checking
       ↓
VIP checking again
       ↓
Calculation
```

### After

```text
CalculateFinalAmount
      │
      ├── ValidateSubtotal()
      │
      ├── ApplyCustomerDiscount()
      │
      └── ApplyCoupon()
```

Much easier to understand.

---

# Step 15 — Very important: run it again

AI said the refactoring preserves behavior.

**Don't believe it automatically.**

Stop:

```text
Ctrl + C
```

Build:

```powershell
dotnet build
```

Run:

```powershell
dotnet run
```

Retest VIP:

```text
http://localhost:5187/price?subtotal=200&customerType=VIP&hasCoupon=true
```

Expected:

```text
160
```

Retest Regular:

```text
http://localhost:5187/price?subtotal=200&customerType=Regular&hasCoupon=true
```

Expected:

```text
171
```

If both are correct, the refactoring is behaving properly for our test cases.

---

# Step 16 — Ask AI to explain the refactored code

Now ask:

```text
Explain the refactored OrderService.cs to me as a beginner.

Explain:

1. Why ValidateSubtotal was extracted.
2. Why ApplyCustomerDiscount was extracted.
3. Why ApplyCoupon was extracted.
4. Why the new version is easier to maintain.
5. What behavior was preserved.
6. What risks I should still test.
```

This is important because the course is not:

```text
AI writes code → Finished
```

It should be:

```text
AI writes code
      ↓
Developer understands
      ↓
Developer validates
      ↓
Developer accepts
```

---

# Step 17 — Ask AI to review its own work

Enter:

```text
Act as a senior C# code reviewer.

Review the refactored OrderService.cs.

Check for:

- accidental behavior changes
- incorrect discount calculations
- hidden edge cases
- maintainability problems
- naming problems

Do not modify anything.

Give me your findings first.
```

This teaches you another useful AI pattern:

```text
AI Developer
     ↓
creates code

Different AI Prompt
     ↓
reviews code
```

---

# Step 18 — Optional Git part

The original Lab 3 also wants the **bug fix and refactoring separated**.

If you have Git installed, before starting Lab 3:

```powershell
git init
```

Then:

```powershell
git add .
```

```powershell
git commit -m "Baseline before Lab 3"
```

Create the lab branch:

```powershell
git switch -c lab03-legacy-refactor-debug
```

After fixing only the bug:

```powershell
git add .
git commit -m "Fix VIP coupon stacking bug"
```

After refactoring:

```powershell
git add .
git commit -m "Refactor order discount calculation"
```

Then:

```powershell
git log --oneline
```

You should see something like:

```text
31fe582 Refactor order discount calculation
79e924a Fix VIP coupon stacking bug
b1128fa Baseline before Lab 3
```

That's actually a very good SDLC practice.

---

# Step 19 — What did Lab 3 actually teach you?

The point of Lab 3 is **not C# discount calculation**.

The discount example is just the vehicle.

The actual skill is:

```text
                Existing Code
                     ↓
              AI Explanation
                     ↓
             Reproduce Problem
                     ↓
             AI Root-Cause Analysis
                     ↓
                Human Review
                     ↓
                  Bug Fix
                     ↓
                   Test
                     ↓
                Refactoring
                     ↓
                   Test
                     ↓
              AI Code Review
                     ↓
                Human Approval
```

## The difference between debugging and refactoring

This distinction is important.

**Debugging**

```text
Something is WRONG
       ↓
Find the cause
       ↓
Fix the defect
```

Example:

```text
VIP expected RM160
but received RM144
```

**Refactoring**

```text
Program already works
       ↓
Improve internal code
       ↓
Behavior should NOT change
```

Example:

```text
Before:
Large nested CalculateFinalAmount()

After:
CalculateFinalAmount()
ApplyCustomerDiscount()
ApplyCoupon()
ValidateSubtotal()
```

So in Lab 3 you are practicing **two different AI-assisted activities**:

```text
AI-Assisted Debugging
        +

AI-Assisted Refactoring
```

### Your final Lab 3 checklist

By the end, you should be able to show:

```text
[✓] OrderService.cs created

[✓] Bug reproduced
    VIP + RM200 + coupon
    Actual originally = RM144

[✓] AI explained the legacy code

[✓] AI generated root-cause hypotheses

[✓] Bug identified:
    VIP coupon incorrectly stacked

[✓] Bug fixed

[✓] Correct result = RM160

[✓] Legacy code refactored

[✓] Application still builds

[✓] API still runs

[✓] Results tested after refactoring

[✓] AI reviewed the final code
```

**This is how I think Lab 3 should have been written in the original 30-lab guide.** It shouldn't tell a beginner, “take a 30–50-line legacy method with a reproducible defect” and assume they already have one. The lab should **provide the legacy code, deliberately introduce the defect, specify the expected result, and then lead the participant through the AI-assisted SDLC workflow.**
