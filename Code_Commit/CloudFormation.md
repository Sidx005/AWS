# AWS CodePipeline + CloudFormation — Notes

## 1. CloudFormation as a CodePipeline Deploy Target

**AWS CloudFormation** can be used as a **Deploy action** inside CodePipeline.

It allows your pipeline to deploy AWS infrastructure such as:

* EC2 instances
* Auto Scaling Groups
* Application Load Balancers (ALB)
* Lambda functions
* IAM roles
* VPC resources
* Databases
* Other AWS services

Infrastructure can be defined using:

* **CloudFormation templates**
* **AWS CDK** (which synthesizes CloudFormation templates)
* **AWS SAM** for serverless applications

---

# 2. Basic Pipeline Flow

A typical architecture:

```text
CodeCommit
    ↓
CodePipeline
    ↓
CodeBuild
    ↓
CloudFormation
    ↓
AWS Infrastructure
```

Example:

```text
Source
(CodeCommit)
      ↓
Build
(CodeBuild)
      ↓
Deploy Infrastructure
(CloudFormation)
```

The source code may contain:

```text
application code
+
CloudFormation template.yaml
```

---

# 3. Using Change Sets

A **Change Set** allows you to preview infrastructure changes before applying them.

### Flow:

```text
CodeCommit
    ↓
CodePipeline
    ↓
Create Change Set
    ↓
Manual Approval
    ↓
Execute Change Set
```

### Why use it?

Before modifying production infrastructure, you can see:

* Which resources will be created
* Which resources will be modified
* Which resources will be deleted
* Whether the changes look correct

### Example

Current infrastructure:

```text
EC2 Instance
ALB
Auto Scaling Group
```

New template changes:

```text
EC2 instance type:
t3.medium → t3.large
```

CloudFormation creates a Change Set showing the planned modification.

Then:

```text
Manual Approval
        ↓
Approved?
   ↙         ↘
 Yes          No
  ↓            ↓
Execute       Stop Pipeline
```

---

# 4. Deploying Without Manual Approval

If you don't need to review infrastructure changes manually, CodePipeline can directly deploy the stack.

```text
CodeCommit
    ↓
CodePipeline
    ↓
CloudFormation
CREATE_UPDATE
    ↓
Infrastructure Updated
```

This is useful for:

* Development environments
* Testing environments
* Automated deployments

---

# 5. CloudFormation Action Modes

CloudFormation provides different action modes in CodePipeline.

## A. Change Set Modes

### `CHANGE_SET_REPLACE`

Creates a new Change Set or replaces an existing one.

```text
Pipeline
    ↓
CloudFormation
    ↓
Create/Replace Change Set
```

Usually followed by:

```text
Manual Approval
    ↓
CHANGE_SET_EXECUTE
```

---

### `CHANGE_SET_EXECUTE`

Executes the previously created Change Set.

```text
Change Set
    ↓
Execute
    ↓
Infrastructure Updated
```

---

## B. Stack Modes

### `CREATE_UPDATE`

Creates a new stack if it doesn't exist.

Updates the stack if it already exists.

```text
Stack doesn't exist
        ↓
Create Stack

Stack exists
        ↓
Update Stack
```

This is probably the most commonly used mode.

---

### `DELETE_ONLY`

Deletes the CloudFormation stack.

```text
CloudFormation Stack
        ↓
DELETE_ONLY
        ↓
Infrastructure Deleted
```

Useful for temporary infrastructure.

---

### `REPLACE_ON_FAILURE`

If a stack is in a failed state, CloudFormation replaces it.

Useful when deployments fail and the existing stack cannot be updated normally.

---

# 6. Example: Temporary Test Infrastructure

One powerful use case is creating infrastructure temporarily for testing.

```text
                 CodePipeline
                       │
                       ▼
                  CodeBuild
                       │
                       ▼
             Build Application
                       │
                       ▼
              Create Test Stack
             CloudFormation
                       │
                       ▼
              Test Environment
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
        ALB                 EC2 Instances
             │                   │
             └─────────┬─────────┘
                       │
                       ▼
                 Run Tests
                 CodeBuild
```

---

# 7. Complete Example Pipeline

```text
┌──────────────────┐
│    CodeCommit    │
│  Source Code     │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    CodeBuild     │
│ Build Application│
│ Generate Template│
└────────┬─────────┘
         │
         ▼
┌────────────────────────┐
│   CloudFormation       │
│   CREATE_UPDATE        │
│                        │
│ Create Test Environment│
└────────────┬───────────┘
             │
             ▼
┌────────────────────────┐
│ Test Infrastructure    │
│                        │
│ ALB                    │
│ Auto Scaling Group     │
│ EC2 Instances          │
└────────────┬───────────┘
             │
             ▼
┌──────────────────┐
│    CodeBuild     │
│ Run Tests        │
│                  │
│ Functional Tests │
│ Load Tests       │
│ HTTP Tests       │
└────────┬─────────┘
         │
         ▼
       Success?
      /        \
    Yes         No
     │           │
     ▼           ▼
 Delete       Stop Pipeline
 Test Stack
     │
     ▼
Deploy Production
```

---

# 8. Testing the Application

After deploying the test infrastructure, another **CodeBuild stage** can test the application.

For example:

```text
CodeBuild
    ↓
Send HTTP Request
    ↓
ALB
    ↓
Application
```

Example test:

```text
GET http://test-alb-url/health
```

Expected:

```text
HTTP 200 OK
```

You could perform:

* Functional testing
* Integration testing
* Load testing
* API testing
* Smoke testing

If tests fail:

```text
❌ Pipeline fails
```

If tests succeed:

```text
✅ Continue deployment
```

---

# 9. Deleting Test Infrastructure

Once testing is complete:

```text
CloudFormation
DELETE_ONLY
```

The temporary test stack is destroyed.

```text
Test Environment
      ↓
Tests Complete
      ↓
DELETE_ONLY
      ↓
Infrastructure Removed
```

### Benefits

* Saves money 💰
* No unused infrastructure
* Clean environment
* Fresh infrastructure for future tests

---

# 10. Deploying Production Infrastructure

After successful testing:

```text
CloudFormation
CREATE_UPDATE
```

deploys or updates production.

```text
Tests Successful
       ↓
CloudFormation
       ↓
CREATE_UPDATE
       ↓
Production Infrastructure
```

---

# 11. CloudFormation StackSets

Normally, a CloudFormation stack deploys into:

```text
One AWS Account
+
One AWS Region
```

But **StackSets** allow deployment across:

```text
Multiple AWS Accounts
+
Multiple AWS Regions
```

Example:

```text
                 StackSet
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼

   Account A     Account B     Account C
   us-east-1     eu-west-1    ap-south-1
```

Useful for organizations with multiple AWS accounts.

Example:

```text
Development Account
Production Account
Security Account
Analytics Account
```

You can deploy the same infrastructure configuration across all of them.

---

# 12. CloudFormation Configuration in CodePipeline

When configuring a CloudFormation action, you can specify:

### Stack Name

```text
my-production-stack
```

---

### Change Set Name

```text
production-update-v2
```

---

### Template File

Example:

```text
template.yaml
```

or:

```text
infrastructure/template.yaml
```

---

### IAM Role

CloudFormation needs permissions to create resources.

```text
CodePipeline
       │
       ▼
CloudFormation Service Role
       │
       ▼
Creates AWS Resources
```

Example resources:

```text
EC2
ALB
IAM Roles
Lambda
S3
```

---

### Action Mode

Examples:

```text
CREATE_UPDATE
CHANGE_SET_REPLACE
CHANGE_SET_EXECUTE
DELETE_ONLY
REPLACE_ON_FAILURE
```

---

# 13. Parameter Overrides

CloudFormation templates can have parameters.

Example:

```yaml
Parameters:
  Environment:
    Type: String

  InstanceType:
    Type: String
```

Normally:

```text
Environment = dev
InstanceType = t3.micro
```

But CodePipeline can override these values during deployment.

For example:

```json
{
  "Environment": "production",
  "InstanceType": "t3.large"
}
```

So the same template can be reused:

```text
Same CloudFormation Template
            │
     ┌──────┼──────┐
     ▼      ▼      ▼
    DEV    TEST   PROD
```

With different parameters:

```text
DEV
Environment = dev
InstanceType = t3.micro

TEST
Environment = test
InstanceType = t3.medium

PROD
Environment = prod
InstanceType = t3.large
```

---

# 14. Static vs Dynamic Parameter Overrides

Parameter values can be provided through:

### Static values

Hardcoded configuration:

```json
{
  "Environment": "prod"
}
```

### Template configuration file

A separate configuration file contains parameter values.

### Dynamic values

Values generated during the pipeline.

For example:

```text
Build ID
Commit ID
Image Tag
Environment
```

These can be passed into CloudFormation at deployment time.

---

# 15. Two Important Deployment Patterns

## Pattern 1: Change Set + Manual Approval

Best for production infrastructure.

```text
Source
  ↓
Build
  ↓
Create Change Set
  ↓
Manual Approval
  ↓
Execute Change Set
```

### Use when:

* Production changes are sensitive
* Infrastructure changes need review
* You want controlled deployments

---

## Pattern 2: Direct Deployment

```text
Source
  ↓
Build
  ↓
CREATE_UPDATE
```

### Use when:

* Development
* Testing
* Fully automated deployments

---

# 🎯 Key Exam Takeaways

### CloudFormation in CodePipeline is a:

> **Deploy action provider**

---

### `CREATE_UPDATE`

> Creates a stack if it doesn't exist; updates it if it exists.

---

### `CHANGE_SET_REPLACE`

> Creates or replaces a Change Set.

---

### `CHANGE_SET_EXECUTE`

> Executes an existing Change Set.

---

### `DELETE_ONLY`

> Deletes the CloudFormation stack.

---

### Change Sets are useful when:

> You want to review infrastructure changes before applying them.

---

### StackSets are useful when:

> You need to deploy infrastructure across multiple AWS accounts and/or Regions.

---

# 🧠 Easy Way to Remember

```text
Want to PREVIEW changes?
        ↓
CHANGE SET

Want HUMAN review?
        ↓
Manual Approval

Want automatic deployment?
        ↓
CREATE_UPDATE

Want to REMOVE infrastructure?
        ↓
DELETE_ONLY

Want multi-account / multi-region?
        ↓
STACKSETS
```

## Final Summary

```text
CodePipeline
     │
     ├── Source → CodeCommit
     │
     ├── Build → CodeBuild
     │
     ├── Deploy Test → CloudFormation CREATE_UPDATE
     │
     ├── Test → CodeBuild
     │
     ├── Cleanup → CloudFormation DELETE_ONLY
     │
     └── Deploy Prod → CloudFormation CREATE_UPDATE
```

This pattern is especially useful because **CloudFormation lets your CI/CD pipeline create, test, destroy, and deploy entire infrastructure environments automatically**.
