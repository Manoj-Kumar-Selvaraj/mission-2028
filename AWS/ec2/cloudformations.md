## CloudFormation Basics

### 1. What is CloudFormation?

**AWS CloudFormation is Infrastructure as Code (IaC).**

You define AWS infrastructure in a template, then CloudFormation creates and manages it.

```text
Template
   ↓
CloudFormation
   ↓
AWS Resources
```

Example resources:

- EC2
- VPC
- Security Groups
- IAM
- RDS
- S3
- ALB
- ECS

### 2. Template

A **CloudFormation template** is usually written in:

- YAML
- JSON

Example:

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
```

Think:

> Template = infrastructure definition

### 3. Stack

A **Stack** is the actual deployed set of resources created from a template.

```text
Template
   ↓
Create Stack
   ↓
EC2 + SG + S3 + IAM
```

Think:

> Template = blueprint  
> Stack = deployed infrastructure

### 4. Main Template Sections

The most important sections are:

```text
Parameters
Resources
Outputs
Mappings
Conditions
```

#### Parameters
Inputs to the template.

Example:

```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
```

#### Resources
Actual AWS resources to create.

```yaml
Resources:
  WebServer:
    Type: AWS::EC2::Instance
```

This is the **only mandatory main section** in a normal template.

#### Outputs
Values returned after deployment.

Example:

```yaml
Outputs:
  InstanceId:
    Value: !Ref WebServer
```

### 5. Intrinsic Functions

CloudFormation has built-in functions.

Important ones:

```text
!Ref
!GetAtt
!Sub
!Join
```

Example:

```yaml
InstanceType: !Ref InstanceType
```

`!Ref` refers to another parameter/resource.

### 6. Update

If you modify the template:

```text
Template v1
   ↓
Change AMI / SG / Instance Type
   ↓
Update Stack
```

CloudFormation calculates what needs to:

- update in place
- replace
- create
- delete

### 7. Change Set

A **Change Set** shows what CloudFormation plans to modify before you actually apply the update.

```text
Template Change
      ↓
Change Set
      ↓
Review
      ↓
Execute
```

Very useful in production.

### 8. Rollback

If stack creation/update fails, CloudFormation can roll back changes.

Example:

```text
Create EC2 ✅
Create SG ✅
Create RDS ❌
     ↓
Rollback
```

### 9. CloudFormation vs Terraform

Very simple distinction:

```text
CloudFormation
→ AWS-native IaC

Terraform
→ Multi-cloud IaC
```

CloudFormation uses AWS resource types like:

```text
AWS::EC2::Instance
AWS::S3::Bucket
AWS::IAM::Role
```

Terraform uses provider resources like:

```text
aws_instance
aws_s3_bucket
aws_iam_role
```

## Quick Memory

```text
Template
→ infrastructure definition

Stack
→ deployed resources

Parameters
→ inputs

Resources
→ what to create

Outputs
→ useful returned values

Change Set
→ preview changes

Rollback
→ recover from failed deployment
```

### Interview one-liner

> **AWS CloudFormation is AWS's native Infrastructure-as-Code service where infrastructure is defined in YAML or JSON templates and deployed as stacks, with support for parameters, dependencies, updates, change sets and rollback.**