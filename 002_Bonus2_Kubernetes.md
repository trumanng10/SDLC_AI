
# Bonus 02 — Deploy Lab02API to Kubernetes Using VS Code and GitHub Copilot

## 1. Lab Objective

In Bonus Lab 1, we completed:

```text
Lab02API
   ↓
Dockerfile
   ↓
Docker Image
   ↓
Docker Hub
```

In this lab, we continue from Docker Hub and deploy the application into Kubernetes.

We will learn how to:

1. Enable Kubernetes in Docker Desktop.
2. Verify Kubernetes using `kubectl`.
3. Configure Kubernetes support in VS Code.
4. Use GitHub Copilot to create Kubernetes YAML files.
5. Create a Kubernetes Deployment.
6. Create multiple Lab02API Pods.
7. Create a Kubernetes Service.
8. Access the Web API through the Service.
9. Scale the application up.
10. Scale the application down.
11. Monitor Pods.
12. Perform a rolling application update.
13. Roll back an unsuccessful update.
14. Clean up the Kubernetes resources.

---

# 2. Architecture of This Lab

The completed architecture will look like this:

```text
                          Docker Hub
                              │
                              │ pull image
                              ▼
                    Kubernetes Deployment
                              │
                       desired replicas
                              │
               ┌──────────────┼──────────────┐
               ▼              ▼              ▼
          ┌─────────┐    ┌─────────┐    ┌─────────┐
          │ Pod #1  │    │ Pod #2  │    │ Pod #3  │
          │Lab02API │    │Lab02API │    │Lab02API │
          │  :8080  │    │  :8080  │    │  :8080  │
          └────┬────┘    └────┬────┘    └────┬────┘
               │              │              │
               └──────────────┼──────────────┘
                              │
                              ▼
                      Kubernetes Service
                       lab02api-service
                              │
                              ▼
                           Client
                              │
                              ▼
                    http://localhost:8080
```

The **Deployment** manages Pods and their desired state. Kubernetes Deployments support operations including scaling, controlled rollouts and rollback. ([Kubernetes][2])

The **Service** provides a stable endpoint for a group of Pods, so clients do not need to know which individual Pod is handling the request. ([Kubernetes][3])

---

# Part A — Prerequisites

Before starting this lab, Lab 02 must already be working.

You should have:

```text
✓ VS Code
✓ GitHub Copilot
✓ .NET 10
✓ Docker Desktop
✓ Docker Hub account
✓ Lab02API Docker image
```

Your Docker Hub image should resemble:

```text
YOUR_DOCKERHUB_USERNAME/lab02api:1.0
```

For example:

```text
trumanng/lab02api:1.0
```

Replace `YOUR_DOCKERHUB_USERNAME` throughout this lab with your actual Docker Hub username.

For this beginner lab, use a **public Docker Hub repository**.

---

# Part B — Enable Kubernetes

## Step 1 — Start Docker Desktop

Start:

```text
Docker Desktop
```

Make sure Docker itself is running.

From VS Code Terminal or PowerShell:

```powershell
docker version
```

---

# Step 2 — Enable Kubernetes in Docker Desktop

Current Docker Desktop versions provide Kubernetes management directly from the Kubernetes view. Docker Desktop can create either a `kubeadm` single-node cluster or a `kind` cluster. ([Docker Documentation][1])

For this beginner exercise, use:

```text
kubeadm
```

The single-node cluster is enough to learn:

```text
Deployment
Pods
Service
Scaling
Rolling Updates
```

Open:

```text
Docker Desktop
   ↓
Kubernetes
   ↓
Create cluster
```

Choose:

```text
kubeadm
```

Then select:

```text
Create
```

When Kubernetes is enabled, Docker Desktop installs the `kubectl` command-line tool on Windows as part of the setup. ([Docker Documentation][1])

---

# Step 3 — Verify Kubernetes

Open the VS Code terminal:

```text
Terminal
→ New Terminal
```

Run:

```powershell
kubectl version --client
```

Then:

```powershell
kubectl get nodes
```

Expected output should resemble:

```text
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   ...   ...
```

Docker documents `kubectl get nodes` as a way to confirm that the local Docker Desktop Kubernetes cluster is running. ([Docker Documentation][1])

---

# Step 4 — Check Your Kubernetes Context

Run:

```powershell
kubectl config current-context
```

Expected:

```text
docker-desktop
```

If it shows something else, run:

```powershell
kubectl config use-context docker-desktop
```

Then check again:

```powershell
kubectl config current-context
```

Docker documents `docker-desktop` as the context to select when `kubectl` is pointing to another Kubernetes environment. ([Docker Documentation][1])

---

# Part C — Configure VS Code for Kubernetes

## Step 5 — Install the Kubernetes Extension

In VS Code press:

```text
Ctrl + Shift + X
```

Search:

```text
Kubernetes
```

Install:

```text
Kubernetes
Publisher: Microsoft
```

The Microsoft Kubernetes extension supports Kubernetes manifest authoring and lets you browse and manage Kubernetes clusters from VS Code. ([Visual Studio Code][4])

After installation, look for:

```text
KUBERNETES
```

in the VS Code interface.

You should eventually be able to see resources such as:

```text
Clusters
Namespaces
Nodes
Deployments
Pods
Services
```

---

# Part D — Prepare Lab02API

Your Lab02API directory currently resembles:

```text
Lab02API
│
├── Dockerfile
├── .dockerignore
├── Program.cs
├── Lab02API.csproj
├── appsettings.json
└── Properties
```

We are going to add:

```text
k8s
```

so the final structure becomes:

```text
Lab02API
│
├── Dockerfile
├── .dockerignore
├── Program.cs
├── Lab02API.csproj
│
└── k8s
    │
    ├── deployment.yaml
    └── service.yaml
```

---

# Step 6 — Create the Kubernetes Folder

From the VS Code terminal:

```powershell
mkdir k8s
```

---

# Part E — Use GitHub Copilot to Create Kubernetes Configuration

## Step 7 — Give GitHub Copilot This Prompt

Open GitHub Copilot Chat in VS Code.

Paste:

```text
You are an expert DevOps and Kubernetes Engineer.

I am a beginner learning Kubernetes.

My current project is Lab02API, an ASP.NET Core .NET 10 Web API.

The application has already been containerised and uploaded to Docker Hub.

Docker image:

YOUR_DOCKERHUB_USERNAME/lab02api:1.0

The ASP.NET Core application listens on container port 8080.

Create Kubernetes configuration files inside a folder named k8s.

Create:

1. k8s/deployment.yaml
2. k8s/service.yaml

Deployment requirements:

- Kubernetes apps/v1 Deployment
- Deployment name: lab02api-deployment
- Application label: app: lab02api
- Start with 2 replicas
- Docker image:
  YOUR_DOCKERHUB_USERNAME/lab02api:1.0
- Container name: lab02api
- Container port: 8080
- Use imagePullPolicy: IfNotPresent
- Do not create a database
- Do not create Ingress
- Do not create Helm charts

Service requirements:

- Service name: lab02api-service
- Select Pods with app: lab02api
- Service type: ClusterIP
- Service port: 80
- Forward traffic to container port 8080

After creating the files:

1. Explain every Kubernetes section in beginner-friendly language.
2. Explain Deployment, Pod, Replica and Service.
3. Give me commands to deploy everything.
4. Give me commands to check the Pods and Service.
5. Give me commands to scale the application from 2 Pods to 5 Pods.
6. Give me commands to scale it back to 1 Pod.
7. Show me how to access the Service locally using kubectl port-forward.
8. Do not make unnecessary changes to Program.cs.
```

Review Copilot's files before accepting the changes.

---

# Part F — Create deployment.yaml

## Step 8 — Check deployment.yaml

The file should resemble:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: lab02api-deployment

spec:
  replicas: 2

  selector:
    matchLabels:
      app: lab02api

  template:
    metadata:
      labels:
        app: lab02api

    spec:
      containers:
        - name: lab02api

          image: YOUR_DOCKERHUB_USERNAME/lab02api:1.0

          imagePullPolicy: IfNotPresent

          ports:
            - containerPort: 8080
```

Remember to replace:

```text
YOUR_DOCKERHUB_USERNAME
```

with your Docker Hub username.

---

# Step 9 — Understand the Deployment YAML

## `apiVersion`

```yaml
apiVersion: apps/v1
```

Tells Kubernetes which API definition is being used.

---

## `kind`

```yaml
kind: Deployment
```

We are creating a:

```text
Kubernetes Deployment
```

A Deployment manages application Pods and ReplicaSets. ([Kubernetes][2])

---

## Deployment Name

```yaml
metadata:
  name: lab02api-deployment
```

The Deployment is called:

```text
lab02api-deployment
```

---

## Number of Replicas

```yaml
replicas: 2
```

This means:

```text
Kubernetes must maintain
        2
Lab02API Pods
```

Conceptually:

```text
Deployment
    │
    ├──── Pod 1
    │
    └──── Pod 2
```

---

## Pod Labels

```yaml
labels:
  app: lab02api
```

Each Lab02API Pod receives the label:

```text
app=lab02api
```

The Service will use this label to locate the application Pods.

---

## Docker Image

```yaml
image: YOUR_DOCKERHUB_USERNAME/lab02api:1.0
```

Kubernetes retrieves your container image and starts it as Pods.

Therefore:

```text
Docker Hub
     │
     ▼
Docker Image
     │
     ▼
Kubernetes Pod
```

---

## Container Port

```yaml
containerPort: 8080
```

Our ASP.NET application listens inside the container on:

```text
8080
```

---

# Part G — Create the Kubernetes Service

## Step 10 — Check service.yaml

The file should resemble:

```yaml
apiVersion: v1

kind: Service

metadata:
  name: lab02api-service

spec:
  selector:
    app: lab02api

  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

  type: ClusterIP
```

---

# Step 11 — Understand the Service

The important configuration is:

```yaml
selector:
  app: lab02api
```

This tells the Service:

```text
Find Pods where:

app = lab02api
```

Our Deployment creates Pods containing exactly this label.

Therefore:

```text
Service
   │
   │ selector: app=lab02api
   │
   ├──────────────→ Pod 1
   │
   ├──────────────→ Pod 2
   │
   └──────────────→ Pod 3
```

Services exist specifically to provide network access to sets of Pods instead of requiring clients to work with individual Pod addresses. ([Kubernetes][3])

---

# Step 12 — Understand port and targetPort

Our Service has:

```yaml
port: 80
targetPort: 8080
```

This means:

```text
Service Port               Pod Container Port

     80          ───────→        8080
```

The application itself is still running on:

```text
8080
```

inside each container.

---

# Part H — Deploy Lab02API to Kubernetes

## Step 13 — Check Your YAML Files

Your directory should be:

```text
Lab02API
│
└── k8s
    ├── deployment.yaml
    └── service.yaml
```

---

# Step 14 — Deploy the Deployment

Run:

```powershell
kubectl apply -f k8s/deployment.yaml
```

Expected:

```text
deployment.apps/lab02api-deployment created
```

---

# Step 15 — Create the Service

Run:

```powershell
kubectl apply -f k8s/service.yaml
```

Expected:

```text
service/lab02api-service created
```

Alternatively, both files can be applied with:

```powershell
kubectl apply -f k8s/
```

---

# Part I — Check the Kubernetes Resources

## Step 16 — Check Deployment

Run:

```powershell
kubectl get deployments
```

You should see:

```text
NAME                  READY   UP-TO-DATE   AVAILABLE
lab02api-deployment   2/2     2            2
```

---

# Step 17 — Check the Pods

Run:

```powershell
kubectl get pods
```

Expected:

```text
NAME                                   READY   STATUS
lab02api-deployment-xxxxxxxxxx-abc12   1/1     Running
lab02api-deployment-xxxxxxxxxx-def34   1/1     Running
```

Notice there are:

```text
2 Pods
```

because:

```yaml
replicas: 2
```

---

# Step 18 — Look at the Pods With Labels

Run:

```powershell
kubectl get pods -l app=lab02api
```

This asks:

```text
Show Pods where:

app = lab02api
```

---

# Step 19 — Check the Service

Run:

```powershell
kubectl get services
```

or:

```powershell
kubectl get svc
```

Expected:

```text
NAME               TYPE        CLUSTER-IP       PORT(S)
lab02api-service   ClusterIP   xxx.xxx.xxx.xxx  80/TCP
```

A `ClusterIP` Service is designed to be reachable inside the cluster rather than directly from an external client. ([Kubernetes][3])

---

# Step 20 — See Everything Together

Run:

```powershell
kubectl get all
```

You should see resources resembling:

```text
pod/lab02api-deployment-xxxxx
pod/lab02api-deployment-yyyyy

service/lab02api-service

deployment.apps/lab02api-deployment

replicaset.apps/lab02api-deployment-xxxxx
```

The relationship is approximately:

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ├──── Pod
    └──── Pod


Service
    │
    └──── Finds Pods using labels
```

---

# Part J — Access the Web Service

## Step 21 — Use Port Forwarding

Because we deliberately created a simple internal `ClusterIP` Service, use `kubectl port-forward` to access it from your Windows PC.

Run:

```powershell
kubectl port-forward service/lab02api-service 8080:80
```

You should see:

```text
Forwarding from 127.0.0.1:8080 -> ...
```

Keep this terminal running.

The flow is now:

```text
Browser
   │
   │ localhost:8080
   ▼
kubectl port-forward
   │
   ▼
Kubernetes Service :80
   │
   ▼
Lab02API Pod :8080
```

---

# Step 22 — Test Lab02API

Open:

```text
http://localhost:8080
```

Expected:

```text
AI SDLC Lab 02 API
```

Then test:

```text
http://localhost:8080/discount?age=65
```

Expected:

```json
{
  "age": 65,
  "discount": 0.2
}
```

Now your Web API is running inside Kubernetes.

---

# Part K — Scale Up Lab02API

This is one of Kubernetes' most important capabilities.

Scaling means changing the number of application instances.

Kubernetes' `kubectl scale` command changes the number of replicas for a Deployment. ([Kubernetes][5])

Currently:

```text
replicas = 2
```

Architecture:

```text
Deployment
    │
    ├── Pod 1
    └── Pod 2
```

---

# Step 23 — Scale From 2 Pods to 5 Pods

Run:

```powershell
kubectl scale deployment lab02api-deployment --replicas=5
```

Check:

```powershell
kubectl get pods
```

You should eventually see:

```text
Pod 1   Running
Pod 2   Running
Pod 3   Running
Pod 4   Running
Pod 5   Running
```

The architecture becomes:

```text
                    Service
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
      Pod 1          Pod 2          Pod 3
        │
        ├──────────── Pod 4
        │
        └──────────── Pod 5
```

Scaling a Deployment causes Kubernetes to create additional Pods until the desired replica count is reached. ([Kubernetes][6])

---

# Step 24 — Watch Kubernetes Scale

Run:

```powershell
kubectl get pods -w
```

`-w` means:

```text
watch
```

You can watch Pods being:

```text
Pending
↓
ContainerCreating
↓
Running
```

Press:

```text
Ctrl + C
```

to stop watching.

---

# Part L — Scale Down Lab02API

## Step 25 — Scale From 5 Pods to 1 Pod

Run:

```powershell
kubectl scale deployment lab02api-deployment --replicas=1
```

Check:

```powershell
kubectl get pods
```

Kubernetes will terminate unnecessary Pods until only one remains.

Conceptually:

```text
Before

Pod 1
Pod 2
Pod 3
Pod 4
Pod 5


Scale Down


After

Pod 1
```

Kubernetes also allows a Deployment to be scaled to zero if required. ([Kubernetes][6])

For example:

```powershell
kubectl scale deployment lab02api-deployment --replicas=0
```

would stop all Lab02API Pods without deleting the Deployment itself.

For this exercise, keep:

```text
1
```

running.

---

# Part M — Very Important: Declarative vs Manual Scaling

Your YAML still contains:

```yaml
replicas: 2
```

But you manually changed Kubernetes to:

```text
replicas = 1
```

If you later apply:

```powershell
kubectl apply -f k8s/deployment.yaml
```

the YAML desired state can restore the Deployment to:

```text
2 replicas
```

This is an important Kubernetes concept.

The YAML file describes your:

```text
Desired State
```

Kubernetes attempts to make the actual system match that desired state. Deployments are specifically designed around this declarative desired-state model. ([Kubernetes][2])

---

# Part N — Scale Using the YAML File

Instead of using:

```powershell
kubectl scale
```

open:

```text
k8s/deployment.yaml
```

Change:

```yaml
replicas: 2
```

to:

```yaml
replicas: 3
```

Save.

Then run:

```powershell
kubectl apply -f k8s/deployment.yaml
```

Check:

```powershell
kubectl get pods
```

You should get:

```text
3 Pods
```

This method is preferable when you want the configuration file to remain the source of truth.

---

# Part O — Ask GitHub Copilot to Scale the Application

You can also ask Copilot:

```text
I have a Kubernetes Deployment called lab02api-deployment.

Explain how to safely scale the application from its current number of replicas to 5 replicas.

Give me:

1. The kubectl scale command.
2. The kubectl command to watch Pods being created.
3. The command to verify the Deployment.
4. The command to verify the Service.
5. The command to scale back to 2 replicas.

Do not delete or recreate the Deployment.
Explain each command in beginner-friendly language.
```

---

# Part P — Understand How the Service Helps Scaling

Imagine we have:

```text
Pod 1
Pod 2
Pod 3
Pod 4
Pod 5
```

Each Pod gets its own internal IP address.

The client should **not** have to know those addresses.

Instead:

```text
                  Client
                     │
                     ▼
             lab02api-service
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
       Pod 1       Pod 2       Pod 3
         ▼           ▼
       Pod 4       Pod 5
```

This is why Kubernetes Services are important when applications scale. The Service selects the relevant Pods and provides a stable access abstraction even as Pod instances change. ([Kubernetes][3])

---

# Part Q — Examine Kubernetes Using VS Code

With the Microsoft Kubernetes extension installed, find:

```text
KUBERNETES
```

in VS Code.

Expand your cluster.

Look for:

```text
Workloads
```

and:

```text
Services
```

You should be able to find resources corresponding to:

```text
lab02api-deployment

lab02api-service
```

The extension is designed to let developers browse and manage Kubernetes clusters directly from VS Code. ([Visual Studio Code][4])

---

# Part R — Inspect a Pod

Run:

```powershell
kubectl get pods
```

Copy one Pod name.

Example:

```text
lab02api-deployment-74b867cf75-abc12
```

Then:

```powershell
kubectl describe pod lab02api-deployment-74b867cf75-abc12
```

This shows details such as:

```text
Container
Image
Ports
Status
Events
Node
Pod IP
```

---

# Part S — View Application Logs

First:

```powershell
kubectl get pods
```

Then:

```powershell
kubectl logs POD_NAME
```

For example:

```powershell
kubectl logs lab02api-deployment-74b867cf75-abc12
```

You can also ask:

```powershell
kubectl logs -l app=lab02api
```

This is useful when the application does not start correctly.

---

# Part T — Ask Copilot to Troubleshoot Kubernetes

If a Pod fails, paste this prompt into Copilot:

```text
Act as a senior Kubernetes DevOps Engineer.

My Lab02API Kubernetes Deployment is not working.

The application is:

ASP.NET Core .NET 10

Docker image:
YOUR_DOCKERHUB_USERNAME/lab02api:1.0

Application container port:
8080

Review:

- k8s/deployment.yaml
- k8s/service.yaml
- Dockerfile
- Program.cs

Then ask me to run only the Kubernetes diagnostic commands necessary to identify the problem.

Check for:

- ImagePullBackOff
- CrashLoopBackOff
- incorrect Docker image
- incorrect container port
- incorrect Service targetPort
- incorrect Service selector
- Deployment label mismatch
- application startup errors
- Docker Hub image availability

Explain the root cause in beginner-friendly language.

Make only the minimum necessary changes.
```

---

# Part U — Useful Troubleshooting Commands

## Check Pods

```powershell
kubectl get pods
```

---

## Get More Details

```powershell
kubectl get pods -o wide
```

---

## Describe Deployment

```powershell
kubectl describe deployment lab02api-deployment
```

---

## Describe Service

```powershell
kubectl describe service lab02api-service
```

---

## Check Events

```powershell
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## Check Logs

```powershell
kubectl logs -l app=lab02api
```

---

# Part V — Typical Kubernetes Errors

## Error 1 — ImagePullBackOff

Example:

```text
ImagePullBackOff
```

Possible reason:

```text
Kubernetes cannot download your Docker image.
```

Check:

```yaml
image: YOUR_DOCKERHUB_USERNAME/lab02api:1.0
```

Make sure the repository and tag actually exist.

For this beginner lab, using a public Docker Hub repository avoids needing registry credentials.

---

# Error 2 — CrashLoopBackOff

Example:

```text
CrashLoopBackOff
```

This generally means the container starts and then repeatedly exits.

Run:

```powershell
kubectl logs POD_NAME
```

and:

```powershell
kubectl describe pod POD_NAME
```

Then give the output to GitHub Copilot for analysis.

---

# Error 3 — Service Cannot Reach Application

Check:

```powershell
kubectl describe service lab02api-service
```

Verify:

```yaml
selector:
  app: lab02api
```

matches the Pod label:

```yaml
labels:
  app: lab02api
```

Also check:

```yaml
targetPort: 8080
```

because Lab02API listens on port:

```text
8080
```

---

# Part W — Update Lab02API to Version 1.1

Now simulate a real DevOps update.

Open:

```text
Program.cs
```

Change:

```csharp
app.MapGet("/", () => "AI SDLC Lab 02 API");
```

to:

```csharp
app.MapGet("/", () => "AI SDLC Lab 02 API - Version 1.1");
```

Save.

---

# Step 26 — Build Docker Image Version 1.1

Run:

```powershell
docker build -t YOUR_DOCKERHUB_USERNAME/lab02api:1.1 .
```

For example:

```powershell
docker build -t trumanng/lab02api:1.1 .
```

---

# Step 27 — Upload Version 1.1

Run:

```powershell
docker push YOUR_DOCKERHUB_USERNAME/lab02api:1.1
```

---

# Step 28 — Update Kubernetes

Open:

```text
k8s/deployment.yaml
```

Change:

```yaml
image: YOUR_DOCKERHUB_USERNAME/lab02api:1.0
```

to:

```yaml
image: YOUR_DOCKERHUB_USERNAME/lab02api:1.1
```

Apply:

```powershell
kubectl apply -f k8s/deployment.yaml
```

---

# Step 29 — Watch the Rolling Update

Run:

```powershell
kubectl rollout status deployment/lab02api-deployment
```

Then:

```powershell
kubectl get pods
```

Kubernetes Deployments perform controlled replacement of old Pods when the Pod template, such as its container image, changes. ([Kubernetes][2])

Conceptually:

```text
Version 1.0 Pods
       │
       ▼
Start Version 1.1 Pods
       │
       ▼
Check new Pods
       │
       ▼
Remove old Version 1.0 Pods
       │
       ▼
Version 1.1 Running
```

---

# Step 30 — Test Version 1.1

Run port forwarding again if necessary:

```powershell
kubectl port-forward service/lab02api-service 8080:80
```

Open:

```text
http://localhost:8080
```

Expected:

```text
AI SDLC Lab 02 API - Version 1.1
```

---

# Part X — Roll Back an Update

Suppose Version 1.1 is broken.

Check rollout history:

```powershell
kubectl rollout history deployment/lab02api-deployment
```

Roll back:

```powershell
kubectl rollout undo deployment/lab02api-deployment
```

Then:

```powershell
kubectl rollout status deployment/lab02api-deployment
```

Deployments maintain rollout revisions when the Pod template changes, enabling rollback to an earlier revision. ([Kubernetes][2])

---

# Part Y — Optional Advanced Exercise: Automatic Scaling

So far we performed:

```text
Manual Scaling
```

using:

```powershell
kubectl scale
```

Kubernetes also supports:

```text
Horizontal Pod Autoscaler
HPA
```

Conceptually:

```text
Low Traffic
    │
    ▼
 2 Pods


Traffic / CPU increases
    │
    ▼
 HPA detects demand
    │
    ▼
 5 Pods


Demand decreases
    │
    ▼
 HPA scales down
    │
    ▼
 2 Pods
```

Kubernetes HPA can automatically adjust replicas based on metrics such as CPU or memory utilization. ([Kubernetes][7])

However, HPA requires working resource metrics, commonly provided through Metrics Server, and the Deployment should have appropriate resource requests configured. ([Kubernetes][8])

For a first Kubernetes class, I recommend learning:

```text
Manual scale up
Manual scale down
Deployment
Service
Rolling update
Rollback
```

before adding HPA.

---

# Part Z — Clean Up the Lab

When finished, remove the Kubernetes resources.

Run:

```powershell
kubectl delete -f k8s/
```

Check:

```powershell
kubectl get all
```

Lab02API resources should be gone.

The Kubernetes cluster itself remains available for future labs.

---

# Kubernetes Command Cheat Sheet

## Check Kubernetes

```powershell
kubectl get nodes
```

## Current Context

```powershell
kubectl config current-context
```

## Deploy Application

```powershell
kubectl apply -f k8s/
```

## Check Deployment

```powershell
kubectl get deployments
```

## Check Pods

```powershell
kubectl get pods
```

## Watch Pods

```powershell
kubectl get pods -w
```

## Check Service

```powershell
kubectl get svc
```

## View Everything

```powershell
kubectl get all
```

## Scale Up to 5

```powershell
kubectl scale deployment lab02api-deployment --replicas=5
```

## Scale Down to 2

```powershell
kubectl scale deployment lab02api-deployment --replicas=2
```

## Scale Down to 1

```powershell
kubectl scale deployment lab02api-deployment --replicas=1
```

## Access Service

```powershell
kubectl port-forward service/lab02api-service 8080:80
```

## Describe Deployment

```powershell
kubectl describe deployment lab02api-deployment
```

## Describe Service

```powershell
kubectl describe service lab02api-service
```

## View Logs

```powershell
kubectl logs -l app=lab02api
```

## Check Rollout

```powershell
kubectl rollout status deployment/lab02api-deployment
```

## Rollout History

```powershell
kubectl rollout history deployment/lab02api-deployment
```

## Roll Back

```powershell
kubectl rollout undo deployment/lab02api-deployment
```

## Delete Application

```powershell
kubectl delete -f k8s/
```

---

# Final Lab Workflow

```text
                 Lab 02
                    │
                    ▼
               Lab02API
                    │
               Dockerfile
                    │
             docker build
                    │
                    ▼
               Docker Image
                    │
             docker push
                    │
                    ▼
                Docker Hub
                    │
                    │
════════════════════╪════════════════════
                 LAB 03
                    │
                    ▼
             Kubernetes Cluster
                    │
                    ▼
            deployment.yaml
                    │
              kubectl apply
                    │
                    ▼
                Deployment
                    │
                    ▼
               ReplicaSet
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        Pod 1     Pod 2     Pod 3
          │         │         │
          └─────────┼─────────┘
                    │
                    ▼
                  Service
                    │
                    ▼
           kubectl port-forward
                    │
                    ▼
         http://localhost:8080
```

---

# Scaling Workflow

```text
Initial Deployment

replicas = 2

      Deployment
          │
      ┌───┴───┐
      ▼       ▼
    Pod 1   Pod 2


       SCALE UP

kubectl scale deployment
lab02api-deployment
--replicas=5

          │
          ▼

      Deployment
          │
 ┌────┬───┼───┬────┐
 ▼    ▼   ▼   ▼    ▼
P1   P2  P3  P4   P5


       SCALE DOWN

kubectl scale deployment
lab02api-deployment
--replicas=2

          │
          ▼

      Deployment
          │
      ┌───┴───┐
      ▼       ▼
    Pod 1   Pod 2
```

---

# What the Student Should Understand

At the end of Lab 03, the student should be able to explain:

### Docker Image

```text
The packaged Lab02API application.
```

### Kubernetes Pod

```text
A running instance of our Lab02API container.
```

### Deployment

```text
Manages Lab02API Pods
and controls how many replicas should run.
```

### Replica

```text
Another instance of Lab02API.
```

### Service

```text
Provides stable network access
to the Lab02API Pods.
```

### Scaling

```text
Increasing or decreasing
the number of application Pods.
```

---

# Lab Success Criteria

The lab is successfully completed when the student can demonstrate:

```text
✓ Kubernetes cluster running

✓ kubectl connected to docker-desktop

✓ Lab02API Deployment created

✓ Two Lab02API Pods running

✓ lab02api-service created

✓ API reachable through Service + port-forward

✓ Application scaled from 2 → 5 Pods

✓ Application scaled from 5 → 1 Pod

✓ Application returned to desired replica count

✓ Version 1.1 deployed

✓ Kubernetes rolling update observed

✓ Rollback command understood
```

---

# Final DevOps Learning Journey

```text
                  SOURCE CODE
                      │
                      ▼
                   .NET 10
                      │
                      ▼
                   Docker
                      │
                      ▼
                Docker Image
                      │
                      ▼
                 Docker Hub
                      │
                      ▼
                 Kubernetes
                      │
              ┌───────┴────────┐
              ▼                ▼
          Deployment         Service
              │                │
              ▼                │
          ReplicaSet           │
              │                │
       ┌──────┼──────┐         │
       ▼      ▼      ▼         │
     Pod 1  Pod 2  Pod 3 ◄─────┘
       │      │      │
       └──────┼──────┘
              │
              ▼
          Web Service
              │
              ▼
           End User
```

## GitHub Copilot's Role in This Lab

```text
Developer
    │
    ▼
GitHub Copilot
    │
    ├── Generate deployment.yaml
    │
    ├── Generate service.yaml
    │
    ├── Explain Kubernetes configuration
    │
    ├── Generate kubectl commands
    │
    ├── Diagnose Pod errors
    │
    ├── Explain scaling
    │
    └── Assist with rolling updates
```

The important point is that **Copilot assists the DevOps engineer; Kubernetes still performs the actual deployment and scaling**.

For a beginner class, I would **stop this lab at manual scaling first** rather than immediately adding HPA. Once students understand `Deployment → ReplicaSet → Pods → Service`, the next lab can introduce **Horizontal Pod Autoscaler: automatically scale Lab02API from 2 to 10 Pods based on CPU load**, including Metrics Server and a load-generation test.

[1]: https://docs.docker.com/desktop/use-desktop/kubernetes/ "Explore the Kubernetes view | Docker Docs"
[2]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ "Deployments | Kubernetes"
[3]: https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/?utm_source=chatgpt.com "Using a Service to Expose Your App"
[4]: https://code.visualstudio.com/docs/azure/kubernetes "Working with Kubernetes in VS Code"
[5]: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_scale/ "kubectl scale | Kubernetes"
[6]: https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-intro/?utm_source=chatgpt.com "Running Multiple Instances of Your App"
[7]: https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/?utm_source=chatgpt.com "Horizontal Pod Autoscaling"
[8]: https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/?utm_source=chatgpt.com "Resource metrics pipeline"
