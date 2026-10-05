# IAM & Resource Policy Library

A collection of least-privilege IAM and resource policies for common AWS security patterns. Each entry documents what the policy solves, why it's scoped the way it is, and how it would be validated (IAM Policy Simulator, Access Analyzer, or a live test) — the same methodology used in the [IAM Access Control Assessment](../projects/01-iam-access-control/README.md).

Every JSON file uses placeholder account IDs, ARNs, and resource names. No real account identifiers are included anywhere in this repository.

## Logging & Monitoring

- [VPC Flow Logs → S3 Delivery Policy](./logging-monitoring/vpc-flow-logs-to-s3/README.md) — S3 bucket policy that lets the flow logs delivery service write to a destination bucket, scoped with `SourceAccount`/`SourceArn` conditions to prevent the confused deputy problem.

## Governance

- [SCP: Deny Disabling Core Security Services](./governance/scp-deny-disable-security-services/README.md) — Organization-wide guardrail preventing CloudTrail, Config, and GuardDuty from being disabled by anyone outside a designated security admin role.

## IAM & Access

- [Cross-Account Role with External ID](./iam-access/cross-account-role-external-id/README.md) — Trust policy and paired least-privilege permissions policy for granting a third-party account scoped access, gated by an external ID.

## Data Protection

- [KMS Key Policy: Administrators vs. Usage](./data-protection/kms-admin-vs-usage/README.md) — Key policy separating who can manage a KMS key's lifecycle from who can use it to encrypt/decrypt data.

---

More entries will be added here as I continue through AWS Certified Security – Specialty (SCS-C03) prep — each new domain studied becomes a candidate for a policy in this library.
