**NOT** starting with database MCP, GitHub MCP, Kubernetes MCP, or a complicated external service.

Use something you already understand: **Lab02Api**.

The simplest useful Lab 31 is:

# Bonus Lab 5 — Integrate Lab02Api with GitHub Copilot Using MCP

### Use case

GitHub Copilot should be able to ask your running Lab02Api questions through MCP:

```text
“Check whether Lab02Api is healthy.”

“What discount does a 65-year-old receive?”

“Compare the discount for ages 59, 60 and 61.”
```

The architecture will be:

```text
┌─────────────────────────────┐
│ VS Code                     │
│ GitHub Copilot Chat         │
│                             │
│ User: "Check API status"    │
└─────────────┬───────────────┘
              │
              │ MCP
              ▼
┌─────────────────────────────┐
│ Lab31McpServer              │
│                             │
│ MCP Tools:                  │
│ • CheckApiStatus            │
│ • CalculateDiscount         │
└─────────────┬───────────────┘
              │
              │ HTTP
              ▼
┌─────────────────────────────┐
│ Lab02Api                    │
│ http://localhost:5050       │
│                             │
│ GET /status                 │
│ GET /discount?age=65        │
└─────────────┬───────────────┘
              │
              ▼
          JSON Result
              │
              ▼
      MCP → GitHub Copilot
              │
              ▼
       Human-friendly answer
```

This is an excellent first MCP lab because you can **see exactly what MCP is doing**.

---

# 1. What Are We Learning?

By the end of Lab 31, you should understand:

* What MCP is.
* What an MCP Client is.
* What an MCP Server is.
* What an MCP Tool is.
* How GitHub Copilot calls an MCP tool.
* How an MCP tool calls your .NET REST API.
* How arguments such as `age=65` are passed.
* How API JSON comes back to Copilot.
* How to troubleshoot an MCP connection.
* Why MCP is different from a normal REST API.

---

# 2. First Understand MCP in Simple Language

MCP means:

**Model Context Protocol**

Think of it like a standard connector between AI and tools.

Without MCP:

```text
GitHub Copilot

"I know how to generate code,
but how exactly should I interact
with your Lab02Api?"
```

With MCP:

```text
GitHub Copilot
      ↓
MCP Server tells Copilot:

"I have these tools:

1. CheckApiStatus()
2. CalculateDiscount(age)"
      ↓
Copilot can select
and execute the tool
```

---

# 3. API vs MCP

Do not confuse these.

### REST API

Your existing Lab02Api provides:

```text
GET /status

GET /discount?age=65
```

An application can call these URLs.

### MCP Server

The MCP Server takes those APIs and exposes them to an AI as understandable **tools**:

```text
REST API

GET /status

           becomes

MCP Tool

CheckApiStatus()
```

and:

```text
REST API

GET /discount?age=65

           becomes

MCP Tool

CalculateDiscount(age=65)
```

So:

> **REST API is for applications. MCP makes capabilities available as structured tools to AI clients such as GitHub Copilot.**

---

# 4. What Are the Components?

For our Lab:

| Component           | What it is             |
| ------------------- | ---------------------- |
| GitHub Copilot      | AI assistant           |
| VS Code             | MCP client environment |
| Lab31McpServer      | MCP server we build    |
| `CheckApiStatus`    | MCP tool               |
| `CalculateDiscount` | MCP tool               |
| Lab02Api            | Existing REST API      |
| `/status`           | API endpoint           |
| `/discount`         | API endpoint           |

The important relationship is:

```text
COPILOT
MCP CLIENT
    ↓
MCP SERVER
    ↓
MCP TOOL
    ↓
REST API
```

---

# PART A — Prepare the Project

## Step 1 — Use this folder structure

For Lab 31, I recommend:

```text
C:\AI_SDLC_Labs
│
├── Lab02Api
│   ├── Program.cs
│   ├── OrderService.cs
│   ├── CustomerReportService.cs
│   └── Lab02Api.csproj
│
├── Lab02Api.Tests
│
├── Lab31McpServer
│   ├── Program.cs
│   └── Lab31McpServer.csproj
│
└── .vscode
    └── mcp.json
```

Notice:

```text
Lab31McpServer
```

is **beside** Lab02Api.

Do not put the second .NET project inside `Lab02Api`.

---

# Step 2 — Open the parent folder in VS Code

Instead of opening:

```text
C:\AI_SDLC_Labs\Lab02Api
```

open:

```text
C:\AI_SDLC_Labs
```

In VS Code:

```text
File
 ↓
Open Folder
 ↓
C:\AI_SDLC_Labs
```

Now you can see:

```text
AI_SDLC_Labs
├── Lab02Api
├── Lab02Api.Tests
└── ...
```

---

# Step 3 — Create a Git branch

Open VS Code Terminal:

```text
Terminal
 ↓
New Terminal
```

Run:

```powershell
cd C:\AI_SDLC_Labs
```

Then:

```powershell
git checkout -b lab31-mcp-integration
```

Check:

```powershell
git status
```

---

# PART B — Check Lab02Api

Before touching MCP, prove that the API works.

This is critical.

```text
First:

REST API works

Then:

MCP calls REST API
```

Not:

```text
API broken
+
MCP broken

= impossible troubleshooting
```

---

# Step 4 — Check `/status`

Open:

```text
Lab02Api\Program.cs
```

Make sure you have a status endpoint.

If you already created it in Lab 2, keep your existing implementation.

If it is missing, add:

```csharp
app.MapGet("/status", () =>
{
    return Results.Ok(new
    {
        status = "Healthy",
        timestamp = DateTime.UtcNow,
        version = "1.0.0"
    });
});
```

Do not create a second `/status` endpoint if one already exists.

---

# Step 5 — Check `/discount`

You should already have something similar to:

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

For Lab 31 we are going to **consume this endpoint**, not redesign it.

---

# Step 6 — Build Lab02Api

Run:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Then:

```powershell
dotnet build
```

Expected:

```text
Build succeeded.
```

If build fails, fix that before continuing.

---

# PART C — Start Lab02Api on a Known Port

This is important.

Normally `dotnet run` may use a port from your development settings.

For an MCP lab, we want a predictable URL:

```text
http://localhost:5050
```

## Step 7 — Start Lab02Api

Run:

```powershell
dotnet run --urls http://localhost:5050
```

Keep this terminal running.

You should see something similar to:

```text
Now listening on:

http://localhost:5050
```

Do not close this terminal.

---

# Step 8 — Open another terminal

In VS Code:

```text
Terminal
 ↓
New Terminal
```

Now you'll have:

```text
Terminal 1
Lab02Api running


Terminal 2
Commands/testing
```

---

# PART D — Test the API Without MCP

## Step 9 — Test `/status`

From Terminal 2:

```powershell
Invoke-RestMethod http://localhost:5050/status
```

You should receive something similar to:

```text
status  timestamp                 version
------  ---------                 -------
Healthy 2026-...                  1.0.0
```

The exact timestamp will differ.

---

# Step 10 — Test `/discount`

Run:

```powershell
Invoke-RestMethod "http://localhost:5050/discount?age=65"
```

You should get something similar to:

```text
age discount
--- --------
65  0.2
```

Meaning:

```text
0.2
=
20%
```

Test:

```powershell
Invoke-RestMethod "http://localhost:5050/discount?age=59"
```

Expected from the existing rule:

```text
age = 59
discount = 0
```

At this point:

```text
Lab02Api ✓

/status ✓

/discount ✓
```

Now we are ready for MCP.

---

# PART E — Create the MCP Server

Because your application is already **.NET**, I recommend creating the MCP server in **.NET too**.

That means:

```text
No Python required

No Node.js required

No JavaScript required
```

You stay in the same technology.

---

# Step 11 — Create the MCP project

Go back to:

```powershell
cd C:\AI_SDLC_Labs
```

Run:

```powershell
dotnet new console -o Lab31McpServer
```

You should now have:

```text
C:\AI_SDLC_Labs
│
├── Lab02Api
│
└── Lab31McpServer
    ├── Program.cs
    └── Lab31McpServer.csproj
```

---

# Step 12 — Enter the project

```powershell
cd .\Lab31McpServer
```

Build once:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

---

# PART F — Install the MCP SDK

## Step 13 — Add the MCP package

Run:

```powershell
dotnet add package ModelContextProtocol
```

Then:

```powershell
dotnet add package Microsoft.Extensions.Hosting
```

If your NuGet feed reports that only a prerelease of the MCP SDK is available in your environment, use:

```powershell
dotnet add package ModelContextProtocol --prerelease
```

You only need to do that if the normal package command tells you there is no compatible stable version.

---

# Step 14 — Check packages

Run:

```powershell
dotnet package list
```

You should see the MCP and hosting packages in the project dependency list.

---

# PART G — Create Your First MCP Tool

## Step 15 — Open

```text
Lab31McpServer
 ↓
Program.cs
```

Delete the default:

```csharp
Console.WriteLine("Hello, World!");
```

Replace `Program.cs` with:

```csharp
using System.ComponentModel;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using ModelContextProtocol.Server;

var builder = Host.CreateApplicationBuilder(args);

// MCP over stdio uses standard output for protocol messages.
// Avoid ordinary console logging interfering with the protocol.
builder.Logging.ClearProviders();

builder.Services
    .AddMcpServer()
    .WithStdioServerTransport()
    .WithToolsFromAssembly();

await builder.Build().RunAsync();


[McpServerToolType]
public static class Lab02ApiTools
{
    private const string BaseUrl = "http://localhost:5050";

    [McpServerTool]
    [Description("Checks whether Lab02Api is healthy by calling its /status endpoint.")]
    public static async Task<string> CheckApiStatus()
    {
        try
        {
            using var client = new HttpClient();

            var result =
                await client.GetStringAsync(
                    $"{BaseUrl}/status");

            return result;
        }
        catch (Exception ex)
        {
            return
                $"ERROR: Could not reach Lab02Api at {BaseUrl}. " +
                $"Details: {ex.Message}";
        }
    }


    [McpServerTool]
    [Description("Gets the discount for a customer age from the Lab02Api /discount endpoint.")]
    public static async Task<string> CalculateDiscount(
        [Description("Age of the customer.")]
        int age)
    {
        try
        {
            using var client = new HttpClient();

            var url =
                $"{BaseUrl}/discount?age={age}";

            var response =
                await client.GetAsync(url);

            var body =
                await response.Content.ReadAsStringAsync();

            if (!response.IsSuccessStatusCode)
            {
                return
                    $"Lab02Api returned HTTP " +
                    $"{(int)response.StatusCode}: {body}";
            }

            return body;
        }
        catch (Exception ex)
        {
            return
                $"ERROR: Could not reach Lab02Api at {BaseUrl}. " +
                $"Details: {ex.Message}";
        }
    }
}
```

---

# 16. What Did We Just Create?

This:

```csharp
[McpServerToolType]
```

means:

> This class contains tools that MCP can expose.

---

This:

```csharp
[McpServerTool]
```

means:

> Make this method available as an AI tool.

So:

```csharp
CheckApiStatus()
```

becomes conceptually:

```text
MCP Tool:

Check API Status
```

and:

```csharp
CalculateDiscount(int age)
```

becomes:

```text
MCP Tool:

Calculate Discount

Input:
age
```

---

# 17. Why Do We Add Description?

This:

```csharp
[Description(
    "Checks whether Lab02Api is healthy..."
)]
```

is especially important for AI.

It tells Copilot:

> This is what this tool is for.

Think:

```text
Tool Name
+
Tool Description
+
Input Parameters
        ↓
Copilot understands
when to use the tool
```

---

# 18. Why Does the MCP Server Call HTTP?

This code:

```csharp
await client.GetStringAsync(
    $"{BaseUrl}/status");
```

does exactly the same thing you previously did manually:

```powershell
Invoke-RestMethod http://localhost:5050/status
```

The difference is:

```text
BEFORE

Human
 ↓
PowerShell
 ↓
API


AFTER

Copilot
 ↓
MCP Tool
 ↓
API
```

---

# PART H — Build the MCP Server

## Step 19 — Build

Run:

```powershell
cd C:\AI_SDLC_Labs\Lab31McpServer
```

Then:

```powershell
dotnet build
```

You want:

```text
Build succeeded.
```

Do not configure VS Code MCP until the MCP project builds.

---

# Step 20 — Important: Don't Expect a Normal Screen

An MCP stdio server is not a normal web application.

If you manually run:

```powershell
dotnet run
```

it may appear to:

```text
sit there
and do nothing
```

That can be normal.

It is waiting for an MCP client to communicate through its standard input/output channel.

For this lab, let **VS Code start it**.

---

# PART I — Configure VS Code to Use Our MCP Server

Now we're going to tell VS Code:

> I have an MCP server called `lab02api-mcp`.

## Step 21 — Go to the workspace root

Your VS Code folder should be:

```text
C:\AI_SDLC_Labs
```

Create:

```text
.vscode
```

if it does not exist.

Inside it create:

```text
mcp.json
```

So:

```text
C:\AI_SDLC_Labs
│
├── .vscode
│   └── mcp.json
│
├── Lab02Api
│
└── Lab31McpServer
```

---

# Step 22 — Configure `mcp.json`

Put:

```json
{
    "servers": {
        "lab02api-mcp": {
            "type": "stdio",
            "command": "dotnet",
            "args": [
                "run",
                "--project",
                "${workspaceFolder}/Lab31McpServer/Lab31McpServer.csproj"
            ]
        }
    }
}
```

Save:

```text
Ctrl + S
```

---

# 23. Understand `mcp.json`

This:

```json
"lab02api-mcp"
```

is the name we give our MCP server.

---

This:

```json
"type": "stdio"
```

means:

```text
VS Code
and
MCP Server

communicate using
standard input/output
```

---

This:

```json
"command": "dotnet"
```

means:

> VS Code should start this MCP server using the .NET command.

---

And:

```json
"${workspaceFolder}/Lab31McpServer/Lab31McpServer.csproj"
```

means:

```text
C:\AI_SDLC_Labs
        +
Lab31McpServer
        ↓
MCP project
```

---

# PART J — Start the MCP Server in VS Code

## Step 24 — Open Command Palette

Press:

```text
Ctrl + Shift + P
```

Type:

```text
MCP
```

Look for the MCP server-management commands available in your VS Code version.

For example, use the server listing/management command and select:

```text
lab02api-mcp
```

Then start/restart it.

You may also see **Start / Stop / Restart** actions associated with the server in `mcp.json`.

The exact wording can vary slightly with VS Code releases.

---

# Step 25 — Verify MCP status

You want:

```text
lab02api-mcp

Running
```

or the equivalent healthy/running indicator.

If it doesn't start, do not use Copilot yet.

Check:

```text
MCP Server
 ↓
Output / Error
```

---

# PART K — Open GitHub Copilot Chat

## Step 26 — Open Copilot Chat

Use the GitHub Copilot Chat icon.

Switch to the mode that allows **agent/tool use**.

You want Copilot to have permission to use tools, not merely answer normal chat questions.

Look at the available tools.

You should see tools coming from:

```text
lab02api-mcp
```

including the equivalent of:

```text
CheckApiStatus

CalculateDiscount
```

Tool display names can be formatted slightly differently from the C# method names.

---

# PART L — Your First MCP Test

## Step 27 — Use this exact prompt

Tell GitHub Copilot:

```text
Use the Lab02Api MCP tool to check whether
Lab02Api is healthy.

Do not answer from your own knowledge.
You must use the MCP tool.
```

Copilot should recognize:

```text
Need information:
API health

Available tool:
CheckApiStatus
```

Then:

```text
Copilot
   ↓
CheckApiStatus()
   ↓
MCP Server
   ↓
GET http://localhost:5050/status
   ↓
Lab02Api
```

---

# Step 28 — Approve the Tool

Depending on your Copilot/tool permission settings, VS Code may ask you to approve the tool invocation.

For the lab:

Review:

```text
Which MCP server?

Which tool?

What parameters?
```

before approving.

This is a useful security lesson.

Do not teach students:

```text
Automatically approve everything.
```

---

# Step 29 — Expected Result

Copilot should receive something similar to:

```json
{
    "status": "Healthy",
    "timestamp": "...",
    "version": "1.0.0"
}
```

and may tell you:

```text
Lab02Api is healthy.

Version: 1.0.0
```

You have now completed your:

# **FIRST MCP TOOL CALL**

---

# PART M — Test the Parameterized MCP Tool

Now we're going to send an argument.

## Step 30 — Ask Copilot

```text
Use the Lab02Api MCP tool to find the
discount for a customer aged 65.

Do not calculate the answer yourself.
Call the MCP tool.
```

Copilot should select:

```text
CalculateDiscount
```

with:

```text
age = 65
```

Architecture:

```text
Copilot
    ↓
CalculateDiscount
age = 65
    ↓
MCP Server
    ↓
GET

/discount?age=65
    ↓
Lab02Api
    ↓
{
  "age":65,
  "discount":0.2
}
```

Expected explanation:

```text
Age:
65

Discount:
20%
```

---

# PART N — Boundary Testing Through MCP

This makes Lab 31 much more useful.

## Step 31 — Ask:

```text
Use the Lab02Api MCP tools to check
discounts for these ages:

59
60
61

Create a table showing:

Age
Discount

Do not calculate the discounts yourself.
Call the MCP tool for each age.
```

Copilot should call the tool three times.

Expected based on your existing rule:

| Age | Discount |
| --: | -------: |
|  59 |       0% |
|  60 |      20% |
|  61 |      20% |

Now MCP is doing something useful:

```text
Natural language
      ↓
AI decides which tool
      ↓
AI supplies parameters
      ↓
API gets called
      ↓
Results summarized
```

---

# PART O — Prove That Copilot Is Really Using the API

This is an important experiment.

## Step 32 — Stop Lab02Api

Go to Terminal 1 where:

```text
dotnet run
```

is running.

Press:

```text
Ctrl + C
```

Now Lab02Api is stopped.

---

# Step 33 — Ask Copilot again

```text
Use the Lab02Api MCP tool to check API status.
```

Now the MCP tool should report that it cannot reach:

```text
http://localhost:5050
```

This proves:

```text
Copilot is NOT inventing
the API result.

It really called the tool.
```

---

# Step 34 — Restart Lab02Api

Run:

```powershell
cd C:\AI_SDLC_Labs\Lab02Api
```

Then:

```powershell
dotnet run --urls http://localhost:5050
```

Ask again:

```text
Use the Lab02Api MCP tool to check the API status.
```

Now it should work again.

Excellent troubleshooting exercise.

---

# PART P — Ask Copilot to Explain the MCP Flow

Now use AI as the teacher:

```text
Explain exactly what just happened when you
used the CheckApiStatus MCP tool.

Explain these components:

1. GitHub Copilot
2. VS Code as MCP client
3. mcp.json
4. Lab31McpServer
5. CheckApiStatus tool
6. HttpClient
7. Lab02Api
8. /status endpoint
9. JSON response

Use beginner-friendly language.

Do not modify any files.
```

You should understand this:

```text
QUESTION

"Is Lab02Api healthy?"

       ↓

GITHUB COPILOT

"I need a tool."

       ↓

MCP CLIENT
VS CODE

       ↓

MCP SERVER
Lab31McpServer

       ↓

TOOL
CheckApiStatus()

       ↓

HTTP GET

       ↓

Lab02Api
/status

       ↓

JSON

       ↓

MCP SERVER

       ↓

COPILOT

       ↓

HUMAN-FRIENDLY ANSWER
```

---

# PART Q — Ask Copilot to Review the MCP Server

Use:

```text
You are an MCP and .NET code reviewer.

Review:

Lab31McpServer/Program.cs

Do NOT modify it.

Check:

1. Are the MCP tools clearly described?
2. Do tool parameters make sense?
3. Does the MCP layer contain unnecessary
   business logic?
4. Does it safely handle Lab02Api being unavailable?
5. Does it expose secrets?
6. Does it make unnecessary external calls?
7. Is the MCP server doing only what the
   tool description says?
8. Could the tools be made simpler?

Classify findings:

MUST FIX
SHOULD FIX
OPTIONAL

Do not invent security problems.
```

---

# PART R — Important Architecture Rule

Try to keep:

```text
BUSINESS LOGIC

inside:

Lab02Api
```

not:

```text
Lab31McpServer
```

For example, don't do this inside MCP:

```csharp
if (age >= 60)
{
    return "20%";
}
```

Why?

Then you have:

```text
Lab02Api

age >= 60 → 20%

AND

MCP server

age >= 60 → 20%
```

Now the business rule exists twice.

Bad architecture:

```text
API Business Rule
       +
MCP Business Rule
       ↓
Duplication
       ↓
Possible inconsistency
```

Better:

```text
MCP
 ↓
Calls API
 ↓
API owns business rule
```

This is an important lesson.

---

# PART S — Understand Why This is Better Than Copilot Calling a URL in a Prompt

Without MCP:

```text
User:

"Go to localhost:5050/discount..."
```

Copilot may not have an appropriate structured mechanism for that local service.

With MCP:

```text
Available Tool:

CalculateDiscount

Description:
Gets customer discount from Lab02Api.

Input:
age : integer
```

The AI has a **defined contract**.

That is the power of MCP.

---

# PART T — Controlled Test Cases

Run these in Copilot.

### Test 1 — Health

```text
Use MCP to check Lab02Api health.
```

Expected:

```text
Healthy
```

---

### Test 2 — No discount

```text
Use MCP to check discount for age 30.
```

Expected:

```text
0%
```

---

### Test 3 — Boundary below

```text
Use MCP to check discount for age 59.
```

Expected:

```text
0%
```

---

### Test 4 — Boundary

```text
Use MCP to check discount for age 60.
```

Expected:

```text
20%
```

---

### Test 5 — Above boundary

```text
Use MCP to check discount for age 65.
```

Expected:

```text
20%
```

---

### Test 6 — API unavailable

Stop Lab02Api.

Ask:

```text
Use MCP to check Lab02Api health.
```

Expected:

```text
Connection/API unavailable error
```

Then restart Lab02Api.

---

# PART U — Record Your Results

Create:

```text
LAB31_MCP_TEST_RESULTS.md
```

Use:

```markdown
# Lab 31 — MCP Test Results

## Environment

VS Code:
Installed

GitHub Copilot:
Connected

Lab02Api:
http://localhost:5050

MCP Server:
Lab31McpServer

MCP Server Status:
Running

## Tool 1

Tool:
CheckApiStatus

Purpose:
Call GET /status

Result:
PASS / FAIL

Actual Response:
[...]

## Tool 2

Tool:
CalculateDiscount

Input:
age = 65

Result:
PASS / FAIL

Actual Response:
[...]

## Boundary Tests

| Age | Expected | Actual | Result |
|---:|---:|---:|---|
| 59 | 0% | | |
| 60 | 20% | | |
| 61 | 20% | | |

## Failure Test

Lab02Api stopped:
Yes

MCP Result:
[...]

Expected:
Connection failure reported

Result:
PASS / FAIL

## Recovery Test

Lab02Api restarted:
Yes

MCP status after restart:
[...]

Result:
PASS / FAIL
```

---

# PART V — Commit Lab 31

After everything works:

```powershell
cd C:\AI_SDLC_Labs
```

Check:

```powershell
git status
```

Add:

```powershell
git add .
```

Commit:

```powershell
git commit -m "Add Lab 31 Lab02Api MCP integration"
```

Push:

```powershell
git push -u origin lab31-mcp-integration
```

---

# PART W — Lab 31 Final Deliverables

At the end you should have:

```text
AI_SDLC_Labs
│
├── Lab02Api
│
├── Lab02Api.Tests
│
│
├── Lab31McpServer
│   ├── Program.cs
│   └── Lab31McpServer.csproj
│
├── .vscode
│   └── mcp.json
│
├── LAB31_MCP_TEST_RESULTS.md
│
└── ...
```

---

# Lab 31 Completion Checklist

* [ ] Understand what MCP means.
* [ ] Understand MCP Client.
* [ ] Understand MCP Server.
* [ ] Understand MCP Tool.
* [ ] Lab02Api builds.
* [ ] `/status` works.
* [ ] `/discount` works.
* [ ] Lab02Api runs on port `5050`.
* [ ] Lab31McpServer project created.
* [ ] MCP SDK installed.
* [ ] `CheckApiStatus` MCP tool created.
* [ ] `CalculateDiscount` MCP tool created.
* [ ] MCP Server builds.
* [ ] `.vscode/mcp.json` created.
* [ ] VS Code recognizes `lab02api-mcp`.
* [ ] MCP server starts.
* [ ] GitHub Copilot sees MCP tools.
* [ ] Copilot calls `CheckApiStatus`.
* [ ] Copilot calls `CalculateDiscount`.
* [ ] Age 59 tested.
* [ ] Age 60 tested.
* [ ] Age 61 tested.
* [ ] API-offline failure tested.
* [ ] API restarted.
* [ ] MCP works after recovery.
* [ ] MCP code reviewed with Copilot.
* [ ] `LAB31_MCP_TEST_RESULTS.md` created.
* [ ] Code committed to Git.

# The One Diagram to Remember

```text
                      YOU
                       │
                       │ Natural Language
                       ▼
              ┌─────────────────┐
              │ GitHub Copilot  │
              │    VS Code      │
              └────────┬────────┘
                       │
                       │ MCP
                       ▼
              ┌─────────────────┐
              │ Lab31McpServer  │
              ├─────────────────┤
              │ MCP Tools       │
              │                 │
              │ CheckApiStatus  │
              │       OR        │
              │ CalculateDiscount│
              └────────┬────────┘
                       │
                       │ HTTP
                       ▼
              ┌─────────────────┐
              │    Lab02Api     │
              │ localhost:5050  │
              ├─────────────────┤
              │ /status         │
              │ /discount       │
              └────────┬────────┘
                       │
                       │ JSON
                       ▼
                MCP → Copilot
                       │
                       ▼
                 ANSWER TO YOU
```

## Recommended follow-on use cases

Once this basic Lab 31 works, it can naturally grow into later MCP labs:

```text
LAB 31
MCP → Lab02Api
Health + Discount
        ↓

LAB 32
MCP API Testing Agent
Copilot automatically tests
59 / 60 / 61
        ↓

LAB 33
MCP + SonarQube
Ask Copilot for project quality findings
        ↓

LAB 34
MCP + Jenkins
Ask:
"Did the latest pipeline pass?"
        ↓

LAB 35
Multi-Tool DevOps Agent

Copilot
 ├── Lab02Api MCP
 ├── GitHub MCP
 ├── Jenkins MCP
 └── SonarQube MCP
```

For **Lab 31**, however, stop at the two tools. The key learning outcome is simply:

> **“I can ask GitHub Copilot a natural-language question, Copilot chooses an MCP tool, the MCP server calls my real Lab02Api, and the actual API response comes back to Copilot.”**

That is the cleanest first MCP use case for `Lab02Api`.
