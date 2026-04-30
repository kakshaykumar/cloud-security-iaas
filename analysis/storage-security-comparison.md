# Storage Security Comparison: Azure vs GCP Defaults

**Last validated:** April 2025
**Azure:** Azure Blob Storage (default container settings)
**GCP:** Google Cloud Storage (default bucket settings)

---

## Access Control at a Glance

| Feature | Azure Blob Storage | Google Cloud Storage |
|---|---|---|
| Primary access model | RBAC (storage account / container level) | IAM (project, bucket, or object level) |
| Object-level access | Shared Access Signatures (SAS) | Native IAM policies |
| Public access default | **Enabled** — must be explicitly disabled | **Disabled** — private by default |
| Temporary access tokens | SAS tokens (time-limited, permission-scoped) | Signed URLs |
| Secretless authentication | Managed Identities | Workload Identity Federation |

---

## Encryption

Both platforms encrypt all stored data with AES-256 by default. The difference is in key management and customer control.

| Feature | Azure | GCP |
|---|---|---|
| Encryption at rest | AES-256 (always on) | AES-256 (always on) |
| Default key type | Platform-Managed Keys (PMK) | Google-Managed Keys |
| Customer-Managed Keys | Azure Key Vault (CMK) — requires activation | Cloud KMS (CMEK) — requires activation |
| Customer-Supplied Keys | Supported (CSK) | Supported (CSEK) |
| Double encryption | Available but not default | Not offered |
| HSM support | Azure Dedicated HSM + Key Vault HSM | Cloud HSM (via Cloud KMS) |
| Encryption in transit | TLS 1.2+ / HTTPS required | TLS 1.2+ enforced |

**The key management gap on both platforms:** Neither Azure nor GCP activates customer-managed keys by default. In a regulated environment (HIPAA, PCI-DSS, FedRAMP), this is almost always required. The organization is responsible for enabling CMEK/CMK, configuring key rotation, and ensuring keys are stored separately from the data they protect.

---

## Public Access: The Single Biggest Practical Gap

**Azure:** Public access on blob storage is enabled by default. A new storage account created with default settings will allow public reads on any container explicitly marked public. Organizations that don't audit their storage configuration during setup risk exposing sensitive data publicly.

**GCP:** Public access is disabled by default. A new Cloud Storage bucket cannot be read publicly unless an IAM policy explicitly grants `allUsers` or `allAuthenticatedUsers` access. This is a stronger out-of-the-box posture.

This difference has real-world consequences. A significant number of cloud data exposures traced to misconfigured storage involve Azure blobs left with public access enabled. GCP's default-deny posture for public access eliminates this entire class of misconfiguration.

---

## Authentication Models

### Azure
- Microsoft Entra ID integration for authentication
- RBAC roles at storage account, container, or object scope
- Managed Identities for application authentication (no secrets in code)
- Supports SAML, OIDC, and OAuth 2.0 federation
- SAS tokens for fine-grained, time-limited access delegation

### GCP
- IAM policies at project, bucket, or individual object level
- Service accounts for application authentication
- Workload Identity Federation — external workloads authenticate without creating long-lived keys
- Supports SAML, JWT, and OAuth 2.0
- Signed URLs for time-limited access delegation

**GCP's advantage in access granularity:** GCP's native object-level IAM means you can apply different access policies to individual files within a bucket without needing a separate token mechanism. Azure achieves object-level granularity through SAS tokens, which work but add complexity — they need to be generated, distributed, and tracked separately.

---

## What Both Platforms Require You to Do Manually

Neither platform does these things for you by default:

- Enable CMEK/CMK — you must provision a key vault or KMS key, configure the encryption policy, and set up rotation schedules
- Restrict public access explicitly (Azure only — GCP handles this by default)
- Enable storage access logging — reads, writes, and deletes against your buckets/containers are not logged unless you turn on storage analytics or GCS audit logging
- Configure data lifecycle policies — automatic deletion or archival of aged data requires manual setup on both platforms

*See also: [`key-management-comparison.md`](key-management-comparison.md) for key management details | [`vm-configuration-comparison.md`](vm-configuration-comparison.md) for VM-level access controls*
