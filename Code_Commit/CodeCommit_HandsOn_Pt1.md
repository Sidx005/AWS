# AWS CodeCommit — Notes

## 1. What is AWS CodeCommit?

**AWS CodeCommit** is a fully managed **Git-based source control service** from AWS.

It allows you to:

* Store source code securely.
* Create and manage Git repositories.
* Track code changes using commits.
* Create branches.
* Collaborate using pull requests.
* Integrate with AWS CI/CD services.

> CodeCommit works similarly to GitHub/GitLab repositories, but it is integrated into the AWS ecosystem.

---

# 2. AWS Developer Tools

The AWS console groups several CI/CD services together:

* **CodeCommit** → Source code repository
* **CodeBuild** → Build and test code
* **CodeDeploy** → Deploy applications
* **CodePipeline** → Automate the complete CI/CD pipeline

Typical flow:

```text
Developer
    ↓
CodeCommit
    ↓
CodeBuild
    ↓
CodeDeploy
    ↓
Application Environment
```

**CodePipeline** can orchestrate the entire process.

---

# 3. Creating a CodeCommit Repository

Example repository name:

```text
my-nodejs-app
```

While creating a repository, you can optionally add:

* Description
* Tags

After creation, AWS provides connection options to interact with the repository.

---

# 4. Connecting to CodeCommit

CodeCommit supports multiple authentication/connection methods:

### HTTPS

Connect using Git credentials generated for an IAM user.

### SSH

Connect using SSH keys associated with an IAM user.

### HTTPS (Git credentials)

Use AWS-generated Git credentials for authentication.

> ⚠️ The lecture mentions that SSH options depend on using an IAM user rather than the root account.

---

# 5. Exploring the CodeCommit Repository

After creating the repository, you can explore the **Code** section.

You can:

* Browse files.
* Create files directly.
* Upload files.
* View file contents.
* Commit changes.

### Example

Upload:

```text
index.html
```

Then provide:

```text
Author: Stephane
Email: stephane@example.com
Commit message: First commit
```

Click:

```text
Commit changes
```

This creates the first Git commit in the repository.

---

# 6. Commits

A **commit** represents a saved snapshot of changes in Git.

Each commit has:

* Commit ID
* Author
* Commit message
* Timestamp
* Changes made

Example:

```text
Commit: First commit
```

You can:

* Browse the repository at a particular commit.
* View commit history.
* Compare commits.
* Visualize commit history.

---

# 7. Branches

Initially, the repository may have a default branch such as:

```text
master
```

You can create additional branches such as:

```text
dev
test
feature/login
feature/payment
```

Example branching structure:

```text
master
   │
   ├── dev
   │
   ├── test
   │
   └── feature-login
```

Branches allow multiple developers to work independently without directly affecting the main branch.

---

# 8. Pull Requests

A **Pull Request (PR)** is used to request merging changes from one branch into another.

Example:

```text
feature-login
      ↓
Pull Request
      ↓
master
```

Typical workflow:

```text
1. Create a branch
2. Make changes
3. Commit changes
4. Push changes
5. Create Pull Request
6. Review code
7. Approve
8. Merge into main branch
```

Pull requests help with:

* Code reviews
* Collaboration
* Preventing accidental changes to production code

---

# 9. Comparing Commits

CodeCommit allows you to compare:

* One commit with another commit.
* One branch with another branch.

This helps identify:

```text
What changed?
Which files changed?
What lines were added?
What lines were deleted?
```

---

# 10. Notifications

CodeCommit can generate notifications when events occur in the repository.

You can create a **Notification Rule**.

Example:

```text
DemoNotificationRule
```

Notification detail types:

* **Full** → More detailed information.
* **Basic** → Basic information.

---

## Events That Can Trigger Notifications

### Pull Request Events

Notifications can occur when:

* Pull request is created.
* Pull request source is updated.
* Pull request status changes.
* Pull request is merged.
* Approval status changes.

### Branch and Tag Events

Notifications can occur when:

* Branch is created.
* Branch is deleted.
* Branch is updated.
* Tag is created.
* Tag is deleted.

---

# 11. Notification Targets

Notification rules can send events to targets such as:

### Amazon SNS

```text
CodeCommit Event
       ↓
Amazon SNS
       ↓
Email / Other Subscribers
```

SNS can notify users through subscriptions such as email.

### AWS Chatbot / Slack

Notifications can also be integrated with communication tools such as Slack.

Example:

```text
Developer pushes code
        ↓
CodeCommit Event
        ↓
Notification Rule
        ↓
SNS / AWS Chatbot
        ↓
Slack or Email Notification
```

---

# 12. CodeCommit Triggers

A **trigger** is used to perform an action when specific repository events occur.

Example trigger:

```text
DemoTrigger
```

Possible events include:

* Push to an existing branch.
* Create branch.
* Create tag.
* Delete branch.
* Delete tag.

You can restrict a trigger to a specific branch.

Example:

```text
Branch: master
```

---

# 13. Trigger Targets

CodeCommit triggers can invoke services such as:

### Amazon SNS

```text
Code Push
    ↓
CodeCommit Trigger
    ↓
SNS
    ↓
Notification
```

### AWS Lambda

```text
Code Push
    ↓
CodeCommit Trigger
    ↓
Lambda Function
    ↓
Custom Processing
```

---

# 14. Notification Rules vs Triggers

This is an important distinction.

| Feature       | Notification Rules              | Triggers                                    |
| ------------- | ------------------------------- | ------------------------------------------- |
| Purpose       | Send notifications about events | Invoke an action based on repository events |
| Common target | SNS / Chatbot                   | SNS / Lambda                                |
| Typical use   | Notify developers               | Automate workflows                          |
| Example       | PR created → Send Slack message | Code pushed → Invoke Lambda                 |

### Simple way to remember:

**Notification Rule:**

> "Tell someone that something happened."

**Trigger:**

> "Do something because something happened."

---

# 15. Repository Tags

You can add **tags** to the CodeCommit repository.

Example:

```text
Environment = Development
Project = DataTrust
Team = DevOps
```

Tags are useful for:

* Cost management.
* Resource organization.
* Automation.
* Identifying environments/projects.

---

# 16. Important CodeCommit Concepts

```text
Repository
    ↓
Contains
    ↓
Files + Source Code
    ↓
Changes saved as
    ↓
Commits
    ↓
Commits exist on
    ↓
Branches
    ↓
Branches merged using
    ↓
Pull Requests
```

---

# 17. Complete Example Workflow

```text
1. Create CodeCommit Repository
        ↓
2. Clone Repository Locally
        ↓
3. Create / Modify Code
        ↓
4. git add .
        ↓
5. git commit -m "Added feature"
        ↓
6. git push
        ↓
7. Create Pull Request
        ↓
8. Code Review
        ↓
9. Merge into main branch
```

---

# Key Exam Takeaways

* **CodeCommit** = Managed Git repository service.
* Supports **repositories, commits, branches, and pull requests**.
* **Branches** allow parallel development.
* **Pull Requests** help merge and review code.
* **Notification Rules** notify users about repository events.
* **Triggers** can invoke services such as SNS or Lambda based on repository events.
* CodeCommit integrates with:

  * CodeBuild
  * CodeDeploy
  * CodePipeline
  * SNS
  * Lambda

### Easy memory trick

```text
CodeCommit → Store Code
CodeBuild → Build Code
CodeDeploy → Deploy Code
CodePipeline → Automate Everything
```


1. Create a repo
<img width="426" height="258" alt="image" src="https://github.com/user-attachments/assets/ca214343-341c-48ab-ba23-2f01d6451987" />


2. If you're using root  account IAM User then you can see the SSH tab else not.
<img width="589" height="373" alt="image" src="https://github.com/user-attachments/assets/11edf563-865d-4897-8d55-74bb79ec2b08" />

3. Click on the Code navlink in the left sidebar and it takes you inside the code repo
<img width="516" height="338" alt="image" src="https://github.com/user-attachments/assets/6de289b0-50a8-4262-9ddb-048252787d8e" />

4. click on upload file and choose any file(eg: html)
<img width="674" height="379" alt="image" src="https://github.com/user-attachments/assets/0c913112-53c3-4c96-82e5-95ad71ec2648" />

<img width="672" height="366" alt="image" src="https://github.com/user-attachments/assets/55990169-6549-4a22-a815-5a7d780bed14" />

5. View branches
<img width="613" height="333" alt="image" src="https://github.com/user-attachments/assets/e5b7da27-8835-4906-902b-57d8bb891e82" />


6. Pull Requests
<img width="689" height="312" alt="image" src="https://github.com/user-attachments/assets/6b9a94d3-d100-4c4b-b634-fa1eb8e2b00a" />



7. Settings > Notifications
<img width="648" height="378" alt="image" src="https://github.com/user-attachments/assets/d9c06283-423b-4acd-890e-9310b61adeb1" />
<img width="574" height="397" alt="image" src="https://github.com/user-attachments/assets/80c6bfd3-c824-4fdc-9e42-ae03beced7b0" />

Target:
<img width="700" height="412" alt="image" src="https://github.com/user-attachments/assets/3ca18c77-5a4d-4934-b868-f1f1bad25ca5" />


8. Triggers
<img width="668" height="329" alt="image" src="https://github.com/user-attachments/assets/875d15db-40a4-442f-a64d-4bf7f22156c3" />


<img width="655" height="337" alt="image" src="https://github.com/user-attachments/assets/27a45180-5d7c-406e-8a16-e581a48764f4" />
