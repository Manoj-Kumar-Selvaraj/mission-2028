# EC2 Launch Templates

## What is a Launch Template?

A **Launch Template is a reusable blueprint containing the configuration required to launch EC2 instances**.

Instead of manually configuring an EC2 instance every time, we define the configuration once and reuse it.

```text
Launch Template
      |
      ├── AMI
      ├── Instance Type
      ├── Security Groups
      ├── IAM Instance Profile
      ├── Key Pair
      ├── EBS Configuration
      ├── User Data
      ├── Network Configuration
      ├── IMDS Settings
      └── Tags
```

AWS supports using Launch Templates when launching EC2 directly and with services such as EC2 Auto Scaling. citeturn354691search6turn354691search22

---

# Why Use Launch Templates?

- Avoid manually configuring the same EC2 settings repeatedly.
- Ensures **consistent configuration** across multiple EC2 instances.
- Makes EC2 deployments easier to automate.
- Supports **versioning** for controlled configuration changes.
- Commonly used with **Auto Scaling Groups (ASG)**.
- Can be used to standardize security settings such as IAM roles, Security Groups and IMDSv2.
- Can be used to control which approved EC2 configurations users are allowed to launch. citeturn354691search29

### Easy Memory

> **Launch Template = Blueprint for creating EC2 instances.**

---

# Example Launch Template

```text
Launch Template: web-server-template

AMI               = ami-123456
Instance Type     = m7i.large
Security Group    = app-sg
IAM Profile       = app-server-profile
EBS               = 100 GB gp3
IMDS              = IMDSv2 Required
User Data         = bootstrap.sh
Tags              = Environment=Production
```

Whenever this template is used:

```text
Launch Template
       |
       ├──── EC2-1
       ├──── EC2-2
       └──── EC2-3
```

all instances can be launched using the same standardized configuration.

---

# Launch Template Versioning

Launch Templates support **multiple versions**.

Example:

```text
Launch Template: Jenkins-Template

Version 1
---------
AMI          = ami-old
Instance     = m5.large
Java         = 11

Version 2
---------
AMI          = ami-new
Instance     = m6i.large
Java         = 17

Version 3
---------
AMI          = ami-new
Instance     = m6i.xlarge
Java         = 17
```

When a configuration changes, create a **new Launch Template version** rather than changing the old version in place. AWS lets you select a specific version and choose which version is the default. citeturn354691search2turn354691search10

---

## `$Latest`

Represents the **most recently created Launch Template version**.

Example:

```text
v1
v2
v3

$Latest = v3
```

---

## `$Default`

Represents whichever version has been explicitly configured as the **default version**.

Example:

```text
v1
v2  ← Default
v3  ← Latest
```

Therefore:

```text
$Default = v2
$Latest  = v3
```

They do not necessarily point to the same version. citeturn354691search2turn354691search10

---

# Why Versioning is Useful

Versioning provides:

- Controlled configuration changes
- Rollback capability
- Testing of new AMIs
- Configuration history
- Safer patching and upgrades

Example:

```text
v1
AMI = Stable

       ↓

Create v2
AMI = Patched

       ↓

Testing fails

       ↓

Use v1 again
```

### Quick Memory

> **Launch Template versioning = controlled change + rollback.**

---

# Launch Template and Auto Scaling Group

This is the most important relationship.

## Launch Template answers:

> **HOW should an EC2 instance be created?**

It defines:

```text
AMI
Instance Type
Security Group
IAM Role
Storage
User Data
etc.
```

---

## Auto Scaling Group answers:

> **HOW MANY instances should run and where should they run?**

Example:

```text
Auto Scaling Group

Min     = 2
Desired = 3
Max     = 10

        |
        ↓

Launch Template

        |
        ├── AMI
        ├── Instance Type
        ├── Security Group
        ├── IAM Role
        ├── EBS
        └── User Data

        |
        ↓

EC2-1   EC2-2   EC2-3
```

An ASG can use the **default, latest, or a specific Launch Template version** when launching new instances. citeturn354691search1

### Easy Memory

> **Launch Template = HOW**

> **ASG = HOW MANY + WHERE**

---

# Scaling Example

Suppose:

```text
ASG

Min     = 2
Desired = 3
Max     = 10
```

Currently:

```text
EC2-1
EC2-2
EC2-3
```

Traffic increases.

```text
High Traffic
      ↓
Scaling Policy triggered
      ↓
ASG increases Desired Capacity
3 → 5
      ↓
ASG reads Launch Template
      ↓
Creates EC2-4 and EC2-5
```

The Launch Template provides the configuration for the new instances.

---

# Updating a Launch Template Used by an ASG

Suppose current ASG uses:

```text
Launch Template v1

AMI = ami-old
```

Current instances:

```text
EC2-1 → v1
EC2-2 → v1
EC2-3 → v1
```

Now create:

```text
Launch Template v2

AMI = ami-new
```

and update the ASG to use v2.

New instances launched afterward use the updated configuration, while existing instances continue running with the configuration they were originally launched with. citeturn354691search8turn354691search23

Example:

```text
EC2-1 → v1
EC2-2 → v1
EC2-3 → v1

New EC2-4 → v2
New EC2-5 → v2
```

---

# Important Point

Simply creating:

```text
Launch Template v2
```

does **not automatically update existing EC2 instances**.

Typical flow:

```text
Create Launch Template v2
        ↓
Update ASG to use v2
        ↓
New EC2s use v2
        ↓
Existing EC2s still use old configuration
```

To replace existing instances, use an:

> **ASG Instance Refresh**

---

# Instance Refresh

Instance Refresh allows an Auto Scaling Group to **gradually replace existing instances with instances using the new configuration**.

AWS specifically supports using Instance Refresh for changes such as new AMIs, instance types and user-data configurations. citeturn354691search3

Example:

```text
ASG
 |
 | Existing Instances
 |
 ├── EC2-1 → v1
 ├── EC2-2 → v1
 └── EC2-3 → v1

Create v2
   ↓
Update ASG
   ↓
Start Instance Refresh
   ↓
Replace EC2-1
   ↓
New EC2 → v2
   ↓
Health Check
   ↓
Replace EC2-2
   ↓
Health Check
   ↓
Replace EC2-3
```

This allows instances to be updated in a **rolling manner** rather than replacing everything simultaneously. citeturn354691search3turn354691search7

---

# AMI Patching Example

Suppose we have:

```text
100 EC2 Instances

ASG
 ↓
Launch Template v1
 ↓
AMI-old
```

Security team identifies vulnerabilities in the old AMI.

Process:

```text
Patch Base Image
      ↓
Create New AMI
      ↓
Create Launch Template v2
      ↓
AMI = New Patched AMI
      ↓
Test v2
      ↓
Update ASG to v2
      ↓
Start Instance Refresh
      ↓
Gradually replace old EC2 instances
      ↓
Validate application health
```

If the rollout has a problem, Instance Refresh also provides rollback capabilities in supported configurations. citeturn354691search30

---

# Launch Template vs AMI

These are different.

## AMI

Contains the **machine image**.

Example:

```text
Operating System
Java
Packages
Agents
Security patches
Application dependencies
```

## Launch Template

Defines **how to launch EC2 using that AMI**.

```text
Launch Template
   |
   ├── AMI
   ├── Instance Type
   ├── Security Group
   ├── IAM Role
   ├── EBS
   └── User Data
```

### Easy Memory

> **AMI = What's inside the server**

> **Launch Template = How the server should be launched**

---

# Launch Template vs User Data

User Data can itself be stored inside a Launch Template.

Example:

```text
Launch Template
   |
   ├── AMI
   ├── m7i.large
   ├── Security Group
   └── User Data
          |
          ↓
      bootstrap.sh
```

User Data runs bootstrap/configuration commands when the instance launches. citeturn354691search15

---

# Launch Template with Spot and On-Demand Instances

An ASG can use a **Mixed Instances Policy** with a Launch Template.

Example:

```text
Auto Scaling Group
       |
       ↓
Launch Template
       |
       ↓
Mixed Instances Policy
       |
       ├── m7i.large
       ├── m6i.large
       ├── m7a.large
       |
       ├── On-Demand
       └── Spot
```

This allows one ASG to use multiple EC2 instance types and a combination of **On-Demand and Spot capacity**. citeturn354691search0turn354691search19

### Why?

- Improve capacity availability
- Reduce Spot interruption risk
- Optimize cost
- Avoid depending on one instance type

---

# Launch Template Security Considerations

A Launch Template can standardize security configuration such as:

```text
Security Groups
IAM Instance Profile
IMDSv2 settings
Encrypted EBS
Network interfaces
Approved AMI
Tags
```

Organizations can also use IAM controls to require users to launch EC2 instances only through approved Launch Templates. citeturn354691search29

---

# Common Interview Scenario

### Question

We have 50 EC2 instances running in an ASG.

Security team provides a new patched AMI.

How would you deploy it?

### Answer

```text
Create patched AMI
      ↓
Create new Launch Template version
      ↓
Test the new version
      ↓
Update ASG to use the new version
      ↓
Start Instance Refresh
      ↓
Replace instances gradually
      ↓
Monitor EC2 + ALB/application health
      ↓
Rollback if required
```

---

# Important Interview Distinction

```text
Launch Template
      ↓
HOW EC2 should be created
```

```text
Auto Scaling Group
      ↓
HOW MANY EC2s should exist
+
WHERE they should run
```

```text
Scaling Policy
      ↓
WHEN ASG should increase/decrease capacity
```

```text
Instance Refresh
      ↓
HOW existing ASG instances are replaced
with a new configuration
```

---

# Quick Notes

- **Launch Template** = reusable EC2 launch blueprint.
- Stores settings such as **AMI, instance type, SG, IAM role, EBS, User Data, networking and tags**.
- Supports **multiple versions**.
- Use a new version when configuration changes.
- **`$Latest`** = newest created version.
- **`$Default`** = version marked as default.
- Versioning provides **controlled changes and rollback**.
- **Launch Template = HOW to create EC2.**
- **ASG = HOW MANY + WHERE.**
- ASG can use **specific, default or latest** Launch Template versions. citeturn354691search1
- Updating the template/ASG does not automatically reconfigure existing instances. citeturn354691search8
- Use **Instance Refresh** to replace existing instances with the new configuration. citeturn354691search3
- Mixed Instances Policy can combine **multiple instance types + On-Demand + Spot**. citeturn354691search0

## Quick Memory

**Launch Template → HOW**

**ASG → HOW MANY + WHERE**

**Scaling Policy → WHEN**

**Instance Refresh → UPDATE/REPLACE existing instances**