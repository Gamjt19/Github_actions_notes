# GitHub Actions --- From Zero to CI/CD on Azure AKS

## Teaching purpose

This guide is a complete classroom note for learning and teaching GitHub
Actions from beginner level to practical CI/CD.

The teaching project is an **Employee Management Portal** with:

-   React + Vite + TypeScript frontend
-   Node.js + Express + TypeScript backend
-   PostgreSQL database
-   Docker
-   GitHub Actions
-   Azure Container Registry (ACR)
-   Azure Kubernetes Service (AKS)
-   Prometheus + Grafana as an optional monitoring stage

The same concepts can be applied to almost any GitHub repository.

------------------------------------------------------------------------

# PART 1 --- CI/CD FUNDAMENTALS

## 1. What is CI?

**Continuous Integration (CI)** means frequently integrating code
changes into a shared repository and automatically validating those
changes.

Typical CI activities:

``` text
Developer pushes code
        ↓
Build
        ↓
Install dependencies
        ↓
Lint
        ↓
Run tests
        ↓
Package/build
```

Important:

> CI is not just testing. Testing is one important part of CI.

------------------------------------------------------------------------

## 2. What is Continuous Delivery?

Continuous Delivery means keeping software in a releasable state.

The pipeline can automatically build, test and package the application,
but production release may require a human approval.

Example:

``` text
Push
 ↓
Test
 ↓
Build
 ↓
Package
 ↓
Ready for production
 ↓
Manual approval
 ↓
Production
```

------------------------------------------------------------------------

## 3. What is Continuous Deployment?

Continuous Deployment means a validated change is automatically deployed
without a manual production approval.

``` text
Push
 ↓
Test
 ↓
Build
 ↓
Deploy automatically
```

### Delivery vs Deployment

  Concept                 Meaning
  ----------------------- ----------------------------------------------
  Continuous Delivery     Software is kept ready for release
  Continuous Deployment   Validated software is automatically released

------------------------------------------------------------------------

# PART 2 --- WHAT IS GITHUB ACTIONS?

## 4. GitHub Actions

GitHub Actions is GitHub's automation platform for building workflows
that can build, test, package, release and deploy software.

A workflow is normally stored in:

``` text
.github/workflows/
```

Example:

``` text
repository/
└── .github/
    └── workflows/
        └── ci.yml
```

------------------------------------------------------------------------

# PART 3 --- THE MOST IMPORTANT MENTAL MODEL

Memorize:

``` text
EVENT
  ↓
WORKFLOW
  ↓
JOB
  ↓
RUNNER
  ↓
STEPS
  ↓
COMMANDS / ACTIONS
```

### Example

``` text
git push
   ↓
workflow starts
   ↓
test job
   ↓
Ubuntu runner
   ↓
checkout
   ↓
npm ci
   ↓
npm test
```

------------------------------------------------------------------------

# PART 4 --- WORKFLOW

## 5. Workflow

A workflow is a YAML file containing automation instructions.

Example:

``` yaml
name: CI

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Hello
        run: echo "Hello GitHub Actions"
```

### Important sections

``` yaml
name:
on:
jobs:
```

-   `name` → workflow display name
-   `on` → what triggers the workflow
-   `jobs` → work to execute

------------------------------------------------------------------------

# PART 5 --- YAML BASICS

## 6. YAML

YAML is indentation-sensitive.

Example:

``` yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

Do not randomly use tabs.

Lists use `-`:

``` yaml
steps:
  - name: Checkout
    uses: actions/checkout@v7

  - name: Build
    run: npm run build
```

------------------------------------------------------------------------

# PART 6 --- EVENTS / TRIGGERS

## 7. `on`

`on` defines when a workflow runs.

### Push

``` yaml
on:
  push:
```

Runs when code is pushed.

### Push to specific branches

``` yaml
on:
  push:
    branches:
      - main
      - develop
```

### Pull requests

``` yaml
on:
  pull_request:
    branches:
      - main
```

This means PRs targeting `main`.

### Manual execution

``` yaml
on:
  workflow_dispatch:
```

A user can manually run the workflow from GitHub.

### Scheduled execution

``` yaml
on:
  schedule:
    - cron: "0 0 * * *"
```

Cron uses UTC and is not intended for second-level precision.

### Multiple events

``` yaml
on:
  push:
  pull_request:
  workflow_dispatch:
```

These are alternatives (OR), not AND.

------------------------------------------------------------------------

# PART 7 --- WORKFLOW DISPATCH INPUTS

Manual workflows can receive inputs.

``` yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Deployment environment"
        required: true
        type: choice
        options:
          - staging
          - production
```

Use it:

``` yaml
run: echo "${{ inputs.environment }}"
```

Useful for:

-   choosing staging/production
-   selecting a version
-   selecting a deployment target
-   manually starting a release

------------------------------------------------------------------------

# PART 8 --- PATHS AND TAG FILTERS

You can restrict when a workflow runs.

``` yaml
on:
  push:
    branches:
      - main
    paths:
      - "backend/**"
      - "frontend/**"
```

You can also filter tags:

``` yaml
on:
  push:
    tags:
      - "v*"
```

This is useful when a release should happen only for version tags.

------------------------------------------------------------------------

# PART 9 --- JOBS

## 8. Jobs

Jobs are independent units of work.

``` yaml
jobs:
  test:
    runs-on: ubuntu-latest

  build:
    runs-on: ubuntu-latest
```

By default, independent jobs can run in parallel.

------------------------------------------------------------------------

# PART 10 --- JOB IDs VS JOB NAMES

Job ID:

``` yaml
jobs:
  build_app:
```

Display name:

``` yaml
jobs:
  build_app:
    name: Build Application
```

Use simple job IDs.

Good:

``` text
test
build
deploy
docker
```

Avoid YAML-special characters in job IDs.

------------------------------------------------------------------------

# PART 11 --- RUNNERS

## 9. Runner

A runner is the machine that executes a job.

``` yaml
runs-on: ubuntu-latest
```

GitHub-hosted runners are managed by GitHub.

Examples:

``` text
ubuntu-latest
windows-latest
macos-latest
```

A job runs on a runner.

------------------------------------------------------------------------

# PART 12 --- GITHUB-HOSTED VS SELF-HOSTED

## GitHub-hosted runner

GitHub provides and manages the machine.

Advantages:

-   easy setup
-   maintained by GitHub
-   clean environment
-   useful for standard CI

## Self-hosted runner

Your organization provides and manages the machine.

``` yaml
runs-on: self-hosted
```

You can also use labels:

``` yaml
runs-on: [self-hosted, linux, gpu]
```

Useful when you need:

-   private network access
-   on-premises resources
-   custom hardware
-   GPUs
-   special software
-   internal infrastructure

### Security warning

Self-hosted runners require careful security management.

Do not casually expose a persistent self-hosted runner to untrusted
code.

------------------------------------------------------------------------

# PART 13 --- STEPS

## 10. Steps

A step is one unit of work inside a job.

``` yaml
steps:
  - name: Checkout
    uses: actions/checkout@v7

  - name: Install
    run: npm ci

  - name: Test
    run: npm test
```

Every step runs on the same job runner and shares that job's workspace.

------------------------------------------------------------------------

# PART 14 --- `run` VS `uses`

## 11. `run`

Executes a shell command.

``` yaml
- name: Install dependencies
  run: npm ci
```

Examples:

``` yaml
run: npm test
run: docker build -t myapp .
run: kubectl get pods
```

## 12. `uses`

Uses a reusable GitHub Action.

``` yaml
- uses: actions/checkout@v7
```

An Action is a reusable unit of automation.

### Easy rule

``` text
run  → execute a command
uses → call an Action
```

------------------------------------------------------------------------

# PART 15 --- `with`

`with` passes inputs to an Action.

``` yaml
- uses: actions/setup-node@v7
  with:
    node-version: "24"
```

Think:

``` text
uses → which Action?
with  → how should the Action run?
```

------------------------------------------------------------------------

# PART 16 --- ACTIONS MARKETPLACE

Actions can be:

-   official
-   community-created
-   organization-created
-   custom actions in your repository

Format:

``` text
owner/repository@ref
```

Example:

``` yaml
uses: actions/checkout@v7
```

------------------------------------------------------------------------

# PART 17 --- ACTION VERSIONING AND PINNING

Possible references include:

``` text
@main
@v7
@v7.0.1
@<commit SHA>
```

Conceptually:

``` text
@main       → mutable branch
@v7         → major version
@v7.0.1     → exact release
@SHA        → exact commit
```

For production/security-sensitive workflows, exact commit SHA pinning
provides strong reproducibility and protects against a mutable tag being
moved.

For teaching, major-version references are easier:

``` yaml
uses: actions/checkout@v7
```

Always verify current Action versions before a live class.

------------------------------------------------------------------------

# PART 18 --- `needs`

Jobs run independently unless dependencies are defined.

``` yaml
jobs:
  test:
    runs-on: ubuntu-latest

  build:
    needs: test
    runs-on: ubuntu-latest
```

Meaning:

``` text
test
 ↓
build
```

Without `needs`, both jobs may start independently.

Multiple dependencies:

``` yaml
deploy:
  needs:
    - test
    - build
```

Deploy waits for both.

------------------------------------------------------------------------

# PART 19 --- `needs` DOES NOT SHARE FILES

This is a very common misunderstanding.

``` yaml
build:
  ...
deploy:
  needs: build
```

`needs` means:

> Wait for build.

It does NOT mean:

> Use the same runner/filesystem.

GitHub-hosted jobs normally get separate runners.

``` text
Job A runner
  ↓
files
  ↓
job ends
  ↓
runner gone

Job B
  ↓
fresh runner
```

------------------------------------------------------------------------

# PART 20 --- ARTIFACTS

Artifacts transfer/preserve files.

``` text
Runner A
   ↓
upload artifact
   ↓
GitHub artifact storage
   ↓
download artifact
   ↓
Runner B
```

Upload:

``` yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
```

Download:

``` yaml
- uses: actions/download-artifact@v4
  with:
    name: build-output
    path: dist/
```

### Remember

``` text
needs     → dependency/order
artifact  → file transfer/persistence
```

------------------------------------------------------------------------

# PART 21 --- ARTIFACTS VS CACHE

Artifacts:

> Preserve or transfer build outputs.

Cache:

> Speed up repeated work by reusing dependencies or other suitable data.

Do not treat cache as a general replacement for artifacts.

Example npm cache:

``` yaml
- uses: actions/setup-node@v7
  with:
    node-version: "24"
    cache: npm
```

------------------------------------------------------------------------

# PART 22 --- JOB OUTPUTS

Job outputs are useful for passing small values between dependent jobs.

Job A:

``` yaml
jobs:
  build:
    runs-on: ubuntu-latest

    outputs:
      image_tag: ${{ steps.version.outputs.tag }}

    steps:
      - id: version
        run: echo "tag=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"
```

Job B:

``` yaml
deploy:
  needs: build
  runs-on: ubuntu-latest

  steps:
    - run: echo "${{ needs.build.outputs.image_tag }}"
```

Remember:

``` text
outputs → small values
artifacts → files
needs → dependency
```

------------------------------------------------------------------------

# PART 23 --- ENVIRONMENT VARIABLES

Workflow level:

``` yaml
env:
  APP_NAME: employeehub
```

Job level:

``` yaml
jobs:
  build:
    env:
      APP_NAME: employeehub
```

Step level:

``` yaml
- name: Example
  env:
    APP_NAME: backend
  run: echo "$APP_NAME"
```

Scope:

``` text
workflow env
   ↓
all jobs/steps

job env
   ↓
all steps in that job

step env
   ↓
only that step
```

More specific values override broader values.

------------------------------------------------------------------------

# PART 24 --- `vars`

`vars` are GitHub configuration variables for non-sensitive values.

Example:

``` yaml
run: echo "${{ vars.ENVIRONMENT }}"
```

Good examples:

``` text
ENVIRONMENT=production
REGION=centralindia
APP_NAME=employeehub
```

Do not use `vars` for passwords or tokens.

------------------------------------------------------------------------

# PART 25 --- SECRETS

Secrets store sensitive values.

Examples:

``` text
API keys
passwords
cloud credentials
tokens
```

Use:

``` yaml
${{ secrets.API_KEY }}
```

Example:

``` yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

GitHub normally masks secrets in logs.

Never intentionally print secrets.

------------------------------------------------------------------------

# PART 26 --- ENV VS VARS VS SECRETS

  Feature     Purpose
  ----------- ---------------------------------------------
  `env`       Environment variable available to processes
  `vars`      Non-sensitive GitHub configuration
  `secrets`   Sensitive configuration

Simple memory:

``` text
env     → process environment
vars    → non-secret configuration
secrets → sensitive configuration
```

------------------------------------------------------------------------

# PART 27 --- CONTEXTS

Contexts provide information about the workflow/run.

Common examples:

``` yaml
${{ github.repository }}
${{ github.actor }}
${{ github.ref }}
${{ github.sha }}
${{ github.event_name }}
${{ github.run_number }}
```

Examples:

``` text
github.repository → Gamjt19/employee_managment_portal
github.actor     → account that triggered workflow
github.sha       → triggering commit SHA
github.event_name → push / pull_request / etc.
```

------------------------------------------------------------------------

# PART 28 --- EXPRESSIONS

GitHub expressions use:

``` text
${{ ... }}
```

Example:

``` yaml
run: echo "${{ github.sha }}"
```

Shell variable:

``` bash
echo "$APP_NAME"
```

GitHub expression:

``` yaml
${{ env.APP_NAME }}
```

Remember:

``` text
$VAR        → shell expansion
${{ ... }}  → GitHub Actions expression
```

------------------------------------------------------------------------

# PART 29 --- `if`

Conditions control whether a step or job runs.

Example:

``` yaml
if: github.ref == 'refs/heads/main'
```

Example:

``` yaml
deploy:
  if: github.ref == 'refs/heads/main'
```

Meaning:

> Run deployment only for main.

A false condition normally skips the job/step; it does not itself mean
the workflow failed.

------------------------------------------------------------------------

# PART 30 --- `needs` VS `if`

This distinction is important.

``` text
needs → When can I run?
if    → Should I run?
```

Example:

``` yaml
deploy:
  needs: test
  if: github.ref == 'refs/heads/main'
```

Meaning:

1.  Wait for test.
2.  Check whether this is main.
3.  Deploy only if condition is true.

------------------------------------------------------------------------

# PART 31 --- FAILURE BEHAVIOR

Normally:

``` text
Step 1 → success
Step 2 → failure
Step 3 → skipped
Job    → failed
```

A dependent job:

``` text
build
  ↓
deploy
```

If build fails, deploy is normally skipped.

`continue-on-error: true` can allow a step/job to continue despite a
failure.

Do not confuse:

``` text
continue-on-error
```

with making the command itself successful.

------------------------------------------------------------------------

# PART 32 --- STATUS FUNCTIONS

Useful conditions include:

``` yaml
if: failure()
```

Run something after a failure.

Other status functions include:

``` text
success()
failure()
cancelled()
always()
```

Use `always()` carefully, especially around cleanup and
security-sensitive logic.

------------------------------------------------------------------------

# PART 33 --- MATRIX

Matrix testing creates multiple combinations from one job definition.

``` yaml
strategy:
  matrix:
    node-version: ["22", "24"]
```

This can create:

``` text
Node 22
Node 24
```

Add OS:

``` yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node-version: ["22", "24"]
```

This creates combinations:

``` text
Ubuntu + Node 22
Ubuntu + Node 24
Windows + Node 22
Windows + Node 24
```

------------------------------------------------------------------------

# PART 34 --- MATRIX CONTROLS

## fail-fast

``` yaml
fail-fast: false
```

Do not cancel other matrix jobs immediately when one fails.

## max-parallel

``` yaml
max-parallel: 2
```

Limit simultaneous matrix jobs.

## include

Add a special combination.

## exclude

Remove a combination.

Memory:

``` text
matrix       → combinations
fail-fast    → cancellation behavior
max-parallel → concurrency
include      → add
exclude      → remove
```

------------------------------------------------------------------------

# PART 35 --- PERMISSIONS

GitHub provides `GITHUB_TOKEN`.

You can control permissions:

``` yaml
permissions:
  contents: read
  id-token: write
```

`contents: read`:

> Read repository contents.

`contents: write`:

> Write repository contents.

`id-token: write`:

> Allow the workflow to request an OIDC token.

Important:

> `id-token: write` does not mean "write to Azure resources."

Azure permissions are separately controlled by Azure RBAC.

Principle:

> Give workflows only the permissions they need.

------------------------------------------------------------------------

# PART 36 --- OIDC

OIDC is a modern way for GitHub Actions to authenticate to cloud
providers without storing a long-lived cloud password.

Concept:

``` text
GitHub Actions
      ↓
OIDC identity token
      ↓
Microsoft Entra ID
      ↓
Federated credential trust
      ↓
Azure identity
      ↓
Azure RBAC
      ↓
Azure resources
```

Authentication:

> Who are you?

Authorization:

> What are you allowed to do?

OIDC helps establish identity.

Azure RBAC determines access.

------------------------------------------------------------------------

# PART 37 --- OIDC REQUIREMENTS FOR AZURE

You normally need:

1.  Microsoft Entra application or supported managed identity
2.  Federated identity credential
3.  GitHub repository/branch or other trusted subject configured
4.  Azure role assignment
5.  GitHub Actions `id-token: write`
6.  Client ID / tenant ID / subscription ID

Example:

``` yaml
permissions:
  contents: read
  id-token: write
```

Azure login example:

``` yaml
- name: Azure Login
  uses: azure/login@v3
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

Do not put client secrets/passwords directly in YAML.

------------------------------------------------------------------------

# PART 38 --- SERVICE PRINCIPAL CREDENTIALS VS OIDC

Traditional approach:

``` text
Long-lived Azure credential
        ↓
GitHub secret
        ↓
azure/login
```

OIDC:

``` text
Short-lived identity token
        ↓
Federated trust
        ↓
Azure
```

OIDC is the preferred approach for new GitHub-to-Azure authentication
setups when supported.

------------------------------------------------------------------------

# PART 39 --- ENVIRONMENTS

GitHub Environments can represent:

``` text
development
staging
production
```

Example:

``` yaml
deploy:
  environment: production
```

An environment can have:

-   environment secrets
-   environment variables
-   protection rules
-   required reviewers

------------------------------------------------------------------------

# PART 40 --- PRODUCTION APPROVAL

Example flow:

``` text
Test
 ↓
Build
 ↓
Push image
 ↓
Production deployment job
 ↓
Required reviewer approval
 ↓
Deploy
```

The deployment job waits for approval if the environment requires it.

This is a common Continuous Delivery pattern.

------------------------------------------------------------------------

# PART 41 --- REUSABLE WORKFLOWS

A reusable workflow lets you reuse an entire workflow.

Use:

``` yaml
on:
  workflow_call:
```

Concept:

``` text
Repository A ─┐
Repository B ─┼──→ reusable workflow
Repository C ─┘
```

Useful when many repositories need the same CI/CD process.

------------------------------------------------------------------------

# PART 42 --- COMPOSITE ACTIONS

A composite Action packages multiple steps into a reusable Action.

Example idea:

``` text
Checkout
Install dependencies
Run lint
Run tests
```

can be packaged as a reusable step group.

### Important distinction

``` text
Reusable workflow → reuse a workflow/jobs
Composite action  → reuse a group of steps
```

------------------------------------------------------------------------

# PART 43 --- CONCURRENCY

Concurrency can prevent duplicate deployments.

Example:

``` yaml
concurrency:
  group: production
  cancel-in-progress: true
```

Useful when multiple commits are pushed quickly and only the latest
deployment should continue.

Use carefully for production workflows because cancellation behavior
should match the deployment strategy.

------------------------------------------------------------------------

# PART 44 --- SERVICES

Jobs can run service containers such as databases.

Concept:

``` yaml
services:
  postgres:
    image: postgres:...
```

Useful for integration tests.

Example architecture:

``` text
GitHub runner
 ├── application
 └── PostgreSQL service container
```

Then integration tests can connect to the temporary database.

------------------------------------------------------------------------

# PART 45 --- CACHING

Caching reduces repeated dependency downloads.

For Node:

``` yaml
- uses: actions/setup-node@v7
  with:
    node-version: "24"
    cache: npm
```

Cache is for speed, not the primary mechanism for passing build output
between jobs.

Be careful with cache permissions and untrusted pull requests.

------------------------------------------------------------------------

# PART 46 --- DOCKER BASICS

Docker packages an application and its dependencies into a container
image.

``` text
Application
+ dependencies
+ runtime
      ↓
Docker image
      ↓
Container
```

Why?

``` text
Developer environment
       ↓
CI environment
       ↓
Cloud environment
```

The same image can be used across environments.

------------------------------------------------------------------------

# PART 47 --- DOCKER IMAGE NAMING

General form:

``` text
registry/repository:tag
```

Docker Hub:

``` text
username/myapp:v1
```

Azure Container Registry:

``` text
myregistry.azurecr.io/myapp:v1
```

Commit-based tag:

``` text
myregistry.azurecr.io/myapp:${{ github.sha }}
```

------------------------------------------------------------------------

# PART 48 --- WHY USE GITHUB SHA AS IMAGE TAG?

Example:

``` text
myregistry.azurecr.io/employeehub-backend:a8f31c2...
```

The tag identifies the source commit.

Benefits:

-   traceability
-   reproducibility
-   easier rollback
-   clear deployment history

`latest` is mutable and does not uniquely identify a source commit.

------------------------------------------------------------------------

# PART 49 --- DOCKER MULTI-STAGE BUILD

For a frontend:

``` text
Node image
   ↓
npm ci
   ↓
npm run build
   ↓
dist/
   ↓
Nginx image
   ↓
serve dist/
```

Build tools do not need to remain in the final runtime image.

------------------------------------------------------------------------

# PART 50 --- CONTAINER REGISTRY

A registry stores container images.

Examples:

-   Azure Container Registry
-   Docker Hub
-   GitHub Container Registry
-   Amazon ECR
-   Google Artifact Registry

Flow:

``` text
Docker build
     ↓
Docker image
     ↓
Registry
     ↓
Deployment platform pulls image
```

------------------------------------------------------------------------

# PART 51 --- AZURE CONTAINER REGISTRY

ACR is Azure's private container registry.

Example:

``` text
employeehubacr.azurecr.io
```

Images:

``` text
employeehubacr.azurecr.io/employeehub-backend:<sha>
employeehubacr.azurecr.io/employeehub-frontend:<sha>
```

Push flow:

``` text
GitHub Actions
     ↓
Azure authentication
     ↓
ACR authentication
     ↓
docker push
```

------------------------------------------------------------------------

# PART 52 --- AKS

Azure Kubernetes Service is Microsoft's managed Kubernetes service.

Kubernetes manages:

-   containers
-   pods
-   deployments
-   services
-   scaling
-   rolling updates
-   networking

Typical flow:

``` text
Developer
 ↓
GitHub
 ↓
GitHub Actions
 ↓
ACR
 ↓
AKS
```

------------------------------------------------------------------------

# PART 53 --- KUBERNETES CORE CONCEPTS

## Pod

Smallest deployable unit.

Usually contains one main application container.

## Deployment

Manages replicated Pods and rolling updates.

## Service

Provides stable networking to Pods.

Common types:

``` text
ClusterIP
NodePort
LoadBalancer
```

## Namespace

Logical isolation/grouping inside a cluster.

------------------------------------------------------------------------

# PART 54 --- AKS DEPLOYMENT FLOW

Before deployment:

``` text
Docker image
      ↓
ACR
```

Then:

``` text
AKS
 ↓
Deployment manifest
 ↓
Pod
 ↓
Container pulls image from ACR
```

------------------------------------------------------------------------

# PART 55 --- AKS AND ACR TRUST

AKS needs permission to pull images from ACR.

Common architecture:

``` text
GitHub Actions
      ↓ push
ACR
      ↑ pull
AKS
```

GitHub Actions needs push permission.

AKS needs pull permission.

These are separate authorization problems.

------------------------------------------------------------------------

# PART 56 --- AZURE DEPLOYMENT ACTIONS

Common Azure/GitHub Actions concepts include:

``` text
azure/login
azure/aks-set-context
azure/k8s-deploy
azure/setup-kubectl
azure/setup-helm
```

Typical sequence:

``` text
Azure Login
 ↓
Set AKS context
 ↓
Deploy Kubernetes manifests
```

------------------------------------------------------------------------

# PART 57 --- AKS DEPLOYMENT

Example:

``` yaml
- name: Azure Login
  uses: azure/login@v3
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

- name: Set AKS context
  uses: azure/aks-set-context@v5
  with:
    resource-group: ${{ vars.AKS_RESOURCE_GROUP }}
    cluster-name: ${{ vars.AKS_CLUSTER }}

- name: Deploy to AKS
  uses: azure/k8s-deploy@v5
  with:
    namespace: default
    manifests: |
      k8s/backend-deployment.yaml
      k8s/backend-service.yaml
```

Exact Action versions should be checked before a live class.

------------------------------------------------------------------------

# PART 58 --- APPLICATION GATEWAY / INGRESS

For more advanced AKS deployments:

``` text
Internet
   ↓
Ingress / Application Gateway
   ↓
Frontend Service
   ↓
Frontend Pods

API traffic
   ↓
Backend Service
   ↓
Backend Pods
```

Path-based routing example:

``` text
/app       → frontend
/api       → backend
```

Authentication can be implemented by the application or an
identity-aware gateway depending on architecture.

------------------------------------------------------------------------

# PART 59 --- MONITORING

Monitoring answers:

> Is the system healthy and what is happening?

Prometheus:

> Collects metrics.

Grafana:

> Visualizes metrics.

Typical flow:

``` text
Application / Kubernetes
        ↓
     Metrics
        ↓
   Prometheus
        ↓
      Grafana
```

------------------------------------------------------------------------

# PART 60 --- GITHUB ACTIONS SECURITY

Important security practices:

1.  Least privilege
2.  Use OIDC instead of long-lived cloud credentials where possible
3.  Store sensitive values in secrets
4.  Pin critical third-party Actions
5.  Be careful with pull requests from forks
6.  Do not print secrets
7.  Review third-party Actions
8.  Restrict production environments
9.  Protect deployment branches
10. Avoid unnecessary `write` permissions

------------------------------------------------------------------------

# PART 61 --- PULL REQUEST SECURITY

Forked pull requests can be untrusted.

Be especially careful with workflows that expose:

-   secrets
-   write permissions
-   privileged self-hosted runners

Do not assume code from a fork is trusted just because it is a pull
request.

------------------------------------------------------------------------

# PART 62 --- CONTINUE-ON-ERROR

Example:

``` yaml
- name: Optional check
  run: npm run lint
  continue-on-error: true
```

This allows the workflow to continue even if that step fails.

Use carefully.

A security scan or production deployment should not automatically be
allowed to fail unless that behavior is intentional.

------------------------------------------------------------------------

# PART 63 --- REAL PROJECT: EMPLOYEE MANAGEMENT PORTAL

Repository:

``` text
Gamjt19/employee_managment_portal
```

Architecture:

``` text
employee_managment_portal/
├── backend/
├── frontend/
├── package.json
├── package-lock.json
└── ...
```

Root workspace contains:

``` json
"workspaces": [
  "backend",
  "frontend"
]
```

Root build:

``` bash
npm run build
```

which builds:

``` text
backend
+
frontend
```

Backend:

``` text
Node.js
Express
TypeScript
PostgreSQL
```

Frontend:

``` text
React
Vite
TypeScript
Tailwind
```

------------------------------------------------------------------------

# PART 64 --- HANDS-ON PROJECT PLAN

We will build the pipeline in stages.

## Stage 1 --- Local application

Verify:

``` bash
npm ci
npm run build
```

Verify frontend and backend work locally.

------------------------------------------------------------------------

## Stage 2 --- Docker

Backend:

``` bash
docker build -t employeehub-backend:local ./backend
```

Frontend:

``` bash
docker build -t employeehub-frontend:local ./frontend
```

Run and verify both containers.

------------------------------------------------------------------------

## Stage 3 --- Basic CI

Start with:

``` text
push
 ↓
checkout
 ↓
setup Node
 ↓
npm ci
 ↓
npm run build
```

This teaches:

-   events
-   jobs
-   runners
-   steps
-   actions
-   `run`
-   `uses`

------------------------------------------------------------------------

# PART 65 --- BASIC CI WORKFLOW

Create:

``` text
.github/workflows/ci.yml
```

Example:

``` yaml
name: CI

on:
  push:
  pull_request:
    branches:
      - main

permissions:
  contents: read

jobs:
  test:
    name: Build and Validate
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v7

      - name: Setup Node
        uses: actions/setup-node@v7
        with:
          node-version: "24"

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build
```

If the project later adds a root test script:

``` yaml
- name: Run tests
  run: npm test
```

------------------------------------------------------------------------

# PART 66 --- TEACH THE STUDENTS TO WRITE IT THEMSELVES

Do not immediately give the whole YAML.

Ask:

### Step 1

What is the workflow name?

``` yaml
name: CI
```

### Step 2

When should it run?

``` yaml
on:
  push:
  pull_request:
    branches:
      - main
```

### Step 3

Where are jobs defined?

``` yaml
jobs:
```

### Step 4

What is the job called?

``` yaml
test:
```

### Step 5

Where does it run?

``` yaml
runs-on: ubuntu-latest
```

### Step 6

How do we get source code?

``` yaml
uses: actions/checkout@v7
```

### Step 7

How do we install Node?

``` yaml
uses: actions/setup-node@v7
```

### Step 8

How do we install dependencies?

``` yaml
run: npm ci
```

### Step 9

How do we build?

``` yaml
run: npm run build
```

This teaches the students to construct the workflow instead of
memorizing it.

------------------------------------------------------------------------

# PART 67 --- DOCKER CI JOB

Once basic CI works, add:

``` text
test
 ↓
docker
```

Example:

``` yaml
docker:
  needs: test
  runs-on: ubuntu-latest

  steps:
    - name: Checkout
      uses: actions/checkout@v7

    - name: Build backend image
      run: |
        docker build \
          -t employeehub-backend:${{ github.sha }} \
          ./backend

    - name: Build frontend image
      run: |
        docker build \
          -t employeehub-frontend:${{ github.sha }} \
          ./frontend
```

Important teaching point:

`needs: test` provides ordering.

It does not transfer the checkout from the test runner.

Therefore Docker job checks out the repository again.

------------------------------------------------------------------------

# PART 68 --- WHY NOT USE ARTIFACTS HERE?

We could upload build files, but Docker can build the image from source.

For this demonstration:

``` text
test job
 ↓
needs
 ↓
docker job
 ↓
checkout again
 ↓
docker build
```

is simpler.

Artifacts become useful when one job produces a file that another job
needs.

------------------------------------------------------------------------

# PART 69 --- TAG DOCKER IMAGES WITH COMMIT SHA

For ACR:

``` text
employeehubacr.azurecr.io/employeehub-backend:${{ github.sha }}
employeehubacr.azurecr.io/employeehub-frontend:${{ github.sha }}
```

Why?

Because:

``` text
commit → image
```

is traceable.

------------------------------------------------------------------------

# PART 70 --- CREATE AZURE RESOURCES

You need:

1.  Azure subscription
2.  Resource group
3.  Azure Container Registry
4.  AKS cluster
5.  ACR-to-AKS pull access
6.  Azure identity for GitHub Actions
7.  Federated credential
8.  ACR push permission
9.  AKS deployment permission

Example resource names:

``` text
Resource Group:
rg-employeehub

ACR:
employeehubacr

AKS:
aks-employeehub
```

Names must be globally/regionally valid according to the specific Azure
resource rules.

------------------------------------------------------------------------

# PART 71 --- AZURE OIDC SETUP

### Step 1

Microsoft Entra ID:

``` text
App registrations
 → New registration
```

Example:

``` text
github-actions-employeehub
```

Single tenant is sufficient for a typical single-tenant lab setup.

No redirect URI is needed for this GitHub Actions OIDC scenario.

### Step 2

Record:

``` text
Application (client) ID
Directory (tenant) ID
Subscription ID
```

Do not paste sensitive values into public repositories.

------------------------------------------------------------------------

# PART 72 --- FEDERATED CREDENTIAL

Inside the Entra application:

``` text
Certificates & secrets
 → Federated credentials
 → Add credential
```

Choose GitHub Actions.

Configure the repository and trust condition.

For a main-branch-only classroom pipeline, configure trust for:

``` text
Gamjt19/employee_managment_portal
main
```

The exact portal wording can change; verify the
subject/repository/branch values before saving.

------------------------------------------------------------------------

# PART 73 --- AZURE RBAC

The GitHub identity needs permission to push images to ACR.

Give the appropriate ACR push role at the appropriate scope.

Conceptually:

``` text
GitHub identity
     ↓
AcrPush
     ↓
EmployeeHub ACR
```

AKS needs its own permission to pull from ACR.

Conceptually:

``` text
AKS identity
     ↓
AcrPull
     ↓
EmployeeHub ACR
```

Do not confuse the two.

------------------------------------------------------------------------

# PART 74 --- GITHUB SECRETS / VARIABLES

Repository Settings:

``` text
Settings
 → Secrets and variables
 → Actions
```

Secrets:

``` text
AZURE_CLIENT_ID
AZURE_TENANT_ID
AZURE_SUBSCRIPTION_ID
```

Non-sensitive variables:

``` text
ACR_NAME
AKS_CLUSTER
AKS_RESOURCE_GROUP
```

For example:

``` yaml
${{ vars.ACR_NAME }}
```

and:

``` yaml
${{ secrets.AZURE_CLIENT_ID }}
```

------------------------------------------------------------------------

# PART 75 --- AZURE LOGIN

Workflow permission:

``` yaml
permissions:
  contents: read
  id-token: write
```

Login:

``` yaml
- name: Azure Login
  uses: azure/login@v3
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

------------------------------------------------------------------------

# PART 76 --- LOGIN TO ACR

After Azure login:

``` yaml
- name: Login to ACR
  run: az acr login --name "${{ vars.ACR_NAME }}"
```

Now Docker can communicate with the registry.

------------------------------------------------------------------------

# PART 77 --- BUILD AND PUSH

Build:

``` yaml
- name: Build backend
  run: |
    docker build \
      -t ${{ vars.ACR_NAME }}.azurecr.io/employeehub-backend:${{ github.sha }} \
      ./backend
```

Frontend:

``` yaml
- name: Build frontend
  run: |
    docker build \
      -t ${{ vars.ACR_NAME }}.azurecr.io/employeehub-frontend:${{ github.sha }} \
      ./frontend
```

Push:

``` yaml
- name: Push backend
  run: |
    docker push \
      ${{ vars.ACR_NAME }}.azurecr.io/employeehub-backend:${{ github.sha }}
```

``` yaml
- name: Push frontend
  run: |
    docker push \
      ${{ vars.ACR_NAME }}.azurecr.io/employeehub-frontend:${{ github.sha }}
```

------------------------------------------------------------------------

# PART 78 --- FULL BUILD/PUSH JOB

A teaching version:

``` yaml
docker:
  needs: test
  runs-on: ubuntu-latest

  steps:
    - name: Checkout
      uses: actions/checkout@v7

    - name: Azure Login
      uses: azure/login@v3
      with:
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

    - name: Login to ACR
      run: az acr login --name "${{ vars.ACR_NAME }}"

    - name: Build backend
      run: |
        docker build \
          -t ${{ vars.ACR_NAME }}.azurecr.io/employeehub-backend:${{ github.sha }} \
          ./backend

    - name: Build frontend
      run: |
        docker build \
          -t ${{ vars.ACR_NAME }}.azurecr.io/employeehub-frontend:${{ github.sha }} \
          ./frontend

    - name: Push backend
      run: |
        docker push \
          ${{ vars.ACR_NAME }}.azurecr.io/employeehub-backend:${{ github.sha }}

    - name: Push frontend
      run: |
        docker push \
          ${{ vars.ACR_NAME }}.azurecr.io/employeehub-frontend:${{ github.sha }}
```

------------------------------------------------------------------------

# PART 79 --- KUBERNETES MANIFESTS

Create:

``` text
k8s/
├── backend-deployment.yaml
├── backend-service.yaml
├── frontend-deployment.yaml
└── frontend-service.yaml
```

Deployment describes Pods.

Service provides networking.

Example backend Deployment:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: employeehub-backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: employeehub-backend
  template:
    metadata:
      labels:
        app: employeehub-backend
    spec:
      containers:
        - name: backend
          image: employeehubacr.azurecr.io/employeehub-backend:IMAGE_TAG
          ports:
            - containerPort: 5000
```

The exact image and environment variables must match the actual
application.

------------------------------------------------------------------------

# PART 80 --- BACKEND SERVICE

Example:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: employeehub-backend
spec:
  selector:
    app: employeehub-backend
  ports:
    - port: 5000
      targetPort: 5000
  type: ClusterIP
```

Inside the cluster, frontend can communicate with the backend Service.

------------------------------------------------------------------------

# PART 81 --- FRONTEND DEPLOYMENT

Example:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: employeehub-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: employeehub-frontend
  template:
    metadata:
      labels:
        app: employeehub-frontend
    spec:
      containers:
        - name: frontend
          image: employeehubacr.azurecr.io/employeehub-frontend:IMAGE_TAG
          ports:
            - containerPort: 80
```

------------------------------------------------------------------------

# PART 82 --- FRONTEND SERVICE

For a simple classroom demo:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: employeehub-frontend
spec:
  selector:
    app: employeehub-frontend
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer
```

This can expose the frontend externally through an Azure load balancer.

For a production architecture, consider Ingress/Application Gateway and
more robust networking.

------------------------------------------------------------------------

# PART 83 --- DATABASE

The EmployeeHub backend uses PostgreSQL.

For a classroom AKS deployment, choose deliberately:

### Option A --- Azure Database for PostgreSQL

Recommended for a realistic cloud architecture.

``` text
AKS
 ↓
Backend
 ↓
Azure Database for PostgreSQL
```

### Option B --- PostgreSQL inside Kubernetes

Simpler conceptually but adds stateful workload management.

For a production-style Azure deployment, a managed database is generally
easier to operate.

Never hardcode:

``` text
database password
connection string
```

Use Kubernetes Secrets or another appropriate secret-management
solution.

------------------------------------------------------------------------

# PART 84 --- AKS CONTEXT

After Azure login:

``` yaml
- name: Set AKS context
  uses: azure/aks-set-context@v5
  with:
    resource-group: ${{ vars.AKS_RESOURCE_GROUP }}
    cluster-name: ${{ vars.AKS_CLUSTER }}
    admin: "false"
    use-kubelogin: "true"
```

This prepares the runner to interact with the cluster.

------------------------------------------------------------------------

# PART 85 --- DEPLOY TO AKS

Example:

``` yaml
- name: Deploy to AKS
  uses: azure/k8s-deploy@v5
  with:
    namespace: default
    manifests: |
      k8s/backend-deployment.yaml
      k8s/backend-service.yaml
      k8s/frontend-deployment.yaml
      k8s/frontend-service.yaml
    images: |
      ${{ vars.ACR_NAME }}.azurecr.io/employeehub-backend:${{ github.sha }}
      ${{ vars.ACR_NAME }}.azurecr.io/employeehub-frontend:${{ github.sha }}
```

The deployment action can substitute the specified image references into
manifests and apply the Kubernetes deployment.

Verify the exact inputs supported by the Action version used in the live
class.

------------------------------------------------------------------------

# PART 86 --- IMPORTANT: IMAGE TAGS IN KUBERNETES

Avoid:

``` text
:latest
```

for a teaching pipeline that wants clear traceability.

Prefer:

``` text
:${{ github.sha }}
```

Then:

``` text
Git commit
   ↓
ACR image tag
   ↓
Kubernetes deployment
```

------------------------------------------------------------------------

# PART 87 --- PRODUCTION APPROVAL

Create a GitHub Environment:

``` text
Settings
 → Environments
 → production
```

Configure required reviewers.

Then:

``` yaml
deploy:
  environment: production
```

Pipeline:

``` text
Test
 ↓
Build
 ↓
Push to ACR
 ↓
Production approval
 ↓
Deploy to AKS
```

This demonstrates Continuous Delivery.

------------------------------------------------------------------------

# PART 88 --- FINAL CLASSROOM PIPELINE

Conceptually:

``` text
Developer
   │
   │ git push
   ▼
GitHub
   │
   ▼
┌────────────────────┐
│ GitHub Actions     │
└─────────┬──────────┘
          │
          ▼
       CI / Test
          │
       success
          ▼
     Docker Build
       │      │
       ▼      ▼
   Backend  Frontend
       │      │
       └──┬───┘
          ▼
         ACR
          │
          ▼
  Production Approval
          │
          ▼
         AKS
       ┌──┴──┐
       ▼     ▼
   Frontend Backend
              │
              ▼
           PostgreSQL
```

------------------------------------------------------------------------

# PART 89 --- COMPLETE TEACHING WORKFLOW TEMPLATE

Use this as the final teaching template and customize resource names:

``` yaml
name: EmployeeHub CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

jobs:

  test:
    name: Build and Validate
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v7

      - name: Setup Node
        uses: actions/setup-node@v7
        with:
          node-version: "24"

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build


  docker:
    name: Build and Push Images
    needs: test
    if: github.event_name != 'pull_request'
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v7

      - name: Azure Login
        uses: azure/login@v3
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Login to ACR
        run: az acr login --name "${{ vars.ACR_NAME }}"

      - name: Build backend image
        run: |
          docker build \
            -t ${{ vars.ACR_NAME }}.azurecr.io/employeehub-backend:${{ github.sha }} \
            ./backend

      - name: Build frontend image
        run: |
          docker build \
            -t ${{ vars.ACR_NAME }}.azurecr.io/employeehub-frontend:${{ github.sha }} \
            ./frontend

      - name: Push backend image
        run: |
          docker push \
            ${{ vars.ACR_NAME }}.azurecr.io/employeehub-backend:${{ github.sha }}

      - name: Push frontend image
        run: |
          docker push \
            ${{ vars.ACR_NAME }}.azurecr.io/employeehub-frontend:${{ github.sha }}


  deploy:
    name: Deploy to AKS
    needs: docker
    environment: production
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v7

      - name: Azure Login
        uses: azure/login@v3
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Set AKS context
        uses: azure/aks-set-context@v5
        with:
          resource-group: ${{ vars.AKS_RESOURCE_GROUP }}
          cluster-name: ${{ vars.AKS_CLUSTER }}
          admin: "false"
          use-kubelogin: "true"

      - name: Deploy to AKS
        uses: azure/k8s-deploy@v5
        with:
          namespace: default
          manifests: |
            k8s/backend-deployment.yaml
            k8s/backend-service.yaml
            k8s/frontend-deployment.yaml
            k8s/frontend-service.yaml
          images: |
            ${{ vars.ACR_NAME }}.azurecr.io/employeehub-backend:${{ github.sha }}
            ${{ vars.ACR_NAME }}.azurecr.io/employeehub-frontend:${{ github.sha }}
```

### Important

This is a **teaching template**, not a promise that it will run
unchanged in every repository.

Before running it, verify:

-   Node version required by the project
-   actual Dockerfiles
-   ACR name
-   AKS resource group
-   AKS cluster name
-   GitHub secrets
-   GitHub variables
-   OIDC federated credential
-   Azure RBAC
-   PostgreSQL connectivity
-   Kubernetes manifests
-   frontend API URL
-   Action versions

------------------------------------------------------------------------

# PART 90 --- HOW STUDENTS USE THIS FOR THEIR OWN REPOSITORY

Tell students:

> GitHub Actions is not tied to EmployeeHub. The workflow must match the
> technology and commands of your repository.

### Step 1 --- Inspect the project

Find:

``` text
package.json
requirements.txt
pom.xml
build.gradle
Dockerfile
Makefile
```

depending on the technology.

### Step 2 --- Find build/test commands

Examples:

Node:

``` bash
npm ci
npm test
npm run build
```

Python:

``` bash
pip install -r requirements.txt
pytest
```

Java:

``` bash
mvn test
mvn package
```

.NET:

``` bash
dotnet restore
dotnet test
dotnet build
```

### Step 3 --- Identify deployment artifact

Could be:

``` text
Docker image
JAR
ZIP
static files
package
```

### Step 4 --- Write the CI pipeline

``` text
Event
 ↓
Checkout
 ↓
Setup runtime
 ↓
Install dependencies
 ↓
Test
 ↓
Build
```

### Step 5 --- Add CD

``` text
Build
 ↓
Package
 ↓
Registry/artifact store
 ↓
Cloud
```

### Step 6 --- Add environment/security

``` text
Secrets
Variables
Permissions
Environment
Approval
OIDC
```

------------------------------------------------------------------------

# PART 91 --- OTHER CI/CD PLATFORMS

The concepts are portable.

## GitHub Actions

``` text
Workflow → Job → Step
```

## GitLab CI/CD

Usually:

``` text
.gitlab-ci.yml
stages
jobs
```

## Jenkins

Common concepts:

``` text
Pipeline
Stages
Steps
Agents
```

## Azure DevOps

Common concepts:

``` text
Pipeline
Stages
Jobs
Steps
Agents
```

## CircleCI

Common concepts:

``` text
Config
Jobs
Workflows
Executors
```

The syntax changes, but the core CI/CD ideas remain:

``` text
Trigger
 ↓
Build
 ↓
Test
 ↓
Package
 ↓
Deploy
```

------------------------------------------------------------------------

# PART 92 --- GITHUB ACTIONS VS OTHER TOOLS

Do not teach one tool as universally best.

Instead compare based on context.

  Tool             Common strength
  ---------------- -----------------------------------------------
  GitHub Actions   Strong GitHub integration
  GitLab CI/CD     Integrated DevOps platform
  Jenkins          Highly customizable, large plugin ecosystem
  Azure DevOps     Strong Microsoft/Azure enterprise integration
  CircleCI         CI-focused hosted platform

The correct tool depends on:

-   existing source control
-   infrastructure
-   security requirements
-   team skills
-   cost
-   compliance
-   integrations
-   operational model

------------------------------------------------------------------------

# PART 93 --- INTERVIEW QUESTIONS

## Beginner

### What is GitHub Actions?

A GitHub automation platform for building workflows that can build,
test, package and deploy software.

### What is a workflow?

A YAML-defined automation process.

### What is a job?

A unit of work executed on a runner.

### What is a step?

An individual command or Action inside a job.

### What is a runner?

The machine that executes a job.

### What is `run`?

Runs a shell command.

### What is `uses`?

Calls a reusable Action.

------------------------------------------------------------------------

# PART 94 --- INTERMEDIATE QUESTIONS

### What does `needs` do?

Defines a job dependency.

### Does `needs` share files?

No.

### How do you transfer files between jobs?

Artifacts or another external storage mechanism.

### What is a matrix?

A way to run one job definition across multiple combinations.

### What is `github.sha`?

The commit SHA associated with the workflow run.

### Why tag Docker images with SHA?

Traceability, reproducibility and rollback.

### What is a secret?

Sensitive configuration stored securely by GitHub.

### What is a context?

Information exposed by GitHub about the workflow, event, repository,
job, etc.

------------------------------------------------------------------------

# PART 95 --- ADVANCED QUESTIONS

### Why use OIDC?

To establish cloud identity without relying on long-lived cloud
credentials.

### Authentication vs authorization?

Authentication = who are you?

Authorization = what can you do?

### Why use environments?

Environment-specific configuration and deployment protection/approval.

### Reusable workflow vs composite Action?

Reusable workflow = reuse workflows/jobs.

Composite Action = reuse multiple steps as an Action.

### GitHub-hosted vs self-hosted runner?

GitHub-hosted is managed by GitHub.

Self-hosted is managed by the organization.

### Artifact vs cache?

Artifact = output/file transfer.

Cache = speed repeated work.

### `needs` vs `if`?

`needs` controls dependency/order.

`if` controls whether execution should occur.

------------------------------------------------------------------------

# PART 96 --- LIVE FAILURE DEMO

A very useful classroom demonstration:

### Demo 1 --- Break the build

Introduce an intentional TypeScript error.

Push.

Show:

``` text
Checkout      ✓
Setup Node    ✓
npm ci        ✓
npm run build ✗
```

Explain:

> CI stopped because the validation/build step failed.

Fix the error.

Push again.

Show green pipeline.

------------------------------------------------------------------------

# PART 97 --- LIVE DOCKER DEMO

Show:

``` bash
docker images
```

Then:

``` bash
docker build -t employeehub-backend:test ./backend
```

Then:

``` bash
docker images
```

Explain:

``` text
Dockerfile
 ↓
docker build
 ↓
image
```

------------------------------------------------------------------------

# PART 98 --- LIVE ACR DEMO

After authentication:

``` bash
az acr login --name employeehubacr
```

Show:

``` bash
docker push employeehubacr.azurecr.io/employeehub-backend:<sha>
```

Then show the image in Azure Portal.

Explain:

``` text
Local/CI image
      ↓
ACR
      ↓
stored container artifact
```

------------------------------------------------------------------------

# PART 99 --- LIVE AKS DEMO

Show:

``` bash
kubectl get nodes
```

Then:

``` bash
kubectl get pods
```

Then:

``` bash
kubectl get svc
```

After deployment:

``` text
Deployment
 ↓
Pods
 ↓
Service
 ↓
Application
```

Show the application URL.

------------------------------------------------------------------------

# PART 100 --- LIVE ROLLBACK CONCEPT

Suppose:

``` text
v1 → working
v2 → broken
```

If images are tagged by commit:

``` text
ACR
 ├── backend:commit-v1
 └── backend:commit-v2
```

You can deploy the known-good image again.

This is one reason immutable/traceable image tags are valuable.

------------------------------------------------------------------------

# PART 101 --- FINAL TEACHING CHEAT SHEET

Memorize this:

``` text
Git push
   ↓
Event
   ↓
Workflow
   ↓
Job
   ↓
Runner
   ↓
Steps
   ├── run
   └── uses
   ↓
Build/Test
   ↓
Docker
   ↓
Registry
   ↓
Environment/Approval
   ↓
Cloud
   ↓
AKS
   ↓
Application
```

And remember:

``` text
on        → when
jobs      → what work
runs-on   → where
steps     → how
run       → command
uses      → Action
with      → Action inputs
needs     → dependency
if        → condition
env       → environment variables
vars      → non-secret config
secrets   → sensitive config
context   → GitHub information
outputs   → values between jobs
artifacts → files between jobs
cache     → speed repeated work
matrix    → combinations
permissions→ access control
environment→ deployment boundary/approval
OIDC      → cloud identity
```

------------------------------------------------------------------------

# PART 102 --- THE GOLDEN RULE FOR WRITING ANY PIPELINE

When a student is given a new project, ask:

### 1. What starts the pipeline?

``` text
push?
pull request?
manual?
schedule?
tag?
```

### 2. What must happen?

``` text
install
test
build
package
```

### 3. What is the output?

``` text
Docker image?
JAR?
ZIP?
static files?
```

### 4. Where does it go?

``` text
ACR?
Docker Hub?
GitHub Packages?
Azure?
AWS?
Kubernetes?
```

### 5. What credentials are required?

``` text
secrets?
OIDC?
environment?
```

### 6. What dependencies exist?

``` text
needs?
artifacts?
services?
outputs?
```

### 7. What should be protected?

``` text
production environment
approval
permissions
secrets
branches
```

This turns CI/CD from memorization into **pipeline design**.

------------------------------------------------------------------------

# PART 103 --- FINAL CLASSROOM EXERCISE

Ask students to build this from scratch:

``` text
Employee Management Portal
        ↓
GitHub
        ↓
GitHub Actions
        ↓
CI
        ↓
Docker
        ↓
ACR
        ↓
AKS
```

Require them to create:

``` text
.github/workflows/cicd.yml

Dockerfiles

k8s/
├── backend-deployment.yaml
├── backend-service.yaml
├── frontend-deployment.yaml
└── frontend-service.yaml
```

Then require:

``` text
1. Push code
2. CI runs
3. Build succeeds
4. Docker images are created
5. Images are pushed to ACR
6. Deployment waits for production approval
7. AKS receives the new images
8. Pods become Ready
9. Application becomes accessible
```

------------------------------------------------------------------------

# PART 104 --- WHAT STUDENTS SHOULD BE ABLE TO DO AFTER THE CLASS

A student should be able to:

-   explain CI/CD
-   explain GitHub Actions architecture
-   create a workflow from scratch
-   choose appropriate triggers
-   create jobs and steps
-   select runners
-   use `run`, `uses`, and `with`
-   use `needs` and `if`
-   use environment variables
-   use contexts and expressions
-   distinguish `vars` and `secrets`
-   use artifacts and caches correctly
-   use job outputs
-   create matrix workflows
-   configure permissions
-   explain OIDC
-   configure environments and approvals
-   understand reusable workflows
-   understand composite Actions
-   understand self-hosted runners
-   understand Action pinning
-   Dockerize applications
-   build and tag container images
-   push images to ACR
-   deploy images to AKS
-   understand Kubernetes Deployments and Services
-   troubleshoot failed workflows
-   adapt the pipeline to another repository
-   explain the difference between GitHub Actions, Jenkins, GitLab CI/CD
    and Azure DevOps

------------------------------------------------------------------------

# Official references

Use these when preparing the class and before live demos because Action
versions and portal screens can change:

-   GitHub Actions workflow syntax:
    https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
-   GitHub Actions documentation: https://docs.github.com/en/actions
-   GitHub Actions checkout: https://github.com/actions/checkout
-   GitHub Actions setup-node: https://github.com/actions/setup-node
-   Azure Login: https://github.com/Azure/login
-   Azure GitHub OIDC:
    https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure-openid-connect
-   Azure AKS GitHub Actions:
    https://learn.microsoft.com/en-us/azure/aks/kubernetes-action
-   Azure AKS set context: https://github.com/Azure/aks-set-context
-   Azure Kubernetes deploy Action: https://github.com/Azure/k8s-deploy

------------------------------------------------------------------------

# One-page instructor summary

## Explain first

``` text
CI/CD
 ↓
GitHub Actions
 ↓
Event → Workflow → Job → Runner → Step
```

## Then teach YAML

``` text
name
on
jobs
runs-on
steps
run
uses
with
```

## Then control flow

``` text
needs
if
matrix
```

## Then data/configuration

``` text
env
vars
secrets
contexts
outputs
artifacts
cache
```

## Then security

``` text
permissions
OIDC
environments
approvals
Action pinning
self-hosted runner security
```

## Then real-world deployment

``` text
GitHub
 ↓
CI
 ↓
Docker
 ↓
ACR
 ↓
Approval
 ↓
AKS
 ↓
Monitoring
```

## Final teaching principle

Do not teach students to memorize YAML.

Teach them to ask:

> **What triggers the pipeline? What needs to happen? What depends on
> what? What artifact is produced? Where does it go? What credentials
> are required? What should be protected?**

Once they can answer those questions, they can write a GitHub Actions
pipeline for almost any application.
