# KMS Key Policy: Administrators vs. Usage

A customer-managed KMS key policy that separates *who can manage the key* (create, rotate, disable, schedule deletion) from *who can use the key* (encrypt/decrypt data), instead of granting one broad set of permissions to everyone who touches it.

## What This Solves

KMS keys are unusual among AWS resources in that the key policy — not just IAM — is the primary access control mechanism; IAM policies can only grant access to a key if the key policy also allows it. A common mistake is granting the same principal both administrative actions (like `kms:ScheduleKeyDeletion`) and cryptographic usage actions (like `kms:Decrypt`), which means anyone who can *use* the key for an application can also *destroy* it or change who else can use it. Separating these roles limits the blast radius if an application role's credentials are compromised — a compromised app role can decrypt data it's authorized for, but it can't disable the key, delete it, or grant itself broader access.

## Why It's Scoped This Way

| Statement | Principal | Can Do | Cannot Do |
|---|---|---|---|
| `EnableRootAccountPermissions` | Account root | Everything (`kms:*`) | — (required by AWS so the account never loses the ability to manage its own keys via IAM) |
| `AllowKeyAdministration` | Key admin role | Create, rotate, tag, enable/disable, schedule/cancel deletion | Cannot `Encrypt`/`Decrypt`/`GenerateDataKey` — an admin managing the key's lifecycle doesn't need to read the data it protects |
| `AllowKeyUsage` | Application role | `Encrypt`, `Decrypt`, `GenerateDataKey*`, `DescribeKey` | Cannot disable, delete, or modify the key's policy — an application using the key for encryption can't affect its own future access or destroy the key |

The `EnableRootAccountPermissions` statement is included because AWS requires at least one statement granting the account root full access, or lockout is possible if every other principal's permissions were ever misconfigured — this is AWS's own documented default, not an over-grant.

## How This Would Be Validated

1. Create the key with this policy attached.
2. As the key admin role, confirm `kms:EnableKeyRotation` and `kms:ScheduleKeyDeletion` succeed, and confirm `kms:Encrypt` is denied.
3. As the application role, confirm `kms:Encrypt`/`kms:Decrypt` succeed against test data, and confirm `kms:ScheduleKeyDeletion` is denied.
4. Run both roles through the **IAM Policy Simulator** against this key policy to document the allow/deny outcomes before granting either role in production.

## Sanitization Note

`YOUR-ACCOUNT-ID`, `YOUR-KEY-ADMIN-ROLE`, and `YOUR-APPLICATION-ROLE` are placeholders — substitute your actual account ID and role ARNs before applying this policy.

## Reference

- [Key policies in AWS KMS — AWS documentation](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)
