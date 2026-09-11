# AWS CodeDeploy — Simplified Notes

AWS CodeDeploy is an AWS service that automates deploying new versions of an application.

For example, you have version 1 running and want to upgrade to version 2 without manually updating every server.

**Main idea: CodeDeploy automates application deployment, controls how traffic or instances are updated, and can roll back failed deployments.**

## 1. What can CodeDeploy deploy to?

| Platform            | Deployment type                   |
| ------------------- | --------------------------------- |
| EC2 instances       | In-place or blue/green            |
| On-premises servers | In-place or blue/green            |
| Lambda functions    | Traffic shifting between versions |
| ECS services        | Blue/green only                   |

---

## 2. CodeDeploy with EC2

Before CodeDeploy can update an EC2 instance, the **CodeDeploy agent** must be installed.

The agent is responsible for receiving and executing deployment instructions on the instance.

### Basic flow

```text
Developer
    ↓
Application Revision
    ↓
S3 Bucket
    ↓
CodeDeploy
    ↓
CodeDeploy Agent
    ↓
EC2 Instance
    ↓
Application v2
```

The application revision is the version of your application that CodeDeploy deploys.

### Important

The EC2 instance needs appropriate IAM permissions to access the S3 bucket containing the application revision.

---

# 3. In-Place Deployment

In-place deployment means updating the existing instances.

You do not create an entirely new set of servers.

### Example: HalfAtATime

You have four EC2 instances running version 1.

![Image](https://images.openai.com/static-rsc-4/6eBw45lZ0iCAQPxpgtJYaS9eNQR1biu71N2NGe-Lhwzb-kS-bOxjk-LkaZ5IWp0ozBxgfQrkWda33ceJ3R54BkToQ8TTlN28ZAI5HtCYX1araGymjUx47Ik_J3vL6aUEPYzFBfrxk_VEPUKUjvbum6-FYazvM9s09TK_hdoZrMZzaMr50HFnobpCjbKklL9G?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/HkS3oLZujkMfGkyKu3ic5GpoJpytOo6gHv4phLZ_qk8ya74h7-m6VQacwSPWIBpY4pAdhgP4QfMNH3DX5nVeNC7bVOOIaR4K7sk1EkRi0Qd8WOXHav8CAz9i5BCsw0aicLrSsJVhrmLO4pkyinVb2jcsJojCManVeu_z40zJqPZAn_v7rgDRgd-5hCYitRA7?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/5iSQOay7LCD0CY-eelE1Vz0c8UVsNgf_7I1xwfhoMg-nLuuUq0MRQgCxQNSDhAm_aoethCkS2_HLrIXF7egEjPzeo4TQPUpjZ20EDHNx8uCSsmLwCiPQS-kpnOHymqoF6ZPVcJyQ9lQKUndvSapFXG27vCVf3CtdTMrG23SKS9eOqh8PvQ3NCxkWMrnwmX_E?purpose=fullsize)

```text
Before:

EC2-1   EC2-2   EC2-3   EC2-4
  v1      v1      v1      v1
```

### Step 1: Update half

```text
EC2-1   EC2-2   EC2-3   EC2-4
  v2      v2      v1      v1
```

### Step 2: Update remaining instances

```text
EC2-1   EC2-2   EC2-3   EC2-4
  v2      v2      v2      v2
```

The CodeDeploy agent updates the instances in batches.

## Deployment configurations

| Configuration | How it works                  | Availability impact         |
| ------------- | ----------------------------- | --------------------------- |
| AllAtOnce     | Update all instances together | Highest risk of downtime    |
| HalfAtATime   | Update half at a time         | Moderate capacity reduction |
| OneAtATime    | Update one at a time          | Lowest capacity reduction   |
| Custom        | Define your own batch size    | Depends on configuration    |

**Exam shortcut:** In-place deployment updates existing instances. The number of instances taken out of service depends on the deployment configuration.

---

# 4. Blue/Green Deployment

Blue/green deployment creates a separate environment for the new application version.

* **Blue** = existing version
* **Green** = new version

![Image](https://images.openai.com/static-rsc-4/tVg6VzDIuUaFpiMKvGSh1WqkrgtmYRA2h10wX7Fses_iPS2WhosKMdPlS01GxzWatzip7jvjX0uaf0q9oWTj3yqOw1xSRWNlsx7whJlQzorW5ULUGcoc0ANfIne72W8_SPIob15OvnDAhy22Vlugm445Wh7XrjbBC_1h_DaF7Qj1R39JO5L_1mLC7vDO7y8p?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ZVKJumEmdu5i6hz2p3xCtLBDUbCKMFv4xLU-A2-dbXOv7C7AB0OKr8gIN0CgSARLU4Y8weLel8dmMqNX_kSof8HNQFqj4OD3gnRbruy2zXyEvcQ5P7MMLnd7pUCK4FJoBLezC6IhS_lEznvU4ShjcgPsglgJzZC8a5IjEdYAtf9tNxNs7iL__g5VLJ3F8R--?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/Xo-2HNC1AjADR0fpmVfCQujG9fusLaIkbItIap8yQnAk4E9SXaYUJ8jup6S7iWM2SJuDOYSiWftF9qyUrDnh-8lEOs1n5s_Jwh5jWlB9mV25KBY0f-CNzdxO5UrxnuiSwZsN_2ZV8pTUagmqYsCqkRVXdJHqV2NCgjvbRtZYcjAllnd9CSoASQq5PgD4dOKL?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/xln3olH1m0XJbhd3ud-n8d5tzuNoF_mujbsAfwt_Blwa0sRHVBRgfCBciZFAkQDap6x6_gm55nTtNwffvDHF-aB_ZQE-7ABgfKOK0AiHoORKNPYp7aFtwS3m_Uw22-ta2kwz3UBSUEcjQthVL4yDA4SJjDQtdapQl4gRmEj5PyIqchNuadQGwrxeipJEv5xK?purpose=fullsize)

### Before deployment

```text
Application Load Balancer
          ↓
      Blue ASG
          ↓
    EC2 instances
          ↓
           v1
```

### During deployment

```text
Application Load Balancer
          │
     ┌────┴────┐
     ↓         ↓
 Blue ASG   Green ASG
    v1          v2
```

CodeDeploy creates or uses a new Auto Scaling group for version 2.

Both versions can run at the same time.

### After successful deployment

```text
Application Load Balancer
          ↓
      Green ASG
          ↓
           v2
```

The old blue environment can then be terminated.

### Why blue/green is useful

* Test the new version separately
* Reduce deployment risk
* Switch traffic to the new version
* Keep the old version available for rollback

---

# 5. In-Place vs Blue/Green

| Feature                       | In-Place                  | Blue/Green                             |
| ----------------------------- | ------------------------- | -------------------------------------- |
| Existing instances updated?   | Yes                       | No, new environment is used            |
| New instances required?       | Not necessarily           | Yes                                    |
| Two versions run in parallel? | Not normally              | Yes                                    |
| Rollback approach             | Redeploy previous version | Switch traffic back to old environment |
| Infrastructure cost           | Usually lower             | Usually higher during deployment       |

**Remember:** Blue/green is not simply updating half the instances. It is creating a separate environment and shifting traffic between environments.

---

# 6. Deployment Speed vs Deployment Type

These are different concepts.

### Deployment type

How the application is updated:

* In-place
* Blue/green

### Deployment speed

How quickly the update happens:

* AllAtOnce
* HalfAtATime
* OneAtATime
* Custom

For example:

> In-place + HalfAtATime

or:

> Blue/green + traffic shifting

---

# 7. CodeDeploy Agent

The CodeDeploy agent must be installed on EC2/on-premises targets.

It can be installed manually or through automation.

### Manual installation

Use Linux commands to install the agent.

### Automated installation

Use AWS Systems Manager to automate installation on managed instances.

```text
Systems Manager
       ↓
Install CodeDeploy Agent
       ↓
EC2 Instance
```

### Exam point

Systems Manager can help install/manage the agent, but **the CodeDeploy agent performs the deployment on the instance**.

---

# 8. `appspec.yml`

The `appspec.yml` file defines how CodeDeploy performs a deployment.

It can specify:

* Files to copy
* Where to copy them
* Scripts to run
* Deployment lifecycle hooks

Example:

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /var/www/myapp

hooks:
  ApplicationStart:
    - location: scripts/start.sh
      timeout: 300
```

This is a simplified example.

**Exam shortcut:** `appspec.yml` is the CodeDeploy deployment specification file.

---

# 9. Rollbacks

CodeDeploy can automatically roll back a deployment when configured conditions are met.

For example:

```text
Version 1
    ↓
Deploy Version 2
    ↓
CloudWatch Alarm
    ↓
Alarm triggered
    ↓
Rollback
    ↓
Version 1
```

A rollback can be triggered by:

* Deployment failure
* A configured CloudWatch alarm
* Other configured deployment conditions

### Important

Rollback is not guaranteed for every failure automatically. You must configure the appropriate rollback behavior and alarms.

---

# 10. CodeDeploy with Lambda

CodeDeploy supports traffic shifting between Lambda versions using an alias.

Example:

```text
PROD Alias
    ↓
Lambda v1
```

You deploy version 2:

```text
PROD Alias
    ↓
Lambda v1 + Lambda v2
```

CodeDeploy gradually moves traffic from version 1 to version 2.

Eventually:

```text
PROD Alias
    ↓
Lambda v2
```

### Important terms

* **Lambda version:** A published, immutable version of a Lambda function.
* **Lambda alias:** A named pointer to a Lambda version.
* **Traffic shifting:** Gradually changing which version receives requests.

---

# 11. Lambda Deployment Strategies

## Linear

Traffic gradually increases toward the new version.

Example:

`LambdaLinear10PercentEvery3Minutes`

| Time        | v2 traffic |
| ----------- | ---------: |
| Start       |         0% |
| After 3 min |        10% |
| After 6 min |        20% |
| After 9 min |        30% |
| ...         |        ... |
| Final       |       100% |

## Canary

Send a small amount of traffic to the new version first, wait, then move all traffic if successful.

Example:

`LambdaCanary10Percent5Minutes`

| Time        | v2 traffic |
| ----------- | ---------: |
| Start       |        10% |
| After 5 min |       100% |

## AllAtOnce

Immediately switch all traffic to the new version.

| Time        | v2 traffic |
| ----------- | ---------: |
| Start       |         0% |
| Immediately |       100% |

### Exam memory

| Strategy  | Meaning                                |
| --------- | -------------------------------------- |
| Linear    | Gradually increase traffic             |
| Canary    | Small traffic first, then full traffic |
| AllAtOnce | Immediate full traffic switch          |

---

# 12. CodeDeploy with ECS

For ECS, CodeDeploy supports **blue/green deployments**.

The new application version runs using a new ECS task definition.

### Before deployment

```text
Application Load Balancer
          ↓
      Blue Target Group
          ↓
       ECS Tasks v1
```

### During deployment

```text
Application Load Balancer
          │
     ┌────┴────┐
     ↓         ↓
 Blue TG    Green TG
    v1          v2
```

The green target group runs the new ECS task definition.

### After deployment

```text
Application Load Balancer
          ↓
     Green Target Group
          ↓
       ECS Tasks v2
```

Traffic shifting can use:

* Linear
* Canary
* AllAtOnce

These strategies are similar to Lambda traffic shifting.

---

# 13. Complete CodeDeploy Architecture

```text
                CodeDeploy
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
       EC2        Lambda        ECS
        │           │            │
     Agent       Alias       Target Groups
        │           │            │
    In-place/    Traffic       Blue/Green
    Blue/Green   Shifting       Deployment
```

---

# ⭐ AWS DevOps Exam Cheat Sheet

| Question                                       | Answer                          |
| ---------------------------------------------- | ------------------------------- |
| What automates application deployment?         | CodeDeploy                      |
| Where can CodeDeploy deploy?                   | EC2, on-premises, Lambda, ECS   |
| What file defines deployment steps?            | `appspec.yml`                   |
| What performs deployment on EC2?               | CodeDeploy agent                |
| How to install agent automatically?            | Systems Manager                 |
| Where can EC2 application revisions be stored? | S3                              |
| What does AllAtOnce mean?                      | Update all instances together   |
| What does HalfAtATime mean?                    | Update half the instances       |
| What does OneAtATime mean?                     | Update one instance at a time   |
| What is blue/green?                            | New environment + traffic shift |
| What is Lambda linear?                         | Gradually increase traffic      |
| What is Lambda canary?                         | Small traffic first, then full  |
| What is AllAtOnce for Lambda?                  | Immediately switch traffic      |
| ECS deployment type supported by CodeDeploy?   | Blue/green                      |

## Final Mental Model

> **CodeDeploy = Deploy application versions safely and automatically.**

For EC2:

> Agent + `appspec.yml` + deployment configuration.

For Lambda:

> Alias + traffic shifting.

For ECS:

> New task definition + target groups + blue/green deployment.
