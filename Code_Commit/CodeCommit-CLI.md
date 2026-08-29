# AWS CodeCommit — Git Connection, Authorization & Commands

This lecture is mainly about **how your local/CloudShell Git repository connects to CodeCommit and how changes move from your machine to AWS**.

---

## 1. UI Walkthrough

In the CodeCommit repository, you can select **Clone URL / Connection steps**.

You will typically see connection methods such as:

* **HTTPS**
* **SSH**
* **HTTPS (GRC)**

The exact options available depend on how you authenticate to AWS.

### In this lecture

The instructor is using a **federated account**, so they use:

> **HTTPS (GRC)**

The important idea is:

```text
AWS CodeCommit
      ↓
Connection Steps
      ↓
Configure Git authentication
      ↓
git clone
```

---

# 2. Authorization / Authentication

There are two related concepts:

### Authentication

> **Who are you?**

For example, AWS needs to know which IAM identity is making the CodeCommit request.

### Authorization

> **What are you allowed to do?**

After AWS identifies you, IAM permissions determine whether you can:

* Clone
* Pull
* Push
* Create branches
* Create commits
* etc.

So:

```text
Git command
    ↓
Authentication
"Who are you?"
    ↓
IAM authorization
"What can you do?"
    ↓
CodeCommit
```

### Important

Git itself does **not** decide whether you are allowed to push to CodeCommit.

AWS IAM permissions ultimately control access to the CodeCommit repository.

---

# 3. CloudShell

AWS CloudShell is useful for this demonstration because it already provides common tools such as:

* Git
* Python
* AWS CLI

So instead of installing everything locally:

```text
Browser
   ↓
AWS CloudShell
   ↓
Git
   ↓
CodeCommit
```

---

# 4. Install `git-remote-codecommit`

For the HTTPS GRC approach shown in the lecture:

```bash
pip install git-remote-codecommit
```

This installs the Git remote helper needed to interact with CodeCommit using the GRC connection method.

---

# 5. Clone the Repository

The CodeCommit UI gives you the appropriate **Git clone command**.

Conceptually:

```bash
git clone <CodeCommit-clone-url>
```

This downloads the repository into your current environment.

Example:

```text
CloudShell
   │
   ├── my-nodejs-app/
   │      └── index.html
   │
   └── other files...
```

After cloning:

```bash
cd my-nodejs-app
```

You are now inside the local Git repository.

---

# 6. Understanding Local vs Remote Repository

This is one of the **most important concepts** from the lecture.

There are two repositories:

```text
LOCAL                          REMOTE
CloudShell                     CodeCommit
──────────                     ─────────
my-nodejs-app                  my-nodejs-app
index.html                     index.html
app.js                         app.js
package.json                   package.json
```

When you run:

```bash
git commit
```

the commit is created **locally**.

It does **NOT** automatically go to CodeCommit.

You need:

```bash
git push
```

to send it to the remote CodeCommit repository.

### Remember:

> **commit = local**

> **push = remote**

---

# 7. Uploading Files to CloudShell

The lecture uploads files from the computer into CloudShell:

```text
Computer
   ↓
CloudShell
   ↓
my-nodejs-app/
```

Files include:

```text
app.js
cron.yaml
package.json
```

Then:

```bash
ls
```

shows:

```text
app.js
cron.yaml
index.html
package.json
```

---

# 8. `git status`

Run:

```bash
git status
```

Git tells you what's happening in your working directory.

Initially:

```text
Untracked files:
    app.js
    cron.yaml
    package.json
```

### What does "untracked" mean?

Git sees the files, but they haven't been added to the next commit.

Think:

```text
File created
     ↓
UNTRACKED
     ↓
git add
     ↓
STAGED
     ↓
git commit
     ↓
COMMITTED
     ↓
git push
     ↓
CodeCommit
```

---

# 9. `git add .`

Command:

```bash
git add .
```

The `.` means:

> Add all relevant changes in the current directory.

Now:

```bash
git status
```

will show the files as **staged**.

### Staging area

Git has an intermediate area:

```text
Working Directory
       ↓
   git add .
       ↓
Staging Area
       ↓
   git commit
       ↓
Local Repository
```

---

# 10. `git commit`

Command:

```bash
git commit -m "added three files"
```

### Meaning

* `git commit` → create a commit
* `-m` → provide a commit message
* `"added three files"` → description of the change

This creates a new commit **in the local repository**.

```text
app.js
cron.yaml
package.json
       ↓
   git add .
       ↓
   git commit
       ↓
LOCAL COMMIT
```

It still isn't in CodeCommit yet.

---

# 11. Git Identity Configuration

Before committing, Git may require your identity.

Configure email:

```bash
git config --global user.email "stephane@example.com"
```

Configure username:

```bash
git config --global user.name "stephane"
```

Then retry:

```bash
git commit -m "added three files"
```

### Why?

Git records the author information associated with the commit.

```text
Commit
 ├── Author: stephane
 ├── Email: stephane@example.com
 └── Changes
```

---

# 12. `git push`

This is the final step in the lecture:

```bash
git push
```

It sends the local commit to the remote CodeCommit repository.

```text
Local Repository
      ↓
   git push
      ↓
CodeCommit
```

After refreshing the CodeCommit console, the new files appear.

---

# 13. Complete Git Workflow

This is the **main workflow you should remember**:

```text
              CodeCommit
                  ↑
                  │
              git push
                  │
            Local Repository
                  ↑
                  │
             git commit
                  ↑
                  │
              git add .
                  ↑
                  │
            Working Directory
```

Or in simple terms:

```text
Create/modify files
        ↓
   git status
        ↓
    git add .
        ↓
    git commit
        ↓
     git push
        ↓
   CodeCommit
```

---

# 14. Command Definitions

| Command                             | Meaning                                    |
| ----------------------------------- | ------------------------------------------ |
| `git clone`                         | Download/clone a remote repository locally |
| `cd`                                | Move into a directory                      |
| `ls`                                | List files/directories                     |
| `git status`                        | Show current Git state                     |
| `git add .`                         | Stage changes                              |
| `git commit`                        | Create a local commit                      |
| `git push`                          | Upload commits to remote repository        |
| `git config`                        | Configure Git settings                     |
| `git config --global user.name`     | Configure Git username                     |
| `git config --global user.email`    | Configure Git email                        |
| `pip install git-remote-codecommit` | Install CodeCommit Git remote helper       |

---

# 15. `clone` vs `pull` vs `push` vs `commit`

These are easy exam/interview questions.

### `git clone`

Used when you **first get a repository**.

```text
CodeCommit → Local
```

### `git pull`

Used to get **new changes from the remote repository**.

```text
CodeCommit → Local
```

### `git add`

Moves changes into the **staging area**.

```text
Working Directory → Staging
```

### `git commit`

Records staged changes in your **local repository**.

```text
Staging → Local Repository
```

### `git push`

Sends local commits to the **remote repository**.

```text
Local Repository → CodeCommit
```

---

# ⭐ Most Important Mental Model

Don't memorize the commands independently. Remember the pipeline:

```text
                 YOUR MACHINE / CLOUDSHELL
                 ─────────────────────────

                 Edit/Create Files
                        ↓
                   git status
                        ↓
                    git add .
                        ↓
                    git commit
                        ↓
                 LOCAL REPOSITORY
                        ↓
                    git push
                        ↓
                 AWS CODECOMMIT
```

And when you want someone else's latest changes:

```text
AWS CODECOMMIT
      ↓
   git pull
      ↓
LOCAL REPOSITORY
```

### One-line memory trick

> **Clone → get the repository**
> **Status → see changes**
> **Add → stage changes**
> **Commit → save locally**
> **Push → send to CodeCommit**
> **Pull → bring remote changes locally**

**For the AWS DevOps exam, the key takeaway from this lecture is less about memorizing Git syntax and more about understanding that CodeCommit is a Git repository and that authentication/authorization determine whether your Git operations are permitted.**
