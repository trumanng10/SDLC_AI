Truman, **Lab 17 is the automation version of Lab 16**.

In Lab 16, you manually did this:

```text
1. Open application
2. Enter age = 60
3. Click Calculate
4. Check discount = 20%
5. Mark PASS / FAIL
```

In Lab 17, **Playwright performs those steps automatically**:

```text
Playwright
    ↓
Opens browser
    ↓
Opens Angular app
    ↓
Enters age = 60
    ↓
Clicks Calculate
    ↓
Checks "Discount: 20%"
    ↓
PASS / FAIL automatically
```

This fits your original course's Module 6 topics: **AI-assisted test-case generation, automated test-script generation, regression testing optimization, test data creation, coverage/gap detection and QA automation**. 

For this lab, I recommend **Playwright + the Angular Discount Checker from Lab 15**. Playwright is currently an end-to-end framework for web apps and supports Chromium, Firefox and WebKit; its standard setup uses `npm init playwright@latest`. ([Playwright][1])

# Lab 17 — Generate an Automated End-to-End Test Script with AI

## Objectives

By the end of the lab, you should be able to:

* Explain what an E2E test is.
* Install Playwright.
* Automatically open the Angular application.
* Find UI elements.
* Enter test data.
* Click buttons.
* Validate the displayed result.
* Run the test automatically.
* See PASS/FAIL.
* Generate additional tests with AI.
* Review AI-generated Playwright code.
* Generate an HTML test report.

---

# Part 1 — Unit Test vs E2E Test

This distinction is important.

### Lab 13 — Unit Test

```text
OrderService
      ↓
Call method directly
      ↓
Assert result
```

No browser.

### Lab 15 — Angular Component Test

```text
DiscountComponent
       ↓
TestBed
       ↓
Mock service
       ↓
Check DOM
```

Still not a real browser/user journey.

### Lab 17 — E2E Test

```text
REAL Browser
      ↓
Angular Application
      ↓
User types age
      ↓
User clicks button
      ↓
Application calculates
      ↓
Result appears
      ↓
Playwright validates result
```

That's why it's called:

**End-to-End.**

---

# Part 2 — Use the Angular project from Lab 15

Go to:

```powershell
cd C:\AI_SDLC_Labs\lab15-angular
```

Check:

```powershell
node --version
```

Then:

```powershell
npm --version
```

Then make sure Angular works:

```powershell
npx ng serve
```

Open:

```text
http://localhost:4200
```

If the application opens, you're ready.

Press:

```text
Ctrl + C
```

to stop Angular for now.

---

# Part 3 — We need a UI the user can actually interact with

Our earlier Lab 15 component had a fixed:

```typescript
age = 65;
```

For E2E testing, it's better to let the user enter an age.

Open:

```text
src/app/discount/discount.component.ts
```

Change it to:

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { DiscountService } from '../discount.service';

@Component({
  selector: 'app-discount',
  standalone: true,
  imports: [FormsModule],
  template: `
    <h1>Discount Checker</h1>

    <label for="age">
      Customer Age
    </label>

    <input
      id="age"
      name="age"
      type="number"
      [(ngModel)]="age"
    />

    <button
      type="button"
      (click)="calculate()">
      Calculate Discount
    </button>

    <p>
      Discount: {{ discount * 100 }}%
    </p>
  `
})
export class DiscountComponent {

  age = 0;

  discount = 0;

  constructor(
    private discountService: DiscountService
  ) {}

  calculate(): void {

    this.discount =
      this.discountService.calculateDiscount(
        this.age
      );
  }
}
```

Now your screen will contain:

```text
Discount Checker

Customer Age
[       ]

[Calculate Discount]

Discount: 0%
```

---

# Part 4 — Make Angular show this component

Your current Angular project structure may differ slightly depending on the Angular version.

Open your main application component, commonly:

```text
src/app/app.ts
```

or:

```text
src/app/app.component.ts
```

Make sure it imports:

```typescript
DiscountComponent
```

and displays:

```html
<app-discount></app-discount>
```

The goal is simply:

```text
http://localhost:4200
        ↓
Discount Checker appears
```

---

# Part 5 — Test it manually first

Run:

```powershell
npx ng serve
```

Open:

```text
http://localhost:4200
```

Enter:

```text
60
```

Click:

```text
Calculate Discount
```

Expected:

```text
Discount: 20%
```

Try:

```text
59
```

Expected:

```text
Discount: 0%
```

This is important:

> **Don't automate a workflow you haven't first confirmed manually.**

---

# Part 6 — Stop Angular

Press:

```text
Ctrl + C
```

---

# Part 7 — Install Playwright

From:

```text
C:\AI_SDLC_Labs\lab15-angular
```

run:

```powershell
npm init playwright@latest
```

Playwright's current installation guide uses this command to initialize Playwright in an existing or new Node project. ([Playwright][1])

You'll be asked several questions.

Choose approximately:

```text
Do you want to use TypeScript or JavaScript?

TypeScript
```

Then:

```text
Where to put your end-to-end tests?

e2e
```

If it asks about GitHub Actions:

```text
No
```

for this beginner lab.

If asked:

```text
Install Playwright browsers?

Yes
```

---

# Part 8 — What did Playwright create?

Your project should now have roughly:

```text
lab15-angular
│
├── src
│
├── e2e
│   └── example.spec.ts
│
├── playwright.config.ts
├── package.json
├── angular.json
└── ...
```

Playwright's normal scaffold includes a configuration file and test directory. ([Playwright][1])

---

# Part 9 — What is `.spec.ts`?

This:

```text
discount.spec.ts
```

means:

```text
Automated test specification
```

Similar to Angular:

```text
discount.component.spec.ts
```

But now:

```text
Angular .spec.ts
    ↓
Component/unit tests

Playwright .spec.ts
    ↓
Browser/E2E tests
```

---

# Part 10 — Delete the sample Playwright test

You can delete:

```text
e2e/example.spec.ts
```

Create:

```text
e2e/discount.spec.ts
```

---

# Part 11 — Start Angular in one terminal

Open **Terminal 1**:

```powershell
cd C:\AI_SDLC_Labs\lab15-angular
```

Run:

```powershell
npx ng serve
```

Leave it running.

You should see:

```text
http://localhost:4200
```

---

# Part 12 — Open a second terminal

Open **Terminal 2**:

```powershell
cd C:\AI_SDLC_Labs\lab15-angular
```

We'll execute Playwright from here.

So:

```text
Terminal 1
Angular application
      ↓
localhost:4200


Terminal 2
Playwright tests
      ↓
Open localhost:4200
```

---

# Part 13 — Write your first E2E test

Put this in:

```text
e2e/discount.spec.ts
```

```typescript
import { test, expect } from '@playwright/test';

test('customer aged 60 receives 20% discount', async ({ page }) => {

  // Arrange
  await page.goto('http://localhost:4200');

  // Act
  await page
    .getByLabel('Customer Age')
    .fill('60');

  await page
    .getByRole('button', {
      name: 'Calculate Discount'
    })
    .click();

  // Assert
  await expect(
    page.getByText('Discount: 20%')
  ).toBeVisible();

});
```

Playwright recommends resilient locators based on user-facing information, particularly `getByRole()`, `getByLabel()` and similar locators. ([Playwright][2])

---

# Part 14 — Understand the script

First:

```typescript
import { test, expect }
from '@playwright/test';
```

means:

```text
Use Playwright's:

test
+
assertion tools
```

Then:

```typescript
test(
  'customer aged 60 receives 20% discount',
```

means:

> Create an automated test with this name.

---

# Part 15 — `page.goto()`

This:

```typescript
await page.goto(
  'http://localhost:4200'
);
```

means:

```text
Open browser
     ↓
Navigate to Angular application
```

Playwright's normal tests begin by navigating to the application with `page.goto()`. ([Playwright][3])

---

# Part 16 — Find the Age input

This:

```typescript
page.getByLabel('Customer Age')
```

tells Playwright:

> Find the input associated with the label "Customer Age."

Our HTML contains:

```html
<label for="age">
  Customer Age
</label>

<input id="age">
```

Therefore Playwright can identify it.

Then:

```typescript
.fill('60')
```

means:

```text
Type 60 into the field
```

---

# Part 17 — Find the button

This:

```typescript
page.getByRole(
  'button',
  {
    name: 'Calculate Discount'
  }
)
```

means:

> Find the button that the user sees as "Calculate Discount."

Then:

```typescript
.click();
```

means:

```text
Click it.
```

Playwright recommends locating interactive controls such as buttons by role and accessible name where possible. ([Playwright][2])

---

# Part 18 — Validate the result

This:

```typescript
await expect(
  page.getByText('Discount: 20%')
).toBeVisible();
```

means:

```text
Look for:

Discount: 20%

        ↓

Visible?
 /       \
YES       NO
 ↓         ↓
PASS      FAIL
```

That's your automated expected result.

---

# Part 19 — Run the E2E test

In Terminal 2:

```powershell
npx playwright test e2e/discount.spec.ts
```

Playwright's standard command for running tests is `npx playwright test`; you can also specify one test file. ([Playwright][1])

You want something similar to:

```text
1 passed
```

Congratulations.

You have created your first automated E2E test.

---

# Part 20 — Watch the browser execute it

Normally Playwright may run without showing you the browser.

For classroom demonstration, run:

```powershell
npx playwright test e2e/discount.spec.ts --headed
```

Playwright supports `--headed` to show the actual browser window while tests run. ([Playwright][1])

Now you'll physically see:

```text
Browser opens
     ↓
Age 60 entered automatically
     ↓
Button clicked automatically
     ↓
20% displayed
     ↓
Browser closes
```

This is great for demonstrating automation to participants.

---

# Part 21 — Deliberately make the test FAIL

Change:

```typescript
page.getByText(
  'Discount: 20%'
)
```

to:

```typescript
page.getByText(
  'Discount: 30%'
)
```

Run:

```powershell
npx playwright test e2e/discount.spec.ts
```

Now:

```text
FAIL
```

because:

```text
Expected:
Discount: 30%

Actual:
Discount: 20%
```

This proves your automated test actually detects incorrect behavior.

Change it back:

```text
Discount: 20%
```

---

# Part 22 — Now bring AI into the lab

Don't tell Cursor:

```text
Generate many Playwright tests.
```

Instead give AI your Lab 16 requirements.

Prompt:

```text
You are a Senior QA Automation Engineer.

Review:

- DISCOUNT_REQUIREMENTS.md
- QA_TEST_CASES.md
- discount.component.ts
- discount.service.ts
- e2e/discount.spec.ts

Do NOT modify code yet.

Identify which approved manual QA test cases from Lab 16
are suitable for Playwright end-to-end automation.

For every candidate provide:

1. Test Case ID
2. Requirement ID
3. User journey
4. Test data
5. Expected result
6. Automation suitability:
   High / Medium / Low
7. Reason
8. Required UI element

Do not automate undefined requirements.
```

This is the correct workflow:

```text
Manual Test
      ↓
Automation Candidate
      ↓
AI proposes script
```

not:

```text
AI invents random browser tests
```

---

# Part 23 — Ask AI to generate additional E2E tests

After reviewing the list:

```text
Generate Playwright TypeScript tests ONLY for the
approved P0 and P1 QA test cases.

Requirements:

1. Do not invent requirements.
2. Use meaningful test names.
3. Prefer getByRole(), getByLabel() and getByText().
4. Avoid brittle CSS selectors.
5. Use explicit assertions.
6. Each test should validate one clear business scenario.
7. Do not add arbitrary waits or sleep().
8. Keep the tests beginner-friendly.
9. Do not modify the application source.
```

Playwright recommends user-facing locators such as role, label and text because they tend to be more resilient than implementation-dependent selectors. ([Playwright][2])

---

# Part 24 — Test age 59

AI could create:

```typescript
test('customer aged 59 receives no discount', async ({ page }) => {

  await page.goto(
    'http://localhost:4200'
  );

  await page
    .getByLabel('Customer Age')
    .fill('59');

  await page
    .getByRole(
      'button',
      {
        name: 'Calculate Discount'
      }
    )
    .click();

  await expect(
    page.getByText('Discount: 0%')
  ).toBeVisible();

});
```

Now we have a boundary pair:

```text
59 → 0%

60 → 20%
```

Excellent test coverage.

---

# Part 25 — Test age 61

Add:

```typescript
test('customer aged 61 receives 20% discount', async ({ page }) => {

  await page.goto(
    'http://localhost:4200'
  );

  await page
    .getByLabel('Customer Age')
    .fill('61');

  await page
    .getByRole(
      'button',
      {
        name: 'Calculate Discount'
      }
    )
    .click();

  await expect(
    page.getByText('Discount: 20%')
  ).toBeVisible();

});
```

Now:

```text
59
↓
Below boundary
0%

60
↓
Exact boundary
20%

61
↓
Above boundary
20%
```

---

# Part 26 — Run all E2E tests

Run:

```powershell
npx playwright test
```

You might see:

```text
3 passed
```

Use your actual result.

---

# Part 27 — Open Playwright's HTML report

Run:

```powershell
npx playwright show-report
```

Playwright includes an HTML report that lets you inspect passed, failed, skipped and flaky tests and their details. ([Playwright][1])

You'll get a browser report showing something like:

```text
Playwright Test Report

✓ customer aged 59 receives no discount
✓ customer aged 60 receives 20% discount
✓ customer aged 61 receives 20% discount
```

This is excellent classroom evidence.

---

# Part 28 — Use Playwright UI mode

For teaching, also try:

```powershell
npx playwright test --ui
```

Playwright's UI mode lets you visually inspect and rerun tests with detailed steps. ([Playwright][1])

For beginners, this is often easier to understand than only looking at PowerShell.

---

# Part 29 — Don't use `waitForTimeout(5000)`

AI sometimes generates:

```typescript
await page.waitForTimeout(5000);
```

Avoid this unless there is a very specific reason.

Playwright locators have built-in auto-waiting/retry behavior, which is one reason locators are central to Playwright's testing model. ([Playwright][4])

Prefer:

```typescript
await expect(
  page.getByText('Discount: 20%')
).toBeVisible();
```

rather than:

```text
Wait five seconds and hope.
```

---

# Part 30 — Don't let AI generate brittle selectors

Bad:

```typescript
page.locator(
  'body > app-root > div:nth-child(2) > button'
)
```

Why bad?

If someone slightly restructures the HTML:

```text
Test breaks
```

even though the application still works perfectly.

Better:

```typescript
page.getByRole(
  'button',
  {
    name: 'Calculate Discount'
  }
)
```

That's closer to how the user identifies the control. Playwright specifically recommends prioritizing user-facing attributes and explicit contracts in locators. ([Playwright][2])

---

# Part 31 — Challenge AI's automated tests

Use:

```text
Act as an independent Senior Playwright QA Engineer.

Review e2e/discount.spec.ts.

Do NOT modify anything.

For every automated test check:

1. Which Lab 16 requirement does it validate?
2. Which manual QA test case does it automate?
3. Is the expected result correct?
4. Are selectors stable?
5. Is the test independent?
6. Does it contain arbitrary waits?
7. Does it duplicate another test?
8. Would it fail if the business behavior broke?
9. Is it testing user-visible behavior?

Classify each test:

KEEP
IMPROVE
REMOVE
```

That's the AI review part of the lab.

---

# Part 32 — Create a simple traceability table

Your Lab 17 report should show:

| Requirement    | Manual Case   | Automated Test | Status    |
| -------------- | ------------- | -------------- | --------- |
| Age < 60 → 0%  | TC002         | Age 59 test    | Automated |
| Age ≥ 60 → 20% | TC001         | Age 60 test    | Automated |
| Age ≥ 60 → 20% | Boundary case | Age 61 test    | Automated |
| Negative age   | TC003         | Not yet        | Pending   |

This proves automation came from your **requirements**, not from random AI generation.

---

# Part 33 — Why might negative-age E2E be pending?

Remember Lab 16?

Requirement:

```text
Negative age must be rejected.
```

But your current Angular service throws:

```typescript
throw new Error(
  'Age cannot be negative'
);
```

Your UI does not yet specify:

```text
What should the user SEE?
```

Should it display:

```text
Invalid age
```

or:

```text
Age must be between 0 and 120
```

or something else?

That is a **requirement/UI design gap**.

Don't let AI invent it.

So record:

```text
Negative-age E2E:
Pending requirement/UI behavior clarification.
```

That's good QA.

---

# Part 34 — Create the Lab 17 report

Create:

```text
E2E_AUTOMATION_REPORT.md
```

Ask Cursor:

```text
Create E2E_AUTOMATION_REPORT.md using actual Lab 17
results.

Structure:

# Lab 17 — Automated End-to-End Testing

## 1. Application Under Test

Angular Discount Checker

## 2. Automation Tool

Playwright

## 3. Source Test Cases

List Lab 16 manual QA test cases selected for automation.

## 4. Automation Selection

Test Case
Requirement
Priority
Automation Suitability
Reason

## 5. Automated Test Scripts

List implemented Playwright tests.

## 6. Execution Results

Test
Expected
Actual
PASS / FAIL

## 7. Traceability

Requirement
Manual Test Case
Automated Test
Coverage Status

## 8. Automation Gaps

List tests not automated and explain why.

## 9. AI Review

Document:
- duplicate tests
- brittle locators
- arbitrary waits
- invalid assumptions

## 10. Conclusion

Do not invent execution results.
```

---

# Lab 17 Completion Checklist

* [ ] Angular Discount Checker runs.
* [ ] Age input is available.
* [ ] Calculate button works.
* [ ] Playwright installed.
* [ ] Playwright browsers installed.
* [ ] `e2e` test folder exists.
* [ ] First `discount.spec.ts` created.
* [ ] Test opens `localhost:4200`.
* [ ] Test fills age.
* [ ] Test clicks Calculate.
* [ ] Test validates displayed result.
* [ ] Test deliberately failed once to prove detection.
* [ ] Age 59 tested.
* [ ] Age 60 tested.
* [ ] Age 61 tested.
* [ ] `npx playwright test` succeeds.
* [ ] `--headed` mode demonstrated.
* [ ] HTML report viewed.
* [ ] AI maps manual tests to automation candidates.
* [ ] AI-generated tests reviewed.
* [ ] No arbitrary sleeps.
* [ ] Stable Playwright locators used.
* [ ] Undefined requirements are not automated.
* [ ] Requirement-to-automation traceability created.
* [ ] `E2E_AUTOMATION_REPORT.md` created.

The easiest way to remember Labs 16–17 is:

```text
                  REQUIREMENTS
                       │
                       ▼
                    LAB 16
                       │
                AI generates
                 QA scenarios
                       │
                       ▼
                Manual Test Cases
                       │
                       ▼
                    P0 / P1
                       │
                       ▼
                    LAB 17
                       │
               Which tests can
               be automated?
                       │
                       ▼
                     AI
                       │
             generates Playwright
                       │
                       ▼
               REAL WEB BROWSER
                       │
            ┌──────────┼──────────┐
            ▼          ▼          ▼
          Navigate    Enter      Click
            │          │          │
            └──────────┼──────────┘
                       ▼
                  Check Result
                       │
                 ┌─────┴─────┐
                 ▼           ▼
                PASS        FAIL
```

So the key distinction is:

**Lab 16:** “What should QA test based on the requirements?”

**Lab 17:** “Which of those QA tests can Playwright execute automatically from the user's point of view?”

For your course, I would definitely teach Lab 17 this way—**one manual P0 scenario first, automate it successfully, deliberately make it fail, then let AI generate the additional scripts.** That makes students understand what E2E automation is before AI starts writing large amounts of Playwright code.

[1]: https://playwright.dev/docs/intro?utm_source=chatgpt.com "Installation | Playwright"
[2]: https://playwright.dev/docs/locators?utm_source=chatgpt.com "Locators | Playwright"
[3]: https://playwright.dev/docs/next/writing-tests?utm_source=chatgpt.com "Writing tests | Playwright"
[4]: https://playwright.dev/docs/api/class-locator?utm_source=chatgpt.com "Locator | Playwright"
