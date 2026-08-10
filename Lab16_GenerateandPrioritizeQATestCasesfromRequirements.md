**Lab 16 is not about writing more unit-test code yet.** It teaches you to take a **business requirement written in English** and turn it into a professional QA test plan.

Your original course places this under **QA Automation with Artificial Intelligence**, including *Test Case Design Principles, AI-Assisted Test Case Generation, Test Scenario Identification and Prioritization, Test Data Creation, Defect Analysis, Coverage Analysis and Gap Detection*. 

The easiest way to remember Lab 16 is:

```text
REQUIREMENT
    ↓
"What should the system do?"
    ↓
AI extracts business rules
    ↓
AI generates test scenarios
    ↓
Human checks them
    ↓
Prioritize P0 / P1 / P2
    ↓
Detailed QA test cases
    ↓
Requirement ↔ Test traceability
```

# Lab 16 — Generate and Prioritize QA Test Cases from Requirements

## Objectives

By the end of this lab, you will be able to:

* Read a business requirement.
* Identify testable requirements.
* Detect unclear or missing requirements.
* Ask AI to generate test scenarios.
* Separate positive, negative, boundary and error tests.
* Turn scenarios into detailed QA test cases.
* Prioritize tests according to business risk.
* Generate suitable test data.
* Challenge AI-generated test cases.
* Create a Requirements Traceability Matrix.
* Produce a reusable QA test-case document.

---

# Part 1 — Unit Test vs QA Test Case

This is important because you just completed Labs 13–15.

### Lab 13

You wrote code like:

```csharp
Assert.Equal(160m, result);
```

That is a:

```text
UNIT TEST
```

### Lab 16

You instead create something like:

| Step | Action                  | Expected Result           |
| ---- | ----------------------- | ------------------------- |
| 1    | Open `/discount?age=65` | API responds successfully |
| 2    | Inspect discount        | Discount = 0.20           |

That's a:

```text
QA TEST CASE
```

So:

```text
UNIT TEST
Developer-oriented
Tests code directly

        vs

QA TEST CASE
Business/user-oriented
Tests system behavior
```

---

# Part 2 — Start with a requirement

Create this file in Cursor:

```text
DISCOUNT_REQUIREMENTS.md
```

For Lab 16, use this requirement:

```text
Discount Eligibility Requirements

R1.
The system shall provide a discount API.

R2.
The API accepts the customer's age.

R3.
Customers aged 60 years or older shall receive
a 20% discount.

R4.
Customers younger than 60 shall receive
no discount.

R5.
Age must not be negative.

R6.
Age greater than 120 shall be considered invalid.

R7.
Invalid age input shall return an appropriate
validation error.

R8.
The API response shall include:
- age
- discount

R9.
Discount shall be represented as:
20% = 0.20
No discount = 0
```

We're deliberately giving AI something concrete.

---

# Part 3 — Understand what QA does first

Do **not** immediately ask AI:

```text
Generate 100 test cases.
```

First:

```text
Requirements
     ↓
Understand them
     ↓
Identify business rules
     ↓
Identify ambiguity
     ↓
Then generate tests
```

---

# Part 4 — Ask AI to analyze the requirements

In Cursor Chat:

```text
You are a Senior QA Test Engineer.

Review DISCOUNT_REQUIREMENTS.md.

Do NOT generate test cases yet.

Extract every testable requirement.

For each requirement provide:

1. Requirement ID
2. Requirement description
3. Business rule
4. Input
5. Expected output
6. Validation rule
7. Testability
8. Any ambiguity or missing information

Do not invent requirements.
```

AI should produce something resembling:

| ID | Rule                           | Testable? |
| -- | ------------------------------ | --------- |
| R1 | Discount API exists            | Yes       |
| R2 | Accept age                     | Yes       |
| R3 | Age ≥60 → 20%                  | Yes       |
| R4 | Age <60 → 0%                   | Yes       |
| R5 | Age cannot be negative         | Yes       |
| R6 | Age >120 invalid               | Yes       |
| R7 | Invalid age returns error      | Yes       |
| R8 | Response includes age/discount | Yes       |
| R9 | 20%=0.20                       | Yes       |

---

# Part 5 — Ask AI to identify unclear requirements

Use:

```text
Review the requirements specifically for ambiguity.

Do NOT invent missing requirements.

Identify anything that QA cannot determine confidently.

For every ambiguity provide:

1. Requirement ID
2. Missing information
3. Why it matters for testing
4. Question that should be asked to Product Owner
```

For example, AI might notice:

```text
R7 says:

"appropriate validation error"

But what exactly does that mean?
```

Questions could include:

```text
Should invalid age return HTTP 400?

What JSON error format should be used?

Is decimal age allowed?

What happens when age is missing?

What happens when age contains letters?
```

These are genuine **requirements gaps**.

---

# Part 6 — Don't let AI silently invent answers

Suppose requirement says:

```text
Invalid input shall return an appropriate error.
```

AI might invent:

```text
HTTP 422
```

But nothing says `422`.

Your prompt should say:

```text
Do not assume an HTTP status code unless it is
explicitly specified.

Mark undefined behavior as REQUIREMENT GAP.
```

This is a crucial QA skill.

---

# Part 7 — Generate high-level test scenarios

Now ask:

```text
Generate high-level QA test scenarios from
DISCOUNT_REQUIREMENTS.md.

Do not produce detailed execution steps yet.

Include:

- Positive scenarios
- Negative scenarios
- Boundary scenarios
- Invalid-input scenarios
- Response-validation scenarios

For every scenario provide:

Scenario ID
Requirement ID
Scenario
Input
Expected result
Test type

Do not invent undefined behavior.
```

---

# Part 8 — Expected test scenarios

You should get scenarios approximately like:

| ID   | Requirement | Input   | Expected                |
| ---- | ----------- | ------- | ----------------------- |
| TS01 | R3          | Age 65  | 0.20                    |
| TS02 | R3          | Age 60  | 0.20                    |
| TS03 | R4          | Age 59  | 0                       |
| TS04 | R4          | Age 25  | 0                       |
| TS05 | R5          | Age -1  | Validation error        |
| TS06 | R6          | Age 121 | Validation error        |
| TS07 | R6          | Age 120 | Valid                   |
| TS08 | R8          | Age 65  | age + discount returned |

Notice:

```text
60
59
120
121
```

These are particularly useful because they're **boundaries**.

---

# Part 9 — Understand positive testing

Positive tests ask:

> What happens when users give valid input?

Examples:

```text
Age = 65
↓
20%

Age = 30
↓
0%
```

So:

```text
VALID INPUT
     ↓
Expected business result
```

---

# Part 10 — Understand negative testing

Negative testing asks:

> What happens when input is invalid?

Examples:

```text
Age = -1
Age = 121
Age = ABC
Age missing
```

But remember:

```text
ABC
Missing age
```

may not yet be defined in your requirements.

Therefore AI should classify them:

```text
Potential test scenario
BUT requirement clarification needed.
```

---

# Part 11 — Boundary Value Analysis

Our first rule says:

```text
Age >= 60
→ 20%
```

The most important numbers are:

```text
59
60
61
```

Why?

```text
59
↓
below boundary

60
↓
exact boundary

61
↓
above boundary
```

Similarly:

```text
Maximum allowed age = 120
```

Test:

```text
119
120
121
```

Not just:

```text
30
40
50
70
80
```

because those don't challenge the edges.

---

# Part 12 — Ask AI specifically for boundary tests

Use:

```text
Perform Boundary Value Analysis on
DISCOUNT_REQUIREMENTS.md.

For every numeric boundary identify:

- Minimum valid value
- Value immediately below
- Value exactly at boundary
- Value immediately above
- Maximum valid value

Generate only meaningful boundary scenarios.

Explain why each is important.
```

For this requirement:

```text
0
59
60
61
119
120
121
```

are much more valuable than random ages.

---

# Part 13 — Equivalence Partitioning

Another useful QA concept sounds complicated but is simple.

You don't need to test:

```text
Age 1
Age 2
Age 3
Age 4
...
Age 59
```

because most behave the same.

Group them:

```text
Partition 1

Age < 0
INVALID


Partition 2

Age 0–59
NO DISCOUNT


Partition 3

Age 60–120
20% DISCOUNT


Partition 4

Age >120
INVALID
```

Then choose representative values:

```text
-1
30
65
121
```

That's **equivalence partitioning**.

---

# Part 14 — Ask AI to apply it

Prompt:

```text
Apply equivalence partitioning to the age input.

Identify all valid and invalid input partitions.

For each partition provide:

- Range
- Valid/invalid
- Expected behavior
- Representative test value
- Relevant requirement

Do not create unnecessary duplicate tests.
```

You should get something like:

| Partition         | Range    | Test |
| ----------------- | -------- | ---: |
| Invalid Low       | `<0`     |   -1 |
| Valid No Discount | `0–59`   |   30 |
| Valid Discount    | `60–120` |   65 |
| Invalid High      | `>120`   |  121 |

---

# Part 15 — Now prioritize test cases

This is the second half of Lab 16.

Not every test has equal importance.

Imagine you have:

```text
100 QA tests
```

but only 30 minutes before release.

Which ones do you run?

That's why we prioritize.

---

# Part 16 — Use P0 / P1 / P2 / P3

For this course, use:

```text
P0 — Critical
Must run.

P1 — High
Very important.

P2 — Medium
Useful.

P3 — Low
Nice to have.
```

Example:

```text
P0
Age 60 → correct discount

P0
Age 59 → no discount

P0
Negative age rejected

P1
Age 120 accepted

P1
Age 121 rejected

P2
Response property names

P3
Cosmetic formatting
```

---

# Part 17 — How do we decide priority?

Use three factors:

```text
BUSINESS IMPACT

How bad is failure?


LIKELIHOOD

How likely is this scenario?


CRITICALITY

Is this core functionality?
```

Conceptually:

```text
High Impact + High Likelihood
              ↓
             P0
```

---

# Part 18 — Ask AI to prioritize

Use:

```text
Prioritize the generated QA scenarios.

Use:

P0 = Critical
P1 = High
P2 = Medium
P3 = Low

Base priority on:

1. Business impact
2. Probability of failure
3. Core business functionality
4. Financial impact
5. Boundary risk
6. Input-validation risk

For each test explain WHY it received that priority.

Do not assign everything P0.
```

That last instruction is important.

AI sometimes says:

```text
Everything is critical!
```

which makes prioritization useless.

---

# Part 19 — Example prioritization

A sensible result might be:

| Test | Scenario         | Priority |
| ---- | ---------------- | -------- |
| TC01 | Age 60 → 20%     | P0       |
| TC02 | Age 59 → 0%      | P0       |
| TC03 | Age -1 rejected  | P0       |
| TC04 | Age 121 rejected | P1       |
| TC05 | Age 65 → 20%     | P1       |
| TC06 | Age 30 → 0%      | P1       |
| TC07 | Age 120 valid    | P1       |
| TC08 | Response fields  | P2       |

---

# Part 20 — Now turn scenarios into detailed test cases

A **scenario** is:

```text
Test age 60.
```

A **test case** is much more detailed:

```text
TC001

Requirement:
R3

Title:
Verify customer aged exactly 60 receives 20% discount

Precondition:
Discount API is running.

Input:
age = 60

Steps:
1. Send GET request.
2. Set age=60.
3. Inspect response.

Expected Result:
HTTP request succeeds.
Response age = 60.
Response discount = 0.20.

Priority:
P0
```

---

# Part 21 — Ask AI to generate detailed cases

Use:

```text
Convert the approved test scenarios into detailed
manual QA test cases.

Use this format:

Test Case ID
Requirement ID
Title
Priority
Preconditions
Test Data
Steps
Expected Result
Actual Result
Status

Requirements:

- One test objective per test case.
- Keep steps executable by a junior QA tester.
- Expected results must be measurable.
- Do not invent undefined behavior.
- Mark requirements gaps clearly.
```

---

# Part 22 — Example detailed test case

### TC001 — Exact Discount Boundary

```text
Test Case ID:
TC001

Requirement:
R3

Title:
Verify age 60 receives 20% discount

Priority:
P0

Precondition:
Discount API is running.

Test Data:
age = 60

Steps:
1. Open API client or browser.
2. Send request:
   /discount?age=60
3. Inspect the response.

Expected Result:
Response contains:
age = 60
discount = 0.20

Actual Result:
[Tester enters result]

Status:
PASS / FAIL
```

---

# Part 23 — Another test: below boundary

```text
Test Case ID:
TC002

Requirement:
R4

Title:
Verify age 59 receives no discount

Priority:
P0

Test Data:
age = 59

Steps:
1. Send /discount?age=59.
2. Inspect response.

Expected:
age = 59
discount = 0
```

Now your QA test cases are directly traceable to business requirements.

---

# Part 24 — Invalid-age test

```text
TC003

Requirement:
R5

Title:
Verify negative age is rejected

Priority:
P0

Test Data:
age = -1

Expected Result:
Validation error is returned.
```

But wait.

Our requirement doesn't define the exact:

```text
HTTP status code
error JSON
error message
```

Therefore document:

```text
REQUIREMENT GAP:
Exact validation error contract is not defined.
```

This is good QA work.

---

# Part 25 — Requirements Traceability Matrix

Now create:

```text
REQUIREMENT_TRACEABILITY_MATRIX.md
```

A traceability matrix answers:

> Which tests prove each requirement?

Example:

| Requirement | Test Cases   | Covered? |
| ----------- | ------------ | -------- |
| R1          | TC001        | Yes      |
| R3          | TC001, TC005 | Yes      |
| R4          | TC002, TC006 | Yes      |
| R5          | TC003        | Yes      |
| R6          | TC004, TC007 | Yes      |
| R8          | TC001, TC002 | Yes      |

---

# Part 26 — Why Traceability matters

Without it:

```text
20 test cases
```

looks impressive.

But perhaps:

```text
Requirement R6
```

has **zero tests**.

Traceability lets you discover:

```text
Requirement
     ↓
Tests?
   /    \
 YES     NO
 ↓        ↓
Covered  GAP
```

---

# Part 27 — Ask AI to build the matrix

Prompt:

```text
Using DISCOUNT_REQUIREMENTS.md and the approved QA test
cases, create a Requirements Traceability Matrix.

Columns:

Requirement ID
Requirement
Test Case IDs
Coverage Status
Coverage Gap
Comments

Do not mark a requirement covered unless at least one
test directly validates it.

Identify any requirements with zero test coverage.
```

---

# Part 28 — Ask AI to identify duplicate tests

AI can over-generate.

Suppose it creates:

```text
Age 70 → 20%
Age 71 → 20%
Age 72 → 20%
Age 73 → 20%
Age 74 → 20%
```

Do we really need all five?

Probably not.

Ask:

```text
Review all generated QA test cases.

Identify redundant or low-value duplicate tests.

Use equivalence partitioning and boundary-value analysis.

For each test classify:

KEEP
MERGE
REMOVE
```

This is how AI should improve QA productivity instead of just producing more paperwork.

---

# Part 29 — Challenge AI's expected results

Use:

```text
Act as an independent Senior QA Reviewer.

Review every generated test case.

For each test verify:

1. Requirement supports the expected result.
2. Test data is valid.
3. Boundary calculations are correct.
4. No requirement was invented.
5. Priority makes sense.
6. Test is not redundant.
7. Expected result is measurable.

Classify each:

APPROVE
REVISE
REMOVE
REQUIREMENT CLARIFICATION NEEDED
```

This is very similar to the validation work you learned in Labs 9 and 13.

---

# Part 30 — Compare requirements against your current API

Now you can make Lab 16 even more useful.

Remember your original API code:

```csharp
app.MapGet("/discount", (int age) =>
{
    double discount = 0;

    if (age >= 60)
    {
        discount = 0.20;
    }

    return Results.Ok(new
    {
        age,
        discount
    });
});
```

Look carefully.

What happens with:

```text
age = -1
```

Current implementation:

```text
discount = 0
HTTP success
```

But the new requirement says:

```text
Age must not be negative.
```

Therefore:

```text
REQUIREMENT:
Negative age invalid

CURRENT IMPLEMENTATION:
Negative age accepted

              ↓

POTENTIAL DEFECT
```

Now your QA test actually has value.

---

# Part 31 — Execute a few P0 tests

Run your API:

```powershell
dotnet run
```

Test:

```text
/discount?age=60
```

Expected:

```json
{
  "age": 60,
  "discount": 0.2
}
```

Test:

```text
/discount?age=59
```

Expected:

```json
{
  "age": 59,
  "discount": 0
}
```

Then:

```text
/discount?age=-1
```

Requirement expects:

```text
Validation Error
```

But if current application returns:

```json
{
  "age": -1,
  "discount": 0
}
```

then:

```text
TC003
Status: FAIL
```

You just found a real QA defect.

---

# Part 32 — Ask AI to draft a defect record

Use:

```text
A QA test has failed.

Requirement:
R5 — Age must not be negative.

Test Case:
TC003

Input:
age = -1

Expected:
Validation error.

Actual:
API accepts age -1 and returns discount = 0.

Create a defect record containing:

Defect ID
Title
Requirement
Test Case
Severity
Priority
Preconditions
Steps to Reproduce
Expected Result
Actual Result
Business Impact
Evidence

Do NOT propose a code fix yet.
```

This naturally leads into the later defect-analysis labs.

---

# Part 33 — Your Lab 16 deliverables

At the end, you should have:

```text
Lab16-QA
│
├── DISCOUNT_REQUIREMENTS.md
│
├── QA_TEST_CASES.md
│
├── REQUIREMENT_TRACEABILITY_MATRIX.md
│
└── DEFECTS.md
```

You don't strictly need a separate folder, but it keeps the artifacts easy to understand.

---

# Lab 16 Completion Checklist

* [ ] Business requirements created/read.
* [ ] Requirements converted into testable rules.
* [ ] Ambiguous requirements identified.
* [ ] AI does not invent missing requirements.
* [ ] Positive scenarios generated.
* [ ] Negative scenarios generated.
* [ ] Boundary scenarios generated.
* [ ] Equivalence partitions identified.
* [ ] Test scenarios prioritized P0/P1/P2/P3.
* [ ] Priorities justified by business risk.
* [ ] Detailed manual test cases generated.
* [ ] Expected results are measurable.
* [ ] AI-generated tests reviewed.
* [ ] Duplicate tests removed.
* [ ] Requirement-to-test traceability created.
* [ ] Requirements with no tests identified.
* [ ] Selected P0 tests executed.
* [ ] Actual vs expected results recorded.
* [ ] Failed test converted into a defect where applicable.

## The main AI prompt for Lab 16

For your classroom, I would give participants this reusable prompt:

```text
You are a Senior QA Test Engineer.

Review the supplied software requirements.

Do NOT invent requirements.

STEP 1 — REQUIREMENT ANALYSIS

For every requirement identify:

- Requirement ID
- Business rule
- Input
- Expected output
- Validation
- Ambiguity
- Missing information

STEP 2 — TEST DESIGN

Generate:

- Positive scenarios
- Negative scenarios
- Boundary-value scenarios
- Invalid-input scenarios
- Error scenarios

Apply:

- Boundary Value Analysis
- Equivalence Partitioning

STEP 3 — PRIORITIZATION

Prioritize:

P0 — Critical
P1 — High
P2 — Medium
P3 — Low

Consider:

- business impact
- likelihood
- financial impact
- core functionality
- boundary risk
- validation risk

Do not make every test P0.

STEP 4 — DETAILED TEST CASES

For every approved scenario provide:

- Test Case ID
- Requirement ID
- Title
- Priority
- Preconditions
- Test Data
- Steps
- Expected Result
- Actual Result
- Status

STEP 5 — TRACEABILITY

Create a Requirements Traceability Matrix:

Requirement
Test Cases
Coverage Status
Coverage Gap

STEP 6 — REVIEW

Identify:

- duplicate tests
- missing test coverage
- invented assumptions
- requirements requiring clarification

Do not write automated test scripts yet.
```

And the progression now becomes very clear:

```text
LAB 13
Developer Unit Tests
       ↓
"Does this C# method work?"


LAB 14
Test Coverage
       ↓
"What code paths are still untested?"


LAB 15
Angular Tests
       ↓
"Does the frontend logic/UI work?"


LAB 16
QA REQUIREMENTS
       ↓
"What should we test from a
BUSINESS/USER perspective?"
       ↓
Requirements
       ↓
Test Scenarios
       ↓
Risk Priority
       ↓
Detailed QA Cases
       ↓
Traceability
       ↓
Execute
       ↓
PASS / FAIL
```

**The biggest lesson in Lab 16 is that AI should not start by generating test cases. It should first understand the requirements and expose ambiguities.** Otherwise it can very confidently generate dozens of beautifully formatted tests for business rules that nobody actually specified.
