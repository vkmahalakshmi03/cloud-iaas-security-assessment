# Key Findings

> **Assessment Date:** April 2025  
> **Version:** 1.0  
> **Scope:** Azure and GCP default IaaS configurations  
> **Findings:** 4 High · 5 Medium · 3 Low

---

## Assessment Outcome

12 security gaps were identified across Azure and GCP default IaaS configurations — 4 High, 5 Medium, 3 Low. The findings break down as follows:

| Category | Count | Findings |
|----------|-------|----------|
| Detection gaps (logging/monitoring disabled) | 7 | AZ-001, AZ-003, AZ-006, GCP-002, GCP-003, GCP-004, GCP-006 |
| Access control over-permissioning | 3 | AZ-005, GCP-001, AZ-002 |
| Encryption key control gaps | 2 | AZ-004, GCP-005 |

The dominant finding category is detection gaps. More than half of all findings relate to logging and monitoring capabilities that are available on both platforms but disabled by default. This means the primary risk in a default cloud deployment isn't exposure in the traditional sense — it's the inability to detect and investigate incidents when they occur.

---

## Risk Matrix

Findings plotted by likelihood (how easy is this to exploit in the default state) and impact (business and security consequence).

| | High Likelihood | Medium Likelihood | Low Likelihood |
|---|---|---|---|
| **Critical Impact** | GCP-001 | AZ-002 | — |
| **High Impact** | AZ-001, GCP-002 | AZ-005, GCP-004 | AZ-004 |
| **Medium Impact** | GCP-003 | AZ-003, GCP-006 | AZ-006, GCP-005 |

**Likelihood definition:**
- **High** — exploitable with minimal skill; default condition directly enables the attack
- **Medium** — requires some access or reconnaissance first
- **Low** — requires specific conditions or technical skill to exploit

---

## Critical Attack Scenarios

### GCP-001 — Default Editor Service Account

The highest-risk finding in the assessment. An attacker who gains code execution on any GCP VM can retrieve the default service account's OAuth token from the metadata endpoint and use it to access every resource in the project. The full exploit chain:

```
Attacker gains code execution on VM (web app vuln, SSRF, compromised container)
  → Query metadata endpoint for service account token
  → Token returned with Editor-level permissions
  → Access all Cloud Storage buckets, enumerate resources, read secrets
  → Create backdoor service account keys for persistent access
```

No credentials required. No privilege escalation tools needed. The metadata endpoint is accessible by default from every VM.

### AZ-002 — Public Blob Access

Azure storage accounts ship with public blob access enabled. If any container is set to "Blob" or "Container" access level, its contents are accessible to anyone on the internet. Combined with disabled data access logging, there's no way to determine what was accessed or when the exposure began.

### Shared Gap — No Network Visibility

Both platforms disable network flow logging by default. An attacker moving laterally between resources generates no network-level audit trail. This gap compounds every other finding — even if the initial compromise is detected through other means, reconstructing the attacker's movement through the environment is impossible without flow log data.

---

## Top Remediation Priorities

Ordered by risk reduction impact:

1. **Remove Editor role from GCP default service accounts** — eliminates the highest privilege escalation path in the assessment
2. **Enable NSG Flow Logs (Azure) and VPC Flow Logs (GCP)** — restores network visibility on both platforms
3. **Disable public blob access on Azure storage accounts** — one configuration change, immediate exposure reduction
4. **Enable Data Access Audit Logs on both platforms** — closes the data exfiltration blind spot
5. **Restrict default IAM roles and configure alerting on IAM changes** — reduces persistence and privilege escalation risk

---

## What These Findings Mean for Production Environments

These findings are not specific to test or free-tier environments. They reflect the exact configurations that production workloads inherit when infrastructure is provisioned through the Azure Portal or GCP Console without a hardening baseline applied.

**Forensic readiness is zero on day one.** Flow logs that were never enabled cannot be retroactively generated. If a storage account was publicly accessible for three months before someone noticed, there's no access log to determine what was read or by whom. Data Access Audit Logs weren't on, so there's no record. Incident response in an environment with default logging means starting from almost nothing.

**Default IAM creates lateral movement paths that persist until audited.** GCP's default service account with Editor permissions means every VM in the project can access every resource through the metadata endpoint. On Azure, over-permissioned RBAC at the subscription level grants broader access than any individual workload needs. These aren't misconfigurations in the traditional sense — they're the shipped defaults that persist until someone explicitly reviews and restricts them.

**Compliance gaps start accumulating immediately.** PCI-DSS requires 12 months of log retention. HIPAA requires 6 years. Both platforms default to 30-day retention. An organization that deploys and assumes the defaults satisfy compliance is accumulating audit risk from the first day of operations. The gap between default retention and regulatory requirements has to be addressed through explicit log routing and storage configuration.

**The hardening documented in this project is not optional.** It's the baseline that needs to be in place before any workload should be considered production-ready. The platform defaults are a starting point for getting infrastructure running — not for operating it securely.
