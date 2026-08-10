**Lab 13 is much easier than SonarQube once you understand one idea: a unit test is simply C# code that automatically checks whether another piece of C# code returns the expected answer.**

Your original course specifically calls for **Unit Testing Principles, Designing Effective Unit Tests, .NET Unit Test Class Development, Creating and Maintaining Test Suites, AI-Assisted Unit Test Generation, Improving Test Coverage, and Identifying Testing Gaps**. 

For Lab 13, I recommend testing the `OrderService.cs` you created earlier because its business rules are simple and measurable.

# Lab 13 — Generate Effective .NET Unit Tests with AI

## Objectives

By the end of Lab 13, you should be able to:

* Understand what a unit test is.
* Create a `.NET` xUnit test project.
* Connect the test project to `Lab02Api`.
* Write and run your first test.
* Understand `Arrange → Act → Assert`.
* Use AI to identify test scenarios.
* Use AI to generate `[Fact]` and `[Theory]` tests.
* Test normal cases, boundaries, and invalid input.
* Detect a failing test.
* Challenge AI-generated tests.
* Run the complete test suite with `dotnet test`.

Microsoft's current .NET documentation uses `dotnet new xunit` to create an xUnit test project and `dotnet test` to execute its tests. The test project then references the source project being tested. ([Microsoft Learn][1])

---

# Part 1 — What is a unit test?

Remember your `OrderService`?

You give it:

```text
Subtotal: RM200
Customer: VIP
Coupon: Yes
```

According to our business rule:

```text
VIP >= RM100
→ 20% discount

VIP cannot use coupon

RM200 × 80%
= RM160
```

Normally, you tested this manually through the browser:

```text
/price?subtotal=200&customerType=VIP&hasCoupon=true
```

and checked:

```text
total = 160
```

A **unit test automates that checking**.

Conceptually:

```text
             UNIT TEST
                 │
                 ▼
          Create OrderService
                 │
                 ▼
       Give it RM200 / VIP / Coupon
                 │
                 ▼
       CalculateFinalAmount()
                 │
                 ▼
       Actual result = RM160?
              /       \
            YES        NO
             │          │
            PASS       FAIL
```

---

# Part 2 — Unit Test vs manually testing the API

Previously you did:

```text
Browser
   ↓
/price
   ↓
Program.cs
   ↓
OrderService
   ↓
Result
```

That's useful, but it involves several pieces.

A unit test will directly test:

```text
OrderService
     ↓
CalculateFinalAmount()
     ↓
Result
```

No browser.

No URL.

No SonarQube.

No database.

Just the class we want to test.

---

# Part 3 — Check your application first

Open PowerShell:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Run:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

Because:

```text
Application broken before testing
        ↓
Don't start blaming the tests
```

---

# Part 4 — Go up one folder

Run:

```powershell
cd ..
```

You should now be at:

```text
C:\AI_SDLC_Labs
```

Check:

```powershell
dir
```

You should see:

```text
Lab02Api
```

---

# Part 5 — Create the test project

Run:

```powershell
dotnet new xunit -o Lab02Api.Tests
```

This creates a separate xUnit test project. Microsoft's current tutorial uses the same `dotnet new xunit` approach for creating a unit-test project. ([Microsoft Learn][1])

You should now have:

```text
C:\AI_SDLC_Labs
│
├── Lab02Api
│   ├── Program.cs
│   ├── OrderService.cs
│   ├── CustomerReportService.cs
│   └── Lab02Api.csproj
│
└── Lab02Api.Tests
    ├── UnitTest1.cs
    └── Lab02Api.Tests.csproj
```

This separation is intentional:

```text
Lab02Api
      ↓
REAL APPLICATION

Lab02Api.Tests
      ↓
CODE THAT TESTS
THE REAL APPLICATION
```

---

# Part 6 — Connect the test project to your application

Right now:

```text
Lab02Api.Tests
```

doesn't know what:

```text
OrderService
```

is.

We must add a **project reference**.

Run:

```powershell
dotnet add .\Lab02Api.Tests\Lab02Api.Tests.csproj reference .\Lab02Api\Lab02Api.csproj
```

Microsoft's unit-testing guidance uses a project reference so the test project can access the source code it needs to test. ([Microsoft Learn][1])

You should see something similar to:

```text
Reference '..\Lab02Api\Lab02Api.csproj'
added to the project.
```

---

# Part 7 — Open the parent folder in Cursor

Instead of opening only:

```text
Lab02Api
```

open:

```text
C:\AI_SDLC_Labs
```

In Cursor:

**File → Open Folder → `C:\AI_SDLC_Labs`**

Now Cursor should show:

```text
AI_SDLC_Labs
│
├── Lab02Api
│
└── Lab02Api.Tests
```

Excellent.

Now you can work on the application and tests together.

---

# Part 8 — Delete the meaningless sample test

Inside:

```text
Lab02Api.Tests
```

you will probably have:

```text
UnitTest1.cs
```

Open it.

It may resemble:

```csharp
namespace Lab02Api.Tests;

public class UnitTest1
{
    [Fact]
    public void Test1()
    {
    }
}
```

This test doesn't actually test anything useful.

Delete:

```text
UnitTest1.cs
```

We'll create meaningful tests.

---

# Part 9 — Create `OrderServiceTests.cs`

Right-click:

```text
Lab02Api.Tests
```

Choose:

**New File**

Create:

```text
OrderServiceTests.cs
```

---

# Part 10 — Write your first test

Paste:

```csharp
public class OrderServiceTests
{
    [Fact]
    public void CalculateFinalAmount_VipCustomerOver100_Returns20PercentDiscount()
    {
        // Arrange
        var service = new OrderService();

        // Act
        decimal result =
            service.CalculateFinalAmount(
                200m,
                "VIP",
                false);

        // Assert
        Assert.Equal(160m, result);
    }
}
```

Save it.

---

# Part 11 — Understand `[Fact]`

This:

```csharp
[Fact]
```

means:

> This method is a test that xUnit should execute.

xUnit uses test attributes such as `[Fact]`, while `dotnet test` discovers and executes the test project. ([xUnit.net][2])

So:

```csharp
[Fact]
public void CalculateFinalAmount...
```

means:

```text
xUnit:
"Hey, this method is a test.
Run it."
```

---

# Part 12 — Understand Arrange, Act, Assert

This is probably the most important unit-testing pattern for you to remember.

## ARRANGE

Prepare everything:

```csharp
var service = new OrderService();
```

Think:

```text
Arrange
↓
Get things ready
```

---

## ACT

Perform the thing being tested:

```csharp
decimal result =
    service.CalculateFinalAmount(
        200m,
        "VIP",
        false);
```

Think:

```text
Act
↓
Do something
```

---

## ASSERT

Check the answer:

```csharp
Assert.Equal(160m, result);
```

Think:

```text
Assert
↓
Was the answer correct?
```

Therefore:

```text
ARRANGE
Prepare

   ↓

ACT
Execute

   ↓

ASSERT
Check
```

I recommend students remember simply:

> **Prepare → Do → Check**

---

# Part 13 — Run your first unit test

Open PowerShell:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api.Tests
```

Run:

```powershell
dotnet test
```

`dotnet test` builds and runs the tests; Microsoft's xUnit tutorial uses it as the standard CLI workflow. ([Microsoft Learn][1])

You want something resembling:

```text
Test summary:
total: 1
failed: 0
succeeded: 1
```

The exact wording depends on your SDK/test-runner version.

The important part is:

```text
Passed: 1
Failed: 0
```

Congratulations.

You just created a real automated unit test.

---

# Part 14 — Prove that the test actually works

Now deliberately change:

```csharp
Assert.Equal(160m, result);
```

to:

```csharp
Assert.Equal(150m, result);
```

Run:

```powershell
dotnet test
```

This time the test should:

```text
FAIL
```

because:

```text
Expected:
150

Actual:
160
```

This is important.

A test that never fails isn't very useful.

Put it back:

```csharp
Assert.Equal(160m, result);
```

Run:

```powershell
dotnet test
```

Now:

```text
PASS
```

---

# Part 15 — Now bring AI into Lab 13

This is where the actual **AI-assisted testing** starts.

Do **not** immediately tell Cursor:

```text
Generate tests.
```

Instead ask it to understand the business rules.

Use:

```text
Review OrderService.cs.

Do NOT generate test code yet.

I am learning .NET unit testing.

Extract all business rules implemented by
CalculateFinalAmount().

For every rule identify:

1. Input condition
2. Expected behavior
3. Boundary values
4. Invalid input
5. Important edge cases

Return a test scenario table.

Do not modify source code.
```

AI should discover rules such as:

```text
VIP ≥ 100
→ 20%

VIP < 100
→ 10%

Regular ≥ 100
→ 5%

Regular < 100
→ 0%

Regular + coupon
→ additional 10%

VIP + coupon
→ coupon ignored

Negative subtotal
→ exception
```

---

# Part 16 — Build a test matrix BEFORE generating code

Ask Cursor:

```text
Create a unit-test matrix for OrderService.

Do NOT generate C# yet.

Columns:

Test ID
Scenario
Subtotal
Customer Type
Has Coupon
Expected Result
Why This Test Matters

Include:

- normal cases
- boundary cases
- invalid input
- business-rule interactions

Avoid duplicate tests.
```

You might get something like:

| Test              | Input           |  Expected |
| ----------------- | --------------- | --------: |
| VIP ≥100          | 200/VIP/No      |       160 |
| VIP <100          | 50/VIP/No       |        45 |
| Regular ≥100      | 200/Regular/No  |       190 |
| Regular <100      | 50/Regular/No   |        50 |
| Regular coupon    | 200/Regular/Yes |       171 |
| VIP coupon        | 200/VIP/Yes     |       160 |
| Negative subtotal | -1              | Exception |

This is much better than asking AI to generate 50 random tests.

---

# Part 17 — Test #2: VIP below RM100

Add:

```csharp
[Fact]
public void CalculateFinalAmount_VipCustomerBelow100_Returns10PercentDiscount()
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            50m,
            "VIP",
            false);

    Assert.Equal(45m, result);
}
```

Because:

```text
RM50
− 10%
= RM45
```

---

# Part 18 — Test #3: Regular customer over RM100

Add:

```csharp
[Fact]
public void CalculateFinalAmount_RegularCustomerOver100_Returns5PercentDiscount()
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            200m,
            "Regular",
            false);

    Assert.Equal(190m, result);
}
```

Because:

```text
RM200
− 5%
= RM190
```

---

# Part 19 — Test #4: Regular below RM100

Add:

```csharp
[Fact]
public void CalculateFinalAmount_RegularCustomerBelow100_ReturnsNoDiscount()
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            50m,
            "Regular",
            false);

    Assert.Equal(50m, result);
}
```

---

# Part 20 — Test coupon behavior

This business rule deserves a test.

```csharp
[Fact]
public void CalculateFinalAmount_RegularCustomerWithCoupon_AppliesAdditional10PercentDiscount()
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            200m,
            "Regular",
            true);

    Assert.Equal(171m, result);
}
```

Why `171`?

```text
RM200

Regular ≥ 100
↓
5% discount

RM200 × 0.95
= RM190

Coupon
↓
additional 10%

RM190 × 0.90
= RM171
```

---

# Part 21 — Test the bug you fixed in Lab 3

This is particularly valuable.

Remember the Lab 3 bug?

```text
VIP + Coupon

Expected:
RM160

Buggy result:
RM144
```

Create:

```csharp
[Fact]
public void CalculateFinalAmount_VipCustomerWithCoupon_DoesNotStackCoupon()
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            200m,
            "VIP",
            true);

    Assert.Equal(160m, result);
}
```

This test prevents someone from accidentally reintroducing the old bug later.

That's called a **regression test**.

Conceptually:

```text
Old Bug
   ↓
Fix
   ↓
Write Unit Test
   ↓
Future developer changes code
   ↓
Bug comes back?
     /      \
   YES       NO
    ↓         ↓
 TEST FAIL   PASS
```

This is one of the most valuable uses of unit tests.

---

# Part 22 — Test invalid input

Your `OrderService` contains:

```csharp
if (subtotal < 0)
{
    throw new ArgumentException(
        "Subtotal cannot be negative.");
}
```

So create:

```csharp
[Fact]
public void CalculateFinalAmount_NegativeSubtotal_ThrowsArgumentException()
{
    var service = new OrderService();

    Assert.Throws<ArgumentException>(() =>
        service.CalculateFinalAmount(
            -1m,
            "Regular",
            false));
}
```

This test expects:

```text
ArgumentException
```

If no exception occurs:

```text
TEST FAILS
```

---

# Part 23 — Run all tests

Run:

```powershell
dotnet test
```

Ideally:

```text
Total tests: 7
Passed:      7
Failed:      0
```

Your exact count depends on what you've added.

Now you have a **test suite**.

---

# Part 24 — Test boundaries, not just obvious values

Here AI becomes useful.

Your business rule says:

```csharp
subtotal >= 100
```

What values are especially important?

Not:

```text
500
600
700
```

Those mostly test the same condition.

More useful:

```text
99.99
100.00
100.01
```

Because the decision changes at:

```text
100
```

That's called a **boundary**.

Ask AI:

```text
Identify the boundary values in
OrderService.CalculateFinalAmount().

Explain why each boundary deserves testing.

Do not generate code yet.
```

AI should identify:

```text
subtotal = 100
```

and possibly:

```text
subtotal = 0
subtotal < 0
```

---

# Part 25 — Test exactly RM100

Create:

```csharp
[Fact]
public void CalculateFinalAmount_VipCustomerExactly100_Applies20PercentDiscount()
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            100m,
            "VIP",
            false);

    Assert.Equal(80m, result);
}
```

Why is this important?

Because the code says:

```csharp
subtotal >= 100
```

not:

```csharp
subtotal > 100
```

One character:

```text
>=
vs
>
```

could create a boundary bug.

---

# Part 26 — Use `[Theory]` to test multiple inputs

Writing separate tests for every similar number can become repetitive.

xUnit supports parameterized tests using `[Theory]` with data such as `[InlineData]`. ([xUnit.net][3])

Instead of:

```text
Test RM200
Test RM300
Test RM500
```

you can write one test structure.

For example:

```csharp
[Theory]
[InlineData(100, 80)]
[InlineData(200, 160)]
[InlineData(500, 400)]
public void CalculateFinalAmount_VipCustomerOverOrEqual100_Applies20PercentDiscount(
    decimal subtotal,
    decimal expected)
{
    var service = new OrderService();

    decimal result =
        service.CalculateFinalAmount(
            subtotal,
            "VIP",
            false);

    Assert.Equal(expected, result);
}
```

Now xUnit executes the same test logic three times.

Think:

```text
THEORY

Same business rule
      │
      ├── Data 1
      ├── Data 2
      └── Data 3
```

---

# Part 27 — `[Fact]` vs `[Theory]`

Easy way to remember:

```text
[Fact]

One scenario
One fixed input
One expected result
```

Example:

```text
Negative subtotal must throw exception.
```

versus:

```text
[Theory]

Same rule
Multiple inputs
Multiple expected results
```

Example:

```text
VIP 100 → 80
VIP 200 → 160
VIP 500 → 400
```

---

# Part 28 — Now let AI generate tests

Once you have understood the basic structure, use this prompt:

```text
Review OrderService.cs and OrderServiceTests.cs.

Generate additional xUnit unit tests ONLY for
meaningful testing gaps.

Requirements:

1. Do not duplicate existing tests.
2. Test business behavior, not implementation details.
3. Include boundary cases.
4. Include invalid input.
5. Include important interactions between rules.
6. Use [Theory] where several cases test the same rule.
7. Use descriptive test names.
8. Follow Arrange-Act-Assert.
9. Do not modify OrderService.cs.
10. Do not change expected results merely to make tests pass.

Before generating code, list the missing test scenarios
and explain why each is necessary.
```

This is a far better AI prompt than:

```text
Write unit tests for my code.
```

---

# Part 29 — Very important: don't let AI change production code

Suppose an AI-generated test fails.

Do **not** immediately allow AI to modify:

```text
OrderService.cs
```

Ask:

```text
This AI-generated unit test failed.

Do NOT change production code yet.

Analyze:

1. Is the expected result in the test correct?
2. Does the test match the documented business rule?
3. Could the test itself be wrong?
4. Could the production code contain a defect?
5. What evidence would distinguish the two?

Show your reasoning using the source code and
business rules.
```

Because sometimes:

```text
FAIL
```

means:

```text
PRODUCTION CODE BUG
```

but sometimes it means:

```text
AI WROTE A BAD TEST
```

---

# Part 30 — Example of a bad AI test

Suppose AI generates:

```csharp
[Fact]
public void VipCustomerWithCoupon_Returns144()
{
    ...
    Assert.Equal(144m, result);
}
```

But our documented business rule says:

```text
VIP coupon cannot stack.
```

Correct:

```text
160
```

The AI has encoded the **old bug as the expected behavior**.

That's dangerous.

A passing test is not automatically a correct test.

---

# Part 31 — Challenge AI's tests

After AI generates tests, use:

```text
Act as a senior .NET test engineer.

Review OrderServiceTests.cs.

Do NOT modify anything.

For every test determine:

1. Which business rule it validates.
2. Whether its expected result is mathematically correct.
3. Whether it duplicates another test.
4. Whether it tests behavior or implementation details.
5. Whether the test could pass for the wrong reason.
6. Whether an important boundary is missing.

Classify every test:

KEEP
IMPROVE
REMOVE
```

This is one of the key learning objectives of **AI-assisted testing**.

---

# Part 32 — Test names matter

Avoid:

```csharp
Test1()
Test2()
TestDiscount()
```

Prefer:

```csharp
CalculateFinalAmount_VipCustomerWithCoupon_DoesNotStackCoupon()
```

The name tells you:

```text
METHOD
CalculateFinalAmount

SCENARIO
VIP customer with coupon

EXPECTED BEHAVIOR
Coupon does not stack
```

A good test name almost reads like documentation.

---

# Part 33 — Your complete test class

At this stage, yours might resemble:

```csharp
public class OrderServiceTests
{
    [Fact]
    public void CalculateFinalAmount_VipCustomerOver100_Returns20PercentDiscount()
    {
        var service = new OrderService();

        decimal result =
            service.CalculateFinalAmount(
                200m,
                "VIP",
                false);

        Assert.Equal(160m, result);
    }

    [Fact]
    public void CalculateFinalAmount_VipCustomerBelow100_Returns10PercentDiscount()
    {
        var service = new OrderService();

        decimal result =
            service.CalculateFinalAmount(
                50m,
                "VIP",
                false);

        Assert.Equal(45m, result);
    }

    [Fact]
    public void CalculateFinalAmount_RegularCustomerOver100_Returns5PercentDiscount()
    {
        var service = new OrderService();

        decimal result =
            service.CalculateFinalAmount(
                200m,
                "Regular",
                false);

        Assert.Equal(190m, result);
    }

    [Fact]
    public void CalculateFinalAmount_RegularCustomerBelow100_ReturnsNoDiscount()
    {
        var service = new OrderService();

        decimal result =
            service.CalculateFinalAmount(
                50m,
                "Regular",
                false);

        Assert.Equal(50m, result);
    }

    [Fact]
    public void CalculateFinalAmount_RegularCustomerWithCoupon_AppliesAdditional10PercentDiscount()
    {
        var service = new OrderService();

        decimal result =
            service.CalculateFinalAmount(
                200m,
                "Regular",
                true);

        Assert.Equal(171m, result);
    }

    [Fact]
    public void CalculateFinalAmount_VipCustomerWithCoupon_DoesNotStackCoupon()
    {
        var service = new OrderService();

        decimal result =
            service.CalculateFinalAmount(
                200m,
                "VIP",
                true);

        Assert.Equal(160m, result);
    }

    [Fact]
    public void CalculateFinalAmount_NegativeSubtotal_ThrowsArgumentException()
    {
        var service = new OrderService();

        Assert.Throws<ArgumentException>(() =>
            service.CalculateFinalAmount(
                -1m,
                "Regular",
                false));
    }

    [Fact]
    public void CalculateFinalAmount_VipCustomerExactly100_Applies20PercentDiscount()
    {
        var service = new OrderService();

        decimal result =
            service.CalculateFinalAmount(
                100m,
                "VIP",
                false);

        Assert.Equal(80m, result);
    }
}
```

---

# Part 34 — Run your complete test suite

From:

```text
C:\AI_SDLC_Labs\Lab02Api.Tests
```

run:

```powershell
dotnet test
```

Your target is:

```text
Failed: 0
```

If you have eight tests:

```text
Total:  8
Passed: 8
Failed: 0
```

Again, the exact console format depends on your installed SDK and test runner. ([xUnit.net][3])

---

# Part 35 — The most important testing lesson

Do **not** measure AI test quality by:

```text
AI generated 50 tests!
```

That's meaningless.

Better questions are:

```text
Did we test every business rule?

Did we test boundaries?

Did we test failure cases?

Did we test the previous bug?

Are expected results correct?

Would a real defect make the tests fail?
```

For example:

```text
100 meaningless tests
<
8 meaningful tests
```

---

# Part 36 — Unit tests become your safety net

Now imagine Lab 5-style refactoring again:

```text
OrderService
      ↓
AI refactors code
      ↓
Looks cleaner
```

Before unit tests you had to manually call URLs.

Now:

```powershell
dotnet test
```

And immediately:

```text
8 passed
```

or:

```text
7 passed
1 failed
```

So:

```text
                AI REFACTORING
                      │
                      ▼
                  CODE CHANGE
                      │
                      ▼
                  dotnet test
                      │
             ┌────────┴────────┐
             ▼                 ▼
          ALL PASS          FAILURE
             │                 │
             ▼                 ▼
        More confidence     Investigate
        behavior intact     regression
```

That's why automated tests matter in an AI-powered SDLC.

---

# Part 37 — Do not chase coverage yet

You may remember SonarQube showing:

```text
Coverage = 0%
```

Don't worry yet.

**Lab 13's primary goal is writing meaningful tests.**

Microsoft's .NET tooling can collect coverage through tools such as Coverlet, and the standard xUnit project template has commonly included coverage support. ([Microsoft Learn][4])

But I recommend keeping the sequence clean:

```text
LAB 13
Can we create GOOD unit tests?
        ↓

Next testing lab
How much code do those tests cover?
        ↓

Later
Send coverage to SonarQube
```

Otherwise students start chasing:

```text
90% coverage
```

without understanding whether their tests are actually useful.

---

# Lab 13 Completion Checklist

* [ ] Existing `Lab02Api` builds.
* [ ] `Lab02Api.Tests` xUnit project created.
* [ ] Test project references `Lab02Api`.
* [ ] `OrderServiceTests.cs` created.
* [ ] First `[Fact]` test written.
* [ ] Arrange → Act → Assert understood.
* [ ] Test deliberately failed once to verify it works.
* [ ] VIP ≥ RM100 tested.
* [ ] VIP < RM100 tested.
* [ ] Regular ≥ RM100 tested.
* [ ] Regular < RM100 tested.
* [ ] Coupon behavior tested.
* [ ] Previous VIP coupon bug covered by regression test.
* [ ] Negative subtotal tested.
* [ ] Boundary at exactly RM100 tested.
* [ ] AI generates a test-scenario matrix before code.
* [ ] AI identifies testing gaps.
* [ ] `[Theory]` understood for multiple related inputs.
* [ ] AI-generated tests reviewed by a human.
* [ ] No expected result changed merely to get green.
* [ ] `dotnet test` reports all approved tests passing.

## The easiest way to remember Lab 13

```text
                 BUSINESS RULE
                       │
                       ▼
               What should happen?
                       │
                       ▼
                 TEST SCENARIO
                       │
                       ▼
              AI suggests tests
                       │
                       ▼
                 Human reviews
                       │
                       ▼
              Write xUnit Test
                       │
                ┌──────┼──────┐
                ▼      ▼      ▼
             Arrange   Act   Assert
                │      │      │
                └──────┼──────┘
                       ▼
                  dotnet test
                       │
                ┌──────┴──────┐
                ▼             ▼
              PASS           FAIL
                │             │
                ▼             ▼
        Expected behavior   Investigate
          confirmed        code OR test
```

And the progression from SonarQube into testing is now very logical:

```text
LAB 10
SonarQube:
"Your project has quality problems / low coverage."

        ↓

LAB 11
"Why are these findings happening?"

        ↓

LAB 12
"Fix confirmed SonarQube issues."

        ↓

LAB 13
"Now create automated tests so future AI/code
changes don't silently break the business rules."
```

**For teaching, I strongly prefer starting Lab 13 with `OrderService` rather than asking AI to generate tests for the entire Web API.** It lets participants clearly understand input → expected result → assertion first. Once they understand unit tests, then you can move them into test coverage, API/integration tests, mocking, and SonarQube coverage without overwhelming them.

[1]: https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-csharp-with-xunit "https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-csharp-with-xunit"
[2]: https://xunit.net/docs/getting-started/v2/getting-started "https://xunit.net/docs/getting-started/v2/getting-started"
[3]: https://xunit.net/docs/getting-started/v3/getting-started "https://xunit.net/docs/getting-started/v3/getting-started"
[4]: https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage "https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage"
