# GCP IaaS Hardening Checklist

> **Assessment Date:** April 2025  
> **Version:** 1.0  
> **Benchmark:** CIS GCP Benchmark v1.3  
> **Controls:** 30 across IAM, Network, Storage, Encryption, Logging

Mapped to CIS GCP Benchmark v1.3. Apply these controls immediately after provisioning any IaaS environment.

---

## Prerequisites

- **Required role:** Project Owner or IAM Admin for IAM changes; Editor for resource configuration changes; Organization Admin for Org Policy constraints
- **Tools:** Google Cloud CLI (`gcloud`) installed and authenticated, or Cloud Shell access
- **Dependencies:** Cloud KMS keyring must be created before configuring CMEK. Log sink destination (Cloud Storage bucket) must exist before configuring log exports.

---

## IAM and Access Control

- [ ] **GCP-IAM-01** — Remove the Editor role from the default Compute Engine service account  
  `gcloud projects remove-iam-policy-binding <PROJECT> --member="serviceAccount:<PROJECT_NUMBER>-compute@developer.gserviceaccount.com" --role="roles/editor"`  
  *CIS: 1.5 | NIST: PR.AC-4*

- [ ] **GCP-IAM-02** — Create custom service accounts with minimum required permissions for each workload  
  *CIS: 1.5 | NIST: PR.AC-4*

- [ ] **GCP-IAM-03** — Apply Org Policy `iam.automaticIamGrantsForDefaultServiceAccounts` to prevent automatic Editor grants  
  *CIS: 1.5 | NIST: PR.AC-4*

- [ ] **GCP-IAM-04** — Enforce MFA for all user accounts with project-level access  
  *CIS: 1.1 | NIST: PR.AC-1*

- [ ] **GCP-IAM-05** — Audit all project-level IAM bindings; remove any Editor/Owner bindings not explicitly justified  
  *CIS: 1.4 | NIST: PR.AC-4*

- [ ] **GCP-IAM-06** — Use Workload Identity Federation for external service authentication; eliminate long-lived service account keys  
  *CIS: 1.7 | NIST: PR.AC-1*

**Verify:** `gcloud projects get-iam-policy <PROJECT> --format=json | jq '.bindings[] | select(.role=="roles/editor")'` — confirm the default Compute Engine SA no longer holds Editor.

---

## Network Security

- [ ] **GCP-NET-01** — Enable VPC Flow Logs on all subnets; set aggregation interval to 5 seconds  
  *CIS: 3.8 | NIST: DE.CM-1*

- [ ] **GCP-NET-02** — Enable firewall rule logging on all firewall rules (including default rules)  
  *CIS: 3.7 | NIST: DE.CM-1*

- [ ] **GCP-NET-03** — Replace default-allow-ssh and default-allow-rdp rules with restricted source ranges; use Identity-Aware Proxy (IAP) for SSH access  
  `gcloud compute ssh <INSTANCE> --tunnel-through-iap`  
  *CIS: 3.6 | NIST: PR.AC-3*

- [ ] **GCP-NET-04** — Delete the default VPC network and create custom VPCs with explicit firewall rules  
  *CIS: 3.1 | NIST: PR.AC-5*

- [ ] **GCP-NET-05** — Enable Private Google Access for subnets to allow VM access to Google APIs without public IPs  
  *CIS: 3.8 | NIST: PR.AC-5*

- [ ] **GCP-NET-06** — Enforce Org Policy to prevent external IP assignment on VMs where not required  
  `constraints/compute.vmExternalIpAccess`  
  *CIS: 3.9 | NIST: PR.AC-5*

**Verify:** `gcloud compute networks subnets list --format="table(name,region,enableFlowLogs)"` — confirm flow logs are enabled on all subnets.

---

## Storage Security

- [ ] **GCP-STG-01** — Enforce uniform bucket-level access (disables ACLs, enforces IAM-only control)  
  `gsutil uniformbucketlevelaccess set on gs://<BUCKET>`  
  *CIS: 5.2 | NIST: PR.AC-4*

- [ ] **GCP-STG-02** — Verify public access is disabled; apply Org Policy to prevent public bucket creation  
  `constraints/storage.publicAccessPrevention`  
  *CIS: 5.1 | NIST: PR.DS-1*

- [ ] **GCP-STG-03** — Configure CMEK for Cloud Storage buckets handling sensitive data  
  *CIS: 5.2 | NIST: PR.DS-1*

- [ ] **GCP-STG-04** — Enable bucket lock and retention policies on compliance-relevant buckets  
  *CIS: 5.3 | NIST: PR.DS-1*

- [ ] **GCP-STG-05** — Enable versioning on critical storage buckets to support recovery and forensics  
  *CIS: 5.4 | NIST: PR.DS-1*

**Verify:** `gsutil publicAccessPrevention get gs://<BUCKET>` — confirm public access prevention is enforced. `gsutil uniformbucketlevelaccess get gs://<BUCKET>` — confirm uniform access is enabled.

---

## Encryption and Key Management

- [ ] **GCP-ENC-01** — Configure CMEK for Cloud Storage, BigQuery, and Cloud SQL handling regulated data  
  *CIS: 5.2 | NIST: PR.DS-1*

- [ ] **GCP-ENC-02** — Enable automatic key rotation in Cloud KMS (recommended: 90-day rotation)  
  *CIS: 1.10 | NIST: PR.DS-1*

- [ ] **GCP-ENC-03** — Restrict Cloud KMS key access to specific service accounts using IAM conditions  
  *CIS: 1.11 | NIST: PR.AC-4*

- [ ] **GCP-ENC-04** — Use Cloud HSM for key storage when FIPS 140-2 Level 3 compliance is required  
  *CIS: 1.10 | NIST: PR.DS-1*

- [ ] **GCP-ENC-05** — Enable key destruction protection; configure scheduled destruction delay (minimum 24 hours)  
  *CIS: 1.10 | NIST: PR.DS-1*

**Verify:** `gcloud kms keys list --location=<LOCATION> --keyring=<KEYRING> --format="table(name,rotationPeriod,nextRotationTime)"` — confirm rotation is configured and scheduled.

---

## Logging and Monitoring

- [ ] **GCP-LOG-01** — Enable Data Access Audit Logs for all services at the organization or project level  
  *CIS: 2.1 | NIST: DE.CM-7*

- [ ] **GCP-LOG-02** — Configure log sinks to export logs to Cloud Storage for long-term retention  
  *CIS: 2.2 | NIST: RS.AN-1*

- [ ] **GCP-LOG-03** — Enable Security Command Center Premium for behavioral threat detection  
  *CIS: 2.4 | NIST: DE.CM-4*

- [ ] **GCP-LOG-04** — Configure custom alert policies in Cloud Monitoring for: IAM changes, service account key creation, firewall modifications, bulk storage operations  
  *CIS: 2.4 | NIST: DE.AE-3*

- [ ] **GCP-LOG-05** — Set up log-based metrics and alerting for failed authentication attempts and privilege escalation events  
  *CIS: 2.4 | NIST: DE.AE-3*

- [ ] **GCP-LOG-06** — Set log retention to minimum 1 year via Cloud Storage routing (regulatory requirement for HIPAA/FedRAMP)  
  *CIS: 2.2 | NIST: RS.AN-1*

**Verify:** `gcloud logging sinks list --format="table(name,destination,filter)"` — confirm log sinks exist and are routing to the correct Cloud Storage bucket.
