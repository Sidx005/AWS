# AWS Elastic Beanstalk — Complete Notes & Hands-on

## 1. What is AWS Elastic Beanstalk?

**AWS Elastic Beanstalk** is a Platform as a Service (PaaS) that makes it easier to deploy, manage, and scale web applications.

You upload your application code, select a platform, and Elastic Beanstalk provisions and manages the underlying AWS infrastructure for you.

It can manage resources such as:

- Amazon EC2 — runs your application.
- Elastic Load Balancing — distributes traffic across instances.
- Auto Scaling — adjusts the number of instances.
- Amazon S3 — stores application versions and deployment artifacts.
- Amazon CloudWatch — monitors metrics and alarms.
- AWS CloudFormation — provisions and tracks infrastructure resources.

### Key idea

You focus on **your application**, while Elastic Beanstalk handles much of the infrastructure setup and management.

> Elastic Beanstalk does not eliminate infrastructure. It automates its provisioning and management.

---

## 2. Core Components

| Component | Meaning |
|---|---|
| **Application** | Logical container for your application versions and environments. |
| **Application Version** | A specific, identifiable version of your application code. |
| **Environment** | The AWS resources running a particular application version. |
| **Environment Tier** | Defines how the environment processes requests or background jobs. |
| **Platform** | The runtime and operating system configuration, such as Node.js or Python. |

### Relationship between components

```text
Elastic Beanstalk Application
│
├── Application Version 1
├── Application Version 2
│
├── Development Environment
│   └── Running Version 1
│
└── Production Environment
    └── Running Version 2
```

One application can have multiple versions and multiple environments.

For example, you can test a new version in Development before deploying it to Production.

---

## 3. Environment Tiers

Elastic Beanstalk supports two main environment tiers.

### A. Web Server Environment

Used for applications that receive and respond to HTTP/HTTPS requests.

Examples:
- React frontend served by a backend.
- Node.js web application.
- Python Flask or Django application.
- REST API.

```text
User / Browser
      │
      ▼
HTTP / HTTPS Request
      │
      ▼
Elastic Load Balancer (if configured)
      │
      ▼
EC2 Instance(s)
      │
      ▼
Application Response
```

### B. Worker Environment

Used for background processing rather than responding directly to each user request.

A worker environment can consume messages from an Amazon SQS queue and process them asynchronously.

```text
User Request
     │
     ▼
Web Application
     │
     ▼
Amazon SQS Queue
     │
     ▼
Worker Environment
     │
     ▼
Background Processing
```

**Example:** A user uploads a video. The web application queues a processing task, and a worker generates thumbnails without making the user wait for the entire process.

| Web Server Tier | Worker Tier |
|---|---|
| Handles incoming web requests | Processes background jobs |
| Returns HTTP responses | Consumes queued tasks |
| Often sits behind a load balancer | Commonly integrates with SQS |
| Example: REST API | Example: video processing |

---

## 4. Deployment Configurations

### A. Single-instance deployment

Best for learning, development, and small workloads.

```text
Users
  │
  ▼
Elastic Beanstalk Environment
  │
  ▼
Single EC2 Instance
  │
  ▼
Application
```

Advantages:
- Simple setup.
- Lower infrastructure cost.
- Useful for experimentation.

Limitations:
- A single instance is a single point of failure.
- Limited capacity compared with a scaled environment.
- Does not provide the same high availability as a multi-instance setup.

### B. Load-balanced, Auto Scaling deployment

Suitable for production workloads that need greater availability and scalability.

```text
                   Users
                     │
                     ▼
              Load Balancer
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      EC2 Instance 1       EC2 Instance 2
          │                     │
          └──────────┬──────────┘
                     ▼
                  Application
```

Auto Scaling can add or remove instances according to configured scaling policies and demand.

**Remember:** A load balancer distributes incoming requests. Auto Scaling adjusts the number of instances.

---

## 5. Three-Tier Architecture

A common web application architecture separates the application into three layers.

```text
┌─────────────────────────────┐
│ Presentation Tier           │
│ Browser / Frontend          │
└──────────────┬──────────────┘
               │ HTTP / API
               ▼
┌─────────────────────────────┐
│ Application Tier            │
│ Elastic Beanstalk / EC2     │
│ Business Logic              │
└──────────────┬──────────────┘
               │ Database Queries
               ▼
┌─────────────────────────────┐
│ Data Tier                   │
│ Database / Data Storage     │
└─────────────────────────────┘
```

Elastic Beanstalk can manage the application tier. The frontend and database may be deployed separately, depending on the architecture.

---

# 6. Hands-on: Create Your First Elastic Beanstalk Application

**Goal:** Deploy a sample Node.js application using a single EC2 instance.

## Step 1: Open Elastic Beanstalk

1. Sign in to the AWS Management Console.
2. Search for **Elastic Beanstalk**.
3. Open the Elastic Beanstalk service.
4. Choose **Create application**.

Official AWS guide:

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/GettingStarted.html

### Screenshot: Create an application

![AWS Elastic Beanstalk console showing the Create application workflow](https://docs.aws.amazon.com/images/elasticbeanstalk/latest/dg/images/aeb-gs-01-create-app.png)

## Step 2: Configure the application

Use the following example values:

| Setting | Value |
|---|---|
| Application name | `My Application` |
| Environment name | `My Application Dev` |
| Environment tier | Web server environment |
| Platform | Node.js |
| Application code | Sample application |
| Preset | Single instance |

The exact console layout and available platform versions may change.

### What does the sample application do?

Elastic Beanstalk deploys a sample application so you can verify that the environment works before uploading your own code.

## Step 3: Configure the service role and EC2 instance profile

Elastic Beanstalk needs IAM permissions to create and manage AWS resources.

Two important IAM concepts are involved:

| IAM component | Purpose |
|---|---|
| **Service role** | Allows Elastic Beanstalk to perform management operations on your behalf. |
| **EC2 instance profile** | Provides AWS permissions to the EC2 instance running your application. |

If the console asks you to create or select these roles, follow the prompts and use the recommended permissions for your environment.

Do not confuse the service role with the instance profile: they serve different identities.

## Step 4: Review and create

1. Skip optional configurations if you are following the basic learning exercise.
2. Review the application and environment settings.
3. Choose **Submit**, **Create environment**, or the equivalent button shown in your console.
4. Wait for the environment creation and deployment process to finish.

Initial deployment can take several minutes.

---

# 7. What Happens Behind the Scenes?

Elastic Beanstalk provisions the infrastructure needed for the selected environment. It uses AWS CloudFormation to create and track many of the underlying resources.

For a basic single-instance environment, resources can include:

- An EC2 instance.
- A security group.
- An Elastic IP address, depending on the configuration.
- A CloudFormation stack.
- An S3 bucket for application versions and deployment artifacts.
- CloudWatch alarms and monitoring resources, depending on the configuration.
- An environment-specific domain name.

The exact resources vary with the environment configuration and AWS platform behavior.

### Architecture

![Official AWS Elastic Beanstalk web environment architecture](https://docs.aws.amazon.com/images/elasticbeanstalk/latest/dg/images/aeb-overview-web-only.png)

### Infrastructure creation flow

```text
You Configure Elastic Beanstalk
              │
              ▼
    Elastic Beanstalk Environment
              │
              ▼
       AWS CloudFormation
              │
       ┌──────┼──────────┐
       ▼      ▼          ▼
      EC2   Security    Other AWS
            Groups      Resources
       │
       ▼
  Application Deployed
       │
       ▼
  Environment Domain
```

**Important:** CloudFormation is the infrastructure provisioning mechanism; Elastic Beanstalk provides the higher-level application deployment experience.

---

# 8. Verify the Deployment

Once environment creation is complete:

1. Open your Elastic Beanstalk environment.
2. Check its status and health.
3. Look for a healthy status such as **Ok**, when applicable.
4. Find the environment URL or choose **Go to environment**.
5. Open the URL in your browser.

### Screenshot: Environment console

![AWS Elastic Beanstalk environment console overview](https://docs.aws.amazon.com/images/elasticbeanstalk/latest/dg/images/aeb-gs-console-environment.png)

### Screenshot: Deployed sample application

![AWS Elastic Beanstalk sample application running successfully](https://docs.aws.amazon.com/images/elasticbeanstalk/latest/dg/images/aeb-gs-launched-app.png)

If the sample application loads, the initial deployment has succeeded.

If it does not load, check the environment health, events, logs, security groups, and deployment status.

---

# 9. Important Environment Tabs

Elastic Beanstalk provides several tabs or sections for monitoring and troubleshooting.

| Section | What it is used for |
|---|---|
| **Events** | Shows environment creation, configuration, deployment, and error messages. |
| **Health** | Shows environment and instance health information. |
| **Logs** | Retrieves application and instance logs. |
| **Monitoring** | Displays metrics such as request counts, latency, and instance-related metrics where available. |
| **Alarms** | Shows or helps manage relevant CloudWatch alarms. |
| **Configuration** | Lets you modify environment settings and underlying resource configuration. |

### Troubleshooting workflow

```text
Application Not Working
          │
          ▼
    Check Events
          │
          ▼
    Check Health
          │
          ▼
     Retrieve Logs
          │
          ▼
 Review Configuration
          │
          ▼
    Fix and Redeploy
```

Use the Events section first for a quick overview of what went wrong.

---

# 10. Deploy a New Application Version

After the initial deployment, you can upload a new application version without creating an entirely new Elastic Beanstalk application.

Typical workflow:

1. Prepare your application code in the required deployment package format.
2. Open the existing environment in Elastic Beanstalk.
3. Choose the deployment or **Upload and deploy** option.
4. Upload the application package.
5. Provide a version label if prompted.
6. Start deployment.
7. Monitor Events and Health.
8. Open the environment URL to verify the changes.

```text
Prepare Code
     │
     ▼
Package Application
     │
     ▼
Upload and Deploy
     │
     ▼
New Application Version
     │
     ▼
Environment Updated
     │
     ▼
Verify Application
```

Official AWS documentation:

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html

**Note:** Deployment behavior and downtime depend on the deployment policy and environment configuration. Do not assume every deployment is zero-downtime.

---

# 11. Development and Production Environments

You can create multiple environments under the same Elastic Beanstalk application.

For example:

```text
My Application
│
├── My Application Dev
│   └── Test new features
│
├── My Application Staging
│   └── Validate release candidate
│
└── My Application Prod
    └── Serve real users
```

Each environment can have its own configuration, capacity, URL, and deployed version.

A common workflow is:

1. Deploy to Development.
2. Test functionality.
3. Deploy to Staging for release validation.
4. Deploy the approved version to Production.

**Important:** Creating a separate environment can provision additional billable AWS resources.

---

# 12. Elastic Beanstalk vs CloudFormation

| Elastic Beanstalk | AWS CloudFormation |
|---|---|
| Application-centric | Infrastructure-centric |
| Simplifies application deployment | Defines and provisions AWS resources |
| Manages environment configuration and deployment | Uses templates to describe infrastructure |
| Can use CloudFormation behind the scenes | Can provision resources directly |
| Good when you want a managed application platform | Good when you need detailed infrastructure control |

### Simple analogy

- **Elastic Beanstalk:** “Deploy and manage my application.”
- **CloudFormation:** “Create and manage these specific AWS resources.”

They are not mutually exclusive. Elastic Beanstalk can use CloudFormation internally.

---

# 13. Cleanup — Avoid Unnecessary AWS Charges

After finishing the practical exercise:

1. Open Elastic Beanstalk.
2. Select the environment you created.
3. Choose **Terminate environment**.
4. Confirm termination.
5. Check whether any separately created resources remain.
6. Delete unused resources only after verifying that they are not shared or needed by another application.

Terminating an environment does not necessarily remove every resource that might have been created separately or retained by configuration.

Also check your AWS billing and Free Tier/credit usage. A single-instance setup is cheaper than many production configurations, but it is not automatically free.

---

# 14. Quick Revision

- **Elastic Beanstalk:** Managed platform for deploying and operating applications.
- **Application:** Groups versions and environments.
- **Application version:** A particular release of application code.
- **Environment:** Resources running a version of the application.
- **Web server tier:** Handles incoming HTTP/HTTPS requests.
- **Worker tier:** Handles asynchronous background tasks, commonly using SQS.
- **Single instance:** Simple and relatively low-cost, but has a single point of failure.
- **Load balancer:** Distributes incoming traffic.
- **Auto Scaling:** Adjusts instance capacity according to configured policies.
- **Service role:** Gives Elastic Beanstalk permissions to manage resources.
- **EC2 instance profile:** Supplies AWS permissions to the application instance.
- **CloudFormation:** Provisions and tracks underlying infrastructure.
- **Events and Logs:** Useful starting points for troubleshooting.
- **Multiple environments:** Support separate Development, Staging, and Production deployments.
- **Cleanup:** Terminate unused environments and verify remaining resources.

## Official AWS References

1. [Getting started with Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/GettingStarted.html)
2. [Managing Elastic Beanstalk applications](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/applications-console.html)
3. [Deploying an existing application version](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html)



<img width="943" height="559" alt="image" src="https://github.com/user-attachments/assets/9cac4aa3-ca8c-459c-b618-87f15d5f92bc" />

<img width="477" height="268" alt="image" src="https://github.com/user-attachments/assets/24bef633-711a-41c1-94f2-90e9d94db991" />

**Domain will bw autogenerated**

# Platform

<img width="483" height="262" alt="image" src="https://github.com/user-attachments/assets/4f5fb53b-988b-49a2-90c9-d15a62a6bee5" />


# Application Code
<img width="485" height="312" alt="image" src="https://github.com/user-attachments/assets/e3b04289-6aca-4fc4-be43-51641c924a66" />


# Configure Service Access
<img width="544" height="333" alt="image" src="https://github.com/user-attachments/assets/9361cdbc-15d5-4476-a5ad-1511dab40ce7" />


# Service Trusted entity
<img width="637" height="322" alt="image" src="https://github.com/user-attachments/assets/ad207b48-1133-46c9-ac27-5554dd3f58d9" />


# Permissions
<img width="715" height="293" alt="image" src="https://github.com/user-attachments/assets/1ac52704-aa9a-4573-bc02-777d778c19ef" />


<img width="671" height="444" alt="image" src="https://github.com/user-attachments/assets/79c6b221-cdfa-40c6-a0be-c7784e063e78" />


<img width="774" height="376" alt="image" src="https://github.com/user-attachments/assets/bcd63d19-e6e7-4f9f-965a-40a7114c8435" />


# Review


<img width="795" height="433" alt="image" src="https://github.com/user-attachments/assets/bd51d748-435d-4a42-8f61-6c9eb1cb4782" />





<img width="551" height="337" alt="image" src="https://github.com/user-attachments/assets/2f54da77-46ba-4d6e-9441-ed1e8dbbba2a" />
