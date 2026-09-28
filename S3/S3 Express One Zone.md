# AWS S3 Express One Zone — Study Notes

## 1. Overview

**Amazon S3 Express One Zone** is a high-performance S3 storage class designed for applications that require very low latency and high request rates.

Unlike standard S3 buckets, it uses a special type of bucket called a **Directory Bucket**.

### Key Characteristics

- High-performance object storage.
- Stores data in a **single Availability Zone (AZ)**.
- Supports hundreds of thousands of requests per second.
- Provides single-digit millisecond latency.
- Designed for latency-sensitive and data-intensive workloads.
- Allows compute and storage resources to be located in the same AZ.

## 2. How S3 Express One Zone Works

### Standard S3 vs S3 Express One Zone

| Feature | S3 Standard | S3 Express One Zone |
|---|---|---|
| Bucket type | General-purpose bucket | Directory bucket |
| Storage location | Multiple Availability Zones | Single Availability Zone |
| Performance | High performance | Higher request rates and lower latency for suitable workloads |
| Latency | Millisecond-level | Single-digit millisecond latency |
| Availability Zone resilience | Designed for AZ failures | No multi-AZ resilience within the storage class |
| Main purpose | General-purpose object storage | High-performance, latency-sensitive workloads |

**Important:** S3 Express One Zone is not designed as a replacement for S3 Standard in every use case. It is intended for workloads where performance and latency are especially important.

## 3. Directory Buckets

S3 Express One Zone uses a special bucket type called a **Directory Bucket**.

Characteristics:
- Data is stored in one selected Availability Zone.
- You choose the Availability Zone when creating the directory bucket.
- It differs from a standard S3 general-purpose bucket.
- It is designed to support high request rates and low-latency access.

## 4. Performance and Cost

According to the lecture:

- Up to approximately **10 times the performance** of S3 Standard for suitable workloads.
- Potentially around **50% lower cost** for the relevant request-intensive workloads.
- Supports hundreds of thousands of requests per second.
- Provides single-digit millisecond latency.

**Note:** These are indicative comparisons from the lecture. Actual performance and cost savings depend on workload, region, request patterns, and pricing.

## 5. Availability and Durability

Since S3 Express One Zone stores objects in a single Availability Zone:

- It does not provide the same multi-AZ resilience as S3 Standard.
- An outage affecting the selected AZ can affect access to the stored objects.
- It is better suited to data that can be recreated or workloads with appropriate recovery strategies.

**Key distinction:**
- **Durability:** How reliably data is preserved.
- **Availability:** How readily data can be accessed.

Do not assume that single-AZ storage automatically means poor durability; durability and availability are separate properties.

## 6. Why Use S3 Express One Zone?

The main advantage is the ability to **co-locate storage and compute resources in the same Availability Zone**.

Benefits:
- Reduced latency.
- High request throughput.
- Potentially reduced networking costs.
- Improved performance for data-intensive applications.

## 7. Use Cases

S3 Express One Zone is suitable for:

- AI and Machine Learning training
- Financial modeling
- Media processing
- High-performance computing (HPC)
- Latency-sensitive applications
- Data-intensive applications
- Large-scale analytics workloads

## 8. AWS Service Integrations

S3 Express One Zone can be used with AWS data and compute services, including:

- Amazon SageMaker Model Training
- Amazon Athena
- Amazon EMR
- AWS Glue

These integrations can help accelerate data processing and analytics workloads.

## 9. S3 Express One Zone vs Other Storage Classes

| Requirement | Suitable Storage Class |
|---|---|
| Frequently accessed, general-purpose data | S3 Standard |
| Infrequently accessed data | S3 Standard-IA |
| Re-creatable data in one AZ | S3 One Zone-IA |
| High-performance, low-latency workloads | S3 Express One Zone |
| Rarely accessed data requiring instant retrieval | S3 Glacier Instant Retrieval |
| Long-term archival data | S3 Glacier Deep Archive |
| Unpredictable access patterns | S3 Intelligent-Tiering |

## 10. Exam-Focused Points to Remember

- **S3 Express One Zone = High performance + Single AZ.**
- Uses **Directory Buckets**, unlike standard S3 general-purpose buckets.
- Designed for single-digit millisecond latency.
- Supports very high request rates.
- Enables compute and storage co-location within the same AZ.
- Does not provide the same multi-AZ resilience as S3 Standard.
- Suitable for AI/ML training, analytics, media processing, financial modeling, and HPC.
- Choose it when performance and latency are priorities and the workload can accommodate single-AZ storage.

## 11. Easy Memory Trick

| Keyword in the Question | Think Of |
|---|---|
| High performance | S3 Express One Zone |
| Directory bucket | S3 Express One Zone |
| Single Availability Zone | S3 Express One Zone |
| Single-digit millisecond latency | S3 Express One Zone |
| Compute and storage co-location | S3 Express One Zone |
| General-purpose, multi-AZ object storage | S3 Standard |
| Lowest-cost long-term archive | S3 Glacier Deep Archive |
