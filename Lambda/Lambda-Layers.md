# AWS Lambda Layers

An **AWS Lambda Layer** is a reusable package that contains libraries, dependencies, or common code that can be shared across multiple Lambda functions.

Think of a Lambda Layer as a shared library. Instead of copying the same code into every Lambda function, you create it once as a layer and attach it to the functions that need it.

## 1. Need for Lambda Layers

Imagine having three Lambda functions:

- **Lambda 1:** Processes customer data.
- **Lambda 2:** Processes order data.
- **Lambda 3:** Generates reports.

Suppose all three functions use the same Python library or common business logic.

**Without Lambda Layers:**

You would have to include the shared code separately in each Lambda deployment package.

**With Lambda Layers:**

You package the common code once and attach the same layer to all three functions.

<img width="664" height="653" alt="Lambda Layers architecture" src="https://github.com/user-attachments/assets/acb10a5b-9f36-4139-ba71-d268d4aba8d5" />

<img width="1536" height="1024" alt="Lambda Layers illustration" src="https://github.com/user-attachments/assets/b1055de9-f8eb-4d74-be41-aae311d39205" />

## 2. Creating a Custom Python Lambda Layer

### Step 1: Create the folder structure

For Python 3.12, create the following directory structure in your IDE:

```text
Lambda-Layer/
│
└── python/
    └── lib/
        └── python3.12/
            └── site-packages/
                └── my_module/
                    ├── __init__.py
                    └── my_function.py
```

### What does each folder mean?

| Folder/File | Purpose |
|---|---|
| `python/` | Required top-level directory in the Python layer ZIP |
| `lib/` | Contains Python libraries |
| `python3.12/` | Matches the Python runtime version |
| `site-packages/` | Stores Python packages and modules |
| `my_module/` | Our custom Python module |
| `__init__.py` | Marks the directory as a Python package |
| `my_function.py` | Contains our reusable function |

### Step 2: Write the reusable function

Inside `my_module/my_function.py`, write:

```python
def function_from_lambda_layer():
    return "Hello from my layer function"
```

This function represents common code that can be reused by multiple Lambda functions.

### Step 3: Create the ZIP file

Once the folder structure and Python code are ready, navigate to the `Lambda-Layer` project directory and run:

```bash
zip -r python.zip python/
```

**Important:** The ZIP file must contain the `python/` directory at its root.

The resulting structure should look like:

```text
python.zip
└── python/
    └── lib/
        └── python3.12/
            └── site-packages/
                └── my_module/
                    ├── __init__.py
                    └── my_function.py
```

## 3. Uploading the Layer to AWS Lambda

### Step 1: Open the Layers section

1. Open the **AWS Management Console**.
2. Navigate to **AWS Lambda**.
3. Select **Layers** from the left navigation menu.
4. Click **Create layer**.

<img width="800" height="543" alt="AWS Lambda Layers console" src="https://github.com/user-attachments/assets/879157a1-209f-4633-8af2-06aa16d6683a" />

### Step 2: Configure the layer

Enter the following details:

| Field | Value |
|---|---|
| Name | `my-custom-layer` |
| Description | Reusable Python functions |
| Upload | `python.zip` |
| Architecture | `x86_64` |
| Compatible runtime | Python 3.12 |

Click **Create**.

<img width="1400" height="819" alt="AWS Lambda Create Layer configuration" src="https://github.com/user-attachments/assets/72a62855-f262-420c-a860-39c395d481d8" />

### Step 3: Understand the layer version and ARN

After creating the layer, AWS generates the following details:

- **Layer name:** `my-custom-layer`
- **Version:** `1`
- **ARN:** A unique identifier for that specific layer version.

Example ARN:

```text
arn:aws:lambda:ap-south-1:123456789012:layer:my-custom-layer:1
```

The ARN identifies the layer and its version.

Every time you publish an updated layer, AWS creates a new version. Existing versions are immutable, meaning they cannot be modified in place.

## 4. Types of Layers Available When Adding a Layer

When attaching a layer to a Lambda function, AWS provides three options:

1. **AWS Layers:** Use layers published by AWS.
2. **Custom Layers:** Use layers created in your AWS account.
3. **Specify an ARN:** Attach a layer using its ARN.

## 5. Advantages and Limitations

### Advantages

- **Code reusability:** Share common functions across multiple Lambda functions.
- **Less duplication:** Avoid packaging the same dependencies repeatedly.
- **Easy maintenance:** Update shared code by publishing a new layer version.
- **Smaller function packages:** Keep dependencies separate from the function's own code.

### Limitations

- You must create and attach layers separately.
- The layer must be compatible with the Lambda runtime and architecture.
- Python packages containing compiled components must be compatible with Lambda's Linux environment.
- A Lambda function can have a maximum of **five layers** attached.
- Layers work with ZIP-based Lambda deployments, not directly with container-image deployments.

## 6. Key Takeaways

- Lambda Layers are reusable packages containing common code, libraries, and dependencies.
- Multiple Lambda functions can use the same layer.
- For Python layers, the ZIP archive must have a root-level `python/` directory.
- Each published layer has its own version and ARN.
- You must attach a layer to a Lambda function before using its code.

**Remember:** Lambda functions contain the code that performs a task, while Lambda Layers contain reusable code and dependencies that multiple functions can share.
