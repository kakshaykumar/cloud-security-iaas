# Cloud Security Assessment: Azure vs GCP Default IaaS Security Review

---

## Overview

Most cloud security incidents don't start with sophisticated attacks — they start with someone deploying a resource with default settings and not knowing what those defaults actually mean. This project evaluates the default security posture of Infrastructure-as-a-Service (IaaS) configurations in Microsoft Azure and Google Cloud Platform (GCP), with the goal of identifying exactly where those defaults fall short and what needs to be done to fix them.

Provisioned virtual machines and cloud storage on both platforms using only default settings, then systematically evaluated each across four dimensions: IAM and network controls, storage security, logging and monitoring, and encryption.

**Result:** 12 default-configuration gaps (6 per platform) and a 62-control CIS/NIST-aligned hardening checklist.

This is directly relevant to real enterprise cloud deployments. Organizations frequently assume "cloud-native" means "secure by default." This project shows exactly why that assumption is wrong on both platforms — and documents the specific configurations that need to change.

---

## Repository Structure

```
cloud-security-iaas/
│
├── README.md                                      ← You are here
│
├── docs/
│   ├── IaaS-Security-Posture-Azure-vs-GCP.pdf    ← Full research report (April 2025 — see CHANGELOG for later corrections)
│   ├── IaaS-Security-Posture-Presentation.pdf    ← Presentation deck (presented April 2025)
│
├── analysis/
│   ├── vm-configuration-comparison.md            ← Azure vs GCP VM defaults side-by-side
│   ├── storage-security-comparison.md            ← Storage access, encryption, public access defaults
│   ├── key-management-comparison.md              ← PMK vs CMK/CMEK, Key Vault vs Cloud KMS
│   └── logging-monitoring-gaps.md               ← Default logging gaps and what they miss
│
├── recommendations/
│   └── hardening-checklist.md                   ← Actionable hardening steps for both platforms
│
├── references.md                                 ← All sources and frameworks referenced
└── CHANGELOG.md                                  ← Version history and last-validated dates
```

---

## How to Reproduce This Evaluation

The following prerequisites and steps were used to conduct the analysis. Results may differ as Azure and GCP update their default configurations over time — see `CHANGELOG.md` for the validation date of findings in this repository.

**Prerequisites:**
- Active Azure subscription (free tier sufficient — B1s VM is within free tier limits)
- Active GCP account (free tier sufficient — e2-micro VM is within free tier limits)

**Azure steps:**
1. Log in to [portal.azure.com](https://portal.azure.com)
2. Create a new Resource Group in your preferred region (e.g., East US)
3. Deploy a B1s VM using the Azure Marketplace default Ubuntu 22.04 image — accept all networking defaults (this auto-creates an NSG)
4. Deploy an Azure Blob Storage account — check the **Allow Blob anonymous access** setting (off by default for new accounts since late 2023) and whether any Azure Policy prevents it from being turned on
5. Navigate to Network Watcher → Flow logs and confirm network flow logging is **off** by default
6. Navigate to Defender for Cloud and note the Secure Score baseline with no custom configuration

**GCP steps:**
1. Log in to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project
3. Deploy an e2-micro VM via Compute Engine using the default Debian image — accept all networking defaults (note the default service account assignment)
4. Deploy a Cloud Storage bucket — note that public access is **blocked** by default
5. Navigate to VPC Network → Flow Logs and confirm they are **off** by default
6. Navigate to Security Command Center and review the default findings posture

---

## Key Findings

### Azure — Default Configuration Gaps

| Area | Default State | Risk |
|---|---|---|
| Network Flow Logs (NSG / VNet) | **Disabled** | No network traffic visibility — lateral movement goes undetected |
| Anonymous Blob Access Guardrail | **Not enforced** | Anonymous access is off at the account level by default, but one account setting re-enables it and no policy blocks that by default |
| Double Encryption | **Disabled** | Platform-managed keys only — no customer key control by default |
| VM Monitoring / Diagnostics | **Optional** | Boot diagnostics and performance metrics require manual activation |
| Managed Identity Permissions | **Over-permissioned** | Default role assignments broader than least-privilege requires |
| Log Retention | 30 days (Activity Logs), up to 90 days (some Log Analytics tables) | Insufficient for PCI-DSS (12 months), HIPAA (6 years) |

### GCP — Default Configuration Gaps

| Area | Default State | Risk |
|---|---|---|
| VPC Flow Logs | **Disabled** | Same network visibility gap as Azure — anomalous traffic goes unlogged |
| Data Access Audit Logs | **Disabled** | No record of who reads or modifies stored data by default |
| Default Service Account Permissions | **Editor-level** | Broad project-wide permissions — significant privilege escalation surface |
| CMEK (Customer-Managed Keys) | **Not enabled** | Google-managed keys used by default — no customer key lifecycle control |
| Firewall Rule Logging | **Disabled** | Firewall activity is not logged unless manually configured |
| Log Retention | **30 days default** | Extendable via Cloud Storage routing, but requires manual setup |

### Where GCP Has an Edge by Default
- Public access prevention on Cloud Storage is enforced at the bucket and organization level; Azure relies on an account-level setting that can be switched back on
- IAM supports object-level policies natively — no need for SAS tokens
- Logs can be routed to Cloud Storage for cost-effective long-term retention beyond 30 days (Azure routes to Log Analytics Workspace at higher cost per GB)
- Workload Identity Federation eliminates long-lived service account keys

### Where Azure Has an Edge by Default
- Native Microsoft Sentinel SIEM integration — Chronicle requires additional setup on GCP
- Microsoft Entra ID + RBAC provides deep hybrid enterprise identity management
- Double encryption option (though not enabled by default)
- Tighter integration with enterprise compliance tools (Compliance Manager, Secure Score)

---

## The Shared Responsibility Model in Practice

Both Azure and GCP operate on the **Shared Responsibility Model** — a fundamental principle of cloud security that defines what the cloud provider secures versus what the customer is responsible for. In an IaaS context, the provider handles physical infrastructure, hypervisor security, and network hardware. Everything above that — operating system configuration, IAM policies, encryption key management, network logging, and application security — is the customer's responsibility.

This project is essentially a deep dive into that boundary. The default configurations on both platforms represent the minimum the provider does on your behalf. Every gap documented in this project — disabled flow logs, over-permissioned service accounts, platform-managed encryption keys — is a place where the customer's responsibility begins and where organizations routinely fall short because they assume the cloud is "secure by default."

It isn't. The defaults are a starting point, not a finished posture.

---

## The Bottom Line

Neither platform is "secure by default" in any meaningful enterprise sense. The defaults are a starting point, not a finish line. Both require the same fundamental hardening steps:

1. Enable network flow logs (VNet flow logs on Azure, VPC Flow Logs on GCP)
2. Enforce storage public-access restrictions with policy, not just default settings
3. Scope down service account and managed identity permissions to least privilege
4. Enable Customer-Managed Encryption Keys
5. Configure SIEM integration and customize alert thresholds
6. Extend log retention to meet compliance requirements

See [`recommendations/hardening-checklist.md`](recommendations/hardening-checklist.md) for the full platform-specific checklist.

---

## Skills and Concepts Demonstrated

- Cloud infrastructure provisioning on Azure and GCP
- IaaS security configuration assessment
- IAM design — RBAC, ABAC, service accounts, managed identities
- Encryption at rest and in transit — AES-256, CMEK, CSEK, key management
- Cloud logging and monitoring — Azure Monitor, Microsoft Sentinel, Cloud Logging, SCC
- CIS Benchmarks (v3.0.0) and NIST framework alignment
- Comparative security analysis and risk documentation
