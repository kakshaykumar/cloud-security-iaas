# Logging & Monitoring: Default Gaps on Azure and GCP

**Last validated:** April 2025

The single most consistent finding across both platforms is this: the logs that matter most for security are not enabled by default.

---

## What Each Platform Provides

### Azure
| Tool | Purpose | Default State |
|---|---|---|
| Azure Monitor | Infrastructure and application metrics | On |
| Activity Logs | Subscription-level operations | On |
| Diagnostic Logs | Resource-level detail | Off — must be configured per resource |
| NSG Flow Logs | Network traffic between VMs and subnets | **Off** |
| Firewall Logs | Traffic allowed/denied by firewall rules | **Off** |
| Microsoft Defender for Cloud | Threat detection and security recommendations | Basic tier on; advanced off |
| Microsoft Sentinel | Cloud-native SIEM | Requires workspace setup |

### GCP
| Tool | Purpose | Default State |
|---|---|---|
| Cloud Monitoring | Infrastructure metrics | On |
| Cloud Logging | Centralized log aggregation | On (Admin Activity logs only) |
| Cloud Audit Logs — Admin Activity | Who did what at the control plane | On |
| Cloud Audit Logs — Data Access | Who read or modified actual data | **Off** |
| VPC Flow Logs | Network traffic between VM instances | **Off** |
| Firewall Rule Logs | Traffic matched against firewall rules | **Off** |
| Security Command Center | Vulnerability and threat findings | Basic tier on; Premium off |
| Chronicle | Cloud-native SIEM | Requires separate setup |

---

## The Gaps That Matter

### 1. Network Visibility — Both Platforms Fail Here

NSG Flow Logs (Azure) and VPC Flow Logs (GCP) are the foundation of network security monitoring. They're the logs that would tell you:
- Which IPs are communicating with your VMs
- Whether traffic is being allowed or blocked by your rules
- Whether lateral movement is occurring between internal hosts
- Whether data exfiltration is happening via unusual outbound connections

Both platforms disable these by default. This is arguably the most dangerous default gap in the entire evaluation — not because the feature doesn't exist, but because organizations routinely deploy infrastructure without ever turning it on.

### 2. Data Access Audit Logs (GCP)

On GCP, Admin Activity logs (who created, modified, or deleted resources) are on by default. But Data Access logs (who read data) are off. This means if an attacker with valid credentials reads your Cloud Storage objects or queries your database, there is no audit trail of that access unless you manually enable Data Access Audit Logs.

This is particularly concerning in regulated environments where data access auditing is a compliance requirement (HIPAA, PCI-DSS, SOC 2).

### 3. Alerting Calibration — Both Platforms

Both Defender for Cloud and Security Command Center have alert mechanisms, but their default thresholds are set conservatively to avoid overwhelming new users. In practice this means:
- Real threats may generate alerts at a lower severity than they deserve
- Alert fatigue from broad thresholds can cause analysts to deprioritize genuine incidents
- Custom alert rules and correlation logic are required before either system is operationally effective

### 4. Log Retention

| Platform | Default Retention | Compliance Risk |
|---|---|---|
| Azure | 30 days (Activity Logs), up to 90 days (some Log Analytics tables) | Insufficient for most compliance frameworks |
| GCP | 30 days (Cloud Logging) | Insufficient for most compliance frameworks |

PCI-DSS requires 12 months of audit log retention (3 months immediately accessible). HIPAA requires 6 years for certain records. Neither platform's default retention meets these requirements. Both offer longer retention — it just costs extra and must be manually configured.

---

## Comparison Summary

| Gap | Azure | GCP |
|---|---|---|
| Network traffic logging | NSG Flow Logs off | VPC Flow Logs off |
| Firewall logging | Off by default | Off by default |
| Data access auditing | Diagnostic Logs require per-resource setup | Data Access Audit Logs off globally |
| SIEM out of the box | Sentinel requires workspace + configuration | Chronicle requires separate provisioning |
| Default log retention | 30–90 days | 30 days |
| Alert tuning required | Yes — default thresholds too broad | Yes — same problem |

---

## What Should Be Enabled on Day One

For any production-grade deployment on either platform, these should be treated as non-negotiable baseline configurations — not optional enhancements:

**Azure:**
- Enable NSG Flow Logs for every Network Security Group
- Enable Diagnostic Settings on every critical resource (VMs, storage, Key Vault)
- Configure Microsoft Sentinel workspace and connect data connectors
- Set log retention to minimum 90 days in Log Analytics; route to Storage Account for longer-term retention
- Enable Defender for Cloud Standard tier on production subscriptions

**GCP:**
- Enable VPC Flow Logs on every subnet
- Enable Data Access Audit Logs at the organization or project level
- Enable Firewall Rule Logging for critical rules
- Route logs to Cloud Storage for long-term retention beyond 30 days
- Configure Security Command Center Premium for threat detection

---

## Verifying Enablement

Enabling a control and confirming it works are two different things. After each enablement step, verify delivery:

**Azure — Verify NSG Flow Logs:**
```bash
# In Azure Portal: Monitor → Network Watcher → NSG Flow Logs
# Confirm status shows "Enabled" and the storage account is receiving .json log files
# Or via CLI:
az network watcher flow-log show --resource-group <rg> --nsg <nsg-name>
```

**Azure — Verify Diagnostic Settings:**
```bash
az monitor diagnostic-settings list --resource <resource-id>
# Confirm logs are routing to Log Analytics Workspace or Storage Account
```

**GCP — Verify VPC Flow Logs:**
```bash
gcloud compute networks subnets describe <subnet-name> --region=<region> \
  --format="get(enableFlowLogs)"
# Should return: True
```

**GCP — Verify Data Access Audit Logs:**
```bash
gcloud projects get-iam-policy <project-id> --format=json | \
  grep -A5 "auditLogConfigs"
# Confirm DATA_READ and DATA_WRITE are listed for relevant services
```

**GCP — Verify logs are flowing to Cloud Storage (for long-term retention):**
```bash
gcloud logging sinks list
# Confirm a sink exists routing to your Cloud Storage bucket
```

---

*See also: [`vm-configuration-comparison.md`](vm-configuration-comparison.md) for network-level defaults | [`hardening-checklist.md`](../recommendations/hardening-checklist.md) for the full remediation checklist*
