# Assessment Methodology

## Objective

Evaluate what security posture an organization inherits when provisioning IaaS resources in Azure and GCP using default configurations — without applying any post-deployment hardening. The goal was to identify gaps that exist before any hardening is applied, and map those gaps to real attack paths.

---

## Environment Setup

Both environments were provisioned using free-tier accounts with default settings. No post-provisioning changes were made before assessment.

### Azure Environment
- **VM:** B1s instance, Windows Server 2022 image, default NSG, default authentication settings
- **Storage:** General Purpose v2 storage account, default access tier, no custom policies applied
- **IAM:** Default subscription-level role assignments reviewed
- **Region:** East US

### GCP Environment
- **VM:** e2-micro, Debian 11 image, default VPC (default network), default firewall rules applied
- **Storage:** Regional Cloud Storage bucket, default access settings
- **IAM:** Default project-level IAM reviewed, default Compute Engine service account examined
- **Region:** us-central1

---

## Assessment Domains

Each domain was evaluated independently and then compared across platforms.

### 1. IAM and Access Control
- Reviewed default role assignments on provisioned VMs and storage
- Examined default service account permissions (GCP) and RBAC assignments (Azure)
- Identified privilege escalation paths available in default state
- Evaluated identity federation options and whether least-privilege was the default

### 2. Network Security
- Reviewed default NSG rules (Azure) and VPC firewall rules (GCP)
- Checked whether flow logging was enabled by default
- Assessed default inbound/outbound rule configurations
- Examined network segmentation defaults

### 3. Storage Security
- Reviewed default access control settings on storage accounts / buckets
- Verified encryption defaults (at rest and in transit)
- Checked public access settings
- Evaluated access granularity (container-level vs object-level)

### 4. Encryption and Key Management
- Reviewed encryption algorithm and key type used by default
- Evaluated customer-managed key (CMK/CMEK) configuration paths
- Compared key lifecycle management options
- Assessed HSM support and compliance-relevant encryption features

### 5. Logging and Monitoring
- Verified which log types were enabled by default
- Checked default retention periods
- Reviewed native security tooling (Defender for Cloud, Security Command Center)
- Assessed SIEM integration options and default alert coverage

---

## Tooling

| Tool | Purpose |
|------|---------|
| Azure Portal | Resource provisioning, IAM review, NSG inspection |
| GCP Cloud Console | Resource provisioning, IAM review, firewall review |
| Microsoft Defender for Cloud | Default security posture score and recommendations review |
| Google Security Command Center | Default findings review |
| Cloud Shell (Azure & GCP) | CLI-based config inspection |
| CIS Benchmark for Azure v2.0 | Control baseline for Azure findings |
| CIS Benchmark for GCP v1.3 | Control baseline for GCP findings |

---

## Evaluation Criteria

Each finding was assessed on:

- **Severity:** Based on exploitability, impact, and likelihood in a realistic production scenario
- **Default state:** Is this a misconfiguration or simply a missing hardening step?
- **Attack path:** Is there a realistic path from this gap to a meaningful impact?
- **Remediation complexity:** Can this be fixed in minutes (configuration toggle) or does it require architectural changes?
- **Framework alignment:** Which CIS Benchmark control and NIST CSF category applies?

---

## Limitations

- Assessment reflects configurations at time of testing; cloud providers update defaults over time
- Free-tier environments may differ slightly from enterprise provisioning workflows
- Assessment focused on default IaaS configurations — PaaS and SaaS defaults were out of scope
- No automated scanning tools were used; findings are based on manual configuration review and documentation analysis
- Production workload-specific risks (data classification, regulatory requirements) were not modeled

---

## Framework Mapping

| Framework | Application |
|-----------|------------|
| CIS Benchmark for Azure v2.0 | Primary control reference for Azure findings |
| CIS Benchmark for GCP v1.3 | Primary control reference for GCP findings |
| NIST CSF | Risk categorization (Identify, Protect, Detect, Respond, Recover) |
| MITRE ATT&CK | Tactic mapping for detection gap analysis |
