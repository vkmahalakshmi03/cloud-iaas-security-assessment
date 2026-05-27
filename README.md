# Cloud IaaS Security Posture Assessment: Azure vs GCP

**Scope:** Default IaaS configuration security evaluation across Microsoft Azure and Google Cloud Platform  
**Assessment Type:** Configuration review, IAM analysis, network exposure, storage access controls, logging and monitoring gaps  
**Frameworks:** CIS Benchmarks v2.0 (Azure & GCP), NIST CSF, MITRE ATT&CK

---

## Overview

Default cloud configurations are not secure configurations. This assessment evaluates what an organization actually gets when they spin up IaaS resources in Azure and GCP without modifying defaults — and what that means from an attacker's perspective.

Both platforms were provisioned using free-tier defaults. Azure VMs (B1s), GCP Compute Engine instances (e2-micro), storage buckets, and associated IAM configurations were assessed across five domains: IAM and access control, network security, storage security, encryption and key management, and logging and monitoring.

**12 findings were identified** — 4 High, 5 Medium, 3 Low — spanning both platforms. All findings map to CIS Benchmark controls and include remediation guidance.

> Conducted as part of graduate-level cloud security research at George Mason University (MS, Applied Information Technology — Cybersecurity Concentration).

---

## Key Findings Summary

| ID | Platform | Finding | Severity |
|----|----------|---------|----------|
| AZ-001 | Azure | NSG Flow Logs disabled by default | High |
| AZ-002 | Azure | Public blob access enabled on storage accounts | High |
| AZ-003 | Azure | VM diagnostic logs not enabled by default | Medium |
| AZ-004 | Azure | Double encryption not enforced by default | Medium |
| AZ-005 | Azure | Over-permissioned default RBAC role assignments | Medium |
| GCP-001 | GCP | Default service accounts assigned Editor role | High |
| GCP-002 | GCP | VPC Flow Logs disabled by default | High |
| GCP-003 | GCP | Data Access Audit Logs disabled by default | Medium |
| GCP-004 | GCP | Firewall rule logging disabled by default | Medium |
| GCP-005 | GCP | CMEK not configured on storage buckets | Low |
| AZ-006 | Azure | Firewall diagnostic logs require manual activation | Low |
| GCP-006 | GCP | Default alert thresholds too broad for threat detection | Low |

---

## Repository Structure

```
cloud-iaas-security-assessment/
├── findings/
│   ├── azure-findings.md          # Detailed Azure findings with evidence and remediation
│   ├── gcp-findings.md            # Detailed GCP findings with evidence and remediation
│   └── findings-summary.md       # Side-by-side platform comparison
├── methodology/
│   └── assessment-approach.md    # Environment setup, tooling, and evaluation criteria
├── remediation/
│   ├── azure-hardening-checklist.md
│   └── gcp-hardening-checklist.md
├── evidence/
│   └── README.md                  # Screenshots and config exports from test environments
└── references/
    └── frameworks.md              # CIS Benchmark controls, NIST CSF mappings
```

---

## Tools and Platforms Used

- Microsoft Azure Portal — VM and storage provisioning, NSG review, IAM role inspection
- Google Cloud Console — Compute Engine, Cloud Storage, IAM, VPC review
- Microsoft Defender for Cloud — default security posture dashboard review
- Google Security Command Center (SCC) — default findings review
- CIS Benchmarks for Azure and GCP — control mapping
- NIST CSF — risk categorization

---

## Platform Comparison: TL;DR

| Domain | Azure | GCP |
|--------|-------|-----|
| IAM defaults | RBAC available; over-permissioned defaults | Editor-role service accounts; high privilege escalation risk |
| Network logging | NSG Flow Logs OFF by default | VPC Flow Logs OFF by default |
| Storage access | Public blob access ON by default | Public access OFF by default ✓ |
| Encryption | AES-256 default; CMK requires configuration | AES-256 default; CMEK requires configuration |
| Audit logging | Activity logs on; Diagnostic logs require enablement | Cloud Audit Logs on; Data Access logs OFF by default |
| Native SIEM | Microsoft Sentinel (native) | Chronicle (third-party setup required) |
| Log retention | 30–90 days depending on service | 30 days default; free long-term via Cloud Storage |

---

## Remediation Approach

All findings include:
- Specific remediation steps (not generic advice)
- CIS Benchmark control reference
- Risk context mapped to MITRE ATT&CK where applicable

See [`remediation/`](./remediation/) for platform-specific hardening checklists.

---

## Related Work

- [Ransomware Threat Detection — Hybrid ML + Signature Model](https://github.com/)  
- [Network & Digital Forensics Lab Work](https://github.com/)

