# AWS Scope & Failure Domains

- **Failure domain:** Boundary where failures can occur independently, e.g. **AZ** or **Region**.
- **Latency:** Same AZ is lowest, cross-AZ is slightly higher, cross-Region is much higher.
- **Global resources:** IAM, Route 53, CloudFront.
- **Regional resources:** VPC, S3, ALB/NLB, DynamoDB.
- **AZ-scoped resources:** Subnet, EC2, EBS, ENI, NAT Gateway.
- **Multi-AZ:** Used for high availability within a Region.
- **Multi-Region:** Used for disaster recovery, global resilience, or lower user latency.

# Cloud Service Models

- **IaaS — Infrastructure as a Service:** Cloud provides compute, storage, networking and virtualization; you manage **OS, middleware, runtime and applications**. Example: **AWS EC2, Azure VMs**.

- **PaaS — Platform as a Service:** Cloud manages **infrastructure + OS + runtime/platform**; you mainly manage **application code and data**. Example: **AWS Elastic Beanstalk, Google App Engine**.

- **SaaS — Software as a Service:** Provider manages the **entire application and infrastructure**; users simply access and use the software. Example: **Gmail, Salesforce, Microsoft 365**.

- **FaaS — Function as a Service:** Run individual functions without managing servers; provider handles **infrastructure, scaling and runtime**. Example: **AWS Lambda, Azure Functions**.

### Quick Memory

**IaaS → Build on infrastructure**  
**PaaS → Build/deploy applications**  
**SaaS → Use the software**  
**FaaS → Run functions/code**


- **PaaS — Platform as a Service:** You deploy a **full application** onto a managed platform. You still think in terms of an app/service. Example: **Elastic Beanstalk, App Engine**.
- **FaaS — Function as a Service:** You deploy **small functions** that run on demand or on events. You think in terms of individual functions, not full servers/apps. Example: **AWS Lambda**.

**Shortcut:** `PaaS = managed app platform` | `FaaS = event-driven functions`

# Control Plane vs Data Plane

- **Control Plane:** APIs and management operations used to **create, configure, scale, or delete resources**. Example: launching EC2, updating an EKS deployment, changing an S3 bucket policy.

- **Data Plane:** The actual **runtime traffic and workload operations**. Example: users hitting an EC2-hosted app, pods serving requests, applications reading/writing S3 objects.

- **Key idea:** A control-plane outage may stop **changes/new provisioning**, while existing workloads can often continue serving through the data plane.

- **Example:** If the EC2 API is impaired, you may be unable to launch/terminate instances, but already-running EC2 instances can still serve application traffic.

