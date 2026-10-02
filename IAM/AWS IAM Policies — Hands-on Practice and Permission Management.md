# AWS IAM Policies — Hands-on Practice and Permission Management

**AWS Certified Cloud Practitioner · Study Notes · October 2026**

## 1. Understanding Inherited Permissions

In this hands-on exercise, the IAM user `Stephane` belongs to the `admins` group, which has the `AdministratorAccess` policy attached.

Therefore, Stephane inherits administrative permissions from the group.

```text
IAM Group: admins
    |
    |-- AdministratorAccess
    |
    └── IAM User: Stephane
            └── Inherits administrator permissions
```

When a policy is attached to a group, its users receive the permissions granted by that policy, subject to other applicable authorization controls.

## 2. What Happens When a User Is Removed from a Group?

The instructor removes Stephane from the `admins` group.

As a result:

* Stephane no longer inherits `AdministratorAccess` from that group.
* Stephane loses the administrative permissions provided by that membership.
* Actions that require those permissions begin returning `AccessDenied` errors.

For example, attempting to list IAM users might return an error related to:

```text
iam:ListUsers
```

This happens because Stephane no longer has permission to perform that API operation.

**Important:** Removing a user from one group does not remove permissions granted through other groups, directly attached policies, or other applicable authorization mechanisms.

## 3. Giving the User Read-Only IAM Access

Instead of adding Stephane back to the administrators group, the instructor attaches the AWS-managed policy `IAMReadOnlyAccess` directly to the user.

### Steps

1. Open **IAM → Users**.
2. Select `Stephane`.
3. Open the permissions section.
4. Choose **Add permissions**.
5. Select **Attach policies directly**.
6. Search for `IAMReadOnlyAccess`.
7. Attach the policy.

### Result

Stephane can perform permitted IAM read operations, such as listing users and viewing user groups, but cannot perform IAM operations that require write permissions, such as creating a group.

| Action              | Expected result                   |
| ------------------- | --------------------------------- |
| List IAM users      | Allowed, if covered by the policy |
| View IAM groups     | Allowed, if covered by the policy |
| Create an IAM group | Denied by this read-only policy   |
| Delete an IAM user  | Denied by this read-only policy   |

The exact access depends on the policy statements and any other applicable permissions or restrictions.

**Key takeaway:** Give users only the permissions required for their responsibilities.

## 4. Adding a User to Multiple Groups

The instructor next creates a `developers` group, adds Stephane to it, and attaches an example AWS-managed policy called `AlexaForBusiness`.

Stephane is then added back to the `admins` group.

The resulting configuration is:

```text
IAM User: Stephane
    |
    |-- Direct policy
    |      └── IAMReadOnlyAccess
    |
    |-- Group: admins
    |      └── AdministratorAccess
    |
    └-- Group: developers
           └── AlexaForBusiness
```

The user receives permissions from multiple sources.

| Permission source  | Policy              | Purpose                                   |
| ------------------ | ------------------- | ----------------------------------------- |
| Directly attached  | IAMReadOnlyAccess   | IAM read operations covered by the policy |
| `admins` group     | AdministratorAccess | Broad administrative access               |
| `developers` group | AlexaForBusiness    | Permissions defined by that policy        |

Since Stephane is a member of `admins`, the user inherits the group's administrative permissions again.

The additional policies do not replace one another. They contribute to the user's effective permissions, subject to explicit denies, permission boundaries, organization policies, and other applicable controls.

**Important security lesson:** Adding a restrictive policy does not automatically restrict permissions granted by another policy. For example, attaching `IAMReadOnlyAccess` does not cancel the broader permissions from `AdministratorAccess`.

## 5. Understanding AdministratorAccess

Navigate to **IAM → Policies → AdministratorAccess**.

AWS-managed policies can be inspected to understand which actions they permit.

The `AdministratorAccess` policy grants broad administrative access across AWS services.

Its central permission statement is conceptually equivalent to:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

### Understanding the JSON

| Element         | Meaning                                           |
| --------------- | ------------------------------------------------- |
| `Version`       | IAM policy language version                       |
| `Effect: Allow` | Allows the specified actions                      |
| `Action: "*"`   | Matches all actions covered by this statement     |
| `Resource: "*"` | Applies to all resources covered by the statement |

The combination of `Action: "*"` and `Resource: "*"` provides broad administrative permissions, subject to applicable authorization controls.

The number of services shown in the AWS console can change as AWS introduces new services and operations.

## 6. Understanding IAMReadOnlyAccess

The instructor also inspects the AWS-managed `IAMReadOnlyAccess` policy.

This policy grants read-oriented permissions for IAM operations covered by its statements.

Its JSON may include action patterns such as:

```json
{
  "Effect": "Allow",
  "Action": [
    "iam:Get*",
    "iam:List*"
  ],
  "Resource": "*"
}
```

This is an illustrative example of wildcard patterns, not a complete copy of the current AWS-managed policy.

### What Does `Get*` Mean?

`Get*` matches action names that begin with `Get`, such as:

* `iam:GetUser`
* `iam:GetGroup`

### What Does `List*` Mean?

`List*` matches action names that begin with `List`, such as:

* `iam:ListUsers`
* `iam:ListGroups`

Wildcards reduce the need to specify every matching action individually.

**Important:** A wildcard matches action names; it does not automatically mean that every possible operation is allowed. The policy must include the relevant action pattern, and the statement must apply to the requested resource and conditions.

## 7. Creating a Custom IAM Policy

AWS allows you to create your own policies using either the visual editor or the JSON editor.

### Option A: Visual Editor

The visual editor allows you to configure permissions through the console without writing the entire JSON document manually.

Typical steps:

1. Navigate to **IAM → Policies**.
2. Select **Create policy**.
3. Open the visual editor.
4. Choose the service, such as IAM.
5. Select the required actions.
6. Specify the resources.
7. Configure optional conditions.
8. Review the policy.
9. Give it a name and create it.

### Option B: JSON Editor

The JSON editor allows you to define the policy directly.

For example, the instructor creates a custom policy named `MyIAMPermissions` to allow only two actions.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:ListUsers",
        "iam:GetUser"
      ],
      "Resource": "*"
    }
  ]
}
```

### Explanation

| Element    | Value           | Meaning                                             |
| ---------- | --------------- | --------------------------------------------------- |
| `Effect`   | `Allow`         | Permits the listed actions                          |
| `Action`   | `iam:ListUsers` | Allows listing IAM users                            |
| `Action`   | `iam:GetUser`   | Allows retrieving IAM user information              |
| `Resource` | `*`             | Applies to all resources supported by these actions |

For some IAM list operations, the applicable policy requires `"Resource": "*"`, because those operations do not support resource-level permissions.

**Important:** Creating a policy does not automatically grant it to a user. You must attach it to a user, group, or role.

## 8. What Is Least Privilege?

The **principle of least privilege** means granting only the permissions required to complete a task.

For example, if an employee only needs to view IAM users, they should not automatically receive full administrative access.

| Requirement                 | Permission approach                                 |
| --------------------------- | --------------------------------------------------- |
| View IAM users              | Grant the necessary IAM read operations             |
| Create IAM groups           | Grant the required IAM write operations             |
| Administer all AWS services | Use broad administrative access only when justified |

This reduces the risk of accidental changes and unauthorized access.

## 9. Cleaning Up the Hands-on Exercise

At the end of the demonstration, the instructor restores the original configuration.

1. Delete the temporary `developers` group.
2. Remove `IAMReadOnlyAccess` from Stephane.
3. Keep Stephane in the `admins` group.
4. Verify that Stephane inherits `AdministratorAccess`.

The final configuration becomes:

```text
IAM Group: admins
    |
    |-- AdministratorAccess
    |
    └── IAM User: Stephane
```

Stephane once again has the administrative permissions granted by the `admins` group.

## 10. Key Concepts for the AWS Exam

| Concept               | Remember This                                      |
| --------------------- | -------------------------------------------------- |
| Inherited permissions | Users receive permissions from their groups        |
| Direct permissions    | Policies attached directly to a user               |
| Multiple groups       | Permissions can come from several groups           |
| `AccessDenied`        | The requested operation is not authorized          |
| `AdministratorAccess` | Broad administrative permissions                   |
| `IAMReadOnlyAccess`   | Read-oriented IAM permissions                      |
| `Action: "*"`         | Matches all action names covered by the statement  |
| `Resource: "*"`       | Matches all resources within the statement's scope |
| `Get*`                | Matches action names beginning with `Get`          |
| `List*`               | Matches action names beginning with `List`         |
| Visual editor         | Creates policies through a graphical interface     |
| JSON editor           | Creates policies by editing JSON                   |
| Least privilege       | Grant only the permissions required                |
| Explicit deny         | Overrides an applicable allow                      |

## 11. Final Takeaway

This hands-on demonstrates how IAM permissions change as group membership and policy attachments change.

Remember these core points:

* Removing a user from a group removes the permissions inherited from that group.
* Policies can be attached directly to users or through groups.
* A user can inherit permissions from multiple groups.
* `AdministratorAccess` grants broad administrative permissions.
* `IAMReadOnlyAccess` allows covered read operations, not IAM write operations.
* Wildcards such as `Get*` and `List*` can match multiple API actions.
* Custom policies can be created using the visual editor or JSON editor.
* Attaching a read-only policy does not cancel permissions granted by an administrator policy.
* Always follow the principle of least privilege when designing access controls.





## Images

**Step 1: Remove User from Admin Group (Lose All Access)**
<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/bafdae58-ec3a-40f3-ab67-d186d9cc8122" />


<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/e7b1635d-b668-4a5b-8c64-e7237f39d923" />
<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/43b46aad-9e92-4395-8c42-4962ce3f1519" />


**Step 2: Attach IAMReadOnlyAccess Directly to User**
<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/cb906bba-9238-4c33-b905-35d5fe5ef33c" />

<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/f00d6315-ed2e-4881-b52d-5f8a2268e5de" />


<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/c2a7690e-ad4c-4d5a-aa14-c306b2d9d9a6" />
<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/23e8d7a4-e25f-4ae5-a5ea-a5c346e56bfa" />


**View All Permissions on Mino**
<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/8330dd69-d34e-466c-9e09-8081b98ce22e" />



**Create Your Own Policy (Visual Editor)**
<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/0ce348b6-48be-488f-a1c6-88d045ef2c42" />
