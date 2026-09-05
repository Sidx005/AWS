# AWS CodePipeline — Complete Hands-On Notes 🚀

## 1. What is AWS CodePipeline?

**AWS CodePipeline** is a fully managed **CI/CD service** that automates the process of:

* Getting source code
* Building the application
* Testing it
* Deploying it

A pipeline consists of **stages**, and each stage contains **actions**.

```text
Source → Build → Test → Deploy
```

In this hands-on:

```text
GitHub
   ↓
CodePipeline
   ↓
Elastic Beanstalk (Dev)
   ↓
Manual Approval
   ↓
Elastic Beanstalk (Prod)
```

---

# 2. Creating the Pipeline

Pipeline name:

```text
MyFirstPipeline
```

### Execution mode

The lecture uses:

```text
Queued
```

This means pipeline executions are handled in a queue when multiple executions occur.

---

# 3. CodePipeline Service Role

When creating the pipeline, AWS creates a **service role**.

```text
CodePipeline
      │
      │ Assumes IAM Role
      ▼
Permissions to access AWS services
```

The service role allows CodePipeline to interact with services such as:

* GitHub connection
* S3 artifact storage
* CodeBuild
* Elastic Beanstalk

> Without the necessary IAM permissions, CodePipeline cannot perform its actions.

---

# 4. Source Providers

CodePipeline supports many source providers, including:

* CodeCommit
* Amazon ECR
* Amazon S3
* Bitbucket
* GitHub
* GitHub Enterprise Server
* GitLab
* GitLab Self-Managed

For this demo:

```text
Source Provider: GitHub Version 2
```

---

# 5. Connecting GitHub to AWS

Create a connection:

```text
MyGitHubConnection
```

AWS uses a GitHub connection to securely connect CodePipeline with your GitHub repositories.

The process involves:

```text
AWS CodePipeline
        ↓
GitHub Connection
        ↓
Authorize AWS Connector
        ↓
Install/Authorize GitHub App
        ↓
Connection Available ✅
```

Once the connection status becomes:

```text
Available
```

it can be selected as the source connection in CodePipeline.

---

# 6. Configure the Source Stage

Repository:

```text
my-nodejs-app
```

Branch:

```text
main
```

So the pipeline watches:

```text
GitHub Repository
my-nodejs-app
        │
        │ Push to main
        ▼
CodePipeline
```

---

# 7. Pipeline Trigger

The pipeline is configured to trigger when code is pushed to:

```text
main
```

So:

```text
Developer pushes code
        ↓
GitHub main branch
        ↓
GitHub Event
        ↓
CodePipeline Automatically Starts
```

This is the beginning of automation in CI/CD.

---

# 8. Output Artifact

The source stage produces an **artifact**.

```text
GitHub Source Code
        ↓
Source Artifact
        ↓
Used by later stages
```

The lecture uses the default CodePipeline artifact format.

This artifact can later be passed to:

* CodeBuild
* CodeDeploy
* Elastic Beanstalk
* Other pipeline actions

---

# 9. Build Stage

CodePipeline supports build providers such as:

* **AWS CodeBuild**
* **Jenkins**

For this hands-on, the build stage is skipped.

So the pipeline initially looks like:

```text
GitHub
   ↓
Source
   ↓
Deploy
```

Later, in a real application, you might have:

```text
GitHub
   ↓
CodeBuild
   ↓
Tests
   ↓
Deploy
```

---

# 10. Deploying to Elastic Beanstalk

Deployment provider:

```text
AWS Elastic Beanstalk
```

The application selected is the Elastic Beanstalk application created previously.

The first deployment environment is:

```text
env
```

So the pipeline becomes:

```text
GitHub
   ↓
CodePipeline
   ↓
Elastic Beanstalk
   ↓
env Environment
```

---

# 11. CodePipeline IAM Permission Issue

Sometimes the CodePipeline service role may not have enough permissions to deploy to Elastic Beanstalk.

For the demo, the instructor attaches:

```text
AdministratorAccess-AWSElasticBeanstalk
```

to the CodePipeline service role.

This allows CodePipeline to deploy to Elastic Beanstalk.

⚠️ **Real-world best practice:** Prefer least-privilege IAM permissions instead of administrator-level permissions.

---

# 12. First Automatic Deployment

The GitHub repository initially contains an `index.html` file with:

```text
Background color = Blue
```

The pipeline flow is:

```text
GitHub
   │
   │ Source pulled
   ▼
CodePipeline
   │
   │ Deploy
   ▼
Elastic Beanstalk (env)
```

After deployment:

```text
Website Background = Blue 🔵
```

This confirms that CodePipeline successfully deployed the GitHub code to Elastic Beanstalk.

---

# 13. Adding a Production Deployment Stage

The pipeline can be edited to add more stages.

A new stage is created:

```text
DeployToProd
```

Inside this stage, an action deploys to:

```text
Environment: prod
```

The deployment action uses:

```text
Input Artifact: SourceArtifact
```

So the same source artifact is reused.

Pipeline structure:

```text
Source
   ↓
Deploy to env
   ↓
DeployToProd
   ↓
Deploy to prod
```

---

# 14. Manual Approval Before Production

Before deploying to production, a **Manual Approval** action is added.

The production stage now becomes:

```text
DeployToProd Stage
│
├── Manual Approval
│
└── Deploy to Production
```

Conceptually:

```text
Deploy to Dev
      ↓
Manual Approval Required
      ↓
Human Reviews Application
      ↓
Approve?
   ┌──────┐
   │ YES  │
   ▼
Deploy to Production
```

This is a very common real-world CI/CD pattern.

---

# 15. Changing the Application

The instructor changes:

```text
Background: Blue 🔵
```

to:

```text
Background: Red 🔴
```

Then commits the change to the `main` branch.

```text
GitHub
    ↓
New Commit
    ↓
Push to main
    ↓
CodePipeline Automatically Triggered
```

---

# 16. Deployment to Development Environment

The pipeline first deploys the updated application to:

```text
env
```

After the deployment:

```text
Development Environment = Red 🔴
```

The pipeline then moves to the production stage.

---

# 17. Pipeline Stops for Manual Approval

At this point:

```text
DeployToProd
      ↓
Manual Approval ⏸️
```

The pipeline waits.

Nothing is deployed to production until someone approves it.

The reviewer checks the development environment and confirms:

```text
Looks good in red! ✅
```

Then approves the pipeline.

---

# 18. Deployment to Production

After approval:

```text
Manual Approval
       ↓ Approved
Deploy to Prod
       ↓
Elastic Beanstalk Production
```

The production environment changes:

```text
Old Version = Green 🟢
```

to:

```text
New Version = Red 🔴
```

---

# 19. Final Pipeline Architecture

```text
┌──────────────────────┐
│       GitHub         │
│    Push to main      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    Source Stage      │
│   SourceArtifact     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Deploy Stage       │
│ Elastic Beanstalk    │
│       env            │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   DeployToProd       │
│                      │
│  Manual Approval ⏸️ │
│          ↓           │
│ Deploy to prod 🚀    │
└──────────────────────┘
```

---

# 20. Important Concept: Stages vs Actions

This is important for the exam.

### Pipeline

Contains:

```text
Stages
```

Each stage contains:

```text
Actions
```

Example:

```text
Pipeline
│
├── Source Stage
│      └── Get GitHub Code
│
├── Deploy Dev Stage
│      └── Deploy to Elastic Beanstalk
│
└── Deploy Prod Stage
       ├── Manual Approval
       └── Deploy to Production
```

---

# 21. Pipeline History

CodePipeline provides a **Pipeline History** section.

You can see:

* Previous executions
* Execution status
* Duration
* Associated commits
* Success/failure information

Example:

```text
Execution #1 → Succeeded ✅
Execution #2 → Succeeded ✅
Execution #3 → Failed ❌
```

---

# Key Exam Takeaways 📝

### CodePipeline

> Automates CI/CD workflows using stages and actions.

### Pipeline structure

```text
Pipeline
   → Stages
       → Actions
```

### Source providers

CodePipeline can integrate with:

* GitHub
* CodeCommit
* GitLab
* Bitbucket
* S3
* ECR

### Manual Approval

Use **Manual Approval** when human validation is required before continuing.

```text
Dev Deployment
      ↓
Manual Approval
      ↓
Production Deployment
```

### Artifacts

Artifacts are outputs from one stage that can be passed to later stages.

```text
Source Code
    ↓
SourceArtifact
    ↓
Build / Deploy Stage
```

---

# One-Line Summary

```text
GitHub Push → CodePipeline → Deploy to Dev → Manual Approval → Deploy to Production
```

This is a classic **CI/CD pipeline workflow** and is the main concept demonstrated in this hands-on.


<img width="631" height="314" alt="image" src="https://github.com/user-attachments/assets/1e9b16b2-875c-4234-b129-46eb2cfc9fb3" />


<img width="715" height="397" alt="image" src="https://github.com/user-attachments/assets/ec0acfb4-0087-48b8-ba78-607bc9af5e8a" />


<img width="582" height="356" alt="image" src="https://github.com/user-attachments/assets/d1a87e58-f29a-4043-9c71-15f683d2abb7" />

# Source providers (Recommended: Github v2)
<img width="646" height="326" alt="image" src="https://github.com/user-attachments/assets/50d62d8b-f17e-4903-b01f-2f65d0d6f0e7" />


<img width="646" height="326" alt="image" src="https://github.com/user-attachments/assets/68f7fc6e-3a38-40c5-aa3f-a7c97d1f88de" />


<img width="699" height="352" alt="image" src="https://github.com/user-attachments/assets/c5a5c65e-dea9-43fa-bfc8-19ff410deabe" />


<img width="700" height="427" alt="image" src="https://github.com/user-attachments/assets/83ce393a-a60e-4d55-a173-d9a9a96cf5bf" />


<img width="715" height="367" alt="image" src="https://github.com/user-attachments/assets/aff4062e-57ae-4e47-b09c-02f5393d0df8" />

<img width="674" height="339" alt="image" src="https://github.com/user-attachments/assets/d795e07c-880f-40e8-8ac3-0a3f180cc032" />

<img width="663" height="415" alt="image" src="https://github.com/user-attachments/assets/c12c4edb-bc4b-459e-97c8-48af0708d6ac" />
