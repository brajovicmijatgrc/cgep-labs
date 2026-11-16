# Compliant S3 primitive (Lab 2.3)

This Terraform primitive deploys a single Amazon S3 bucket locked down to a security baseline and enforces five NIST SP 800-53 controls: SC-28 (AES-256 server-side encryption at rest), AC-3 (all four S3 public access block settings enabled), AU-3 (server access logging to a dedicated log bucket), AU-6 (logs kept in a separate, encrypted, non-public bucket for review), and CM-6 (versioning plus four required compliance tags applied to every resource through provider `default_tags`). Evidence of the deployed configuration is exported as Terraform plan and state JSON to `evidence/lab-2-3/`.

## Controls and where they live

| Control | Implementation |
|---------|----------------|
| SC-28 | `aws_s3_bucket_server_side_encryption_configuration` (AES256) on both buckets |
| AC-3  | `aws_s3_bucket_public_access_block` with all four flags `true` on both buckets |
| AU-3  | `aws_s3_bucket_logging` to the log bucket, prefix `access-logs/` |
| AU-6  | Separate log bucket, encrypted and non-public |
| CM-6  | `aws_s3_bucket_versioning` + `default_tags` (Project, Environment, ManagedBy, ComplianceScope) |

## Documented deviation: region

The lab guide specifies `us-east-1`. This deployment uses **`eu-north-1`**.

- **Reason:** the AWS account was created through AWS's new project-based sign-up (Free plan). AWS applies a managed Service Control Policy that pins the project to its home region (`eu-north-1`) and denies regional API calls elsewhere. `s3:CreateBucket` in `us-east-1` returned `AccessDenied ... explicit deny in a service control policy`; the same call in `eu-north-1` succeeded.
- **Impact on controls:** none. All five controls are implemented identically; only the provider region changed.
- **Verification commands** use `--region eu-north-1` instead of `us-east-1`.

## Usage

```bash
terraform init
terraform plan -out=tfplan -var="project_name=cgep-lab" -var="environment=dev"
terraform apply -auto-approve tfplan
```
