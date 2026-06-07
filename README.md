# Cloud IaaS Security Posture Assessment: Azure vs GCP

## Overview

Cloud providers ship default IaaS configurations that are functional but not secure. Organizations that deploy infrastructure on Azure or GCP without applying post-provisioning hardening inherit security gaps across IAM, network controls, storage, encryption, and logging — most of which generate no alerts and produce no audit trail.

This project assesses exactly what those gaps are. Default configurations on both Microsoft Azure and Google Cloud Platform were provisioned and evaluated against CIS Benchmarks, NIST CSF, and MITRE ATT&CK to identify what an organization is exposed to on day one — before any security team touches the environment.

---

## Why This Assessment Exists

Most cloud security guidance focuses on what to enable. This project focuses on what's missing by default — and why that matters.

A VM deployed on GCP with default settings gets a service account with Editor-level access to the entire project. A storage account created on Azure has public blob access enabled by default. Neither platform enables network flow logging, data access audit logs, or meaningful detection coverage out of the box. These aren't theoretical risks. They're the actual starting state.

The goal is to document these defaults, quantify the exposure, and provide platform-specific remediation that maps directly to industry frameworks.

---

## Scope

| | Azure | GCP |
|---|---|---|
| Compute | B1s VM, Windows Server 2022, East US | e2-micro, Debian 11, us-central1 |
| Storage | General Purpose v2 Storage Account | Regional Cloud Storage Bucket |
| Network | Default NSG | Default VPC + Default Firewall Rules |
| IAM | Default RBAC assignments | Default project IAM, Default Compute SA |
| Monitoring | Defender for Cloud (free tier) | Security Command Center (free tier) |

Both environments provisioned using free-tier defaults. No post-provisioning changes applied before assessment.

---

## Findings at a Glance

**12 findings identified: 4 High · 5 Medium · 3 Low**

- 7 of 12 findings are detection gaps — logging and monitoring defaults leave both platforms near-blind
- Zero network flow logging on either platform by default
- GCP default service account grants Editor role — full project access from any compromised VM
- Azure storage accounts ship with public blob access enabled — data reachable without authentication
- Neither platform enables data access audit logs by default — storage reads leave no trace

---

## Frameworks Applied

- **CIS Benchmarks** — Azure v2.0, GCP v1.3 (control-level mapping for each finding)
- **NIST Cybersecurity Framework** — Risk categorization across Identify, Protect, Detect, Respond, Recover
- **MITRE ATT&CK** — Tactic and technique mapping for each finding to model attacker behavior against default configurations

---

## Tools Used

- Azure Portal, Azure Cloud Shell
- GCP Cloud Console, GCP Cloud Shell
- Microsoft Defender for Cloud
- Google Security Command Center

---

## Repository Structure

```
cloud-iaas-security-assessment/
├── README.md                              — Project overview and scope
├── analysis/
│   ├── analysis.md                        — Platform comparison, risk matrix, MITRE mapping, detection gaps
│   ├── azure-default-gaps.md              — Azure default configuration deficiencies (AZ-001 to AZ-006)
│   └── gcp-default-gaps.md                — GCP default configuration deficiencies (GCP-001 to GCP-006)
├── remediation/
│   ├── remediation.md                     — Fixes applied to close identified gaps
│   ├── azure-hardening-checklist.md       — Azure-specific hardening steps mapped to CIS controls
│   └── gcp-hardening-checklist.md         — GCP-specific hardening steps mapped to CIS controls
├── docs/
│   └── project-report.pdf                 — Full project report
├── overall-findings.md                    — Key findings, outcomes, and how defaults can be improved
├── references.md                          — Framework references and external sources
├── LICENSE                                — MIT
└── .gitignore
```

---

## How to Read This Repo

Start with the **analysis/** folder to understand what's deficient on each platform and how the gaps compare. Move to **remediation/** to see what fixes address those gaps and the step-by-step checklists for each provider. **overall-findings.md** ties it together — what was done, what came out of it, and how these findings help organizations harden default cloud deployments. The full project report is in **docs/**.

---

## License

[MIT](./LICENSE)
