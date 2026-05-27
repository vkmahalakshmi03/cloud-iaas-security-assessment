# Findings Summary — Azure vs GCP

## Findings by Severity

| Severity | Azure | GCP | Total |
|----------|-------|-----|-------|
| High | 2 | 2 | 4 |
| Medium | 3 | 2 | 5 |
| Low | 1 | 2 | 3 |
| **Total** | **6** | **6** | **12** |

---

## Domain-by-Domain Comparison

### IAM and Access Control

| Finding | Azure | GCP |
|---------|-------|-----|
| Default privilege level | Contributor-scoped defaults; broad at subscription level | Editor role on default service accounts — project-wide |
| Privilege escalation risk | Medium — RBAC misconfigurations | **High** — metadata endpoint + Editor SA = full project access |
| Least privilege enforcement | Available via custom RBAC roles; not default | Requires manual SA creation and role scoping |
| Identity federation | Managed Identities (no key storage) | Workload Identity Federation (no key storage) |
| Recommendation | Audit and downscope all role assignments; enable PIM | Replace default SA with purpose-built SAs; disable Editor binding |

**Verdict:** GCP's default service account misconfiguration carries higher inherent risk due to the straightforward privilege escalation path via the instance metadata endpoint.

---

### Network Visibility

| Finding | Azure | GCP |
|---------|-------|-----|
| Traffic logging default | NSG Flow Logs: **OFF** | VPC Flow Logs: **OFF** |
| Firewall logging default | Diagnostic logs: **OFF** | Firewall rule logging: **OFF** |
| Default ingress rules | NSG required; no implicit allow | Default VPC allows SSH/RDP from 0.0.0.0/0 |
| Lateral movement detection | Blind without flow logs | Blind without flow logs; broader default exposure |
| Recommendation | Enable NSG flow logs; route to Log Analytics | Enable VPC flow logs; restrict/remove default allow rules |

**Verdict:** Both platforms are equally blind at the network layer by default. GCP's default-allow-ssh/rdp rules create additional initial access exposure.

---

### Storage Security

| Finding | Azure | GCP |
|---------|-------|-----|
| Public access default | **Enabled** — requires explicit disabling | **Disabled** — safer default |
| Encryption at rest | AES-256, platform-managed keys | AES-256, Google-managed keys |
| Customer key management | Azure Key Vault (CMK requires configuration) | Cloud KMS / CMEK (requires configuration) |
| Object-level access control | Container-level RBAC + SAS tokens | IAM down to object level — more granular |
| Recommendation | Disable public blob access at account level immediately | Configure CMEK for regulated data; review bucket IAMs |

**Verdict:** GCP has a stronger storage security default (public access off). Azure's default public blob access is a significant misconfiguration for any production environment.

---

### Encryption and Key Management

| Finding | Azure | GCP |
|---------|-------|-----|
| Default encryption | AES-256, PMK | AES-256, Google-managed keys |
| Double/layered encryption | Supported; not default | Not natively supported |
| Customer key support | CMK, CSK, BYOK, HYOK | CMEK, CSEK, BYOK (no HYOK natively) |
| Key visibility and audit | Azure Key Vault + Monitor — strong auditability | Cloud KMS + Audit Logs — requires configuration |
| Recommendation | Enable CMK for regulated workloads; enforce key rotation | Configure CMEK for regulated data; enable audit logs on KMS |

**Verdict:** Azure has a slight edge in key management flexibility (HYOK support, deeper audit integration). Both require active configuration to move beyond platform-managed keys.

---

### Logging and Monitoring

| Finding | Azure | GCP |
|---------|-------|-----|
| Activity logging | Activity Log: ON by default | Admin Activity Audit Logs: ON by default |
| Data access logging | Diagnostic Logs: **OFF** by default | Data Access Audit Logs: **OFF** by default |
| Native SIEM | Microsoft Sentinel — native, deep integration | Chronicle — third-party, additional setup required |
| Threat detection | Defender for Cloud (free tier available) | Security Command Center (free tier limited) |
| Log retention (default) | 90 days (some services); 30 days for others | 30 days default; free long-term via Cloud Storage |
| Alert defaults | Broad; requires custom alert rules | Broad; requires log-based metric filters |
| Recommendation | Enable Diagnostic Settings across all resources; configure Sentinel alerts | Enable Data Access logs; create log-based metric filters for critical events |

**Verdict:** Azure has an advantage with Sentinel as a native SIEM. GCP's free long-term log retention via Cloud Storage is a cost advantage for compliance. Both require significant manual configuration to achieve meaningful detection coverage.

---

## Overall Risk Assessment

| Domain | Higher Default Risk |
|--------|-------------------|
| IAM / Privilege Escalation | GCP |
| Network Exposure | GCP (default allow rules) |
| Storage Public Access | Azure |
| Logging Coverage | Tied — both require manual enablement |
| Encryption Defaults | Tied — both AES-256 with Google/MS managed keys |

Neither platform is secure out of the box. Both require deliberate post-provisioning hardening before being used for any production or sensitive workload.

---

## MITRE ATT&CK Coverage Gaps (Default State)

| Tactic | Detection Gap (Default) |
|--------|------------------------|
| TA0001 – Initial Access | No firewall rule logging to detect brute force or unauthorized access |
| TA0003 – Persistence | No alerting on IAM changes or service account key creation |
| TA0004 – Privilege Escalation | GCP metadata endpoint exposure with Editor SA; Azure broad RBAC |
| TA0007 – Discovery | No flow logs to detect internal reconnaissance |
| TA0008 – Lateral Movement | No network visibility to detect east-west traffic |
| TA0009 – Collection | No data access logging on storage reads |
| TA0010 – Exfiltration | No egress traffic logging or anomaly alerting |
