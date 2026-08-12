Truman, the simplest reliable setup is:

**Developer → GitHub → GitHub Webhook → Jenkins → Jenkinsfile → Build/Test**

Jenkins recommends keeping the pipeline definition as a `Jenkinsfile` inside source control rather than typing the pipeline directly into Jenkins. ([Jenkins][1])

### 1. First, install the required Jenkins plugins

Open your Jenkins:

`http://159.223.103.209:8080/`

Then go to:

**Manage Jenkins → Plugins → Available plugins**

Search for and install:

* **Pipeline**
* **Git**
* **GitHub**

The Git plugin provides Git checkout/fetch support, while the GitHub plugin provides GitHub integration and webhook-triggered builds. ([Jenkins Plugins][2])

Restart Jenkins if Jenkins asks you to.

---

## 2. Prepare a simple GitHub repository

For example:

```text
https://github.com/YOUR-USERNAME/jenkins-lab.git
```

Your repository should eventually look like:

```text
jenkins-lab/
│
├── README.md
├── Jenkinsfile
└── src/
```

For the first lab, you don't even need a real application.

---

## 3. Create the `Jenkinsfile`

In the **root folder** of your GitHub repository, create a file named exactly:

```text
Jenkinsfile
```

Do **not** call it:

```text
Jenkinsfile.txt
jenkinsfile
JenkinsFile
```

Use this simple pipeline:

```groovy
pipeline {
    agent any

    stages {

        stage('Verify Source Code') {
            steps {
                echo 'Checking source code...'
                sh '''
                    echo "Current directory:"
                    pwd

                    echo "Files:"
                    ls -la

                    echo "Git commit:"
                    git log -1 --oneline
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building application...'
                sh '''
                    echo "Simple Build Successful"
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh '''
                    echo "Simple Test Successful"
                '''
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging application...'
                sh '''
                    echo "Simple Packaging Successful"
                '''
            }
        }
    }

    post {
        success {
            echo 'PIPELINE SUCCESSFUL!'
        }

        failure {
            echo 'PIPELINE FAILED!'
        }
    }
}
```

This deliberately doesn't require .NET, Java, Node.js, or Docker yet. Its purpose is to prove:

```text
GitHub
   ↓
Jenkins
   ↓
Checkout
   ↓
Build
   ↓
Test
   ↓
Package
```

A Declarative Jenkins pipeline automatically makes the checked-out source available on its agent, and Jenkins recommends storing this definition in the repository. ([Jenkins][1])

---

## 4. Commit the Jenkinsfile to GitHub

From VS Code:

```bash
git add Jenkinsfile
git commit -m "Add Jenkins CI pipeline"
git push origin main
```

Then check GitHub and make sure you can see:

```text
Jenkinsfile
```

in the repository root.

---

# 5. Create the Jenkins Pipeline Job

Go back to:

`http://159.223.103.209:8080/`

Click:

**New Item**

Enter:

```text
GitHub-Simple-Pipeline
```

Select:

**Pipeline**

Click:

**OK**

---

# 6. Connect Jenkins to GitHub

Scroll down to the **Pipeline** section.

For:

**Definition**

select:

```text
Pipeline script from SCM
```

Then:

**SCM**

select:

```text
Git
```

Enter your repository URL:

```text
https://github.com/YOUR-USERNAME/jenkins-lab.git
```

For example:

```text
https://github.com/blackjackiv2008/jenkins-lab.git
```

---

# 7. If your GitHub repository is PUBLIC

This is easy.

Under:

**Credentials**

select:

```text
- none -
```

Then use:

```text
Branch Specifier:
*/main
```

and:

```text
Script Path:
Jenkinsfile
```

Your configuration becomes roughly:

```text
Definition:
    Pipeline script from SCM

SCM:
    Git

Repository URL:
    https://github.com/YOUR-USERNAME/jenkins-lab.git

Credentials:
    - none -

Branch:
    */main

Script Path:
    Jenkinsfile
```

Click:

**Save**

---

# 8. Run the pipeline manually first

Before configuring the webhook, make sure Jenkins can actually read the GitHub repository.

Open your job:

```text
GitHub-Simple-Pipeline
```

Click:

**Build Now**

You should see:

```text
#1
```

appear under **Build History**.

Click:

```text
#1
```

Then:

**Console Output**

You should eventually see something similar to:

```text
Checking source code...

Current directory:
/var/jenkins_home/workspace/GitHub-Simple-Pipeline

Files:
...
Jenkinsfile
README.md

Git commit:
abc1234 Add Jenkins CI pipeline

Building application...
Simple Build Successful

Running tests...
Simple Test Successful

Packaging application...
Simple Packaging Successful

PIPELINE SUCCESSFUL!

Finished: SUCCESS
```

If you see:

```text
Finished: SUCCESS
```

your basic GitHub → Jenkins integration is working.

---

# 9. Configure automatic build on GitHub push

Now we want:

```text
git push
       ↓
GitHub
       ↓
Webhook
       ↓
Jenkins
       ↓
Pipeline automatically starts
```

The Jenkins GitHub plugin provides the **GitHub hook trigger for GITScm polling** option specifically for this. ([Jenkins Plugins][3])

Go to:

**Jenkins → GitHub-Simple-Pipeline → Configure**

Find:

**Build Triggers**

Enable:

```text
☑ GitHub hook trigger for GITScm polling
```

Click:

**Save**

---

# 10. Create the GitHub Webhook

Open your GitHub repository.

Go to:

**Settings → Webhooks → Add webhook**

GitHub's current webhook setup follows exactly this repository Settings → Webhooks → Add webhook flow. ([GitHub Docs][4])

For **Payload URL**, enter:

```text
http://159.223.103.209:8080/github-webhook/
```

### Important

The final `/` is intentional.

Use:

```text
/github-webhook/
```

not:

```text
/github-webhook
```

The Jenkins GitHub plugin documents this endpoint format as:

```text
$JENKINS_BASE_URL/github-webhook/
```

([Jenkins Plugins][3])

---

## 11. Configure the webhook

Set:

```text
Payload URL:
http://159.223.103.209:8080/github-webhook/

Content type:
application/json
```

For events choose:

```text
Just the push event
```

Enable:

```text
☑ Active
```

Then click:

**Add webhook**

GitHub sends a `ping` event immediately after a webhook is created, which lets you check whether the endpoint is reachable. ([GitHub Docs][4])

You ideally want to see:

```text
✓
```

or a successful delivery under GitHub's webhook **Recent Deliveries**.

---

# 12. Test the complete automatic pipeline

Change your README:

```markdown
# Jenkins Lab

Testing Jenkins CI/CD Pipeline
```

Then run:

```bash
git add .
git commit -m "Test Jenkins webhook"
git push origin main
```

Now don't click **Build Now**.

GitHub should automatically call:

```text
http://159.223.103.209:8080/github-webhook/
```

Jenkins should automatically create:

```text
Build #2
```

And the flow becomes:

```text
                git push
                    │
                    ▼
              ┌──────────┐
              │  GitHub  │
              └────┬─────┘
                   │
                   │ Webhook
                   ▼
       159.223.103.209:8080
              ┌──────────┐
              │ Jenkins  │
              └────┬─────┘
                   │
                   ▼
              Jenkinsfile
                   │
          ┌────────┴─────────┐
          ▼                  ▼
       BUILD               TEST
          │                  │
          └────────┬─────────┘
                   ▼
                PACKAGE
                   │
                   ▼
                SUCCESS
```

---

# 13. If the GitHub repository is PRIVATE

Do **not** put your GitHub password in Jenkins. GitHub removed password authentication for Git operations; HTTPS authentication should use a token instead. ([GitHub Docs][5])

In Jenkins go to:

**Manage Jenkins → Credentials**

Then:

**System → Global credentials → Add Credentials**

Select:

```text
Kind:
Username with password
```

Enter:

```text
Username:
YOUR-GITHUB-USERNAME

Password:
YOUR-GITHUB-PERSONAL-ACCESS-TOKEN

ID:
github-credentials

Description:
GitHub Repository Access
```

Jenkins supports storing username/password, secret text such as tokens, SSH keys and other credentials in its credential store. ([Jenkins][6])

Then return to:

**GitHub-Simple-Pipeline → Configure**

Under Git SCM choose:

```text
Credentials:
github-credentials
```

Do not put a GitHub token directly inside:

```groovy
Jenkinsfile
```

and do not put it inside:

```text
https://TOKEN@github.com/...
```

---

# 14. One security issue with your current Jenkins URL

Your Jenkins server is currently being accessed through:

```text
http://159.223.103.209:8080
```

That means it is a public IP using **HTTP rather than HTTPS**.

For a training lab, you can use this configuration temporarily, but I would **not use this configuration for production or enter valuable production credentials through it**.

My preferred production design would be:

```text
Internet
    │
    ▼
https://jenkins.yourdomain.com
    │
    ▼
Nginx / Caddy
TLS / HTTPS
    │
    ▼
Jenkins
:8080
```

Jenkins officially documents running behind reverse proxies such as Nginx, Apache and Caddy, with the proxy providing HTTPS/SSL in front of Jenkins. ([Jenkins][7])

Also, you generally don't want Jenkins administration on port `8080` freely exposed to the entire Internet.

---

# 15. Your completed Lab 1 architecture

After these steps you will have:

```text
┌─────────────────────────────┐
│        Developer PC         │
│                             │
│ VS Code                     │
│ git add                     │
│ git commit                  │
│ git push                    │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│           GitHub            │
│                             │
│ Source Code                 │
│ Jenkinsfile                 │
└──────────────┬──────────────┘
               │
               │ Webhook
               ▼
┌─────────────────────────────┐
│      Jenkins Server         │
│   159.223.103.209:8080      │
│                             │
│  1. Checkout Source         │
│  2. Build                   │
│  3. Test                    │
│  4. Package                 │
└──────────────┬──────────────┘
               │
               ▼
          SUCCESS / FAIL
```

## What I recommend you do next

Once **this simple pipeline works**, don't jump directly into Kubernetes. Build it progressively:

```text
LAB 1
GitHub
   ↓
Jenkins
   ↓
Simple Build
        ✅

LAB 2
GitHub
   ↓
Jenkins
   ↓
dotnet restore
   ↓
dotnet build
   ↓
dotnet test

LAB 3
GitHub
   ↓
Jenkins
   ↓
.NET Build
   ↓
Docker Build
   ↓
Trivy Scan

LAB 4
GitHub
   ↓
Jenkins
   ↓
.NET Build/Test
   ↓
Docker Build
   ↓
Trivy Scan
   ↓
Docker Hub

LAB 5
GitHub
   ↓
Jenkins
   ↓
Build
   ↓
Test
   ↓
Security Scan
   ↓
Docker Hub
   ↓
Kubernetes Deployment
```

**I recommend getting Lab 1 completely green first.** It isolates GitHub/webhook/Jenkins problems before we introduce .NET, Docker credentials, Trivy and Kubernetes.

[1]: https://www.jenkins.io/doc/book/pipeline/jenkinsfile/ "Using a Jenkinsfile"
[2]: https://plugins.jenkins.io/git/?utm_source=chatgpt.com "Git | Jenkins plugin"
[3]: https://plugins.jenkins.io/github/ "GitHub | Jenkins plugin"
[4]: https://docs.github.com/en/webhooks/using-webhooks/creating-webhooks "Creating webhooks - GitHub Docs"
[5]: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens?utm_source=chatgpt.com "Managing your personal access tokens"
[6]: https://www.jenkins.io/doc/book/using/using-credentials/?utm_source=chatgpt.com "Using credentials"
[7]: https://www.jenkins.io/doc/book/system-administration/reverse-proxy-configuration-with-jenkins/?utm_source=chatgpt.com "Reverse proxy configuration"
