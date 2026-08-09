## Lab 1 — Start From Zero

### Objective

You will give **the exact same coding problem** to:

**GitHub Copilot → Cursor → Claude Code**

Then compare which produces the best solution.

You only need **two tools minimum** if you do not currently have all three. That is consistent with the prerequisite in the lab guide. 

### Step 1 — Check .NET

Open **PowerShell** and type:

```powershell
dotnet --version
```

If you get something such as:

```text
8.0.xxx
9.0.xxx
10.0.xxx
```

you can continue.

Also check Git:

```powershell
git --version
```

---

### Step 2 — Create the Lab folder

```powershell
mkdir C:\AI_SDLC_Labs
cd C:\AI_SDLC_Labs
```

Create our first project:

```powershell
dotnet new web -n Lab01Api
cd Lab01Api
```

You should now have something similar to:

```text
Lab01Api
│
├── Lab01Api.csproj
├── Program.cs
├── appsettings.json
└── Properties
```

---

### Step 3 — Open `Program.cs`

Replace everything inside `Program.cs` with:

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/", () => "AI SDLC Lab 1");

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

app.Run();
```

The API calculates a simple senior-citizen discount.

But there is deliberately a problem.

For example:

```text
age = -20
```

is accepted.

So our AI assistants will be asked to fix this problem.

---

### Step 4 — Run the original application

From PowerShell:

```powershell
dotnet run --urls http://localhost:5000
```

You should see something similar to:

```text
Now listening on: http://localhost:5000
```

Open:

```text
http://localhost:5000/
```

You should see:

```text
AI SDLC Lab 1
```

Now test:

```text
http://localhost:5000/discount?age=65
```

Expected:

```json
{
  "age": 65,
  "discount": 0.2
}
```

Try:

```text
http://localhost:5000/discount?age=-10
```

It will probably still return:

```json
{
  "age": -10,
  "discount": 0
}
```

**This is our problem.**

Age `-10` should not be accepted.

Stop the application with:

```text
Ctrl + C
```

---

### Step 5 — Create the Git baseline

Now we preserve this deliberately imperfect version.

```powershell
git init
git add .
git commit -m "Lab 1 baseline"
```

Create your lab branch:

```powershell
git checkout -b lab01-ai-assistant-comparison
```

Your starting point is now protected.

---

### Step 6 — Define ONE identical AI task

This is extremely important.

Don't ask Copilot one question and Cursor a different question. Otherwise the comparison is meaningless.

Use **exactly this prompt for all three AI tools**:

```text
Review this ASP.NET Core application.

Task:
Add input validation to the /discount endpoint.

Requirements:
1. Age must be between 0 and 120 inclusive.
2. If age is outside this range, return HTTP 400.
3. The response must contain:
   { "error": "Age must be between 0 and 120." }
4. Do not add any external NuGet packages.
5. Do not change the existing behavior for valid ages.
6. Keep the implementation simple and maintainable.
7. Explain the changes you made.
8. Identify any security or code-quality concerns.

Do not modify unrelated code.
```

Save this prompt somewhere.

Call it:

```text
Lab01_Prompt.txt
```

---

## Step 7 — Test GitHub Copilot first

Open the project folder in VS Code:

```powershell
code .
```

Your folder should be:

```text
C:\AI_SDLC_Labs\Lab01Api
```

Open:

```text
Program.cs
```

Open **GitHub Copilot Chat**.

Give Copilot the prompt above.

Do **not** immediately assume Copilot is correct.

Look at what it wants to change.

A reasonable solution may look approximately like:

```csharp
app.MapGet("/discount", (int age) =>
{
    if (age < 0 || age > 120)
    {
        return Results.BadRequest(new
        {
            error = "Age must be between 0 and 120."
        });
    }

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

Don't worry if Copilot's solution looks different.

**The purpose of the lab is to evaluate it.**

---

### Step 8 — Verify Copilot's code

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
dotnet run --urls http://localhost:5000
```

Test a valid age:

```powershell
curl.exe -i "http://localhost:5000/discount?age=65"
```

You should get HTTP:

```text
200 OK
```

Test:

```powershell
curl.exe -i "http://localhost:5000/discount?age=-10"
```

You should get:

```text
400 Bad Request
```

Test:

```powershell
curl.exe -i "http://localhost:5000/discount?age=150"
```

Again:

```text
400 Bad Request
```

Test the boundaries:

```powershell
curl.exe -i "http://localhost:5000/discount?age=0"
```

and:

```powershell
curl.exe -i "http://localhost:5000/discount?age=120"
```

Both should be:

```text
200 OK
```

Now you have **objective evidence**, not merely "Copilot says the code works." That matches the validation principle in your guide. 

---

## Step 9 — Record Copilot's result

Create:

```text
Lab01_Comparison.md
```

Put:

| Criteria                 | Copilot | Evidence |
| ------------------------ | ------: | -------- |
| Functional correctness   |      /5 |          |
| Code quality             |      /5 |          |
| Explanation quality      |      /5 |          |
| Security awareness       |      /5 |          |
| Manual correction needed |      /5 |          |

For example:

```text
Functional Correctness: 5/5
Evidence: dotnet build succeeded and all five API tests passed.
```

Don't just write:

```text
5/5 because it looks good.
```

Use actual evidence.

---

## Step 10 — Now test Cursor

First return your code to the original baseline:

```powershell
git restore Program.cs
```

If necessary, check:

```powershell
git status
```

Now open the project in **Cursor**.

Give Cursor **exactly the same prompt**.

Don't improve the prompt.

Don't tell Cursor what Copilot did.

Let Cursor independently solve it.

Then repeat:

```powershell
dotnet build
```

and:

```powershell
dotnet run --urls http://localhost:5000
```

Run exactly the same tests.

Score Cursor.

---

## Step 11 — Test Claude Code

Again restore the baseline:

```powershell
git restore Program.cs
```

Open Claude Code from the project directory if it is installed:

```powershell
claude
```

Give Claude **exactly the same prompt**.

Again test:

```powershell
dotnet build
```

and all the same HTTP requests.

Score Claude.

If you don't currently have Claude Code, **just compare Copilot and Cursor for now**.

---

## Step 12 — Your final Lab 1 result

Your table should eventually look something like this:

| Criteria                   |   Copilot |    Cursor | Claude Code |
| -------------------------- | --------: | --------: | ----------: |
| Functional correctness     |         5 |         5 |           5 |
| Code quality               |         4 |         5 |           5 |
| Explanation                |         4 |         4 |           5 |
| Security awareness         |         3 |         4 |           5 |
| Manual correction required |         4 |         5 |           5 |
| **Total**                  | **20/25** | **23/25** |   **25/25** |

Those scores above are only an **example**. Your participants must score the actual outputs they receive.

Then write something like:

```text
Conclusion

Copilot was effective for quick inline coding assistance.
Cursor performed well when modifying code with repository context.
Claude Code provided the most detailed reasoning and explanation.

The experiment shows that the most suitable AI assistant depends on the
engineering task rather than one tool being universally superior.
```

That last point is important because your original Lab 1 specifically requires participants to distinguish strengths by use case rather than simply declare one tool universally superior. 

### The Lab 1 flow is therefore simply:

**Create tiny API → establish bad behavior → give identical problem to Copilot → test → score → restore baseline → Cursor → test → score → restore → Claude → test → score → compare.**

This is the level of detail I think the **entire 30-lab guide should use**. The current guide starts too far ahead for a classroom participant who doesn't already have the repository and environment prepared.
