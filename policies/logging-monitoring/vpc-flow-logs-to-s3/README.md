# VPC Flow Logs → S3 Delivery Policy

An S3 bucket resource policy that allows the VPC Flow Logs delivery service to write log files to a destination bucket, without over-granting access to any IAM principal.

## What This Solves

VPC Flow Logs can publish to Amazon S3 for long-term retention and downstream analysis (Athena, Security Lake, SIEM ingestion). Delivery isn't performed by an IAM user or role you control — it's performed by the AWS log delivery service on your behalf. That means the destination bucket needs a policy that trusts the delivery service directly, scoped tightly enough that no other principal or account can write to (or read the ACL of) that bucket.

## Why It's Scoped This Way

| Statement | Purpose |
|---|---|
| `AWSLogDeliveryWrite` | Grants `s3:PutObject` only to the `delivery.logs.amazonaws.com` service principal — not to an account ARN, IAM role, or `*`. |
| `AWSLogDeliveryAclCheck` | Grants the same service principal `s3:GetBucketAcl`, which the delivery service needs to confirm write permissions before publishing. |
| `aws:SourceAccount` condition | Confirms the flow log (the actual log source) belongs to the expected account, not just any account that happens to invoke the delivery service. |
| `aws:SourceArn` condition | Scopes delivery to log sources in a specific account/region, closing the "confused deputy" gap where the delivery service could otherwise be tricked into writing on behalf of an unrelated resource. |
| `DenyUnencryptedTransport` | Explicit deny on any `s3:*` call over plain HTTP, regardless of principal — defense in depth independent of the two allow statements above. |

This follows AWS's own guidance to grant permissions to the service principal rather than an account ARN, and to use `SourceAccount`/`SourceArn` specifically to prevent the confused deputy problem — see [Amazon S3 bucket permissions for flow logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-s3-permissions.html).

## How This Would Be Validated

1. Attach the policy to a test S3 bucket via the console or `aws s3api put-bucket-policy`.
2. Create a VPC Flow Log with S3 as the destination, pointed at the bucket.
3. Confirm log objects begin appearing under `AWSLogs/<account-id>/` within the standard 5–15 minute delivery window.
4. Run the policy through **IAM Access Analyzer** (bucket policy findings) to confirm it doesn't grant unintended public or cross-account access beyond the log delivery service.
5. Attempt an unauthenticated / non-service write to the bucket and confirm it's denied.

## Sanitization Note

`YOUR-FLOW-LOG-BUCKET`, `YOUR-ACCOUNT-ID`, and `YOUR-REGION` are placeholders. Substitute your actual bucket name, 12-digit account ID, and region before deploying — never commit real values to a public repository.

## Reference

- [Publish flow logs to Amazon S3 — AWS documentation](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-s3.html)
- [Amazon S3 bucket permissions for flow logs — AWS documentation](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-s3-permissions.html)
