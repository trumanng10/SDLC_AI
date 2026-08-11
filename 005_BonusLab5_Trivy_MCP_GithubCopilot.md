**Bonus Lab 05 — AI-Assisted Container Security with Trivy MCP + GitHub Copilot**. One important correction: the reusable Copilot skill file must be named exactly **`SKILL.md`**, and project skills can live under `.github/skills/<skill-name>/`. VS Code can automatically load relevant Agent Skills when Copilot needs them. ([GitHub Docs][1])

# Bonus Lab 05 — Container Vulnerability Scanning with Trivy MCP, VS Code and GitHub Copilot

## 1. Lab Objective

In this lab, you will integrate:

```text
Docker
   +
Trivy
   +
Trivy MCP Server
   +
VS Code
   +
GitHub Copilot
   +
Agent Skill
```

You will then use natural language to ask GitHub Copilot to:

```text
Download Container Image
        ↓
Scan Container Image
        ↓
Receive Trivy Results
        ↓
Analyse Vulnerabilities
        ↓
Explain CVEs
        ↓
Prioritise Risks
        ↓
Recommend Remediation
        ↓
Suggest Safer Image
```

Aqua Security provides an official Trivy MCP plugin that exposes Trivy security scanning capabilities to MCP-compatible tools including VS Code. It supports container-image, filesystem and repository scanning. ([GitHub][2])

---

# 2. Lab Architecture

The architecture is:

```text
                 Developer
                     │
                     ▼
                  VS Code
                     │
                     ▼
              GitHub Copilot
                Agent Mode
                     │
                     ▼
              Agent SKILL.md
                     │
                     ▼
              MCP Protocol
                     │
                     ▼
             Trivy MCP Server
                     │
                     ▼
                   Trivy
                     │
                     ▼
               Docker Image
                     │
                     ▼
             Vulnerability DB
                     │
                     ▼
                 CVE Results
                     │
                     ▼
              GitHub Copilot
                     │
           ┌─────────┼──────────┐
           ▼         ▼          ▼
       Explain    Prioritise   Remediate
```

MCP allows Copilot to invoke external tools rather than only discuss them. VS Code supports local MCP servers and makes their tools available to Copilot Chat. ([Visual Studio Code][3])

---

# Part A — Prerequisites

You need:

```text
Windows 10/11

VS Code

GitHub Copilot

GitHub Copilot Chat

Docker Desktop

Internet connection
```

Docker Desktop must be running because this exercise downloads a container image locally.

Check Docker:

```powershell
docker --version
```

Then:

```powershell
docker info
```

If both commands work, continue.

---

# Part B — Install Trivy on Windows

## Step 1 — Download Trivy

Trivy's official Windows installation method is to download the Windows 64-bit ZIP package from Aqua Security's Trivy release page, extract it and make the executable available on your PATH. ([Trivy][4])

Download the latest:

```text
trivy_x.xx.x_windows-64bit.zip
```

For example, your downloaded file may resemble:

```text
trivy_0.xx.x_windows-64bit.zip
```

Do not worry if the version number is different from screenshots in this lab.

---

# Step 2 — Extract Trivy

Create:

```text
C:\Tools\Trivy
```

Extract:

```text
trivy.exe
```

into:

```text
C:\Tools\Trivy
```

You should have:

```text
C:
└── Tools
    └── Trivy
        └── trivy.exe
```

---

# Step 3 — Add Trivy to Windows PATH

Search Windows for:

```text
Edit environment variables for your account
```

Open:

```text
Environment Variables
```

Select:

```text
Path
```

Click:

```text
Edit
```

Then:

```text
New
```

Enter:

```text
C:\Tools\Trivy
```

Click:

```text
OK
```

Close and reopen VS Code after changing PATH.

---

# Step 4 — Verify Trivy

Open the VS Code terminal:

```text
Terminal
→ New Terminal
```

Run:

```powershell
trivy --version
```

Expected result:

```text
Version: 0.xx.x
```

The exact version may be newer.

---

# Part C — Test Trivy Without AI First

Before MCP integration, make sure Trivy itself works.

Run:

```powershell
trivy image python:3.4-alpine
```

Trivy's own getting-started documentation uses `python:3.4-alpine` as a basic container-image scanning example. ([Trivy][5])

You should eventually see a vulnerability report.

It might contain categories such as:

```text
Vulnerability ID

Package

Installed Version

Fixed Version

Severity
```

Possible severities include:

```text
UNKNOWN

LOW

MEDIUM

HIGH

CRITICAL
```

The exact CVEs and number of vulnerabilities will change over time because vulnerability databases are updated.

Therefore:

```text
DO NOT memorise the vulnerability numbers.
```

The objective is learning how to analyse the live results.

---

# Part D — Install Trivy MCP

Trivy and Trivy MCP are different components.

Think of them like this:

```text
Trivy

Security scanning engine
        │
        ▼
Trivy MCP

Allows AI/MCP clients
to communicate with Trivy
        │
        ▼
GitHub Copilot
```

Aqua's official installation command for the MCP plugin is:

```powershell
trivy plugin install mcp
```

([GitHub][2])

---

# Step 5 — Install the MCP Plugin

In VS Code Terminal:

```powershell
trivy plugin install mcp
```

Wait for installation to complete.

---

# Step 6 — Verify the Plugin

Run:

```powershell
trivy plugin list
```

Look for:

```text
mcp
```

Then test:

```powershell
trivy mcp --help
```

The Trivy MCP plugin uses `stdio` transport by default, which is suitable for IDE integrations. ([GitHub][6])

---

# Part E — Connect Trivy MCP to VS Code

Modern VS Code supports workspace MCP configuration using:

```text
.vscode/mcp.json
```

Workspace MCP configuration can be stored with the project and shared with the team. ([Visual Studio Code][3])

---

# Step 7 — Create the MCP Configuration

Inside your project create:

```text
.vscode
```

Then create:

```text
mcp.json
```

Your structure becomes:

```text
Lab05-Trivy
│
└── .vscode
    └── mcp.json
```

---

# Step 8 — Configure the Trivy MCP Server

Put this inside:

```text
.vscode/mcp.json
```

```json
{
    "servers": {
        "trivy": {
            "type": "stdio",
            "command": "trivy",
            "args": [
                "mcp"
            ]
        }
    }
}
```

The important part is:

```text
command:
trivy

arguments:
mcp
```

VS Code is essentially launching:

```powershell
trivy mcp
```

for GitHub Copilot.

---

# Step 9 — If VS Code Cannot Find Trivy

If you get an error similar to:

```text
trivy command not found
```

change:

```json
"command": "trivy"
```

to:

```json
"command": "C:\\Tools\\Trivy\\trivy.exe"
```

Your complete file becomes:

```json
{
    "servers": {
        "trivy": {
            "type": "stdio",
            "command": "C:\\Tools\\Trivy\\trivy.exe",
            "args": [
                "mcp"
            ]
        }
    }
}
```

---

# Step 10 — Start the MCP Server

Press:

```text
Ctrl + Shift + P
```

Search:

```text
MCP: List Servers
```

Select:

```text
trivy
```

Then:

```text
Start Server
```

VS Code may ask whether you trust the MCP server.

Review the configuration and approve it.

Local MCP servers can execute code with your local permissions, so VS Code deliberately uses a trust mechanism before starting them. Only configure MCP servers from sources you trust. ([Visual Studio Code][3])

---

# Step 11 — Check MCP Server Status

Press:

```text
Ctrl + Shift + P
```

Run:

```text
MCP: List Servers
```

You want:

```text
trivy

Running
```

If it fails, select:

```text
Show Output
```

VS Code provides MCP server logs through this command for troubleshooting. ([Visual Studio Code][3])

---

# Part F — Connect GitHub Copilot to the Trivy Tools

## Step 12 — Open Copilot Chat

Use:

```text
Ctrl + Alt + I
```

Open GitHub Copilot Chat.

Select:

```text
Agent
```

Do **not** use:

```text
Ask
```

for this exercise.

Aqua's Trivy MCP documentation specifically instructs users to use Copilot/IDE chat in Agent mode when invoking the MCP scanning tools. ([GitHub][6])

---

# Step 13 — Check Available Tools

In Copilot Chat select:

```text
Configure Tools
```

or the:

```text
Tools
```

icon.

Look for Trivy-related MCP tools.

Make sure they are enabled.

The exact tool names can change between Trivy MCP versions, so do not build the lab around memorising one particular internal MCP function name.

---

# Part G — First Test Using Natural Language

Before creating the skill, test the integration.

Enter into GitHub Copilot:

```text
Use the Trivy MCP tools.

Scan the container image:

python:3.4-alpine

Show me the vulnerabilities found.

Group them by severity.

Do not make any changes yet.
```

GitHub Copilot should call Trivy through MCP and display the results.

Trivy MCP officially supports natural-language requests for container image scanning, including queries asking for vulnerabilities in specific container images. ([GitHub][7])

---

# Part H — Understand What Just Happened

Without MCP:

```text
Human

   ↓

trivy image xxx

   ↓

Terminal report

   ↓

Human interprets report
```

With MCP:

```text
Human

   ↓

"Scan this image"

   ↓

GitHub Copilot

   ↓

MCP

   ↓

Trivy

   ↓

CVE Results

   ↓

Copilot

   ↓

Explanation
```

This is one of the key ideas behind AI-assisted DevSecOps.

---

# Part I — Create a GitHub Copilot Agent Skill

We will now make the process reusable.

GitHub Copilot Agent Skills are directories containing a mandatory file named:

```text
SKILL.md
```

VS Code can detect workspace skills stored under:

```text
.github/skills
```

and load them when the current task matches the skill's description. ([Visual Studio Code][8])

---

# Step 14 — Create the Skill Directory

Create:

```text
.github
```

Then:

```text
skills
```

Then:

```text
trivy-image-scan
```

The final structure is:

```text
Lab05-Trivy
│
├── .vscode
│   └── mcp.json
│
└── .github
    └── skills
        └── trivy-image-scan
            └── SKILL.md
```

The filename must be:

```text
SKILL.md
```

not:

```text
Skills.md
```

and not:

```text
skill.md
```

---

# Step 15 — Create the Simple SKILL.md

Put the following inside:

```text
.github/skills/trivy-image-scan/SKILL.md
```

```markdown
---
name: trivy-image-scan
description: Pulls a deliberately old training container image, scans it using Trivy MCP, explains vulnerabilities, and recommends remediation. Use when the user asks to perform a Trivy container image security lab or analyse container vulnerabilities.
---

# Trivy Container Image Security Scan

Use this skill for educational container vulnerability scanning.

## Default training image

Use:

python:3.4-alpine

unless the user specifies another public training image.

Never run the vulnerable container.

## Workflow

Follow these steps.

### 1. Check Docker

Verify Docker is available.

Run:

docker --version

If Docker is unavailable, explain the problem and stop.

### 2. Download the image

Run:

docker pull python:3.4-alpine

unless the user supplied another image.

Do not run the container.

### 3. Verify the image

Run:

docker images python

Confirm that the image has been downloaded.

### 4. Scan with Trivy MCP

Use the available Trivy MCP tools to scan the downloaded container image.

Focus first on:

- CRITICAL
- HIGH

Also summarise:

- MEDIUM
- LOW

Do not invent vulnerabilities.

Use only vulnerabilities actually returned by Trivy.

### 5. Present the findings

Show a summary containing:

- Severity
- CVE ID
- Package
- Installed version
- Fixed version
- Whether a fix is available

Prioritise CRITICAL findings first, then HIGH.

If there are many vulnerabilities, show the most important findings first and provide severity totals.

### 6. Explain the risk

For important vulnerabilities explain:

- what component is vulnerable
- why the issue matters
- likely security impact
- whether a patched version exists

Keep explanations understandable to a beginner.

Do not provide exploitation instructions.

### 7. Recommend remediation

Recommend appropriate remediation such as:

- upgrade vulnerable package
- move to a supported base image
- use a newer container image
- rebuild the Docker image
- remove unnecessary packages
- use a smaller runtime image

Never claim a vulnerability is fixed unless the scan provides evidence of a fixed version or a rescan confirms it.

### 8. Prioritise actions

Classify recommendations as:

P1 - Immediate

P2 - High priority

P3 - Planned improvement

### 9. Rescan

After remediation, recommend another Trivy scan.

Compare:

Before remediation

versus

After remediation

when both scan results are available.

## Required final report

Produce:

1. Executive Summary
2. Vulnerability Severity Summary
3. Top Vulnerabilities
4. Risk Explanation
5. Remediation
6. Recommended Base Image Strategy
7. Priority Actions
8. Rescan Recommendation
```

---

# Part J — What the Skill Does

The skill creates this repeatable workflow:

```text
User
 │
 │ "Perform Trivy container scan"
 ▼
GitHub Copilot
 │
 ▼
Find relevant Skill
 │
 ▼
SKILL.md
 │
 ├── Check Docker
 │
 ├── docker pull
 │
 ├── Verify image
 │
 ├── Use Trivy MCP
 │
 ├── Analyse CVEs
 │
 ├── Prioritise
 │
 ├── Recommend remediation
 │
 └── Recommend rescan
```

GitHub explains that when a skill is relevant, Copilot loads its `SKILL.md` instructions into the agent's context and can use other files or scripts stored alongside it. ([GitHub Docs][1])

---

# Part K — Check Whether VS Code Found Your Skill

Press:

```text
Ctrl + Shift + P
```

Search:

```text
Chat: Configure Skills
```

Look for:

```text
trivy-image-scan
```

VS Code provides this command for viewing Agent Skills available to Copilot. ([Visual Studio Code][9])

---

# Part L — Execute the Skill

Open Copilot Chat.

Select:

```text
Agent
```

Prompt:

```text
Perform the Trivy container image security lab.

Use the trivy-image-scan skill.

Download the default vulnerable training image.

Scan it using Trivy MCP.

Show me the vulnerabilities and explain the results.

Do not run the vulnerable container.
```

---

# Expected Workflow

Copilot should perform something similar to:

```text
Step 1

Check Docker
     ↓

docker --version


Step 2

Download image
     ↓

docker pull python:3.4-alpine


Step 3

Verify image
     ↓

docker images python


Step 4

Copilot
     ↓

Trivy MCP
     ↓

Trivy Image Scan


Step 5

Receive CVE Results


Step 6

Copilot interprets findings
```

You may need to approve terminal or MCP tool calls depending on your VS Code security settings.

---

# Part M — Example of a Raw Trivy Finding

A live result may conceptually resemble:

```text
CVE-20xx-xxxxx

Package:
example-package

Installed Version:
1.2.3

Fixed Version:
1.2.7

Severity:
CRITICAL
```

Do not use this example as a real vulnerability.

Always use the actual data returned by Trivy.

---

# Part N — Ask Copilot to Interpret the Results

After the scan has completed, enter:

```text
Analyse the Trivy results as a DevSecOps Security Engineer.

For each important HIGH or CRITICAL vulnerability explain:

1. CVE ID
2. Vulnerable package
3. Installed version
4. Fixed version
5. Severity
6. What the vulnerability means in simple language
7. Possible security impact
8. Whether remediation is available
9. Recommended remediation
10. Remediation priority

Do not invent information that is not contained in the Trivy result.

Separate confirmed facts from your recommendations.
```

---

# Part O — Ask Copilot to Create an Executive Security Summary

Prompt:

```text
Using the Trivy MCP scan results, create a security executive summary.

Show:

Total vulnerabilities

CRITICAL count

HIGH count

MEDIUM count

LOW count

Then classify the container as:

Low Risk

Moderate Risk

High Risk

Critical Risk

Explain the reason for your classification.

Identify the three most important remediation actions.
```

Do not expect the same vulnerability counts every time; Trivy's data and upstream packages change over time.

---

# Part P — Ask Copilot to Prioritise Vulnerabilities

Prompt:

```text
Prioritise the Trivy findings.

Use:

P1 = immediate remediation

P2 = high priority

P3 = planned remediation

Prioritise based on:

Severity

Availability of a fix

Affected component

Potential application impact

Return a table containing:

Priority

CVE

Package

Severity

Fixed Version

Recommended Action
```

This is far more useful than simply seeing hundreds of CVE lines.

---

# Part Q — Ask Copilot for Remediation Recommendations

Prompt:

```text
Based only on the Trivy results:

Recommend how I should remediate this container.

Consider:

1. Upgrading the base image
2. Upgrading vulnerable OS packages
3. Upgrading application dependencies
4. Removing unnecessary packages
5. Using a supported image
6. Using a smaller runtime image
7. Rebuilding and rescanning the image

Tell me which actions should be performed first.

Do not automatically modify anything.
```

---

# Part R — Why Changing the Base Image Is Important

In this lab we deliberately use an extremely old Python image:

```text
python:3.4-alpine
```

It is useful for learning because students can compare an outdated software stack with a more modern supported base image.

The goal is not simply:

```text
Fix CVE-1
Fix CVE-2
Fix CVE-3
```

A better DevSecOps question is often:

```text
Why are we using such an old image
in the first place?
```

Copilot should therefore learn to recommend architecture-level remediation when appropriate, not just individual package upgrades.

---

# Part S — Scan a Newer Image

Now ask Copilot:

```text
Use Trivy MCP to scan:

python:3.12-alpine

Compare it against:

python:3.4-alpine

Compare:

CRITICAL

HIGH

MEDIUM

LOW

Then explain which image has the better security posture and why.

Do not assume that the newer image has zero vulnerabilities.
Use only the actual scan results.
```

Trivy MCP's official example queries include asking whether a Python container image has vulnerabilities, so this is a natural MCP-assisted workflow. ([GitHub][7])

---

# Part T — Recommended Comparison

Ask Copilot to format the result like:

```text
Security Comparison

                         OLD IMAGE       NEW IMAGE

Image                    Python 3.4      Python 3.12

CRITICAL                     xx              xx

HIGH                         xx              xx

MEDIUM                       xx              xx

LOW                          xx              xx

Supported stack?             Poor            Better

Recommendation               Replace          Prefer
```

The actual numbers must come from your live scans.

---

# Part U — Ask Copilot to Recommend a Dockerfile Change

If your existing application contains:

```dockerfile
FROM some-old-image
```

you can ask:

```text
Review my Dockerfile together with the Trivy scan.

Recommend the minimum Dockerfile changes necessary to reduce the vulnerabilities.

Do not make the changes yet.

Explain:

Current problem

Recommended image

Security benefit

Compatibility risk

How to validate the change
```

Then review Copilot's recommendation before changing your application.

---

# Part V — Remediation Workflow

The complete professional workflow should be:

```text
Container Image
      │
      ▼
Trivy Scan
      │
      ▼
Vulnerabilities
      │
      ▼
Copilot Analysis
      │
      ▼
Risk Prioritisation
      │
      ▼
Remediation Recommendation
      │
      ▼
Developer Review
      │
      ▼
Dockerfile / Package Update
      │
      ▼
Rebuild Image
      │
      ▼
Trivy Rescan
      │
      ▼
Compare Results
```

Never stop at:

```text
Scan completed.
```

A useful DevSecOps process must continue to:

```text
Understand
   ↓
Prioritise
   ↓
Remediate
   ↓
Validate
```

---

# Part W — Ask Copilot to Produce a Final Security Report

Use:

```text
Generate the final container security assessment based on the Trivy MCP results.

Structure it as:

# Container Security Assessment

## 1. Image Scanned

## 2. Executive Summary

## 3. Vulnerability Summary

## 4. Critical Findings

## 5. High Findings

## 6. Root Cause

## 7. Recommended Remediation

## 8. Recommended Base Image

## 9. Remediation Priority

## 10. Rescan Plan

For every CVE you mention, ensure it actually exists in the scan result.

Do not invent CVEs.

Clearly distinguish:

Confirmed Trivy Finding

from

AI Recommendation
```

This distinction is important:

```text
Trivy

provides security findings


Copilot

interprets and explains them
```

Copilot should not become the vulnerability database.

---

# Part X — Important Security Rule for This Lab

Because the training image is deliberately old:

```text
DO:

docker pull
```

and:

```text
DO:

trivy image scan
```

But:

```text
DO NOT:

docker run
```

the image for this exercise.

We only need the artifact for security analysis.

---

# Part Y — Troubleshooting

## Problem 1 — `trivy` Not Recognised

Run:

```powershell
trivy --version
```

If Windows responds:

```text
trivy is not recognized...
```

check that:

```text
C:\Tools\Trivy
```

is in your PATH.

Restart VS Code.

---

# Problem 2 — MCP Plugin Missing

Run:

```powershell
trivy plugin list
```

If MCP is missing:

```powershell
trivy plugin install mcp
```

Aqua documents this as the official Trivy MCP plugin installation command. ([GitHub][2])

---

# Problem 3 — Trivy MCP Does Not Start

Test:

```powershell
trivy mcp --help
```

Then in VS Code:

```text
Ctrl + Shift + P
```

Run:

```text
MCP: List Servers
```

Select:

```text
trivy
```

Then:

```text
Show Output
```

---

# Problem 4 — Copilot Does Not Use Trivy

Check:

```text
Copilot Chat
```

Make sure:

```text
Agent
```

is selected.

Then check:

```text
Configure Tools
```

and verify Trivy MCP tools are enabled.

Aqua's MCP guide specifically requires Agent mode for its natural-language security queries. ([GitHub][7])

---

# Problem 5 — Skill Is Not Detected

Check the directory:

```text
.github
└── skills
    └── trivy-image-scan
        └── SKILL.md
```

Then:

```text
Ctrl + Shift + P
```

run:

```text
Chat: Configure Skills
```

VS Code automatically looks for workspace Agent Skills in `.github/skills`. ([Visual Studio Code][9])

---

# Problem 6 — Docker Image Cannot Be Downloaded

Check:

```powershell
docker info
```

Then:

```powershell
docker pull python:3.4-alpine
```

If Docker cannot connect, confirm Docker Desktop is running.

---

# Part Z — Lab Success Criteria

Students have successfully completed the lab when they can demonstrate:

```text
✓ Trivy installed

✓ trivy --version works

✓ Trivy MCP plugin installed

✓ .vscode/mcp.json created

✓ Trivy MCP server running

✓ Copilot Agent mode enabled

✓ Trivy MCP tools visible

✓ SKILL.md detected

✓ Vulnerable training image downloaded

✓ Image NOT executed

✓ Image scanned through Trivy MCP

✓ Copilot displays vulnerability findings

✓ Copilot explains HIGH and CRITICAL findings

✓ Copilot recommends remediation

✓ Copilot prioritises remediation

✓ Student understands the difference between
  scanner evidence and AI recommendations
```

---

# Final Lab Architecture

```text
                     VS CODE
                        │
                        ▼
                GitHub Copilot
                   Agent Mode
                        │
               ┌────────┴─────────┐
               │                  │
               ▼                  ▼
           SKILL.md           MCP Tools
               │                  │
               └────────┬─────────┘
                        ▼
                 Trivy MCP Server
                        │
                        ▼
                      Trivy
                        │
                        ▼
              python:3.4-alpine
                        │
                        ▼
                Vulnerability Scan
                        │
              ┌─────────┼──────────┐
              ▼         ▼          ▼
          CRITICAL     HIGH       MEDIUM
              │         │          │
              └─────────┼──────────┘
                        ▼
                   Scan Results
                        │
                        ▼
                 GitHub Copilot
                        │
       ┌────────────────┼─────────────────┐
       ▼                ▼                 ▼
    Explain          Prioritise        Recommend
      CVE              Risk            Remediation
       │                │                 │
       └────────────────┼─────────────────┘
                        ▼
                 Developer Review
                        │
                        ▼
                    Remediate
                        │
                        ▼
                     Rebuild
                        │
                        ▼
                      Rescan
```

# Key Learning Point

The most important concept in this lab is:

```text
MCP gives Copilot access to the security tool.

SKILL.md teaches Copilot the repeatable workflow.

Trivy produces the security evidence.

GitHub Copilot helps humans understand and act on that evidence.
```

The security scanner should remain the source of truth for vulnerabilities; the AI's role is primarily orchestration, explanation, prioritisation and remediation assistance.

My recommendation for the class is to **show the direct `trivy image ...` command once first**, then repeat the same exercise through MCP and finally through `SKILL.md`. Students can then clearly see the progression from **manual security tool → AI-connected tool → reusable AI DevSecOps workflow**. ([Trivy][10])

[1]: https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills "Adding agent skills for GitHub Copilot - GitHub Docs"
[2]: https://github.com/aquasecurity/trivy-mcp "GitHub - aquasecurity/trivy-mcp: Trivy plugin for starting an MCP server · GitHub"
[3]: https://code.visualstudio.com/docs/agent-customization/mcp-servers "Add and manage MCP servers in VS Code"
[4]: https://trivy.dev/docs/latest/getting-started/installation/ "Installation - Trivy"
[5]: https://trivy.dev/docs/latest/getting-started/?utm_source=chatgpt.com "First steps"
[6]: https://github.com/aquasecurity/trivy-mcp/blob/main/docs/quickstart.md "trivy-mcp/docs/quickstart.md at main · aquasecurity/trivy-mcp · GitHub"
[7]: https://github.com/aquasecurity/trivy-mcp/blob/main/docs/example-queries.md "trivy-mcp/docs/example-queries.md at main · aquasecurity/trivy-mcp · GitHub"
[8]: https://code.visualstudio.com/docs/agent-customization/agent-skills?utm_source=chatgpt.com "Use Agent Skills in VS Code"
[9]: https://code.visualstudio.com/updates/v1_109?utm_source=chatgpt.com "January 2026 (version 1.109)"
[10]: https://trivy.dev/docs/latest/references/configuration/cli/trivy_image/?utm_source=chatgpt.com "Image"
