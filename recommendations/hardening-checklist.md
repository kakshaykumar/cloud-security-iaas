# Cloud IaaS Hardening Checklist: Azure and GCP

Based on findings from our evaluation of default IaaS configurations. These are the specific configurations that must be changed from their defaults to achieve a production-ready security posture.

---

## Azure Hardening Checklist

### Identity and Access Management
- [ ] Replace broad built-in roles (Owner, Contributor) with custom roles scoped to minimum required permissions
- [ ] Enable just-in-time (JIT) VM access via Defender for Cloud — eliminates persistent inbound SSH/RDP
- [ ] Activate Managed Identities for all VMs that need to access other Azure services (avoid storing credentials in code or config)
- [ ] Review and scope down all Service Principal permissions
- [ ] Enable Microsoft Entra ID Conditional Access policies for management plane access
- [ ] Enable PIM (Privileged Identity Management) for all privileged role assignments

### Network Security
- [ ] Enable NSG Flow Logs on every Network Security Group
- [ ] Route Flow Logs to a Log Analytics Workspace or Storage Account
- [ ] Review and tighten NSG inbound rules — remove any `Any` source rules
- [ ] Enable Azure Firewall or a WAF for internet-facing resources
- [ ] Use Private Endpoints for storage accounts, databases, and Key Vault to remove public endpoints

### Storage Security
- [ ] **Disable public blob access** on all storage accounts (not the default — must be set manually)
- [ ] Enable Customer-Managed Keys via Azure Key Vault for all storage accounts holding sensitive data
- [ ] Configure automated key rotation in Key Vault
- [ ] Enable storage account diagnostic logging (read, write, delete operations)
- [ ] Apply resource locks on production storage accounts to prevent accidental deletion
- [ ] Use Private Endpoints for all storage accounts where possible

### Encryption
- [ ] Activate CMK via Azure Key Vault (platform-managed keys are the default — insufficient for regulated workloads)
- [ ] Enable double encryption where required by compliance policy
- [ ] Enforce TLS 1.2 minimum on all storage accounts and web endpoints
- [ ] Regularly rotate encryption keys per organizational policy

### Logging and Monitoring
- [ ] Deploy Microsoft Sentinel workspace and connect all data sources
- [ ] Enable Diagnostic Settings on every resource (VMs, storage, Key Vault, NSGs)
- [ ] Set Log Analytics retention to 90+ days; route older logs to Storage Account for compliance
- [ ] Configure custom alert rules in Sentinel — don't rely on default thresholds
- [ ] Enable Defender for Cloud Standard tier on all production subscriptions
- [ ] Configure notifications for high-severity alerts via email, on-call platforms (e.g., PagerDuty), or messaging tools (e.g., Slack) — use whatever fits your organization's incident workflow

### Compliance
- [ ] Apply Azure Policy initiatives (CIS Azure Benchmark, NIST SP 800-53)
- [ ] Review Secure Score recommendations and remediate high-priority items
- [ ] Enable Compliance Manager for regulatory tracking (HIPAA, GDPR, FedRAMP)

---

## GCP Hardening Checklist

### Identity and Access Management
- [ ] **Replace the default service account** on all VMs with a custom service account scoped to minimum required roles
- [ ] Disable the default Compute Engine service account or remove its Editor-level binding
- [ ] Use Workload Identity Federation for external workloads — eliminates long-lived service account keys
- [ ] Implement org-level IAM policies using Organization Policy Service
- [ ] Enable Access Transparency to audit Google employee access to your data
- [ ] Regularly audit service account key age — rotate or eliminate any keys older than 90 days (`gcloud iam service-accounts keys list --iam-account=<sa-email>`)

### Network Security
- [ ] **Enable VPC Flow Logs on every subnet** (disabled by default)
- [ ] Enable Firewall Rule Logging for critical allow and deny rules
- [ ] Restrict firewall rules to specific source IPs — remove `0.0.0.0/0` rules except where explicitly required
- [ ] Use VPC Service Controls to restrict API access to specific IP ranges
- [ ] Enable Private Google Access for VMs that need to reach Google APIs without public IPs
- [ ] Consider Shared VPC for multi-project environments to centralize network control

### Storage Security
- [ ] Verify Uniform Bucket-Level Access is enabled (enforces IAM over legacy ACLs)
- [ ] Audit all buckets for any `allUsers` or `allAuthenticatedUsers` IAM bindings
- [ ] Enable CMEK via Cloud KMS for all buckets containing sensitive data
- [ ] Configure automated key rotation in Cloud KMS
- [ ] Enable Cloud Storage audit logging (Data Access Audit Logs at the project level)
- [ ] Set object retention policies and lifecycle rules per data classification

### Encryption
- [ ] Enable CMEK (Customer-Managed Encryption Keys) via Cloud KMS — Google-managed keys are the default
- [ ] For highest-sensitivity data, consider CSEK (Customer-Supplied Encryption Keys)
- [ ] Configure key rotation schedules in Cloud KMS
- [ ] Enable Cloud HSM for key operations requiring hardware-backed protection

### Logging and Monitoring
- [ ] **Enable Data Access Audit Logs** at the organization or project level (off by default)
- [ ] Enable VPC Flow Logs on all subnets (off by default)
- [ ] Route logs to Cloud Storage for long-term retention beyond the 30-day default
- [ ] Enable Security Command Center Premium tier for advanced threat detection
- [ ] Configure Chronicle for SIEM capabilities and create custom detection rules
- [ ] Set up alerting policies in Cloud Monitoring for security-relevant metrics

### Compliance
- [ ] Apply Organization Policy constraints (CIS GCP Benchmark recommendations)
- [ ] Enable Security Command Center's compliance monitoring for relevant standards
- [ ] Configure continuous compliance posture monitoring using Security Command Center Premium built-in compliance reports, or a third-party CSPM tool (e.g., Wiz, Orca Security, or Prisma Cloud)
- [ ] Review SCC security findings and prioritize HIGH/CRITICAL remediations

---

## Priority Order (Both Platforms)

If you can only do a few things immediately, do these first:

1. **Enable network flow logs** — NSG Flow Logs (Azure) / VPC Flow Logs (GCP). Biggest visibility gain for least effort.
2. **Restrict public access on storage** — Azure requires explicit action; verify GCP buckets are clean.
3. **Scope down service account / managed identity permissions** — reduces blast radius of any compromise.
4. **Enable Data Access Audit Logs** (GCP) or Diagnostic Settings (Azure) — creates the audit trail for compliance.
5. **Enable CMEK/CMK** — required for most regulated workloads; gets you control over your encryption keys.

Everything else builds on these foundations.

---

*Based on CIS Microsoft Azure Foundations Benchmark v3.0.0 and CIS Google Cloud Platform Foundation Benchmark v3.0.0.*
*Also aligned with NIST SP 800-53 (Security and Privacy Controls for Information Systems) and NIST SP 800-144 (Guidelines on Security and Privacy in Public Cloud Computing).*
