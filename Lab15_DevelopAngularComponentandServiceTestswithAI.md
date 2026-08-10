**Lab 15 is the Angular/frontend version of Lab 13**.

In Lab 13:

```text
C# OrderService
      ↓
xUnit Tests
      ↓
dotnet test
```

In Lab 15:

```text
Angular Component + Service
          ↓
Angular Unit Tests
          ↓
ng test
```

Your original syllabus specifically requires **Angular Component Testing, Angular Service Testing, AI-Assisted Angular Test Development, Frontend Quality Validation, and Maintaining Test Quality and Reliability**. 

One important current update: for **new Angular CLI projects, Vitest is now the default test runner**. Older Angular training materials often use Karma/Jasmine, but for a new Lab 15 project I recommend using the current **Angular + Vitest + TestBed** approach. ([Angular][1])

# Lab 15 — Develop Angular Component and Service Tests with AI

## What exactly are we testing?

We'll create a tiny Angular Discount application.

The **Service** contains the business rule:

```text
Age >= 60
    ↓
20% discount

Age < 60
    ↓
0% discount

Negative age
    ↓
Invalid
```

The **Component** uses the service and displays the result:

```text
DiscountComponent
       │
       │ asks
       ▼
DiscountService
       │
       ▼
calculateDiscount(age)
       │
       ▼
0% or 20%
       │
       ▼
Component displays result
```

Angular's documentation describes services as the place where application/business logic commonly lives, while component tests check the component class together with its rendered DOM/template behavior. ([Angular][2])

## Component vs Service

| Item              | Purpose                  | What we test                    |
| ----------------- | ------------------------ | ------------------------------- |
| Angular Service   | Business/data logic      | Correct calculations            |
| Angular Component | UI + interaction         | Correct display and calls       |
| `.spec.ts`        | Test file                | Automated checks                |
| `TestBed`         | Angular test environment | Creates/injects Angular objects |
| Vitest            | Test runner              | Runs and reports tests          |

## Step-by-step Lab

1. **Check whether Node.js and npm are available.** Open PowerShell and run:

```powershell
node --version
```

Then:

```powershell
npm --version
```

You should get version numbers. If either command is not recognized, Node.js needs to be installed before continuing.

2. **Create a separate Angular project.** Do not put the Angular source inside `Lab02Api`. From your labs folder:

```powershell
cd C:\AI_SDLC_Labs
```

Create the project:

```powershell
npx -p @angular/cli@latest ng new lab15-angular --routing=false --style=css --test-runner=vitest --skip-git
```

The current Angular CLI supports `--test-runner=vitest`, and Vitest is the default for new projects. ([Angular][3])

Enter the project:

```powershell
cd lab15-angular
```

You should now have roughly:

```text
AI_SDLC_Labs
│
├── Lab02Api
├── Lab02Api.Tests
│
└── lab15-angular
    ├── src
    ├── angular.json
    ├── package.json
    └── node_modules
```

3. **Run the Angular application once.**

```powershell
npx ng serve
```

You should eventually see something similar to:

```text
Local:
http://localhost:4200/
```

Open:

```text
http://localhost:4200
```

If the Angular starter page appears, you're ready.

Stop it with:

```text
Ctrl + C
```

4. **Run the default Angular tests.**

```powershell
npx ng test --no-watch
```

Angular's standard test command is `ng test`; with current new projects it uses Vitest and normally runs in a Node.js environment with `jsdom` simulating the browser DOM. ([Angular][1])

You should see tests execute.

That means:

```text
Angular
   ✓

Vitest
   ✓

Testing environment
   ✓
```

5. **Generate our Discount Service.**

Run:

```powershell
npx ng generate service discount --type=service
```

The Angular CLI generates a service and its `.spec.ts` unit-test file unless tests are explicitly skipped. ([Angular][4])

You should get files similar to:

```text
src/app/discount.service.ts
src/app/discount.service.spec.ts
```

6. **Replace `discount.service.ts` with our business logic.**

Use:

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class DiscountService {

  calculateDiscount(age: number): number {

    if (age < 0) {
      throw new Error('Age cannot be negative');
    }

    if (age >= 60) {
      return 0.20;
    }

    return 0;
  }
}
```

This is our frontend business rule.

7. **Understand what the service does.**

```text
calculateDiscount(age)

        │
        ▼
    age < 0 ?
     /      \
   YES       NO
    ↓         ↓
 Error     age >= 60 ?
             /     \
           YES      NO
            ↓        ↓
           20%       0%
```

Now we have something meaningful to test.

8. **Ask AI to identify service test cases before generating code.** In Cursor Chat use:

```text
Review discount.service.ts.

Do NOT generate test code yet.

Extract all business rules implemented by
calculateDiscount().

Create a test scenario table containing:

- Scenario
- Input age
- Expected result
- Boundary condition
- Invalid input
- Why the test matters

Pay special attention to the age 60 boundary.

Do not modify the source code.
```

AI should identify cases such as:

| Scenario       | Age | Expected |
| -------------- | --: | -------: |
| Senior         |  65 |      20% |
| Boundary       |  60 |      20% |
| Below boundary |  59 |       0% |
| Young customer |  30 |       0% |
| Invalid        |  -1 |    Error |

9. **Create the service unit tests.** Replace the generated `discount.service.spec.ts` with:

```typescript
import { TestBed } from '@angular/core/testing';
import { beforeEach, describe, expect, it } from 'vitest';

import { DiscountService } from './discount.service';

describe('DiscountService', () => {

  let service: DiscountService;

  beforeEach(() => {

    TestBed.configureTestingModule({});

    service = TestBed.inject(DiscountService);

  });

  it('returns 20% discount when age is above 60', () => {

    const result = service.calculateDiscount(65);

    expect(result).toBe(0.20);

  });

  it('returns 20% discount when age is exactly 60', () => {

    const result = service.calculateDiscount(60);

    expect(result).toBe(0.20);

  });

  it('returns no discount when age is below 60', () => {

    const result = service.calculateDiscount(59);

    expect(result).toBe(0);

  });

  it('throws an error when age is negative', () => {

    expect(() =>
      service.calculateDiscount(-1)
    ).toThrow('Age cannot be negative');

  });

});
```

Angular officially recommends `TestBed` for creating an isolated Angular testing environment and injecting services; its current service-testing examples use Vitest's `describe`, `it`, `expect`, and `beforeEach`. ([Angular][2])

10. **Run only the service tests.**

```powershell
npx ng test --no-watch
```

You want something equivalent to:

```text
✓ returns 20% discount when age is above 60
✓ returns 20% discount when age is exactly 60
✓ returns no discount when age is below 60
✓ throws an error when age is negative

4 passed
0 failed
```

Now you have successfully completed:

```text
Angular Service
      ↓
Service Unit Tests
      ↓
PASS
```

11. **Now create an Angular Component.**

Run:

```powershell
npx ng generate component discount --type=component --inline-template --inline-style
```

Angular CLI components are standalone by default in current projects, and the CLI automatically generates a matching `.spec.ts` unless tests are skipped. ([Angular][5])

You'll get a `discount` folder containing files similar to:

```text
src/app/discount/
├── discount.component.ts
└── discount.component.spec.ts
```

12. **Replace the generated component code.**

Use:

```typescript
import { Component } from '@angular/core';
import { DiscountService } from '../discount.service';

@Component({
  selector: 'app-discount',
  standalone: true,
  template: `
    <h2>Discount Checker</h2>

    <p data-testid="age">
      Age: {{ age }}
    </p>

    <button
      data-testid="calculate-button"
      (click)="calculate()">
      Calculate Discount
    </button>

    <p data-testid="result">
      Discount: {{ discount * 100 }}%
    </p>
  `
})
export class DiscountComponent {

  age = 65;

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

The component does not contain the discount rule.

It delegates:

```text
DiscountComponent
       ↓
calculate()
       ↓
DiscountService
       ↓
calculateDiscount(age)
```

That's good separation.

13. **Before generating component tests, ask AI what should actually be tested.**

Use:

```text
Review discount.component.ts.

Do NOT generate test code yet.

Identify the important component behaviors to test.

Separate them into:

1. Component creation
2. Initial display
3. User interaction
4. Service interaction
5. Updated DOM output

Do NOT retest DiscountService's internal business
rules inside the component test.

Return a test scenario table.
```

This last instruction matters.

We already tested:

```text
65 → 20%
```

inside:

```text
DiscountService tests
```

The component's responsibility is different:

```text
Button clicked
     ↓
Service called
     ↓
Result displayed
```

14. **Create the component tests using a mock service.**

Replace `discount.component.spec.ts` with:

```typescript
import { TestBed } from '@angular/core/testing';
import {
  beforeEach,
  describe,
  expect,
  it,
  vi
} from 'vitest';

import { DiscountComponent } from './discount.component';
import { DiscountService } from '../discount.service';

describe('DiscountComponent', () => {

  let discountServiceMock: {
    calculateDiscount: ReturnType<typeof vi.fn>
  };

  beforeEach(async () => {

    discountServiceMock = {
      calculateDiscount: vi.fn()
    };

    await TestBed.configureTestingModule({

      imports: [
        DiscountComponent
      ],

      providers: [
        {
          provide: DiscountService,
          useValue: discountServiceMock
        }
      ]

    }).compileComponents();

  });

  it('creates the component', () => {

    const fixture =
      TestBed.createComponent(
        DiscountComponent
      );

    expect(
      fixture.componentInstance
    ).toBeTruthy();

  });

  it('displays the initial age', () => {

    const fixture =
      TestBed.createComponent(
        DiscountComponent
      );

    fixture.detectChanges();

    const ageText =
      fixture.nativeElement
        .querySelector(
          '[data-testid="age"]'
        )
        .textContent;

    expect(ageText).toContain('65');

  });

  it('calls DiscountService when calculate is executed', () => {

    discountServiceMock
      .calculateDiscount
      .mockReturnValue(0.20);

    const fixture =
      TestBed.createComponent(
        DiscountComponent
      );

    const component =
      fixture.componentInstance;

    component.age = 65;

    component.calculate();

    expect(
      discountServiceMock.calculateDiscount
    ).toHaveBeenCalledWith(65);

  });

  it('displays the discount returned by the service', () => {

    discountServiceMock
      .calculateDiscount
      .mockReturnValue(0.20);

    const fixture =
      TestBed.createComponent(
        DiscountComponent
      );

    fixture.componentInstance.calculate();

    fixture.detectChanges();

    const result =
      fixture.nativeElement
        .querySelector(
          '[data-testid="result"]'
        )
        .textContent;

    expect(result).toContain('20%');

  });

});
```

Angular's component-testing guidance specifically emphasizes testing the component **together with its DOM/template**, using `TestBed` to create the component and inspect its rendered behavior. ([Angular][6])

15. **Understand why we're mocking `DiscountService`.**

This:

```typescript
discountServiceMock = {
  calculateDiscount: vi.fn()
};
```

means:

> Don't use the real DiscountService in this component test. Give the component a fake one that we control.

Conceptually:

```text
NORMAL APPLICATION

Component
   ↓
Real DiscountService


COMPONENT UNIT TEST

Component
   ↓
Fake DiscountService
```

Why?

Because we already tested the real service separately.

Now we're asking:

> Does the **component use the service correctly?**

Vitest's `vi.fn()` creates a mock/spying function, and Angular's current service-testing guidance demonstrates providing mocked dependencies through `TestBed`. ([Angular][2])

16. **Run everything.**

```powershell
npx ng test --no-watch
```

You should now have something roughly like:

```text
DiscountService
✓ age 65
✓ age 60
✓ age 59
✓ negative age

DiscountComponent
✓ creates
✓ displays age
✓ calls service
✓ displays result

8 passed
0 failed
```

Use your actual count.

17. **Now let AI find missing tests.**

Use this prompt:

```text
Act as a Senior Angular QA Engineer.

Review:

- discount.service.ts
- discount.service.spec.ts
- discount.component.ts
- discount.component.spec.ts

Do NOT modify code yet.

Identify missing test scenarios.

For each gap provide:

- Source behavior
- Existing test coverage
- Missing scenario
- Risk
- Recommended test
- Priority: High / Medium / Low

Check specifically for:

- boundary values
- invalid input
- DOM rendering
- button interaction
- service calls
- incorrect duplicate tests
- tests that verify implementation instead of behavior

Do not recommend tests solely to increase the
number of tests.
```

18. **Finally challenge the AI-generated tests before accepting them.**

Use:

```text
Review all Angular tests critically.

For every test classify:

KEEP
IMPROVE
REMOVE

Check:

1. Does it test real behavior?
2. Is its expected result correct?
3. Does it duplicate another test?
4. Would the test fail if the behavior broke?
5. Is the service test testing service logic?
6. Is the component test testing component behavior?
7. Is the component test unnecessarily retesting
   DiscountService?
8. Are important boundaries missing?

Do not modify production code.
```

That completes the core Lab 15 workflow.

## What you should understand at the end

The **service test** looks like:

```text
                SERVICE TEST

Age
 │
 ▼
DiscountService
 │
 ▼
calculateDiscount()
 │
 ▼
Expected result?
 │
 ├── YES → PASS
 └── NO  → FAIL
```

The **component test** looks like:

```text
               COMPONENT TEST

           DiscountComponent
                  │
        user clicks Calculate
                  │
                  ▼
          Mock DiscountService
                  │
           returns 0.20
                  │
                  ▼
             Component
                  │
                  ▼
          DOM displays "20%"
                  │
                  ▼
                PASS
```

And the biggest distinction is:

| Service Test     | Component Test           |
| ---------------- | ------------------------ |
| Test calculation | Test UI behavior         |
| Age 60 → 20%     | Button calls service     |
| Negative → error | Result appears in DOM    |
| Business logic   | Presentation/integration |
| Usually no DOM   | Uses DOM                 |
| Real service     | Often mock dependency    |

## Lab 15 deliverables

Your Angular project should end up roughly like:

```text
lab15-angular
│
├── src
│   └── app
│       ├── discount.service.ts
│       ├── discount.service.spec.ts
│       │
│       └── discount
│           ├── discount.component.ts
│           └── discount.component.spec.ts
│
├── angular.json
├── package.json
└── ...
```

Your completion check is: the Angular project builds, `DiscountService` is tested, the age-60 boundary is tested, invalid age is tested, `DiscountComponent` is created through `TestBed`, the service dependency is mocked, service interaction is verified, rendered DOM output is verified, and `npx ng test --no-watch` finishes with **zero failed approved tests**.

The progression from Labs 13–15 is therefore:

```text
LAB 13
.NET Backend
     ↓
xUnit
     ↓
Business Logic Tests


LAB 14
.NET Backend
     ↓
Coverage Analysis
     ↓
Find Missing Tests


LAB 15
Angular Frontend
     ↓
┌──────────────┬───────────────┐
│              │               │
Service       Component
Tests          Tests
│              │
Logic          UI / DOM
│              │
└───────┬──────┘
        ↓
      Vitest
        ↓
     ng test
```

**For this course, I recommend using Vitest rather than teaching new students Karma/Jasmine first.** Karma is still supported, but Angular's current new-project testing setup uses Vitest, so this keeps Lab 15 aligned with modern Angular while still teaching the enduring concepts: `TestBed`, service isolation, component DOM testing, mocking, assertions, and AI-assisted test generation. ([Angular][1])

[1]: https://angular.dev/guide/testing?utm_source=chatgpt.com "Testing • Overview • Angular"
[2]: https://angular.dev/guide/testing/services?utm_source=chatgpt.com "Testing services • Angular"
[3]: https://angular.dev/cli/new?utm_source=chatgpt.com "ng new • Angular"
[4]: https://angular.dev/cli/generate/service?utm_source=chatgpt.com "service • Angular"
[5]: https://angular.dev/cli/generate/component?utm_source=chatgpt.com "component • Angular"
[6]: https://angular.dev/guide/testing/components-basics?utm_source=chatgpt.com "Basics of testing components • Angular"
