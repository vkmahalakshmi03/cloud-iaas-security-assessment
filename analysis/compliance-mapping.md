# Compliance Mapping

> **Assessment Date:** April 2025  
> **Version:** 1.0  
> **Scope:** 12 findings mapped against PCI-DSS v4.0, HIPAA Security Rule, SOC 2 Trust Criteria, FedRAMP

Each finding from this assessment is mapped to the specific compliance controls it violates or puts at risk. This mapping shows which regulatory requirements are impacted by default cloud configurations — before any hardening is applied.

---

## Finding-to-Compliance Matrix

| Finding ID | Finding | PCI-DSS v4.0 | HIPAA | SOC 2 | FedRAMP |
|------------|---------|-------------|-------|-------|---------|
| AZ-001 | NSG Flow Logs disabled | 10.2 — Log all access to network resources | §164.312(b) — Audit controls | CC7.2 — Monitor infrastructure | AU-12 — Audit generation |
| AZ-002 | Public blob access enabled | 7.1 — Restrict access by need to know | §164.312(a)(1) — Access control | CC6.1 — Logical access security | AC-3 — Access enforcement |
| AZ-003 | VM diagnostic logs disabled | 10.2 — Log all system components | §164.312(b) — Audit controls | CC7.2 — Monitor infrastructure | AU-6 — Audit review |
| AZ-004 | Double encryption not enforced | 3.5 — Protect stored cryptographic keys | §164.312(a)(2)(iv) — Encryption | CC6.1 — Logical access security | SC-28 — Protection at rest |
| AZ-005 | Over-permissioned RBAC | 7.2 — Restrict based on job function | §164.312(a)(1) — Access control | CC6.3 — Role-based access | AC-6 — Least privilege |
| AZ-006 | Firewall logs require activation | 10.2 — Log all security events | §164.312(b) — Audit controls | CC7.2 — Monitor infrastructure | AU-12 — Audit generation |
| GCP-001 | Default SA with Editor role | 7.2 — Restrict based on job function | §164.312(a)(1) — Access control | CC6.3 — Role-based access | AC-6 — Least privilege |
| GCP-002 | VPC Flow Logs disabled | 10.2 — Log all access to network resources | §164.312(b) — Audit controls | CC7.2 — Monitor infrastructure | AU-12 — Audit generation |
| GCP-003 | Data Access Audit Logs disabled | 10.2 — Log all access to cardholder data | §164.312(b) — Audit controls | CC7.2 — Monitor infrastructure | AU-12 — Audit generation |
| GCP-004 | Firewall rule logging disabled | 10.2 — Log all security events | §164.312(b) — Audit controls | CC7.2 — Monitor infrastructure | AU-12 — Audit generation |
| GCP-005 | CMEK not configured | 3.5 — Protect stored cryptographic keys | §164.312(a)(2)(iv) — Encryption | CC6.1 — Logical access security | SC-12 — Key management |
| GCP-006 | Alert thresholds too broad | 10.6 — Review logs for anomalies | §164.308(a)(5) — Security awareness | CC7.3 — Evaluate identified events | SI-4 — System monitoring |

---

## Compliance Impact by Framework

### PCI-DSS v4.0

PCI-DSS is the most directly impacted framework. The default configurations on both platforms violate multiple requirements:

**Requirement 7 (Restrict Access)** — AZ-002 (public blob access), AZ-005 (over-permissioned RBAC), and GCP-001 (Editor service account) all violate the principle of restricting access to cardholder data by business need-to-know. Default IAM on both platforms grants broader access than PCI allows.

**Requirement 10 (Log and Monitor)** — 7 of 12 findings directly violate Requirement 10. Network flow logs, data access logs, firewall logs, and VM diagnostics are all disabled by default. PCI requires logging of all access to network resources and cardholder data environments. Default configurations on both platforms produce none of this telemetry.

**Requirement 10.7 (Retention)** — PCI requires 12 months of audit log retention with a minimum of 3 months immediately available. Azure defaults to 30–90 days. GCP defaults to 30 days. Both fall short without manual configuration of log export and extended retention.

**Requirement 3 (Protect Stored Data)** — AZ-004 and GCP-005 rely on provider-managed encryption keys. PCI requires documented key management procedures with customer-controlled rotation. Default encryption on both platforms satisfies the encryption requirement but not the key management controls.

### HIPAA Security Rule

HIPAA's Security Rule applies to any environment handling protected health information (PHI).

**§164.312(a)(1) — Access Control** — The default IAM configurations on both platforms (AZ-002, AZ-005, GCP-001) fail to implement the HIPAA minimum necessary standard. Public blob access on Azure and Editor-level service accounts on GCP grant access far beyond what HIPAA's access control provisions require.

**§164.312(b) — Audit Controls** — HIPAA requires mechanisms to record and examine activity in systems containing PHI. With 7 of 12 findings being detection gaps, the default configurations provide insufficient audit capability. Data Access Audit Logs are disabled on both platforms — reads and writes to storage leave no record.

**§164.312(a)(2)(iv) — Encryption** — Both platforms encrypt data at rest by default (AES-256), satisfying the encryption addressable specification. However, HIPAA's broader security management requirements expect documented key management, which provider-managed keys alone may not satisfy for organizations with strict compliance postures.

**Retention** — HIPAA requires audit logs to be retained for 6 years. Default retention on both platforms (30–90 days) falls far short. Without explicit log routing to long-term storage, an organization is non-compliant from the first day of operations.

### SOC 2 Trust Services Criteria

SOC 2 applies to service organizations and covers security, availability, processing integrity, confidentiality, and privacy.

**CC6.1 (Logical Access Security)** — Default IAM on both platforms does not enforce least-privilege logical access. AZ-005 and GCP-001 represent access control gaps that a SOC 2 auditor would flag as control deficiencies.

**CC6.3 (Role-Based Access)** — SOC 2 expects role-based access controls that restrict access to authorized users. Default RBAC assignments on Azure and default service account roles on GCP both violate this criterion.

**CC7.2 (Monitor Infrastructure Components)** — SOC 2 requires monitoring of infrastructure for anomalies and security events. With network flow logs, firewall logs, and data access logs disabled by default, neither platform meets this criterion without post-provisioning hardening.

**CC7.3 (Evaluate Identified Events)** — GCP-006 (broad alert thresholds) directly impacts this criterion. Default SCC alerting does not produce actionable security events for evaluation.

### FedRAMP

FedRAMP is based on NIST 800-53 controls and applies to cloud services used by US federal agencies.

**AC-6 (Least Privilege)** — Both AZ-005 and GCP-001 violate the least privilege control. FedRAMP Moderate baseline requires documented access control policies enforcing minimum necessary permissions. Default IAM on both platforms does not satisfy this.

**AU-12 (Audit Generation)** — FedRAMP requires audit record generation for events defined in AU-2. With 7 detection-gap findings, the default configurations cannot generate the audit records FedRAMP requires. This is a direct control failure at the Moderate and High baselines.

**SC-28 (Protection of Information at Rest)** — Both platforms satisfy the encryption requirement with AES-256 by default. However, FedRAMP Moderate and High baselines increasingly expect customer-managed key controls (SC-12), which neither platform enables by default.

**SC-12 (Cryptographic Key Management)** — AZ-004 and GCP-005 indicate that default encryption relies on provider-managed keys. FedRAMP expects documented key management processes, which may require CMEK configuration to satisfy.

---

## Log Retention Compliance Gap

| Framework | Required Retention | Azure Default | GCP Default | Gap |
|-----------|-------------------|---------------|-------------|-----|
| PCI-DSS v4.0 | 12 months (3 months immediately available) | 30–90 days | 30 days | 9–11 months short |
| HIPAA | 6 years | 30–90 days | 30 days | ~6 years short |
| SOC 2 | Defined by organization (typically 12 months) | 30–90 days | 30 days | Organization-dependent |
| FedRAMP (Moderate) | 90 days online, 12 months total | 30–90 days | 30 days | Meets online minimum; total retention gap |

Both platforms require manual configuration of log export and extended storage to meet any of these retention requirements.
