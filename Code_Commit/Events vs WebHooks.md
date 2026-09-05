# AWS CodePipeline — Additional Concepts & Exam Notes 📝

## 1. How Can a CodePipeline Start?

CodePipeline can be triggered in three main ways:

1. **Events** ✅ Recommended
2. **Webhooks**
3. **Polling** ❌ Less efficient

---

# 2. Event-Based Triggering ⭐ Preferred

This is the **recommended and preferred method**.

The pipeline starts immediately when a relevant event occurs.

### Example: AWS CodeCommit

```text
Developer pushes code
        ↓
CodeCommit event
        ↓
Amazon EventBridge
        ↓
EventBridge Rule
        ↓
CodePipeline starts 🚀
```

For example, a new commit can generate an event that ultimately triggers the pipeline.

### Why is this preferred?

* Fast ⚡
* Event-driven
* No unnecessary checking
* Efficient
* Near real-time response

---

# 3. GitHub Event-Based Triggering

GitHub is external to AWS, so AWS uses a connection mechanism such as a **CodeStar Source Connection**.

```text
Developer pushes to GitHub
          ↓
GitHub Event
          ↓
CodeStar Source Connection
          ↓
AWS CodePipeline
          ↓
Pipeline starts 🚀
```

The important idea is:

> **GitHub changes can trigger CodePipeline in an event-driven way through an AWS connection.**

---

# 4. Webhooks

A webhook is another way to start a pipeline.

CodePipeline can expose an HTTP endpoint.

```text
External Script/Application
          │
          │ HTTP Request + Payload
          ▼
       Webhook
          │
          ▼
    CodePipeline
          │
          ▼
   Pipeline Starts
```

The external system sends a request to the webhook endpoint, which triggers the pipeline.

### Example

```text
Custom Application
       ↓
HTTP POST Request
       ↓
CodePipeline Webhook
       ↓
Pipeline Starts
```

The lecture describes this as an older triggering mechanism compared with modern event-driven integrations.

---

# 5. Polling

Polling means CodePipeline repeatedly checks the source.

```text
CodePipeline
     │
     ├── Any new changes? ❌
     │
     ├── Any new changes? ❌
     │
     ├── Any new changes? ❌
     │
     └── Any new changes? ✅
               ↓
        Start Pipeline
```

Example:

```text
CodePipeline → GitHub
CodePipeline → GitHub
CodePipeline → GitHub
CodePipeline → GitHub
```

### Why is polling less preferred?

* Repeated unnecessary requests
* Less efficient
* Changes may not be detected instantly

### Remember:

```text
Events = Preferred ✅
Webhooks = Possible
Polling = Less preferred ❌
```

---

# 6. CodePipeline Actions

A CodePipeline consists of:

```text
Pipeline
   ↓
Stages
   ↓
Actions
```

Each action has concepts such as:

* **Owner**
* **Action Type**
* **Provider**

---

# 7. Action Owners

There are three important action owners:

| Owner           | Meaning            |
| --------------- | ------------------ |
| **AWS**         | AWS service/action |
| **Third Party** | External provider  |
| **Custom**      | Custom integration |

### Examples

#### AWS

```text
AWS CodeBuild
AWS Lambda
Elastic Beanstalk
```

#### Third Party

```text
GitHub
Bitbucket
```

#### Custom

```text
Jenkins
Custom actions
```

### Easy Memory Trick 🧠

```text
AWS → AWS services
Third Party → External services
Custom → Your own/custom integration
```

---

# 8. Action Types

CodePipeline supports different action categories.

## Source

Examples:

* S3
* ECR
* GitHub
* CodeCommit

```text
Source
   ↓
Get Application Code
```

---

## Build

Examples:

* CodeBuild
* Jenkins

```text
Source Code
     ↓
Build
     ↓
Application Artifact
```

---

## Test

Examples:

* CodeBuild
* Device Farm
* Jenkins

```text
Application
     ↓
Testing
     ↓
Pass / Fail
```

---

## Approval ⭐ Important for Exam

The approval action type is:

```text
Owner: AWS
Action Type: Manual
```

This is important.

### Manual Approval

```text
Deploy to Dev
      ↓
Manual Approval ⏸️
      ↓
Human Approves
      ↓
Deploy to Production
```

---

## Invoke

Used to invoke services such as:

* Lambda
* Step Functions

```text
CodePipeline
      ↓
Invoke Action
      ↓
Lambda / Step Functions
```

---

## Deploy

Examples include:

* Amazon S3
* CloudFormation
* CodeDeploy
* Elastic Beanstalk
* ECS
* Service Catalog

```text
Pipeline Artifact
       ↓
Deploy Action
       ↓
AWS Environment
```

---

# 9. Manual Approval in Detail ⭐⭐⭐

A manual approval action pauses the pipeline.

```text
Pipeline
    ↓
Stage 1 ✅
    ↓
Stage 2 ✅
    ↓
Manual Approval ⏸️
    │
    ├── Approve ✅ → Continue
    │
    └── Reject ❌ → Pipeline does not proceed
```

You can configure an **SNS topic** to notify someone that approval is required.

```text
Manual Approval Required
          ↓
       SNS Topic
          ↓
         Email
          ↓
       IAM User
          ↓
Approve / Reject
```

---

# 10. IAM Permissions Required for Manual Approval ⭐ Exam Important

The user who approves the pipeline needs appropriate IAM permissions.

### 1. Permission to view/access the pipeline

```text
codepipeline:GetPipeline*
```

This allows the user to access pipeline information.

### 2. Permission to approve or reject

```text
codepipeline:PutApprovalResult
```

This allows the user to submit:

* Approve
* Reject

Conceptually:

```text
IAM User
   │
   ├── GetPipeline*          → View Pipeline
   │
   └── PutApprovalResult     → Approve / Reject
```

---

# 11. Manual Approval — Exam Answer

If an exam question asks:

> **What is the action type for a CodePipeline manual approval action?**

Remember:

```text
Owner: AWS
Action Type: Manual
```

If asked:

> **What permission allows an IAM user to approve a CodePipeline manual approval action?**

Remember:

```text
codepipeline:PutApprovalResult
```

---

# Final Summary 🧠

## Pipeline Trigger Methods

```text
🥇 Events → Recommended
🥈 Webhooks → HTTP-triggered
🥉 Polling → Repeated checking, less efficient
```

## Manual Approval

```text
Owner       → AWS
Action Type → Manual
```

## IAM Permissions

```text
View Pipeline       → codepipeline:GetPipeline*
Approve / Reject    → codepipeline:PutApprovalResult
```

### One-line exam memory trick:

> **Events start pipelines efficiently, and Manual Approval requires AWS/Manual action configuration plus `PutApprovalResult` permission.**


<img width="683" height="362" alt="image" src="https://github.com/user-attachments/assets/0bf45ddf-7420-4107-afe1-4c333e739260" />


### Note: Events are the default and recommended

<img width="688" height="389" alt="image" src="https://github.com/user-attachments/assets/c315eafd-1615-4d5e-bd79-14f0c145f225" />

<img width="715" height="386" alt="image" src="https://github.com/user-attachments/assets/8eb70acf-46d0-445e-ab9f-0d4531f98096" />


This diagram explains **how a Manual Approval stage works in AWS CodePipeline**, especially the **SNS notification flow and IAM permissions**.

## 1. The CodePipeline flow

At the top:

```text
CodeCommit
    ↓ new commit
CodeBuild
    ↓ trigger
Manual Approval ⏸️
    ↓ deploy (after approval)
CodeDeploy
```

So the normal CI/CD flow is:

1. Developer pushes a new commit to **CodeCommit**
2. **CodeBuild** builds/tests the application
3. Pipeline reaches **Manual Approval**
4. Pipeline pauses ⏸️
5. A human approves or rejects it
6. If approved → deployment continues to **CodeDeploy**

---

# 2. What happens when Manual Approval is reached?

The diagram shows:

```text
Manual Approval
      │
      │ trigger
      ▼
     SNS
      │
      │ email
      ▼
   IAM User
```

When CodePipeline reaches the Manual Approval action:

* It triggers an **SNS topic**
* SNS can send an **email notification**
* The responsible user receives the notification
* The user goes to AWS and approves/rejects the pipeline

Example:

```text
🚨 Email:
"Pipeline MyFirstPipeline is waiting for approval"
```

---

# 3. The IAM User approves the pipeline

The IAM user then interacts with the:

```text
Manual Approval Stage
```

The user can choose:

```text
Approve ✅
```

or

```text
Reject ❌
```

If approved:

```text
Manual Approval
       ↓
Pipeline continues
       ↓
CodeDeploy
       ↓
Application deployed 🚀
```

---

# 4. Important Exam Point ⭐

The diagram highlights:

```text
Owner = AWS
Action = Manual
```

This is how the Manual Approval action is categorized in CodePipeline.

### Remember:

| Property        | Value  |
| --------------- | ------ |
| **Owner**       | AWS    |
| **Action Type** | Manual |

So if an exam asks:

> What is the action type used for human approval in CodePipeline?

Answer:

```text
Manual
```

---

# 5. IAM Permissions Required

The right side of the image shows the IAM policy permissions required by the user.

### Permission 1: View the pipeline

```json
"Action": "codepipeline:GetPipeline*"
```

This allows the IAM user to access/view the pipeline.

The resource is restricted to something like:

```text
MyFirstPipeline
```

---

### Permission 2: Approve or Reject

```json
"Action": "codepipeline:PutApprovalResult"
```

This is the **most important permission**.

It allows the IAM user to submit an approval result:

```text
Approve ✅
```

or:

```text
Reject ❌
```

The permission is applied to the specific approval action:

```text
MyFirstPipeline/ApprovalStage/MyApprovalAction
```

---

# Complete Flow 🧠

```text
Developer
    │
    │ New Commit
    ▼
CodeCommit
    │
    ▼
CodeBuild
    │
    ▼
Manual Approval ⏸️
    │
    ├── Trigger SNS
    │       │
    │       ▼
    │     Email 📧
    │       │
    │       ▼
    │    IAM User 👤
    │
    │ User clicks Approve
    │
    ▼
CodeDeploy 🚀
```

## The easiest way to remember it

> **SNS tells the person that approval is needed, and the IAM user needs `codepipeline:PutApprovalResult` to approve or reject the pipeline.**

And for the exam:

```text
Manual Approval Action
Owner  → AWS
Action → Manual
```
