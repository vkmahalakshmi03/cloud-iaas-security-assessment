# Executive Summary

**Assessment:** Default IaaS Security Posture — Microsoft Azure vs Google Cloud Platform
**Scope:** IAM, Network Security, Storage, Encryption, Logging & Monitoring
**Findings:** 12 total — 4 High · 5 Medium · 3 Low
**Frameworks:** CIS Benchmarks v2.0, NIST CSF, MITRE ATT&CK

---

## What Was Assessed

Both environments provisioned using free-tier accounts with default settings. No changes made before assessment began.

| Resource | Azure | GCP |
|----------|-------|-----|
| Compute | B1s VM, Windows Server 2022 | e2-micro, Debian 11 |
| Storage | General Purpose v2 Storage Account | Regional Cloud Storage Bucket |
| Network | Default NSG | Default VPC, Default Firewall Rules |
| IAM | Default RBAC assignments | Default project IAM, Default Compute SA |
| Monitoring | Defender for Cloud (free tier) | Security Command Center (free tier) |

---

## High Severity Findings

**GCP-001 — Default Service Accounts Assigned Editor Role**
GCP attaches the default Compute Engine SA with project-level Editor role automatically. Any code running on the VM can hit the metadata endpoint (`169.254.169.254`) to retrieve the token and get read/write access to everything in the project. No credentials needed beyond initial VM access.

**GCP-002 — VPC Flow Logs Disabled by Default**
No network traffic is logged. Lateral movement, data exfiltration, and C2 activity are undetectable at the network layer in the default state.

**AZ-001 — NSG Flow Logs Disabled by Default**
Same gap on Azure. No network flow data is recorded without manually configuring NSG flow logs.

**AZ-002 — Public Blob Access Enabled by Default**
Azure storage accounts allow public blob access unless explicitly disabled. Data in public containers is accessible without authentication and generates no audit log entry.

---

## Medium Severity Findings

- **AZ-003** — VM diagnostic logs not enabled; limits forensic capability during incident response
- **AZ-004** — Double encryption not enforced; single-layer platform-managed encryption only
- **AZ-005** — Broad RBAC at subscription level; one compromised account = full subscription access
- **GCP-003** — Data Access Audit Logs disabled; no record of what data was read or written
- **GCP-004** — Firewall rule logging disabled; broad default allow rules with zero visibility

---

## Low Severity Findings

- **GCP-005** — CMEK not configured on storage buckets
- **AZ-006** — Firewall diagnostic logs require manual activation
- **GCP-006** — No targeted alerting on IAM changes or service account key creation

---

## Detection Gaps by MITRE ATT&CK Tactic

| Tactic | Gap |
|--------|-----|
| TA0001 – Initial Access | No firewall logging to detect brute force or unauthorized SSH/RDP |
| TA0003 – Persistence | No alerting on IAM changes or service account key creation |
| TA0004 – Privilege Escalation | GCP metadata endpoint exposure; Azure broad RBAC |
| TA0007 – Discovery | No flow logs to detect internal network scanning |
| TA0008 – Lateral Movement | No east-west traffic visibility |
| TA0009 – Collection | No data access logging on storage reads |
| TA0010 – Exfiltration | No egress monitoring or storage access alerts |

---

## Where the Risk Differs by Platform

| Platform | Highest Risk | Why |
|----------|-------------|-----|
| GCP | IAM | Editor SA gives full project access from any VM compromise |
| Azure | Storage + Network | Public storage and no network logging create a dual blind spot |
| Both | Logging | Near-zero detection capability across all attack tactics by default |

---

## Top Priorities

1. Remove Editor role from GCP default service accounts
2. Enable NSG and VPC Flow Logs on both platforms
3. Disable public blob access on Azure storage
4. Enable Data Access Audit Logs on both platforms
5. Configure alerting on IAM changes

Full remediation steps: [`remediation/top-5-priorities.md`](./remediation/top-5-priorities.md)
