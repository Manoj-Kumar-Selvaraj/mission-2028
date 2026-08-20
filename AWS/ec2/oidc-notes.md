Sure. For AWS, an **OIDC identity provider** lets external workloads such as GitHub Actions, EKS service accounts, or another OIDC-compatible platform obtain **temporary AWS credentials through STS**, without storing long-lived access keys.

## OIDC Provider Setup — Core Flow

```text
External Workload
     ↓
OIDC Token / JWT
     ↓
AWS IAM OIDC Provider
     ↓
IAM Role Trust Policy
     ↓
STS AssumeRoleWithWebIdentity
     ↓
Temporary AWS Credentials
```

### 1. Create the OIDC Identity Provider

In AWS:

```text
IAM
→ Identity providers
→ Add provider
→ OpenID Connect
```

You provide:

```text
Provider URL
Audience / Client ID
```

For example, GitHub Actions uses:

```text
Provider URL:
https://token.actions.githubusercontent.com

Audience:
sts.amazonaws.com
```

### 2. Create an IAM Role

Create a role that the OIDC workload will assume.

The role needs a trust policy allowing the OIDC provider.

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity"
    }
  ]
}
```

The important action is:

```text
sts:AssumeRoleWithWebIdentity
```

### 3. Add Conditions

This is critical.

Do not allow every identity from the provider to assume the role.

Restrict the JWT claims.

For GitHub:

```json
"Condition": {
  "StringEquals": {
    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
  },
  "StringLike": {
    "token.actions.githubusercontent.com:sub": "repo:my-org/my-repo:*"
  }
}
```

This means only tokens from the intended repository can assume the role.

### 4. Attach Permissions to the Role

Trust policy answers:

> **Who can assume the role?**

Permissions policy answers:

> **What can the role do after it is assumed?**

Example:

```text
OIDC Role
   |
   ├── Trust Policy
   |      ↓
   |   GitHub repo allowed
   |
   └── Permissions Policy
          ↓
       S3 / ECR / EKS / Terraform actions
```

### 5. External Workload Requests Credentials

The external platform gets an OIDC JWT and sends it to AWS STS.

Conceptually:

```text
JWT
 ↓
STS AssumeRoleWithWebIdentity
 ↓
AWS validates:
  issuer
  audience
  subject
  signature
  trust policy
 ↓
Temporary credentials
```

AWS returns:

```text
AccessKeyId
SecretAccessKey
SessionToken
Expiration
```

No permanent AWS access key needs to be stored in the external platform.

## Example — GitHub Actions

GitHub workflow:

```yaml
permissions:
  id-token: write
  contents: read
```

Then the AWS credentials action requests the GitHub OIDC token and assumes the AWS role.

Conceptually:

```text
GitHub Actions
     ↓
GitHub OIDC Token
     ↓
AWS IAM Provider
     ↓
IAM Role
     ↓
STS
     ↓
Temporary Credentials
```

## Example — EKS IRSA

For EKS, the OIDC provider is usually based on the **EKS cluster's OIDC issuer**.

Then the IAM role trust policy restricts the `sub` claim to a Kubernetes service account:

```text
system:serviceaccount:namespace:service-account
```

Example:

```json
"Condition": {
  "StringEquals": {
    "oidc.eks.us-east-1.amazonaws.com/id/ABC123:aud": "sts.amazonaws.com",
    "oidc.eks.us-east-1.amazonaws.com/id/ABC123:sub":
      "system:serviceaccount:payments:payment-api"
  }
}
```

Then:

```text
Pod
 ↓
Service Account
 ↓
OIDC JWT
 ↓
STS
 ↓
IAM Role
 ↓
Temporary AWS credentials
```

## Easy Memory

```text
OIDC Provider
→ WHO issued the token?

Trust Policy
→ WHO is allowed to assume the role?

Conditions
→ WHICH repo / service account / workload?

Permissions Policy
→ WHAT can the assumed role do?

STS
→ Gives temporary credentials
```

### Interview one-liner

> **To configure OIDC federation in AWS, I create an IAM OIDC identity provider with the external issuer URL and audience, create an IAM role that trusts that provider using `sts:AssumeRoleWithWebIdentity`, restrict the trust using claims such as `aud` and `sub`, and attach least-privilege permissions to the role. The external workload then exchanges its signed OIDC token for temporary AWS credentials through STS.**