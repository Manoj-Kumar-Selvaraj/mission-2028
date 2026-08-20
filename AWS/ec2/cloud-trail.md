## CloudTrail Basics

### 1. What is CloudTrail?

**AWS CloudTrail records activity performed in your AWS account.**

It tells you things like:

- **Who** performed an action
- **What** API action was performed
- **When** it happened
- **From where** it was performed
- **Which resource** was affected
- Whether the request **succeeded or failed**

CloudTrail records actions performed through the AWS Console, CLI, SDK/API, IAM identities, and AWS services. 

Think:

```text
User / Role / AWS Service
        ↓
     API Call
        ↓
    CloudTrail
        ↓
   Audit Event
```

---

# 2. Example

Suppose someone terminates an EC2 instance:

```bash
aws ec2 terminate-instances \
  --instance-ids i-123456789
```

CloudTrail records an event containing information similar to:

```text
eventName       = TerminateInstances
eventSource     = ec2.amazonaws.com
userIdentity    = Manoj / assumed-role
sourceIPAddress = 10.20.30.40
eventTime       = 2026-08-14T10:00:00Z
resources       = i-123456789
```

So during an incident you can answer:

> **Who terminated this EC2 instance?**

---

# 3. Event

An **event** is one recorded activity.

Example:

```text
StartInstances
StopInstances
TerminateInstances
CreateSecurityGroup
AuthorizeSecurityGroupIngress
CreateUser
DeleteUser
CreateCluster
```

One API operation generally creates a CloudTrail event describing that activity. 

---

# 4. Important Event Fields

These are the fields worth remembering.

### `eventTime`

When the action occurred.

```text
eventTime = 2026-08-14T09:30:00Z
```

---

### `eventName`

The actual API operation.

```text
eventName = TerminateInstances
```

Another example:

```text
eventName = AuthorizeSecurityGroupIngress
```

---

### `eventSource`

Which AWS service handled the API request.

```text
ec2.amazonaws.com
iam.amazonaws.com
eks.amazonaws.com
```

---

### `userIdentity`

Who performed the request.

Could represent:

```text
IAM User
Assumed IAM Role
AWS Service
Federated User
Root
```

If temporary STS credentials were used, `userIdentity` also contains information about how those credentials were obtained. 

---

### `sourceIPAddress`

Where the request originated.

Example:

```text
203.0.113.20
```

Useful during:

- Security investigations
- Suspicious API activity
- Auditing

---

### `userAgent`

What client made the request.

Examples could indicate:

```text
AWS Console
AWS CLI
Terraform/AWS SDK
Another AWS service
```

---

### `requestParameters`

What was supplied to the API request.

Example:

```text
Security Group:
sg-12345

Port:
22

CIDR:
0.0.0.0/0
```

This field becomes extremely useful when investigating security-group changes.

---

### `responseElements`

What AWS returned from the API operation.

---

### `errorCode` / `errorMessage`

If the operation failed:

```text
errorCode = AccessDenied
```

Useful for troubleshooting permission problems.

CloudTrail event records contain these and other structured fields describing the request, identity, resources and outcome. 

---

# 5. Management Events

Management events are **control-plane operations**.

Think:

> **Changes to AWS infrastructure/configuration.**

Examples:

```text
RunInstances
TerminateInstances

CreateSecurityGroup
AuthorizeSecurityGroupIngress

CreateUser
DeleteUser

CreateRole
AttachRolePolicy

CreateCluster
DeleteCluster
```

These are the events we will mainly use for our **EC2, Security Group, IAM and EKS investigations**.

---

# 6. Data Events

Data events represent operations **inside/on the resource**, often called data-plane operations.

Examples include high-volume resource-level actions such as access to supported S3 objects or other resources that expose data events.

Unlike management events, **data events are not logged by trails/event data stores by default** and can incur additional charges. 

Easy memory:

```text
Management Event
→ Manage/configure the RESOURCE

Data Event
→ Operate on the DATA inside the resource
```

---

# 7. CloudTrail Event History

Even if you haven't created a trail, AWS provides:

> **CloudTrail Event History**

It provides the previous:

> **90 days**

of **management events in each AWS Region**.

You can search/filter them from:

```text
CloudTrail
→ Event history
```

You can filter by things such as event name, resource, username and event source. 

This is perfect for questions like:

```text
Who terminated this EC2 yesterday?

Who modified this Security Group?

Who deleted this IAM role?
```

---

# 8. What is a Trail?

For longer-term logging, create a **CloudTrail Trail**.

Conceptually:

```text
AWS Account
    ↓
CloudTrail
    ↓
Trail
    ↓
S3 Bucket
```

A trail continuously delivers selected CloudTrail events to an S3 bucket.

This allows:

- Long-term retention
- Compliance
- Security auditing
- Athena queries
- Centralized multi-account logging

IAM and other AWS services can continuously deliver their CloudTrail events to S3 through a trail. 

---

# 9. CloudTrail Lake

**CloudTrail Lake** is designed for storing and analyzing CloudTrail events using SQL.

Architecture:

```text
CloudTrail events
      ↓
Event Data Store
      ↓
CloudTrail Lake
      ↓
SQL Queries
```

You can run queries such as conceptually:

```sql
SELECT *
FROM event_data_store
WHERE eventName = 'TerminateInstances';
```

CloudTrail Lake supports SQL-based querying over event data stores. 

We will use this when we get to the practical queries.

---

# 10. CloudTrail vs CloudWatch

Very important interview distinction:

### CloudWatch

```text
What is happening to the system?
```

Examples:

```text
CPU = 95%
Memory = 90%
Application error
Disk = 95%
```

### CloudTrail

```text
Who changed something in AWS?
```

Examples:

```text
Who terminated EC2?
Who opened port 22?
Who deleted IAM user?
Who updated EKS?
```

So:

> **CloudWatch = Monitoring**

> **CloudTrail = Auditing/API activity**

---

# Practical Example

Production goes down.

You discover:

```text
Security Group no longer allows port 443.
```

CloudWatch tells you:

```text
ALB target unhealthy
Application unavailable
```

CloudTrail tells you:

```text
eventName:
RevokeSecurityGroupIngress

userIdentity:
arn:aws:sts::123456789:assumed-role/DevOpsRole/...

sourceIPAddress:
203.x.x.x

eventTime:
09:32 UTC
```

Now you know **what changed, who changed it and when**.

---

## Quick Memory

```text
CloudTrail
   ↓
AWS API AUDIT

WHO
→ userIdentity

WHAT
→ eventName

WHICH SERVICE
→ eventSource

WHEN
→ eventTime

WHERE FROM
→ sourceIPAddress

WHAT PARAMETERS
→ requestParameters

FAILED?
→ errorCode / errorMessage
```

And:

```text
Event History
→ 90 days management events

Trail
→ Continuous delivery, commonly to S3

CloudTrail Lake
→ SQL queries / deeper investigation
```

### Interview one-liner

> **AWS CloudTrail is the AWS audit service that records account and API activity, allowing us to determine who performed an action, what API was called, when and from where it was called, which resources were affected, and whether the request succeeded or failed.** 