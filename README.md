# Cloud IaaS Security Posture Assessment: Azure vs GCP

Default IaaS configurations on Azure and GCP were assessed across five domains — IAM, network security, storage, encryption, and logging — to identify what an organization is exposed to before any hardening is applied.

**12 findings identified: 4 High · 5 Medium · 3 Low**

---

## Assessment Scope

| | Azure | GCP |
|---|---|---|
| Compute | B1s VM, Windows Server 2022, East US | e2-micro, Debian 11, us-central1 |
| Storage | General Purpose v2 Storage Account | Regional Cloud Storage Bucket |
| Network | Default NSG | Default VPC + Default Firewall Rules |
| IAM | Default RBAC assignments | Default project IAM, Default Compute SA |
| Monitoring | Defender for Cloud (free tier) | Security Command Center (free tier) |

Both environments provisioned using free-tier defaults. No post-provisioning changes before assessment.

See [Environment Architecture](./architecture/environment-setup.md) for full setup and network topology.

---

## Findings

| ID | Platform | Finding | Severity |
|----|----------|---------|----------|
| GCP-001 | GCP | Default service accounts assigned Editor role | 🔴 High |
| GCP-002 | GCP | VPC Flow Logs disabled by default | 🔴 High |
| AZ-001 | Azure | NSG Flow Logs disabled by default | 🔴 High |
| AZ-002 | Azure | Public blob access enabled on storage accounts | 🔴 High |
| AZ-003 | Azure | VM diagnostic logs not enabled by default | 🟠 Medium |
| AZ-004 | Azure | Double encryption not enforced by default | 🟠 Medium |
| AZ-005 | Azure | Over-permissioned default RBAC role assignments | 🟠 Medium |
| GCP-003 | GCP | Data Access Audit Logs disabled by default | 🟠 Medium |
| GCP-004 | GCP | Firewall rule logging disabled by default | 🟠 Medium |
| GCP-005 | GCP | CMEK not configured on storage buckets | 🟡 Low |
| AZ-006 | Azure | Firewall diagnostic logs require manual activation | 🟡 Low |
| GCP-006 | GCP | Default alert thresholds too broad for threat detection | 🟡 Low |

Full findings with remediation: [`findings/`](./findings/)

---

## Key Numbers

- 7 of 12 findings are detection gaps — logging and monitoring defaults leave both platforms near-blind
- Zero network flow logging on either platform by default
- GCP-001: default Editor SA enables full project access from any compromised VM via the metadata endpoint
- AZ-002: public blob access is on by default — data reachable without authentication
- Both platforms: data access audit logs off by default, storage reads leave no trace

---

## Platform Comparison

| Domain | Azure | GCP |
|--------|-------|-----|
| IAM defaults | Broad RBAC at subscription level | Editor role on default service accounts |
| Network logging | NSG Flow Logs OFF | VPC Flow Logs OFF |
| Default firewall exposure | NSG required; no implicit allow | SSH/RDP open to 0.0.0.0/0 |
| Storage public access | ON by default | OFF by default |
| Data access logging | OFF by default | OFF by default |
| Native SIEM | Sentinel (built-in) | Chronicle (third-party) |
| Log retention default | 30–90 days | 30 days; free long-term via GCS |

---

## Tools Used

- Azure Portal, Azure Cloud Shell
- GCP Cloud Console, GCP Cloud Shell
- Microsoft Defender for Cloud
- Google Security Command Center
- CIS Benchmark for Azure v2.0
- CIS Benchmark for GCP v1.3
- NIST CSF, MITRE ATT&CK

---

## Repo Structure

```
cloud-iaas-security-assessment/
├── executive-summary.md
├── conclusion.md
├── architecture/
│   └── environment-setup.md
├── findings/
│   ├── azure-findings.md
│   ├── gcp-findings.md
│   ├── findings-summary.md
│   └── risk-matrix.md
├── methodology/
│   └── assessment-approach.md
├── remediation/
│   ├── azure-hardening-checklist.md
│   ├── gcp-hardening-checklist.md
│   └── top-5-priorities.md
├── evidence/
│   └── README.md
└── references/
    └── frameworks.md
```

---

## License

[MIT](./LICENSE)
