## Lab 2 — Create and Run Your First .NET Web API

### Objective

By the end of this lab, you will be able to:

* Create a .NET Web API from scratch.
* Run the API locally.
* Test an existing API endpoint.
* Use Cursor/GitHub Copilot/Claude Code to create a new `/status` endpoint.
* Test the AI-generated endpoint.
* Understand what the generated code does.

This still matches the original course's goal of using AI for code generation, explanation and documentation. 

---

# Part 1 — Check whether .NET is installed

Open **Command Prompt**, **PowerShell**, or the terminal inside Cursor.

Run:

```powershell
dotnet --version
```

You should get something similar to:

```text
10.0.100
```

or another installed SDK version.

Microsoft's current ASP.NET Core tutorial uses **.NET 10 SDK**, which is the current LTS release. ([Microsoft Learn][1])

If you get:

```text
'dotnet' is not recognized
```

then the .NET SDK is not installed yet.

---

# Part 2 — Create a folder for the labs

For Windows:

```powershell
cd C:\
mkdir AI_SDLC_Labs
cd AI_SDLC_Labs
```

Your location should now be:

```text
C:\AI_SDLC_Labs
```

---

# Part 3 — Create your first Web API

Run:

```powershell
dotnet new webapi -o Lab02Api
```

Microsoft documents `dotnet new webapi` as the standard template command for creating an ASP.NET Core Web API. ([Microsoft Learn][2])

You should see something like:

```text
The template "ASP.NET Core Web API" was created successfully.

Processing post-creation actions...
Restoring...
Restore succeeded.
```

You now have:

```text
C:\AI_SDLC_Labs
│
└── Lab02Api
    ├── Program.cs
    ├── Lab02Api.csproj
    ├── appsettings.json
    ├── appsettings.Development.json
    ├── Properties
    └── ...
```

**You have created a .NET Web API.**

---

# Part 4 — Enter the project

Run:

```powershell
cd Lab02Api
```

Your terminal should now show something similar to:

```text
PS C:\AI_SDLC_Labs\Lab02Api>
```

---

# Part 5 — Build the API

Before running it, check whether the code compiles:

```powershell
dotnet build
```

Look for:

```text
Build succeeded.
```

That means the C# code is valid.

---

# Part 6 — Run the Web API

Now execute:

```powershell
dotnet run
```

`dotnet run` builds and runs the project directly from its source code. ([Microsoft Learn][3])

You should eventually see something similar to:

```text
Building...

info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5187

info: Microsoft.Hosting.Lifetime[0]
      Application started.
      Press Ctrl+C to shut down.
```

**Do not close this terminal.**

Your Web API is now running.

Think of it like this:

```text
Your Computer
      ↓
.NET Web API
      ↓
http://localhost:5187
```

Your port number may be different.

---

# Part 7 — Test the existing API

The default Web API template contains a sample endpoint called:

```text
/weatherforecast
```

Microsoft's .NET 10 template currently includes this endpoint as the sample API. ([Microsoft Learn][1])

Suppose your terminal says:

```text
Now listening on: http://localhost:5187
```

Open your browser and enter:

```text
http://localhost:5187/weatherforecast
```

You should receive JSON resembling:

```json
[
  {
    "date": "2026-08-11",
    "temperatureC": 25,
    "temperatureF": 76,
    "summary": "Warm"
  }
]
```

Congratulations — **you have now created, run and called a .NET Web API.**

The flow is:

```text
Browser
   │
   │ GET /weatherforecast
   ▼
.NET Web API
   │
   ▼
Program.cs
   │
   ▼
WeatherForecast code
   │
   ▼
JSON response
```

---

# Part 8 — Open the project in Cursor

Now open **Cursor**.

Select:

**File → Open Folder**

Open:

```text
C:\AI_SDLC_Labs\Lab02Api
```

You should see files such as:

```text
Lab02Api
├── Properties
├── appsettings.json
├── appsettings.Development.json
├── Lab02Api.csproj
└── Program.cs
```

Open:

```text
Program.cs
```

This is the most important file for our simple lab.

---

# Part 9 — Understand `Program.cs`

You will see code containing something like:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenApi();

var app = builder.Build();
```

Later you'll see:

```csharp
app.MapGet("/weatherforecast", () =>
{
    ...
});
```

And at the bottom:

```csharp
app.Run();
```

Microsoft's template uses `MapGet` to map an HTTP GET request to code that produces a response. ([Microsoft Learn][1])

For now, understand only three things:

```text
WebApplication.CreateBuilder
        ↓
Create the application

app.MapGet(...)
        ↓
Create an API endpoint

app.Run()
        ↓
Start the Web API
```

You don't need to understand everything yet.

---

# Part 10 — Now bring AI into the lab

This is where **Lab 2 actually starts becoming an AI lab**.

In Cursor Chat, enter:

```text
I am a beginner learning ASP.NET Core Web API.

Please examine my Program.cs.

I want to create a new GET endpoint:

GET /status

The endpoint should return JSON containing:

1. status
2. current UTC timestamp
3. application version

Before changing any code:

1. Explain your proposed solution.
2. Tell me which file you will modify.
3. Do not add unnecessary packages or dependencies.
4. Use the simplest implementation suitable for a beginner.
```

This corresponds to the intended Lab 2 workflow: have AI propose the smallest implementation before writing code.

---

# Part 11 — Ask Cursor to implement it

After reviewing the answer, tell Cursor:

```text
Implement the /status endpoint in Program.cs.

Keep the existing /weatherforecast endpoint.

Use:
status = "Healthy"
version = "1.0.0"

For the timestamp, use the current UTC time.

Keep the implementation simple.
```

A simple implementation might look like:

```csharp
app.MapGet("/status", () =>
{
    return new
    {
        status = "Healthy",
        timestampUtc = DateTime.UtcNow,
        version = "1.0.0"
    };
});
```

You do **not** need to type this yourself if the purpose of the lab is practicing AI-assisted coding.

The important thing is understanding what AI changed.

---

# Part 12 — Save the code

In Cursor:

```text
Ctrl + S
```

Then stop the existing API if it is still running:

```text
Ctrl + C
```

---

# Part 13 — Build again

Run:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

If you get an error, **don't immediately ask AI to rewrite everything**.

Copy the error and ask Cursor:

```text
dotnet build produced the following error:

[paste error]

Explain the cause in beginner-friendly language.

Tell me the smallest change required to fix it.
Do not rewrite unrelated code.
```

This is much closer to how developers should use AI.

---

# Part 14 — Run the modified API

Run:

```powershell
dotnet run
```

Again, look for:

```text
Now listening on: http://localhost:5187
```

Your number can be different.

---

# Part 15 — Test your new `/status` API

Open:

```text
http://localhost:5187/status
```

You should receive something similar to:

```json
{
  "status": "Healthy",
  "timestampUtc": "2026-08-10T00:45:31.125Z",
  "version": "1.0.0"
}
```

Now you've completed the core technical part of Lab 2.

---

# Part 16 — Understand what just happened

Ask Cursor:

```text
Explain this code to me as if I have never developed a .NET Web API before:

app.MapGet("/status", () =>
{
    return new
    {
        status = "Healthy",
        timestampUtc = DateTime.UtcNow,
        version = "1.0.0"
    };
});

Explain:

1. What MapGet means.
2. What /status means.
3. What () => means.
4. Why this becomes JSON.
5. What DateTime.UtcNow means.
```

This is an important part of the course. **AI should not only generate code; it should teach the developer what the generated code is doing.**

---

# Part 17 — Understand the request flow

Conceptually:

```text
                    GET
Browser ───────────────────────────► /status
                                      │
                                      ▼
                                ASP.NET Core
                                      │
                                      ▼
                              app.MapGet(...)
                                      │
                                      ▼
                           Execute C# function
                                      │
                                      ▼
                         Create C# object
                                      │
                                      ▼
                         Convert object → JSON
                                      │
                                      ▼
Browser ◄────────────────────────── JSON
```

For example:

```text
GET http://localhost:5187/status
```

results in:

```json
{
  "status": "Healthy",
  "timestampUtc": "...",
  "version": "1.0.0"
}
```

---

# Part 18 — Use AI to generate documentation

Now ask Cursor:

```text
Create a short README section documenting the /status endpoint.

Include:

- Purpose
- HTTP method
- URL
- Example request
- Example JSON response
- How to run the API locally
- How to test the endpoint

Keep it suitable for a junior developer.
```

The output should describe something like:

```markdown
## Status API

### Endpoint

GET /status

### Purpose

Checks whether the application is running.

### Example

GET http://localhost:5187/status

### Response

{
  "status": "Healthy",
  "timestampUtc": "2026-08-10T00:45:31Z",
  "version": "1.0.0"
}
```

---

# Part 19 — Your final Lab 2 result

At the end, your project should be roughly:

```text
Lab02Api
│
├── Program.cs
├── README.md
├── Lab02Api.csproj
├── appsettings.json
└── Properties
```

And you should have **two working API endpoints**:

```text
GET /weatherforecast
GET /status
```

Test:

```text
http://localhost:xxxx/weatherforecast
```

and:

```text
http://localhost:xxxx/status
```

## What you actually learned

The lab is not really about becoming an ASP.NET expert. It teaches this AI-assisted SDLC workflow:

```text
Requirement
    ↓
Ask AI for implementation plan
    ↓
Review AI proposal
    ↓
AI generates code
    ↓
dotnet build
    ↓
Fix errors
    ↓
dotnet run
    ↓
Test API
    ↓
Ask AI to explain code
    ↓
Ask AI to document code
    ↓
Human validates result
```

That is much better aligned with your course than assuming participants already know how to create the Web API.

### Five commands you need to remember

```powershell
dotnet --version
```

```powershell
dotnet new webapi -o Lab02Api
```

```powershell
cd Lab02Api
```

```powershell
dotnet build
```

```powershell
dotnet run
```

**I would revise Lab 2 in the 30-Lab guide to include Parts 1–19 above.** The same issue probably applies to several later labs: if the learners are expected to use SonarQube, Jenkins, .NET testing and Angular, the labs should include the environment creation/setup commands instead of saying “prerequisite: a working project.”

[1]: https://learn.microsoft.com/en-us/aspnet/core/tutorials/min-web-api?view=aspnetcore-10.0&utm_source=chatgpt.com "Tutorial: Create a Minimal API with ASP.NET Core | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-new-sdk-templates?utm_source=chatgpt.com ".NET default templates for 'dotnet new' - .NET CLI | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-run?utm_source=chatgpt.com "dotnet run command - .NET CLI | Microsoft Learn"
