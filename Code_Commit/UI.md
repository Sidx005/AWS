# AWS CodeCommit — Notes

AWS CodeCommit is AWS's managed **Git-based source-code repository service**. It is used in the **Source** stage of a CI/CD pipeline.

> **Exam idea:** CodeCommit stores your source code; it does **not** build, test, or deploy your application.

---

## 1. CodeCommit Basic Flow

```text
Developer
    ↓
CodeCommit Repository
    ↓
Branches
    ↓
Commits
    ↓
Pull Request
    ↓
Merge into main/master
```

A CodeCommit repository can contain:

* Source code
* Multiple branches
* Commits
* Tags
* Pull requests

---

## 2. Creating a Repository

When creating a repository, you can provide:

* Repository name
* Description
* Tags

Example:

```text
Repository: my-nodejs-app
Branch: master
File: index.html
```

After committing a file, CodeCommit generates a **commit ID**.

The commit represents a specific version/change of the repository.

---

# 3. Connecting to CodeCommit

CodeCommit supports several ways to connect:

### HTTPS

```text
Git Client
    ↓ HTTPS
CodeCommit
```

### SSH

```text
Git Client
    ↓ SSH
CodeCommit
```

### HTTPS (Git credentials)

You can configure Git credentials for an IAM user and use HTTPS authentication.

### Important exam point

The lecture highlights that **SSH access depends on using an IAM user rather than the root account**.

More generally, avoid using the AWS root user for day-to-day development or Git access; use appropriately permissioned IAM identities instead.

---

# 4. Commits

A **commit** represents a set of changes to the repository.

Example:

```text
Commit 1
   ↓
index.html

Commit 2
   ↓
index.html modified

Commit 3
   ↓
CSS added
```

Each commit has a unique identifier.

You can use CodeCommit to:

* View commits
* Browse repository contents at a particular commit
* Visualize commit history
* Compare commits

---

# 5. Branches

A branch allows developers to work on different versions of the code independently.

Example:

```text
              master
                 │
        ┌────────┴────────┐
        ↓                 ↓
      dev                test
```

Typical workflow:

```text
master
   ↓
dev branch
   ↓
Developer changes
   ↓
Pull Request
   ↓
master
```

The repository initially has a default branch. The lecture uses `master`.

### Exam point

**Branch = independent line of development.**

---

# 6. Pull Requests

A **pull request (PR)** is used to propose merging changes from one branch into another.

Example:

```text
dev branch
    ↓
Changes
    ↓
Pull Request
    ↓
Review / Approval
    ↓
master
```

Pull requests help with:

* Code review
* Collaboration
* Reviewing changes before merging
* Controlled merging

### Remember

> **Pull Request ≠ deployment**

It is primarily about **reviewing and merging code changes**.

---

# 7. Compare Commits

CodeCommit allows you to compare different commits/branches.

For example:

```text
master
  vs
dev
```

This lets you identify what changed between versions.

---

# 8. Tags

Repository tags can be used to identify important versions/releases.

Example:

```text
v1.0
v1.1
v2.0
```

Conceptually:

```text
Commit ───────→ v1.0
Commit ───────→ v1.1
Commit ───────→ v2.0
```

Tags are useful for identifying particular releases or versions.

---

# 9. CodeCommit Notifications

CodeCommit supports **notification rules**.

These notify you when important repository events occur.

Examples include events involving:

* Commits
* Pull requests
* Approval status
* Branches
* Tags

Conceptually:

```text
CodeCommit Event
       ↓
Notification Rule
       ↓
SNS
       ↓
Notification
```

---

# 10. SNS Integration

Amazon SNS can be used as a target for CodeCommit notifications.

For example:

```text
Developer pushes code
        ↓
CodeCommit
        ↓
Notification Rule
        ↓
SNS Topic
        ↓
Subscribers / Notifications
```

This can be useful for sending alerts when repository events occur.

---

# 11. Notification Rule vs Trigger

This is an important distinction.

### Notification Rule

Used primarily for **notifications about repository events**.

```text
CodeCommit Event
       ↓
Notification Rule
       ↓
SNS / Chatbot
```

Examples:

* Pull request created
* Pull request merged
* Approval status changed
* Commit-related events
* Branch/tag changes

### Trigger

Used to **invoke an action/service when a repository event occurs**.

Example:

```text
Push to master
      ↓
CodeCommit Trigger
      ↓
SNS / Lambda
```

The lecture specifically demonstrates a trigger for:

> **Push to an existing branch**

---

# 12. CodeCommit Trigger

A trigger can react to repository events such as:

* Push to existing branch
* Branch created
* Tag created
* Branch deleted
* Tag deleted

Example:

```text
Push code to master
        ↓
CodeCommit Trigger
        ↓
SNS
```

Or:

```text
Push code
    ↓
CodeCommit Trigger
    ↓
Lambda Function
```

This becomes especially useful when integrating CodeCommit with other AWS services.

---

# 13. Notification Rule vs Trigger — Quick Comparison

| Feature               | Notification Rule                   | Trigger          |
| --------------------- | ----------------------------------- | ---------------- |
| Purpose               | Notify about events                 | Invoke an action |
| Example               | PR created                          | Push to branch   |
| SNS                   | ✅                                   | ✅                |
| Lambda                | Not the main target in this lecture | ✅                |
| Branch/tag events     | ✅                                   | ✅                |
| Used for alerts       | ✅                                   | Possible         |
| Used to invoke Lambda | ❌                                   | ✅                |

### Easy memory trick

> **Notification Rule = "Tell me something happened."**

> **Trigger = "Do something when it happens."**

---

# 14. CodeCommit in CI/CD

CodeCommit is only the **Source** component.

A complete AWS CI/CD pipeline could look like:

```text
               SOURCE
                  ↓
             CodeCommit
                  ↓
                BUILD
                  ↓
             CodeBuild
                  ↓
                 TEST
                  ↓
               DEPLOY
                  ↓
             CodeDeploy
                  ↓
          EC2 / Lambda / ECS
```

And:

```text
              CodePipeline
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
      Source      Build      Deploy
        ↓          ↓          ↓
   CodeCommit  CodeBuild  CodeDeploy
```

---

# ⭐ Exam Cheat Sheet

| Term                  | Remember                                   |
| --------------------- | ------------------------------------------ |
| **CodeCommit**        | Source-code repository                     |
| **Commit**            | Recorded change/version                    |
| **Branch**            | Independent development line               |
| **Pull Request**      | Review/propose merging changes             |
| **Tag**               | Identify a specific version/release        |
| **Notification Rule** | Notify when repository events occur        |
| **Trigger**           | Invoke a target based on repository events |
| **SNS**               | Messaging/notification target              |
| **Lambda**            | Can be invoked by CodeCommit triggers      |
| **CodeBuild**         | Build + test                               |
| **CodeDeploy**        | Deploy                                     |
| **CodePipeline**      | Orchestrate the pipeline                   |

### The key chain to memorize

> **CodeCommit → CodeBuild → CodeDeploy → CodePipeline**

More precisely:

> **CodeCommit = Source → CodeBuild = Build/Test → CodeDeploy = Deploy → CodePipeline = Orchestrate**


# Demo:
<img width="1134" height="619" alt="image" src="https://github.com/user-attachments/assets/c48daf9e-5e22-4328-a22f-e098547989c3" />


<img width="725" height="415" alt="image" src="https://github.com/user-attachments/assets/dd243f55-9d49-4f0a-a409-23522dc06c7a" />


<img width="562" height="397" alt="image" src="https://github.com/user-attachments/assets/2d8928fe-1331-4632-af1e-c31a67b567c8" />


 <img width="701" height="410" alt="image" src="https://github.com/user-attachments/assets/2280bfee-0d8d-4a6e-a5c2-ee7ea9a7e379" />



<img width="644" height="412" alt="image" src="https://github.com/user-attachments/assets/0fd2869d-38b1-4cc0-8531-12611b6abf94" />


<img width="655" height="416" alt="image" src="https://github.com/user-attachments/assets/7ad5d0b4-88e6-4ffb-9510-53c214299836" />


<img width="622" height="379" alt="image" src="https://github.com/user-attachments/assets/25765ba8-74a5-45d4-af53-53e41a670c8d" />


<img width="689" height="341" alt="image" src="https://github.com/user-attachments/assets/a6823ad8-c2ed-4e97-a1e2-786b72aaa61f" />




<img width="695" height="545" alt="image" src="https://github.com/user-attachments/assets/d9bea125-d982-4042-9981-c4bacd762b24" />



<img width="720" height="416" alt="image" src="https://github.com/user-attachments/assets/622a4c6f-9924-4f79-9663-243a7947a542" />


<img width="703" height="442" alt="image" src="https://github.com/user-attachments/assets/b9276e44-8029-44a6-ac5f-989edaa32bd6" />


<img width="441" height="355" alt="image" src="https://github.com/user-attachments/assets/80ec7b07-78f8-47cd-82bc-38213b467723" />



<img width="674" height="476" alt="image" src="https://github.com/user-attachments/assets/0bd0c9d9-5bb9-4232-93ed-2b1c03b669b7" />



<img width="743" height="467" alt="image" src="https://github.com/user-attachments/assets/97d63abf-3f46-499f-9983-81ba3cb497c4" />
