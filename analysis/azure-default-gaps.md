# Azure Default Configuration Deficiencies

Six security gaps identified in Azure's default IaaS configuration. Each finding includes the default state, what it exposes, the associated risk, and framework mapping.

---

## AZ-001 — NSG Flow Logs Disabled by Default

**Severity:** High  
**Domain:** Logging and Monitoring  
**MITRE ATT&CK:** Defense Evasion — T1562.008 (Disable or Modify Cloud Logs)  
**CIS Azure v2.0:** 6.4  
**NIST CSF:** DE.CM-1 (Detect — Continuous Monitoring)

**Default state:** Network Security Group Flow Logs are not enabled when an NSG is created. No network traffic metadata is recorded.

**What this means:** All inbound and outbound traffic through NSGs generates no log data. Lateral movement between VMs, outbound connections to command-and-control infrastructure, and data exfiltration over the network produce no audit trail. A SOC team monitoring this environment has zero network telemetry to ingest into a SIEM.

**Attack path:** An attacker who gains access to one VM can move laterally to other resources in the VNet. Without flow logs, there is no record of the connection being made — no source/destination IP, no port data, no timestamp. Detection depends entirely on endpoint-level telemetry, which is also limited by default (see AZ-003).

**Remediation:** Enable NSG Flow Logs for all NSGs and route to a Log Analytics Workspace. Set retention to a minimum of 90 days. Configure Traffic Analytics for automated anomaly detection.

---

## AZ-002 — Public Blob Access Enabled on Storage Accounts

**Severity:** High  
**Domain:** Storage Security  
**MITRE ATT&CK:** Initial Access — T1530 (Data from Cloud Storage)  
**CIS Azure v2.0:** 3.5  
**NIST CSF:** PR.DS-1 (Protect — Data Security)

**Default state:** When a General Purpose v2 storage account is created, the "Allow Blob public access" setting is enabled. Individual containers can be set to public access without any additional approval or override.

**What this means:** Any container within the storage account can be configured for anonymous public read access. If a container is set to "Blob" or "Container" access level, its contents are accessible to anyone on the internet without authentication. Combined with the fact that data access audit logs are off by default, there is no record of who accessed the data or when.

**Attack path:** An attacker who discovers a publicly accessible container (via enumeration or URL guessing) can download all contents without authentication. Automated tools exist for this. Because data access logging is disabled, the organization has no way to determine what was accessed or how long the exposure lasted.

**Remediation:** Disable public blob access at the storage account level immediately after provisioning. Use `az storage account update --name <NAME> --allow-blob-public-access false`. Apply Azure Policy to prevent re-enablement across the subscription.

---

## AZ-003 — VM Diagnostic Logs Not Enabled by Default

**Severity:** Medium  
**Domain:** Logging and Monitoring  
**MITRE ATT&CK:** Defense Evasion — T1562.008 (Disable or Modify Cloud Logs)  
**CIS Azure v2.0:** 5.1  
**NIST CSF:** DE.CM-1 (Detect — Continuous Monitoring)

**Default state:** VM boot diagnostics and performance metrics collection are not enabled when a VM is provisioned through the Azure Portal with default settings.

**What this means:** No OS-level telemetry is collected. Boot failures, performance anomalies, and system-level events are not recorded. If a VM is compromised, there is no historical data to support incident investigation at the host level.

**Attack path:** An attacker operating on a compromised VM generates no host-level log data. Activities like process creation, service installation, scheduled task creation, and local account manipulation leave no trace in Azure's monitoring layer unless diagnostics are enabled and connected to a Log Analytics Workspace.

**Remediation:** Enable boot diagnostics and guest OS diagnostics on all VMs. Install the Azure Monitor Agent and configure data collection rules to forward OS logs to a centralized Log Analytics Workspace.

---

## AZ-004 — Double Encryption Not Enforced by Default

**Severity:** Medium  
**Domain:** Encryption  
**MITRE ATT&CK:** Collection — T1530 (Data from Cloud Storage)  
**CIS Azure v2.0:** 3.2  
**NIST CSF:** PR.DS-1 (Protect — Data Security)

**Default state:** Data at rest is encrypted using AES-256 with platform-managed keys (PMK) only. The double encryption option (CMK layered over PMK) is available but not enabled.

**What this means:** Encryption relies entirely on keys managed by Microsoft. The organization has no control over key lifecycle, rotation schedule, or access policies. For regulated data (HIPAA, PCI-DSS, FedRAMP), platform-managed keys alone may not satisfy compliance requirements for customer key control.

**Attack path:** If an attacker gains access to the Azure management plane, platform-managed encryption provides no additional barrier — the decryption keys are managed by the same platform the attacker has compromised. Customer-managed keys stored in Key Vault with separate access controls create an independent layer of protection.

**Remediation:** Configure Customer-Managed Keys via Azure Key Vault for all storage accounts and managed disks handling sensitive or regulated data. Enable double encryption for defense-in-depth. Set up automated key rotation policies (90-day maximum recommended).

---

## AZ-005 — Over-Permissioned Default RBAC Assignments

**Severity:** Medium  
**Domain:** IAM  
**MITRE ATT&CK:** Privilege Escalation — T1078.004 (Valid Accounts: Cloud Accounts)  
**CIS Azure v2.0:** 1.21  
**NIST CSF:** PR.AC-4 (Protect — Access Control)

**Default state:** Default RBAC role assignments at the subscription level often include Contributor or Owner roles assigned more broadly than required. Role assignments are inherited by all resource groups and resources within the subscription.

**What this means:** Users or service principals with subscription-level Contributor access can create, modify, and delete resources across the entire subscription. This violates least-privilege principles and increases the blast radius if any identity is compromised. Inheritance means a single over-permissioned assignment propagates to every resource group and resource below it.

**Attack path:** An attacker who compromises an identity with Contributor access at the subscription level can create new VMs (for crypto mining or lateral movement), modify NSGs (to open ports), access storage accounts, and delete resources. The breadth of access is determined by the scope of the role assignment, not the attacker's skill.

**Remediation:** Audit all subscription-level role assignments. Remove any Contributor or Owner assignments not explicitly justified. Replace with custom roles scoped to specific resource groups. Enable Privileged Identity Management (PIM) for just-in-time elevation.

---

## AZ-006 — Firewall Diagnostic Logs Require Manual Activation

**Severity:** Low  
**Domain:** Logging and Monitoring  
**MITRE ATT&CK:** Defense Evasion — T1562.008 (Disable or Modify Cloud Logs)  
**CIS Azure v2.0:** 6.5  
**NIST CSF:** DE.CM-1 (Detect — Continuous Monitoring)

**Default state:** Azure Firewall diagnostic logs (Application Rule Logs, Network Rule Logs, DNS Proxy Logs) are not enabled when the firewall is deployed.

**What this means:** Firewall rule evaluations — both allowed and denied traffic — are not recorded. Without these logs, there is no way to determine what traffic the firewall is processing, which rules are being triggered, or whether deny rules are blocking malicious activity.

**Attack path:** The risk here is less about direct exploitation and more about the absence of detection data. If an attacker is probing the network perimeter or attempting connections that should be blocked, the firewall is enforcing the rules but generating no record of doing so. Threat hunting and incident response are limited without this data.

**Remediation:** Enable diagnostic logs on all Azure Firewall instances. Route to a Log Analytics Workspace and configure Sentinel data connectors for firewall log ingestion. Set retention to match organizational compliance requirements.
