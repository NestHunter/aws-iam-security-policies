# AWS IAM and Security Policies

**Last updated:** October 2026

Hands-on AWS identity and access work: an IAM access control assessment and a library of least-privilege IAM, resource, and organization policies. Each entry documents what the policy solves, why it is scoped the way it is, and how it is validated.

All JSON uses placeholder account IDs, ARNs, and resource names. No real account identifiers are included in this repository.

---

## IAM Access Control Assessment

### [Read the assessment](./projects/01-iam-access-control/README.md)

Identity-based and resource-based policy evaluation, ABAC through matched principal and resource tags, permissions boundaries as a delegation ceiling, and validation through the IAM Policy Simulator. Includes a production-hardening section covering Identity Center federation, service control policies, and continuous auditability with CloudTrail, Config, and IAM Access Analyzer.

---

## Policy Library

### [Browse the library](./policies/README.md)

| Domain | Policy | Control |
|---|---|---|
| Logging and Monitoring | [VPC Flow Logs to S3](./policies/logging-monitoring/vpc-flow-logs-to-s3/README.md) | Bucket policy for log delivery with `SourceAccount` and `SourceArn` conditions to prevent the confused deputy problem |
| Governance | [SCP: Deny Disabling Core Security Services](./policies/governance/scp-deny-disable-security-services/README.md) | Organization guardrail protecting CloudTrail, Config, and GuardDuty |
| IAM and Access | [Cross-Account Role with External ID](./policies/iam-access/cross-account-role-external-id/README.md) | Scoped third-party access gated by an external ID |
| Data Protection | [KMS Key Policy: Administrators vs. Usage](./policies/data-protection/kms-admin-vs-usage/README.md) | Separates key management from encrypt and decrypt use |

New entries are added as I work through AWS Certified Security - Specialty (SCS-C03) domains.

---

## Related Repositories

- [aws-labs](https://github.com/NestHunter/aws-labs): hands-on AWS builds and troubleshooting
- [NestHunter](https://github.com/NestHunter): profile and featured projects

## Connect

- [LinkedIn](https://www.linkedin.com/in/deandre-wilson-939217141)
- [CyberNest](https://cybernesthub.com/)
