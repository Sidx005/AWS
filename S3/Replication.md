# AWS S3 Replication — Hands-on Notes

## 1. Objective

Configure S3 replication to automatically copy objects from a source bucket to a destination bucket.

There are two types of replication:

* CRR (Cross-Region Replication): Copies objects to a bucket in a different AWS Region.

* SRR (Same-Region Replication): Copies objects to another bucket in the same AWS Region.

In this hands-on, we'll configure Cross-Region Replication (CRR).

## 2. Create the Source Bucket (Origin)

Create the first S3 bucket.

Example configuration:

```
Bucket Name: s3-stephane-bucket-origin-v2
Region:      EU West 1 (Ireland)
Versioning:  Enabled
```

Steps:

1. Open the Amazon S3 console.

2. Click Create bucket.

3. Enter a globally unique bucket name.

4. Select the desired AWS Region.

5. Enable Bucket Versioning.

6. Click Create bucket.

## 3. Create the Destination Bucket (Replica)

Create a second bucket to receive the replicated objects.

Example configuration:

```
Bucket Name: s3-stephane-bucket-replica-v2
Region:      US East 1 (N. Virginia)
Versioning:  Enabled
```

Steps:

1. Click Create bucket.

2. Enter a unique bucket name.

3. Select the destination Region.

4. Enable Bucket Versioning.

5. Click Create bucket.

Important: Versioning must be enabled on both the source and destination buckets for replication to work.

## 4. Upload an Initial File

Open the source bucket and upload:

```
beach.jpeg
```

At this point:

```
Source Bucket                 Destination Bucket
(origin-v2)                    (replica-v2)

beach.jpeg                     Empty
```

The file will not be replicated automatically because the replication rule has not been configured yet.

## 5. Create a Replication Rule

Open the source bucket.

Navigate to:

```
S3
 → Source Bucket
 → Management
 → Replication rules
 → Create replication rule
```

Configure the rule as follows:

|
Setting

|

Value

|
| --- | --- |
|

Rule name

|

`demo-replication-rule`

|
|

Status

|

Enabled

|
|

Rule scope

|

Apply to all objects

|
|

Destination

|

Bucket in this account

|
|

Destination bucket

|

`s3-stephane-bucket-replica-v2`

|
|

Destination Region

|

US East 1

|
|

IAM role

|

Create a new IAM role

|

Save the replication rule.

### What does the IAM role do?

S3 needs permission to read objects from the source bucket and replicate them to the destination bucket. The IAM role grants the required permissions to the S3 replication process.

## 6. Replication of Existing Objects

After creating the rule, AWS may ask:

Do you want to replicate existing objects?

There are two choices:

* No: Replicate eligible objects uploaded after the rule is configured.

* Yes: Initiate replication of eligible existing objects using S3 Batch Replication.

For this hands-on, select No.

Therefore, the previously uploaded `beach.jpeg` will not be replicated automatically by this newly created rule.

## 7. Upload a New File After Replication Is Enabled

Now upload a new file to the source bucket:

```
coffee.jpeg
```

The replication process will look like this:

```
Source Bucket (EU West 1)
          │
          │ Upload coffee.jpeg
          ▼
    Replication Rule
          │
          ▼
Destination Bucket (US East 1)
          │
          └── coffee.jpeg
```

Replication is asynchronous, so the destination object may take a short time to appear.

Refresh the destination bucket to verify the replicated file.

## 8. Verify Object Version IDs

Enable Show versions in both buckets.

Example:

Source Bucket

Object

Version ID

coffee.jpeg

GBK...

S3 Replication

Destination Bucket

Object

Version ID

coffee.jpeg

GBK...

The replicated object retains its source version ID, allowing you to identify the corresponding object version in the destination bucket.

## 9. Replicate an Existing Object by Uploading It Again

The original `beach.jpeg` wasn't replicated because it was uploaded before the rule was configured.

To replicate it in this demonstration:

1. Return to the source bucket.

2. Upload `beach.jpeg` again.

3. S3 creates a new object version because versioning is enabled.

4. The replication rule processes the eligible new version.

5. Refresh the destination bucket and check for `beach.jpeg`.

   Source Bucket
   │
   ├── beach.jpeg (old version)
   └── beach.jpeg (new version)
   │
   ▼
   Replication Rule
   │
   ▼
   Destination Bucket
   └── beach.jpeg (replicated new version)

## 10. Important Concepts to Remember

|
Concept

|

Explanation

|
| --- | --- |
|

Source bucket

|

Bucket where objects are uploaded

|
|

Destination bucket

|

Bucket receiving replicated objects

|
|

Versioning

|

Must be enabled on both buckets

|
|

Replication rule

|

Defines which objects should be replicated and where

|
|

IAM role

|

Grants S3 permission to perform replication

|
|

CRR

|

Replication across different Regions

|
|

SRR

|

Replication within the same Region

|
|

Existing objects

|

Require Batch Replication to replicate previously existing versions

|
|

Replication timing

|

Asynchronous; objects may not appear immediately

|
|

Version IDs

|

Replicated object versions retain their source version IDs

|

### Quick mental model

```
Create Source Bucket
        ↓
Enable Versioning
        ↓
Create Destination Bucket
        ↓
Enable Versioning
        ↓
Create Replication Rule
        ↓
Configure IAM Role
        ↓
Upload New Object
        ↓
S3 Automatically Replicates It
        ↓
Verify in Destination Bucket
```

Key takeaway: S3 replication automatically copies eligible object versions between buckets. Versioning must be enabled on both buckets, and a replication rule with the required IAM permissions must be configured on the source bucket.



<img width="1088" height="603" alt="image" src="https://github.com/user-attachments/assets/aff436cb-ebd6-467c-bd3f-490e4751afa4" />


**Create another bucket**
<img width="830" height="637" alt="image" src="https://github.com/user-attachments/assets/bdd1eb54-2574-4fab-a3b0-5d4340a7262f" />


**Replication Rules**
<img width="1000" height="517" alt="image" src="https://github.com/user-attachments/assets/d0d13c76-c1b9-45f6-a09c-fa4bfa9ec530" />
<img width="335" height="116" alt="image" src="https://github.com/user-attachments/assets/f678d545-e36b-41d0-a65a-64069ce3435c" />


<img width="790" height="497" alt="image" src="https://github.com/user-attachments/assets/8c6ace88-469e-4307-8d8d-875a67f88402" />

<img width="815" height="527" alt="image" src="https://github.com/user-attachments/assets/e6aac2fb-41b3-4ad0-bf4f-9cedd5a540b0" />


<img width="796" height="481" alt="image" src="https://github.com/user-attachments/assets/a3065cdb-6290-4e61-b627-08b3a88afbaa" />

**Choose a target bucket**
<img width="816" height="525" alt="image" src="https://github.com/user-attachments/assets/84afb44b-4ef8-4c83-8478-f0d18153ff11" />


<img width="707" height="383" alt="image" src="https://github.com/user-attachments/assets/17289e86-2cb9-4c1f-bd45-632d931e6955" />

<img width="804" height="473" alt="image" src="https://github.com/user-attachments/assets/64859271-f832-4c62-8748-b156ba87ab14" />


<img width="703" height="352" alt="image" src="https://github.com/user-attachments/assets/23a3feac-d35b-44f2-a1a5-159557d6c63b" />

<img width="1033" height="453" alt="image" src="https://github.com/user-attachments/assets/6a768261-6d4a-4d5c-92e3-8e3a2661a822" />

**Upload jpeg file**
<img width="940" height="423" alt="image" src="https://github.com/user-attachments/assets/35c177a4-f378-4694-8b90-2c049f788a9d" />

<img width="1032" height="557" alt="image" src="https://github.com/user-attachments/assets/c34af86a-30bd-4c6a-b3f2-356ec12de83f" />


<img width="1041" height="553" alt="image" src="https://github.com/user-attachments/assets/9ae02580-d4ad-4ad3-a72f-72741c04aca5" />
