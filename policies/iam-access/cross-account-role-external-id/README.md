# Cross-Account Role with External ID

An IAM role trust policy that allows a specific, trusted external AWS account to assume a role in this account — gated by an external ID — paired with a least-privilege permissions policy scoping what that assumed role can actually do.

## What This Solves

Granting a third party (a vendor, auditor, or partner account) access to your AWS environment is a common but risky pattern. The naive approach — an IAM user with long-lived access keys shared out-of-band — creates credentials that can leak, never expire on their own, and aren't tied to the third party's own identity lifecycle. Cross-account roles solve the credential problem, but on their own they're vulnerable to the **confused deputy problem**: if Account B is a SaaS vendor serving many customers, and it assumes a role using only the trust policy's `Principal`, a malicious customer of that vendor could potentially trick the vendor into assuming *your* role instead of their own.

## Why It's Scoped This Way

- **`sts:ExternalId` condition on the trust policy.** This is the AWS-documented mitigation for the confused deputy problem — the external account must present a shared secret value known only to you and them, in addition to being the correct principal. Without it, being listed as the trusted `Principal` is enough on its own, which is a weaker guarantee if the trusted account serves multiple downstream customers.
- **No permissions on the trust policy itself.** The trust policy only answers "who can assume this role" (`sts:AssumeRole`). All actual access is defined separately in the permissions policy attached to the role, so the two concerns — *who* can act, and *what* they can do — stay independently auditable.
- **Least-privilege permissions policy.** The attached permissions policy grants only `s3:GetObject` and `s3:ListBucket` on a single named bucket, not `s3:*` or account-wide access. Whatever the third party's use case, the role should not be able to do more than that specific task requires.
- **`root` principal, not a specific IAM user/role in the trusted account.** This delegates identity management to the trusted account — they can rotate their own users/roles freely without you needing to update the trust policy, while `sts:ExternalId` still prevents any principal in that account from assuming the role unless they also know the shared secret.

## How This Would Be Validated

1. Create the role with this trust policy and attach the permissions policy.
2. From the trusted account, attempt `sts:AssumeRole` without the correct `ExternalId` and confirm it's denied.
3. Attempt it again with the correct `ExternalId` and confirm it succeeds.
4. From the assumed role's temporary credentials, confirm `s3:GetObject`/`s3:ListBucket` succeed on the named bucket and that any other action (e.g., `s3:DeleteObject`, access to a different bucket) is denied.
5. Run the trust policy through **IAM Access Analyzer** to confirm it doesn't flag the role as unintentionally granting broader external access than intended.

## Sanitization Note

`TRUSTED-ACCOUNT-ID`, `YOUR-UNIQUE-EXTERNAL-ID`, and `YOUR-SHARED-BUCKET` are placeholders. The external ID in particular should be a long, random, non-guessable value — treat it with the same care as a shared secret, not a public identifier.

## Reference

- [How to use an external ID when granting access to your AWS resources to a third party — AWS documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html)
