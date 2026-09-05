# AWS CodeCommit — Extra Topics & Advanced Features

## 1. CodeCommit + EventBridge

AWS CodeCommit can send repository events to **Amazon EventBridge**.

Examples of events include:

* Pull request created
* Pull request status changed
* New reference created
* Reference updated
* New comment created

Architecture:

```text
CodeCommit
    ↓ Event
EventBridge
    ↓
┌──────────┬──────────┬──────────────┐
│   SNS    │  Lambda  │ CodePipeline │
└──────────┴──────────┴──────────────┘
```

### Example

When a new Pull Request is created:

```text
Pull Request Created
        ↓
    EventBridge
        ↓
      Lambda
        ↓
Run automated validation
```

This provides powerful **event-driven automation**.

---

# 2. Migrating a Repository to CodeCommit

You can migrate an existing Git repository from platforms such as:

* GitHub
* GitLab
* Bitbucket

The basic idea is:

```text
Existing Git Repository
        ↓ git clone
Local Machine
        ↓ git push
AWS CodeCommit
```

## Step 1: Clone the Existing Repository

```bash
git clone <existing-repository-url>
```

This downloads the repository locally, including:

* Source code
* Commit history
* Git metadata
* Branches (depending on how the migration is performed)

## Step 2: Create a CodeCommit Repository

Create an empty repository in AWS CodeCommit.

## Step 3: Push to CodeCommit

You can configure CodeCommit as a remote and push your repository there.

Conceptually:

```text
GitHub/GitLab
      ↓
   git clone
      ↓
Local Repository
      ↓
   git push
      ↓
CodeCommit
```

### Key Idea

Git repositories are portable.

As long as you have the Git repository locally, you can configure another remote repository and push the code there.

---

# 3. Cross-Region Replication for CodeCommit

You may want cross-region replication for:

* Disaster recovery / backup
* Global development teams
* Lower latency
* High availability

Example:

```text
Source Region: us-east-1
        ↓
Replication
        ↓
Target Region: eu-west-2
```

## How can this be achieved?

CodeCommit can generate events such as:

* `referenceCreated`
* `referenceUpdated`

These events can be captured by **EventBridge**.

Example architecture:

```text
CodeCommit (us-east-1)
        │
        │ Push / Branch Change
        ▼
EventBridge
        │
        ▼
ECS Task / CodeBuild
        │
        │ git clone + git push
        ▼
CodeCommit (eu-west-2)
```

The ECS task or CodeBuild job can:

1. Clone the source repository.
2. Connect to the target CodeCommit repository.
3. Push the repository changes to the target region.

### Exam Concept

There is no need to memorize a specific implementation tool.

The important concept is:

> **CodeCommit events → EventBridge → Compute service → Replicate Git repository to another region**

---

# 4. Branch Security in CodeCommit

Suppose you have three branches:

```text
development
staging
production
```

You want:

* Junior developers → Push only to development.
* Senior developers → Push to production.

The challenge is that general push access could allow a user to push to multiple branches.

To control this, use **IAM policies**.

---

## Example Concept

Attach a policy to junior developers that explicitly denies pushing to:

```text
main
prod
production
```

The policy concept is:

```text
Allow normal CodeCommit access

BUT

Explicitly DENY pushes to production branches
```

### Important IAM Rule

```text
Explicit DENY
      ↓
Overrides
      ↓
ALLOW
```

Therefore, even if another policy allows repository access, an explicit deny can prevent pushes to protected branches.

---

# 5. Branch Security Uses IAM Policies

Branch-level security is implemented using **IAM policies attached to**:

* IAM users
* IAM groups
* IAM roles

Conceptually:

```text
Junior Developers Group
        │
        ▼
IAM Policy
        │
        ├── Allow → dev
        ├── Allow → staging
        └── Deny  → production
```

The lecture's key point is that branch restrictions are controlled through IAM rather than attaching a resource policy directly to the repository.

---

# 6. Pull Request Approval Rules

A Pull Request represents a proposed change that needs to be merged into another branch.

You can create **approval rules** to ensure the code is reviewed before merging.

Example:

```text
Developer creates PR
        ↓
Code Review Required
        ↓
Required Approvals Received?
        │
    ┌───┴────┐
    │        │
   No       Yes
    │        │
Cannot     Merge PR
 Merge
```

---

## Example: 2 Approvals Required

Suppose there are 5 users who are allowed to approve:

```text
User 1
User 2
User 3
User 4
User 5
```

The rule requires:

```text
Required approvals = 2
```

Then:

```text
User 1 approves ✅
User 3 approves ✅
```

Result:

```text
2 approvals received
        ↓
PR satisfies approval requirement
        ↓
Can be merged
```

---

# 7. Who Can Approve a Pull Request?

Approval rules can specify a pool of IAM principals, such as:

* IAM Users
* IAM Roles
* IAM Groups
* Federated Users

These are identified using their **ARNs**.

Conceptually:

```text
Approval Pool
     │
     ├── User A
     ├── User B
     ├── IAM Role C
     └── Federated User D
```

Then you specify how many approvals are required.

---

# 8. Approval Rule Templates

Instead of manually creating approval rules for every Pull Request, you can use an **Approval Rule Template**.

For example:

```text
Any PR targeting dev
        ↓
Require 1 approval
```

```text
Any PR targeting production
        ↓
Require 2 senior developer approvals
```

Architecture:

```text
New Pull Request
       ↓
Approval Rule Template Applied Automatically
       ↓
Required Reviewers Approve
       ↓
Pull Request Can Be Merged
```

---

# 9. EventBridge vs Notification Rules vs Triggers

This is useful to understand:

| Feature                 | Main Purpose                            |
| ----------------------- | --------------------------------------- |
| **Notification Rules**  | Notify people/systems about events      |
| **CodeCommit Triggers** | Trigger SNS/Lambda on repository events |
| **EventBridge**         | Build flexible event-driven automation  |

### Easy way to remember

```text
Notification Rule → "Tell me"

Trigger → "Invoke something"

EventBridge → "React and automate workflows"
```

---

# 10. Complete Advanced Architecture

```text
Developer Pushes Code
        │
        ▼
AWS CodeCommit
        │
        ├──────────────► Pull Request
        │                     │
        │                     ▼
        │              Approval Rules
        │                     │
        │                     ▼
        │                   Merge
        │
        ▼
EventBridge
        │
        ├──► SNS → Notifications
        │
        ├──► Lambda → Automation
        │
        ├──► CodePipeline → CI/CD
        │
        └──► ECS / CodeBuild
                   │
                   ▼
          Cross-Region Replication
```

# Key Exam Takeaways 📝

* **CodeCommit emits events to EventBridge**.
* EventBridge can trigger services like **SNS, Lambda, and CodePipeline**.
* Git repositories can be migrated from **GitHub/GitLab → CodeCommit** using Git clone/push workflows.
* Cross-region replication can be implemented using **CodeCommit events + EventBridge + ECS/CodeBuild**.
* Branch-level security is controlled using **IAM policies**.
* Use **explicit Deny** to prevent users from pushing to protected branches.
* **Pull Request Approval Rules** enforce code reviews before merging.
* **Approval Rule Templates** automatically apply approval requirements to new Pull Requests.

### One-line summary:

```text
CodeCommit = Git + IAM Security + Pull Requests + Approvals + Event-Driven AWS Automation
```



# Montioring with Event Bridge
1. You can monitor CodeCommit Events in EventBridge (near real-time)
2. pullRequestCreated,pullRequestStatusChanged,referencCreated ... are all examples of EventBridges
<img width="629" height="339" alt="image" src="https://github.com/user-attachments/assets/22abb765-fd62-48b4-aa8d-53a8810d2ac7" />

<img width="589" height="340" alt="image" src="https://github.com/user-attachments/assets/ebfc468c-23a9-4a31-97aa-4db21f885b6a" />


# Cross-Region Replication
<img width="670" height="353" alt="image" src="https://github.com/user-attachments/assets/a6c29c52-2492-4e22-80af-fc4b65ad780a" />

<img width="653" height="358" alt="image" src="https://github.com/user-attachments/assets/35ad35fe-2214-4ae1-94a4-33e19319e906" />


# Code Commit - Branch Security

<ul>
  <li>By default a user who has push permissions to a CodeCommit repository can contribute to any branch</li>
  <li>Use IAM policies to restrict users to push or merge code to a specific branch</li>
  <li>Example: Only Senior developers can push to production branch</li>
</ul>

<img width="612" height="361" alt="image" src="https://github.com/user-attachments/assets/d22ee4f1-d42f-4b79-a433-3cce49c51bcf" />



# Pull request Approval
<img width="622" height="350" alt="image" src="https://github.com/user-attachments/assets/245a9189-a65d-4c28-b6ba-a6a477c3b263" />
