# Compliance policies (OPA / Rego)

Policy-as-code library evaluated against `terraform show -json` plan output (GCP).
Framework: NIST SP 800-53 Rev. 5.

| Policy file | Control | Severity | What it checks | Remediation |
|---|---|---|---|---|
| `sc28_encryption.rego` | SC-28 Protection of Information at Rest | high | Every `google_storage_bucket` uses a customer-managed encryption key (CMEK) | Add `encryption { default_kms_key_name = ... }` referencing a KMS key you control |
| `ac3_no_public.rego` | AC-3 Access Enforcement | critical | Buckets enforce uniform bucket-level access and `public_access_prevention = "enforced"`; firewalls do not open ports 22/3389 to `0.0.0.0/0` | Set `uniform_bucket_level_access = true`, `public_access_prevention = "enforced"`; narrow `source_ranges` or remove the rule |
| `cm6_required_tags.rego` | CM-6 Configuration Settings | medium | Resources carry labels `project`, `environment`, `managed_by`, `compliance_scope` | Add the four required labels |

## Tests

Unit tests with compliant and non-compliant fixtures live in `policies/tests/`.

    opa test -v policies/    # PASS: 8/8

## Running against a plan

    opa eval -d policies -i terraform/primitives/policy-fixture/plan.json data.compliance.<sc28|ac3|cm6>.deny --format=pretty

## Evidence

`evidence/lab-3-3/opa-test-results.json` — JSON output of the full test run.