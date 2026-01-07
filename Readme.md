   README: Fixing the “Never-Ending Deployment Loop” (DevOps Case Study)
Overview
The deployment failures at QuickServe — missing environment variables, port conflicts, and old containers running in production — all point to one core issue: an unreliable pipeline with no clean handoff between stages.

A stable CI/CD workflow requires a strict chain from:
Code → Container → Registry → Deployment → Runtime Environment
Any break in this chain leads to inconsistent builds and unpredictable production behavior.


1. What Was Going Wrong
1.1. Environment Variables Missing
Errors like “Environment variable not found” happen when:


Secrets are manually added on servers


CI/CD doesn’t inject them during deployment


Build and runtime environments don’t match


This makes deployments fragile and non-reproducible.


1.2. Port Already in Use
This occurs when:


Old containers are never stopped


New containers try to bind to the same port


No orchestration handles lifecycle or cleanup


Result: half-deployed versions and random failures.


1.3. Old Containers Still Running on AWS
Typical causes:


Manual docker runs on EC2


No rolling update or blue-green strategy


No version tagging for images


This creates environment drift — different servers running different versions.


1.4. Pipeline Failing Midway
If a pipeline succeeds sometimes and fails sometimes, it means:


The pipeline isn’t idempotent


Previous deployments leave junk behind


Environment is not isolated


A proper CI/CD system must start from a clean state every run.


2. How Proper Containerization Solves These Issues
2.1. Containers Should Contain ONLY the App + Dependencies
A correct Docker image:


Does NOT contain secrets


Does NOT contain environment-specific config


Produces identical output on any machine


This creates a predictable, reproducible build artifact.


2.2. Environment Variables Must Be Injected at Runtime
Correct Secret Management:


AWS SSM Parameter Store / Secrets Manager


Azure Key Vault


GitHub Actions Encrypted Secrets


Secrets should be attached:


at ECS task definition


or during container startup


not during docker build


This removes 90% of env-var errors.


3. CI/CD Pipeline: Fixing the Chain of Trust
A stable deployment pipeline must have clear, isolated stages:


Stage 1: Code → Build

Lint and test


Build Docker image


Tag using Git commit SHA:

quickserve-app:<sha>


Guarantees every build is traceable.


Stage 2: Build → Registry

Push tagged images to ECR/ACR


No “latest” tag


Only versioned images


Prevents mix-ups between old and new builds.


Stage 3: Registry → Deployment
Use a managed orchestrator:


AWS ECS + Fargate


Azure Web App for Containers


Kubernetes


It handles:


stopping old containers


starting new ones


port management


rollback if health checks fail


Removes port conflict + stale container problems.


Stage 4: Deployment → Runtime Environment
Attach:


environment variables


secrets


port mappings


health checks


at runtime, not at build time.


4. Final Deployment Workflow (Clean & Reliable)

Developer pushes code


CI builds Docker image


Image is tagged with commit SHA


Image is pushed to registry


Cloud service pulls that exact image


Secrets injected from SSM/Key Vault


Old containers gracefully shut down


New containers start on clean ports


Health checks validate the service


Traffic switches to the new version


No leftovers. No surprises. No inconsistent versions.


5. Result
By enforcing:


proper containerization


strict secret management


versioned deployment


orchestrated rollouts