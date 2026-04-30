# VM Configuration Comparison: Azure vs GCP Defaults

**Scope:** Default IaaS virtual machine configuration on both platforms
**Azure VM:** B1s (Free Tier) via Azure Portal
**GCP VM:** e2-micro (Free Tier) via Google Cloud Console

---

## Side-by-Side Defaults

| Feature | Azure | GCP |
|---|---|---|
| VM Creation Portal | Azure Portal | GCP Console |
| Free Tier Option | B1s | e2-micro |
| OS Options | Windows, Linux | Windows, Linux |
| Authentication | SSH Key or Password | SSH Key or Browser-based SSH |
| Network | VNet + NSG | VPC + Firewall Rules |
| Flow Logs | **Disabled by default** | **Disabled by default** |
| Monitoring | Boot diagnostics (optional) | Cloud Logging (optional) |
| SIEM | Microsoft Sentinel (native) | Chronicle (Google-native SIEM) |
| Audit Logs | Activity Logs, Diagnostic Settings | Cloud Audit Logs (Admin, Access, Data) |
| Log Retention | 30–90 days default | 30 days default, extendable via Cloud Storage |
| Compliance Tools | Azure Policy, Secure Score, Compliance Manager | Org Policy, SCC Recommendations, Access Transparency |

---

## Security Configuration: Default vs Hardened

| Security Element | Azure Default | Azure Hardened | GCP Default | GCP Hardened |
|---|---|---|---|---|
| Access Control | Azure AD + RBAC | Custom roles, just-in-time access | IAM + default service account | Scoped service account, Workload Identity |
| Encryption at Rest | AES-256 (PMK) | CMK via Key Vault | AES-256 (Google-managed) | CMEK via Cloud KMS |
| Encryption in Transit | TLS 1.2+ | TLS 1.3 enforced | TLS 1.2+ | TLS 1.3 enforced |
| Key Management | Azure Key Vault (not activated) | CMK/CSK enabled, rotation automated | Cloud KMS (not activated) | CMEK enabled, rotation scheduled |
| Threat Detection | Defender for Cloud (basic) | Defender with custom policies | Security Command Center (basic) | SCC Premium + custom findings |
| Network Logging | NSG Flow Logs off | NSG Flow Logs enabled + Sentinel | VPC Flow Logs off | VPC Flow Logs enabled + Chronicle |

---

## Identity and Access — The Real Gaps

### Azure
Azure uses Azure AD in conjunction with RBAC, which provides role assignments at subscription, resource group, or VM level. Managed Identities let VMs authenticate to other Azure services without storing credentials.

**The default gap:** Over-permissioned role assignments are common in default deployments. The built-in Contributor or Owner roles give far broader access than a VM actually needs to function. Most environments should be using custom roles scoped to specific resource operations.

NSG (Network Security Group) logging is off by default. This means network traffic patterns — including lateral movement between VMs — are invisible unless you explicitly enable NSG Flow Logs.

### GCP
GCP's IAM supports role bindings at multiple levels (organization → folder → project → resource → object). This granularity is a significant advantage for least-privilege enforcement. Workload Identity Federation allows external workloads to authenticate without creating long-lived service account keys.

**The default gap:** GCP automatically assigns the default Compute Engine service account to new VMs. That service account typically has Editor-level permissions on the entire project — meaning a compromised VM could modify or delete any project resource. This is a significant privilege escalation surface that needs to be addressed before production deployment.

---

## Network Visibility: Both Platforms Fail Here by Default

This is the most operationally significant gap on both platforms:

- **Azure** — NSG Flow Logs must be manually enabled. Without them, you have no record of which IPs are communicating with your VMs, no visibility into rejected traffic, and no baseline for anomaly detection.
- **GCP** — VPC Flow Logs must be manually enabled. Same problem — no network traffic record, no anomaly baseline.

Both platforms offer the tooling to fix this. Neither enables it out of the box.

The security consequence is direct: if an attacker gains access and begins lateral movement between VMs, neither platform will generate a log entry by default. Detection depends entirely on other signals (e.g., authentication anomalies, command-line activity). Enabling flow logs is the single highest-priority action for improving network visibility on either platform.

---

## Shielded VMs (GCP Advantage)

GCP supports Shielded VMs — instances that use Secure Boot, vTPM, and integrity monitoring to protect against rootkits and firmware-level attacks. This is a meaningful security capability that Azure doesn't offer in an equivalent default form.

For high-security workloads, Shielded VMs should be the default choice on GCP.
