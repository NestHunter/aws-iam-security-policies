# SCP: Deny Disabling Core Security Services

An AWS Organizations service control policy (SCP) that prevents any principal in a member account — except a designated security admin role — from turning off CloudTrail, AWS Config, or GuardDuty.

## What This Solves

Detective controls only work if they can't be quietly disabled by whoever compromises (or over-permissions) an account. An attacker or a well-meaning but overly broad IAM policy can otherwise call `cloudtrail:StopLogging`, delete the Config recorder, or disable a GuardDuty detector, and monitoring goes dark with no alert generated at the moment it matters most. An SCP is the right control layer for this because it applies organization-wide as a permission *ceiling* — no identity-based policy in any member account, however permissive, can override it.

## Why It's Scoped This Way

- **Deny, not allow-list.** SCPs work as guardrails on top of existing IAM policies, so this is written as an explicit deny on the specific disable/delete actions for each service, rather than trying to allow-list everything else.
- **Carve-out for one role, not a blanket exception.** The `aws:PrincipalArn` condition exempts only a single designated security admin role, so break-glass changes are still possible but require assuming a specific, auditable identity — not just any admin-level principal.
- **Targets the disable actions specifically**, not the full service namespace. This SCP doesn't block `cloudtrail:Describe*`, `config:Get*`, or `guardduty:List*` — visibility and read access stay open; only the destructive/disabling actions are denied.
- **Three services, one policy.** CloudTrail (audit log), Config (configuration drift), and GuardDuty (threat detection) are grouped together because they're the minimum detective-control baseline referenced in the production-hardening section of the [IAM Access Control Assessment](../../01-iam-access-control/README.md) in this same repository.

## How This Would Be Validated

1. Attach the SCP to an OU or account in AWS Organizations.
2. As a non-exempt principal, attempt `aws cloudtrail stop-logging` and confirm it's denied with an explicit-deny error even if the IAM identity policy would otherwise allow it.
3. Repeat for the Config and GuardDuty actions.
4. As the exempted security admin role, confirm the same calls succeed, proving the carve-out works as intended.
5. Use IAM Policy Simulator or AWS Organizations' policy simulator to confirm the deny takes precedence regardless of the identity policy attached.

## Sanitization Note

`YOUR-ACCOUNT-ID` and `YOUR-SECURITY-ADMIN-ROLE` are placeholders — substitute your organization's actual account ID and the ARN of your designated break-glass/security-admin role.

## Reference

- [Service control policies (SCPs) — AWS Organizations documentation](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
