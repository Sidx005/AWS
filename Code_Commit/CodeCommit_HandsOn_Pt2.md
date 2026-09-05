# AWS CodeCommit — Connecting and Pushing Code with Git

## 1. Getting the Connection Steps

In the CodeCommit repository:

```text
Clone URL → Connection steps
```

AWS provides connection instructions depending on your authentication method:

* **HTTPS**
* **SSH**
* **HTTPS (GRC / Git Remote CodeCommit)**

The lecture uses **HTTPS GRC** because the instructor is using a **federated account**.

---

# 2. Using CloudShell

The easiest environment to practice is **AWS CloudShell** because it already includes:

* Git
* Python
* AWS environment configuration

So you don't necessarily need to install everything manually.

Check files using:

```bash
ls
```

---

# 3. Install `git-remote-codecommit`

For the HTTPS GRC approach, install the required Python package:

```bash
pip install git-remote-codecommit
```

This package allows Git to communicate with AWS CodeCommit using AWS authentication.

---

# 4. Clone the CodeCommit Repository

Copy the clone command provided by CodeCommit and run:

```bash
git clone <codecommit-repository-url>
```

For example:

```text
git clone codecommit://my-nodejs-app
```

After cloning:

```bash
ls
```

You should see:

```text
my-nodejs-app
```

Enter the repository:

```bash
cd my-nodejs-app
```

Then:

```bash
ls
```

You might see:

```text
index.html
```

This confirms that the repository has been successfully cloned.

---

# 5. Important Concept: Clone

When you run:

```bash
git clone
```

Git creates a **local copy** of the remote CodeCommit repository.

```text
AWS CodeCommit Repository
          │
          │ git clone
          ▼
Local Machine / CloudShell
```

Now you can modify the code locally.

---

# 6. Adding New Files

Suppose the repository initially contains:

```text
index.html
```

You upload additional files to CloudShell:

```text
app.js
cron.yaml
package.json
```

Then move them into the cloned repository:

```bash
mv app.js my-nodejs-app/
mv cron.yaml my-nodejs-app/
mv package.json my-nodejs-app/
```

Your repository now contains:

```text
my-nodejs-app/
│
├── index.html
├── app.js
├── cron.yaml
└── package.json
```

---

# 7. Checking Git Status

Run:

```bash
git status
```

Git will show:

```text
Untracked files:
  app.js
  cron.yaml
  package.json
```

### What does "Untracked" mean?

Git sees the files, but Git is **not yet tracking them**.

```text
File exists
     ↓
Git sees it
     ↓
Untracked ❌
```

You need to add them to Git tracking.

---

# 8. Git Add

To add all files:

```bash
git add .
```

The `.` means:

> Add all changes in the current directory.

Now run:

```bash
git status
```

The files should appear as ready to be committed.

---

# 9. Git Workflow: Working Directory → Staging → Commit

This is the most important Git concept:

```text
Working Directory
       │
       │ git add
       ▼
Staging Area
       │
       │ git commit
       ▼
Local Git Repository
       │
       │ git push
       ▼
Remote Repository (CodeCommit)
```

### Commands:

```bash
git add .
git commit -m "message"
git push
```

---

# 10. Configuring Your Git Identity

Before creating a commit, Git needs to know who you are.

Configure your email:

```bash
git config --global user.email "stephane@example.com"
```

Configure your username:

```bash
git config --global user.name "stephane"
```

You can verify using:

```bash
git config --global --list
```

> The `--global` option applies the configuration to your user account.

---

# 11. Creating a Commit

Create a commit:

```bash
git commit -m "Added three files"
```

Git creates a **local commit**.

Example:

```text
[master abc123] Added three files
 3 files changed
 create mode app.js
 create mode cron.yaml
 create mode package.json
```

At this point, the changes are committed **locally**.

⚠️ They are **not yet in AWS CodeCommit**.

---

# 12. Local Commit vs Remote Repository

This distinction is extremely important.

After:

```bash
git commit
```

Your situation is:

```text
Local Repository ✅ Updated

AWS CodeCommit ❌ Not Updated Yet
```

The commit exists only locally.

To send it to CodeCommit, use:

```bash
git push
```

---

# 13. Pushing to CodeCommit

Run:

```bash
git push
```

Now:

```text
Local Git Repository
        │
        │ git push
        ▼
AWS CodeCommit Repository
```

After refreshing the CodeCommit console, you should see:

```text
index.html
app.js
cron.yaml
package.json
```

---

# Complete Workflow

```bash
# Clone repository
git clone <repository-url>

# Enter repository
cd my-nodejs-app

# Check current files
ls

# Check Git status
git status

# Add all changes
git add .

# Configure Git identity (if needed)
git config --global user.email "your-email@example.com"
git config --global user.name "Your Name"

# Create a commit
git commit -m "Added three files"

# Push to AWS CodeCommit
git push
```

---

# Key Difference: `git add`, `git commit`, and `git push`

| Command      | What it does                              |
| ------------ | ----------------------------------------- |
| `git add .`  | Moves changes to the staging area         |
| `git commit` | Saves changes in the local Git repository |
| `git push`   | Sends commits to the remote repository    |

### Easy way to remember:

```text
git add     → Prepare changes
git commit  → Save locally
git push    → Upload remotely
```

---

# Overall Architecture

```text
Your Files
    │
    ▼
Working Directory
    │
    │ git add .
    ▼
Staging Area
    │
    │ git commit
    ▼
Local Git Repository
    │
    │ git push
    ▼
AWS CodeCommit ☁️
```

## Key takeaway

**`git commit` does not automatically update CodeCommit.**

You need:

```bash
git push
```

to send your local commits to the remote AWS CodeCommit repository.


<img width="690" height="426" alt="image" src="https://github.com/user-attachments/assets/4883b2c8-af91-4b40-9261-b2cd0168de94" />


<img width="675" height="412" alt="image" src="https://github.com/user-attachments/assets/2c53e553-7473-4696-96a5-b18839b3f9c0" />
