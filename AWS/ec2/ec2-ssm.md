To configure **SSM Session Manager for an EC2 instance**, think of it as 4 requirements:

## 1. SSM Agent on the EC2 instance

The instance needs the **SSM Agent** installed and running. Many AWS-provided AMIs already have it preinstalled. 

Check on Linux:

```bash
sudo systemctl status amazon-ssm-agent
```

Start it if needed:

```bash
sudo systemctl start amazon-ssm-agent
sudo systemctl enable amazon-ssm-agent
```

---

## 2. Attach IAM role to EC2

Create an IAM role with trust:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "ec2.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}
```

Attach the AWS-managed policy:

```text
AmazonSSMManagedInstanceCore
```

Then attach that role through an **Instance Profile** to the EC2 instance. AWS confirms this policy provides the basic permissions needed by Session Manager. 

Conceptually:

```text
EC2
 |
Instance Profile
 |
IAM Role
 |
AmazonSSMManagedInstanceCore
```

---

## 3. EC2 needs network connectivity to SSM

The SSM Agent must be able to communicate **outbound** with AWS Systems Manager.

You can provide this through either:

```text
Private EC2
   |
   v
NAT Gateway
   |
   v
Internet
   |
   v
AWS Systems Manager
```

or for a fully private architecture:

```text
Private EC2
   |
   v
VPC Interface Endpoints
   |
   v
Systems Manager
```

AWS supports private connectivity through VPC interface endpoints, so the instance does not necessarily need internet access. 

You **do not need inbound port 22** for Session Manager. 

---

## 4. Give the engineer permission to start sessions

The person connecting also needs IAM permissions for Session Manager.

For example, permissions involving:

```text
ssm:StartSession
ssm:TerminateSession
ssm:ResumeSession
```

Normally you'd restrict these to the required EC2 instances/users rather than allowing all resources.

---

# Once configured

Go to:

```text
EC2
→ Instances
→ Select Instance
→ Connect
→ Session Manager
→ Connect
```

Or use CLI:

```bash
aws ssm start-session \
  --target i-0123456789abcdef0
```

For CLI-based sessions, your local machine also needs the **Session Manager plugin**. 

## Easy flow to remember

```text
Engineer IAM Permission
        ↓
AWS Systems Manager
        ↓
SSM Agent
        ↓
EC2

EC2 side requires:
SSM Agent
+
IAM Instance Role
+
Network connectivity to SSM
```

### Interview one-liner

> **To enable Session Manager, I ensure the EC2 has SSM Agent running, attach an instance role with Systems Manager permissions such as `AmazonSSMManagedInstanceCore`, provide outbound connectivity to SSM through NAT or VPC endpoints, and grant the administrator IAM permission to start the session. No inbound SSH port is required.**