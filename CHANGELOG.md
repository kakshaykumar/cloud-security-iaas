# Changelog

Cloud platform default configurations change with service updates. This file records when findings were last validated against actual platform behavior, so readers can assess currency.

---

## [1.1.0] — November 2026

**Corrections after re-checking against Microsoft documentation**

- **Azure Blob anonymous access:** the April 2025 finding listed public access as *Enabled* by default. Microsoft changed this default in late 2023 — new storage accounts are created with anonymous blob access **disallowed** at the account level. The finding is reframed as a guardrail gap: the setting can be re-enabled per account and no policy prevents that by default. Gap count is unchanged at 12.
- **Azure flow logs:** Microsoft stopped allowing new NSG flow logs after June 30, 2025 and retires them on September 30, 2027. Recommendations now point to **VNet flow logs**, which are also off by default.
- The PDF report and presentation in `docs/` reflect the original April 2025 wording and have not been regenerated.

---

## [1.0.0] — April 2025

**Initial release — findings validated against live Azure and GCP consoles**

### Validated against:
- Azure Portal (April 2025) — B1s VM, Azure Blob Storage, NSG defaults
- GCP Console (April 2025) — e2-micro VM, Cloud Storage bucket, VPC defaults

### Key default behaviors confirmed at time of evaluation:
- Azure NSG Flow Logs: **Disabled** by default
- Azure Blob Storage public access: recorded as **Enabled** by default (corrected in 1.1.0 — see above)
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
