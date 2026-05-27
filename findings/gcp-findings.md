# GCP IaaS Security Findings

**Platform:** Google Cloud Platform  
**Resources Assessed:** Compute Engine (e2-micro), Cloud Storage Bucket, VPC Firewall, IAM, Cloud Audit Logs  
**Configuration Baseline:** Default settings — no post-provisioning hardening applied

---

## GCP-001 — Default Service Accounts Assigned Editor Role

**Severity:** High  
**Category:** Identity and Access Management  
**CIS GCP Benchmark:** Control 1.4 — Ensure that Service Account has no Admin privileges; Control 1.5 — Ensure that Service accounts do not have admin privileges  
**MITRE ATT&CK:** TA0004 – Privilege Escalation, TA0003 – Persistence

**Observation:**  
When a Compute Engine VM is provisioned using default settings, GCP automatically attaches the default Compute Engine service account. This service account is granted the **Editor** role at the project level by default — which allows read and write access to nearly all GCP services within that project.

**Risk:**  
This is one of the highest-impact default misconfigurations in GCP. If an attacker gains code execution on a VM (via vulnerability exploitation, container escape, or compromised application), they can immediately query the instance metadata endpoint (`169.254.169.254/computeMetadata/v1/`) to retrieve the service account token and escalate to project-level Editor access. This provides access to all storage buckets, databases, secrets, and compute resources in the project.

**Proof of concept path:**  
`VM compromise → metadata token retrieval → gcloud auth activate-service-account → project-level access`

**Remediation:**  
1. Remove the default service account's Editor role at the project level
2. Create purpose-specific service accounts with only the permissions required for each workload
3. Disable the default service account where not needed: `gcloud iam service-accounts disable <SA_EMAIL>`
4. If a VM doesn't need a service account, remove it entirely during instance creation
5. Enable Workload Identity Federation for external workloads instead of exported keys
6. Enforce via Org Policy: `constraints/iam.disableServiceAccountCreation` for non-admin users

---

## GCP-002 — VPC Flow Logs Disabled by Default

**Severity:** High  
**Category:** Network Visibility / Logging  
**CIS GCP Benchmark:** Control 3.9 — Ensure VPC Flow logs is enabled for every subnet  
**MITRE ATT&CK:** TA0007 – Discovery, TA0008 – Lateral Movement

**Observation:**  
VPC subnets in GCP do not enable flow logging by default. Flow logs must be explicitly enabled per subnet after creation. Default Compute Engine VM deployments are placed in the default VPC without flow logging.

**Risk:**  
Without VPC flow logs, there is no network-layer visibility into traffic between VMs, to/from the internet, or to Google services. Lateral movement between instances, data exfiltration, and command-and-control beaconing cannot be detected at the network level. This significantly limits both real-time detection and post-incident forensic capability.

**Remediation:**  
1. Enable flow logs per subnet via Console: VPC Networks → Subnets → Edit → Flow Logs: On
2. Via CLI: `gcloud compute networks subnets update <SUBNET> --enable-flow-logs --region <REGION>`
3. Set sampling rate to 1.0 for critical subnets (default 0.5 may miss traffic)
4. Route to Cloud Logging and set retention per compliance requirements
5. Enforce via Org Policy or Security Command Center custom findings

---

## GCP-003 — Data Access Audit Logs Disabled by Default

**Severity:** Medium  
**Category:** Audit Logging  
**CIS GCP Benchmark:** Control 2.1 — Ensure that Cloud Audit Logging is configured properly across all services  
**MITRE ATT&CK:** TA0009 – Collection, TA0040 – Impact

**Observation:**  
GCP Cloud Audit Logs include Admin Activity logs (enabled by default) and Data Access logs (disabled by default). Data Access logs capture read operations on data — including storage bucket reads, database queries, and API data reads — which are the most relevant logs for detecting unauthorized data access or insider threats.

**Risk:**  
Without Data Access logs enabled, an attacker who gains access to a service account with storage read permissions can exfiltrate data from Cloud Storage buckets without generating any audit log entry. There is no record of what data was accessed, by whom, or when.

**Remediation:**  
1. Navigate to IAM & Admin → Audit Logs
2. Enable Data Read and Data Write log types for: Cloud Storage, Compute Engine, Cloud KMS, BigQuery, and Cloud SQL
3. Note: Data Access logs generate significant log volume — configure log exclusions for high-volume low-risk services to manage cost
4. Route to Cloud Logging with appropriate retention; export to Cloud Storage for long-term compliance retention (free)

---

## GCP-004 — Firewall Rule Logging Disabled by Default

**Severity:** Medium  
**Category:** Network Visibility  
**CIS GCP Benchmark:** Control 3.7 — Ensure that RDP access is restricted from the internet; Control 3.10 — Ensure Firewall Rules logging is enabled  
**MITRE ATT&CK:** TA0001 – Initial Access, TA0011 – Command and Control

**Observation:**  
GCP VPC firewall rules do not enable logging by default. The default VPC includes allow rules for SSH (port 22) and RDP (port 3389) from `0.0.0.0/0` — these broad rules exist without any logging of whether or how they are used.

**Risk:**  
Broad ingress rules with no logging mean that brute force attempts, port probes, and unauthorized SSH/RDP access attempts go unrecorded. The absence of deny-log data also makes it impossible to identify reconnaissance activity targeting the environment.

**Remediation:**  
1. Enable logging on all existing firewall rules:  
   `gcloud compute firewall-rules update <RULE_NAME> --enable-logging`
2. Restrict SSH/RDP source ranges immediately — replace `0.0.0.0/0` with specific IP ranges or use Identity-Aware Proxy (IAP) for SSH
3. Use IAP TCP forwarding as a zero-trust alternative to exposing SSH/RDP directly:  
   `gcloud compute ssh <INSTANCE> --tunnel-through-iap`
4. Review and remove the default-allow-rdp and default-allow-ssh rules if not required

---

## GCP-005 — Customer-Managed Encryption Keys Not Configured on Storage Buckets

**Severity:** Low  
**Category:** Encryption / Key Management  
**CIS GCP Benchmark:** Control 5.1 — Ensure that Cloud Storage buckets have uniform bucket-level access enabled; Control 5.2 — Ensure that Cloud Storage buckets are not anonymously or publicly accessible

**Observation:**  
GCP encrypts all Cloud Storage data at rest by default using Google-managed keys (AES-256). However, Customer-Managed Encryption Keys (CMEK) via Cloud KMS are not configured by default. Organizations storing sensitive or regulated data have no control over the key lifecycle, rotation, or revocation.

**Risk:**  
Without CMEK, the organization cannot independently revoke access to stored data if there is a platform-level key compromise or if compliance requirements mandate customer key control. Key rotation is managed by Google on an opaque schedule.

**Remediation:**  
1. Create a Cloud KMS key ring and key in the same region as the storage bucket
2. Grant the GCS service account the Cloud KMS CryptoKey Encrypter/Decrypter role
3. Configure the bucket to use the CMEK key:  
   `gsutil kms encryption -k <KEY_RESOURCE_NAME> gs://<BUCKET_NAME>`
4. Enable automatic key rotation (recommended: 90-day rotation period)

---

## GCP-006 — Default Alert Thresholds Too Broad for Threat Detection

**Severity:** Low  
**Category:** Monitoring / Detection  
**CIS GCP Benchmark:** Control 2.4 — Ensure log metric filters and alerts exist for IAM changes; Control 2.5 — Ensure log metric filters exist for Audit Configuration changes

**Observation:**  
Security Command Center (SCC) in its default configuration surfaces findings at a high level but does not have pre-configured alerting for specific high-signal security events — such as IAM policy changes, service account key creation, or Compute Engine firewall rule modifications.

**Risk:**  
Without targeted alerting, security events are buried in the SCC findings list. An attacker making IAM changes to establish persistence, or creating service account keys for exfiltration, may not be noticed in time for effective response.

**Remediation:**  
1. Create log-based metrics and alert policies for:
   - IAM policy changes: `protoPayload.methodName="SetIamPolicy"`
   - Service account key creation: `protoPayload.methodName="google.iam.admin.v1.CreateServiceAccountKey"`
   - Firewall rule changes: `resource.type="gce_firewall_rule"`
   - Bucket ACL/IAM changes: `resource.type="gcs_bucket" AND protoPayload.methodName=("storage.setIamPermissions" OR "storage.objects.update")`
2. Route to a notification channel (Email, PagerDuty, Slack) via Cloud Monitoring
3. Consider enabling Security Command Center Premium for automated threat detection (Event Threat Detection)
