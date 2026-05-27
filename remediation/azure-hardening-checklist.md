# Azure IaaS Hardening Checklist

Mapped to CIS Azure Benchmark v2.0. Apply these controls immediately after provisioning any IaaS environment.

---

## IAM and Access Control

- [ ] **AZ-IAM-01** — Audit all subscription-level role assignments; remove any Contributor/Owner not explicitly required  
  *CIS: 1.21 | NIST: PR.AC-4*

- [ ] **AZ-IAM-02** — Replace broad role assignments with custom roles scoped to specific resource groups  
  *CIS: 1.14 | NIST: PR.AC-4*

- [ ] **AZ-IAM-03** — Enable Privileged Identity Management (PIM) for just-in-time access to Owner and Contributor roles  
  *CIS: 1.14 | NIST: PR.AC-6*

- [ ] **AZ-IAM-04** — Enforce MFA for all accounts with management plane access  
  *CIS: 1.1 | NIST: PR.AC-1*

- [ ] **AZ-IAM-05** — Use Managed Identities for all service-to-service authentication; eliminate stored credentials  
  *CIS: 1.20 | NIST: PR.AC-1*

- [ ] **AZ-IAM-06** — Enable Azure AD Conditional Access policies; restrict management console access by location and device compliance  
  *CIS: 1.2 | NIST: PR.AC-4*

---

## Network Security

- [ ] **AZ-NET-01** — Enable NSG Flow Logs for all Network Security Groups; retain for minimum 90 days  
  *CIS: 6.4 | NIST: DE.CM-1*

- [ ] **AZ-NET-02** — Remove or restrict any NSG rules allowing inbound access from 0.0.0.0/0  
  *CIS: 6.1 | NIST: PR.AC-5*

- [ ] **AZ-NET-03** — Enable Azure Firewall diagnostic logs (Application Rules, Network Rules, DNS Proxy)  
  *CIS: 6.5 | NIST: DE.CM-1*

- [ ] **AZ-NET-04** — Enable DDoS Standard protection for public-facing resources  
  *CIS: 6.6 | NIST: PR.DS-4*

- [ ] **AZ-NET-05** — Restrict management port access (SSH 22, RDP 3389) using Just-In-Time VM Access  
  *CIS: 7.2 | NIST: PR.AC-3*

---

## Storage Security

- [ ] **AZ-STG-01** — Disable public blob access at the storage account level for all accounts  
  `az storage account update --name <NAME> --allow-blob-public-access false`  
  *CIS: 3.5 | NIST: PR.DS-1*

- [ ] **AZ-STG-02** — Enforce HTTPS-only on all storage accounts  
  *CIS: 3.1 | NIST: PR.DS-2*

- [ ] **AZ-STG-03** — Enable soft delete for blobs and containers (minimum 7-day retention)  
  *CIS: 3.8 | NIST: PR.DS-1*

- [ ] **AZ-STG-04** — Restrict storage account access using Service Endpoints or Private Endpoints  
  *CIS: 3.7 | NIST: PR.AC-5*

- [ ] **AZ-STG-05** — Enable storage account logging for Read, Write, and Delete operations  
  *CIS: 3.10 | NIST: DE.CM-7*

---

## Encryption and Key Management

- [ ] **AZ-ENC-01** — Configure Customer-Managed Keys (CMK) for all storage accounts handling regulated data  
  *CIS: 3.2 | NIST: PR.DS-1*

- [ ] **AZ-ENC-02** — Enable double encryption for managed disks on sensitive VMs  
  *CIS: 3.2 | NIST: PR.DS-1*

- [ ] **AZ-ENC-03** — Configure key rotation policy in Azure Key Vault (maximum 90-day rotation recommended)  
  *CIS: 8.1 | NIST: PR.DS-1*

- [ ] **AZ-ENC-04** — Enable purge protection and soft delete on all Key Vaults  
  *CIS: 8.4 | NIST: PR.DS-1*

- [ ] **AZ-ENC-05** — Restrict Key Vault access using Private Endpoints; remove public network access  
  *CIS: 8.6 | NIST: PR.AC-5*

---

## Logging and Monitoring

- [ ] **AZ-LOG-01** — Enable Diagnostic Settings on all VMs, NSGs, storage accounts, and Key Vaults  
  *CIS: 5.1 | NIST: DE.CM-1*

- [ ] **AZ-LOG-02** — Route all logs to a centralized Log Analytics Workspace  
  *CIS: 5.3 | NIST: DE.CM-7*

- [ ] **AZ-LOG-03** — Enable Microsoft Defender for Cloud on all subscriptions (Standard tier for production)  
  *CIS: 2.1 | NIST: DE.CM-4*

- [ ] **AZ-LOG-04** — Configure Microsoft Sentinel with data connectors for: Azure AD, Azure Activity, Microsoft Defender  
  *CIS: 5.3 | NIST: DE.AE-1*

- [ ] **AZ-LOG-05** — Create alert rules in Sentinel for: IAM changes, resource deletions, failed authentications, privilege escalation  
  *CIS: 5.2 | NIST: DE.AE-3*

- [ ] **AZ-LOG-06** — Set log retention to minimum 1 year in Log Analytics Workspace (regulatory requirement for HIPAA/FedRAMP)  
  *CIS: 5.1 | NIST: RS.AN-1*
