 **Lab 18 combines three QA skills into one workflow**:

```text
PART A
AI creates GOOD TEST DATA

PART B
A test fails
      ↓
AI helps classify WHY

PART C
You have many tests
      ↓
AI decides which ones should be rerun
after a code change
```

So Lab 18 is really:

> **Test Data → Defect Analysis → Smart Regression Testing**

This follows your original course's QA topics around **test-data creation, defect analysis/classification, regression-testing optimization, coverage analysis and gap detection**. 

# Lab 18 — Use AI for Test Data Creation, Defect Classification and Regression Optimization

## Objectives

By the end of Lab 18, you should be able to:

* Generate meaningful test data with AI.
* Separate valid, invalid and boundary data.
* Avoid generating hundreds of useless data values.
* Parameterize Playwright tests using test data.
* Execute tests and collect actual failures.
* Distinguish a **product defect** from a **test defect**.
* Distinguish a defect from a **requirement gap**.
* Assign defect type, severity and priority.
* Ask AI to identify impacted regression tests.
* Create Smoke, Targeted Regression and Full Regression suites.
* Tag Playwright tests.
* Run only relevant tests.
* Create a QA optimization report.

---

# Part 1 — Understand the Lab 18 workflow

Labs 16 and 17 gave you:

```text
LAB 16

Requirement
    ↓
QA Test Cases


LAB 17

QA Test Cases
    ↓
Playwright Automation
```

Lab 18 now asks:

```text
              LAB 18

Requirements
     ↓
Generate useful TEST DATA
     ↓
Run automated tests
     ↓
        PASS / FAIL
              │
              ▼
      If test FAILS:
      What actually failed?
              │
      ┌───────┼─────────┐
      ▼       ▼         ▼
   Product   Test    Requirement
   Defect   Defect       Gap
      │
      ▼
Classify Severity
and Priority
      │
      ▼
What tests must we
rerun after fixing it?
      │
      ▼
Regression Optimization
```

---

# PART A — AI-ASSISTED TEST DATA CREATION

# Part 2 — What is test data?

You already tested:

```text
Age = 59
Age = 60
Age = 61
```

Those values are:

```text
TEST DATA
```

Your test itself says:

```text
Open page
Enter age
Click Calculate
Check discount
```

Test data tells it:

```text
Which age?
```

---

# Part 3 — Bad test-data generation

Don't ask AI:

```text
Give me 100 ages to test.
```

You might get:

```text
1
2
3
4
5
6
7
...
100
```

That's mostly useless.

You don't need every possible age.

You want values representing meaningful **classes and boundaries**.

---

# Part 4 — Start with the requirements

From the requirements used in Lab 16:

```text
Age < 0
→ Invalid

Age 0–59
→ No discount

Age 60–120
→ 20% discount

Age >120
→ Invalid
```

So useful data is:

```text
INVALID LOW
-1

MINIMUM
0

NORMAL NO DISCOUNT
30

BELOW DISCOUNT BOUNDARY
59

EXACT DISCOUNT BOUNDARY
60

ABOVE DISCOUNT BOUNDARY
61

NORMAL DISCOUNT
65

BELOW MAXIMUM
119

MAXIMUM
120

ABOVE MAXIMUM
121
```

Notice:

```text
10 intelligent values

can be more useful than

100 random values.
```

---

# Part 5 — Ask AI to create the data matrix

In Cursor Chat:

```text
You are a Senior QA Test Engineer.

Review:

- DISCOUNT_REQUIREMENTS.md
- QA_TEST_CASES.md

Do NOT generate automation code yet.

Create a test-data matrix for the age field.

Use:

- Valid data
- Invalid data
- Normal data
- Boundary data
- Equivalence partitions

For each test-data value provide:

1. Data ID
2. Age
3. Data category
4. Requirement ID
5. Expected business behavior
6. Why this value is useful
7. Priority

Avoid redundant values.

Do not invent requirements.
```

You should get something approximately like:

| Data ID | Age | Type     | Expected         |
| ------- | --: | -------- | ---------------- |
| TD01    |  -1 | Invalid  | Validation error |
| TD02    |   0 | Minimum  | 0%               |
| TD03    |  30 | Normal   | 0%               |
| TD04    |  59 | Boundary | 0%               |
| TD05    |  60 | Boundary | 20%              |
| TD06    |  61 | Boundary | 20%              |
| TD07    |  65 | Normal   | 20%              |
| TD08    | 119 | Boundary | 20%              |
| TD09    | 120 | Maximum  | 20%              |
| TD10    | 121 | Invalid  | Validation error |

---

# Part 6 — Create a reusable test-data file

Inside:

```text
lab15-angular
```

create:

```text
e2e/data/discount-test-data.ts
```

Put:

```typescript
export const validDiscountTestData = [

  {
    id: 'TD02',
    age: 0,
    expectedDiscount: '0%',
    category: 'minimum'
  },

  {
    id: 'TD03',
    age: 30,
    expectedDiscount: '0%',
    category: 'normal'
  },

  {
    id: 'TD04',
    age: 59,
    expectedDiscount: '0%',
    category: 'boundary'
  },

  {
    id: 'TD05',
    age: 60,
    expectedDiscount: '20%',
    category: 'boundary'
  },

  {
    id: 'TD06',
    age: 61,
    expectedDiscount: '20%',
    category: 'boundary'
  },

  {
    id: 'TD09',
    age: 120,
    expectedDiscount: '20%',
    category: 'maximum'
  }

];
```

Playwright supports parameterizing tests by iterating over data values and generating one test per data record. ([Playwright][1])

---

# Part 7 — Replace duplicate tests with data-driven tests

Instead of writing:

```text
Test age 30

Test age 59

Test age 60

Test age 61

Test age 120
```

individually, you can use one structure.

Open:

```text
e2e/discount.spec.ts
```

Use:

```typescript
import { test, expect } from '@playwright/test';

import {
  validDiscountTestData
} from './data/discount-test-data';

for (const data of validDiscountTestData) {

  test(
    `${data.id} - age ${data.age} returns ${data.expectedDiscount}`,
    async ({ page }) => {

      await page.goto(
        'http://localhost:4200'
      );

      await page
        .getByLabel('Customer Age')
        .fill(data.age.toString());

      await page
        .getByRole(
          'button',
          {
            name: 'Calculate Discount'
          }
        )
        .click();

      await expect(
        page.getByText(
          `Discount: ${data.expectedDiscount}`
        )
      ).toBeVisible();

    }
  );

}
```

Now the same automation runs with:

```text
0
30
59
60
61
120
```

Playwright's official parameterization guidance uses this same general pattern: data records are iterated and each creates an independent test. ([Playwright][1])

---

# Part 8 — Why this is better

Before:

```text
Test A
20 lines

Test B
20 lines

Test C
20 lines

Test D
20 lines
```

Lots of duplicate code.

After:

```text
ONE test structure
        +
Multiple test-data rows
```

Therefore:

```text
Test Logic
   ↓
Reusable

Test Data
   ↓
Change independently
```

---

# Part 9 — Run the data-driven tests

Make sure Angular is running in Terminal 1:

```powershell
npx ng serve
```

In Terminal 2:

```powershell
npx playwright test e2e/discount.spec.ts
```

Use your actual results.

You might get:

```text
6 passed
```

Do not copy this number unless six really pass.

---

# Part 10 — Ask AI whether the data is redundant

Prompt:

```text
Review discount-test-data.ts.

Do NOT add more data automatically.

For every data row determine whether it provides
unique testing value.

Classify:

KEEP
MERGE
REMOVE

Check whether the value represents:

- a unique equivalence partition
- a business boundary
- a historical defect
- an important normal case
- redundant data

Prefer a small high-value dataset.
```

This is the **optimization** part of test-data creation.

---

# PART B — DEFECT CLASSIFICATION

# Part 11 — What happens when a test fails?

Suppose you run:

```text
TD10
age = 121
```

Requirement says:

```text
Age > 120
→ Invalid
```

But your current application might not yet implement that rule.

Do **not** assume what it will do.

Execute it and record the actual result.

That's important:

```text
Expected
       ↓
Requirement

Actual
       ↓
Real application
```

Only after comparing those do we decide whether there is a defect.

---

# Part 12 — First classify the FAILURE

A failing automated test does **not** automatically mean:

```text
APPLICATION BUG
```

A failure could come from:

```text
FAILED TEST
    │
    ├── Product defect
    ├── Test-script defect
    ├── Test-data defect
    ├── Environment problem
    ├── Requirement gap
    └── Flaky automation
```

This is extremely important.

---

# Part 13 — Example: Product defect

Requirement:

```text
Age 60
→ 20%
```

Actual:

```text
Age 60
→ 0%
```

If the requirement and test are both correct:

```text
PRODUCT DEFECT
```

---

# Part 14 — Example: Test defect

Suppose AI generated:

```typescript
expect(
  page.getByText('Discount: 30%')
)
```

But requirement says:

```text
20%
```

Application shows:

```text
20%
```

The test fails.

But the application is correct.

Therefore:

```text
TEST DEFECT
```

not:

```text
PRODUCT DEFECT
```

---

# Part 15 — Example: Test-data defect

Suppose someone creates:

```text
Age:
"sixty"
```

but the test case was meant specifically to test:

```text
numeric age = 60
```

That may be:

```text
TEST DATA DEFECT
```

or a separate invalid-input test, depending on the requirement.

---

# Part 16 — Example: Requirement gap

Suppose the UI encounters:

```text
Age = -1
```

and the requirement only says:

```text
"Return an appropriate validation error."
```

But doesn't specify:

```text
Exact message?
UI display?
HTTP code?
Error format?
```

Then some expected behavior cannot be determined precisely.

Classify:

```text
REQUIREMENT GAP
```

not automatically:

```text
APPLICATION DEFECT
```

---

# Part 17 — Ask AI to classify a real test failure

After a test actually fails, give Cursor:

```text
A Playwright test failed.

Do NOT fix anything yet.

Requirement:
[paste exact requirement]

Test Case:
[paste test case]

Test Data:
[paste data]

Expected Result:
[paste expected]

Actual Result:
[paste actual]

Playwright Error:
[paste actual error]

Application Code:
[paste relevant code]

Classify the failure as ONE of:

- PRODUCT DEFECT
- TEST SCRIPT DEFECT
- TEST DATA DEFECT
- ENVIRONMENT DEFECT
- REQUIREMENT GAP
- FLAKY / INTERMITTENT
- NEEDS MORE CONTEXT

For your classification provide:

1. Evidence
2. Root-cause hypothesis
3. Confidence
4. Additional information required
5. Recommended next action

Do NOT modify code.
```

This is the central **defect-classification prompt** for Lab 18.

---

# Part 18 — Defect TYPE vs Severity vs Priority

These are different things.

### Defect Type

What kind of problem?

```text
Functional
Validation
UI
Security
Performance
Automation
Requirement
Environment
```

### Severity

How badly does it affect the system?

```text
Critical
High
Medium
Low
```

### Priority

How urgently should we fix it?

```text
P0
P1
P2
P3
```

So:

```text
TYPE
"What kind of defect?"


SEVERITY
"How bad is the impact?"


PRIORITY
"How soon should we deal with it?"
```

---

# Part 19 — Severity and Priority are NOT identical

Example:

A spelling error on the homepage of an investor demo:

```text
Severity:
LOW

Priority:
P0
```

Why?

It doesn't break the system, but management may want it fixed immediately before the presentation.

Another example:

An obscure calculation bug affecting a rarely used administrative report:

```text
Severity:
HIGH

Priority:
P1
```

because it could produce incorrect financial output even though only a small user group sees it.

---

# Part 20 — Ask AI to classify the defect

Prompt:

```text
Based ONLY on the confirmed evidence, classify this defect.

Provide:

Defect Type:
Functional / Validation / UI / Security /
Performance / Automation / Other

Severity:
Critical / High / Medium / Low

Priority:
P0 / P1 / P2 / P3

For each classification explain WHY.

Also identify:

- impacted requirement
- impacted users
- business impact
- reproducibility
- workaround
- regression risk

Do not exaggerate severity.
```

---

# Part 21 — Create a defect register

Create:

```text
DEFECT_REGISTER.md
```

Use:

```text
# Lab 18 — Defect Register

## DEF-001

Requirement:
R6

Test Case:
TC004

Test Data:
TD10

Title:
[actual defect title]

Type:
[actual]

Severity:
[actual]

Priority:
[actual]

Expected Result:
[...]

Actual Result:
[...]

Steps to Reproduce:
1. [...]
2. [...]
3. [...]

Evidence:
[...]

Root-Cause Status:
Confirmed / Hypothesis / Needs Investigation

Regression Risk:
[...]

Status:
Open
```

Do not fill it with fake failures.

Only document actual findings.

---

# Part 22 — Ask AI to detect duplicate defects

Suppose you have:

```text
DEF-001
Age 121 accepted

DEF-002
Age 122 accepted

DEF-003
Age 150 accepted
```

These may all represent the **same defect**:

```text
Maximum-age validation missing
```

Ask:

```text
Review DEFECT_REGISTER.md.

Identify defects that may share the same root cause.

Classify each as:

UNIQUE DEFECT
DUPLICATE
POSSIBLY RELATED

Explain the evidence.

Do not merge defects solely because their symptoms look
similar.
```

Very useful for real QA teams.

---

# PART C — REGRESSION TEST OPTIMIZATION

# Part 23 — What is regression testing?

Suppose a developer fixes:

```text
Age > 120 validation
```

Question:

> What else could that fix accidentally break?

You rerun existing tests.

That's:

```text
REGRESSION TESTING
```

Conceptually:

```text
Bug Fix
   ↓
Code Changed
   ↓
Old functionality might break
   ↓
Run existing tests again
```

---

# Part 24 — The wrong approach

Imagine you eventually have:

```text
1,000 automated tests
```

Developer changes:

```text
DiscountService age validation
```

Do you always need to run:

```text
all 1,000
```

on the developer's laptop?

Not necessarily.

You can intelligently select:

```text
Affected tests
+
Critical smoke tests
+
Boundary tests
```

and reserve a broader regression run for later.

---

# Part 25 — Three regression levels

For this course, use:

### Level 1 — Smoke

Very small suite:

```text
Does the core application still work?
```

Examples:

```text
Open application

Age 59 → 0%

Age 60 → 20%
```

---

### Level 2 — Targeted Regression

Tests directly related to the code change.

If change:

```text
Age validation
```

run:

```text
-1
0
59
60
61
120
121
```

---

### Level 3 — Full Regression

Run:

```text
All approved tests
```

typically before an important release or as part of a broader CI validation strategy.

So:

```text
SMOKE
smallest

       ↓

TARGETED REGRESSION
affected area

       ↓

FULL REGRESSION
everything
```

---

# Part 26 — Ask AI for change-impact analysis

Suppose the developer changed:

```text
discount.service.ts
```

Prompt:

```text
You are a Senior QA Regression Test Analyst.

The following file changed:

discount.service.ts

Change summary:
Age validation was modified.

Review:

- DISCOUNT_REQUIREMENTS.md
- QA_TEST_CASES.md
- discount.service.ts
- discount.service.spec.ts
- discount.component.spec.ts
- e2e/discount.spec.ts
- DEFECT_REGISTER.md

Do NOT modify tests.

Identify which existing tests should be rerun.

Classify tests into:

1. SMOKE
2. TARGETED REGRESSION
3. FULL REGRESSION ONLY
4. NOT IMPACTED

For each test explain:

- impacted requirement
- relationship to changed code
- defect history
- business risk
- reason for selection

Do not simply select every test.
```

---

# Part 27 — Add Playwright tags

Playwright currently supports tags beginning with `@`, and those tags can be used to filter test execution. ([Playwright][2])

For example:

```typescript
test(
  'age 60 receives 20% discount',
  {
    tag: [
      '@smoke',
      '@regression',
      '@boundary'
    ]
  },
  async ({ page }) => {

    await page.goto(
      'http://localhost:4200'
    );

    await page
      .getByLabel('Customer Age')
      .fill('60');

    await page
      .getByRole(
        'button',
        {
          name: 'Calculate Discount'
        }
      )
      .click();

    await expect(
      page.getByText(
        'Discount: 20%'
      )
    ).toBeVisible();

  }
);
```

---

# Part 28 — Create sensible tags

For Lab 18 use:

```text
@smoke
```

Core feature works.

```text
@regression
```

Part of normal regression coverage.

```text
@boundary
```

Boundary-value behavior.

```text
@validation
```

Input-validation behavior.

```text
@defect
```

Regression test created because of a previous defect.

---

# Part 29 — Example tagging strategy

| Test    | Tags                                |
| ------- | ----------------------------------- |
| Age 59  | `@smoke @boundary @regression`      |
| Age 60  | `@smoke @boundary @regression`      |
| Age 61  | `@boundary @regression`             |
| Age 0   | `@validation @regression`           |
| Age 120 | `@boundary @regression`             |
| Age 121 | `@validation @boundary @regression` |

If a future defect is fixed for age `121`, its test could also receive:

```text
@defect
```

---

# Part 30 — Run only Smoke tests

Use:

```powershell
npx playwright test --grep @smoke
```

Playwright supports filtering tests through `grep`/`--grep`, including tagged tests. ([Playwright][3])

Now instead of running everything:

```text
Run only critical tests
```

---

# Part 31 — Run Boundary tests

```powershell
npx playwright test --grep @boundary
```

Now:

```text
59
60
61
120
121
```

or whichever tests you've actually tagged.

---

# Part 32 — Run Validation tests

```powershell
npx playwright test --grep @validation
```

Again, this is much more intelligent than repeatedly running arbitrary tests.

---

# Part 33 — Run full regression

For the entire Playwright suite:

```powershell
npx playwright test
```

Playwright can run the whole configured suite or selected subsets from the command line. ([Playwright][4])

---

# Part 34 — What about flaky tests?

Suppose a test does:

```text
Run 1 → FAIL

Run 2 → PASS

Run 3 → FAIL

Run 4 → PASS
```

Possible classification:

```text
FLAKY TEST
```

Don't immediately classify:

```text
Product defect.
```

Playwright supports retries, and distinguishes tests that pass immediately, tests that pass only after retry and tests that continue to fail. ([Playwright][5])

For investigation you can run:

```powershell
npx playwright test --retries=2
```

But:

> **Retries should help expose flakiness, not hide bad tests.**

If a test repeatedly requires retries, investigate the root cause.

---

# Part 35 — Ask AI to analyze flaky behavior

Use:

```text
This Playwright test is intermittently failing.

Do NOT add arbitrary waits.

Evidence:

Run 1:
[...]

Run 2:
[...]

Run 3:
[...]

Test code:
[...]

Classify possible causes:

- application timing
- unstable locator
- shared test state
- environment
- test-data dependency
- genuine intermittent product defect

Rank the hypotheses by confidence.

Recommend diagnostic steps before changing the test.
```

Playwright is designed around isolated test execution and automatically manages separate page fixtures for tests, so shared state and explicit dependencies between tests are worth investigating when flakiness occurs. ([Playwright][5])

---

# Part 36 — Don't optimize by removing important tests

Regression optimization does **not** mean:

```text
Run fewer tests
at all costs.
```

It means:

```text
Run the RIGHT tests
at the RIGHT time.
```

Think:

```text
Developer change
      ↓
Fast feedback

SMOKE + TARGETED
      ↓
Developer gets result quickly

Later
      ↓
FULL REGRESSION
```

---

# Part 37 — Create the regression matrix

Create:

```text
REGRESSION_TEST_PLAN.md
```

Example structure:

| Test    | Requirement | Risk   | Smoke | Targeted | Full |
| ------- | ----------- | ------ | ----- | -------- | ---- |
| Age 59  | R4          | High   | ✓     | ✓        | ✓    |
| Age 60  | R3          | High   | ✓     | ✓        | ✓    |
| Age 61  | R3          | Medium |       | ✓        | ✓    |
| Age 120 | R6          | High   |       | ✓        | ✓    |
| Age 121 | R6          | High   |       | ✓        | ✓    |

---

# Part 38 — Ask AI to optimize the regression suite

Use:

```text
Review my entire QA test suite.

Create an optimized regression strategy.

Inputs:

- Requirements
- Test priorities
- Automated test cases
- Current defects
- Historical defects
- Changed source files
- Business-critical functions

Produce:

1. Smoke Suite
2. Targeted Regression Suite
3. Full Regression Suite

For every test selected explain why.

Also identify:

- duplicate tests
- low-value tests
- missing regression tests
- high-risk tests that must never be excluded

Do not optimize purely for the smallest possible test count.
```

---

# Part 39 — After fixing a defect, add a regression test

Suppose:

```text
DEF-001
Age 121 incorrectly accepted
```

Developer fixes it.

What next?

```text
DEFECT FOUND
    ↓
FIX
    ↓
VERIFY FIX
    ↓
CREATE/PRESERVE TEST
    ↓
TAG @defect @regression
```

Why?

Because six months later someone might accidentally reintroduce the problem.

This is exactly the same principle you learned with the VIP coupon regression test in the .NET labs.

---

# Part 40 — Final Lab 18 report

Create:

```text
LAB18_QA_OPTIMIZATION_REPORT.md
```

Give Cursor:

```text
Create LAB18_QA_OPTIMIZATION_REPORT.md using ONLY
actual Lab 18 results.

Structure:

# Lab 18 — AI-Assisted QA Optimization

## 1. Requirements Reviewed

List requirements used.

## 2. Test Data Design

Document:
- equivalence partitions
- boundaries
- valid data
- invalid data

## 3. Test Data Matrix

Data ID
Value
Category
Requirement
Expected Behavior
Priority

## 4. Data-Driven Automation

Document how Playwright tests consume reusable
test data.

## 5. Test Execution Results

Test
Data
Expected
Actual
PASS / FAIL

## 6. Defect Classification

For every actual failure:

Defect ID
Failure Classification
Type
Severity
Priority
Evidence
Root-Cause Status

## 7. Duplicate / Related Defects

Document findings.

## 8. Regression Impact Analysis

Describe source-code changes and affected requirements.

## 9. Regression Strategy

Smoke Suite

Targeted Regression Suite

Full Regression Suite

## 10. Playwright Tags

Document:
@smoke
@regression
@boundary
@validation
@defect

## 11. Regression Execution

Record actual selected test commands and results.

## 12. Flaky Test Analysis

Document any actual intermittent test behavior.

## 13. Remaining QA Risks

List genuine remaining risks.

## 14. Conclusion

Explain how AI improved test-data selection,
defect triage and regression-test prioritization.

Do not invent test results or defects.
```

# Lab 18 Completion Checklist

* [ ] Requirements from Lab 16 reused.
* [ ] AI identifies equivalence partitions.
* [ ] Boundary test data created.
* [ ] Invalid test data identified.
* [ ] Redundant data removed.
* [ ] `discount-test-data.ts` created.
* [ ] Playwright tests parameterized.
* [ ] Automated tests executed.
* [ ] Actual PASS/FAIL results recorded.
* [ ] Failed tests not automatically assumed to be product bugs.
* [ ] Failure classified as Product/Test/Data/Environment/Requirement/Flaky where appropriate.
* [ ] Defect type assigned.
* [ ] Severity assigned.
* [ ] Priority assigned.
* [ ] Duplicate defects checked.
* [ ] `DEFECT_REGISTER.md` created.
* [ ] Source-code change impact analyzed.
* [ ] Smoke tests identified.
* [ ] Targeted regression tests identified.
* [ ] Full regression suite identified.
* [ ] Playwright tags added.
* [ ] `--grep @smoke` demonstrated.
* [ ] `--grep @boundary` demonstrated.
* [ ] Flaky tests distinguished from confirmed defects.
* [ ] Fixed defects retained as regression tests.
* [ ] `REGRESSION_TEST_PLAN.md` created.
* [ ] `LAB18_QA_OPTIMIZATION_REPORT.md` created.

## The easiest way to remember Labs 16–18

```text
                  LAB 16
                     │
                     ▼
               REQUIREMENTS
                     │
                     ▼
                TEST CASES
                     │
                     ▼
                  LAB 17
                     │
                     ▼
               PLAYWRIGHT
                     │
                     ▼
              AUTOMATED TEST
                     │
                     ▼
                  LAB 18
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       TEST DATA    DEFECT    REGRESSION
          │          │          │
          ▼          ▼          ▼
     Good inputs   Why did    What should
     + boundaries  it fail?   we rerun?
          │          │          │
          └──────────┼──────────┘
                     ▼
                SMARTER QA
```

So the three questions for **Lab 18** are simply:

```text
1. TEST DATA
"What values should I test?"

2. DEFECT CLASSIFICATION
"The test failed — is the PRODUCT actually wrong?"

3. REGRESSION OPTIMIZATION
"Something changed — which existing tests should
I rerun first?"
```

That is what makes Lab 18 valuable: instead of using AI merely to **generate more tests**, you're using it to help QA decide **better data, better defect triage, and better regression scope**.

[1]: https://playwright.dev/docs/test-parameterize?utm_source=chatgpt.com "Parameterize tests | Playwright"
[2]: https://playwright.dev/docs/api/class-test?utm_source=chatgpt.com "Playwright Test | Playwright"
[3]: https://playwright.dev/docs/test-projects?utm_source=chatgpt.com "Projects | Playwright"
[4]: https://playwright.dev/docs/running-tests?utm_source=chatgpt.com "Running and debugging tests | Playwright"
[5]: https://playwright.dev/docs/test-retries?utm_source=chatgpt.com "Retries | Playwright"
