# Key Management: Azure vs GCP

**Last validated:** April 2025

**Scope:** Encryption key management defaults and options on Azure Blob Storage and GCP Cloud Storage, and across VM-attached storage.

Key management is where the gap between "encrypted by default" and "actually secure" becomes most visible. Both platforms encrypt everything at rest automatically — but who controls the keys, and how, determines whether that encryption is meaningful in a breach scenario or a compliance audit.

---

## The Three Key Models (Both Platforms)

| Model | Who holds the key | Who rotates it | Use case |
|---|---|---|---|
| **Platform-Managed Keys (PMK/Google-managed)** | Cloud provider | Cloud provider | Default — acceptable for non-sensitive workloads |
| **Customer-Managed Keys (CMK/CMEK)** | Customer, stored in Key Vault / Cloud KMS | Customer-defined schedule | Regulated workloads — HIPAA, PCI-DSS, FedRAMP |
| **Customer-Supplied Keys (CSK/CSEK)** | Customer, never sent to cloud | Customer | Highest control — keys never leave your infrastructure |

Both platforms default to platform-managed keys. Neither activates CMK or CMEK automatically. This means out-of-the-box, neither platform gives the customer any control over encryption key lifecycle.

---

## Azure Key Management

### Default State
Azure uses Platform-Managed Keys (PMK) by default for all storage and VM disk encryption. The encryption happens automatically — AES-256, always on — but Microsoft holds and manages the keys. Customers have no visibility into key rotation schedules and cannot revoke access to keys independently of revoking access to the service itself.

### Azure Key Vault
Azure Key Vault is the central key management service for customer-controlled encryption. It supports three configurations:

- **Customer-Managed Keys (CMK):** Keys are stored in Key Vault and used by Azure services to encrypt/decrypt data. The customer controls key creation, rotation, and deletion. Critically — if you delete the key, the encrypted data becomes permanently inaccessible. This is the "bring your own key" (BYOK) model most regulated workloads require.
- **Customer-Supplied Keys (CSK):** The customer provides the key material directly for each operation. Azure never stores the key — it's provided per-request. Highest control, highest operational complexity.
- **Managed HSM:** Hardware Security Module backed key storage — keys are generated and stored in FIPS 140-2 Level 3 validated hardware. Required for some government and financial compliance frameworks.

### Double Encryption
Azure offers an additional layer — encrypting already-encrypted data with a second key from a different key management system. This provides protection against a compromise of one encryption layer. **Not enabled by default.** Must be explicitly opted in during storage account creation.

### Default Gap
Azure's default key posture means:
- You cannot audit who accessed your encryption keys
- You cannot revoke key access without deleting the storage account
- You have no control over when Microsoft rotates the underlying key material
- Double encryption protection is unavailable unless manually configured

---

## GCP Key Management

### Default State
GCP uses Google-managed encryption keys by default — functionally identical to Azure's PMK model. All data is encrypted with AES-256, Google manages the keys, and customers have no control over key lifecycle.

### Cloud KMS
Google Cloud Key Management Service provides customer-controlled encryption across GCP services:

- **Customer-Managed Encryption Keys (CMEK):** Keys are created and stored in Cloud KMS. GCP services use these keys to encrypt data, but the customer controls rotation, deletion, and access policies. Deleting the key renders the data permanently unreadable — same as Azure CMK.
- **Customer-Supplied Encryption Keys (CSEK):** Keys are provided by the customer for each API request. Cloud KMS never stores them. GCP uses them to encrypt/decrypt data for that specific operation only.
- **Cloud HSM:** Keys are stored and operations are performed in FIPS 140-2 Level 3 validated HSMs. Provides hardware-backed key security for high-compliance environments.
- **Cloud External Key Manager (EKM):** Keys are stored entirely outside GCP — in an on-premises HSM or a third-party key management service. GCP can only access the key when the external system authorizes it. Strongest isolation model available.

### Default Gap
GCP's default key posture shares the same fundamental problem as Azure:
- Google-managed keys mean Google could theoretically access your encrypted data
- No customer visibility into key rotation timing or access logs
- CMEK must be manually enabled per service, per resource — it does not apply retroactively
- Default service accounts accessing Cloud KMS would have overly broad permissions without explicit scoping

---

## Side-by-Side Comparison

| Feature | Azure | GCP |
|---|---|---|
| Default key type | Platform-Managed Keys (PMK) | Google-Managed Keys |
| Customer-managed option | Azure Key Vault CMK | Cloud KMS CMEK |
| Customer-supplied option | CSK (per-request) | CSEK (per-request) |
| HSM-backed keys | Key Vault Managed HSM | Cloud HSM |
| External key storage | Supported via Key Vault | Cloud External Key Manager (EKM) |
| Double encryption | Available, not default | Not offered |
| Key rotation — default | Microsoft-controlled, opaque | Google-controlled, opaque |
| Key rotation — CMK/CMEK | Customer-configurable | Customer-configurable |
| Retroactive CMEK application | No — must be set at creation | No — must be set at creation |
| Key deletion effect | Data permanently inaccessible | Data permanently inaccessible |

---

## The Shared Responsibility Line on Key Management

This is where the Shared Responsibility Model becomes concrete. When you use platform-managed keys:

- **Provider is responsible for:** Key generation, storage, rotation, and protection
- **You are responsible for:** Nothing — but you also have no control or visibility

When you enable CMK/CMEK:

- **Provider is responsible for:** Encrypting and decrypting data when your key authorizes it
- **You are responsible for:** Key creation, access policies, rotation schedules, backup, and ensuring keys aren't accidentally deleted

The shift from platform-managed to customer-managed keys moves significant responsibility to you — but also gives you the control that compliance frameworks actually require.

---

## What Needs to Be Done on Both Platforms

Neither platform activates CMK/CMEK by default. For any workload subject to regulatory requirements, this must be addressed manually:

**Azure:**
```
1. Create an Azure Key Vault instance
2. Generate or import a key
3. Enable CMK on the storage account, VM disk, or database
4. Set an automated key rotation policy (Key Vault supports this natively)
5. Configure Key Vault access policies to restrict which identities can use the key
```

**GCP:**
```
1. Enable the Cloud KMS API
2. Create a key ring and cryptographic key
3. Grant the service account the roles/cloudkms.cryptoKeyEncrypterDecrypter role (scoped only to that key)
4. Specify the CMEK key when creating the storage bucket, VM disk, or database
5. Set a rotation period on the key in Cloud KMS
```

---

*Both platforms require manual configuration to move from platform-managed to customer-managed keys. This is not optional for HIPAA, PCI-DSS, FedRAMP, or any framework that requires customer key custody.*

*See also: [`storage-security-comparison.md`](storage-security-comparison.md) for storage access controls | [`hardening-checklist.md`](../recommendations/hardening-checklist.md) for CMEK enablement steps*
