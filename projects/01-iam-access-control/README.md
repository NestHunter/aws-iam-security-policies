# IAM Access Control Assessment

A hands-on AWS IAM assessment focused on evaluating access through identity-based policies, S3 resource-based policies, attribute-based access control (ABAC), and permissions boundaries.

> **Documentation note:** This project records a completed guided skills assessment. Screenshots were not retained; the configuration approach, validation outcomes, and security rationale are documented below. No credentials, account IDs, bucket names, or other account-specific values are included.

## Objectives

- Create and attach a least-privilege identity-based S3 read policy.
- Grant object read access with an S3 bucket policy.
- Validate allowed and denied actions with the IAM Policy Simulator.
- Apply a tag-based ABAC model.
- Use a permissions boundary to cap the maximum permissions available to an IAM user.

## Architecture

```text
IAM user (developer-user)
  |
  |-- Identity policy: list bucket + read objects
  |-- ABAC policy: access conditioned on matching project tags
  |-- Permissions boundary: IAM API actions explicitly denied
  |
  +--> S3 bucket (project-bucket)
         |-- Bucket policy: allows developer-user to read objects
         +-- Resource tag: project=app1
```

## Controls Implemented

### 1. Identity-based policy

A customer-managed IAM policy granted only the access required for the lab bucket:

- `s3:ListBucket` on the bucket ARN.
- `s3:GetObject` on objects in that bucket.

This demonstrates least privilege by omitting write, deletion, and administrative permissions.

### 2. Resource-based policy

An S3 bucket policy named a specific IAM user as its principal and allowed `s3:GetObject` against objects in the lab bucket. This demonstrates that S3 can grant access through a resource policy in addition to an identity policy.

### 3. Policy evaluation

The IAM Policy Simulator was used to validate the intended authorization outcomes:

| Action | Expected outcome | Result |
|---|---|---|
| `s3:GetObject` | Allowed | Allowed |
| `s3:DeleteObject` | Denied | Denied |

The test confirms that an allow must be explicitly granted and that actions outside the allowed scope remain denied.

### 4. Attribute-based access control (ABAC)

The IAM user and S3 resource were tagged with the same attribute:

```text
project=app1
```

The ABAC policy compared the principal tag (`aws:PrincipalTag/project`) to the resource tag (`aws:ResourceTag/project`). This pattern supports scalable authorization: access can be driven by business attributes rather than maintaining separate policies for every individual resource and principal.

### 5. Permissions boundary

A permissions boundary was attached to the IAM user with an explicit deny on `iam:*`. The boundary functions as a permission ceiling: even if an identity policy were later attached that allowed IAM administration, the explicit deny would prevent those IAM actions.

## Security Takeaways

- Identity-based and resource-based policies can both contribute to an authorization decision.
- Least privilege requires limiting actions and resources rather than granting broad service access.
- ABAC can reduce policy sprawl when access should align to attributes such as project, department, environment, or data classification.
- Permissions boundaries are guardrails for delegated administration; they do not grant permissions on their own.
- Explicit denies override otherwise applicable allows.

## Potential Production Enhancements

- Replace long-lived IAM users with IAM Identity Center or federated, short-lived role sessions.
- Apply organization-level guardrails with AWS Organizations service control policies (SCPs).
- Enable CloudTrail, AWS Config, IAM Access Analyzer, and Security Hub for continuous auditability and detection.
- Restrict bucket access further with encryption requirements, TLS-only conditions, VPC endpoint conditions, and narrowly scoped prefixes.
- Define policies and tags with Terraform or CloudFormation for repeatable deployment and peer review.

## Status

**Completed** — guided AWS Security skills assessment; no issues encountered during implementation.
