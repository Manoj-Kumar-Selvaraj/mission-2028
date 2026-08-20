# AWS SERVICES

# EC2

---

EC2 is AWS's virtual compute service. We can customize with a required CPU, memory, storage, networking, operating system and security configuration without managing the underlying physical hardware.

## Elements to Create a EC2 Instance:

```text
AMI
 |
Instance Type
 |
VPC
 |
Subnet
 |
Security Group
 |
Storage / EBS
 |
IAM Role
 |
Key/Access mechanism
 |
User Data
 |
Tags
 |
EC2 Instance
```

---

## Instance Types:

- M family → balanced
- C family → compute
- R family → memory
- I family → storage
- G/P families → accelerated/GPU

## `c7gn.2xlarge`

```text
c      7      g n      .      2xlarge
│      │      │ │             │
│      │      │ │             └─ Instance size
│      │      │ └────────────── Network optimized
│      │      └──────────────── Graviton processor ARM-64 or x86 processor
│      └─────────────────────── Generation - Newer gen for better efficiency
└────────────────────────────── Instance family/series
```

---

## Instance store volumes

Storage volumes for temporary data that is deleted when you stop, hibernate, or terminate your instance.

---

## Placement Group

A Placement Group lets you influence where EC2 instances are physically placed on AWS infrastructure.

Normally, AWS decides the underlying hardware placement for you. With a placement group, you tell AWS:

> “Keep these instances physically close together”

or

> “Keep these instances physically separated.”

AWS currently supports Cluster, Partition, Spread, and also Precision Time placement groups. For normal EC2 interviews, focus mainly on the first three.

### 1. Cluster Placement Group

**Purpose:**

Maximum network performance and minimum latency between EC2 instances.

```text
Same Availability Zone

Rack / AWS network segment
---------------------------
EC2   EC2   EC2   EC2
 ↔     ↔     ↔     ↔
Very low latency
High throughput
```

### 2. Spread Placement Group

This is almost the opposite.

You tell AWS:

> Keep my critical EC2 instances on separate underlying hardware.

Conceptually:

```text
Rack A       Rack B       Rack C
-------      -------      -------
 EC2-1        EC2-2        EC2-3
```

AWS recommends spread placement groups for a small number of critical instances where you want to reduce correlated hardware failures.

Spread = Maximum separation for a small number of important instances.

### 3. Partition Placement Group

Partition is somewhere between cluster and spread.

Instead of placing every instance separately, AWS creates groups called partitions.

Example:

```text
Partition 1       Partition 2       Partition 3
-----------       -----------       -----------
EC2 EC2 EC2       EC2 EC2 EC2       EC2 EC2 EC2

Rack group A      Rack group B      Rack group C
```

For a Partition Placement Group:

- You can create up to 7 partitions per Availability Zone. AWS Documentation
- There is no fixed EC2-instance limit per partition. The overall number of instances you can launch into the placement group is constrained mainly by your EC2 account/service quotas and available capacity. AWS Documentation
- AWS tries to distribute instances across the partitions, but it does not guarantee perfectly even distribution.

---

### EC2 Purchasing / Capacity Notes

#### 1. On-Demand

- No long-term commitment; pay for what you use.
- Best for temporary, unpredictable, or new workloads.
- **Use when:** dev/test, short-term servers, unpredictable traffic.

#### 2. Reserved Instance

- Commit for 1 or 3 years and get discounted EC2 pricing.
- Mainly useful for stable, predictable workloads.
- **Use when:** production EC2 runs continuously for a long time.
- **Remember:** Reserved Instance = **cost saving**.

#### 3. Savings Plan

- Commit to a certain amount of compute usage/spend for 1 or 3 years.
- More flexible than traditional Reserved Instances.
- **Use when:** compute usage is predictable but instance family/type may change.
- **Remember:** Savings Plan = **flexible long-term saving**.

#### 4. Spot Instance

- Uses spare AWS capacity at a very low price.
- AWS can interrupt/reclaim the instance.
- **Use when:** workload can restart or tolerate interruption.
- Examples: batch jobs, CI runners, stateless workers.
- **Remember:** Spot = **cheap but interruptible**.

---

### Capacity

#### 5. Capacity Reservation

- Reserves EC2 capacity in a specific Availability Zone.
- Ensures capacity is available when you need to launch instances.
- **Use when:** DR or critical workloads cannot risk insufficient AWS capacity.
- **Remember:** Capacity Reservation = **guaranteed capacity**.

---

### Dedicated Hardware

#### 6. Dedicated Instance

- EC2 runs on hardware dedicated to your AWS account.
- AWS still manages the physical host placement.
- **Use when:** compliance or isolation requires dedicated hardware.
- **Remember:** Dedicated Instance = **hardware isolation**.

#### 7. Dedicated Host

- Entire physical server is dedicated to you.
- Gives more host-level visibility/control.
- **Use when:** BYOL, socket/core-based licensing, strict compliance.
- **Remember:** Dedicated Host = **entire physical server**.

### Quick Memory

**On-Demand → Flexible**  
**Reserved → Save money**  
**Savings Plan → Flexible savings**  
**Spot → Cheap + interruptible**  
**Capacity Reservation → Guarantee capacity**  
**Dedicated Instance → Hardware isolation**  
**Dedicated Host → Entire physical host**

---

# EBS — Elastic Block Store

EBS is persistent block storage for EC2.

## 1. Main EBS volume types

The important ones are:

### gp3 — General Purpose SSD

- Default choice for most workloads.
- Good balance of price + performance.
- You can configure size, IOPS and throughput independently.
- Use for application servers, Jenkins, normal databases, general workloads.

### gp2 — Older General Purpose SSD

- Older generation.
- Performance is more tied to volume size.
- For new workloads, gp3 is generally preferred.

### io2 — Provisioned IOPS SSD

- Designed for very high IOPS, low latency and critical workloads.
- Use for I/O-intensive databases or workloads requiring predictable storage performance. AWS Documentation

### st1 — Throughput Optimized HDD

- HDD-based.
- Designed for large sequential I/O rather than lots of small random reads/writes.
- Use for large data processing, logs, big-data style workloads.

### sc1 — Cold HDD

- Cheapest HDD option.
- For infrequently accessed data where performance is not important.

### Quick memory

- gp3 = normal workloads
- io2 = high-performance database
- st1 = large sequential throughput
- sc1 = cold/infrequent data

| Type | Size | Max IOPS | Max Throughput | Best For |
|---|---:|---:|---:|---|
| **gp3** | 1 GiB – 64 TiB | **80,000** | **2,000 MiB/s** | General workloads |
| **gp2** | 1 GiB – 16 TiB | **16,000** | **250 MiB/s** | Older general workloads |
| **io2 Block Express** | 4 GiB – 64 TiB | **256,000** | **4,000 MiB/s** | Critical DB / ultra-low latency |
| **st1** | 125 GiB – 16 TiB | ~500* | **500 MiB/s** | Large sequential workloads |
| **sc1** | 125 GiB – 16 TiB | ~250* | **250 MiB/s** | Cold sequential data |

---

## 2. IOPS vs Throughput

Very important.

### IOPS

Input/Output Operations Per Second

Means:

> How many read/write operations can storage perform each second?

Example:

A database doing thousands of small random reads:

```text
4 KB read
4 KB write
4 KB read
4 KB write
...
```

That workload cares heavily about IOPS.

### Throughput

Means:

> How much total data can be transferred per second.

Usually measured in:

```text
MB/s
MiB/s
```

Example:

Reading one huge 10 GB file sequentially cares more about throughput.

### Easy memory

- IOPS = how many operations
- Throughput = how much data

---

## Incremental snapshots

Conceptually, after the first snapshot, later snapshots store only changed blocks.

---

```text
EC2: ap-south-1a
EBS: ap-south-1b
❌ direct attach
``` 


To move data to another AZ:

```text
EBS
 ↓
Snapshot
 ↓
Create volume in another AZ
 ↓
Attach
```

---

## Very common interview scenario

Disk is 95% full. How do you increase it?

A good answer:

- Identify the volume and filesystem.
- Check whether cleanup/log rotation can solve the issue first.
- Take snapshot if required by change policy.
- Increase EBS volume size.
- Verify new block-device size using `lsblk`.
- Extend partition if needed.
- Extend filesystem.
- Validate using `df -h`.
- Add monitoring/alerting to prevent recurrence.

Don't answer only:

> “Increase EBS from AWS console.”

Because AWS volume resize alone may not make the filesystem see the additional space.

---

## EBS performance can also be limited by EC2

Very important.

Suppose you provision a very fast EBS volume.

That doesn't automatically guarantee full performance.

The EC2 instance itself has limits for:

- EBS bandwidth
- network bandwidth
- IOPS

---

## Instance Store

- Instance Store = temporary/ephemeral block storage physically attached to the EC2 host.
- It is usually very fast because the storage is local to the underlying physical server.
- It is available only on EC2 instance types that support instance-store disks; the number and size depend on the instance type.
- You cannot attach an instance-store volume later like EBS; it must be available/attached when the instance launches.

---

## 1. ENI — Elastic Network Interface

ENI is the virtual network card (NIC) attached to an EC2 instance.

Every EC2 instance gets a primary ENI.

An ENI can contain:

- Primary private IP
- Secondary private IPs
- Security Groups
- MAC address
- Public/Elastic IP association

Some EC2 instance types support multiple ENIs.

```text
EC2
 |
 └── ENI eth0
      ├── Private IP
      ├── Security Groups
      └── Public/EIP mapping
```

```text
EC2 Firewall

ENI-1 → Public/Frontend network
ENI-2 → Internal/Backend network
```

---

## Public IP

The public IP is not actually configured directly on the operating system's ENI in the normal EC2 model.

AWS performs one-to-one NAT between:

```text
Public IP
    ↓
Private IP on ENI
```

### Public IP can change

If an automatically assigned public IPv4 address is used:

```text
Stop instance
     ↓
Start instance
     ↓
Public IP may be different
```

AWS releases that public IPv4 address on stop/hibernate/termination and can assign a new one when the instance starts.

---

## Elastic IP — EIP

An Elastic IP is a static public IPv4 address allocated to your AWS account.

Unlike an automatically assigned public IP:

---

## Source/Destination Check

- **Source/Destination Check** is enabled by default on EC2 and verifies that the instance is either the **source or destination** of the traffic.
- For a normal web/app server, keep it **enabled**, because traffic is directly going to or coming from that EC2 instance.
- If the EC2 acts as a **NAT instance, router, or firewall appliance**, it forwards traffic for other systems.
- In that case, the packet’s real source and destination are different from the forwarding EC2, so AWS can block it.
- Therefore, **disable Source/Destination Check** on EC2 instances that need to forward network traffic.

---

## Elements of SG

- Protocol
- Port
- Source
- Destination

### Inbound rule

Controls traffic coming into EC2.

Example:

```text
Protocol: TCP
Port: 443
Source: 0.0.0.0/0
```

### Outbound rule

Controls traffic leaving EC2.

Example:

```text
Protocol: TCP
Port: 443
Destination: 0.0.0.0/0
```

### Security Groups are Stateful

This is one of the most important points.

AWS automatically allows that return traffic when an incoming traffic comes to ec2, even if there isn't a corresponding explicit outbound rule for the ephemeral return connection.

### Security Groups only have Allow rules

### Security Group referencing another Security Group

Suppose:

```text
Internet
   |
   v
ALB
   |
   v
EC2
```

Instead of configuring EC2 as:

```text
Port 8080
Source = 0.0.0.0/0
```

you can configure:

```text
EC2 Security Group:

Port: 8080
Source: ALB-Security-Group
```

Meaning:

> Only resources associated with that ALB SG can reach the application port.

This is much more secure.

```text
ALB-SG
   |
   | TCP 8080
   v
APP-SG
```

### One EC2 can have multiple Security Groups

### One Security Group can be used by multiple EC2 instances

### Default Security Group behavior

```text
No inbound rules

Allow all outbound
```

### Can you change Security Groups while EC2 is running?

Yes

NACL — Network Access Control List


A NACL is a subnet-level firewall. It 

NACLs are stateless, so return traffic is not automatically allowed. 

NACLs support both ALLOW and DENY rules, unlike Security Groups, which only support allow rules. 

Every subnet must be associated with one NACL; if you don't choose one, AWS associates the subnet with the default NACL.


The key is this: when your laptop connects to a server on port 443, your laptop does not also use port 443 as its own source port.


Client/Laptop                EC2 Server
192.168.1.10:54321  ------>  10.0.1.20:443
             ↑                     ↑
        Source port           Destination port

Here, 443 is the server's HTTPS port. 54321 is a temporary ephemeral port automatically chosen by the client OS for that connection.
When EC2 replies, the direction reverses:

EC2 Server                   Client/Laptop
10.0.1.20:443       ------>  192.168.1.10:54321

So with a stateless NACL, you need to think about both directions:
Inbound to EC2: allow destination 443
Outbound from EC2: allow the client's ephemeral port range
That is why NACL configuration often includes ephemeral ports such as roughly 1024-65535, depending on the client operating system and architecture.
Security Groups are easier here because they are stateful: if the inbound connection to 443 was allowed, AWS automatically allows the reply back to 54321.

Short notes
Rule number: lower number = higher priority.
First matching rule wins.
NACL supports ALLOW + DENY.
Inbound: specify source CIDR.
Outbound: specify destination CIDR.
Because NACL is stateless, configure both request and return traffic.
Default NACL → allows all.
New custom NACL → denies all until rules are added.
Final * rule → DENY everything not already matched.

Prefix List
A Prefix List is a reusable collection of CIDR blocks.
Instead of putting many CIDRs directly into every Security Group:

Prefix List: corporate-networks

10.10.0.0/16
10.20.0.0/16
10.30.0.0/16

AWS gives it an ID such as:

pl-0123456789abcdef

# IMDSv1 vs IMDSv2

## IMDS

**IMDS = Instance Metadata Service**

- Used by applications running inside EC2 to retrieve instance metadata.
- Metadata endpoint:

```text
169.254.169.254
```

- Can provide:
  - Instance ID
  - AMI ID
  - Instance type
  - Private IP
  - Region / Availability Zone
  - IAM role temporary credentials

---

## IMDSv1

- Uses a simple **GET request**.
- Does **not require a token**.
- Easier to use but less secure.

### Retrieve metadata using IMDSv1

```bash
curl http://169.254.169.254/latest/meta-data/
```

Retrieve Instance ID:

```bash
curl http://169.254.169.254/latest/meta-data/instance-id
```

Retrieve AMI ID:

```bash
curl http://169.254.169.254/latest/meta-data/ami-id
```

Retrieve private IP:

```bash
curl http://169.254.169.254/latest/meta-data/local-ipv4
```

**Remember:**  
**IMDSv1 = Direct GET request, no token**

---

## IMDSv2

- Uses a **session token**.
- First request a token using `PUT`.
- Then send the token with metadata requests.
- More secure and preferred for production workloads. citeturn430487search0turn430487search2

### Step 1 — Get IMDSv2 Token

```bash
TOKEN=$(curl -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
```

`21600` seconds = **6 hours**.

### Step 2 — View Available Metadata

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/
```

### Retrieve Instance ID

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

### Retrieve AMI ID

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/ami-id
```

### Retrieve Instance Type

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-type
```

### Retrieve Private IP

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/local-ipv4
```

### Retrieve Availability Zone

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/availability-zone
```

AWS documents this token-first IMDSv2 request pattern for retrieving EC2 metadata. citeturn430487search0

---

## Retrieve IAM Role Credentials

First, find the IAM role attached to the instance:

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

Example response:

```text
MyEC2Role
```

Then retrieve its temporary credentials:

```bash
curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/MyEC2Role
```

The response contains temporary values such as:

```text
AccessKeyId
SecretAccessKey
Token
Expiration
```

Normally AWS SDKs and AWS CLI retrieve and rotate these temporary credentials automatically, so applications should not manually store them. citeturn430487search1turn430487search12

---

## IMDSv2 Flow

```text
EC2 Application
      ↓
PUT request for Token
      ↓
IMDS returns Token
      ↓
GET metadata + Token
      ↓
Metadata / Temporary Credentials
```

**Remember:**  
**IMDSv2 = Token first → Metadata next**

---

## IMDS and IAM Roles

```text
EC2 Application
      ↓
IMDSv2
      ↓
Instance Profile / IAM Role
      ↓
Temporary AWS Credentials
      ↓
AWS Services
```

This avoids storing long-lived AWS access keys inside EC2.

---

## Enforcing IMDSv2

```text
HttpTokens = required
```

Means:

> IMDSv2 is mandatory and IMDSv1 requests are rejected.

---

## Quick Comparison

| Feature | IMDSv1 | IMDSv2 |
|---|---|---|
| Token required | No | Yes |
| Access method | Direct GET | PUT token → GET metadata |
| Security | Lower | Higher |
| Recommended | Legacy | Yes |
| IAM temporary credentials | Yes | Yes |

### Quick Memory

**IMDS → EC2 metadata service**  
**Endpoint → `169.254.169.254`**  
**IMDSv1 → Direct GET, no token**  
**IMDSv2 → Get token first, then retrieve metadata**  
**`HttpTokens = required` → Enforce IMDSv2**  
**IAM role credentials → Available through IMDS and automatically used by AWS SDK/CLI**

## SSRF — Server-Side Request Forgery

- SSRF is an attack where an attacker tricks a **server-side application into making requests on their behalf**.
- It can be used to access internal resources that are not directly reachable from the internet.
- In EC2, an attacker may try to make the application access the metadata endpoint:

  `http://169.254.169.254`

- SSRF can expose:
  - EC2 metadata
  - IAM role credentials
  - Internal APIs
  - Private services

### Example

Attacker → Vulnerable Application → EC2 Metadata Service

### Quick Memory

**SSRF = Attacker makes the server send the request.**

Launch Templates
1. What is a Launch Template?
A Launch Template stores the configuration required to launch EC2 instances.
Instead of manually selecting the same settings every time, you define them once and reuse them. AWS Documentation
A Launch Template can contain things like:

Auto Scaling Group
Min     = 2
Desired = 3
Max     = 10
        |
        v
Launch Template
        |
        ├── AMI
        ├── Instance Type
        ├── SG
        ├── IAM Role
        ├── EBS
        └── User Data
        |
        v
    EC2  EC2  EC2

### User Data / Bootstrap Scripts — 5 Points

- **User Data** is a script or configuration passed to an EC2 instance at launch to perform **initial bootstrap tasks**.
- It is commonly used to install packages, configure services, fetch application files, or register the instance with another platform.
- Example: install Nginx, start an application, configure monitoring agents, or pull configuration from S3/SSM.
- **AMI** contains the prebuilt OS/software baseline; **User Data** handles environment-specific or launch-time configuration.
- **Use AMI for stable reusable software**, and **User Data for lightweight dynamic configuration** that may differ between dev/stage/prod.

