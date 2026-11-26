# Uploading to AWS Lambda

AWS Lambda code can be uploaded using two methods:

1. ZIP File Upload
2. Uploading Through an S3 Bucket

## 1. ZIP File Upload

### Step 1: Create a Lambda Function in Your IDE

- Create a Python function in your IDE that you want to deploy.
- Use the Lambda handler format:

**`lambda_handler(event, context)`**

**Example:**

```python
def lambda_handler(event, context):
    print("Lambda from my code")

    return {
        "statusCode": 200,
        "body": "Hello Lambda from my code"
    }
```

### Step 2: Create a ZIP File

- Save your Python file.
- Create a ZIP file containing the function code.
- Ensure that the Python file is at the root of the ZIP archive.

### Step 3: Upload the ZIP File to AWS Lambda

- Go to the **AWS Management Console**.
- Open **AWS Lambda** and select your Lambda function.
- Navigate to the **Code** tab.
- Click **Upload from** and select **.zip file**.
- Choose your ZIP file and upload it.

<img width="592" height="208" alt="image" src="https://github.com/user-attachments/assets/bbbbafa6-eda6-4fd5-981e-1bcb9ba02e61" />

<img width="712" height="357" alt="image" src="https://github.com/user-attachments/assets/67ac2e0a-7945-40b5-a548-9537dd5dbf7d" />

### Step 4: Configure the Handler

After uploading the ZIP file:

- Navigate to **Runtime settings**.
- Click **Edit**.
- Configure the handler using the format:

```text
filename.function_name
```

For example:

```text
lambda_function.lambda_handler
```

Here:
- `lambda_function` is the Python filename (without `.py`).
- `lambda_handler` is the function name.

<img width="528" height="298" alt="image" src="https://github.com/user-attachments/assets/d8325f34-5c03-4da6-85a6-f35bff6ebbb1" />

### Step 5: Test the Lambda Function

- Navigate to the **Test** tab.
- Create a test event if required.
- Click **Test** to execute the function.
- Check the execution result and logs to verify that the function is working.

<img width="613" height="356" alt="image" src="https://github.com/user-attachments/assets/c88a1a3d-b1a6-461d-bcce-2021075e4ec0" />

---

## 2. Uploading Through an S3 Bucket

This method involves uploading the ZIP file to an Amazon S3 bucket first and then providing its location to AWS Lambda.

### Step 1: Open the S3 Bucket

- Navigate to **Amazon S3** in the AWS Management Console.
- Open an existing bucket or create a new bucket.

<img width="629" height="425" alt="image" src="https://github.com/user-attachments/assets/a7d365c1-a8cc-4608-bae3-5b193314b20f" />

### Step 2: Upload the ZIP File

- Open the selected bucket.
- Click **Upload**.
- Select the ZIP file from your local system.
- Complete the upload.

<img width="611" height="311" alt="image" src="https://github.com/user-attachments/assets/66c29ca2-e81c-409b-9f6d-7955665584e6" />

### Step 3: Select the S3 Upload Option in Lambda

- Navigate to **AWS Lambda**.
- Select your Lambda function.
- Open the **Code** tab.
- Click **Upload from**.
- Select **Amazon S3 location**.

<img width="514" height="203" alt="image" src="https://github.com/user-attachments/assets/baf4020e-2780-499e-bdfd-6164ac3fa40b" />

### Step 4: Copy the S3 File Location

- Open the S3 bucket containing your uploaded ZIP file.
- Select the ZIP file.
- Copy its S3 URI or object location.

<img width="627" height="426" alt="image" src="https://github.com/user-attachments/assets/c9b511fa-8ce1-410a-bba2-d936a0c370e7" />

<img width="665" height="372" alt="image" src="https://github.com/user-attachments/assets/7013b25b-cefa-4379-9a6f-1201c8c03c1f" />

### Step 5: Upload the ZIP File to Lambda

- Return to the AWS Lambda console.
- Paste the copied S3 URI into the S3 location field.
- Click **Save**.
- Test the Lambda function to verify that the upload was successful.

<img width="829" height="419" alt="image" src="https://github.com/user-attachments/assets/322cd902-8178-4380-bcd1-bd6505fb0f72" />

---

## Quick Comparison

| Method | Description |
|---|---|
| ZIP File Upload | Upload the ZIP file directly from your local system to AWS Lambda. |
| S3 Bucket Upload | Upload the ZIP file to S3 first, then provide its location to Lambda. |

**Key Points:**
- A Lambda function must have a valid handler.
- The handler follows the `filename.function_name` format.
- The Python file should be at the root of the ZIP archive.
- You can upload code directly or through an S3 bucket.
- Test the Lambda function after uploading the code.
