Use this shorter version:

```md
# Auto Scaling Group (ASG)

An **Auto Scaling Group automatically maintains and scales a group of EC2 instances** based on configured capacity, health checks, and scaling policies.

---

## Core Capacity Settings

### Minimum
Minimum number of EC2 instances that must run.

### Desired
Number of EC2 instances ASG currently tries to maintain.

### Maximum
Maximum number of EC2 instances ASG can scale up to.

Example:

Min = 2  
Desired = 3  
Max = 10

If one of the 3 instances becomes unhealthy, ASG launches a replacement to bring the count back to 3.

---

## ASG + Launch Template

ASG uses a **Launch Template** to know how to create new EC2 instances.

Launch Template defines:

- AMI
- Instance type
- Security Group
- IAM role
- EBS
- User Data

Easy memory:

**Launch Template = HOW to create EC2**

**ASG = HOW MANY + WHERE**

---

## Multi-AZ

ASG can distribute instances across multiple Availability Zones.

Example:

AZ-A → EC2-1, EC2-2  
AZ-B → EC2-3, EC2-4

This provides **high availability and fault tolerance**.

---

## Scale Out vs Scale In

### Scale Out

Add more instances when demand increases.

3 EC2 → 5 EC2

### Scale In

Remove instances when demand decreases.

5 EC2 → 3 EC2

---

## Scaling Policies

### Target Tracking

Maintain a target metric.

Example:

Target CPU = 50%

If CPU becomes high, ASG adds instances.  
If CPU becomes low, ASG can remove instances.

### Step Scaling

Different thresholds perform different actions.

Example:

CPU > 60% → Add 1 EC2  
CPU > 80% → Add 2 EC2

### Scheduled Scaling

Scale at predefined times.

Example:

8 AM → Desired = 10  
10 PM → Desired = 2

### Predictive Scaling

AWS analyzes historical patterns and predicts future capacity requirements.

---

## Health Checks

ASG can use:

- EC2 health checks
- Load Balancer health checks

Example:

EC2 is running  
but application health check fails

→ ASG can mark the instance unhealthy  
→ terminate it  
→ launch a replacement

---

## Health Check Grace Period

A newly launched application may take time to start.

The **Health Check Grace Period** prevents ASG from replacing the instance too early while it is initializing.

---

## Instance Warmup

Gives a newly launched instance time to become fully operational before its metrics significantly affect scaling decisions.

---

## ASG + Load Balancer

Typical architecture:

Internet
   |
   v
  ALB
   |
Target Group
   |
-------------
|           |
EC2         EC2
AZ-A        AZ-B
 \           /
     ASG

New ASG instances are registered with the target group and start receiving traffic after becoming healthy.

---

## Instance Refresh

Used when we need to roll out a new AMI or Launch Template version.

Example:

New patched AMI
      ↓
Launch Template v2
      ↓
Update ASG
      ↓
Instance Refresh
      ↓
Old EC2 instances gradually replaced

---

## Lifecycle Hooks

Lifecycle Hooks pause an instance during launch or termination so custom actions can run.

Examples:

- Register with CMDB
- Run initialization
- Drain connections
- Upload logs before termination

---

## Mixed Instances

ASG can use multiple instance types and combine:

- On-Demand
- Spot

Example:

m6i.large  
m7i.large  
m7a.large

This improves **cost optimization and capacity availability**.

---

## Quick Memory

**Launch Template** → HOW to create EC2

**ASG** → HOW MANY + WHERE

**Scaling Policy** → WHEN to scale

**Health Check** → IS the instance healthy?

**Instance Refresh** → Replace existing instances with new configuration

---

## Interview One-Liner

> An Auto Scaling Group maintains the required number of EC2 instances, automatically replaces unhealthy instances, distributes capacity across Availability Zones, and scales instances in or out based on workload demand.
```