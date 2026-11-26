
# Chapter 5: Updating Lambda and Enabling the Function URL

**Duration: 20 minutes, 57 seconds**

## 1. What is an AWS Lambda Function URL?

A Lambda Function URL provides a dedicated HTTP(S) endpoint through which you can invoke your Lambda function directly.

Once enabled, you can access your Lambda function using its unique URL without needing to configure API Gateway.

**Basic flow:**

```text
Browser / Client
       |
       v
Lambda Function URL
       |
       v
AWS Lambda Function
       |
       v
Lambda Response
```

## 2. Enabling a Lambda Function URL

### Step 1: Open the Lambda Function

1. Navigate to the **AWS Management Console**.
2. Open **AWS Lambda**.
3. Select the Lambda function you created earlier.
4. Navigate to the **Configuration** tab.

### Step 2: Open the Function URL Settings

1. From the left-side menu, select **Function URL**.
2. Click **Create function URL**.

### Step 3: Select the Authentication Type

AWS Lambda provides two authentication options:

| Authentication Type | Description |
|---|---|
| NONE | Makes the Lambda Function URL publicly accessible. Anyone with the URL can invoke it. |
| AWS_IAM | Requires AWS IAM authentication. Requests must be properly signed using AWS credentials. |

### Option 1: NONE (Public Access)

1. Select **NONE** under Authentication type.
2. Review the permissions that AWS automatically configures.
3. Click **Save**.

When you select `NONE`, AWS creates the required resource-based permissions to allow public invocation of the function URL.

### Step 4: Access the Function URL

After saving:

1. Scroll down to the **Function URL** section.
2. Copy the generated URL.
3. Open a new browser tab.
4. Paste the URL and access it.

**Expected Output:**

```text
Hello Lambda from my code
```

The same Lambda function that was uploaded through the S3 bucket can now be executed directly through its Function URL.

## 3. Securing the Lambda Function URL Using AWS_IAM

Instead of allowing public access, you can configure the Function URL to require AWS IAM authentication.

### Step 1: Change the Authentication Type

1. Go back to the Lambda function's **Configuration → Function URL** section.
2. Click **Edit**.
3. Change the Authentication type from `NONE` to `AWS_IAM`.
4. Click **Save**.

The Function URL itself remains the same, but authentication is now required.

### Step 2: Try Accessing the URL Without Authentication

1. Copy the Function URL.
2. Open it in a new browser tab.

You will receive a **403 Forbidden** response because the request does not contain the required AWS authentication credentials.

This confirms that the Lambda function URL is no longer publicly accessible.

## 4. Testing an AWS_IAM Protected Function URL Using Postman

To access the protected Lambda Function URL, you can use Postman to send an authenticated request.

### Step 1: Open Postman

1. Open the Postman application.
2. Create a new HTTP request.
3. Copy and paste your Lambda Function URL into the request URL field.

### Step 2: Configure AWS Signature Authentication

1. Navigate to the **Authorization** tab.
2. Select **AWS Signature** from the authentication type dropdown.

Provide the following details:

| Field | Value |
|---|---|
| AccessKey | Your AWS Access Key ID |
| SecretKey | Your AWS Secret Access Key |
| AWS Region | The region where your Lambda function is deployed |
| Service Name | `lambda` |

**Note:** Use your own AWS credentials. Never expose or share your access key or secret key.

### Step 3: Send the Request

1. Verify that the AWS credentials and region are correct.
2. Ensure the service name is set to `lambda`.
3. Click **Send**.

### Step 4: Verify the Response

If the credentials and request signature are valid, Postman successfully invokes the Lambda function.

**Expected Output:**

```text
Hello Lambda from my code
```

This confirms that the Lambda function can be executed through its Function URL using AWS IAM authentication.

---

## 5. Quick Comparison: NONE vs AWS_IAM

| Feature | NONE | AWS_IAM |
|---|---|---|
| Authentication | Not required | Required |
| Accessibility | Public | Restricted to authorized IAM principals |
| Credentials | Not required | AWS credentials required |
| Browser access | Directly accessible | Requires a signed request |
| Security | Less restrictive | More secure when correctly configured |

## Key Takeaways

- A **Lambda Function URL** provides a unique HTTP(S) endpoint to invoke a Lambda function.
- You can create it from **Configuration → Function URL**.
- `NONE` allows public invocation without AWS authentication.
- `AWS_IAM` requires authenticated, signed requests.
- Postman supports AWS Signature authentication for testing protected Lambda Function URLs.
- The AWS service name used for signing Lambda Function URL requests is `lambda`.
- The Function URL remains the same when switching between `NONE` and `AWS_IAM`; only the authentication requirement changes.

- 
<img width="895" height="600" alt="image" src="https://github.com/user-attachments/assets/273a90b9-c80f-46ea-916d-5a270b39f9ff" />


<img width="368" height="250" alt="image" src="https://github.com/user-attachments/assets/08daf46a-7bd0-43ee-9d61-afd43bc201e4" />


<img width="411" height="196" alt="image" src="https://github.com/user-attachments/assets/a4c09a2c-ed7e-4e87-a80a-65edb1de49bd" />


<img width="429" height="246" alt="image" src="https://github.com/user-attachments/assets/9992da3b-5935-4506-9523-82577f2d7602" />
