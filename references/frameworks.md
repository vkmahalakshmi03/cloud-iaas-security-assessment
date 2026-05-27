# Framework and Compliance References

## CIS Benchmarks

### CIS Microsoft Azure Foundations Benchmark v2.0
Used as the primary control reference for all Azure findings.

| Control Area | Relevant Controls Used |
|---|---|
| Identity and Access Management | 1.1, 1.2, 1.14, 1.20, 1.21 |
| Microsoft Defender for Cloud | 2.1 |
| Storage Accounts | 3.1, 3.2, 3.5, 3.7, 3.8, 3.10 |
| Logging and Monitoring | 5.1, 5.2, 5.3 |
| Networking | 6.1, 6.4, 6.5, 6.6 |
| Virtual Machines | 7.2 |
| Key Vault | 8.1, 8.4, 8.6 |

Reference: https://www.cisecurity.org/benchmark/azure

---

### CIS Google Cloud Platform Foundation Benchmark v1.3
Used as the primary control reference for all GCP findings.

| Control Area | Relevant Controls Used |
|---|---|
| Identity and Access Management | 1.2, 1.4, 1.5, 1.6, 1.10, 1.15 |
| Logging and Monitoring | 2.1, 2.2, 2.3, 2.4, 2.5, 2.7 |
| Networking | 3.1, 3.6, 3.8, 3.9, 3.10 |
| Virtual Machines (Compute) | 4.1, 4.2 |
| Storage | 5.1, 5.2, 5.3, 5.4 |

Reference: https://www.cisecurity.org/benchmark/google_cloud_computing_platform

---

## NIST Cybersecurity Framework (CSF)

Findings were categorized using NIST CSF functions:

| CSF Function | Findings Mapped |
|---|---|
| **Identify (ID)** | IAM role auditing, default configuration inventory |
| **Protect (PR)** | Access control, encryption defaults, network segmentation |
| **Detect (DE)** | Logging gaps, monitoring defaults, SIEM integration |
| **Respond (RS)** | Log retention, forensic data availability |
| **Recover (RC)** | Soft delete, bucket versioning, key backup |

Reference: https://www.nist.gov/cyberframework

---

## MITRE ATT&CK

ATT&CK tactics were used to contextualize detection gaps in the default logging configurations.

| Tactic | ID | Applied To |
|---|---|---|
| Initial Access | TA0001 | Default firewall rule exposure, public storage |
| Persistence | TA0003 | No alerting on IAM changes, SA key creation |
| Privilege Escalation | TA0004 | GCP default SA Editor role, Azure broad RBAC |
| Defense Evasion | TA0005 | No diagnostic logging on VMs |
| Discovery | TA0007 | No network flow logs to detect internal scanning |
| Lateral Movement | TA0008 | No east-west traffic visibility |
| Collection | TA0009 | No data access audit logs on storage |
| Exfiltration | TA0010 | No egress monitoring or storage access logging |
| Impact | TA0040 | No alerting on data modification or deletion |

Reference: https://attack.mitre.org/

---

## Regulatory Standards (Informational)

The following standards were considered when framing remediation recommendations. This assessment does not constitute a formal compliance audit.

| Standard | Relevant Areas |
|---|---|
| HIPAA | Log retention (6 years), access controls, audit logging, encryption |
| PCI-DSS | Log retention (1 year), network segmentation, access control |
| FedRAMP | CMEK requirements, MFA, continuous monitoring, log retention |
| GDPR | Data access logging, encryption, access control |
| SOC 2 | Monitoring, logging, access reviews, change management |
