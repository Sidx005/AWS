# AWS Lambda Environment Variables

1. What are Environment Variables?

Environment variables allow you to store configuration values separately from your Lambda function's source code.

These values can be accessed inside your Lambda function whenever it executes.

For example, instead of hardcoding a configuration value in your Python code, you can define it as an environment variable in AWS Lambda.

Example:

Variable Name

Value

my_environment_variable

random value from env

This variable is configured in the Lambda console and accessed from the Python code.

2. Creating Environment Variables in AWS Lambda

Step 1: Create a Lambda Function

Open the AWS Management Console.

Navigate to AWS Lambda.

Click Create function.

Enter the function name: lambda-with-environment-variable.

Select the runtime: Python 3.12.

Select architecture: x86_64.

Click Create function.

Step 2: Configure Environment Variables

Open the newly created Lambda function.

Navigate to the Configuration tab.

Select Environment variables from the left-side menu.

Click Edit.

Click Add environment variable.

Enter the variable name and value.

Example:

Key:   my_environment_variable
Value: random value from env

Click Save.

3. Accessing Environment Variables in Python

Python provides the os library to access environment variables.

Example Lambda Function:

import os

def lambda_handler(event, context):

    # Fetch the environment variable
    my_var = os.environ.get("my_environment_variable")

    return {
        "statusCode": 200,
        "body": f"AWS Lambda with environment variable: {my_var}"
    }

Understanding the Code

import os: Imports Python's OS library.

os.environ.get(): Retrieves the value of an environment variable.

my_environment_variable: Name of the variable configured in AWS Lambda.

my_var: Stores the retrieved value.

Important: The environment variable name in your Python code must match the name configured in the Lambda console.

4. Deploy and Test the Lambda Function

Step 1: Deploy

Open the Code tab.

Add the Python code.

Click Deploy to apply the changes.

Step 2: Create a Test Event

Click Test.

Create a new test event.

Enter an event name, such as test-with-env.

Keep the default event JSON because the function does not require any input from the event.

Click Save.

Step 3: Execute the Function

Click Test to execute the Lambda function.

Expected Response:

{
    "statusCode": 200,
    "body": "AWS Lambda with environment variable: random value from env"
}

The response confirms that the Lambda function successfully retrieved the value configured in its environment variables.

5. Important Points

You can create multiple environment variables for a single Lambda function.

Environment variables help separate configuration from application code.

Python uses os.environ.get("VARIABLE_NAME") to retrieve environment variables.

If a variable does not exist, os.environ.get() returns None unless a default value is provided.

Environment variables are configured separately for each Lambda function.

Avoid storing sensitive information directly in environment variables when a dedicated service such as AWS Secrets Manager or Systems Manager Parameter Store is more appropriate.


<img width="473" height="219" alt="image" src="https://github.com/user-attachments/assets/7a3e0896-6db2-4ca2-b46c-9aa02ebf7994" />


<img width="533" height="318" alt="image" src="https://github.com/user-attachments/assets/84999de5-9a00-4657-8f52-2d84c7ec62b1" />


<img width="582" height="374" alt="image" src="https://github.com/user-attachments/assets/836fe3d2-2c6c-416c-a67a-95ef1d455cfd" />



<img width="602" height="388" alt="image" src="https://github.com/user-attachments/assets/47c4aa41-d671-4ba1-a357-0052c1ab6d81" />


<img width="649" height="459" alt="image" src="https://github.com/user-attachments/assets/c6301cbb-3e85-44d5-9036-cb46f7ad9d73" />


<img width="664" height="317" alt="image" src="https://github.com/user-attachments/assets/3f9a3c9a-c620-4384-bb00-3398cc49f224" />


<img width="583" height="422" alt="image" src="https://github.com/user-attachments/assets/9c5dc3d4-bf25-4769-aa59-b8166238d55a" />
