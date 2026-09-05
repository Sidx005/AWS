# Beanstalk
# What is AWS Elastic Beanstalk? 🌱

**AWS Elastic Beanstalk** is a service that makes it easy to **deploy and run web applications** without manually managing all the underlying infrastructure.

You provide your application code, and Beanstalk handles much of the infrastructure setup for you.

## Simple idea

```text
Your Application Code
        ↓
Elastic Beanstalk
        ↓
Automatically sets up infrastructure
        ↓
EC2 + Load Balancer + Auto Scaling + Monitoring
        ↓
Your Application Running 🚀
```

### Example

Suppose you have a Node.js application:

```text
app.js
package.json
index.html
```

Instead of manually:

1. Creating an EC2 instance
2. Installing Node.js
3. Configuring the server
4. Setting up Auto Scaling
5. Configuring a Load Balancer

You can simply upload your application to **Elastic Beanstalk**.

---

## What Beanstalk manages

Depending on your configuration, it can provision:

* **EC2 instances** → Run your application
* **Auto Scaling** → Add/remove servers based on demand
* **Elastic Load Balancer** → Distribute traffic
* **CloudWatch** → Monitoring and logs
* **Security Groups** → Control network access

---

## Why are you creating two environments?

In your CodePipeline lecture, you created environments such as:

```text
Elastic Beanstalk Application
│
├── Dev Environment
│      ↓
│   Testing happens here
│
└── Prod Environment
       ↓
    Real production users
```

Then CodePipeline can deploy code like this:

```text
CodeCommit
    ↓
CodePipeline
    ↓
Elastic Beanstalk (Dev)
    ↓
Elastic Beanstalk (Prod)
```

### Easy definition for exams 📝

> **AWS Elastic Beanstalk is a Platform as a Service (PaaS) that lets you quickly deploy and manage applications without worrying about the underlying infrastructure.**

**In one line:** You give Beanstalk your **code**, and AWS helps manage the **servers and infrastructure needed to run it**.


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
