# GCP Default Configuration Deficiencies

Six security gaps identified in GCP's default IaaS configuration. Each finding includes the default state, what it exposes, the associated risk, and framework mapping.

---

## GCP-001 — Default Service Account Assigned Editor Role

**Severity:** High  
**Domain:** IAM  
**MITRE ATT&CK:** Privilege Escalation — T1078.004 (Valid Accounts: Cloud Accounts)  
**CIS GCP v1.3:** 1.5  
**NIST CSF:** PR.AC-4 (Protect — Access Control)

**Default state:** Every GCP project has a default Compute Engine service account. This account is automatically assigned the Editor role at the project level and is attached to every VM created with default settings.

**What this means:** The Editor role grants read/write access to most GCP resources within the project — including Cloud Storage, Compute Engine, BigQuery, and Cloud Functions. Any VM using the default service account can access all of these resources by retrieving the service account's OAuth token from the instance metadata endpoint.

**Attack path:**
1. Attacker gains code execution on a GCP VM (via web app vulnerability, SSRF, or compromised container)
2. Queries the metadata endpoint: `curl "http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token" -H "Metadata-Flavor: Google"`
3. Retrieves an OAuth2 access token with Editor-level permissions
4. Uses the token to access Cloud Storage buckets, enumerate resources, read secrets, or create backdoor service account keys

No credentials are required. No privilege escalation tool is needed. The metadata endpoint is accessible from every VM by default. This is the highest-risk finding in the assessment.

**Remediation:** Create custom service accounts with minimum required permissions for each workload. Remove the default Compute Engine service account's Editor role binding at the project level. Apply the Organization Policy constraint `iam.automaticIamGrantsForDefaultServiceAccounts` to prevent automatic role grants.

---

## GCP-002 — VPC Flow Logs Disabled by Default

**Severity:** High  
**Domain:** Logging and Monitoring  
**MITRE ATT&CK:** Defense Evasion — T1562.008 (Disable or Modify Cloud Logs)  
**CIS GCP v1.3:** 3.8  
**NIST CSF:** DE.CM-1 (Detect — Continuous Monitoring)

**Default state:** VPC Flow Logs are not enabled on any subnet when the default VPC is created.

**What this means:** No network traffic metadata is recorded for any subnet in the default VPC. Connections between VMs, egress traffic to the internet, and traffic to Google APIs generate no log data. Lateral movement, data exfiltration, and C2 communication are undetectable at the network level.

**Attack path:** Identical in impact to Azure's NSG Flow Log gap (AZ-001). An attacker moving between resources in the VPC generates no network log trail. Detection depends entirely on application-level or endpoint-level logs, which are also limited in the default configuration.

**Remediation:** Enable VPC Flow Logs on all subnets. Set aggregation interval to 5 seconds for near-real-time visibility. Route flow logs to Cloud Logging and configure log-based alerting for anomalous traffic patterns. Export to Cloud Storage for long-term retention beyond 30 days.

---

## GCP-003 — Data Access Audit Logs Disabled by Default

**Severity:** Medium  
**Domain:** Logging and Monitoring  
**MITRE ATT&CK:** Defense Evasion — T1562.008 (Disable or Modify Cloud Logs)  
**CIS GCP v1.3:** 2.1  
**NIST CSF:** DE.CM-7 (Detect — Monitoring for Unauthorized Activity)

**Default state:** Data Access Audit Logs (Admin Read, Data Read, Data Write) are not enabled by default for most GCP services.

**What this means:** API calls that read or modify data — including Cloud Storage object reads, BigQuery queries, and IAM policy lookups — are not recorded. Admin Activity logs are enabled by default and capture resource creation/deletion, but the data plane activity that matters most for detecting data exfiltration goes unlogged.

**Attack path:** An attacker who has gained access (via GCP-001 or other means) can read storage bucket contents, query databases, and download sensitive data. Without Data Access Audit Logs, there is no record of what was accessed. Post-incident forensics cannot determine the scope of the data exposure.

**Remediation:** Enable Data Access Audit Logs at the organization or project level for all services. Configure log sinks to export to Cloud Storage for long-term retention. Be aware that enabling data access logs significantly increases log volume and may affect costs — scope to critical services first if cost is a constraint.

---

## GCP-004 — Firewall Rule Logging Disabled by Default

**Severity:** Medium  
**Domain:** Logging and Monitoring  
**MITRE ATT&CK:** Defense Evasion — T1562.008 (Disable or Modify Cloud Logs)  
**CIS GCP v1.3:** 3.7  
**NIST CSF:** DE.CM-1 (Detect — Continuous Monitoring)

**Default state:** Firewall rule logging is disabled on all default VPC firewall rules, including the default-allow-ssh, default-allow-rdp, and default-allow-icmp rules.

**What this means:** Firewall rules are enforced but generate no log data. There is no record of which connections were allowed or denied by firewall rules. This is particularly significant because GCP's default firewall rules allow SSH and RDP from 0.0.0.0/0 — meaning the most permissive rules in the environment are also the ones with no logging.

**Attack path:** An attacker scanning for open SSH or RDP ports across GCP IP ranges will find instances with default firewall rules. Successful and failed connection attempts generate no log entry. Brute-force attacks against SSH leave no trace in the firewall logs. Detection depends on OS-level authentication logs, which require a separate logging agent to forward.

**Remediation:** Enable firewall rule logging on all rules, including default rules. For high-volume rules, consider enabling metadata-only logging to reduce cost while maintaining visibility. Create custom firewall rules to replace default rules with explicit source restrictions and logging enabled.

---

## GCP-005 — CMEK Not Configured on Storage Buckets

**Severity:** Low  
**Domain:** Encryption  
**MITRE ATT&CK:** Collection — T1530 (Data from Cloud Storage)  
**CIS GCP v1.3:** 5.2  
**NIST CSF:** PR.DS-1 (Protect — Data Security)

**Default state:** Cloud Storage buckets are encrypted with Google-managed keys by default. Customer-Managed Encryption Keys (CMEK) are not configured.

**What this means:** Encryption is active, but key lifecycle — rotation, access control, revocation — is entirely managed by Google. The organization has no visibility into key usage and no ability to revoke access by destroying the encryption key. For compliance frameworks that require customer control over encryption keys (PCI-DSS, HIPAA, FedRAMP), Google-managed keys may not satisfy the requirement.

**Attack path:** The direct exploitation risk is low because data is encrypted regardless. The risk is compliance-related: an organization that needs to demonstrate key control for regulatory purposes cannot do so with Google-managed keys. In a forensic scenario, the inability to independently audit key access is a gap.

**Remediation:** Configure CMEK via Cloud KMS for all storage buckets handling sensitive or regulated data. Set up automatic key rotation (90-day interval recommended). Restrict Cloud KMS key access to specific service accounts using IAM conditions.

---

## GCP-006 — Default Alert Thresholds Too Broad for Threat Detection

**Severity:** Low  
**Domain:** Logging and Monitoring  
**MITRE ATT&CK:** Defense Evasion — T1562.006 (Indicator Removal)  
**CIS GCP v1.3:** 2.4  
**NIST CSF:** DE.AE-3 (Detect — Analysis and Escalation)

**Default state:** Security Command Center (SCC) free tier provides findings for common misconfigurations, but alert thresholds and detection rules are not calibrated for actionable threat detection.

**What this means:** SCC flags broad categories of issues (public buckets, open firewall rules, default service accounts) but does not provide behavioral detection — no alerting on anomalous API calls, suspicious login patterns, or unusual data access volumes. The default configuration generates findings that are useful for posture management but insufficient for operational threat detection.

**Attack path:** An attacker operating within the environment triggers no behavioral alerts from SCC. Activities like bulk data download, service account key creation, or IAM policy modification may not generate a finding unless they create a specific misconfiguration that SCC is scanning for. This is a detection coverage gap rather than a direct vulnerability.

**Remediation:** Upgrade to SCC Premium for behavioral threat detection, or integrate with Chronicle or a third-party SIEM. Configure custom alert policies in Cloud Monitoring for critical events: IAM policy changes, service account key creation, firewall rule modifications, and bulk storage operations. Tune alert thresholds to the organization's baseline to reduce false positives and surface actionable alerts.
