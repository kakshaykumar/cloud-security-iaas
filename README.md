# Evaluating the Security Posture of Default IaaS Configurations: Azure vs Google Cloud

*(Cloud Security Assessment Lead — IAM, Network Security & Recommendations)*

---

## Overview

Most cloud security incidents don't start with sophisticated attacks — they start with someone deploying a resource with default settings and not knowing what those defaults actually mean. This project evaluates the default security posture of Infrastructure-as-a-Service (IaaS) configurations in Microsoft Azure and Google Cloud Platform (GCP), with the goal of identifying exactly where those defaults fall short and what needs to be done to fix them.

We provisioned virtual machines and cloud storage on both platforms using only default settings, then systematically evaluated each across four dimensions: IAM and network controls, storage security, logging and monitoring, and encryption. The result is a direct, evidence-based comparison of where each platform starts you off — and what you still have to do yourself before you'd call it secure.

This is directly relevant to real enterprise cloud deployments. Organizations frequently assume "cloud-native" means "secure by default." This project shows exactly why that assumption is wrong on both platforms — and documents the specific configurations that need to change.

---

## Repository Structure

```
cloud-security-iaas/
│
├── README.md                                      ← You are here
│
├── docs/
│   ├── IaaS-Security-Posture-Azure-vs-GCP.pdf   ← Full research report
│   └── IaaS-Security-Posture-Presentation.pdf   ← Presentation deck (presented April 2025)
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
└── references.md                                 ← All sources and frameworks referenced
```

---

## What We Actually Did

This wasn't a desk research project. We provisioned real resources on both platforms:

- **Azure:** B1s VM via Azure Portal with default networking (NSG), authentication, and monitoring settings
- **GCP:** e2-micro VM via Google Cloud Console with default VPC firewall rules and IAM service account assignment

For storage, we provisioned:
- **Azure Blob Storage:** Default container with RBAC, verified public access settings
- **Google Cloud Storage:** Default bucket, verified IAM policy scope and public access defaults

We then examined each configuration against documented security best practices (CIS Benchmarks, NIST guidelines) and each platform's own security tooling (Defender for Cloud, Security Command Center).

---

## Key Findings

### Azure — Default Configuration Gaps

| Area | Default State | Risk |
|---|---|---|
| NSG Flow Logs | **Disabled** | No network traffic visibility — lateral movement goes undetected |
| Public Access on Blob Storage | **Enabled** | Storage containers publicly accessible unless explicitly restricted |
| Double Encryption | **Disabled** | Platform-managed keys only — no customer key control by default |
| VM Monitoring / Diagnostics | **Optional** | Boot diagnostics and performance metrics require manual activation |
| Managed Identity Permissions | Over-permissioned | Default role assignments broader than least-privilege requires |
| Log Retention | 30 days (Activity Logs), up to 90 days (some Log Analytics tables) | Insufficient for PCI-DSS (12 months), HIPAA (6 years) |

### GCP — Default Configuration Gaps

| Area | Default State | Risk |
|---|---|---|
| VPC Flow Logs | **Disabled** | Same network visibility gap as Azure — anomalous traffic goes unlogged |
| Data Access Audit Logs | **Disabled** | No record of who reads or modifies stored data by default |
| Default Service Account Permissions | Editor-level | Broad project-wide permissions — significant privilege escalation surface |
| CMEK (Customer-Managed Keys) | **Not enabled** | Google-managed keys used by default — no customer key lifecycle control |
| Firewall Rule Logging | **Disabled** | Firewall activity is not logged unless manually configured |
| Log Retention | 30 days default | Extendable via Cloud Storage routing, but requires manual setup |

### Where GCP Has an Edge by Default
- Public access on Cloud Storage is **disabled** by default — Azure enables it
- IAM supports object-level policies natively — no need for SAS tokens
- 365-day free log retention via Cloud Storage (Azure defaults to 30–90 days depending on log type)
- Workload Identity Federation eliminates long-lived service account keys

### Where Azure Has an Edge by Default
- Native Microsoft Sentinel SIEM integration — Chronicle requires additional setup on GCP
- Azure AD + RBAC provides deep hybrid enterprise identity management
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

1. Enable network flow logs (NSG Flow Logs on Azure, VPC Flow Logs on GCP)
2. Restrict public access on storage
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
- Cloud logging and monitoring — Azure Monitor, Sentinel, Cloud Logging, SCC
- CIS Benchmarks and NIST framework alignment
- Comparative security analysis and risk documentation
