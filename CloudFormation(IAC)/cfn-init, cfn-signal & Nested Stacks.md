# CloudFormation: `cfn-init`, `cfn-signal`, Wait Conditions & Nested Stacks

 ## 53\. `cfn-init`

 `cfn-init` is used to **configure an EC2 instance** after it launches.

 ### Flow

```
CloudFormation
      ↓
EC2 launches
      ↓
cfn-init
      ↓
Install packages / create files / start services
```

 ### Example

```
Resources:

  MyInstance:
    Type: AWS::EC2::Instance
    Metadata:
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              httpd: []

          files:
            /var/www/html/index.html:
              content: |
                <h1>Hello from CloudFormation!</h1>

          commands:
            start-httpd:
              command: systemctl start httpd

    Properties:
      ImageId: ami-xxxxxxxx
      InstanceType: t2.micro

      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          /opt/aws/bin/cfn-init \
            --stack ${AWS::StackName} \
            --resource MyInstance \
            --region ${AWS::Region}
```

 ### Important

 The configuration is defined under:

```
Metadata:
  AWS::CloudFormation::Init:
```

 And User Data runs:

```
cfn-init
```

 ### Remember

 > **cfn-init = configure the EC2 instance**

---

 # 54\. `cfn-signal` & Wait Condition

 `cfn-signal` tells CloudFormation whether the EC2 configuration succeeded.

 ### Flow

```
EC2
 ↓
cfn-init
 ↓
Setup application
 ↓
cfn-signal
 ↓
CloudFormation
```

 Example:

```
Resources:

  MyInstance:
    Type: AWS::EC2::Instance

    Metadata:
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              httpd: []

    CreationPolicy:
      ResourceSignal:
        Count: 1
        Timeout: PT5M

    Properties:
      ImageId: ami-xxxxxxxx
      InstanceType: t2.micro

      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash

          /opt/aws/bin/cfn-init \
            --stack ${AWS::StackName} \
            --resource MyInstance \
            --region ${AWS::Region}

          /opt/aws/bin/cfn-signal \
            --stack ${AWS::StackName} \
            --resource MyInstance \
            --region ${AWS::Region} \
            --exit-code $?
```

 ### What's happening?

```
CreationPolicy
      ↓
CloudFormation waits
      ↓
EC2 runs cfn-init
      ↓
cfn-signal
      ↓
CloudFormation receives signal
```

 `Count: 1` means CloudFormation expects **one successful signal**.

 `Timeout: PT5M` means wait up to **5 minutes**.

---

 # 55\. `cfn-signal` Failure

 The important part is:

```
--exit-code $?
```

 `$?` represents the exit status of the previous command.

```
0       → Success
non-zero → Failure
```

 So:

```
cfn-init ...

cfn-signal --exit-code $?
```

 means:

 > "Tell CloudFormation whether cfn-init succeeded or failed."

 ### Example

 If `cfn-init` succeeds:

```
cfn-init → 0
           ↓
cfn-signal → SUCCESS
```

 If it fails:

```
cfn-init → 1
           ↓
cfn-signal → FAILURE
```

 CloudFormation can then fail/rollback the resource instead of incorrectly assuming everything worked.

---

 # 56\. Wait Conditions

 A Wait Condition is another mechanism for making CloudFormation wait for an external signal.

 Conceptually:

```
Resources:

  MyWaitCondition:
    Type: AWS::CloudFormation::WaitCondition
    Properties:
      Handle: !Ref MyWaitHandle
      Timeout: "300"
      Count: 1

  MyWaitHandle:
    Type: AWS::CloudFormation::WaitConditionHandle
```

 Flow:

```
CloudFormation
      ↓
WaitCondition
      ↓
Wait for signal
      ↓
External process signals
      ↓
Continue
```

 ### Important distinction

 For modern EC2 bootstrapping, you'll commonly see:

```
CreationPolicy
+
cfn-signal
```

 rather than relying on the older Wait Condition pattern.

---

 # 57\. Nested Stacks

 Nested stacks allow a **parent CloudFormation stack to create child stacks**.

 ### Parent

```
Resources:

  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://example-bucket.s3.amazonaws.com/network.yaml

  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://example-bucket.s3.amazonaws.com/app.yaml
```

 The parent template creates two child stacks:

```
Parent Stack
│
├── NetworkStack
│     └── VPC, Subnets
│
└── ApplicationStack
      └── EC2, ALB
```

---

 ## Child `network.yaml`

```
Resources:

  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
```

---

 ## Child `app.yaml`

```
Resources:

  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxxxxx
      InstanceType: t2.micro
```

 The parent controls both:

```
Parent
  │
  ├── network.yaml
  │
  └── app.yaml
```

---

 # ⭐ Final Cheat Sheet

```
cfn-init
   ↓
Configure EC2

cfn-signal
   ↓
Report success/failure to CloudFormation

CreationPolicy
   ↓
Tell CloudFormation to wait for signals

WaitCondition
   ↓
Explicitly wait for an external signal

Nested Stack
   ↓
Break a large CloudFormation template into smaller stacks
```

 ### Most important code to remember

```
cfn-init
```

 → **Do/configure the work**

```
cfn-signal
```

 → **Report the result**

```
CreationPolicy:
  ResourceSignal:
    Count: 1
    Timeout: PT5M
```

 → **Make CloudFormation wait for the signal**

```
Type: AWS::CloudFormation::Stack
```

 → **Create a nested stack**
