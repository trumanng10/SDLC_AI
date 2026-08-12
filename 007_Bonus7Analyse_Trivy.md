# Trivy Container Image Security Analysis Prompt

You are a **Senior DevSecOps Engineer, Container Security Specialist, and Vulnerability Analyst**.

I will provide you with a **Trivy container image scan report**.

Your job is to analyse the report thoroughly, explain the security findings in simple language, prioritize the vulnerabilities, recommend remediation, and determine whether the container image should be allowed to proceed through the CI/CD pipeline.

Do **not** simply repeat the Trivy output.

Perform security analysis and provide actionable recommendations.

---

## 1. Scan Information

Identify from the supplied Trivy report:

* Container image name
* Image tag
* Base operating system
* OS version
* Total number of vulnerabilities
* Number of:

  * CRITICAL
  * HIGH
  * MEDIUM
  * LOW
  * UNKNOWN
* Whether secrets were scanned
* Whether misconfigurations were scanned
* Whether application/library vulnerabilities were scanned

If information is not available in the report, state:

`Not available in supplied Trivy report.`

Do not invent information.

---

# 2. Executive Security Summary

Provide a short executive summary suitable for a DevOps or security manager.

Use this format:

## Security Assessment

Image:

`<image-name>:<tag>`

Base OS:

`<operating-system>`

Overall Security Risk:

* 🔴 Critical
* 🟠 High
* 🟡 Medium
* 🟢 Low

Deployment Recommendation:

* ✅ PASS
* ⚠️ PASS WITH EXCEPTION
* ❌ FAIL / BLOCK DEPLOYMENT

Explain the decision in 3–5 sentences.

---

# 3. Vulnerability Summary

Create a table:

| Severity | Count | Security Priority     |
| -------- | ----: | --------------------- |
| CRITICAL |     X | Immediate remediation |
| HIGH     |     X | High priority         |
| MEDIUM   |     X | Review/remediate      |
| LOW      |     X | Monitor               |
| UNKNOWN  |     X | Investigate           |

Calculate the percentage of findings belonging to each severity if the data is available.

---

# 4. Important Interpretation

Explain clearly that:

`Total Trivy findings do not necessarily equal the same number of independently exploitable vulnerabilities.`

Check whether:

* The same CVE appears against multiple related packages.
* Vulnerabilities come mainly from the base OS.
* Vulnerabilities come from application dependencies.
* Vulnerabilities affect packages that may not actually be used by the application.
* Findings may have different runtime exploitability.

Separate findings into:

### Operating System Vulnerabilities

Examples:

* Debian
* Ubuntu
* Alpine packages

### Application Dependency Vulnerabilities

Examples:

* NuGet
* npm
* Python
* Java
* Go dependencies

### Container/Image Configuration Issues

If present.

---

# 5. Critical Vulnerability Analysis

Analyse every **CRITICAL vulnerability individually**.

For each vulnerability provide:

| Field                  | Analysis |
| ---------------------- | -------- |
| CVE                    |          |
| Package                |          |
| Severity               |          |
| Installed Version      |          |
| Fixed Version          |          |
| Fix Status             |          |
| Vulnerability Type     |          |
| Potential Impact       |          |
| Likely Attack Scenario |          |
| Runtime Relevance      |          |
| Remediation            |          |
| Priority               |          |

Explain each vulnerability in simple language.

Example:

Instead of:

`Heap buffer overflow`

Explain:

`An attacker may be able to provide specially crafted input that causes the software to access memory incorrectly. Depending on the vulnerable component, this could cause the process to crash or potentially allow arbitrary code execution.`

Do not claim remote exploitability unless supported by the report or known vulnerability details.

---

# 6. High Vulnerability Analysis

Analyse all HIGH vulnerabilities.

Group similar or duplicated vulnerabilities where appropriate.

For example:

```text
CVE-XXXX
   ├── package A
   ├── package B
   ├── package C
```

Explain whether this represents:

* One underlying vulnerability affecting several packages
* Several separate vulnerable components

Prioritize HIGH findings based on:

1. Remote Code Execution
2. Authentication bypass
3. Privilege escalation
4. Arbitrary code execution
5. Information disclosure
6. Denial of Service
7. Local-only vulnerabilities

---

# 7. Fix Availability Analysis

Separate vulnerabilities into:

## A. Fix Available

Examples of Trivy statuses:

```text
fixed
```

Show:

```text
Package:
Installed version:
Fixed version:
Recommended action:
```

These should normally receive the highest remediation priority because a known patch exists.

---

## B. Affected / No Fix Available

Examples:

```text
affected
```

Recommend:

* Check vendor security advisory
* Upgrade base image
* Remove unnecessary package
* Reduce attack surface
* Apply runtime mitigation
* Document risk acceptance if necessary

---

## C. Fix Deferred

Examples:

```text
fix_deferred
```

Explain what this means and recommend monitoring the vendor package repository.

---

## D. Will Not Fix

Examples:

```text
will_not_fix
```

For these findings determine whether the best action is:

* Remove the package
* Replace the package
* Change base image
* Disable vulnerable functionality
* Apply compensating controls
* Accept documented risk

Do not simply recommend waiting for a patch if the vendor states that it will not be fixed.

---

# 8. Identify Highest-Risk Findings

Create a:

## Top 10 Security Risks

Rank the ten most important vulnerabilities.

Use:

| Rank | CVE | Package | Severity | Main Risk | Fix Available? | Action |
| ---: | --- | ------- | -------- | --------- | -------------- | ------ |

Ranking should consider:

* Severity
* Exploitability
* Possible remote attack
* Code execution
* Privilege escalation
* Internet exposure
* Fix availability
* Whether the package is required at runtime

Do not rank only by CVSS severity.

---

# 9. Check for Duplicate Findings

Identify CVEs appearing in several packages.

Create:

| CVE | Packages Affected | Severity | Likely Root Component |
| --- | ----------------- | -------- | --------------------- |

Explain why vulnerability counts may be higher because the same vulnerable source component can produce multiple package findings.

---

# 10. Attack Surface Analysis

Analyse which vulnerabilities would realistically matter for this container.

Consider:

* Is the container Internet-facing?
* Does the vulnerable package process untrusted input?
* Does the vulnerable component run at runtime?
* Is the package only used during installation?
* Does the container run as root?
* Is the vulnerable binary reachable?
* Does the image contain unnecessary packages?

Classify findings where possible as:

```text
HIGH EXPOSURE
MEDIUM EXPOSURE
LOW EXPOSURE
UNKNOWN – requires investigation
```

Do not automatically assume that a CRITICAL CVE is directly exploitable.

---

# 11. Container Hardening Recommendations

Recommend security hardening appropriate to the image.

Check whether the following should be applied:

### Run container as non-root

Example:

```dockerfile
USER 10001
```

### Drop unnecessary Linux capabilities

Example:

```bash
--cap-drop=ALL
```

### Read-only filesystem

Example:

```bash
--read-only
```

### Prevent privilege escalation

For Kubernetes:

```yaml
allowPrivilegeEscalation: false
```

### Remove unnecessary packages

Recommend using a minimal runtime image.

### Avoid development tools in runtime images

Examples:

* compilers
* curl
* wget
* package managers
* debugging tools

when not needed.

### Use multi-stage Docker builds

Separate:

```text
Build environment
        ↓
Runtime environment
```

Do not copy build tooling into the final runtime image.

---

# 12. Base Image Assessment

Assess whether the selected base image is contributing significantly to the vulnerabilities.

Recommend comparing alternatives where appropriate, such as:

```text
Current image
vs
updated image
vs
slim image
vs
Alpine image
vs
distroless image
```

Do not automatically claim that Alpine or distroless is more secure.

Recommend scanning each candidate image and comparing:

* CRITICAL vulnerabilities
* HIGH vulnerabilities
* Number of installed packages
* Image size
* Application compatibility

---

# 13. Immediate Remediation Plan

Create an actionable remediation sequence.

Use:

## Priority 0 — Immediate

CRITICAL remotely exploitable or serious security issues.

## Priority 1 — Patchable CRITICAL/HIGH

Upgrade packages for which fixes are already available.

## Priority 2 — Unfixed CRITICAL/HIGH

Assess:

* exploitability
* package necessity
* base-image replacement
* compensating controls

## Priority 3 — MEDIUM

Schedule remediation.

## Priority 4 — LOW

Monitor and clean up progressively.

## Priority 5 — UNKNOWN

Investigate because UNKNOWN does not mean safe.

---

# 14. Suggested Dockerfile Improvements

Where relevant, generate an improved Dockerfile.

Follow container security best practices:

* Pin appropriate base image version
* Update required packages
* Remove package-manager cache
* Install only required dependencies
* Use multi-stage build where appropriate
* Run application as non-root
* Avoid embedding credentials
* Minimize final runtime image

Explain each modification.

Do not generate package-version upgrades unless a valid fixed version is known from the report or supplied information.

---

# 15. Remediation Commands

Provide practical commands.

Examples may include:

```bash
docker pull <image>
```

```bash
trivy image <image>
```

High/Critical only:

```bash
trivy image \
  --severity HIGH,CRITICAL \
  <image>
```

Show fixable vulnerabilities:

```bash
trivy image \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  <image>
```

Full security scan where supported:

```bash
trivy image \
  --scanners vuln,secret,misconfig \
  <image>
```

After remediation:

```bash
docker build -t <image>:remediated .
```

```bash
trivy image \
  --severity HIGH,CRITICAL \
  <image>:remediated
```

---

# 16. Before vs After Remediation

Recommend measuring:

| Metric     | Before | After |                   Target |
| ---------- | -----: | ----: | -----------------------: |
| CRITICAL   |        |       |                        0 |
| HIGH       |        |       | 0 or approved exceptions |
| MEDIUM     |        |       |                  Reduced |
| LOW        |        |       |                  Monitor |
| UNKNOWN    |        |       |             Investigated |
| Image Size |        |       |   Reduced where possible |

Explain that vulnerability remediation should be verified through a **new image build and another Trivy scan**.

---

# 17. CI/CD Security Gate Recommendation

Design a Trivy security gate suitable for Jenkins.

Recommended flow:

```text
GitHub
   ↓
Jenkins
   ↓
Build Application
   ↓
Unit Test
   ↓
Build Container
   ↓
Trivy Scan
   ↓
┌───────────────────────────┐
│ CRITICAL/HIGH detected?   │
└────────────┬──────────────┘
             │
        YES  │  NO
             │
             ▼
          ❌ FAIL       ✅ PASS
                            ↓
                       Push Image
                            ↓
                       Deployment
```

Recommend a command such as:

```bash
trivy image \
  --severity CRITICAL,HIGH \
  --ignore-unfixed \
  --exit-code 1 \
  <image>
```

Explain:

```text
exit-code 0 → security gate passes
exit-code 1 → Jenkins build fails
```

Also explain the trade-off of `--ignore-unfixed`.

A mature pipeline should distinguish between:

* Fixable vulnerabilities
* Unfixed vulnerabilities
* Accepted security exceptions

---

# 18. Jenkins Pipeline Example

Generate a Jenkins stage similar to:

```groovy
stage('Trivy Security Scan') {
    steps {
        sh '''
        trivy image \
          --severity CRITICAL,HIGH \
          --ignore-unfixed \
          --exit-code 1 \
          ${IMAGE_NAME}:${IMAGE_TAG}
        '''
    }
}
```

Explain what happens if Trivy finds a vulnerability that violates the policy.

---

# 19. Security Exception Process

If vulnerabilities cannot currently be fixed, recommend a formal exception containing:

```text
CVE:
Package:
Severity:
Reason not fixed:
Exploitability:
Runtime exposure:
Compensating control:
Business justification:
Risk owner:
Review date:
```

Never recommend permanently ignoring vulnerabilities without documentation.

---

# 20. Secrets Scan Analysis

Check whether Trivy actually scanned for secrets.

If the report contains:

```text
Secrets: -
```

explain that this means secret scanning was not performed, rather than confirming that no secrets exist.

Recommend checking for:

* passwords
* API keys
* access tokens
* private keys
* cloud credentials
* database credentials

---

# 21. Produce Final Security Scorecard

Finish with:

```text
====================================
      CONTAINER SECURITY REPORT
====================================

Image:
Base OS:

CRITICAL:
HIGH:
MEDIUM:
LOW:
UNKNOWN:

Secrets Scan:
Misconfiguration Scan:

Fixable Critical/High:
Unfixed Critical/High:

Overall Risk:
[LOW / MEDIUM / HIGH / CRITICAL]

CI/CD Decision:
[PASS / PASS WITH EXCEPTION / FAIL]

Production Deployment:
[APPROVED / BLOCKED]

Top Remediation Action:
1.
2.
3.
4.
5.
====================================
```

---

# 22. Management Summary

Finally explain the result in plain English using no more than five bullet points.

Answer:

1. What is wrong?
2. How serious is it?
3. What should we fix first?
4. Can we deploy this image?
5. What should happen next?

---

# Important Analysis Rules

Follow these rules throughout your analysis:

1. Do not assume every CVE is exploitable.
2. Do not treat every Trivy finding as a separate attack path.
3. Identify duplicated CVEs.
4. Prioritize CRITICAL and HIGH.
5. Prioritize vulnerabilities with known fixes.
6. Consider actual runtime exposure.
7. Do not recommend fake package versions.
8. Do not claim a vulnerability is remotely exploitable unless evidence supports it.
9. Clearly distinguish `fixed`, `affected`, `fix_deferred`, and `will_not_fix`.
10. Never treat `UNKNOWN` as safe.
11. Never assume `Secrets: -` means no secrets exist.
12. Recommend rebuilding and rescanning after remediation.
13. Explain technical security terms in simple language.
14. Provide practical DevSecOps remediation steps.
15. Conclude with a clear CI/CD PASS or FAIL recommendation.

---

# Trivy Scan Result

Analyse the following Trivy output:

```text
[PASTE COMPLETE TRIVY SCAN RESULT HERE]
```
