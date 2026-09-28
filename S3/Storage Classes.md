# AWS S3 Storage Classes — Study Notes

## 1. Overview

Amazon S3 provides different storage classes depending on:
- How frequently data is accessed
- How quickly data needs to be retrieved
- Storage cost
- Availability requirements
- Data retention requirements

The storage class can be selected when uploading an object, changed manually later, or changed automatically using **S3 Lifecycle Rules**.

## 2. Durability vs Availability

| Concept | Meaning | S3 Example |
|---|---|---|
| **Durability** | How reliably data is preserved without being lost | 99.999999999% (11 nines) |
| **Availability** | How readily a service is accessible when requested | S3 Standard: 99.99% |

- **Durability:** S3 is designed to provide 99.999999999% durability across its storage classes.
- **Availability:** Depends on the storage class and its design.
- Durability and availability are different concepts. A highly durable object can still be temporarily unavailable.

## 3. Types of S3 Storage Classes

### 3.1 S3 Standard — General Purpose

**Best for:** Frequently accessed data.

- Availability: **99.99%**
- Low latency and high throughput.
- Designed to withstand the loss of two Availability Zones simultaneously.
- No minimum storage duration.
- Suitable for frequently accessed data.

**Use cases:**
- Big data analytics
- Mobile applications
- Gaming applications
- Content distribution
- Websites

### 3.2 S3 Standard-IA (Infrequent Access)

**Best for:** Infrequently accessed data that still requires rapid access.

- Availability: **99.9%**
- Data is stored across multiple Availability Zones.
- Lower storage cost than S3 Standard.
- Retrieval charges apply.
- Minimum storage duration: **30 days**.

**Use cases:**
- Backups
- Disaster recovery
- Older files that still require rapid access

### 3.3 S3 One Zone-IA

**Best for:** Infrequently accessed data that can be recreated.

- Availability: **99.5%**
- Stores data in a single Availability Zone.
- Lower cost than Standard-IA.
- Data can be lost if that Availability Zone is destroyed.
- Minimum storage duration: **30 days**.

**Use cases:**
- Re-creatable data
- Secondary copies of backups
- Derived data

### 3.4 S3 Glacier Instant Retrieval

**Best for:** Rarely accessed archival data that requires immediate retrieval.

- Retrieval time: Milliseconds.
- Lower storage cost for rarely accessed data.
- Retrieval charges apply.
- Minimum storage duration: **90 days**.

**Use cases:**
- Medical images
- Archived documents
- Backups that might occasionally be needed immediately

### 3.5 S3 Glacier Flexible Retrieval

**Best for:** Archival data where retrieval can take minutes to hours.

| Retrieval Option | Typical Retrieval Time |
|---|---|
| Expedited | 1–5 minutes |
| Standard | 3–5 hours |
| Bulk | 5–12 hours |

- Minimum storage duration: **90 days**.
- Retrieval charges may apply.
- Bulk retrieval is typically free of retrieval charges.

**Use cases:**
- Long-term backups
- Archives
- Data that is rarely needed urgently

### 3.6 S3 Glacier Deep Archive

**Best for:** Long-term archival storage.

- Lowest-cost S3 storage class for long-term archival.
- Retrieval takes hours.
- Standard retrieval: Approximately **12 hours**.
- Bulk retrieval: Approximately **48 hours**.
- Minimum storage duration: **180 days**.

**Use cases:**
- Regulatory records
- Compliance archives
- Historical records
- Data retained for years

### 3.7 S3 Intelligent-Tiering

**Best for:** Data with unknown, changing, or unpredictable access patterns.

- Automatically moves eligible objects between access tiers based on usage.
- No retrieval charges for the standard Intelligent-Tiering access tiers.
- A small monthly monitoring and automation fee applies.
- Optional archive tiers can be configured.

**Use cases:**
- Data with unpredictable access patterns
- Applications where manual storage-class management is inconvenient
- Cost optimization

## 4. S3 Intelligent-Tiering Access Tiers

| Tier | When It Is Used | Retrieval |
|---|---|---|
| Frequent Access | Default tier for new objects | Milliseconds |
| Infrequent Access | No access for 30 consecutive days | Milliseconds |
| Archive Instant Access | No access for 90 consecutive days | Milliseconds |
| Archive Access | Optional; configurable archive threshold, typically 90–730 days | Minutes to hours, depending on retrieval |
| Deep Archive Access | Optional; configurable threshold, typically 180–730 days | Hours |

**Important:**
- Archive Access and Deep Archive Access are optional tiers.
- These tiers must be configured.
- Objects in archive tiers need to be restored before access, unlike the instant-access tiers.

## 5. Quick Comparison Table

| Storage Class | Access Pattern | Retrieval Speed | Minimum Storage Duration | Availability |
|---|---|---|---|---|
| S3 Standard | Frequent | Milliseconds | None | 99.99% |
| S3 Standard-IA | Infrequent | Milliseconds | 30 days | 99.9% |
| S3 One Zone-IA | Infrequent, recreatable data | Milliseconds | 30 days | 99.5% |
| Glacier Instant Retrieval | Rare, but immediate access needed | Milliseconds | 90 days | 99.9% |
| Glacier Flexible Retrieval | Archive | Minutes to hours | 90 days | 99.99% |
| Glacier Deep Archive | Very long-term archive | Hours | 180 days | 99.99% |
| Intelligent-Tiering | Unpredictable access | Milliseconds for instant-access tiers | Depends on tier | Depends on tier |

*Note: Availability figures and retrieval times are typical AWS design figures for study purposes. Actual costs and performance depend on configuration and retrieval option.*

## 6. How to Choose the Right Storage Class

| Requirement | Recommended Storage Class |
|---|---|
| Frequently accessed data | S3 Standard |
| Occasionally accessed, rapid retrieval | Standard-IA |
| Infrequently accessed, single AZ, recreatable data | One Zone-IA |
| Rarely accessed, immediate retrieval required | Glacier Instant Retrieval |
| Archival data with flexible waiting time | Glacier Flexible Retrieval |
| Years of retention with low storage cost | Glacier Deep Archive |
| Unknown or changing access patterns | Intelligent-Tiering |

## 7. S3 Lifecycle Rules

S3 Lifecycle configurations can automatically transition objects between storage classes or expire objects when they are no longer needed.

### Example Lifecycle Policy

- After 30 days → Transition to Standard-IA.
- After 90 days → Transition to Glacier Flexible Retrieval.
- After 365 days → Transition to Glacier Deep Archive.
- After a defined retention period → Expire the object.

**Important considerations:**
- Lifecycle transitions must satisfy destination-class rules.
- Not every transition is permitted directly.
- Transition costs and early-deletion charges can apply.
- Lifecycle rules help reduce storage costs by moving older data to cheaper classes.

## 8. Exam-Focused Points to Remember

- **11 nines durability:** The key S3 durability figure.
- **Standard:** Frequently accessed data.
- **Standard-IA:** Infrequently accessed data, but rapid retrieval is needed.
- **One Zone-IA:** Data can be recreated; stored in one Availability Zone.
- **Glacier Instant Retrieval:** Archive with millisecond retrieval.
- **Glacier Flexible Retrieval:** Choose between expedited, standard, and bulk retrieval.
- **Glacier Deep Archive:** Lowest-cost long-term archive; slowest retrieval.
- **Intelligent-Tiering:** Automatically optimizes costs based on access patterns.
- **Lifecycle Rules:** Automate storage-class transitions and expiration.
- **Retrieval charges and minimum storage durations:** Important when selecting a storage class.

## 9. Easy Memory Trick

| If the Question Says... | Think Of... |
|---|---|
| Frequently accessed | Standard |
| Infrequent but immediately available | Standard-IA |
| Infrequent, single AZ, recreatable | One Zone-IA |
| Rarely accessed, instant retrieval | Glacier Instant Retrieval |
| Archival, flexible waiting time | Glacier Flexible Retrieval |
| Years of retention, lowest storage cost | Glacier Deep Archive |
| Unknown or changing access patterns | Intelligent-Tiering |




# Hands On

## Create a bucket
<img width="813" height="663" alt="image" src="https://github.com/user-attachments/assets/7c10f79a-408e-4799-8028-364fbce2e20c" />


## Uploading Img
<img width="761" height="552" alt="image" src="https://github.com/user-attachments/assets/969a568b-57a9-43fa-ad14-98cdf469e43f" />

## Selecting Tiers
<img width="967" height="623" alt="image" src="https://github.com/user-attachments/assets/eafc0358-4c2e-4c6c-9fe7-14f41683c7e5" />
