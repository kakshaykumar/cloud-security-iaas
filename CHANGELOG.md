# Changelog

Cloud platform default configurations change with service updates. This file records when findings were last validated against actual platform behavior, so readers can assess currency.

---

## [1.0.0] — April 2025

**Initial release — findings validated against live Azure and GCP consoles**

### Validated against:
- Azure Portal (April 2025) — B1s VM, Azure Blob Storage, NSG defaults
- GCP Console (April 2025) — e2-micro VM, Cloud Storage bucket, VPC defaults

### Key default behaviors confirmed at time of evaluation:
- Azure NSG Flow Logs: **Disabled** by default
- Azure Blob Storage public access: **Enabled** by default
- Azure Log retention (Activity Logs): **30 days** by default
- GCP VPC Flow Logs: **Disabled** by default
- GCP Data Access Audit Logs: **Disabled** by default
- GCP Storage public access: **Blocked** by default
- GCP default service account permissions: **Editor-level** project-wide
- Both platforms: Customer-managed encryption keys **not activated** by default

### Frameworks referenced:
- CIS Microsoft Azure Foundations Benchmark v3.0.0
- CIS Google Cloud Platform Foundation Benchmark v3.0.0
- NIST SP 800-53 Rev 5
- NIST SP 800-207 (Zero Trust Architecture)

---

## Notes on Currency

Cloud provider defaults are not static. Notable changes that have affected this evaluation area in the past:

- **Azure** enabled Trusted Launch by default for Generation 2 VMs in supported regions (2022–2023). Earlier evaluations may not reflect this.
- **GCP** updated default service account behavior in some regions — verify current defaults when reproducing.
- **Microsoft** renamed Azure Active Directory to **Microsoft Entra ID** in July 2023. References to "Azure AD" in older documentation and tooling refer to the same service.

If you are reading this after mid-2026, re-validate the default states against current platform documentation before relying on the findings in this repository for production decisions.
