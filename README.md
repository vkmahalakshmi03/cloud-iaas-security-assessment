# Cloud IaaS Security Posture Assessment: Azure vs GCP

## Purpose

Cloud platforms ship default IaaS configurations that prioritize ease of deployment over security. Organizations provisioning infrastructure on Azure or GCP without applying post-deployment hardening inherit a set of security gaps that most teams don't discover until something goes wrong — or until an assessment like this one surfaces them.

This project evaluates the default security posture of Infrastructure-as-a-Service configurations on Microsoft Azure and Google Cloud Platform. Both platforms were provisioned with default settings across compute, storage, networking, IAM, and monitoring. No security hardening was applied before the assessment began. The objective was to document what each platform exposes by default, compare how Azure and GCP differ in their approach to out-of-the-box security, and provide remediation guidance mapped to CIS Benchmarks, NIST CSF, and MITRE ATT&CK.

---

## What Was Done

The assessment followed a structured approach across four phases:

**Environment Provisioning** — Virtual machines and cloud storage were deployed on both platforms using default wizard settings. Azure: B1s VM with Windows Server 2022 and a General Purpose v2 Storage Account. GCP: e2-micro VM with Debian 11 and a Regional Cloud Storage Bucket. Default networking, IAM assignments, and monitoring tools were left untouched.

**Configuration Assessment** — Each environment was evaluated across five security domains: IAM and access controls, network security and segmentation, storage access and public exposure, encryption at rest and in transit, and logging and monitoring coverage. Default firewall rules, identity assignments, encryption settings, and audit log configurations were inspected using Azure Cloud Shell, GCP Cloud Shell, Microsoft Defender for Cloud, and Google Security Command Center.

**Comparative Analysis** — Findings were compared side by side to identify where each platform's defaults create exposure and where they diverge. This covered IAM granularity (Azure RBAC vs GCP project-level IAM), storage access models (Shared Access Signatures vs object-level IAM), encryption options (Key Vault and CMK vs Cloud KMS and CMEK), key management capabilities, and logging and SIEM integration differences.

**Risk Evaluation and Remediation** — Each finding was mapped to CIS Benchmark controls, categorized under NIST CSF functions, and tied to MITRE ATT&CK tactics to model what an attacker could actually do with the gap. Severity was assigned based on exploitability, blast radius, and whether the gap creates a detection blind spot. Platform-specific hardening checklists were developed to address each finding.

---

## Assessment Scope

Both environments provisioned on free-tier accounts. No post-provisioning changes applied before assessment.

| | Azure | GCP |
|---|---|---|
| Compute | B1s VM, Windows Server 2022, East US | e2-micro, Debian 11, us-central1 |
| Storage | General Purpose v2 Storage Account | Regional Cloud Storage Bucket |
| Network | Default NSG | Default VPC + Default Firewall Rules |
| IAM | Default RBAC assignments | Default project IAM, Default Compute SA |
| Monitoring | Defender for Cloud (free tier) | Security Command Center (free tier) |

**Assessment conducted:** April 2025. Cloud provider defaults may change — findings reflect configurations observed at the time of assessment.

---

## Default Configuration Comparison

A core part of this project is understanding how Azure and GCP differ in what they ship by default — not after tuning, but on day one. Below is a summary of the key differences observed across each domain. The full domain-by-domain comparison is in [analysis/default-config-comparison.md](./analysis/default-config-comparison.md).

**IAM and Access Controls** — Azure uses Entra ID (formerly Azure AD) with Role-Based Access Control, assigning roles at the subscription or resource group level. Default role assignments tend to be broader than necessary. GCP's IAM supports role bindings from organization level down to individual objects, offering more granular scoping — but GCP undermines this by assigning its default Compute Engine service account the Editor role, which grants project-wide access to any VM that uses it.

**Storage Security** — Azure enables public blob access by default on storage accounts. Data is reachable without authentication unless this is explicitly disabled. GCP takes the opposite approach — public access is blocked by default. On access control granularity, Azure uses RBAC at the container level with Shared Access Signatures (SAS) for object-level access, while GCP supports native IAM bindings down to individual objects without needing a separate token mechanism.

**Encryption** — Both platforms encrypt data at rest using AES-256 with platform-managed keys by default. Azure offers a double encryption option (CMK layered over PMK) but doesn't enable it by default. GCP uses Google-managed keys with options for CMEK and CSEK, but these require manual configuration. Key management on Azure is centralized through Key Vault with support for BYOK and dedicated HSMs. GCP offers Cloud KMS and Cloud HSM with similar capabilities but does not natively support Hold Your Own Key (HYOK).

**Logging and Monitoring** — This is where both platforms share the most significant gap. Azure provides Azure Monitor, Activity Logs, Defender for Cloud, and Sentinel as a native SIEM — but NSG Flow Logs and firewall diagnostic logs are disabled by default. GCP provides Cloud Logging, Cloud Monitoring, Audit Logs, and Security Command Center — but VPC Flow Logs, firewall rule logging, and Data Access Audit Logs are all disabled by default. Both platforms effectively ship with network-level blindness until logging is manually enabled.

---

## Findings

**12 findings identified: 4 High · 5 Medium · 3 Low**

| ID | Platform | Finding | MITRE Tactic | Severity |
|----|----------|---------|--------------|----------|
| GCP-001 | GCP | Default service account assigned Editor role — full project access from any VM via metadata endpoint | Privilege Escalation (T1078.004) | High |
| GCP-002 | GCP | VPC Flow Logs disabled — no network traffic visibility | Defense Evasion (T1562.008) | High |
| AZ-001 | Azure | NSG Flow Logs disabled — lateral movement produces no log trail | Defense Evasion (T1562.008) | High |
| AZ-002 | Azure | Public blob access enabled on storage accounts — data reachable without authentication | Initial Access (T1530) | High |
| AZ-003 | Azure | VM diagnostic logs not enabled — no OS-level telemetry | Defense Evasion (T1562.008) | Medium |
| AZ-004 | Azure | Double encryption not enforced — platform-managed keys only | Collection (T1530) | Medium |
| AZ-005 | Azure | Over-permissioned default RBAC assignments at subscription level | Privilege Escalation (T1078.004) | Medium |
| GCP-003 | GCP | Data Access Audit Logs disabled — storage reads and writes leave no trace | Defense Evasion (T1562.008) | Medium |
| GCP-004 | GCP | Firewall rule logging disabled — no record of allowed or denied traffic | Defense Evasion (T1562.008) | Medium |
| GCP-005 | GCP | CMEK not configured — no customer control over encryption key lifecycle | Collection (T1530) | Low |
| AZ-006 | Azure | Firewall diagnostic logs require manual activation | Defense Evasion (T1562.008) | Low |
| GCP-006 | GCP | Default SCC alert thresholds too broad for actionable threat detection | Defense Evasion (T1562.006) | Low |

Seven of the twelve findings fall under Defense Evasion — not because attackers are actively evading defenses, but because the defenses don't exist yet. The dominant pattern across both platforms is disabled logging. Network flow logs, data access records, and firewall activity are all off by default. The platforms aren't misconfigured in the traditional sense — they're just not configured to produce the telemetry that detection depends on.

Full finding details in [analysis/azure-default-gaps.md](./analysis/azure-default-gaps.md) and [analysis/gcp-default-gaps.md](./analysis/gcp-default-gaps.md).

---

## Hardening Approach

Each finding has a corresponding remediation mapped to CIS Benchmark controls and NIST CSF functions. The hardening checklists in [remediation/](./remediation/) walk through the specific steps for each platform — what to enable, what to restrict, what to configure, and which CIS control it satisfies.

The priority order is straightforward: restore visibility first (enable all logging and flow logs), then restrict access (scope down IAM, disable public storage access), then strengthen encryption (configure customer-managed keys). Detection gaps are prioritized over exposure gaps because you can't respond to what you can't see.

---

## Real-World Implications

These findings aren't specific to test environments. They reflect the exact configurations that production workloads inherit when teams deploy through the Azure Portal or GCP Console without a hardening baseline. The practical implications for enterprise environments:

Organizations relying on default logging configurations have no forensic data if an incident occurs. Flow logs that were never enabled can't be retroactively generated. If a storage account was publicly accessible for three months before someone noticed, there's no access log to determine what was read or by whom — because Data Access Audit Logs weren't on.

Default IAM assignments create lateral movement paths that persist until someone audits them. GCP's default service account with Editor permissions means every VM in the project can access every resource in the project through the metadata endpoint. On Azure, over-permissioned RBAC assignments at the subscription level give broader access than any individual workload needs.

Compliance requirements compound the problem. PCI-DSS requires 12 months of log retention. HIPAA requires 6 years. Both platforms default to 30-day retention. An organization that deploys and assumes the defaults meet compliance is accumulating audit risk from day one.

The hardening steps documented in this project are not optional security enhancements — they're the baseline that needs to be in place before any workload should be considered production-ready.

---

## Repository Structure

```
cloud-iaas-security-assessment/
│
├── README.md                                  ← Full project overview
│
├── analysis/
│   ├── default-config-comparison.md           ← Azure vs GCP defaults compared domain by domain
│   ├── azure-default-gaps.md                  ← Azure deficiencies: AZ-001 through AZ-006
│   └── gcp-default-gaps.md                    ← GCP deficiencies: GCP-001 through GCP-006
│
├── remediation/
│   ├── key-findings.md                        ← Key findings, outcomes, and real-world implications
│   ├── azure-hardening-checklist.md           ← Azure hardening steps mapped to CIS controls
│   └── gcp-hardening-checklist.md             ← GCP hardening steps mapped to CIS controls
│
├── docs/
│   └── project-report.pdf                     ← Full project report
│
├── references.md                              ← Framework and tool references
├── LICENSE                                    ← MIT
└── .gitignore
```

---

## Tools and Frameworks

**Assessment tools:** Azure Portal, Azure Cloud Shell, GCP Cloud Console, GCP Cloud Shell, Microsoft Defender for Cloud, Google Security Command Center

**Frameworks:** CIS Benchmark for Azure v2.0, CIS Benchmark for GCP v1.3, NIST Cybersecurity Framework, MITRE ATT&CK

---

## License

[MIT](./LICENSE)
