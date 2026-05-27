# Azure IaaS Security Findings

**Platform:** Microsoft Azure  
**Resources Assessed:** Virtual Machine (B1s), Storage Account, NSG, IAM/RBAC, Diagnostic Settings  
**Configuration Baseline:** Default settings — no post-provisioning hardening applied

---

## AZ-001 — NSG Flow Logs Disabled by Default

**Severity:** High  
**Category:** Network Visibility / Logging  
**CIS Azure Benchmark:** Control 6.4 — Ensure that Network Security Group flow logs are captured and sent to Log Analytics  
**MITRE ATT&CK:** TA0007 – Discovery, TA0008 – Lateral Movement

**Observation:**  
When a Virtual Machine is provisioned in Azure using default settings, the associated Network Security Group (NSG) does not have flow logging enabled. This means inbound and outbound traffic patterns are not recorded unless manually configured post-deployment.

**Risk:**  
Without flow log data, there is no baseline network traffic record. An attacker who establishes initial access can move laterally across the environment without generating any network-layer audit trail. Detection of unauthorized access, port scanning, or data exfiltration is significantly delayed or impossible in the default state.

**Remediation:**  
1. Navigate to Network Watcher → NSG Flow Logs
2. Enable flow logs for all NSGs
3. Set retention to minimum 90 days (regulatory minimum for most compliance frameworks)
4. Route logs to a Log Analytics Workspace for querying
5. Enforce via Azure Policy: `Audit-NSGFlowLogs-Enabled`

---

## AZ-002 — Public Blob Access Enabled on Storage Accounts

**Severity:** High  
**Category:** Storage Security / Access Control  
**CIS Azure Benchmark:** Control 3.5 — Ensure that 'Public access level' is set to Private for blob containers  
**MITRE ATT&CK:** TA0009 – Collection, TA0010 – Exfiltration

**Observation:**  
Default Azure storage account configurations allow public blob access to be enabled at the container level. Without explicit disabling at the account level, individual containers can be set to public, potentially exposing stored data without authentication.

**Risk:**  
Publicly accessible blob containers are a common misconfiguration behind cloud data breaches. Data stored without access restrictions is indexed by public search engines and cloud scanning tools. There is no authentication or audit trail for access to public blobs.

**Remediation:**  
1. In Storage Account settings → Configuration → set "Allow Blob public access" to **Disabled**
2. Apply this at the storage account level to prevent container-level override
3. Audit existing containers: `az storage container list --account-name <name> --query "[].properties.publicAccess"`
4. Enforce via Azure Policy: `Deny-Storage-AllowBlobPublicAccess`

---

## AZ-003 — VM Diagnostic Logs Not Enabled by Default

**Severity:** Medium  
**Category:** Logging / Visibility  
**CIS Azure Benchmark:** Control 5.1 — Ensure that a Diagnostics Setting exists  
**MITRE ATT&CK:** TA0005 – Defense Evasion

**Observation:**  
Azure Virtual Machines do not enable diagnostic logs (boot diagnostics, guest OS metrics, performance counters) by default. These logs must be manually configured during or after VM deployment.

**Risk:**  
Without diagnostic logging, incident response is severely limited. Boot-level events, OS-level authentication failures, and application errors go unrecorded. In a post-incident investigation, the absence of these logs creates gaps in the forensic timeline.

**Remediation:**  
1. Enable Boot Diagnostics on all VMs (Storage Account or Managed Storage)
2. Enable Guest OS Diagnostics to collect Windows Event Logs or Linux syslog
3. Route logs to Log Analytics Workspace
4. Use Azure Monitor Agent (AMA) as replacement for legacy MMA agent
5. Enforce via Azure Policy: `Deploy-Diagnostics-VM`

---

## AZ-004 — Double Encryption Not Enforced by Default

**Severity:** Medium  
**Category:** Encryption  
**CIS Azure Benchmark:** Control 3.2 — Ensure that 'Double Encryption' is enabled for Azure managed disks

**Observation:**  
Azure supports double encryption (platform-managed key layered over customer-managed key) for managed disks, but this is not enabled by default. Default disk encryption uses platform-managed keys (PMK) only.

**Risk:**  
Single-layer encryption is sufficient for baseline protection but does not meet the defense-in-depth encryption requirements of regulated industries (HIPAA, FedRAMP High). If the platform key management system is ever compromised, there is no secondary encryption layer.

**Remediation:**  
1. Create a Disk Encryption Set with double encryption enabled
2. Assign a Customer-Managed Key via Azure Key Vault
3. Associate the Disk Encryption Set when provisioning new managed disks
4. Enforce via Azure Policy for regulated workloads

---

## AZ-005 — Over-Permissioned Default RBAC Role Assignments

**Severity:** Medium  
**Category:** Identity and Access Management  
**CIS Azure Benchmark:** Control 1.21 — Ensure that no custom subscription owner roles are created; Control 1.1 — Ensure that multi-factor authentication is enabled  
**MITRE ATT&CK:** TA0004 – Privilege Escalation

**Observation:**  
When provisioning resources via the Azure Portal, default role assignments often grant Contributor or Owner roles at the subscription or resource group level. New identities and service principals are frequently assigned broad roles rather than least-privilege scoped roles.

**Risk:**  
Contributor access at subscription scope allows a compromised identity to create, modify, or delete any resource in the subscription. This dramatically increases the blast radius of a credential compromise or insider threat.

**Remediation:**  
1. Audit current role assignments: `az role assignment list --all`
2. Remove or downscope any Contributor/Owner assignments not explicitly required
3. Use custom roles scoped to specific resource groups or resource types
4. Enable Privileged Identity Management (PIM) for just-in-time access to privileged roles
5. Require MFA for all accounts with management plane access

---

## AZ-006 — Firewall Diagnostic Logs Require Manual Activation

**Severity:** Low  
**Category:** Logging  
**CIS Azure Benchmark:** Control 6.5 — Ensure that Azure Network Firewall Policy Analytics is enabled

**Observation:**  
Azure Firewall diagnostic logs (application rules, network rules, DNS proxy) are not enabled by default and must be manually configured per firewall instance.

**Risk:**  
Without firewall logs, there is no record of which traffic was allowed or denied at the network perimeter. This limits detection of command-and-control communication, unauthorized outbound connections, or port scanning.

**Remediation:**  
1. Navigate to Azure Firewall → Diagnostic Settings → Add Diagnostic Setting
2. Enable: AzureFirewallApplicationRule, AzureFirewallNetworkRule, AzureFirewallDnsProxy
3. Route to Log Analytics Workspace
4. Create alert rules for denied traffic spikes or unusual allow patterns
