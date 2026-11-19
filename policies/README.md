# Compliance policies (OPA / Rego)

Policy-as-code library evaluated against `terraform show -json` plan output (GCP and AWS).
Framework: NIST SP 800-53 Rev. 5. Each control ID has one policy per cloud.

| Policy file | Cloud | Control | Severity | What it checks | Remediation |
|---|---|---|---|---|---|
| `sc28_encryption.rego` | GCP | SC-28 Protection of Information at Rest | high | Every `google_storage_bucket` uses a customer-managed encryption key (CMEK) | Add `encryption { default_kms_key_name = ... }` referencing a KMS key you control |
| `sc28_encryption_aws.rego` | AWS | SC-28 Protection of Information at Rest | high | Every `aws_s3_bucket` has an `aws_s3_bucket_server_side_encryption_configuration` referencing it | Add a server-side encryption configuration resource for the bucket |
| `ac3_no_public.rego` | GCP | AC-3 Access Enforcement | critical | Buckets enforce uniform bucket-level access and `public_access_prevention = "enforced"`; firewalls do not open ports 22/3389 to `0.0.0.0/0` | Set `uniform_bucket_level_access = true`, `public_access_prevention = "enforced"`; narrow `source_ranges` or remove the rule |
| `ac3_no_public_aws.rego` | AWS | AC-3 Access Enforcement | critical | Every `aws_s3_bucket` has an `aws_s3_bucket_public_access_block` referencing it with all four flags `true` | Add a public access block with `block_public_acls`, `block_public_policy`, `ignore_public_acls`, `restrict_public_buckets` all `true` |
| `cm6_required_tags.rego` | GCP | CM-6 Configuration Settings | medium | Resources carry labels `project`, `environment`, `managed_by`, `compliance_scope` | Add the four required labels |
| `cm6_required_tags_aws.rego` | AWS | CM-6 Configuration Settings | medium | Resources carry tags `Project`, `Environment`, `ManagedBy`, `ComplianceScope` (including provider `default_tags`) | Add the missing tags or use provider `default_tags` |

## Tests

Unit tests with compliant and non-compliant fixtures live in `policies/tests/` (GCP policies).

    opa test -v policies/    # PASS: 8/8

## Running against a plan

    opa eval -d policies -i terraform/primitives/policy-fixture/plan.json data.compliance.<sc28|ac3|cm6>.deny --format=pretty
    conftest test --policy policies --namespace compliance.<sc28_aws|ac3_aws|cm6_aws> terraform/primitives/compliant-s3/plan.json

## CI gate (Lab 3.4)

    bash scripts/policy-gate.sh --workspace terraform/primitives/compliant-s3

Runs the three `*_aws` namespaces with Conftest and exits non-zero on any violation.
GCP namespaces are deliberately excluded: on an AWS plan they would pass with zero coverage.

## Evidence

- `evidence/lab-3-3/opa-test-results.json` - JSON output of the full OPA unit test run.
- `evidence/lab-3-4/conftest-pass.json` - gate run on the compliant Lab 2.3 plan (PASS).
- `evidence/lab-3-4/conftest-fail.json` - gate run on a copy with the SC-28 encryption resource removed (FAIL).

## Deviations from the course guide

- Conftest v0.71.1 installed manually on Windows (no winget package); SHA256 verified against the release `checksums.txt`.
- Lab 3.4 Step 8: besides deleting the `primary` encryption resource from `/tmp/broken/main.tf`, the `encryption_algorithm` output in `/tmp/broken/outputs.tf` also had to be removed, because it references the deleted resource and `terraform plan` fails otherwise.
