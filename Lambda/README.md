# AWS Lambda

**1. What is AWS Lambda?**

It is a serverless compute service provided by AWS that allows you to run code without provisioning, managing, or maintaining servers.

Normally, when you deploy an application, you need a server(EC2 Instance) to execute your code.


With Lambda, you upload your code, cofigure when it should execute, and AWS takes care of the underlying infrastrucutre.

**No need to:**
- Create or manage EC2 instance.
- Install or maintain operating systems
- Configure server capacity manually
- Manage server availability and scaling.


  AWS automatically manages the infrastructure and executes your code when triggered.

**Important:** Serverless does not mean there are no servers. It means AWS manages the servers for you.


**2. How does AWS Lambda work?**
Lambda works on an event-driven execution model.

Your Lambda function gets executed whenever a particular event or trigger occurs.

For example:
- An image is uploaded to an S3 bucket.
- An HTTP request is received through API Gateway.
- A message arrives in an SQS queue.
- A scheduled event occurs.
- A DynamoDB table receives a particular change.
The event triggers Lambda, which executes your code.
Example: Image Processing Application
Suppose you are building an application where users upload images.

Without Lambda:
1. You provision an EC2 instance.
2. Deploy your image-processing application.
3. Keep the server running.
4. Monitor and maintain the server.
5. Process uploaded images.
