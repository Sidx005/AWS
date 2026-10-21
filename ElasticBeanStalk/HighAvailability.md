# AWS Elastic Beanstalk — Creating a High Availability Production Environment

## 1. Objective

Create a second Elastic Beanstalk environment for Production while keeping the existing Development environment.

| Setting          | Development             | Production                         |
| ---------------- | ----------------------- | ---------------------------------- |
| Environment name | `My Application Dev`    | `MyApplication-prod`               |
| Platform         | Node.js                 | Node.js                            |
| Application code | Sample application      | Same sample application            |
| Preset           | Single Instance         | High Availability                  |
| Infrastructure   | Single EC2 instance     | Load balancer + Auto Scaling Group |
| Purpose          | Development and testing | Production-like deployment         |

Key idea: The application code is the same, but the infrastructure running it is different.

## 2. Create the Production Environment

1. Open AWS Elastic Beanstalk and select your existing application, `My Application`.
2. Choose Create environment and select Web server environment.
3. Set the environment name to `MyApplication-prod`.
4. Select the same managed platform, Node.js, and the sample application.
5. Choose the High Availability preset instead of Single Instance.
6. Select the existing service role and EC2 instance profile.
7. Review the configuration and create the environment. If the console's optional configuration flow behaves unexpectedly, use the instructor's simplified workflow: select the preset, proceed to review, verify the settings, and submit.

Note: Platform versions and console labels may differ from the lecture. Select a currently supported Node.js version rather than an obsolete version such as Node.js 12.

## 3. Important Configuration Options

### Networking

- Select the appropriate VPC.
- Configure the load balancer and instances across multiple Availability Zones (AZs).
- For a public load balancer with private application instances, place the load balancer in public subnets and instances in private subnets with suitable routing.
- The instances generally do not need public IP addresses when they are accessed through the load balancer.

### Database

Elastic Beanstalk can provision a database as part of an environment, but this creates a lifecycle consideration.

- The database may be deleted when the environment is terminated, depending on its configuration.
- For persistent production data, a separately managed database is often preferable.
- Take backups or snapshots before any destructive environment operation.

### Auto Scaling Group (ASG)

An ASG manages the number of EC2 instances running your application.

Example configuration:

| Setting          | Example                  |
| ---------------- | ------------------------ |
| Minimum capacity | 1                        |
| Maximum capacity | 4                        |
| Instance type    | `t3.micro`               |
| Scaling          | Dynamic scaling policies |

The values above are examples from the lesson, not universal production recommendations.

### Load Balancer

Elastic Beanstalk can configure the load balancer and its related resources.

- Application Load Balancer (ALB): Best suited to HTTP/HTTPS applications and Layer 7 routing.
- Network Load Balancer (NLB): Designed for Layer 4 traffic and use cases requiring its particular performance or networking characteristics.
- Dedicated load balancer: Used by one environment.
- Shared load balancer: Can reduce cost when supported by the chosen configuration and multiple environments can share it.

For a typical web application, an ALB is a common choice.

## 4. High Availability Architecture

Users / Internet

Application Load Balancer (ALB)

Public subnets across multiple AZs

Availability Zone A

EC2 Instance

Availability Zone B

EC2 Instance

Auto Scaling Group

Maintains capacity and adjusts instance count according to configured policies

Important: Selecting High Availability does not necessarily mean multiple EC2 instances are running immediately. With a minimum capacity of one, the environment may initially have only one healthy instance. Configure capacity and subnets appropriately for your availability requirements.

## 5. Verify the Resources Created

After the environment finishes launching, inspect the AWS services below.

[Running Web Applications on Amazon EC2 Spot Instances | AWS Compute Blog](https://images.openai.com/static-rsc-4/JOk0nlCI7QPa2Cr55c7aT7dUIfya0Ed7ncMHvxP_UTVXe7C5LhV06lClULVxAY4vn16ePhplzHeNGSK6EfHoleZOoNBzSfhhtaJPbP3fcDQMUwtoWbb9TRfh9HYQMqI4rDfDbiWfhI3wx2itAxciXQHlF7KItan0B50Trsukz88?purpose=inline)

1\. Load Balancer

Open EC2 → Load Balancers. Confirm that the load balancer exists and check its Availability Zones and status.

[Guided Lab: Creating Your First Network Load Balancer - Tutorials Dojo](https://images.openai.com/static-rsc-4/hDQY8oGigO6alMJtV2kjEizk79t1T6ppY6KU-TSVwg1TjSPHU2aJhzmHrGR6KeOqGst-Q8YusczTCYz2OHdZoZs_0PO9RaBGYvqbfgCULfDrRdR9DE_aoRk4-2ti6fdKjGz-yGlQ0UcH5eChYXQZqb4lSw4plFv2SvfQkhgez7g?purpose=inline)

2\. Target Group

Open Target Groups and inspect registered targets. A healthy target indicates that the instance is passing the configured health checks.

[Event-driven Active Directory domain join with Amazon EventBridge | Microsoft Workloads on AWS](https://images.openai.com/static-rsc-4/x-yTBkWguDGZYSTaoYzHjQcv5OXAYED1CDLXi6tG9aZwuCwIDuWClId5ZKCEJKgqdThZmzMwtducZKrIDp2kVc_4IQvn0wAG55ToeUX7kDaQWUVgnVxd4uxZzXeGzuJpNdqIR7Ly2EnXk-2OdHGnlLMf3kVxFXOnl9lK0jiQkfY?purpose=inline)

3\. Auto Scaling Group

Open EC2 → Auto Scaling Groups. Find the group associated with the Production environment and check minimum, maximum, desired capacity, and instance health.

[aws security group types, purpose and default port | Medium](https://images.openai.com/static-rsc-4/J2qIlGu7tNLmsjQEiK9rOgBwIk0EB0jPm0z3CY_sNcUAJAoVhNpzGyPKswDPV774ZyavvSX150tKggFOaSymJ2m0fHHtcLTOlUs2Jp0AP0Yi6I7Ah4W1Uqkpu3LJ4294S-dsu7PmfgOe8ATfktq2Y3tc_3nnucLBnl9q118HnuE?purpose=inline)

4\. Security Groups

Verify that the load balancer accepts the intended incoming traffic and that application instances accept traffic from the load balancer's security group.

### Security group traffic flow

```
Internet
   │
   │ HTTP/HTTPS — only intended public ports
   ▼
Load Balancer Security Group
   │
   │ Application traffic from the LB security group
   ▼
EC2 Instance Security Group
   │
   ▼
Node.js Application
```

Avoid exposing the application instances directly to the internet when the architecture is intended to use a public load balancer and private instances.

## 6. Automatic Scaling Policies

Elastic Beanstalk can configure scaling policies for the ASG.

For example:

- Minimum capacity: Lowest number of instances to maintain.
- Maximum capacity: Upper limit on the number of instances.
- Scaling metric: A metric such as average CPU utilization, depending on the configured policy.
- Scale out: Add instances when the scaling policy's conditions are met.
- Scale in: Remove instances when conditions permit.

The exact thresholds, cooldowns, and policies depend on the environment configuration. Verify them in the Auto Scaling Group rather than assuming every environment has identical settings.

## 7. Enhanced Health Reporting and Updates

Other configuration options include:

- Enhanced health reporting: Provides more detailed information about environment and instance health.
- Managed platform updates: Helps maintain the platform using configured update settings.
- CloudWatch Logs: Can stream application and instance logs to CloudWatch.
- Email notifications: Can notify you about selected environment events.
- Rolling updates: Can update instances in batches, depending on the chosen deployment configuration.

These settings can affect availability, maintenance behavior, monitoring, and cost. Review them before using the environment for real production traffic.

## 8. Development vs Production: Final Architecture

```
My Application
│
├── Development Environment
│   ├── Single EC2 instance
│   └── Used for testing
│
└── Production Environment
    ├── Load Balancer
    ├── Auto Scaling Group
    ├── EC2 instance(s)
    └── Health checks and scaling policies
```

Both environments belong to the same Elastic Beanstalk application, but each has its own environment resources and configuration.

## 9. Key Takeaways

- Elastic Beanstalk can deploy the same application code into different environments.
- The environment preset influences the initial infrastructure configuration.
- High Availability commonly involves a load balancer and an Auto Scaling Group.
- An ALB distributes HTTP/HTTPS requests to healthy targets.
- An ASG manages instance capacity; it is not itself a load balancer.
- Multiple Availability Zones improve resilience only when the environment and its dependencies are configured appropriately.
- Security groups control which traffic can reach the load balancer and application instances.
- A database created with an environment may have lifecycle implications when the environment is deleted.
- Always inspect the actual resources, scaling settings, and health status after deployment.
- Terminate practice environments when finished and check for remaining billable resources.

### Official AWS documentation

- [Create and manage Elastic Beanstalk environments](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.environments.html?utm_source=chatgpt.com)
- [Elastic Beanstalk environment configuration](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options.html?utm_source=chatgpt.com)
- [Elastic Beanstalk health monitoring](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/health-enhanced.html?utm_source=chatgpt.com)
- [Elastic Beanstalk deployment policies](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.rollingupdates.html?utm_source=chatgpt.com)


# DEMO

<img width="503" height="349" alt="image" src="https://github.com/user-attachments/assets/ffddeba4-d136-4ee4-9bdb-db95649db214" />



<img width="471" height="356" alt="image" src="https://github.com/user-attachments/assets/e369bed6-32bc-4269-beab-7254a4a4b09e" />



<img width="540" height="267" alt="image" src="https://github.com/user-attachments/assets/69340137-938a-4e4c-961b-d6454fe4fe39" />


<img width="568" height="258" alt="image" src="https://github.com/user-attachments/assets/97cf3272-9437-4f3f-a9de-b2cd7e7bd2e5" />


<img width="239" height="124" alt="image" src="https://github.com/user-attachments/assets/270cdc98-5510-448e-81c2-51fc68911b9d" />



<img width="631" height="313" alt="image" src="https://github.com/user-attachments/assets/0bdea01e-4c80-4f00-b7dc-ed2356a042c0" />


##EwDT WNCRINENR


<img width="788" height="434" alt="image" src="https://github.com/user-attachments/assets/e9b17537-7132-40a2-94b8-98265f120965" />



<img width="738" height="185" alt="image" src="https://github.com/user-attachments/assets/b9f0ab91-8317-4064-995b-8906089b90f9" />
