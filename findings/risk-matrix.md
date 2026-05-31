# Risk Matrix & Attack Scenarios

---

## Risk Matrix

| | **High Likelihood** | **Medium Likelihood** | **Low Likelihood** |
|---|---|---|---|
| **Critical Impact** | GCP-001 | AZ-002 | — |
| **High Impact** | AZ-001, GCP-002 | AZ-005, GCP-004 | AZ-004 |
| **Medium Impact** | GCP-003 | AZ-003, GCP-006 | AZ-006, GCP-005 |

**Likelihood:**
- High — exploitable directly from the default state with minimal access
- Medium — requires some existing access or reconnaissance
- Low — requires specific conditions or elevated skill to exploit

---

## Attack Scenarios

### GCP-001 — Default Editor Service Account
**Risk:** Critical

```
1. Attacker gains code execution on GCP VM
   (web app vulnerability, SSRF, compromised container)
     ↓
2. Query the instance metadata endpoint:
   curl "http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token"
   -H "Metadata-Flavor: Google"
     ↓
3. Retrieve OAuth2 access token for the default Compute SA
     ↓
4. Authenticate with the token — Editor role = project-wide access:
   gcloud auth activate-service-account --access-token-file=token
     ↓
5. List and read all Cloud Storage buckets, enumerate resources,
   read secrets, create backdoor service account keys for persistence
```

No credentials needed beyond initial VM access. The entire path uses default GCP behavior.

---

### AZ-002 — Public Blob Access Enabled by Default
**Risk:** High

```
1. Attacker identifies target Azure storage account
   (DNS enumeration, leaked URL in code/config, or cloud scanning tools)
     ↓
2. Confirm public access:
   curl "https://<account>.blob.core.windows.net/<container>?restype=container&comp=list"
     ↓
3. List all blobs without authentication
     ↓
4. Download any blob directly — no auth required:
   curl "https://<account>.blob.core.windows.net/<container>/<blob>"
     ↓
5. Data exfiltration complete — no authentication, no audit log entry
```

---

### AZ-001 / GCP-002 — No Flow Logs (Both Platforms)
**Risk:** High

```
1. Attacker establishes initial foothold on any VM
     ↓
2. Internal reconnaissance:
   nmap -sn 10.0.0.0/24
   (no flow logs = not detected)
     ↓
3. Identify other VMs, databases, internal services
     ↓
4. Lateral movement to high-value targets
   (no network-layer audit trail in default state)
     ↓
5. Data exfiltration over standard ports (443, 80)
   (no egress anomaly detection)
```

Without flow logs, the entire kill chain from initial access to exfiltration is invisible at the network layer.

---

### AZ-005 — Over-Permissioned RBAC
**Risk:** Medium-High

```
1. Attacker compromises an Azure account with Contributor
   role at subscription scope (phishing, brute force, credential leak)
     ↓
2. Full access to create/modify/delete any resource in the subscription
     ↓
3. Deploy a new VM with a managed identity
     ↓
4. Use managed identity to access Key Vault, Storage, databases
     ↓
5. Exfiltrate data or create a backdoor service principal for persistence
```

One compromised Contributor account = full subscription access.

---

### GCP-003 — Data Access Audit Logs Disabled
**Risk:** Medium

```
1. Attacker obtains a service account with Storage Object Viewer permission
   (credential leak, IAM misconfiguration, stolen from metadata)
     ↓
2. List and download objects:
   gsutil ls gs://
   gsutil cp gs://<bucket>/<file> .
     ↓
3. Data exfiltration complete — Data Access logs are OFF by default
   No audit entry. No record of what was accessed or when.
```

---

### GCP-004 — Firewall Logging Disabled + Default Allow SSH
**Risk:** Medium

```
1. Attacker scans GCP IP ranges for open port 22
   Default VPC allows SSH from 0.0.0.0/0 — publicly reachable
     ↓
2. Brute force SSH or use a leaked key
   (No firewall logging = brute force attempts not recorded)
     ↓
3. Successful login — attacker on the VM
     ↓
4. From here: GCP-001 path applies
   Metadata endpoint → Editor token → full project access
```

---

### AZ-003 — VM Diagnostic Logs Disabled
**Risk:** Medium — Incident Response Impact

```
Security event occurs on Azure VM
(unauthorized login, malware, privilege escalation attempt)
  ↓
Responder opens Azure Monitor
  ↓
No boot diagnostics. No OS logs. No auth events. No process data.
  ↓
Cannot determine:
  - When the event occurred
  - What ran on the system
  - What auth preceded the compromise
  - Whether the VM was modified or rebooted
  ↓
Investigation stalls. Scope unknown. Dwell time unknown.
```

---

## Findings by NIST CSF Function

| CSF Function | Findings | Note |
|---|---|---|
| Identify | AZ-005, GCP-001 | Over-privileged identities |
| Protect | AZ-002, AZ-004, GCP-005 | Access control and encryption gaps |
| Detect | AZ-001, AZ-003, AZ-006, GCP-002, GCP-003, GCP-004, GCP-006 | 7 of 12 findings |
| Respond | AZ-003, GCP-003 | Insufficient logs for investigation |
| Recover | — | No findings |

58% of findings are in the Detect category. Both platforms default to a near-zero detection posture.
