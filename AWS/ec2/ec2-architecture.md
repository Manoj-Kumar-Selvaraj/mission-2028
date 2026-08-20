## Complete Scenario: ALB + ASG + Multi-AZ + Launch Template + Auto Scaling

Use this architecture as the standard interview scenario:

```text
                         Internet
                            |
                            v
                    Application Load
                       Balancer
                    /             \
             Public Subnet     Public Subnet
                 AZ-A              AZ-B
                    \             /
                     \           /
                      Target Group
                     /           \
                    /             \
            Private Subnet    Private Subnet
                AZ-A              AZ-B
                 |                 |
              EC2-1              EC2-2
                 \                 /
                  \               /
                 Auto Scaling Group
                         |
                         v
                  Launch Template
```

The **ALB spans at least two Availability Zones**, while the ASG can place EC2 capacity across the private subnets in those AZs for resilience. 

---

# Example Requirement

Suppose we need to host a Java/web application.

```text
Region      = ap-south-1

AZ-A        = ap-south-1a
AZ-B        = ap-south-1b

ALB         = Internet facing
Application = HTTP :8080

ASG:
Min         = 2
Desired     = 2
Max         = 6

Scaling:
Target CPU  = 50%
```

---

# Step 1 — Create the Network

Assume a VPC:

```text
10.0.0.0/16
```

Create four subnets:

```text
AZ-A
Public-A   = 10.0.1.0/24
Private-A  = 10.0.11.0/24

AZ-B
Public-B   = 10.0.2.0/24
Private-B  = 10.0.12.0/24
```

Architecture:

```text
                     VPC 10.0.0.0/16

              AZ-A                     AZ-B

         Public-A                  Public-B
            ALB                       ALB

             |                         |

         Private-A                 Private-B
            EC2                       EC2
```

The **ALB goes into the public subnets**, while application EC2 instances go into private subnets.

For an internet-facing ALB, the selected subnets must support the load balancer's internet connectivity. ALB requires subnets in at least two Availability Zones. 

If your private EC2 instances need to download packages from the internet during User Data execution, they also need outbound connectivity such as:

```text
Private subnet
     ↓
NAT Gateway
     ↓
Internet Gateway
```

---

# Step 2 — Configure Security Groups

Create two Security Groups.

### `alb-sg`

Inbound:

```text
HTTPS 443 ← 0.0.0.0/0
```

Optionally:

```text
HTTP 80 ← 0.0.0.0/0
```

and redirect HTTP → HTTPS.

Outbound:

```text
Application Port 8080 → EC2
```

### `app-sg`

Inbound:

```text
TCP 8080
Source = alb-sg
```

Not:

```text
8080 ← 0.0.0.0/0   ❌
```

Architecture:

```text
Internet
   |
  443
   |
 alb-sg
   |
  ALB
   |
 8080
   |
 app-sg
   |
  EC2
```

This means users cannot directly reach the application instances; traffic must come through the ALB.

---

# Step 3 — Create IAM Role for EC2

For example:

```text
Role Name:
app-ec2-role
```

Trust:

```text
EC2 → sts:AssumeRole
```

Attach only the permissions the application needs.

For example:

```text
SSM access
CloudWatch
S3 application bucket access
Secrets Manager access
```

The role will later be referenced through the **IAM instance profile in the Launch Template**.

---

# Step 4 — Create Launch Template

Create:

```text
Name:
app-launch-template
```

Configure:

```text
AMI              = approved/patched AMI
Instance Type    = m7i.large
Security Group   = app-sg
IAM Profile      = app-ec2-role
EBS              = 50 GB gp3
IMDS             = IMDSv2 required
```

And optionally User Data:

```bash
#!/bin/bash

dnf install -y httpd
systemctl enable httpd
systemctl start httpd

echo "Application healthy" > /var/www/html/health
```

The Launch Template contains the information used by the ASG when it launches new EC2 instances, including the AMI and other EC2 configuration. 

Think:

```text
Launch Template
      |
      ├── AMI
      ├── Instance type
      ├── SG
      ├── IAM role
      ├── EBS
      └── User Data
```

---

# Step 5 — Create Target Group

Create:

```text
Name       = app-target-group
Target     = Instances
Protocol   = HTTP
Port       = 8080
VPC        = application VPC
```

Configure health check:

```text
Protocol = HTTP
Path     = /health
Port     = traffic port
```

Conceptually:

```text
Target Group

EC2-1:8080
EC2-2:8080
EC2-3:8080
```

The target group continuously checks its registered targets and makes their health status available to the load balancer. 

You normally **do not manually register ASG instances** here. Once the target group is attached to the ASG, Auto Scaling registers newly launched instances automatically. 

---

# Step 6 — Create the ALB

Create:

```text
Name = app-alb
Type = Application Load Balancer
Scheme = Internet-facing
```

Select:

```text
AZ-A → Public-A
AZ-B → Public-B
```

Attach:

```text
Security Group = alb-sg
```

Configure Listener:

```text
HTTPS :443
      ↓
Forward
      ↓
app-target-group
```

For HTTPS, normally attach an ACM certificate.

You may also configure:

```text
HTTP :80
   ↓
Redirect
   ↓
HTTPS :443
```

Now:

```text
User
 ↓
ALB :443
 ↓
Target Group
```

---

# Step 7 — Create Auto Scaling Group

Create:

```text
ASG Name:
app-asg
```

Select:

```text
Launch Template:
app-launch-template
```

Choose private subnets:

```text
Private-A → AZ-A
Private-B → AZ-B
```

Set:

```text
Minimum = 2
Desired = 2
Maximum = 6
```

AWS Auto Scaling maintains the desired number of EC2 instances within the configured minimum and maximum capacity. 

So initially:

```text
ASG Desired = 2

       AZ-A            AZ-B

      EC2-1            EC2-2
```

---

# Step 8 — Attach Target Group to ASG

During ASG creation or afterward:

```text
ASG
 ↓
Load Balancing
 ↓
Attach existing Target Group
 ↓
app-target-group
```

Now the relationship becomes:

```text
ASG
 |
 | launches EC2
 |
 v
EC2
 |
 | automatically registered
 |
 v
Target Group
 |
 v
ALB
```

Once a target group is attached, Auto Scaling automatically registers instances it launches with that target group. 

---

# Step 9 — Enable Load Balancer Health Checks in ASG

This is important.

By default, ASG uses EC2 health information. If you enable **Elastic Load Balancing health checks**, ASG can also replace an instance that the attached load balancer reports as unhealthy. 

Configure:

```text
Health Check:
EC2
+
Elastic Load Balancing
```

Set a grace period, for example:

```text
Health Check Grace Period = 300 seconds
```

This gives the application time to start before ASG acts on health-check failures. 

---

# Step 10 — Configure Auto Scaling Policy

Example:

### Target Tracking

```text
Metric:
Average CPU Utilization

Target:
50%
```

Target tracking adjusts ASG capacity in an attempt to keep the selected metric near its target value. 

Suppose:

```text
EC2-1 CPU = 85%
EC2-2 CPU = 80%

Average ≈ 82%
```

Target:

```text
50%
```

ASG may scale out:

```text
2 EC2
 ↓
3 EC2
 ↓
4 EC2
```

until the workload per instance falls toward the target.

Later:

```text
Traffic decreases
       ↓
CPU decreases
       ↓
ASG scales in
       ↓
4 → 3 → 2
```

It will not normally scale below:

```text
Min = 2
```

or above:

```text
Max = 6
```

---

# Complete Request Flow

Now assume the application is running.

A user requests:

```text
https://app.example.com/orders
```

Flow:

```text
User
 ↓
DNS / Route 53
 ↓
ALB :443
 ↓
Listener Rule
 ↓
Target Group
 ↓
Select Healthy Target
 ↓
EC2-1 :8080
 ↓
Application
```

---

# What Happens During Scale-Out?

Suppose traffic increases.

```text
Traffic ↑
    ↓
CPU ↑
    ↓
Target Tracking Policy
    ↓
ASG increases Desired
2 → 3
    ↓
ASG reads Launch Template
    ↓
Launch new EC2
    ↓
EC2 starts application
    ↓
Register with Target Group
    ↓
Health Check /health passes
    ↓
ALB starts sending traffic
```

This is the full relationship.

---

# What Happens if EC2 Fails?

Suppose:

```text
EC2-1
Application crashes
```

Then:

```text
/health fails
     ↓
Target Group marks EC2 unhealthy
     ↓
ALB stops normally sending it traffic
     ↓
ASG sees ELB health failure
     ↓
ASG marks instance unhealthy
     ↓
Terminates it
     ↓
Reads Launch Template
     ↓
Creates replacement EC2
     ↓
Registers with Target Group
     ↓
Health Check passes
     ↓
Receives traffic
```

This replacement behavior requires the ELB health check integration to be enabled for the ASG. 

---

# What Happens if One AZ Fails?

Initially:

```text
AZ-A                AZ-B

EC2-1               EC2-2
```

If AZ-A has a problem:

```text
EC2-1 ❌

EC2-2 ✅
```

Traffic can continue through healthy capacity in the other enabled AZ, and ASG works to restore its desired capacity across the configured Availability Zones. AWS recommends spreading ASG capacity across multiple AZs and attaching a load balancer for resiliency. 

---

# When We Patch the Servers

Suppose:

```text
Launch Template v1
AMI-old
```

Create:

```text
AMI-new
      ↓
Launch Template v2
```

Then:

```text
Update ASG → Launch Template v2
        ↓
Start Instance Refresh
        ↓
Old EC2 replaced gradually
        ↓
New EC2 uses AMI-new
```

Instance Refresh supports rolling replacement of ASG instances while maintaining the group's capacity according to the refresh preferences. 

---

# The 5 Components to Remember

| Component | Responsibility |
|---|---|
| **Launch Template** | How EC2 should be created |
| **ASG** | How many EC2s and where |
| **Scaling Policy** | When capacity should change |
| **Target Group** | Backend EC2s + health checks |
| **ALB** | Receives and distributes user traffic |

### Full memory flow

```text
                    ALB
                     ↓
                Target Group
                     ↓
            Healthy EC2 Instances
                 /       \
               AZ-A      AZ-B
                  \       /
                     ASG
                      |
                Scaling Policy
                      |
               Launch Template
```

And the most useful interview explanation is:

> **I would deploy the ALB across public subnets in at least two AZs and keep application EC2 instances in private subnets. The ASG spans those private subnets and launches instances using a versioned Launch Template. The ASG is attached to the ALB target group, so new instances are automatically registered. The target group performs application health checks, and with ELB health checks enabled on the ASG, unhealthy instances can be automatically replaced. Finally, a target-tracking policy such as CPU or request count scales the fleet between the configured minimum, desired and maximum capacity.**