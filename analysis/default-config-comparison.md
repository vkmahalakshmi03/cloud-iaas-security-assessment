# Default Configuration Comparison: Azure vs GCP

> **Assessment Date:** April 2025  
> **Version:** 1.0  
> **Frameworks:** CIS Azure v2.0, CIS GCP v1.3, NIST CSF, MITRE ATT&CK

Side-by-side evaluation of what each platform ships by default across five security domains. No hardening applied — these are the configurations that exist the moment provisioning completes.

---

## IAM and Access Controls

| Attribute | Azure | GCP |
|-----------|-------|-----|
| Identity platform | Microsoft Entra ID (Azure AD) | Google Cloud IAM |
| Default access model | RBAC at subscription/resource group level | IAM role bindings from org to object level |
| Default VM identity | System-assigned Managed Identity (not enabled by default) | Default Compute Engine service account (Editor role) |
| Granularity | Container-level on storage; resource group on compute | Object-level on storage; instance-level on compute |
| Service authentication | Managed Identities (no stored secrets) | Service account keys or Workload Identity Federation |
| MFA enforcement | Not enforced by default | Not enforced by default |

**Key difference:** Azure's default RBAC assignments are broad at the subscription level but don't automatically attach a high-privilege identity to every VM. GCP does — the default Compute Engine service account with Editor role is assigned to every VM unless explicitly changed. This means any compromised GCP VM can query the metadata endpoint and retrieve a token with project-wide read/write access.

Azure's risk is different: default role assignments at the subscription level can grant Contributor or Owner access more broadly than intended, but this requires an identity to already be over-provisioned rather than being automatic.

---

## Network Security

| Attribute | Azure | GCP |
|-----------|-------|-----|
| Default firewall | NSG auto-created with VM | Default VPC firewall rules |
| Implicit allow rules | No implicit inbound allow; NSG required | default-allow-ssh and default-allow-rdp open to 0.0.0.0/0 |
| Network flow logging | NSG Flow Logs — OFF by default | VPC Flow Logs — OFF by default |
| Firewall logging | Diagnostic logs — OFF by default | Firewall rule logging — OFF by default |
| Network segmentation | NSG per subnet/NIC; VNet isolation | VPC firewall rules; VPC isolation |
| DDoS protection | Basic tier included; Standard requires opt-in | Cloud Armor available; not applied by default |

**Key difference:** GCP's default VPC includes firewall rules that allow SSH (port 22) and RDP (port 3389) from any source IP (0.0.0.0/0). Azure's NSG does not include an implicit allow — inbound traffic requires explicit rules. However, both platforms share the same critical gap: neither enables network flow logging by default. This means traffic analysis, lateral movement detection, and forensic reconstruction are impossible until flow logs are manually activated.

---

## Storage Security

| Attribute | Azure | GCP |
|-----------|-------|-----|
| Public access default | Enabled — public blob access ON | Disabled — public access blocked by default |
| Access control model | RBAC at container level; SAS tokens for object-level | IAM bindings at bucket and object level natively |
| Temporary access | Shared Access Signatures (time-limited, permission-scoped) | Signed URLs (time-limited, permission-scoped) |
| Encryption at rest | AES-256 with platform-managed keys | AES-256 with Google-managed keys |
| In-transit encryption | HTTPS enforced by default | TLS 1.2+ enforced by default |
| Data access logging | OFF by default | OFF by default |

**Key difference:** Azure storage accounts are created with public blob access enabled. Unless explicitly disabled, containers can be made publicly accessible — and there's no audit log to show if someone did. GCP blocks public access by default and supports an Organization Policy to prevent it from being re-enabled.

On access control granularity, GCP's native object-level IAM means fine-grained permissions don't require a separate token system. Azure relies on SAS tokens for object-level access, which introduces additional complexity around token generation, expiry management, and revocation.

Both platforms share the same blind spot: data access audit logs are disabled by default. Storage reads and writes on both platforms leave no trace until logging is explicitly enabled.

---

## Encryption and Key Management

| Attribute | Azure | GCP |
|-----------|-------|-----|
| Default encryption | AES-256, platform-managed keys (PMK) | AES-256, Google-managed keys |
| Customer key options | CMK, CSK, BYOK, HYOK | CMEK, CSEK, BYOK |
| Double encryption | Available (CMK + PMK layered) — not enabled by default | Not available as a native layered option |
| Key management service | Azure Key Vault | Cloud KMS |
| HSM support | Dedicated HSM (FIPS 140-2 compliant) | Cloud HSM (FIPS 140-2 Level 3) |
| Key rotation | Manual or policy-based via Key Vault | Manual or automatic via Cloud KMS |
| Key lifecycle visibility | Integrated with Azure Monitor, Sentinel, and Azure Policy | Audit logging available; requires explicit configuration |

**Key difference:** Azure offers a broader range of key management models, including double encryption and Hold Your Own Key (HYOK) — neither of which GCP supports natively. However, none of these are enabled by default on either platform. Both ship with provider-managed keys only.

Azure Key Vault integrates more deeply with the monitoring and policy ecosystem out of the box, giving better visibility into key access patterns. GCP's Cloud KMS audit logging requires manual configuration for granular tracking.

For organizations with strict compliance requirements around key control, both platforms require manual configuration to move beyond provider-managed encryption.

---

## Logging and Monitoring

| Attribute | Azure | GCP |
|-----------|-------|-----|
| Monitoring service | Azure Monitor | Cloud Monitoring |
| Logging service | Azure Activity Logs, Diagnostic Logs | Cloud Logging, Cloud Audit Logs |
| Threat detection | Microsoft Defender for Cloud | Security Command Center (SCC) |
| Native SIEM | Microsoft Sentinel (built-in) | Chronicle (third-party, requires setup) |
| NSG/VPC Flow Logs | OFF by default | OFF by default |
| Firewall logging | OFF by default | OFF by default |
| Data access audit logs | OFF by default | OFF by default |
| VM diagnostic logs | OFF by default | Basic logging enabled; detailed metrics require agent |
| Default log retention | 30 days (Activity Logs); up to 90 days (Log Analytics) | 30 days; free long-term storage via Cloud Storage routing |
| Alert defaults | Basic alerts; thresholds require customization | Basic alerts; thresholds too broad for actionable detection |

**Key difference:** Azure has a significant advantage in SIEM integration — Microsoft Sentinel is a native, first-party SIEM that integrates directly with the Azure ecosystem. GCP relies on Chronicle, which is a separate product requiring additional setup and configuration. For organizations that need immediate detection capability, Azure's Sentinel integration provides a faster path to operational monitoring.

On retention, GCP has a cost advantage: logs can be routed to Cloud Storage for free long-term retention beyond the 30-day default. Azure routes logs to a Log Analytics Workspace, which charges per GB ingested — making long-term retention more expensive.

The critical shared gap: both platforms disable the logging categories that matter most for detection — network flow logs, firewall logs, and data access logs. A SOC team inheriting either environment has no telemetry to ingest into a SIEM until these are manually enabled.

---

## Summary

| Domain | Azure Default Risk | GCP Default Risk |
|--------|-------------------|-----------------|
| IAM | Broad RBAC at subscription level | Editor SA on every VM by default |
| Network | No implicit allow, but no flow logs | SSH/RDP open to 0.0.0.0/0, no flow logs |
| Storage | Public blob access enabled | Public access disabled (advantage) |
| Encryption | PMK only; double encryption available but off | Google-managed keys only; no double encryption option |
| Logging | Sentinel available but logs disabled | Chronicle requires setup; logs disabled |

Neither platform provides a production-ready security posture by default. The defaults on both sides prioritize operational simplicity — getting resources running — over security. The specific risks differ, but the pattern is the same: critical logging is off, IAM is over-permissioned, and encryption key control is left to the provider.

---

## Assessment Methodology

Each platform was provisioned through its native console (Azure Portal, GCP Cloud Console) using default wizard settings. No post-deployment configuration changes were made before the assessment began. The resulting environments were inspected domain by domain using platform-native tools: Azure Cloud Shell and Microsoft Defender for Cloud (free tier) on Azure; GCP Cloud Shell and Security Command Center (free tier) on GCP.

**Finding prioritization** was based on three factors:

- **Exploitability** — Can this gap be exploited with minimal skill and no prerequisites, or does it require prior access and specific conditions?
- **Blast radius** — Does exploitation affect a single resource, a resource group, or the entire project/subscription?
- **Detection blind spot** — Does this gap prevent the organization from detecting that exploitation has occurred?

Findings that scored high on all three (GCP-001, AZ-001, GCP-002, AZ-002) were rated High severity. Findings that require some preconditions or have a narrower blast radius were rated Medium. Findings with low direct exploitation risk or primarily compliance implications were rated Low.

**Frameworks applied:**

| Framework | How It Was Used |
|-----------|----------------|
| CIS Benchmark for Azure v2.0 | Control-level mapping for each Azure finding and hardening step |
| CIS Benchmark for GCP v1.3 | Control-level mapping for each GCP finding and hardening step |
| NIST CSF | Risk categorization across Identify, Protect, Detect, Respond, Recover |
| MITRE ATT&CK (Cloud Matrix) | Tactic and technique mapping to model attacker behavior against each gap |

## Scope Limitations

- Assessment reflects configurations at time of testing (April 2025). Cloud providers update defaults — findings should be revalidated periodically
- Free-tier environments were used. Enterprise provisioning workflows (landing zones, organization policies, management groups) may ship different defaults
- Assessment focused exclusively on default IaaS configurations — PaaS, SaaS, and managed service defaults were out of scope
- No automated scanning tools (ScoutSuite, Prowler, etc.) were used. Findings are based on manual configuration review against CIS Benchmark controls and platform documentation
- Production workload-specific risks (data classification, regulatory requirements, multi-region deployments) were not modeled
- Testing was limited to the default configurations of compute, storage, networking, IAM, and monitoring. Other services (databases, serverless, container platforms) were not assessed
