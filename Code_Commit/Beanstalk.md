# Prerequisite for CodePipeline Hands-On: Create Elastic Beanstalk Environments

Before setting up **AWS CodePipeline**, the lecture recreates an **Elastic Beanstalk application** because CodePipeline will later deploy application updates into these environments.

## 1. Create the Elastic Beanstalk Application

Create a **Web Server Environment**.

### Application

```text
Application name: My First Web App Beanstalk
```

### Environment

Create the first environment using:

* Platform: **Node.js**
* Platform version: **Latest**
* Application code: **Simple application**
* Environment type: **Single instance**

No key pair is required for this demo.

Then:

```text
Next → Skip to Review → Submit
```

After deployment succeeds, Elastic Beanstalk shows the environment as successfully deployed.

---

# 2. Create a Second Environment for Production

The lecture then creates another environment under the same Elastic Beanstalk application.

```text
Application: My First Web App Beanstalk
Environment name: prod
```

Use the same configuration:

* **Web server environment**
* **Node.js**
* **Simple application**
* **Single instance**

So the final structure looks like:

```text
My First Web App Beanstalk
│
├── Environment 1 (Development/Test)
│
└── prod
```

---

# 3. Why Create Two Environments?

CodePipeline will later deploy the application to multiple environments.

Conceptually:

```text
CodeCommit
     ↓
CodePipeline
     ↓
Development Environment
     ↓
Production Environment
```

This allows us to simulate a typical CI/CD workflow where changes can move through environments before reaching production.

---

# 4. Important Cost Warning ⚠️

Elastic Beanstalk itself may not be the main cost, but the resources it creates can cost money, such as:

* EC2 instances
* Load balancers (depending on configuration)
* Other supporting AWS resources

Since this hands-on uses a **Single Instance environment**, an EC2 instance will be running.

Therefore, after completing the lab:

```text
Delete Elastic Beanstalk Environments
        ↓
Avoid unnecessary AWS charges
```

## Key Takeaway

Before starting the CodePipeline hands-on, we need:

```text
Elastic Beanstalk Application
          │
          ├── Environment 1
          │
          └── Production Environment
```

These environments will act as the **deployment targets** for the upcoming CodePipeline setup.


<img width="527" height="364" alt="image" src="https://github.com/user-attachments/assets/8ec34867-8471-4384-b6c3-e70985b56591" />


# Configure ENvironmet
<img width="703" height="382" alt="image" src="https://github.com/user-attachments/assets/d7c272fc-6ca7-4002-84bd-f8d288cf2d38" />


<img width="644" height="417" alt="image" src="https://github.com/user-attachments/assets/8e176474-47a7-43b7-89ca-96c2973b2be2" />
