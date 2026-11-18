# compliant-gcs-bucket (Lab 2.4)

This reusable Terraform module deploys a Google Cloud Storage bucket with a compliance baseline hardcoded inside it, so consumers can set business values (environment, retention) but cannot weaken the security posture. It enforces six NIST SP 800-53 controls: SC-12 (organization-owned Cloud KMS key with 90-day rotation), SC-13 and SC-28 (bucket encrypted at rest with that customer-managed key, CMEK), AC-3 (uniform bucket-level access and public access prevention enforced), AU-11 (versioning plus a retention policy, with production required to keep data at least 365 days), and CM-6 (required compliance labels merged on top of any consumer labels). The module also emits a machine-readable `compliance_attestation` output, captured as evidence in `evidence/lab-2-4/attestation.json`.

## Documented deviation: gcloud output in Git Bash on Windows

The lab's verification commands (`gcloud storage buckets describe ... --format="yaml(...)"`) print Windows line endings (CRLF) when run from Git Bash on Windows. The carriage returns make the output overwrite itself or render as one line with stray symbols, so it can look empty.

- **Workaround:** append `| tr -d '\r'` to the command to strip the carriage returns, for example:

```bash
gcloud storage buckets describe "$BUCKET_URL" \
  --format="yaml(uniform_bucket_level_access,public_access_prevention,labels,retention_policy)" | tr -d '\r'
```

- **Impact on controls:** none. Display only; the returned values (labels, public access prevention, retention, uniform access) are unchanged.
