**Lab 14 is the direct continuation of Lab 13.**

In Lab 13, you created unit tests and got something like:

```text
Passed: 8
Failed: 0
```

But **8 passing tests do not tell you whether all important code paths are tested**. Lab 14 measures **code coverage**, finds the missing paths, asks AI to identify meaningful testing gaps, then adds tests to strengthen the suite. This matches your original course topics of **Improving Test Coverage with AI** and **Identifying Testing Gaps**. 

# Lab 14 — Use AI to Find Test Coverage Gaps and Strengthen the Suite

## Objectives

By the end of this lab, you will be able to:

* Understand what code coverage means.
* Measure `.NET` unit-test coverage.
* Understand **line coverage** and **branch coverage**.
* Generate an HTML coverage report.
* Identify uncovered lines and branches.
* Give actual coverage evidence to AI.
* Ask AI to identify meaningful testing gaps.
* Add tests for missing boundaries and branches.
* Rerun all tests.
* Measure coverage again.
* Compare **Before vs After coverage**.
* Understand why **100% coverage does not automatically mean good tests**.

Microsoft defines code coverage as a measure of how much code is executed by tests, including lines, branches, or methods. ([Microsoft Learn][1])

---

# Part 1 — The difference between Lab 13 and Lab 14

Think of it this way:

```text
LAB 13

Do my tests PASS?

          ↓

8 tests
8 passed
0 failed

          ↓

GOOD
```

But Lab 14 asks:

```text
LAB 14

What did those 8 tests
ACTUALLY test?

          ↓

OrderService
├── This line tested ✓
├── This branch tested ✓
├── This branch NOT tested ✗
├── This line tested ✓
└── This error path NOT tested ✗
```

That's **coverage analysis**.

---

# Part 2 — What is code coverage?

Suppose you have:

```csharp
if (customerType == "VIP")
{
    return 0.20m;
}
else
{
    return 0.05m;
}
```

If your test only uses:

```text
customerType = VIP
```

then:

```text
VIP branch       ✓ TESTED

Regular branch   ✗ NOT TESTED
```

Even though:

```text
All tests passed
```

you still have a **coverage gap**.

---

# Part 3 — Line coverage vs branch coverage

These are the two most useful concepts for this lab.

### Line Coverage

Question:

> Did a test execute this line?

Example:

```csharp
throw new ArgumentException(
    "Subtotal cannot be negative.");
```

If no test uses a negative subtotal:

```text
That line was never executed.
```

Therefore it is uncovered.

---

### Branch Coverage

Question:

> Did we test both possible routes through a decision?

Example:

```csharp
if (subtotal >= 100)
```

There are two routes:

```text
subtotal >= 100
       ↓
     TRUE
```

and:

```text
subtotal >= 100
       ↓
     FALSE
```

Good testing should consider both where the behavior matters. Microsoft explicitly notes that a program with two conditional branches and a test exercising only one can have 50% branch coverage. ([Microsoft Learn][1])

---

# Part 4 — Start from your Lab 13 project

You should currently have:

```text
C:\AI_SDLC_Labs
│
├── Lab02Api
│
│   ├── Program.cs
│   ├── OrderService.cs
│   ├── CustomerReportService.cs
│   ├── PerformanceService.cs
│   └── Lab02Api.csproj
│
└── Lab02Api.Tests
    ├── OrderServiceTests.cs
    └── Lab02Api.Tests.csproj
```

Open PowerShell:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api.Tests
```

---

# Part 5 — Make sure all existing tests pass

Run:

```powershell
dotnet test
```

You want:

```text
Failed: 0
```

For example:

```text
Passed: 8
Failed: 0
```

Don't start coverage analysis with failing tests.

---

# Part 6 — Collect code coverage

Now run:

```powershell
dotnet test --collect:"XPlat Code Coverage"
```

Microsoft's current code-coverage documentation shows this command for Coverlet-based cross-platform coverage collection. It produces a `coverage.cobertura.xml` result under the test project's `TestResults` directory. ([Microsoft Learn][1])

You should see your tests execute normally.

Then you'll get something similar to:

```text
Test Run Successful.

Attachments:
C:\AI_SDLC_Labs\Lab02Api.Tests\TestResults\
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\
coverage.cobertura.xml
```

Your folder ID will be different.

---

# Part 7 — Find the coverage file

Run:

```powershell
Get-ChildItem -Recurse -Filter coverage.cobertura.xml
```

You should find something like:

```text
Lab02Api.Tests
│
└── TestResults
    └── a1234567-....
        └── coverage.cobertura.xml
```

This XML file contains the measured coverage data. ([Microsoft Learn][1])

But XML is ugly to read.

So we'll create a proper HTML report.

---

# Part 8 — Install ReportGenerator

Run once:

```powershell
dotnet tool install -g dotnet-reportgenerator-globaltool
```

Microsoft's coverage documentation uses ReportGenerator to convert Cobertura coverage output into a human-readable HTML report. ([Microsoft Learn][1])

If it says already installed:

```powershell
dotnet tool update -g dotnet-reportgenerator-globaltool
```

---

# Part 9 — Generate the HTML report

Because the `TestResults` folder contains a random GUID, use:

```powershell
reportgenerator `
  -reports:"TestResults\**\coverage.cobertura.xml" `
  -targetdir:"CoverageReport" `
  -reporttypes:Html
```

You should now have:

```text
Lab02Api.Tests
│
├── OrderServiceTests.cs
│
├── TestResults
│
└── CoverageReport
    ├── index.html
    └── ...
```

Microsoft's documented workflow similarly uses ReportGenerator against `coverage.cobertura.xml` to create an HTML report. ([Microsoft Learn][1])

---

# Part 10 — Open the report

Run:

```powershell
start .\CoverageReport\index.html
```

Your browser should open.

You'll see something broadly like:

```text
Code Coverage Report

Assemblies

Lab02Api

Line Coverage:     xx%
Branch Coverage:   xx%
Method Coverage:   xx%
```

**Use your actual numbers.**

Do not copy example numbers from this guide.

---

# Part 11 — Open `OrderService`

Click:

```text
Lab02Api
    ↓
OrderService
```

Now you'll be able to see which source lines were executed.

Conceptually:

```text
public decimal CalculateFinalAmount(...)
{
 ✓   ValidateSubtotal(subtotal);

 ✓   decimal total =
         ApplyCustomerDiscount(...);

 ✓   if (hasCoupon && customerType != "VIP")
     {
 ✓       total = ApplyCoupon(total);
     }

 ✓   return Math.Round(total, 2);
}
```

Depending on your tests, some code may show:

```text
✓ Covered
```

and others:

```text
✗ Not covered
```

The precise report appearance depends on your tool version, but the important information is which lines/branches were exercised.

---

# Part 12 — Record your BEFORE coverage

Create:

```text
TEST_COVERAGE_BASELINE.md
```

Record:

```markdown
# Lab 14 Test Coverage Baseline

## Test Result

Total Tests: [actual]
Passed: [actual]
Failed: 0

## OrderService Coverage

Line Coverage: [actual]%
Branch Coverage: [actual]%
Method Coverage: [actual]%

## Uncovered Areas

1. [actual uncovered line/branch]
2. [actual uncovered line/branch]
3. [actual uncovered line/branch]
```

This is your baseline.

---

# Part 13 — Don't ask AI to guess coverage

Bad prompt:

```text
Review OrderService.cs and tell me what my
coverage probably is.
```

AI cannot know what actually executed.

Instead give AI:

```text
SOURCE CODE
+
TEST CODE
+
ACTUAL COVERAGE RESULTS
```

Then AI can help interpret them.

---

# Part 14 — Give AI the coverage evidence

In Cursor Chat:

```text
Review:

- OrderService.cs
- OrderServiceTests.cs

My actual coverage report shows:

Line Coverage:
[ACTUAL VALUE]

Branch Coverage:
[ACTUAL VALUE]

Method Coverage:
[ACTUAL VALUE]

Uncovered lines/branches:
[PASTE ACTUAL COVERAGE FINDINGS]

Do NOT modify code yet.

Identify meaningful unit-test coverage gaps.

For each gap provide:

1. Uncovered business behavior
2. Relevant source code
3. Why existing tests do not reach it
4. Recommended test scenario
5. Input values
6. Expected result
7. Priority: High / Medium / Low

Do not recommend tests solely to increase a percentage.
```

That last sentence is important.

---

# Part 15 — Why "don't chase the percentage"?

Suppose this line isn't covered:

```csharp
return customerType == VipCustomerType
    ? VipCustomerType
    : "Regular";
```

A useful test asks:

```text
What actual business behavior
have we failed to test?
```

Not:

```text
How can I make the green bar longer?
```

Coverage is a **measurement tool**, not the objective by itself. Microsoft describes coverage as how much code is executed by tests; it doesn't establish that the assertions or scenarios are correct. ([Microsoft Learn][1])

---

# Part 16 — AI should compare source code against tests

Use this prompt:

```text
Compare every decision branch in OrderService.cs
against OrderServiceTests.cs.

Build a branch coverage matrix.

Columns:

Business Rule
Source Condition
True Branch Tested?
False Branch Tested?
Existing Test
Missing Test
Priority

Do NOT create test code yet.
```

You may get something conceptually like:

| Business Rule            | True | False | Gap  |
| ------------------------ | ---- | ----- | ---- |
| subtotal < 0             | ✓    | ✓     | None |
| VIP customer             | ✓    | ✓     | None |
| subtotal >= 100          | ✓    | ✓     | None |
| hasCoupon                | ✓    | ✓     | None |
| VIP + coupon restriction | ✓    | ✓     | None |

Or you may discover missing paths.

Use your actual code/test suite.

---

# Part 17 — Find boundary gaps

Your code contains an important decision:

```csharp
subtotal >= 100
```

We tested:

```text
50
100
200
```

But ask AI:

```text
Review OrderService's numeric boundaries.

Which values immediately below, exactly at,
and immediately above each threshold are worth testing?

Do not generate code yet.
```

For the RM100 threshold:

```text
Below:  99.99
Exact: 100.00
Above: 100.01
```

That's **boundary-value testing**.

---

# Part 18 — Add a below-boundary test

For example:

```csharp
[Fact]
public void CalculateFinalAmount_VipCustomerJustBelow100_Applies10PercentDiscount()
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            99.99m,
            "VIP",
            false);

    Assert.Equal(89.99m, result);
}
```

Why `89.99`?

The implementation rounds to two decimal places:

```text
99.99 × 90%
= 89.991

Rounded
= 89.99
```

This specifically checks the path just below the threshold.

---

# Part 19 — Add an above-boundary test

```csharp
[Fact]
public void CalculateFinalAmount_VipCustomerJustAbove100_Applies20PercentDiscount()
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            100.01m,
            "VIP",
            false);

    Assert.Equal(80.01m, result);
}
```

Now:

```text
99.99
   ↓
10% VIP discount

100.00
   ↓
20% VIP discount

100.01
   ↓
20% VIP discount
```

That strengthens confidence around the exact business boundary.

---

# Part 20 — Look for zero-value gaps

Ask AI:

```text
Is subtotal = 0 explicitly covered?

Based on OrderService.cs:

1. What should happen?
2. Is zero valid?
3. Which branch executes?
4. Should it have its own test?

Do not modify production code.
```

If the business rule permits zero, a test could be:

```csharp
[Fact]
public void CalculateFinalAmount_ZeroSubtotal_ReturnsZero()
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            0m,
            "Regular",
            false);

    Assert.Equal(0m, result);
}
```

But only add it if the implemented/documented business rule supports that behavior.

---

# Part 21 — Look for unknown customer types

Our code treats anything other than `"VIP"` as Regular.

For example:

```text
customerType = "ABC"
```

What should happen?

This is where AI should **not invent requirements**.

Ask:

```text
OrderService currently treats any customer type
other than "VIP" as a regular customer.

Is that clearly supported by our documented business rules?

Classify this as:

1. Valid behavior already specified
2. Testing gap
3. Requirements gap

Do not modify code or invent a requirement.
```

This might actually be a:

```text
REQUIREMENTS GAP
```

rather than a test-coverage gap.

That's an excellent learning point.

---

# Part 22 — Coverage gaps are not always test gaps

There are three possibilities:

```text
UNCOVERED CODE
      │
      ├── Important behavior
      │       ↓
      │    Add test
      │
      ├── Requirement unclear
      │       ↓
      │    Clarify requirement
      │
      └── Dead/unnecessary code
              ↓
           Review code
```

So don't automatically generate a test for every red line.

---

# Part 23 — Ask AI to prioritize gaps

Use:

```text
Rank the uncovered test scenarios.

Prioritize using:

HIGH
- Financial calculation
- Previous production defect
- Boundary condition
- Error handling
- Security-sensitive behavior

MEDIUM
- Common business scenario
- Secondary branch

LOW
- Trivial implementation
- Defensive branch with little business impact

Explain every priority.

Do not generate code yet.
```

Now AI is helping you decide **where testing effort provides value**.

---

# Part 24 — Generate only approved missing tests

Now:

```text
Generate xUnit tests ONLY for the High and Medium
priority confirmed test gaps.

Requirements:

1. Do not duplicate existing tests.
2. Follow Arrange-Act-Assert.
3. Use descriptive names.
4. Use [Theory] when multiple values test the same rule.
5. Test externally observable behavior.
6. Do not test private methods directly.
7. Do not modify OrderService.cs.
8. Do not change existing expected results.
9. Do not create tests for undefined requirements.
```

Allow Cursor to add the approved tests.

---

# Part 25 — Why not test private methods directly?

Suppose `OrderService` contains:

```text
CalculateFinalAmount()
    ↓
ApplyCustomerDiscount()
    ↓
ApplyCoupon()
```

You normally test the public behavior:

```text
CalculateFinalAmount()
```

rather than tying your tests tightly to the internal structure.

Otherwise a future refactor could:

```text
rename ApplyCoupon
```

and break tests even though the application behavior remains perfectly correct.

This is the difference between:

```text
TEST BUSINESS BEHAVIOR
```

and:

```text
TEST IMPLEMENTATION DETAILS
```

---

# Part 26 — Run the strengthened suite

Run:

```powershell
dotnet test
```

You might now have:

```text
Total:  11
Passed: 11
Failed: 0
```

Use your actual values.

---

# Part 27 — If an AI-generated test fails

Do not immediately change production code.

Use:

```text
This new AI-generated test failed.

Do NOT modify OrderService.cs.

Analyze:

1. Which documented business rule does this test represent?
2. Is the expected result correct?
3. Is the calculation correct?
4. Is the test itself wrong?
5. Could OrderService contain a defect?
6. Could the requirement be ambiguous?

Classify the failure as:

TEST DEFECT
PRODUCTION DEFECT
REQUIREMENT GAP
NEEDS MORE CONTEXT

Show evidence.
```

This is very important.

---

# Part 28 — Measure coverage AGAIN

Once all approved tests pass:

```powershell
dotnet test --collect:"XPlat Code Coverage"
```

This creates another Cobertura coverage result. ([Microsoft Learn][1])

Now regenerate the HTML report:

```powershell
reportgenerator `
  -reports:"TestResults\**\coverage.cobertura.xml" `
  -targetdir:"CoverageReportAfter" `
  -reporttypes:Html
```

Open:

```powershell
start .\CoverageReportAfter\index.html
```

---

# Part 29 — Compare BEFORE vs AFTER

Now record:

| Metric          | Before |  After |
| --------------- | -----: | -----: |
| Tests           | actual | actual |
| Passed          | actual | actual |
| Line Coverage   | actual | actual |
| Branch Coverage | actual | actual |
| Method Coverage | actual | actual |

For example only:

```text
BEFORE

Tests:            8
Line Coverage:   78%
Branch Coverage: 65%

AFTER

Tests:           11
Line Coverage:   91%
Branch Coverage: 88%
```

**Do not use these example numbers.**

Use your actual report.

---

# Part 30 — Ask AI to interpret improvement

Give Cursor your real metrics:

```text
Compare my actual test coverage results.

BEFORE

Tests:
[...]

Line Coverage:
[...]

Branch Coverage:
[...]

AFTER

Tests:
[...]

Line Coverage:
[...]

Branch Coverage:
[...]

Explain:

1. Absolute improvement.
2. Which important gaps were closed.
3. Which gaps remain.
4. Whether remaining uncovered areas matter.
5. Whether any new tests appear redundant.

Do not recommend tests merely to reach 100%.
```

---

# Part 31 — 100% coverage is NOT the objective

Suppose:

```text
Line Coverage = 100%
```

Does that mean:

```text
Perfect software?
```

**No.**

Imagine this useless test:

```csharp
[Fact]
public void Test()
{
    var service = new OrderService();

    service.CalculateFinalAmount(
        200m,
        "VIP",
        false);
}
```

It executes the code.

But there's no:

```csharp
Assert.Equal(...)
```

So it might increase coverage without verifying the result.

That's why:

```text
HIGH COVERAGE
      +
WEAK ASSERTIONS
      =
WEAK TEST SUITE
```

Coverage helps identify untested areas, but the test design and assertions still determine whether behavior is meaningfully verified. ([Microsoft Learn][1])

---

# Part 32 — Challenge the strengthened suite

Ask:

```text
Act as a Senior .NET QA Engineer.

Review the final OrderServiceTests.cs.

Do NOT modify anything.

For every test classify:

KEEP
IMPROVE
REMOVE

Assess:

1. Does it validate a real business rule?
2. Is the expected value correct?
3. Is it redundant?
4. Does it cover a meaningful boundary?
5. Does it test a previous defect?
6. Does it have a meaningful assertion?
7. Could it pass while the behavior is wrong?

Then identify remaining high-risk test gaps.
```

---

# Part 33 — Create the Lab 14 report

Create:

```text
TEST_COVERAGE_IMPROVEMENT.md
```

Ask Cursor:

```text
Create TEST_COVERAGE_IMPROVEMENT.md using only
the actual Lab 14 results.

Structure:

# Lab 14 — Test Coverage Gap Analysis

## 1. Baseline

Tests:
Passed:
Failed:

Line Coverage:
Branch Coverage:
Method Coverage:

## 2. Initial Coverage Gaps

List actual uncovered areas.

## 3. AI Gap Analysis

For each:
- Source condition
- Business behavior
- Existing coverage
- Missing scenario
- Priority

## 4. Additional Tests Added

Test name
Purpose
Expected behavior

## 5. Validation

dotnet test result.

## 6. Coverage After Improvement

Tests:
Line Coverage:
Branch Coverage:
Method Coverage:

## 7. Before vs After

Table:
Metric
Before
After
Change

## 8. Remaining Gaps

List only genuine remaining gaps.

## 9. Requirements Gaps

Document behavior that cannot be tested correctly
without clarifying requirements.

## 10. Conclusion

Explain how AI helped identify meaningful coverage gaps
without treating 100% coverage as the objective.

Do not invent metrics.
```

---

# Lab 14 Completion Checklist

* [ ] Lab 13 tests pass.
* [ ] `dotnet test --collect:"XPlat Code Coverage"` executed.
* [ ] `coverage.cobertura.xml` generated.
* [ ] ReportGenerator installed.
* [ ] HTML coverage report created.
* [ ] Line coverage recorded.
* [ ] Branch coverage recorded.
* [ ] Method coverage recorded.
* [ ] `OrderService` coverage inspected.
* [ ] Actual uncovered lines identified.
* [ ] Actual uncovered branches identified.
* [ ] AI receives real coverage evidence.
* [ ] AI builds a branch/test-gap matrix.
* [ ] Boundary-value gaps reviewed.
* [ ] Error paths reviewed.
* [ ] Previous regression path reviewed.
* [ ] Requirements gaps distinguished from testing gaps.
* [ ] Missing tests prioritized.
* [ ] Only meaningful additional tests generated.
* [ ] AI-generated tests reviewed.
* [ ] All approved tests pass.
* [ ] Coverage collected again.
* [ ] Before/after coverage compared.
* [ ] Remaining gaps documented.
* [ ] `TEST_COVERAGE_IMPROVEMENT.md` created.

## The easiest way to remember Labs 13 and 14

```text
                 LAB 13
                    │
                    ▼
            Write Unit Tests
                    │
                    ▼
               dotnet test
                    │
                    ▼
             Do they PASS?
                    │
                   YES
                    │
                    ▼
                 LAB 14
                    │
                    ▼
            Measure Coverage
                    │
                    ▼
          What DIDN'T we test?
                    │
           ┌────────┼────────┐
           ▼        ▼        ▼
        Branch   Boundary   Error
          Gap      Gap      Path
           │        │        │
           └────────┼────────┘
                    ▼
             AI Gap Analysis
                    │
                    ▼
          Human Selects Useful Tests
                    │
                    ▼
           Add Missing Tests
                    │
                    ▼
               dotnet test
                    │
                    ▼
            Measure Coverage Again
                    │
                    ▼
             BEFORE vs AFTER
```

So the key distinction is:

**Lab 13 = “Can AI help me generate correct unit tests?”**

**Lab 14 = “How do I know what those tests still failed to cover, and can AI help me intelligently close the important gaps?”**

[1]: https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage?utm_source=chatgpt.com "Use code coverage for unit testing - .NET | Microsoft Learn"
