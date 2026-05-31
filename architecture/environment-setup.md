# Environment Architecture & Setup

Both environments were provisioned fresh using free-tier accounts with zero post-deployment configuration. This documents exactly what was running during assessment.

---

## Azure Environment

**Compute**
- VM size: B1s (1 vCPU, 1GB RAM)
- Image: Windows Server 2022 Datacenter
- Region: East US
- Boot diagnostics: not enabled (default)

**Networking**
- VNet: auto-created (10.0.0.0/16)
- Subnet: default (10.0.0.1/24)
- NSG: auto-created, attached to NIC
- NSG Flow Logs: disabled (default)
- Public IP: assigned

**Storage**
- Type: General Purpose v2
- Replication: LRS
- Public blob access: enabled (default)
- HTTPS only: enabled (default)
- Encryption: AES-256, platform-managed keys

**IAM**
- Default subscription-level RBAC assignments
- No PIM configured
- No MFA enforced at subscription level

**Monitoring**
- Defender for Cloud: free tier active
- Log Analytics Workspace: not configured
- Diagnostic settings: not enabled (default)

---

## GCP Environment

**Compute**
- Machine type: e2-micro (0.25 vCPU, 1GB RAM)
- Image: Debian 11 (Bullseye)
- Zone: us-central1-a
- Service account: Default Compute Engine SA (auto-attached)

**Networking**
- VPC: default network (auto-mode, 10.128.0.0/9)
- Subnet: default, us-central1
- VPC Flow Logs: disabled (default)
- External IP: ephemeral

**Default Firewall Rules at Assessment**
```
default-allow-icmp        ingress  allow  0.0.0.0/0
default-allow-internal    ingress  allow  10.128.0.0/9
default-allow-rdp         ingress  allow  0.0.0.0/0  tcp:3389
default-allow-ssh         ingress  allow  0.0.0.0/0  tcp:22
```
Firewall rule logging: disabled on all rules (default)

**Storage**
- Type: Regional bucket, us-central1
- Public access: disabled (default) ✓
- Encryption: AES-256, Google-managed keys
- CMEK: not configured

**IAM**
- Default Compute SA: `<project-number>-compute@developer.gserviceaccount.com`
- Role: `roles/editor` at project level (default)

**Monitoring**
- Admin Activity Audit Logs: enabled (default) ✓
- Data Access Audit Logs: disabled (default)
- Security Command Center: free tier
- Alert policies: none configured

---

## Network Topology

```
AZURE                                    GCP
─────────────────────────────────────────────────────────────────

Internet                                 Internet
    │                                        │
    ▼                                        ▼
Public IP                             External IP (ephemeral)
    │                                        │
    ▼                                        ▼
NSG (default)                        VPC Firewall (default)
  allow-rdp: 0.0.0.0/0 ⚠             allow-ssh:  0.0.0.0/0 ⚠
  NSG flow logs: OFF ⚠               allow-rdp:  0.0.0.0/0 ⚠
    │                                 Firewall logging: OFF ⚠
    ▼                                        │
B1s VM (10.0.0.4)                           ▼
  Diagnostic logs: OFF ⚠            e2-micro (10.128.0.x)
    │                                 Default SA: Editor role ⚠
    ▼                                        │
Storage Account                             ▼
  Public access: ON ⚠               Cloud Storage Bucket
  Access logs: OFF ⚠                 Public access: OFF ✓
    │                                 Data access logs: OFF ⚠
    ▼                                        │
Defender for Cloud                          ▼
  (free tier, default posture)       Security Command Center
                                      (free tier, default findings)

⚠ = Misconfiguration in default state
✓ = Secure by default
```

---

## Out of Scope

- PaaS services (Azure SQL, Cloud SQL, App Services, Cloud Run)
- Serverless and container workloads
- Multi-region or HA configurations
- Enterprise-tier features requiring paid plans
