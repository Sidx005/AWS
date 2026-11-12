# AWS Elastic Beanstalk — Deployment Strategies

## 1. Overview

Elastic Beanstalk provides multiple deployment strategies for updating an application from one version to another.

For example, we currently have four EC2 instances running Version 1 (V1) and want to deploy Version 2 (V2).

The deployment strategy determines:

- Whether the application experiences downtime.
- Whether the application runs below, at, or above its normal capacity.
- Whether additional EC2 instances are required.
- How quickly a failed deployment can be rolled back.
- Whether traffic is gradually shifted to the new version.

There are **six important deployment strategies**:

| Deployment strategy | Downtime | Additional cost | Main idea |
|---|---|---|---|
| All at Once | Yes | No | Update all instances together |
| Rolling | Possible reduced capacity | No | Update instances in batches |
| Rolling with Additional Batch | No capacity reduction | Yes | Add temporary instances and update in batches |
| Immutable | No | High | Deploy to entirely new instances |
| Blue/Green | No (during switch, if configured correctly) | Yes | Create a separate environment |
| Traffic Splitting | No (during deployment, if configured correctly) | Yes | Send a percentage of traffic to the new version |

---

## 2. All at Once Deployment

**Fastest deployment**

All instances are updated simultaneously.

Example: Four EC2 instances are running V1. We want to deploy V2.

### Deployment process

**Step 1 — Initial state: V1**

```text
V1          V1          V1          V1
EC2 1       EC2 2       EC2 3       EC2 4
```

**Step 2 — All instances stop serving the application**

```text
Stopped     Stopped     Stopped     Stopped
EC2 1       EC2 2       EC2 3       EC2 4
```

**Step 3 — Deploy V2 to all instances**

```text
V2          V2          V2          V2
EC2 1       EC2 2       EC2 3       EC2 4
```

### Characteristics

- All instances are updated at once.
- Application becomes unavailable during the update.
- Fastest deployment method.
- No additional EC2 instances are required.
- No additional infrastructure cost for deployment capacity.

**Best suited for:** Development and testing environments where downtime is acceptable.

**Remember:** Fastest deployment, but causes downtime.

---

## 3. Rolling Deployment

Rolling deployment updates instances in smaller groups called **batches or buckets**.

The application continues running on the instances that have not yet been updated.

Example: Four EC2 instances with a bucket size of two.

### Deployment process

**Step 1 — Initial state**

```text
V1          V1          V1          V1
EC2 1       EC2 2       EC2 3       EC2 4
```

**Step 2 — First batch of two is updated**

```text
V2          V2          V1          V1
EC2 1       EC2 2       EC2 3       EC2 4
```

The remaining V1 instances continue serving traffic.

**Step 3 — Second batch is updated**

```text
V2          V2          V2          V2
EC2 1       EC2 2       EC2 3       EC2 4
```

Deployment complete.

### Characteristics

- Instances are updated in batches.
- The next batch is updated after the previous batch is healthy.
- Application capacity temporarily decreases during deployment.
- Both V1 and V2 can run simultaneously.
- No additional EC2 instances are needed.
- Deployment may take longer when the bucket size is small.

### What is bucket size?

Bucket size is the number of instances updated together in one batch.

For example:

| Total instances | Bucket size | Number of batches |
|---:|---:|---:|
| 4 | 2 | 2 |
| 10 | 2 | 5 |
| 100 | 10 | 10 |

**Best suited for:** Applications where some temporary reduction in capacity is acceptable.

**Remember:** Same instances, batch-by-batch update, temporarily reduced capacity.

---

## 4. Rolling with Additional Batch

**Maintains original capacity**

This is similar to rolling deployment, but Elastic Beanstalk launches additional EC2 instances before updating the original ones.

This ensures that the application maintains its original serving capacity during deployment.

Example: Four original instances, with a bucket size of two.

### Deployment process

**Step 1 — Initial state: Four V1 instances**

```text
Original instances

V1          V1          V1          V1
EC2 1       EC2 2       EC2 3       EC2 4
```

**Step 2 — Launch two additional V2 instances**

```text
Original instances

V1          V1          V1          V1

Additional instances

V2          V2
```

Six instances are now available.

**Step 3 — Update the original instances in batches**

```text
Original instances

V2          V2          V2          V2

Additional instances

V2          V2
```

**Step 4 — Remove the additional instances**

```text
V2          V2          V2          V2
EC2 1       EC2 2       EC2 3       EC2 4
```

Deployment complete.

### Characteristics

- Temporary additional instances are launched.
- Original instances are updated in batches.
- The original serving capacity is maintained.
- V1 and V2 run simultaneously during deployment.
- Additional instances are terminated after deployment.
- Slightly higher cost due to temporary extra capacity.

**Best suited for:** Production applications that cannot afford reduced capacity during deployment.

**Remember:** Rolling + extra instances = maintain capacity, with additional cost.

---

## 5. Immutable Deployment

**Fast rollback**

Immutable deployment launches a completely new set of EC2 instances in a temporary Auto Scaling Group (ASG).

The existing instances continue running V1 while the new instances receive V2.

### Deployment process

**Step 1 — Existing Auto Scaling Group**

```text
Original ASG

V1          V1          V1
```

**Step 2 — Create temporary ASG**

```text
Original ASG              Temporary ASG

V1          V1          V1          V2
                                      Test instance
```

The first new instance is checked before the remaining new instances are launched.

**Step 3 — New instances pass health checks**

```text
Original ASG              Temporary ASG

V1          V1          V1          V2
                                      V2
                                      V2
```

**Step 4 — V2 replaces V1**

```text
Final serving capacity

V2          V2          V2

Old instances removed
```

### Characteristics

- New instances are created instead of updating existing instances.
- A temporary ASG is used.
- New instances are health-checked before replacing the old ones.
- Old instances continue serving traffic during the deployment.
- Temporary capacity can approximately double the instance count.
- Rollback is quick: the new deployment can be discarded while the original version remains available.

**Best suited for:** Production environments that require a safer deployment and quick rollback.

**Remember:** New instances + temporary ASG + quick rollback + higher cost.

---

## 6. Blue/Green Deployment

Blue/Green deployment creates an entirely separate Elastic Beanstalk environment.

Unlike the previous strategies, which deploy within the same environment, Blue/Green uses two independent environments.

- **Blue:** Existing production environment running V1.
- **Green:** New environment running V2.

### Deployment process

**Step 1 — Two separate environments**

```text
BLUE                         GREEN

V1                           V2
Existing environment         New environment
```

**Step 2 — Test the Green environment**

Validate V2 independently before switching production traffic.

**Step 3 — Switch production traffic**

Before switch:

```text
Production traffic → BLUE (V1)
```

After switch:

```text
Production traffic → GREEN (V2)
```

**Step 4 — Retire Blue when safe**

Green becomes production. Blue can be retained temporarily for rollback or terminated when no longer required.

### How do we switch traffic?

Possible approaches include:

1. **Elastic Beanstalk Swap Environment URLs:** Swap the CNAMEs of two environments.
2. **Amazon Route 53 weighted routing:** Send a chosen percentage of DNS-routed traffic to each environment.

A weighted DNS policy can help with gradual exposure, although DNS caching means it is not necessarily an exact per-request traffic split.

### Characteristics

- Two separate Elastic Beanstalk environments.
- New version can be tested independently.
- Old environment remains available for rollback.
- Requires additional infrastructure while both environments exist.
- Switching can be performed through URL swapping or DNS routing.
- Blue/Green is generally a deployment architecture or approach, rather than one of the built-in Elastic Beanstalk deployment policy settings.

**Best suited for:** Major production releases where independent testing and environment-level rollback are important.

**Remember:** Two separate environments, independent testing, traffic switch.

---

## 7. Traffic Splitting Deployment

**Automated Canary Deployment**

Traffic splitting is designed for Canary testing.

Canary testing means exposing a new application version to a small percentage of real traffic before sending all traffic to it.

Elastic Beanstalk deploys the new version to a temporary ASG and uses an Application Load Balancer to split traffic.

### Deployment process

**Step 1 — Existing and temporary ASGs**

```text
Main ASG                   Temporary ASG

V1                         V2
V1                         V2
V1                         V2
```

**Step 2 — Split traffic through ALB**

```text
             Application Load Balancer
                       |
             ---------------------
             |                   |
           90%                 10%
             |                   |
          V1 traffic          V2 traffic
             |                   |
          Main ASG           Temporary ASG
```

**Step 3 — Monitor new deployment**

Monitor deployment health and configured application metrics.

**Step 4 — Decide the outcome**

```text
             Deployment health
                    |
             -----------------
             |               |
          Healthy         Unhealthy
             |               |
       Complete          Rollback
       deployment
```

### What happens if V2 fails?

Elastic Beanstalk can automatically roll back the deployment based on the configured health checks and deployment settings.

The load balancer stops sending traffic to the temporary V2 deployment, while V1 continues serving users.

This makes rollback very quick.

### Characteristics

- Uses a temporary ASG.
- New deployment receives only a percentage of traffic.
- Main ASG continues serving most traffic.
- Deployment health is monitored.
- Supports automated rollback when configured conditions indicate failure.
- Additional temporary capacity means additional cost.
- Once successful, the new instances replace the old instances.

**Best suited for:** Production deployments where you want to test a new version with a small percentage of real traffic.

**Remember:** Canary testing = Traffic Splitting.

---

## 8. Complete Comparison — Exam Revision

| Feature | All at Once | Rolling | Rolling + Additional Batch | Immutable | Blue/Green | Traffic Splitting |
|---|---|---|---|---|---|---|
| Updates existing instances? | Yes | Yes | Yes | No | No | No, initially |
| New instances created? | No | No | Yes | Yes | Yes | Yes |
| Separate environment? | No | No | No | No | Yes | No |
| Downtime | Yes | Possible | No, under normal configuration | No, under normal configuration | No, if configured correctly | No, if configured correctly |
| Capacity during deployment | Unavailable briefly | Reduced | Maintained | Extra capacity | Both environments available | Extra capacity |
| Additional cost | No | No | Yes | High | Yes | Yes |
| Rollback | Redeploy old version | Redeploy old version | Redeploy old version | Fast | Switch back to Blue | Automated traffic rollback |
| Main use | Development | Cost-conscious updates | Production capacity | Safe production updates | Environment-level releases | Canary testing |

*Cost and downtime are relative to the deployment process. Actual behavior depends on environment configuration, capacity, health checks, and deployment settings.*

---

## 9. AWS Exam Scenarios — Which Deployment Should You Choose?

| Scenario | Correct choice | Why? |
|---|---|---|
| Need the fastest deployment and can accept downtime | All at Once | Updates all instances together |
| Want to update instances in batches without extra EC2 capacity | Rolling | Uses existing instances |
| Cannot afford reduced application capacity during rolling updates | Rolling with Additional Batch | Adds temporary instances |
| Want new instances and fast rollback | Immutable | Original instances remain until new ones pass health checks |
| Want a separate environment for testing before switching production | Blue/Green | Independent environments |
| Want to send 10% traffic to a new version for canary testing | Traffic Splitting | Routes a percentage of traffic to the temporary deployment |
| New version fails during canary testing and automatic rollback is required | Traffic Splitting | Can automatically stop sending traffic to the new version |

---

## 10. Quick Memory Tricks

- **All at Once:** Everyone stops, everyone updates.
- **Rolling:** Update a few, then move to the next batch.
- **Rolling with Additional Batch:** Add extra capacity first, then update.
- **Immutable:** New instances, test, replace old instances.
- **Blue/Green:** Two separate environments, switch traffic.
- **Traffic Splitting:** Send a small percentage of traffic to the new version.

### Most important exam distinctions

1. Fastest: **All at Once**.
2. Reduced capacity: **Rolling**.
3. Maintains original capacity: **Rolling with Additional Batch**.
4. Fast rollback using new instances: **Immutable**.
5. Separate environments: **Blue/Green**.
6. Canary testing: **Traffic Splitting**.
7. Extra cost: Rolling with Additional Batch, Immutable, Blue/Green, and Traffic Splitting all require additional resources during deployment.

---

## 11. Official AWS Documentation

For the detailed deployment comparison and configuration options:

- [Elastic Beanstalk deployment policies](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.rollingupdates.html)
- [Deployment configuration options](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options-general.html)

**One-line summary:** Elastic Beanstalk deployment policies determine how application versions are updated, how much capacity remains available, how traffic is handled, and how failures are rolled back.
