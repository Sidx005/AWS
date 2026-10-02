AWS IAM Roles allow users, applications and services to securely access resources without requiring permanent credentials. Unlike IAM users, which are assigned to specific individuals, roles can be temporarily used by AWS services like EC2 and Lambda or even by external entities. This approach helps businesses manage permissions more efficiently while ensuring security best practices, such as granting access only when necessary.

# Reasons to use IAM Roles
IAM Roles eliminate the need to hardcode AWS access keys and secret keys within your applications. Instead, roles are assumed temporarily by trusted entities (like EC2, Lambda, or EKS), allowing them to perform actions based on defined permissions.

## Eg:
<ul>
<li>A Lambda function can use a role to access S3 or DynamoDB.</li>
<li>An EC2 instance can assume a role to read from S3 without embedding credentials.</li>
<li>EKS pods can assume roles to securely interact with other AWS services.</li>
</ul>


## Types of IAM Roles

<img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/a54bdb5e-833b-40c6-a121-4d2ffc406f8b" />


# AWS IAM Roles — Study Notes

## 1. Service Roles

Service roles are IAM roles created by administrators to allow AWS services such as EC2, Lambda, or CloudFormation to access AWS resources on your behalf.

A service role consists of two important policies:

* **Trust Policy:** Defines which service or identity can assume the role.
* **Permissions Policy:** Defines what actions the role is allowed to perform.

**Example:** An EC2 instance needs to read files from an S3 bucket. An administrator creates an IAM role that allows EC2 to assume the role and grants it permission to read the required S3 objects.

## 2. Service-Linked Roles

Service-linked roles are special IAM roles linked to specific AWS services. AWS defines the required permissions and manages the role's lifecycle.

* Created automatically by the service when needed, or through an authorized setup process.
* Contain permissions required for the service to perform its tasks.
* Have a trust relationship with the associated AWS service.
* Their permissions are managed by AWS and generally cannot be edited directly by users.

**Example:** An AWS service may use a service-linked role to monitor resources or manage infrastructure on your behalf.

## 3. Cross-Account Roles

Cross-account roles allow identities from one AWS account to access resources in another AWS account without requiring duplicate IAM users.

They use two key components:

* **Trust Policy:** Specifies which external account or identity is allowed to assume the role.
* **Permissions Policy:** Defines what the assumed role can do in the target account.

**Example:** A developer in Account A needs read access to an S3 bucket in Account B. Account B creates a role that trusts the appropriate identity from Account A and grants the required S3 permissions.

## 4. Federated Roles

Federated roles allow users authenticated through an external identity provider to access AWS without creating a separate IAM user for each person.

Common identity providers include:

* Microsoft Entra ID (formerly Azure AD)
* Okta
* Google identity services
* Other SAML or OpenID Connect (OIDC) providers

AWS Security Token Service (STS) can provide temporary credentials after successful federation.

**Example:** Employees sign in using their company's identity provider and receive access to AWS according to the IAM role assigned to them.

## 5. Web Identity (OIDC) Roles

Web identity roles allow applications or users authenticated through an OpenID Connect (OIDC) identity provider to obtain temporary AWS credentials.

* Authentication uses an OIDC token.
* AWS verifies the token according to the configured trust relationship.
* STS provides temporary credentials through `AssumeRoleWithWebIdentity`.
* The role's permissions determine which AWS resources can be accessed.

**Example:** A mobile application uses Amazon Cognito or another supported OIDC identity provider to obtain temporary credentials for accessing permitted AWS resources.

## 6. Custom Roles (User-Assumable Roles)

Custom IAM roles can be created to grant specific permissions to IAM users, other roles, or other trusted identities.

They are useful when users need temporary access to permissions beyond their normal access.

* The trust policy determines who can assume the role.
* The permissions policy determines what the role can do.
* STS provides temporary credentials when the role is successfully assumed.

**Example:** A developer normally has read-only access but temporarily assumes a deployment role to publish an application.

## 7. Quick Comparison

| Role Type                  | Main Purpose                                                   | Example                                    |
| -------------------------- | -------------------------------------------------------------- | ------------------------------------------ |
| Service Role               | Allows an AWS service to access resources                      | EC2 reading from S3                        |
| Service-Linked Role        | Allows a specific AWS service to perform service-related tasks | AWS service managing resources             |
| Cross-Account Role         | Grants access across AWS accounts                              | Account A accessing resources in Account B |
| Federated Role             | Grants AWS access to externally authenticated users            | Employees signing in through Entra ID      |
| Web Identity (OIDC) Role   | Grants temporary access using OIDC tokens                      | Mobile application accessing AWS           |
| Custom User-Assumable Role | Grants specific temporary permissions to trusted identities    | Developer assuming a deployment role       |

## 8. Key Concepts to Remember

* **Trust policy = Who can assume the role?**
* **Permissions policy = What can the role do?**
* **IAM user = A long-term identity in AWS IAM.**
* **IAM role = An identity with permissions that trusted entities can assume.**
* **Temporary credentials = Short-lived credentials commonly issued through AWS STS.**
* **Cross-account access = Access between different AWS accounts.**
* **Federation = Using an external identity provider to authenticate users.**

**Important:** These categories can overlap. For example, a cross-account role can also be a custom role, and federated access commonly works by assuming an IAM role.
